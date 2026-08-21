# Parte 1: Fundamentos de React

## Capítulo 10: Detrás de Escena de React y Oportunidades de Optimización

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Evitar la ejecución innecesaria de código mediante los Hooks `useMemo()` y `useCallback()`.
- Cargar código opcional de forma perezosa (*lazy loading*), solo cuando sea necesario, mediante la función `lazy()` de React.
- Utilizar las herramientas de desarrollo de React (*React Developer Tools*) para analizar y optimizar tu aplicación.
- Explorar el compilador de React (*React Compiler*) para mejoras automáticas de rendimiento.

---

### Sección 1: Introducción

Utilizando todas las características cubiertas hasta este punto, puedes crear aplicaciones de React no triviales y, por lo tanto, interfaces de usuario altamente interactivas y reactivas.

Este capítulo, aunque presenta algunas funciones y conceptos nuevos, no te proporcionará herramientas que te permitan crear aplicaciones web aún más avanzadas. No aprenderás sobre conceptos clave e innovadores como el estado o las props (aunque aprenderás sobre conceptos más avanzados en capítulos posteriores).

En su lugar, este capítulo te permite mirar **detrás de escena de React**. Aprenderás cómo React calcula las actualizaciones requeridas del DOM y cómo se asegura de que dichas actualizaciones ocurran sin afectar el rendimiento de una manera inaceptable. También aprenderás sobre algunas otras técnicas de optimización empleadas por React, todas con el objetivo de garantizar que tu aplicación de React se ejecute de la mejor manera posible.

Además de esta mirada entre bastidores, aprenderás sobre varias funciones y conceptos integrados que se pueden utilizar para optimizar aún más el rendimiento de la aplicación. Este capítulo no solo presentará esos conceptos, sino que también explicará por qué existen, cómo deben usarse y cuándo usar cada característica.

---

### Sección 2: Revisitando las evaluaciones y actualizaciones de componentes

Antes de explorar el funcionamiento interno de React, tiene sentido revisar brevemente la lógica de React para ejecutar funciones de componentes.

Las funciones de los componentes se ejecutan cada vez que cambia algún estado (administrado a través de `useState()`) o cuando la función de su componente principal (*parent*) se ejecuta nuevamente. Esto último sucede porque, si se llama a una función de componente principal, todo su código JSX (que apunta a la función del componente secundario o *child*) se reevalúa. Cualquier función de componente a la que se haga referencia en ese código JSX también se invoca nuevamente.

Considera una estructura de componentes como esta:

```javascript
function NestedChild() { 
  console.log('<NestedChild /> is called.'); 
  return ( 
    <p id="nested-child"> 
      A component, deeply nested into the component tree. 
    </p> 
  ); 
} 

function Child() { 
  console.log('<Child /> is called.'); 
  return ( 
    <div id="child"> 
      <p> 
        A component, rendered inside another component, containing yet another component. 
      </p> 
      <NestedChild /> 
    </div> 
  ); 
} 

function Parent() { 
  console.log('<Parent /> is called.'); 
  const [counter, setCounter] = useState(0); 

  function handleIncCounter() { 
    setCounter((prevCounter) => prevCounter + 1); 
  } 

  return ( 
    <div id="parent"> 
      <p> 
        A component, nested into App, containing another component (Child). 
      </p> 
      <p>Counter: {counter}</p> 
      <button onClick={handleIncCounter}>Increment</button> 
      <Child /> 
    </div> 
  ); 
}
```

En esta estructura de ejemplo, el componente `Parent` renderiza un `<div>` con dos párrafos, un botón y otro componente: el componente `Child`. Ese componente a su vez genera un `<div>` con un párrafo y otro componente más: el componente `NestedChild` (que luego solo genera un párrafo).

El componente `Parent` también gestiona algún estado (un contador de prueba), que cambia cada vez que se hace clic en el botón. Los tres componentes imprimen un mensaje a través de `console.log()`, simplemente para que sea fácil detectar cuándo React llama a cada componente.

La siguiente captura de pantalla muestra esos componentes en acción, después de hacer clic en el botón:

**Figura 10.1**: Se ejecuta cada función de componente.

En esta captura de pantalla, no solo puedes ver cómo los componentes están anidados entre sí, sino también cómo React los invoca a todos cuando se hace clic en el botón `Increment`. Se invoca a `Child` y `NestedChild` aunque no administren ni utilicen ningún estado. Pero dado que son un elemento secundario (`Child`) o descendiente (`NestedChild`) del componente `Parent`, que sí recibió un cambio de estado, las funciones de los componentes anidados también se llaman.

Comprender este flujo de ejecución de funciones de componentes es importante porque este flujo implica que cualquier invocación de función de componente también influye en sus componentes descendientes. También te muestra con qué frecuencia React puede invocar funciones de componentes y cuántas funciones de componentes pueden verse afectadas por un solo cambio de estado.

Por lo tanto, hay una pregunta importante que debe responderse: ¿qué sucede con el DOM de la página real (es decir, con el sitio web cargado y renderizado en el navegador) cuando se invocan una o más funciones de componentes? ¿Se vuelve a crear el DOM? ¿Se actualiza la interfaz de usuario renderizada?

#### Qué sucede cuando se llama a una función de componente
Cada vez que se ejecuta una función de componente, **React evalúa si la interfaz de usuario renderizada (es decir, el DOM de la página cargada) debe actualizarse o no**.

Esto es importante: **React evalúa si se necesita una actualización. ¡No fuerza una actualización automáticamente!**

Internamente, React no toma el código JSX devuelto por un componente (o múltiples componentes) y reemplaza el DOM de la página con él.

Eso se podría hacer, pero significaría que cada ejecución de función de componente conduciría a algún tipo de manipulación del DOM, incluso si es solo un reemplazo del contenido del DOM antiguo por una versión nueva y similar. En el ejemplo mostrado anteriormente, el código JSX de `Child` y `NestedChild` se usaría para reemplazar el DOM renderizado actualmente cada vez que se ejecutaran esas funciones de componentes.

Como puedes ver en el ejemplo anterior, esas funciones de componentes se ejecutan con bastante frecuencia. Pero el código JSX devuelto es siempre el mismo porque es estático: no contiene ningún valor dinámico (por ejemplo, estado o props).

Si el DOM de la página real se reemplazara con los elementos del DOM implícitos en el código JSX devuelto, el resultado visual siempre sería el mismo. Pero todavía habría alguna manipulación del DOM detrás de escena. Y eso es un problema porque **manipular el DOM es una tarea que consume bastante rendimiento**, especialmente cuando se realiza con una frecuencia alta. Por lo tanto, eliminar y agregar o actualizar elementos del DOM solo debe hacerse cuando sea necesario, no innecesariamente.

Debido a esto, React no desecha el DOM actual y lo reemplaza con el nuevo DOM (implícito en el código JSX) solo porque se ejecutó una función de componente. En su lugar, React primero verifica si se necesita una actualización. Y si es necesaria, **solo se reemplazan o actualizan las partes del DOM que necesitan cambiar**.

Para determinar si se necesita una actualización (y dónde), React utiliza un concepto llamado **DOM Virtual** (*Virtual DOM*).

---

### Sección 3: El DOM Virtual frente al DOM Real

Para determinar si (y dónde) podría ser necesaria una actualización del DOM, React (específicamente, el paquete `react-dom`) compara la estructura del DOM actual con la estructura implícita en el código JSX devuelto por las funciones de los componentes ejecutados. Si hay una diferencia, el DOM se actualiza en consecuencia; de lo contrario, se deja intacto.

Sin embargo, así como manipular el DOM consume un rendimiento relativamente alto, leer el DOM también lo hace. Incluso sin cambiar nada en el DOM, acceder a él, recorrer los elementos del DOM y derivar la estructura a partir de él es algo que normalmente deseas reducir al mínimo.

Si se ejecutan múltiples funciones de componentes y cada una desencadena un proceso en el que los elementos del DOM renderizados se leen y comparan con la estructura JSX implícita en las funciones de los componentes invocados, el DOM renderizado se verá afectado por operaciones de lectura múltiples veces en un período muy corto.

Para aplicaciones de React más grandes compuestas por docenas, cientos o incluso miles de componentes, es muy probable que se produzcan docenas de ejecuciones de funciones de componentes en un solo segundo. Si eso provocara la misma cantidad de operaciones de lectura del DOM, existe una probabilidad bastante alta de que la aplicación web se sienta lenta para el usuario.

Por eso **React no utiliza el DOM real para determinar si se necesita alguna actualización de la interfaz de usuario**. En su lugar, construye y administra internamente un **DOM Virtual**, una representación en memoria del DOM que se procesa en el navegador. El DOM Virtual no es una función del navegador, sino una función de React. Puedes pensar en él como un objeto JavaScript profundamente anidado que refleja los componentes de tu aplicación web, incluidos todos los componentes HTML integrados como `<div>`, `<p>`, etc. (es decir, los elementos HTML reales que deberían aparecer en la página al final).

**Figura 10.2**: React administra una representación virtual de la estructura de elementos esperada.

En la figura anterior, puedes ver que la estructura de elementos esperada (en otras palabras, el DOM final esperado) se almacena en realidad como un objeto JavaScript (o un array con una lista de objetos). Este es el DOM Virtual, que es administrado por React y utilizado para identificar las actualizaciones requeridas del DOM.

> [!NOTE]
> Ten en cuenta que la estructura real del DOM Virtual es más compleja que la estructura que se muestra en la imagen. El gráfico anterior tiene como objetivo darte una idea de qué es el DOM Virtual y cómo podría verse. No es una representación técnica exacta de la estructura de datos de JavaScript administrada por React.

React administra este DOM Virtual porque comparar este DOM Virtual con el estado esperado de la interfaz de usuario consume mucho menos rendimiento que comunicarse con el DOM real.

Cada vez que se llama a una función de componente, React compara el código JSX devuelto con los nodos respectivos del DOM Virtual almacenados en el DOM Virtual. Si se detectan diferencias, React determinará qué cambios se necesitan para actualizar el DOM. Una vez derivados los ajustes requeridos, **estos cambios se aplican tanto al DOM Virtual como al DOM Real**. Esto garantiza que el DOM real refleje el estado esperado de la interfaz de usuario sin tener que comunicarse con él ni actualizarlo constantemente.

**Figura 10.3**: React detecta las actualizaciones requeridas a través del DOM Virtual.

En la figura anterior, puedes ver cómo React compara primero el DOM actual y la estructura esperada con la ayuda del DOM Virtual, antes de comunicarse con el DOM real para manipularlo en consecuencia.

Como desarrollador de React, no necesitas interactuar activamente con el DOM Virtual. Técnicamente, ni siquiera necesitas saber que existe y que React lo utiliza internamente. Pero siempre es útil comprender la herramienta con la que estás trabajando. Es bueno saber que React realiza varias optimizaciones de rendimiento por ti y que las obtienes además de muchas otras características que (con suerte) hacen tu vida como desarrollador más fácil.

#### Agrupación por lotes de estado (*State Batching*)
Dado que React utiliza el concepto de un DOM Virtual, las ejecuciones frecuentes de funciones de componentes no son un gran problema. Pero, por supuesto, incluso si las comparaciones solo se realizan virtualmente, todavía hay algún código interno que debe ejecutarse. Incluso con el DOM Virtual, el rendimiento podría degradarse si se deben realizar muchas comparaciones innecesarias (y al mismo tiempo bastante complejas) del DOM Virtual.

Un escenario en el que se realizan comparaciones innecesarias es en la ejecución de múltiples actualizaciones de estado secuenciales. Dado que cada actualización de estado hace que la función del componente se ejecute nuevamente (así como todos los posibles componentes anidados), múltiples actualizaciones de estado que se realizan juntas (por ejemplo, en la misma función controladora de eventos) provocarían múltiples invocaciones de la función del componente.

Considera este ejemplo:

```javascript
function App() { 
  const [counter, setCounter] = useState(0); 
  const [showCounter, setShowCounter] = useState(false); 

  function handleIncCounter() { 
    setCounter((prevCounter) => prevCounter + 1); 
    setShowCounter(true); 
  } 

  return ( 
    <> 
      <p>Click to increment + show or hide the counter</p> 
      <button onClick={handleIncCounter}>Increment</button> 
      {showCounter && <p>Counter: {counter}</p>} 
    </> 
  ); 
}
```

Este componente contiene dos valores de estado: `counter` y `showCounter`. Cuando se hace clic en el botón, el contador se incrementa en 1. Además, `showCounter` se establece en `true`. Por lo tanto, la primera vez que se hace clic en el botón, se modifican tanto el estado de `counter` como el de `showCounter` (porque `showCounter` es `false` inicialmente).

Dado que se modifican dos valores de estado, la expectativa sería que React llame dos veces a la función del componente `App`, porque cada actualización de estado hace que la función del componente conectado se invoque nuevamente.

Sin embargo, si agregas una declaración `console.log()` a la función del componente `App` (para rastrear con qué frecuencia se ejecuta), verás que solo se invoca **una vez** cuando se hace clic en el botón `Increment`:

**Figura 10.4**: Solo se muestra un mensaje de registro en la consola.

> [!NOTE]
> Si ves dos mensajes de registro en lugar de uno, asegúrate de no estar utilizando el "Modo Estricto" (*Strict Mode*) de React. Cuando se ejecuta en Modo Estricto durante el desarrollo, React ejecuta las funciones de los componentes con más frecuencia de lo normal.
> Si es necesario, puedes desactivar el Modo Estricto eliminando el componente `<React.StrictMode>` de tu archivo `main.jsx`. Aprenderás más sobre el Modo Estricto de React hacia el final de este capítulo.

Este comportamiento se llama **agrupación por lotes de estado (*state batching*)**. React realiza la agrupación por lotes de estado cuando se inician múltiples actualizaciones de estado desde el mismo lugar en tu código (por ejemplo, desde dentro de la misma función controladora de eventos).

Es una funcionalidad integrada que garantiza que las funciones de tus componentes no se llamen más veces de las necesarias. Esto evita comparaciones innecesarias del DOM Virtual.

La agrupación por lotes de estado es un mecanismo muy útil. Pero hay otro tipo de evaluación innecesaria de componentes que no previene: **las funciones de los componentes secundarios que se ejecutan cuando se llama a la función del componente principal**.

#### Evitar evaluaciones innecesarias de componentes secundarios
Cada vez que se invoca una función de componente (porque su estado cambió, por ejemplo), cualquier función de componente anidada también se llamará. Consulta la primera sección de este capítulo para obtener más detalles.

Como viste en el ejemplo de la primera sección de este capítulo, a menudo ocurre que esos componentes anidados en realidad no necesitan evaluarse nuevamente. Es posible que no dependan del valor de estado que cambió en el componente principal. Es posible que ni siquiera dependan de ningún valor del componente principal en absoluto.

Aquí hay un ejemplo donde la función del componente principal contiene algún estado que no es utilizado por el componente secundario:

```javascript
function Error({ message }) { 
  if (!message) { 
    return null; 
  } 
  return <p className={classes.error}>{message}</p>; 
} 

function Form() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  const [errorMessage, setErrorMessage] = useState(); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleSubmit(event) { 
    event.preventDefault(); 
    if (!enteredEmail.endsWith('.com')) { 
      setErrorMessage('Email must end with .com.'); 
    } 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmit}> 
      <div className={classes.control}> 
        <label htmlFor="email">Email</label> 
        <input id="email" type="email" value={enteredEmail} onChange={handleUpdateEmail} /> 
      </div> 
      <Error message={errorMessage} /> 
      <button>Sign Up</button> 
    </form> 
  ); 
}
```

> [!NOTE]
> Puedes encontrar el código de ejemplo completo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/examples/03-memo](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/examples/03-memo).

En este ejemplo, el componente `Error` se basa en la prop `message`, que se establece en el valor almacenado en el estado `errorMessage` del componente `Form`. Sin embargo, el componente `Form` también gestiona un estado `enteredEmail`, que no es utilizado (ni recibido a través de props) por el componente `Error`. Por lo tanto, los cambios en el estado `enteredEmail` harán que el componente `Error` se ejecute nuevamente, a pesar de que el componente no necesita ese valor.

Puedes rastrear las invocaciones innecesarias de funciones del componente `Error` agregando una declaración `console.log()` a esa función del componente:

```javascript
function Error({ message }) { 
  console.log('<Error /> component function is executed.'); 
  if (!message) { 
    return null; 
  } 
  return <p className={classes.error}>{message}</p>; 
}
```

**Figura 10.5**: La función del componente `Error` se ejecuta para cada pulsación de tecla.

En la captura de pantalla anterior, puedes ver que la función del componente `Error` se ejecuta para cada pulsación de tecla en el campo de entrada (es decir, una vez para cada cambio de estado de `enteredEmail`).

Esto coincide con lo aprendido anteriormente, pero también es innecesario. El componente `Error` sí depende del estado `errorMessage` y ciertamente debe reevaluarse cada vez que ese estado cambie, pero ejecutar la función del componente `Error` porque se actualizó el valor del estado `enteredEmail` claramente no es necesario.

Es por eso que React ofrece otra función integrada que puedes usar para controlar (y prevenir) este comportamiento: la función **`memo()`**.

`memo` se importa de `react` y se utiliza de la siguiente manera:

```javascript
import { memo } from 'react'; 
import classes from './Error.module.css'; 

function Error({ message }) { 
  console.log('<Error /> component function is executed.'); 
  if (!message) { 
    return null; 
  } 
  return <p className={classes.error}>{message}</p>; 
} 

export default memo(Error);
```

Envuelves la función del componente que debe protegerse de reevaluaciones innecesarias iniciadas por el elemento principal con `memo()`. **Esto hace que React verifique si las props del componente cambiaron**, en comparación con la última vez que se llamó a la función del componente. Si los valores de las props son iguales, React sabe que la función del componente no necesita ejecutarse nuevamente.

Al agregar `memo()`, se evitan invocaciones innecesarias de funciones de componentes, como se muestra a continuación:

**Figura 10.6**: No aparecen mensajes de registro en la consola.

Como puedes ver en la figura, no se imprimen mensajes en la consola. Esto demuestra que se evitan ejecuciones innecesarias de componentes (recuerda: antes de agregar `memo()`, se imprimían muchos mensajes en la consola).

`memo()` también acepta un segundo argumento opcional que se puede utilizar para agregar tu propia lógica y determinar si los valores de las props han cambiado o no. Esto puede resultar útil si trabajas con valores de props más complejos (por ejemplo, objetos o arrays) donde podría ser necesaria una lógica de comparación personalizada, como en el siguiente ejemplo:

```javascript
memo(SomeComponent, function(prevProps, nextProps) { 
  return prevProps.user.firstName !== nextProps.user.firstName; 
});
```

El segundo argumento (opcional) pasado a `memo()` debe ser una función que recibe automáticamente el objeto de props anterior y el siguiente objeto de props. La función luego debe devolver `true` si el componente (`SomeComponent`, en este ejemplo) debe reevaluarse y `false` si no debe hacerlo.

A menudo, el segundo argumento no es necesario porque el comportamiento predeterminado de `memo()` (donde compara todas las props por desigualdad) es exactamente lo que necesitas. Pero si se necesita más personalización o control, `memo()` te permite agregar tu lógica personalizada.

Con `memo()` en tu caja de herramientas, es tentador envolver cada función de componente de React con `memo()`. ¿Por qué no lo harías? Después de todo, evita ejecuciones innecesarias de funciones de componentes.

Definitivamente puedes usarlo en todos los componentes, pero **no es necesariamente útil hacerlo porque evitar reevaluaciones innecesarias de componentes usando `memo()` tiene un costo**: comparar props (antiguas frente a nuevas) también requiere que se ejecute algo de código. No es "gratis". Sin embargo, no es un costo enorme: usar `memo()` en muchos (o todos) los componentes probablemente no ralentizará tu aplicación de manera significativa. Pero sigue siendo innecesario si tienes componentes que sí necesitan ser reevaluados mucho. Usar `memo()` en componentes que reciben props que cambian mucho simplemente no aporta nada útil.

Por lo tanto, `memo()` tiene más sentido si tienes props relativamente simples (es decir, props sin objetos profundamente anidados que necesitas comparar manualmente con una función de comparación personalizada) y la mayoría de los cambios de estado del componente principal no afectan esas props del componente secundario. E incluso en esos casos, si tienes una función de componente relativamente simple (es decir, sin ninguna lógica compleja en ella), usar `memo()` aún podría no generar ningún beneficio medible.

El código de ejemplo anterior (el componente `Error`) es un buen ejemplo: en teoría, usar `memo()` tiene sentido aquí. La mayoría de los cambios de estado en el componente principal no afectarán a `Error`, y la comparación de props será muy simple porque solo se debe comparar una prop (la prop `message`, que contiene una cadena). Pero a pesar de eso, es muy probable que no valga la pena usar `memo()` para envolver `Error`. `Error` es un componente extremadamente básico sin lógica compleja. Simplemente no importa si la función del componente se invoca con frecuencia. Por lo tanto, usar `memo()` en este punto sería absolutamente aceptable, pero también lo es no usarlo.

Un excelente lugar para usar `memo()`, por otro lado, es un componente que está **relativamente cerca de la parte superior del árbol de componentes** (o de una rama de componentes profundamente anidada en el árbol de componentes). Si puedes evitar ejecuciones innecesarias de ese componente a través de `memo()`, también evitarás implícitamente ejecuciones innecesarias de todos los componentes anidados debajo de ese componente. Esto se ilustra en el siguiente diagrama:

**Figura 10.7**: Uso de `memo` al inicio de una rama del árbol de componentes.

En la figura anterior, `memo()` se utiliza en el componente `Shop`, que tiene múltiples componentes descendientes anidados. Sin `memo()`, cada vez que se invocara la función del componente `Shop`, también se ejecutarían `Products`, `ProdItem`, `Cart`, etc. Con `memo()`, asumiendo que puede evitar algunas ejecuciones innecesarias de la función del componente `Shop`, todos esos componentes descendientes ya no se evalúan.

#### Evitar cálculos costosos
La función `memo()` puede ayudar a evitar ejecuciones innecesarias de funciones de componentes. Como se mencionó en la sección anterior, esto es especialmente valioso si una función de componente realiza mucho trabajo (por ejemplo, ordenar una lista larga).

Pero como desarrollador de React, también encontrarás situaciones en las que tienes un componente con un uso intensivo de trabajo que debe ejecutarse nuevamente porque cambió algún valor de prop. En tales casos, el uso de `memo()` no evitará que la función del componente se ejecute nuevamente. Sin embargo, es posible que la prop que cambió no sea necesaria para la tarea de rendimiento intensivo que se realiza como parte del componente.

Considera el siguiente ejemplo:

```javascript
function sortItems(items) { 
  console.log('Sorting'); 
  return items.sort(function (a, b) { 
    if (a.id > b.id) { 
      return 1; 
    } else if (a.id < b.id) { 
      return -1; 
    } 
    return 0; 
  }); 
} 

function List({ items, maxNumber }) { 
  const sortedItems = sortItems(items); 
  const listItems = sortedItems.slice(0, maxNumber); 

  return ( 
    <ul> 
      {listItems.map((item) => ( 
        <li key={item.id}> 
          {item.title} (ID: {item.id}) 
        </li> 
      ))} 
    </ul> 
  ); 
} 

export default List;
```

El componente `List` recibe dos valores de props: `items` y `maxNumber`. Luego llama a `sortItems()` para ordenar los elementos por `id`. A partir de entonces, la lista ordenada se limita a una cierta cantidad (`maxNumber`) de elementos. Como último paso, la lista ordenada y acortada se renderiza en la pantalla a través de `map()` en el código JSX.

> [!NOTE]
> Se puede encontrar una aplicación de ejemplo completa en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/examples/04-usememo](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/examples/04-usememo).

Dependiendo del número de elementos pasados al componente `List`, ordenarlo puede llevar una cantidad significativa de tiempo (para listas muy largas, incluso hasta unos pocos segundos). Definitivamente no es una operación que desees realizar innecesariamente o con demasiada frecuencia. La lista debe ordenarse siempre que cambie `items`, pero no debe ordenarse si cambia `maxNumber`, porque esto no afecta los elementos de la lista (es decir, no afecta el orden). Pero con el fragmento de código compartido anteriormente, `sortItems()` se ejecutará cada vez que cambie cualquiera de los dos valores de props, sin importar si es `items` o `maxNumber`.

Como resultado, al ejecutar la aplicación y cambiar la cantidad de elementos mostrados, puedes ver múltiples mensajes de registro `"Sorting"`, lo que implica que `sortItems()` se ejecutó cada vez que se cambió la cantidad de elementos.

**Figura 10.8**: Aparecen múltiples mensajes de registro "Sorting" en la consola.

La función `memo()` no ayudará aquí porque la función del componente `List` debería ejecutarse (y se ejecutará) cada vez que cambie `items` o `maxNumber`. `memo()` no ayuda a controlar la ejecución parcial de código dentro de la función del componente.

Para eso, puedes usar otra función proporcionada por React: el Hook **`useMemo()`**.

`useMemo()` se puede utilizar para **envolver una operación intensiva de cómputo**. Para que funcione correctamente, también debes definir una lista de dependencias que deberían hacer que la operación se ejecute nuevamente. Hasta cierto punto, es similar a `useEffect()` (que también envuelve una operación y define una lista de dependencias), pero la diferencia clave es que **`useMemo()` se ejecuta al mismo tiempo que el resto del código en la función del componente**, mientras que `useEffect()` ejecuta la lógica envuelta después de que ha finalizado la ejecución de la función del componente. `useEffect()` no debe usarse para optimizar tareas con uso intensivo de cómputo, sino para efectos secundarios.

`useMemo()`, por otro lado, existe para controlar la ejecución de tareas de rendimiento intensivo. Aplicado al ejemplo mencionado anteriormente, el código se puede ajustar de la siguiente manera:

```javascript
import { useMemo } from 'react'; 

function List({ items, maxNumber }) { 
  const sortedItems = useMemo( 
    function() { 
      console.log('Sorting'); 
      return items.sort(function (a, b) { 
        if (a.id > b.id) { 
          return 1; 
        } else if (a.id < b.id) { 
          return -1; 
        } 
        return 0; 
      }); 
    }, 
    [items] 
  ); 

  const listItems = sortedItems.slice(0, maxNumber); 

  return ( 
    <ul> 
      {listItems.map((item) => ( 
        <li key={item.id}> 
          {item.title} (ID: {item.id}) 
        </li> 
      ))} 
    </ul> 
  ); 
} 

export default List;
```

`useMemo()` envuelve una función anónima (la función que antes existía como función con nombre, `sortItems`), que contiene todo el código de ordenamiento. El segundo argumento pasado a `useMemo()` es el array de dependencias para las cuales la función debe ejecutarse nuevamente (cuando cambia un valor de dependencia). En este caso, `items` es la única dependencia de la función envuelta, por lo que ese valor se agrega al array.

Con `useMemo()` utilizado de esta manera, la lógica de ordenamiento solo se ejecutará cuando cambie `items`, no cuando cambie `maxNumber` (o cualquier otra cosa). Como resultado, ves que `"Sorting"` se emite en la consola de herramientas de desarrollo solo una vez:

**Figura 10.9**: Solo una salida de "Sorting" en la consola.

`useMemo()` puede ser muy útil para controlar la ejecución de código dentro de las funciones de tus componentes. Puede ser una gran adición a `memo()` (que controla la ejecución general de la función del componente). Pero, al igual que con `memo()`, no deberías comenzar a envolver toda tu lógica con `useMemo()`. Úsalo solo para cálculos muy intensivos en rendimiento, ya que verificar los cambios de dependencia y almacenar y recuperar resultados de cálculos anteriores (lo que `useMemo()` hace internamente) también conlleva un costo de rendimiento.

#### Utilización de `useCallback()`
En los capítulos anteriores, aprendiste sobre `useCallback()`. Así como `useMemo()` se puede utilizar para cálculos "costosos", `useCallback()` se puede utilizar para evitar recreaciones innecesarias de funciones. En el contexto de este capítulo, `useCallback()` puede ser útil porque, junto con `memo()` o `useMemo()`, puede ayudarte a evitar la ejecución innecesaria de código. Puede ayudarte en los casos en que una función se pasa como prop (es decir, donde podrías usar `memo()`) o se usa como una dependencia en algún cálculo "costoso" (es decir, posiblemente resuelto mediante `useMemo()`).

Aquí hay un ejemplo donde `useCallback()` se puede combinar con `memo()` para evitar ejecuciones innecesarias de funciones de componentes:

```javascript
import { memo } from 'react'; 
import classes from './Error.module.css'; 

function Error({ message, onClearError }) { 
  console.log('<Error /> component function is executed.'); 
  if (!message) { 
    return null; 
  } 
  return ( 
    <div className={classes.error}> 
      <p>{message}</p> 
      <button className={classes.errorBtn} onClick={onClearError}>X</button> 
    </div> 
  ); 
} 

export default memo(Error);
```

El componente `Error` está envuelto con la función `memo()` y, por lo tanto, solo se ejecutará si cambia uno de los valores de props recibidos.

El componente `Error` es utilizado por otro componente, el componente `Form`, de esta manera:

```javascript
function Form() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  const [errorMessage, setErrorMessage] = useState(); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleSubmit(event) { 
    event.preventDefault(); 
    if (!enteredEmail.endsWith('.com')) { 
      setErrorMessage('Email must end with .com.'); 
    } 
  } 

  function handleClearError() { 
    setErrorMessage(null); 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmit}> 
      <div className={classes.control}> 
        <label htmlFor="email">Email</label> 
        <input id="email" type="email" value={enteredEmail} onChange={handleUpdateEmail} /> 
      </div> 
      <Error message={errorMessage} onClearError={handleClearError} /> 
      <button>Sign Up</button> 
    </form> 
  ); 
}
```

En este componente, el componente `Error` recibe un puntero a la función `handleClearError` (como un valor para la prop `onClearError`). Podrías recordar un ejemplo muy similar de antes en este capítulo (de la sección *Evitar evaluaciones innecesarias de componentes secundarios*). Allí, se utilizó `memo()` para garantizar que la función del componente `Error` no se invocara cuando cambiara `enteredEmail` (porque su valor no se utilizaba en absoluto en la función del componente `Error`).

Ahora, con el ejemplo ajustado y el puntero a la función `handleClearError` pasado a `Error`, desafortunadamente `memo()` ya no evita las ejecuciones de la función del componente. ¿Por qué? Porque las funciones son objetos en JavaScript y la función `handleClearError` se recrea cada vez que se ejecuta la función del componente `Form` (lo que ocurre en cada cambio de estado, incluidos los cambios en el estado `enteredEmail`).

Dado que se crea un nuevo objeto de función para cada cambio de estado, `handleClearError` es técnicamente un valor diferente para cada ejecución del componente `Form`. Por lo tanto, el componente `Error` recibe un nuevo valor de prop `onClearError` cada vez que se invoca la función del componente `Form`. Para `memo()`, los objetos de función `handleClearError` antiguo y nuevo son diferentes entre sí y, por lo tanto, no impedirá que la función del componente `Error` se ejecute nuevamente.

Ahí es exactamente donde `useCallback()` puede ayudar:

```javascript
const handleClearError = useCallback(() => { 
  setErrorMessage(null); 
}, []);
```

Al envolver `handleClearError` con `useCallback()`, se evita la recreación de la función y, por lo tanto, no se pasa ningún objeto de función nuevo al componente `Error`. Por lo tanto, `memo()` puede detectar la igualdad entre el valor de prop `onClearError` antiguo y nuevo y evita nuevamente ejecuciones innecesarias de funciones de componentes.

De manera similar, `useCallback()` se puede utilizar junto con `useMemo()`. Si la operación con uso intensivo de cómputo envuelta con `useMemo()` utiliza una función como dependencia, puedes usar `useCallback()` para asegurarte de que esta función dependiente no se vuelva a crear innecesariamente.

#### Uso del compilador de React (*React Compiler*)
Considerar y usar `memo()`, `useMemo()` y `useCallback()` para evitar reevaluaciones innecesarias de componentes puede ser una tarea tediosa. Aunque la optimización del rendimiento es importante, como desarrollador de React, normalmente deseas concentrarte en crear interfaces de usuario excelentes e implementar funciones útiles en ellas.

Es por eso que el equipo de React desarrolló un compilador que tiene como objetivo optimizar el código por ti: una herramienta independiente que se puede agregar a los proyectos de React que **envolverá automáticamente tus componentes con `memo()`, usará `useMemo()` cuando sea necesario y envolverá funciones con `useCallback()`**.

Por lo tanto, al usar este compilador, ya no tienes que pensar ni usar estas funciones y Hooks de optimización manualmente.

En otras palabras, el compilador de React optimizará tu código por ti. Al menos, esa es la teoría.

Sin embargo, al momento de escribir este texto, este compilador solo está disponible en modo experimental. Esto significa que no debes usarlo para producción y que puede haber errores o resultados de compilación subóptimos.

No obstante, puedes probarlo cuando trabajes en un proyecto que use React 19 o superior (el compilador no funcionará con versiones anteriores de React).

Agregar el compilador a un proyecto es fácil ya que es solo una dependencia adicional que debe instalarse en tu proyecto:

```bash
npm install babel-plugin-react-compiler
```

> [!NOTE]
> Dado que el compilador aún no es estable, es posible que los pasos de instalación y las instrucciones de uso cambien con el tiempo.
> Por lo tanto, debes visitar la página oficial de documentación del compilador de React para obtener los detalles e instrucciones más recientes: [https://react.dev/learn/react-compiler](https://react.dev/learn/react-compiler).

Con el plugin del compilador instalado, debes ajustar la configuración de tu proceso de compilación para que se utilice el compilador. Cuando trabajes en un proyecto basado en Vite, solo tienes que editar el archivo `vite.config.js`, que debería existir en la carpeta raíz de tu proyecto:

```javascript
// vite.config.js 
const ReactCompilerConfig = { /* ... */ }; 

export default defineConfig(() => { 
  return { 
    plugins: [ 
      react({ 
        babel: { 
          plugins: [ 
            ["babel-plugin-react-compiler", ReactCompilerConfig], 
          ], 
        }, 
      }), 
    ], 
    // ... 
  }; 
});
```

Si estás utilizando otra configuración de proyecto, puedes seguir las instrucciones de instalación en la página de documentación oficial del compilador.

Con el compilador instalado, se ejecutará automáticamente para analizar y ajustar tu código para incluir optimizaciones como `memo()` o `useMemo()`. Ten en cuenta que esas optimizaciones se realizan como parte del proceso de compilación que se invoca ejecutando `npm run dev` o `npm run build`. Por lo tanto, tu código fuente original no cambiará; en su lugar, el compilador optimiza tu código "detrás de escena".

Una vez que el compilador de React sea estable, es muy probable que sea una herramienta estándar que forme parte del proceso de compilación de cada proyecto de React. Por lo tanto, ya no tendrás que usar `memo()`, `useMemo()` o `useCallback()` manualmente en tu código. Pero hasta que ese sea el caso, o en proyectos de React que no puedan usar el compilador, aún tendrás que optimizar el código manualmente.

---

### Sección 4: Evitar la descarga innecesaria de código

Hasta ahora, este capítulo ha analizado principalmente estrategias para evitar la ejecución innecesaria de código. Pero no es solo la ejecución del código lo que puede ser un problema. Tampoco es ideal si los visitantes de tu sitio web tienen que descargar una gran cantidad de código que tal vez nunca se ejecute. Porque cada kilobyte de código JavaScript que deba descargarse ralentizará el tiempo de carga inicial de tu página web, no solo por el tiempo que lleva descargar el paquete de código (que puede ser significativo si los usuarios están en una red lenta y los paquetes de código son grandes), sino también porque el navegador tiene que analizar todo el código descargado antes de que tu página sea interactiva.

Por esta razón, se dedica un gran esfuerzo de la comunidad y del ecosistema a reducir el tamaño de los paquetes de código JavaScript. La minificación (acortamiento automático de nombres de variables y otras medidas para reducir el código final) y la compresión pueden ayudar mucho y, por lo tanto, son una técnica común. De hecho, los proyectos creados con Vite ya vienen con un flujo de trabajo de compilación (iniciado al ejecutar `npm run build`) que producirá un paquete de código optimizado para producción que sea lo más pequeño posible.

Pero también hay pasos que tú, el desarrollador, puedes tomar para reducir el tamaño general del paquete de código:
- Intenta escribir código corto y conciso.
- Sé cuidadoso al incluir bibliotecas de terceros y no las utilices a menos que realmente las necesites.
- Considera el uso de técnicas de **división de código** (*code-splitting*).

El primer punto debería ser bastante obvio. Si escribes menos código, los visitantes de tu sitio web tendrán menos código para descargar. Por lo tanto, intentar ser conciso y escribir código optimizado tiene sentido.

El segundo punto también debería tener sentido. Para algunas tareas, en realidad ahorrarás código al incluir bibliotecas de terceros que pueden ser mucho más elaboradas que la solución de código que podrías idear. Pero también hay situaciones y tareas en las que podrías salir adelante escribiendo tu propio código o usando alguna función integrada en lugar de incluir una biblioteca de terceros. Al menos, siempre deberías pensar en esta alternativa e incluir únicamente las bibliotecas de terceros que absolutamente necesites.

El último punto es algo en lo que React puede ayudar.

#### Reducción del tamaño del bundle mediante división de código / Carga perezosa (*Lazy Loading*)
React expone una función **`lazy()`** que se puede utilizar para cargar el código de los componentes condicionalmente, lo que significa solo cuando realmente se necesita (en lugar de por adelantado).

Considera el siguiente ejemplo, que consta de dos componentes que trabajan juntos.

Un componente `DateCalculator` se define así:

```javascript
import { useState } from 'react'; 
import { add, differenceInDays, format, parseISO } from 'date-fns'; 
import classes from './DateCalculator.module.css'; 

const initialStartDate = new Date(); 
const initialEndDate = add(initialStartDate, { days: 1 }); 

function DateCalculator() { 
  const [startDate, setStartDate] = useState( 
    format(initialStartDate, 'yyyy-MM-dd') 
  ); 
  const [endDate, setEndDate] = useState( 
    format(initialEndDate, 'yyyy-MM-dd') 
  ); 
  const daysDiff = differenceInDays( 
    parseISO(endDate), 
    parseISO(startDate) 
  ); 

  function handleUpdateStartDate(event) { 
    setStartDate(event.target.value); 
  } 

  function handleUpdateEndDate(event) { 
    setEndDate(event.target.value); 
  } 

  return ( 
    <div className={classes.calculator}> 
      <p>Calculate the difference (in days) between two dates.</p> 
      <div className={classes.control}> 
        <label htmlFor="start">Start Date</label> 
        <input id="start" type="date" value={startDate} onChange={handleUpdateStartDate} /> 
      </div> 
      <div className={classes.control}> 
        <label htmlFor="end">End Date</label> 
        <input id="end" type="date" value={endDate} onChange={handleUpdateEndDate} /> 
      </div> 
      <p className={classes.difference}> 
        Difference: {daysDiff} days 
      </p> 
    </div> 
  ); 
} 

export default DateCalculator;
```

Este componente `DateCalculator` luego es renderizado condicionalmente por el componente `App`:

```javascript
import { useState } from 'react'; 
import DateCalculator from './components/DateCalculator.jsx'; 

function App() { 
  const [showDateCalc, setShowDateCalc] = useState(false); 

  function handleOpenDateCalc() { 
    setShowDateCalc(true); 
  } 

  return ( 
    <> 
      <p>This app might be doing all kinds of things.</p> 
      <p> 
        But you can also open a calculator which calculates the difference between two dates. 
      </p> 
      <button onClick={handleOpenDateCalc}>Open Calculator</button> 
      {showDateCalc && <DateCalculator />} 
    </> 
  ); 
} 

export default App;
```

En este ejemplo, el componente `DateCalculator` utiliza una biblioteca de terceros (la biblioteca `date-fns`) para acceder a varias funciones de utilidad relacionadas con fechas (por ejemplo, una función para calcular la diferencia entre dos fechas, o `differenceInDays`).

Luego, el componente acepta dos valores de fecha y calcula la diferencia entre esas fechas en días, aunque la lógica real del componente no es muy importante aquí. Lo que es importante es el hecho de que se utiliza una biblioteca de terceros y varias funciones de utilidad. Esto agrega una gran cantidad de código JavaScript al paquete de código general, y todo ese código debe descargarse cuando se carga el sitio web completo por primera vez, a pesar de que la calculadora de fechas ni siquiera es visible en ese momento (porque se renderiza condicionalmente).

Después de compilar la aplicación para producción (mediante `npm run build`), al obtener una vista previa de esa versión de producción (mediante `npm run preview`), puedes ver que se descarga un archivo de paquete de código principal en la siguiente captura de pantalla:

**Figura 10.10**: Se descarga un archivo de paquete principal.

La pestaña Red (*Network*) en las herramientas de desarrollo del navegador revela las solicitudes de red salientes. Como puedes ver en la captura de pantalla, se descarga un archivo de paquete JavaScript principal. No verás ninguna solicitud adicional enviada al hacer clic en el botón. Esto implica que todo el código, incluido el código necesario para `DateCalculator`, se descargó por adelantado.

Ahí es donde la división de código con la función `lazy()` de React resulta útil.

Esta función se puede envolver alrededor de una **importación dinámica** (*dynamic import*) para cargar el componente importado solo una vez que sea necesario.

> [!NOTE]
> Las importaciones dinámicas son una característica nativa de JavaScript que permite importar archivos de código JavaScript de forma dinámica. Para obtener más información sobre este tema, visita [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import).

En el ejemplo anterior, se usaría de la siguiente manera en el archivo del componente `App`:

```javascript
import { lazy, useState } from 'react'; 

const DateCalculator = lazy(() => 
  import( './components/DateCalculator.jsx' ) 
); 

function App() { 
  const [showDateCalc, setShowDateCalc] = useState(false); 

  function handleOpenDateCalc() { 
    setShowDateCalc(true); 
  } 

  return ( 
    <> 
      <p>This app might be doing all kinds of things.</p> 
      <p> 
        But you can also open a calculator which calculates the difference between two dates. 
      </p> 
      <button onClick={handleOpenDateCalc}>Open Calculator</button> 
      {showDateCalc && <DateCalculator />} 
    </> 
  ); 
} 

export default App;
```

Sin embargo, esto por sí solo no funcionará. También debes envolver el código JSX condicional, donde se usa el componente importado dinámicamente, con otro componente proporcionado por React: el componente **`<Suspense>`**, de esta manera:

```javascript
import { lazy, Suspense, useState } from 'react'; 

const DateCalculator = lazy(() => 
  import( './components/DateCalculator.jsx' ) 
); 

function App() { 
  const [showDateCalc, setShowDateCalc] = useState(false); 

  function handleOpenDateCalc() { 
    setShowDateCalc(true); 
  } 

  return ( 
    <> 
      <p>This app might be doing all kinds of things.</p> 
      <p> 
        But you can also open a calculator which calculates the difference between two dates. 
      </p> 
      <button onClick={handleOpenDateCalc}>Open Calculator</button> 
      <Suspense fallback={<p>Loading...</p>}> 
        {showDateCalc && <DateCalculator />} 
      </Suspense> 
    </> 
  ); 
} 

export default App;
```

> [!NOTE]
> Puedes encontrar el código de ejemplo terminado en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/examples/06-code-splitting](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/examples/06-code-splitting).

`Suspense` es un componente integrado en React que tiene como objetivo **mostrar contenido de respaldo (*fallback*) mientras se carga algún recurso o dato**. Por lo tanto, al usarlo para la carga perezosa (*lazy loading*), debes envolverlo alrededor de cualquier código condicional que use la función `lazy()` de React. `Suspense` también tiene una prop obligatoria que debe proporcionarse, la prop `fallback`, que espera un valor JSX que se renderizará como contenido de respaldo hasta que el contenido cargado dinámicamente esté disponible.

`lazy()` hace que el código JavaScript general se divida en múltiples paquetes. El paquete que contiene el componente `DateCalculator` (y sus dependencias, como el código de la biblioteca `date-fns`) solo se descarga cuando es necesario, es decir, cuando se hace clic en el botón en el componente `App`. Si esa descarga tardara un poco más, el contenido de respaldo de `Suspense` se mostraría en la pantalla mientras tanto.

> [!NOTE]
> El componente `Suspense` de React no se limita a usarse junto con la función `lazy()`. El Capítulo 14, *Gestión de Datos con React Router*, y el Capítulo 17, *Entendiendo React Suspense y el Hook use()*, explorarán cómo se puede usar el componente `Suspense` para mostrar contenido alternativo mientras se obtienen datos.

Después de agregar `lazy()` y el componente `Suspense` como se describe, inicialmente se descarga un paquete más pequeño. Además, si se hace clic en el botón, se descargan archivos de código adicionales:

**Figura 10.11**: Después de hacer clic en el botón, se descarga un archivo de código adicional.

Al igual que con todas las demás técnicas de optimización descritas hasta ahora, la función `lazy()` no es una función con la que debas comenzar a envolver todas tus importaciones. Si un componente importado es muy pequeño y simple (y no utiliza ningún código de terceros), dividir el código realmente no vale la pena, especialmente porque debes considerar que la solicitud HTTP adicional requerida para descargar el paquete adicional también conlleva cierta sobrecarga.

Tampoco tiene sentido usar `lazy()` en componentes que se cargarán inicialmente de todos modos. Solo considera usarlo en componentes cargados condicionalmente.

---

### Sección 5: Modo Estricto (*Strict Mode*)

A lo largo de este capítulo, has aprendido mucho sobre el funcionamiento interno de React y varias técnicas de optimización. No es realmente una técnica de optimización, pero aún está relacionada, otra característica que ofrece React: el **Modo Estricto (*Strict Mode*)**.

Es posible que te hayas topado con un código como este antes:

```javascript
import React from 'react'; 
// ... other code ... 
root.render(<React.StrictMode><App /></React.StrictMode>);
```

`<React.StrictMode>` es otro componente integrado proporcionado por React. No renderiza un elemento visual, pero **habilitará algunas comprobaciones adicionales que React realiza detrás de escena**.

La mayoría de las comprobaciones están relacionadas con la identificación del uso de código no seguro o heredado (es decir, características que se eliminarán en el futuro). Pero también hay algunas comprobaciones que tienen como objetivo ayudarte a identificar problemas potenciales con tu código.

Por ejemplo, cuando se utiliza el Modo Estricto, **React ejecutará las funciones de los componentes dos veces y también desmontará y volverá a montar cada componente cada vez que se monte por primera vez**. Esto se hace para garantizar que estés administrando tu estado y tus efectos secundarios de una manera consistente y correcta (por ejemplo, que sí tengas funciones de limpieza en tus funciones de efectos).

> [!NOTE]
> El Modo Estricto solo afecta a tu aplicación y a su comportamiento durante el desarrollo. No influye en tu aplicación una vez que la compilas para producción. Las comprobaciones adicionales de los efectos, como la doble ejecución de funciones de componentes, no se realizarán en producción.

Crear aplicaciones de React con el Modo Estricto habilitado a veces puede generar confusión o mensajes de error molestos. Por ejemplo, podrías preguntarte por qué los efectos de tus componentes se ejecutan con demasiada frecuencia.

Por lo tanto, es tu decisión personal si deseas utilizar el Modo Estricto o no. Sin embargo, habilitarlo puede ayudarte a detectar y corregir errores a tiempo.

---

### Sección 6: Depuración de código y las React Developer Tools

Anteriormente en este capítulo, aprendiste que las funciones de los componentes pueden ejecutarse con bastante frecuencia y que puedes evitar ejecuciones innecesarias usando `memo()` y `useMemo()` (y que no siempre debes evitarlas).

Identificar las ejecuciones de componentes agregando `console.log()` dentro de las funciones de los componentes es una forma de obtener información sobre un componente. Es el enfoque utilizado a lo largo de este capítulo. Sin embargo, para aplicaciones de React grandes con docenas, cientos o incluso miles de componentes, usar `console.log()` puede volverse tedioso.

Por eso, el equipo de React también creó una herramienta oficial para ayudar a obtener información sobre la aplicación: **React Developer Tools**. Es una extensión que se puede instalar en los principales navegadores (Chrome, Firefox y Edge). Puedes buscar e instalar la extensión simplemente buscando en la web `"<tu navegador> react developer tools"` (por ejemplo, `chrome react developer tools`).

Una vez que hayas instalado la extensión, podrás acceder a ella directamente desde dentro del navegador. Por ejemplo, al usar Chrome, puedes acceder a la extensión React Developer Tools directamente desde las herramientas de desarrollo de Chrome (que se pueden abrir a través del menú en Chrome). Explora la documentación específica de la extensión (en la tienda de extensiones de tu navegador) para obtener detalles sobre cómo acceder a ella.

La extensión React Developer Tools ofrece dos áreas: una página de **Componentes** (*Components*) y una página de **Perfilador** (*Profiler*):

**Figura 10.12**: Se puede acceder a React Developer Tools a través de las herramientas de desarrollo del navegador.

La página **Componentes** se puede utilizar para analizar la estructura de componentes de la página renderizada actualmente. Puedes usar esta página para comprender la estructura de tus componentes (es decir, el "árbol de componentes"), cómo los componentes están anidados entre sí e incluso la configuración (props, estado) de los componentes.

**Figura 10.13**: Se muestran las relaciones y los datos de los componentes.

Esta página puede ser muy útil cuando intentas comprender el estado actual de un componente, cómo se relaciona un componente con otros componentes y qué otros componentes pueden, por lo tanto, influir en un componente (por ejemplo, hacer que se reevalúe).

Sin embargo, en el contexto de este capítulo, la página más útil es la página **Perfilador (*Profiler*)**:

**Figura 10.14**: La página Profiler (sin datos recopilados).

En esta página, puedes comenzar a registrar las evaluaciones de los componentes (es decir, las ejecuciones de las funciones de los componentes). Puedes hacerlo simplemente haciendo clic en el botón Grabar en la esquina superior izquierda (el círculo azul). Este botón luego será reemplazado por un botón Detener, en el que puedes hacer clic para finalizar la grabación.

Después de grabar la aplicación de React durante un par de segundos (e interactuar con ella durante ese período), un resultado de ejemplo podría verse así:

**Figura 10.15**: La página Profiler muestra varias barras después de que finaliza la grabación.

Este resultado consta de dos áreas principales:
1. **Una lista de barras**, que indica el número de reevaluaciones de componentes (cada barra refleja un ciclo de reevaluación que afectó a uno o más componentes). Puedes hacer clic en estas barras para explorar más detalles sobre un ciclo específico.
2. Para el ciclo de evaluación seleccionado, se presenta **una lista de los componentes afectados**. Puedes identificar fácilmente los componentes afectados ya que sus barras están coloreadas y se muestra información de tiempo para ellos.

Puedes seleccionar cualquier ciclo de renderizado desde (1) (en este caso, hay dos para esta sesión de grabación) para ver qué componentes se vieron afectados. La parte inferior de la ventana (2) muestra todos los componentes afectados resaltándolos con algún color y mostrando la cantidad total de tiempo que tardaron los componentes en reevaluarse (por ejemplo, 0,1 ms de 0,3 ms).

> [!NOTE]
> Vale la pena señalar que esta herramienta también demuestra que la evaluación de componentes es extremadamente rápida: 0,1 ms para reevaluar un componente es demasiado rápido para que cualquier humano se dé cuenta de que algo sucedió detrás de escena.

En el lado derecho de la ventana, también obtienes más información sobre este ciclo de evaluación de componentes. Por ejemplo, descubres dónde se originó. En este caso, fue provocado por el componente `Form` (es el mismo ejemplo discutido anteriormente en este capítulo, en la sección *Evitar evaluaciones innecesarias de componentes secundarios*).

Por lo tanto, la página Profiler también puede ayudarte a identificar los ciclos de evaluación de los componentes y determinar qué componentes se ven afectados. En este ejemplo, puedes ver una diferencia si la función `memo()` envuelve el componente `Error`:

**Figura 10.16**: Solo el componente `Form` se ve afectado, no el componente `Error`.

Después de volver a agregar la función `memo()` como envoltorio alrededor del componente `Error` (como se explicó anteriormente en este capítulo), puedes usar la página Profiler de React Developer Tools para confirmar que el componente `Error` ya no se evalúa innecesariamente. Para hacer esto, debes iniciar una nueva sesión de grabación y reproducir la situación en la que anteriormente, sin `memo()`, se habría llamado nuevamente al componente `Error`.

Las líneas diagonales atenuadas a través del componente `Error` en la ventana del generador de perfiles indican que este componente no se vio afectado por la invocación de alguna otra función de componente.

Por lo tanto, React Developer Tools se puede utilizar para obtener conocimientos más profundos sobre tu aplicación de React y tus componentes. Puedes utilizarlas además o en lugar de llamar a `console.log()` en una función de componente.

---

### Sección 7: Resumen y puntos clave

- Los componentes de React se reevalúan (ejecutan) cada vez que cambia su estado o se evalúa el componente principal.
- React optimiza la evaluación de componentes calculando primero los cambios requeridos en la interfaz de usuario con la ayuda de un **DOM Virtual**.
- React agrupa por lotes (*batches*) múltiples actualizaciones de estado que ocurren al mismo tiempo y en el mismo lugar. Esto garantiza que se eviten evaluaciones innecesarias de componentes.
- La función `memo()` se puede utilizar para controlar las ejecuciones de las funciones de los componentes.
- `memo()` busca diferencias en los valores de las props (props antiguas frente a props nuevas) para determinar si una función de componente debe ejecutarse nuevamente.
- `useMemo()` se puede utilizar para envolver cálculos de rendimiento intensivo y realizarlos solo si cambiaron dependencias clave.
- Tanto `memo()` como `useMemo()` deben usarse con cuidado ya que también tienen un costo (las comparaciones realizadas).
- Cuando trabajas con React 19 o superior, puedes instalar y habilitar el compilador de React (experimental) para optimizar automáticamente tu código durante el proceso de compilación.
- El tamaño de descarga inicial del código se puede reducir con la ayuda de la división de código mediante la función `lazy()` (junto con el componente integrado `Suspense`).
- El Modo Estricto de React se puede habilitar (a través del componente integrado `<React.StrictMode>`) para realizar varias comprobaciones adicionales y detectar posibles errores en tu aplicación.
- React Developer Tools se puede utilizar para obtener información más profunda sobre tu aplicación de React (por ejemplo, la estructura de componentes y los ciclos de reevaluación).

---

### Sección 8: ¿Qué sigue?

Como desarrollador, siempre debes conocer y comprender la herramienta con la que estás trabajando; en este caso, React.

Este capítulo te permitió tener una mejor idea de cómo funciona React bajo el capó y qué optimizaciones se implementan automáticamente. Además, también aprendiste sobre varias técnicas de optimización que puedes implementar tú mismo.

El próximo capítulo volverá a resolver problemas reales que podrías enfrentar al intentar crear aplicaciones de React. En lugar de optimizar las aplicaciones de React, aprenderás más sobre técnicas y funciones que se pueden utilizar para resolver problemas más complejos relacionados con la gestión del estado de los componentes y de la aplicación.

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/10-behind-scenes/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/10-behind-scenes/exercises/questions-answers.md):

1. ¿Por qué React utiliza un DOM Virtual para detectar las actualizaciones requeridas del DOM?
2. ¿Cómo se ve afectado el DOM real cuando se ejecuta una función de componente?
3. ¿Qué componentes son excelentes candidatos para la función `memo()`? ¿Cuáles componentes son malos candidatos?
4. ¿En qué se diferencia `useMemo()` de `memo()`?
5. ¿Cuál es la idea detrás de la división de código y la función `lazy()`?

---

### Sección 10: Aplica lo aprendido

Con tu conocimiento recién adquirido sobre el funcionamiento interno de React y algunas de las técnicas de optimización que puedes emplear para mejorar tus aplicaciones, ahora puedes aplicar este conocimiento en la siguiente actividad.

#### Actividad 10.1: Optimizar una aplicación existente
En esta actividad, se te entrega una aplicación de React existente que podría optimizarse en varios lugares. Tu tarea es identificar oportunidades de optimización e implementar soluciones adecuadas. Ten en cuenta que demasiada optimización puede conducir a un resultado peor.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/activities/practice-1-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start` en este caso) para usar la versión correcta del código.

El proyecto proporcionado también utiliza muchas de las funciones cubiertas en capítulos anteriores. Tómate el tiempo para analizarlo y comprender el código proporcionado. Esta es una gran práctica y te permite ver muchos conceptos clave en acción.

Una vez que hayas descargado el código y ejecutado `npm install` en la carpeta del proyecto (para instalar todas las dependencias requeridas), puedes iniciar el servidor de desarrollo mediante `npm run dev`. Como resultado, al visitar `localhost:5173`, deberías ver la siguiente interfaz de usuario:

**Figura 10.17**: El proyecto inicial en ejecución.

Tómate tu tiempo para familiarizarte con el proyecto provisto. Experimenta con los diferentes botones de la interfaz de usuario, completa algunos datos ficticios en los campos de entrada del formulario y analiza el código proporcionado. Ten en cuenta que este proyecto ficticio no envía ninguna solicitud HTTP a ningún servidor. Todos los datos ingresados se descartan en el momento en que se ingresan.

Para completar la actividad, los pasos de la solución son los siguientes:
1. Encuentra oportunidades de optimización buscando ejecuciones innecesarias de funciones de componentes.
2. Además, identifica la ejecución innecesaria de código dentro de las funciones de los componentes (donde no se puede evitar la invocación general de la función del componente).
3. Determina qué código se podría cargar de forma perezosa (*lazy*) en lugar de cargarse por adelantado (*eager*).
4. Utiliza la función `memo()`, el Hook `useMemo()` y la función `lazy()` de React para mejorar el código.

Sabrás que diste con una buena solución y ajustes sensatos si puedes ver solicitudes de red adicionales para obtener código (en la pestaña Red / *Network* de las herramientas de desarrollo de tu navegador) al hacer clic en los botones `Reset password` o `Create a new account`:

**Figura 10.18**: En la solución final, parte del código se carga de forma perezosa.

Además, no deberías ver ningún mensaje de consola `Validated password.` al escribir en los campos de entrada de correo electrónico (`Email` y `Confirm Email`) del formulario de registro (es decir, el formulario al que cambias al hacer clic en `Create a new account`):

**Figura 10.19**: No aparece ningún resultado "Validated password." en la consola.

Tampoco deberías recibir ningún resultado en la consola al hacer clic en el botón `More Information`:

**Figura 10.20**: No hay mensajes en la consola al hacer clic en "More Information".

> [!NOTE]
> Todos los archivos de código utilizados para esta actividad, y la solución, se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/10-behind-scenes/activities/practice-1).
