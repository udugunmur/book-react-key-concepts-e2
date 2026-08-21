# Parte 1: Fundamentos de React

## Capítulo 8: Manejo de Efectos Secundarios (Side Effects)

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Identificar efectos secundarios (*side effects*) en tus aplicaciones de React.
- Comprender y utilizar el Hook `useEffect()`.
- Utilizar las diferentes características y conceptos relacionados con el Hook `useEffect()` para evitar errores y optimizar tu código.
- Manejar efectos secundarios relacionados y no relacionados con cambios de estado.

---

### Sección 1: Introducción

Si bien todos los ejemplos de React cubiertos anteriormente en este libro han sido relativamente sencillos y se introdujeron muchos conceptos clave de React, es poco probable que se puedan crear muchas aplicaciones reales solo con esos conceptos.

La mayoría de las aplicaciones reales que crearás como desarrollador de React también necesitan enviar solicitudes HTTP, acceder al almacenamiento del navegador, registrar datos de análisis (*analytics*) o realizar cualquier otro tipo de tarea similar. Y solo con componentes, props, eventos y estado, a menudo encontrarás problemas al intentar agregar tales funcionalidades a tu aplicación. Se analizarán explicaciones detalladas y ejemplos más adelante en este capítulo, pero el problema central es que **tareas como esta a menudo interferirán con el ciclo de renderizado de componentes de React**, lo que provocará errores inesperados o incluso romperá la aplicación.

Este capítulo examinará más de cerca ese tipo de acciones, analizará lo que tienen en común y, lo más importante, te enseñará cómo manejar correctamente dichas tareas en las aplicaciones de React.

---

### Sección 2: ¿Cuál es el problema?

Antes de explorar una solución, es importante comprender primero el problema concreto.

Las acciones que no están directamente relacionadas con la producción de un (nuevo) estado de la interfaz de usuario a menudo chocan con el ciclo de renderizado de componentes de React. Pueden introducir errores o incluso romper toda la aplicación web.

Considera el siguiente fragmento de código de ejemplo (**importante: no ejecutes este código ya que causará un bucle infinito y enviará una gran cantidad de solicitudes HTTP en segundo plano**):

```javascript
import { useState } from 'react'; 
import classes from './BlogPosts.module.css'; 

async function fetchPosts() { 
  const response = await fetch('https://jsonplaceholder.typicode.com/posts'); 
  const blogPosts = await response.json(); 
  return blogPosts; 
} 

function BlogPosts() { 
  const [loadedPosts, setLoadedPosts] = useState([]); 

  fetchPosts().then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 

  return ( 
    <ul className={classes.posts}> 
      {loadedPosts.map((post) => ( 
        <li key={post.id}>{post.title}</li> 
      ))} 
    </ul> 
  ); 
} 

export default BlogPosts;
```

Entonces, ¿cuál es el problema con este código? ¿Por qué crea un bucle infinito?

En este ejemplo, se crea un componente de React (`BlogPosts`). Además, se define una función que no es un componente (`fetchPosts()`). Esa función utiliza la función integrada `fetch()` (proporcionada por el navegador) para enviar una solicitud HTTP a una interfaz de programación de aplicaciones (**API**) externa y obtener algunos datos.

> [!NOTE]
> La función `fetch()` la pone a disposición el navegador (todos los navegadores modernos admiten esta función). Puedes obtener más información sobre `fetch()` en [https://academind.com/tutorials/xhr-fetch-axios-the-fetch-api](https://academind.com/tutorials/xhr-fetch-axios-the-fetch-api).
> La función `fetch()` devuelve una promesa (*promise*), que, en este ejemplo, se maneja mediante `async/await`. Al igual que `fetch()`, las promesas son un concepto clave del desarrollo web, sobre el cual puedes obtener más información (junto con `async/await`) en [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).
> Una API, en este contexto, es un sitio que expone varias rutas a las que se pueden enviar solicitudes, ya sea para enviar o para obtener datos. `jsonplaceholder.typicode.com` es una API ficticia que responde con datos de prueba. Se puede utilizar en escenarios como el ejemplo anterior, donde solo necesitas una API a la que enviar solicitudes para probar algún concepto o código sin conectar ni crear una API de backend real. En este caso, se utiliza para explorar algunos problemas y conceptos de React. Se esperan conocimientos básicos sobre el envío de solicitudes HTTP con `fetch()` y APIs para este capítulo y el libro en general. Si es necesario, puedes utilizar sitios como MDN ([https://developer.mozilla.org/](https://developer.mozilla.org/)) para reforzar tus conocimientos sobre estos conceptos básicos.

En el fragmento de código anterior, el componente `BlogPosts` utiliza `useState()` para registrar un valor de estado `loadedPosts`. El estado se utiliza para mostrar una lista de publicaciones de blog. Sin embargo, esas publicaciones de blog no están definidas en la propia aplicación: se obtienen de la API externa mencionada anteriormente.

`fetchPosts()`, que es la función de utilidad que contiene el código para obtener datos de publicaciones de blog desde esa API de backend mediante la función `fetch()`, se llama directamente en el cuerpo de la función del componente. Dado que `fetchPosts()` es una función asíncrona (usando `async/await`), devuelve una promesa. En `BlogPosts`, el código que debe ejecutarse una vez que se resuelve la promesa se registra mediante el método integrado `then()`.

> [!NOTE]
> `async/await` no se utiliza directamente en el cuerpo de la función del componente porque los componentes estándar de React no deben ser funciones asíncronas. Dichas funciones devuelven automáticamente una promesa como valor (incluso sin una sentencia `return` explícita), lo cual es un valor de retorno no válido para un componente de React.
> Dicho esto, existen componentes de React a los que se les permite usar `async/await` y devolver una promesa: los llamados **React Server Components**, que no están restringidos a devolver código JSX, cadenas, etc. Esta característica se analizará en detalle en el Capítulo 16, *React Server Components y Server Actions*.

Una vez que se resuelve la promesa de `fetchPosts()`, los datos de publicaciones extraídos (`fetchedPosts`) se establecen como el nuevo estado `loadedPosts` (a través de `setLoadedPosts(fetchedPosts)`).

Si ejecutaras el código anterior (¡lo cual no deberías hacer!), al principio parecería funcionar. Pero detrás de escena, en realidad iniciaría un **bucle infinito**, bombardeando la API con solicitudes HTTP. Esto se debe a que, como resultado de obtener una respuesta de la solicitud HTTP, se utiliza `setLoadedPosts()` para establecer un nuevo estado.

Anteriormente en este libro (en el Capítulo 4, *Trabajando con Eventos y Estado*), aprendiste que **cada vez que cambia el estado de un componente, React reevalúa el componente al que pertenece el estado**. "Reevaluar" significa simplemente que la función del componente se ejecuta nuevamente (por React, automáticamente).

Dado que este componente `BlogPosts` llama a `fetchPosts()` (que envía una solicitud HTTP) directamente dentro del cuerpo de la función del componente, esta solicitud HTTP se enviará cada vez que se ejecute la función del componente. Y como el estado (`loadedPosts`) se actualiza como resultado de obtener una respuesta de esa solicitud HTTP, este proceso comienza nuevamente y se crea un bucle infinito.

El problema de raíz, en este caso, es que **enviar una solicitud HTTP es un efecto secundario** (*side effect*), un concepto que se explorará con mayor detalle en la siguiente sección.

---

### Sección 3: Entendiendo los efectos secundarios

Los **efectos secundarios** (*side effects*) son acciones o procesos que ocurren además de otro proceso principal. Al menos, esta es una definición concisa que ayuda a comprender los efectos secundarios en el contexto de una aplicación de React.

> [!NOTE]
> Si deseas profundizar en el concepto de efectos secundarios, también puedes explorar la siguiente discusión sobre efectos secundarios en Stack Overflow: [https://softwareengineering.stackexchange.com/questions/40297/what-is-a-side-effect](https://softwareengineering.stackexchange.com/questions/40297/what-is-a-side-effect).

En el caso de un componente de React, el **proceso principal** sería el ciclo de renderizado del componente en el que la tarea principal de un componente es renderizar la interfaz de usuario que se define en la función del componente (el código JSX devuelto). El componente de React debe devolver el código JSX final, que luego se traduce en instrucciones de manipulación del DOM.

Para esto, React considera los cambios de estado como el desencadenante para actualizar la interfaz de usuario. Registrar controladores de eventos como `onClick`, agregar refs o renderizar componentes secundarios (posiblemente usando props) serían otros elementos que pertenecen a este proceso principal, porque todos estos conceptos están directamente relacionados con la tarea principal de renderizar la interfaz de usuario deseada.

Sin embargo, enviar una solicitud HTTP, como en el ejemplo anterior, no forma parte de este proceso principal. No influye directamente en la interfaz de usuario. Si bien los datos de respuesta eventualmente podrían mostrarse en la pantalla, definitivamente no se utilizarán en el mismo ciclo de renderizado de componentes en el que se envía la solicitud (porque las solicitudes HTTP son tareas asíncronas).

Dado que el envío de la solicitud HTTP no forma parte del proceso principal (renderizar la interfaz de usuario) realizado por la función del componente, se considera un efecto secundario. Es invocado por la misma función (la función del componente `BlogPosts`), que principalmente tiene un objetivo diferente.

Si la solicitud HTTP se enviara al hacer clic en un botón en lugar de como parte del cuerpo principal de la función del componente, **no sería un efecto secundario**. Considera este ejemplo:

```javascript
import { useState } from 'react'; 
import classes from './BlogPosts.module.css'; 

async function fetchPosts() { 
  const response = await fetch('https://jsonplaceholder.typicode.com/posts'); 
  const blogPosts = await response.json(); 
  return blogPosts; 
} 

function BlogPosts() { 
  const [loadedPosts, setLoadedPosts] = useState([]); 

  function handleFetchPosts() { 
    fetchPosts().then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
  } 

  return ( 
    <> 
      <button onClick={handleFetchPosts}>Fetch Posts</button> 
      <ul className={classes.posts}> 
        {loadedPosts.map((post) => ( 
          <li key={post.id}>{post.title}</li> 
        ))} 
      </ul> 
    </> 
  ); 
} 

export default BlogPosts;
```

Este código es casi idéntico al ejemplo anterior, pero tiene una diferencia importante: se agregó un `<button>` al código JSX. Y es este botón el que invoca una función `handleFetchPosts()` recién agregada, que luego envía la solicitud HTTP (y actualiza el estado).

Con este cambio realizado, la solicitud HTTP no se envía cada vez que el componente se vuelve a renderizar (es decir, se ejecuta de nuevo). En su lugar, solo se envía cada vez que se hace clic en el botón y, por lo tanto, esto no crea un bucle infinito. La solicitud HTTP, en este caso, tampoco constituye un efecto secundario, porque el objetivo principal de `handleFetchPosts()` (es decir, el proceso principal) es obtener nuevas publicaciones y actualizar el estado.

#### Los efectos secundarios no se limitan a solicitudes HTTP
En el ejemplo anterior, aprendiste sobre un posible efecto secundario que podría ocurrir en una función de componente: una solicitud HTTP. También aprendiste que las solicitudes HTTP no siempre son efectos secundarios: depende de dónde se originen.

En general, **cualquier acción que se inicie tras la ejecución de una función de componente de React es un efecto secundario si esa acción no está directamente relacionada con la tarea principal de renderizar la interfaz de usuario del componente**.

Aquí tienes una lista no exhaustiva de ejemplos de efectos secundarios:
- Enviar una solicitud HTTP (como se mostró anteriormente).
- Almacenar datos o recuperar datos del almacenamiento del navegador (por ejemplo, a través del objeto integrado `localStorage`).
- Configurar temporizadores (mediante `setTimeout()`) o intervalos (mediante `setInterval()`).
- Registrar datos en la consola a través de `console.log()`.

Sin embargo, no todos los efectos secundarios provocan bucles infinitos. Tales bucles solo ocurren si el efecto secundario conduce a una actualización del estado.

Aquí hay un ejemplo de un efecto secundario que no causaría un bucle infinito:

```javascript
function ControlCenter() { 
  function handleStart() { 
    // do something ... 
  } 

  console.log('Component is rendering!'); // this is a side effect! 

  return ( 
    <div> 
      <p>Press button to start the review process</p> 
      <button onClick={handleStart}>Start</button> 
    </div> 
  ); 
}
```

En este ejemplo, `console.log(…)` es un efecto secundario porque se ejecuta como parte de cada ejecución de la función del componente y no influye en la interfaz de usuario renderizada (ni para este ciclo de renderizado específico ni indirectamente para ciclos de renderizado futuros en este caso, a diferencia del ejemplo anterior con la solicitud HTTP).

Por supuesto, usar `console.log()` de esta manera no causa ningún problema. Durante el desarrollo, es bastante normal registrar mensajes o datos con fines de depuración. Los efectos secundarios no son necesariamente un problema y, de hecho, efectos secundarios como este se pueden usar o tolerar.

Pero también a menudo necesitas lidiar con efectos secundarios como la solicitud HTTP de antes. A veces, necesitas obtener datos cuando se renderiza un componente, probablemente no para cada ciclo de renderizado, pero típicamente la primera vez que se ejecuta (es decir, cuando la interfaz de usuario generada aparece en la pantalla por primera vez).

React también ofrece una solución para este tipo de problema.

---

### Sección 4: Manejo de efectos secundarios con el Hook `useEffect()`

Para manejar efectos secundarios como la solicitud HTTP mostrada anteriormente de manera segura (es decir, sin crear un bucle infinito), React ofrece otro Hook central: el Hook **`useEffect()`**.

El primer ejemplo se puede corregir y reescribir de la siguiente manera:

```javascript
import { useState, useEffect } from 'react'; 
import classes from './BlogPosts.module.css'; 

async function fetchPosts() { 
  const response = await fetch('https://jsonplaceholder.typicode.com/posts'); 
  const blogPosts = await response.json(); 
  return blogPosts; 
} 

function BlogPosts() { 
  const [loadedPosts, setLoadedPosts] = useState([]); 

  useEffect(function () { 
    fetchPosts().then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
  }, []); 

  return ( 
    <ul className={classes.posts}> 
      {loadedPosts.map((post) => ( 
        <li key={post.id}>{post.title}</li> 
      ))} 
    </ul> 
  ); 
} 

export default BlogPosts;
```

En este ejemplo, se importa y utiliza el Hook `useEffect()` para controlar el efecto secundario (de ahí el nombre del Hook, `useEffect()`, ya que se ocupa de los efectos secundarios en los componentes de React). La sintaxis y el uso exactos se explorarán en la siguiente sección, pero si usas este Hook, puedes ejecutar el ejemplo de forma segura y obtener una salida como esta:

**Figura 8.1**: Una lista de publicaciones de blog de prueba y ningún bucle infinito de solicitudes HTTP.

En la captura de pantalla anterior, puedes ver la lista de títulos de publicaciones de blog de prueba y, lo más importante, al inspeccionar las solicitudes de red enviadas, no encuentras una lista infinita de solicitudes.

`useEffect()` es, por lo tanto, la solución para problemas como el descrito anteriormente. Te ayuda a lidiar con los efectos secundarios para que puedas evitar bucles infinitos y extraerlos del proceso principal de la función de tu componente.

¿Pero cómo funciona `useEffect()` y cómo se usa correctamente?

#### Cómo usar `useEffect()`
Como se muestra en el fragmento de código de ejemplo anterior, `useEffect()`, como todos los Hooks de React, se ejecuta como una función dentro de la función del componente (`BlogPosts`, en este caso).

Aunque, a diferencia de `useState()` o `useRef()`, `useEffect()` no devuelve un valor, sí acepta un argumento (o, en realidad, dos argumentos) como esos otros Hooks:
1. **El primer argumento es siempre una función**. En este caso, la función pasada a `useEffect()` es una función anónima, creada mediante la palabra clave `function`. Alternativamente, también podrías proporcionar una función anónima creada como una función flecha (`useEffect(() => { … })`) o apuntar a alguna función con nombre (`useEffect(doSomething)`). Lo único que importa es que el primer argumento pasado a `useEffect()` debe ser una función. No debe ser ningún otro tipo de valor.
2. En el ejemplo anterior, `useEffect()` también recibe un **segundo argumento**: un array vacío (`[]`). El segundo argumento debe ser un array, pero proporcionarlo es opcional. También podrías omitir el segundo argumento y simplemente pasar el primer argumento (la función) a `useEffect()`. Sin embargo, en la mayoría de los casos, el segundo argumento es necesario para lograr el comportamiento correcto. Ambos argumentos y su propósito se explorarán con mayor detalle a continuación.

El primer argumento es una función que será ejecutada por React. **Se ejecutará después de cada ciclo de renderizado del componente** (es decir, después de cada ejecución de la función del componente).

En el ejemplo anterior, si solo proporcionas este primer argumento y omites el segundo, aún crearás un bucle infinito. Habrá una diferencia de tiempo (invisible) porque la solicitud HTTP ahora se enviará después de cada ejecución de la función del componente (en lugar de como parte de ella), pero aún activarás un cambio de estado, lo que aún hará que la función del componente se ejecute nuevamente. Por lo tanto, la función del efecto se ejecutará nuevamente y se creará un bucle infinito. En este caso, el efecto secundario se extraerá técnicamente de la función del componente, pero el problema con el bucle infinito no se resolverá:

```javascript
useEffect(function () { 
  fetchPosts().then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
}); // this would cause an infinite loop again!
```

Extraer efectos secundarios de las funciones de componentes de React es el trabajo principal de `useEffect()`, por lo que solo el primer argumento (la función que contiene el código del efecto secundario) es obligatorio. Pero, como se mencionó anteriormente, normalmente también necesitarás el segundo argumento para controlar la frecuencia con la que se ejecutará el código del efecto, porque eso es lo que hará el segundo argumento (un array).

El segundo parámetro recibido por `useEffect()` es siempre un array (a menos que se omita). Este array especifica las **dependencias de la función del efecto**. Cualquier dependencia especificada en este array, una vez que cambie, hará que la función del efecto se ejecute nuevamente. Si no se especifica ningún array (es decir, si se omite el segundo argumento), la función del efecto se ejecutará nuevamente para cada ejecución de la función del componente:

```javascript
useEffect(function () { 
  fetchPosts().then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
}, []);
```

En el ejemplo anterior, no se omitió el segundo argumento, sino que es un array vacío. Esto le informa a React que esta función de efecto **no tiene dependencias**. Por lo tanto, la función del efecto nunca se volverá a ejecutar. En su lugar, **solo se ejecutará una vez, cuando el componente se renderice por primera vez**. Si no estableces dependencias (proporcionando un array vacío), React ejecutará la función del efecto una vez, directamente después de que la función del componente se haya ejecutado por primera vez.

Es importante tener en cuenta que especificar un array vacío es muy diferente a omitirlo:
- Si se omite, no se proporciona información de dependencia a React. Por lo tanto, React ejecuta la función del efecto después de cada reevaluación del componente.
- Si se proporciona un array vacío, indicas explícitamente que este efecto no tiene dependencias y, por lo tanto, solo debe ejecutarse una vez.

Sin embargo, esto plantea otra pregunta importante: ¿cuándo deberías agregar dependencias? ¿Y cómo se agregan o especifican exactamente las dependencias?

---

### Sección 5: Efectos y sus dependencias

Omitir el segundo argumento de `useEffect()` hace que la función del efecto (el primer argumento) se ejecute después de cada ejecución de la función del componente. Proporcionar un array vacío hace que la función del efecto se ejecute solo una vez (después de la primera invocación de la función del componente). ¿Pero es eso todo lo que puedes controlar?

No, no lo es.

El array pasado a `useEffect()` puede y **debe contener todas las variables, constantes o funciones que se utilizan dentro de la función del efecto**, siempre que esas variables, constantes o funciones se hayan definido dentro de la función del componente (o en alguna función de componente principal, pasadas a través de props).

Considera este ejemplo:

```javascript
import { useState, useEffect } from 'react'; 
import classes from './BlogPosts.module.css'; 

async function fetchPosts(url) { 
  const response = await fetch(url); 
  const blogPosts = await response.json(); 
  return blogPosts; 
} 

function BlogPosts({ url }) { 
  const [loadedPosts, setLoadedPosts] = useState([]); 

  useEffect(function () { 
    fetchPosts(url) 
      .then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
  }, [url]); 

  return ( 
    <ul className={classes.posts}> 
      {loadedPosts.map((post) => ( 
        <li key={post.id}>{post.title}</li> 
      ))} 
    </ul> 
  ); 
} 

export default BlogPosts;
```

Este ejemplo se basa en el ejemplo anterior, pero se modificó en un punto importante: `BlogPosts` ahora acepta una prop `url`.

Por lo tanto, este componente ahora puede ser utilizado y configurado por otros componentes. Por supuesto, si algún otro componente establece una URL que no devuelve una lista de publicaciones de blog, la aplicación no funcionará según lo previsto. Por lo tanto, este componente puede ser de utilidad práctica limitada, pero ilustra muy bien la importancia de las dependencias de los efectos.

Si ese otro componente cambia la URL (por ejemplo, debido a alguna entrada del usuario allí), se debe enviar una nueva solicitud, por supuesto. Por lo tanto, `BlogPosts` debería enviar otra solicitud de búsqueda cada vez que cambie el valor de la prop `url`.

Es por eso que se agregó `url` al array de dependencias de `useEffect()`. Si el array se hubiera mantenido vacío, la función del efecto solo se ejecutaría una vez (como se describe en la sección anterior). Por lo tanto, cualquier cambio en `url` no tendría ningún efecto en la función del efecto ni en la solicitud HTTP ejecutada como parte de esa función: no se enviaría ninguna nueva solicitud HTTP.

Al agregar `url` al array de dependencias, React registra este valor (en este caso, un valor de prop, pero se puede registrar cualquier valor) y **vuelve a ejecutar la función del efecto siempre que ese valor cambie** (es decir, cada vez que el componente que usa `BlogPosts` establezca un nuevo valor de prop `url`).

Los tipos más comunes de dependencias de efectos son **valores de estado, props y funciones** que podrían ejecutarse dentro de la función del efecto. Esto último se analizará en mayor profundidad más adelante en este capítulo.

Como regla general, **debes agregar todos los valores (incluidas las funciones) que se utilizan dentro de una función de efecto al array de dependencias del efecto**.

Con este nuevo conocimiento en mente, si echas otro vistazo al código de ejemplo de `useEffect()` anterior, podrías detectar algunas dependencias faltantes:

```javascript
useEffect(function () { 
  fetchPosts(url) 
    .then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
}, [url]);
```

¿Por qué `fetchPosts`, `fetchedPosts` y `setLoadedPosts` no se agregan como dependencias? Después de todo, estos son valores y funciones que se utilizan dentro de la función del efecto. La siguiente sección abordará esto en detalle.

#### Dependencias innecesarias
En el ejemplo anterior, podría parecer que `fetchPosts`, `fetchedPosts` y `setLoadedPosts` deberían agregarse como dependencias a `useEffect()`, como se muestra aquí:

```javascript
useEffect(function () { 
  fetchPosts(url) 
    .then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
}, [url, fetchPosts, fetchedPosts, setLoadedPosts]);
```

Sin embargo, para `fetchPosts` y `fetchedPosts`, esto sería incorrecto. Y para `setLoadedPosts`, sería innecesario.

`fetchedPosts` no debe agregarse porque no es una dependencia externa. Es una variable local (o argumento, para ser precisos), definida y utilizada dentro de la función del efecto. No está definida en la función del componente a la que pertenece el efecto. Si intentas agregarla como dependencia, obtendrás un error:

**Figura 8.2**: Ocurrió un error: no se pudo encontrar `fetchedPosts`.

`fetchPosts`, la función que envía la solicitud HTTP real, no es una función definida dentro de la función del efecto. Pero aun así no debe agregarse porque está definida **fuera de la función del componente**.

Por lo tanto, no hay forma de que esta función cambie. Se define una vez (en el archivo `BlogPosts.jsx`) y no puede cambiar. Dicho esto, este no sería el caso si se definiera dentro de la función del componente. En ese caso, cada vez que la función del componente se ejecutara de nuevo, la función `fetchPosts` también se recrearía. Este es un escenario que se discutirá más adelante en este capítulo (en la sección *Funciones como dependencias*).

En este ejemplo, sin embargo, `fetchPosts` no puede cambiar. Por lo tanto, no tiene que agregarse como dependencia (y, en consecuencia, no debería agregarse). Lo mismo ocurriría con funciones, o cualquier tipo de valores, proporcionados por el navegador o paquetes de terceros. Cualquier valor que no esté definido dentro de una función de componente no debe agregarse al array de dependencias.

> [!NOTE]
> Puede resultar confuso que una función pueda cambiar; después de todo, la lógica está codificada de forma fija, ¿verdad? Pero en JavaScript, las funciones son en realidad solo objetos y, por lo tanto, pueden cambiar. Cuando el código que contiene una función se ejecuta de nuevo (por ejemplo, una función de componente que React ejecuta de nuevo), se creará un nuevo objeto de función en la memoria.
> Si esto no es algo con lo que estés familiarizado, el siguiente recurso debería resultarte útil: [https://academind.com/tutorials/javascript-functions-are-objects](https://academind.com/tutorials/javascript-functions-are-objects).

Por lo tanto, ni `fetchedPosts` ni `fetchPosts` deben agregarse (por diferentes razones). ¿Qué pasa con `setLoadedPosts`?

`setLoadedPosts` es la función de actualización de estado devuelta por `useState()` para el valor de estado `loadedPosts`. Por lo tanto, al igual que `fetchPosts`, es una función. Sin embargo, a diferencia de `fetchPosts`, es una función que se define dentro de la función del componente (porque se llama a `useState()` dentro de la función del componente). Es una función creada por React (ya que la devuelve `useState()`), pero sigue siendo una función. Teóricamente, por lo tanto, debería agregarse como dependencia. Y de hecho, puedes agregarla sin consecuencias negativas.

Pero las funciones de actualización de estado devueltas por `useState()` son un caso especial: **React garantiza que esas funciones nunca cambiarán ni se recrearán**. Cuando la función del componente circundante (`BlogPosts`) se ejecuta nuevamente, `useState()` también se ejecuta nuevamente. Sin embargo, solo se crea una nueva función de actualización de estado la primera vez que React llama a una función de componente. Las ejecuciones posteriores no conducen a la creación de una nueva función de actualización de estado.

Debido a este comportamiento especial (es decir, React garantiza que la función en sí nunca cambia), las funciones de actualización de estado se pueden (y de hecho se deben) omitir del array de dependencias.

Por todas estas razones, `fetchedPosts`, `fetchPosts` y `setLoadedPosts` no deben agregarse al array de dependencias de `useEffect()`. `url` es la única dependencia utilizada por la función del efecto que puede cambiar (es decir, cuando el usuario introduce una nueva URL en el campo de entrada) y, por lo tanto, debe aparecer en el array.

En resumen, cuando se trata de agregar valores al array de dependencias del efecto, hay tres tipos de excepciones:
1. **Valores internos (o funciones)** que se definen y utilizan dentro del efecto (como `fetchedPosts`).
2. **Valores externos** que no están definidos dentro de una función de componente (como `fetchPosts`).
3. **Funciones de actualización de estado** (como `setLoadedPosts`).

En todos los demás casos, si se utiliza un valor en la función del efecto, **¡debe agregarse al array de dependencias!** Omitir valores incorrectamente puede provocar ejecuciones inesperadas del efecto (es decir, un efecto que se ejecuta con demasiada frecuencia o no con la frecuencia suficiente).

#### Limpieza después de los efectos (*Cleanup Functions*)
Para realizar una determinada tarea (por ejemplo, enviar una solicitud HTTP), muchos efectos simplemente deben activarse cuando cambian sus dependencias. Si bien algunos efectos se pueden volver a ejecutar varias veces sin problemas, también hay efectos que, si se ejecutan nuevamente antes de que termine la tarea anterior, son una indicación de que la tarea realizada debe cancelarse. O tal vez haya algún otro tipo de trabajo de limpieza que deba realizarse cuando el mismo efecto se ejecute nuevamente.

Aquí hay un ejemplo en el que un efecto establece un temporizador:

```javascript
import { useState, useEffect } from 'react'; 

function Alert() { 
  const [alertDone, setAlertDone] = useState(false); 

  useEffect(function () { 
    console.log('Starting Alert Timer!'); 
    setTimeout(function () { 
      console.log('Timer expired!'); 
      setAlertDone(true); 
    }, 2000); 
  }, []); 

  return ( 
    <> 
      {!alertDone && <p>Relax, you still got some time!</p>} 
      {alertDone && <p>Time to get up!</p>} 
    </> 
  ); 
} 

export default Alert;
```

Este componente `Alert` se utiliza en el componente `App`:

```javascript
import { useState } from 'react'; 
import Alert from './components/Alert.jsx'; 

function App() { 
  const [showAlert, setShowAlert] = useState(false); 

  function handleShowAlert() { 
    // state updating is done by passing a function to setShowAlert 
    // because the new state depends on the previous state (it's the opposite) 
    setShowAlert((isShowing) => !isShowing); 
  } 

  return ( 
    <> 
      <button onClick={handleShowAlert}> 
        {showAlert ? 'Hide' : 'Show'} Alert 
      </button> 
      {showAlert && <Alert />} 
    </> 
  ); 
} 

export default App;
```

En el componente `App`, el componente `Alert` se muestra condicionalmente. El estado `showAlert` se alterna a través de la función `handleShowAlert` (que se activa al hacer clic en un botón).

En el componente `Alert`, se establece un temporizador mediante `useEffect()`. Sin `useEffect()`, se crearía un bucle infinito, ya que el temporizador, al expirar, cambia algún estado del componente (el estado `alertDone` a través de la función de actualización de estado `setAlertDone`).

El array de dependencias es un array vacío porque esta función de efecto no utiliza ningún valor, variable o función del componente. `console.log()` y `setTimeout()` son funciones integradas en el navegador (y por lo tanto funciones externas), y `setAlertDone()` se puede omitir debido a las razones mencionadas en la sección anterior.

Si ejecutas esta aplicación y luego comienzas a alternar la alerta (haciendo clic en el botón), notarás un comportamiento extraño. El temporizador se establece cada vez que se renderiza el componente `Alert`, pero no borra el temporizador existente. Esto se debe al hecho de que se están ejecutando varios temporizadores simultáneamente, como puedes ver claramente si miras la consola de JavaScript en las herramientas de desarrollo de tu navegador:

**Figura 8.3**: Se inician múltiples temporizadores.

Este ejemplo se mantiene deliberadamente simple, pero hay otros escenarios en los que puedes tener una solicitud HTTP en curso que debe cancelarse antes de enviar una nueva. Hay casos como ese, en los que un efecto debe limpiarse primero antes de volver a ejecutarse.

React también proporciona una solución para este tipo de situaciones: **la función del efecto pasada como primer argumento a `useEffect()` puede devolver una función de limpieza (*cleanup function*) opcional**. Si devuelves una función dentro de tu función de efecto, **React ejecutará esa función cada vez antes de volver a ejecutar el efecto**.

Aquí está la llamada a `useEffect()` del componente `Alert` con la función de limpieza devuelta:

```javascript
useEffect(function () { 
  let timer; 
  console.log('Starting Alert Timer!'); 
  timer = setTimeout(function () { 
    console.log('Timer expired!'); 
    setAlertDone(true); 
  }, 2000); 

  return function() { 
    clearTimeout(timer); 
  } 
}, []);
```

En este ejemplo actualizado, se agrega una nueva variable `timer` (una variable local a la que solo se puede acceder dentro de la función del efecto). Esa variable almacena una referencia al temporizador creado por `setTimeout()`. Esta referencia luego se puede usar junto con `clearTimeout()` para eliminar un temporizador.

El temporizador se elimina en una función devuelta por la función del efecto, que es la función de limpieza que React ejecutará automáticamente antes de que se llame a la función del efecto la próxima vez.

Puedes ver la función de limpieza en acción si le agregas una declaración `console.log()`:

```javascript
return function() { 
  console.log('Cleanup!'); 
  clearTimeout(timer); 
}
```

En tu consola de JavaScript, esto se ve de la siguiente manera:

**Figura 8.4**: La función de limpieza se ejecuta antes de que el efecto se vuelva a ejecutar.

En la captura de pantalla anterior, puedes ver que la función de limpieza se ejecuta (indicada por el registro `Cleanup!`) justo antes de que la función del efecto se ejecute nuevamente. También puedes ver que el temporizador se borró correctamente: el primer temporizador nunca expira (no hay ningún registro `Timer expired!` para el primer temporizador en la captura de pantalla).

La función de limpieza no se ejecuta cuando se llama a la función del efecto por primera vez. Sin embargo, **React la llamará cada vez que un componente que contiene un efecto se desmonte (*unmounts*, es decir, cuando se elimina del DOM)**.

Si un efecto tiene múltiples dependencias, la función del efecto se ejecutará cada vez que cambie cualquiera de los valores de dependencia. Por lo tanto, la función de limpieza también se llamará cada vez que cambie alguna dependencia.

#### Manejo de múltiples efectos
Hasta ahora, todos los ejemplos de este capítulo han tratado con una sola llamada a `useEffect()`. Sin embargo, no estás limitado a una sola llamada por componente: puedes llamar a `useEffect()` tantas veces como sea necesario y, por lo tanto, puedes registrar tantas funciones de efectos como sea necesario.

¿Pero cuántas funciones de efectos necesitas?

Podrías comenzar a colocar cada efecto secundario en su propio envoltorio `useEffect()`. Podrías colocar cada solicitud HTTP, cada declaración `console.log()` y cada temporizador en funciones de efectos separadas.

Dicho esto, como puedes ver en algunos de los ejemplos anteriores, específicamente en el fragmento de código de la sección anterior, eso no es necesario. Allí tienes múltiples efectos en una llamada a `useEffect()` (tres declaraciones `console.log()` y un temporizador).

Un mejor enfoque sería **dividir tus funciones de efectos por dependencias**. Si un efecto secundario depende del estado A y otro efecto secundario depende del estado B, podrías colocarlos en funciones de efectos separadas (a menos que esos dos estados estén relacionados), como se muestra aquí:

```javascript
function Demo() { 
  const [a, setA] = useState(0); // state updating functions aren't called 
  const [b, setB] = useState(0); // in this example 

  useEffect(function() { 
    console.log(a); 
  }, [a]); 

  useEffect(function() { 
    console.log(b); 
  }, [b]); 

  // return some JSX code ... 
}
```

Pero el mejor enfoque es **dividir tus funciones de efectos por lógica**. Si un efecto se ocupa de obtener datos a través de una solicitud HTTP y otro efecto se trata de configurar un temporizador, a menudo tendrá sentido colocarlos en diferentes funciones de efectos (es decir, diferentes llamadas a `useEffect()`).

#### Funciones como dependencias
Diferentes efectos tienen diferentes tipos de dependencias, y un tipo común de dependencia son las funciones.

Como se mencionó anteriormente, las funciones en JavaScript son simplemente objetos. Por lo tanto, cada vez que se ejecuta un código que contiene una definición de función, se crea un nuevo objeto de función y se almacena en la memoria. Al llamar a una función, se ejecuta ese objeto de función específico en la memoria. En algunos escenarios (por ejemplo, para funciones definidas en funciones de componentes), es posible que existan múltiples objetos basados en el mismo código de función en la memoria.

Debido a este comportamiento, las funciones a las que se hace referencia en el código no son necesariamente iguales, incluso si se basan en la misma definición de función.

Considera este ejemplo:

```javascript
function Alert() { 
  function setAlert() { 
    setTimeout(function() { 
      console.log('Alert expired!'); 
    }, 2000); 
  } 

  useEffect(function() { 
    setAlert(); 
  }, [setAlert]); 

  // return some JSX code ... 
}
```

En este ejemplo, en lugar de crear un temporizador directamente dentro de la función del efecto, se crea una función `setAlert()` separada en la función del componente. Esa función `setAlert()` luego se usa en la función del efecto pasada a `useEffect()`. Dado que esa función se usa allí y porque está definida en la función del componente, debe agregarse como una dependencia a `useEffect()`.

Otra razón para esto es que cada vez que la función del componente `Alert` se ejecuta nuevamente (por ejemplo, porque cambia algún valor de estado o prop), se creará un nuevo objeto de función `setAlert`. En este ejemplo, eso no sería problemático porque `setAlert` solo contiene código estático. Un nuevo objeto de función creado para `setAlert` funcionaría exactamente de la misma manera que el anterior; por lo tanto, no importaría.

Pero ahora considera este ejemplo ajustado:

> [!NOTE]
> La aplicación completa se puede encontrar en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/examples/function-dependencies](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/examples/function-dependencies).

```javascript
function Alert() { 
  const [alertMsg, setAlertMsg] = useState('Expired!'); 

  function handleChangeAlertMsg(event) { 
    setAlertMsg(event.target.value); 
  } 

  function setAlert() { 
    setTimeout(function () { 
      console.log(alertMsg); 
    }, 2000); 
  } 

  useEffect( 
    function () { 
      setAlert(); 
    }, 
    [] 
  ); 

  return <input type="text" onChange={handleChangeAlertMsg} />; 
} 

export default Alert;
```

Ahora, se utiliza un nuevo estado `alertMsg` para establecer el mensaje de alerta real que se registra en la consola. Además, se eliminó la dependencia `setAlert` de `useEffect()`.

Si ejecutas este código, obtendrás el siguiente resultado:

**Figura 8.5**: El registro de la consola no refleja el valor ingresado.

En esta captura de pantalla, puedes ver que, a pesar de que se ingresa un valor diferente en el campo de entrada, se muestra el mensaje de alerta original.

La razón de este comportamiento es que el nuevo mensaje de alerta no se capta. No se utiliza porque, a pesar de que la función del componente se ejecuta nuevamente (porque el estado cambió), el efecto no se ejecuta nuevamente. Y la ejecución original del efecto todavía utiliza la versión anterior de la función `setAlert`, el antiguo objeto de función `setAlert`, que tiene bloqueado el mensaje de alerta anterior. Así es como funcionan las funciones de JavaScript y es por eso que, en este caso, no se logra el resultado deseado.

La solución al problema es simple: **agrega `setAlert` como una dependencia a `useEffect()`**. Siempre debes agregar todos los valores, variables o funciones utilizados en un efecto como dependencias, y este ejemplo muestra por qué debes hacerlo. Incluso las funciones pueden cambiar.

Si agregas `setAlert` al array de dependencias del efecto, obtendrás una salida diferente:

```javascript
useEffect( 
  function () { 
    setAlert(); 
  }, 
  [setAlert] 
);
```

Ten en cuenta que solo se agrega un puntero a la función `setAlert`. No ejecutas la función en el array de dependencias (eso agregaría el valor de retorno de la función como una dependencia, lo cual no suele ser el objetivo).

**Figura 8.6**: Se inician múltiples temporizadores.

Ahora, se inicia un nuevo temporizador para cada pulsación de tecla y, como resultado, el mensaje ingresado se muestra en la consola.

Por supuesto, este puede no ser el resultado deseado. Es posible que solo te interese el mensaje de error final ingresado. Esto se puede lograr agregando una función de limpieza al efecto (y ajustando `setAlert` un poco):

```javascript
function setAlert() { 
  return setTimeout(function () { 
    console.log(alertMsg); 
  }, 2000); 
} 

useEffect( 
  function () { 
    const timer = setAlert(); 
    return function () { 
      clearTimeout(timer); 
    }; 
  }, 
  [setAlert] 
);
```

Como se muestra en la sección *Limpieza después de los efectos*, el temporizador se borra con la ayuda de una referencia de temporizador y `clearTimeout()` en la función de limpieza del efecto.

Después de ajustar el código de esta manera, solo se mostrará el mensaje de alerta final que se ingresó.

Ver la función de limpieza en acción nuevamente es útil; la conclusión principal es la importancia de agregar todas las dependencias, incluidas las dependencias de funciones.

Una alternativa a incluir la función como dependencia sería mover toda la definición de la función dentro de la función del efecto, porque cualquier valor que se defina y utilice dentro de una función de efecto no debe agregarse como dependencia:

```javascript
useEffect( 
  function () { 
    function setAlert() { 
      return setTimeout(function () { 
        console.log(alertMsg); 
      }, 2000); 
    } 

    const timer = setAlert(); 
    return function () { 
      clearTimeout(timer); 
    }; 
  }, 
  [] 
);
```

Por supuesto, también podrías deshacerte de la función `setAlert` por completo y simplemente mover el código de la función dentro de la función del efecto.

De cualquier manera, tendrás que agregar una nueva dependencia, `alertMsg`, que ahora se usa dentro de la función del efecto. Aunque la función `setAlert` ya no sea una dependencia, aún debes agregar cualquier valor utilizado (y `alertMsg` se usa en la función del efecto ahora):

```javascript
useEffect( 
  function () { 
    function setAlert() { 
      return setTimeout(function () { 
        console.log(alertMsg); 
      }, 2000); 
    } 

    const timer = setAlert(); 
    return function () { 
      clearTimeout(timer); 
    }; 
  }, 
  [alertMsg] 
);
```

Por lo tanto, esta forma alternativa de escribir el código se reduce simplemente a preferencias personales: no reduce el número de dependencias.

Te desharías de una dependencia de función si movieras la función fuera de la función del componente. Esto se debe a que, como se mencionó en la sección *Dependencias innecesarias*, las dependencias externas (por ejemplo, las integradas en el navegador o definidas fuera de las funciones de los componentes) no deben agregarse como dependencias.

Sin embargo, en el caso de la función `setAlert`, esto no es posible porque `setAlert` usa `alertMsg`. Dado que `alertMsg` es un valor de estado del componente, la función que lo usa debe definirse dentro de la función del componente; de lo contrario, no tendrá acceso a ese valor de estado.

Todo esto puede sonar bastante confuso, pero se reduce a dos reglas simples:
1. Agrega siempre todas las dependencias no externas, sin importar si son variables o funciones.
2. Las funciones son solo objetos y pueden cambiar si el código que las rodea se ejecuta nuevamente.

#### Evitar ejecuciones innecesarias de efectos
Dado que todas las dependencias deben agregarse a `useEffect()`, a veces terminas con un código que hace que un efecto se ejecute innecesariamente.

Considera el siguiente componente de ejemplo:

> [!NOTE]
> El ejemplo completo se puede encontrar en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/examples/unnecessary-executions](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/examples/unnecessary-executions).

```javascript
import { useState, useEffect } from 'react'; 

function Alert() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  const [enteredPassword, setEnteredPassword] = useState(''); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleUpdatePassword(event) { 
    setEnteredPassword(event.target.value); 
  } 

  function validateEmail() { 
    if (!enteredEmail.includes('@')) { 
      console.log('Invalid email!'); 
    } 
  } 

  useEffect(function () { 
    validateEmail(); 
  }, [validateEmail]); 

  return ( 
    <form> 
      <div> 
        <label>Email</label> 
        <input type="email" onChange={handleUpdateEmail} /> 
      </div> 
      <div> 
        <label>Password</label> 
        <input type="password" onChange={handleUpdatePassword} /> 
      </div> 
      <button>Save</button> 
    </form> 
  ); 
} 

export default Alert;
```

Este componente contiene un formulario con dos campos de entrada. Los valores ingresados se almacenan en dos valores de estado diferentes (`enteredEmail` y `enteredPassword`). La función `validateEmail()` luego realiza cierta validación de correo electrónico y, si la dirección de correo electrónico no es válida, registra un mensaje en la consola. `validateEmail()` se ejecuta con la ayuda de `useEffect()`.

El problema con este código es que la función del efecto se ejecutará siempre que cambie `validateEmail` porque, correctamente, `validateEmail` se agregó como una dependencia. Pero `validateEmail` cambiará cada vez que la función del componente se ejecute de nuevo. Y ese no es solo el caso de los cambios de estado en `enteredEmail`, sino también cada vez que cambia `enteredPassword`, a pesar de que ese valor de estado no se utiliza en absoluto dentro de `validateEmail`.

Esta ejecución innecesaria del efecto se puede evitar con varias soluciones:
1. Podrías mover el código dentro de `validateEmail` directamente a la función del efecto (`enteredEmail` sería entonces la única dependencia del efecto, evitando ejecuciones del efecto cuando cambie cualquier otro estado).
2. Podrías evitar usar `useEffect()` por completo ya que podrías realizar la validación del correo electrónico dentro de `handleUpdateEmail`. Tener `console.log()` (un efecto secundario) allí sería aceptable ya que no causaría ningún daño.
3. Podrías llamar a `validateEmail()` directamente en la función del componente: como no cambia ningún estado, no provocaría un bucle infinito.

> [!NOTE]
> Hay un artículo en la documentación oficial de React que destaca escenarios en los que es posible que no necesites `useEffect()`: [https://react.dev/learn/you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect).
> Además, creé un video que resume las situaciones más importantes en las que necesitas o no `useEffect()`: [https://www.youtube.com/watch?v=V1f8MOQiHRw](https://www.youtube.com/watch?v=V1f8MOQiHRw).

Por supuesto, en algunos otros escenarios, es posible que necesites usar `useEffect()`. Afortunadamente, React también ofrece una solución para situaciones como esta: puedes envolver la función que se usa como dependencia con otro Hook de React, el Hook **`useCallback()`**.

El código ajustado se vería así:

```javascript
import { useState, useEffect, useCallback } from 'react'; 

function Alert() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  const [enteredPassword, setEnteredPassword] = useState(''); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleUpdatePassword(event) { 
    setEnteredPassword(event.target.value); 
  } 

  const validateEmail = useCallback( 
    function () { 
      if (!enteredEmail.includes('@')) { 
        console.log('Invalid email!'); 
      } 
    }, 
    [enteredEmail] 
  ); 

  useEffect( 
    function() { 
      validateEmail(); 
    }, 
    [validateEmail] 
  ); 

  // return JSX code ... 
} 

export default Alert;
```

`useCallback()`, como todos los Hooks de React, es una función que se ejecuta directamente dentro de la función del componente. Al igual que `useEffect()`, acepta dos argumentos: otra función (que puede ser anónima o una función con nombre) y un array de dependencias.

A diferencia de `useEffect()`, `useCallback()` no ejecuta la función recibida. En su lugar, **`useCallback()` garantiza que una función solo se recree si una de las dependencias especificadas ha cambiado**. El comportamiento predeterminado de JavaScript de crear un nuevo objeto de función cada vez que el código circundante se ejecuta nuevamente se desactiva (sintéticamente).

`useCallback()` devuelve el último objeto de función guardado. Por lo tanto, ese valor devuelto (que es una función) se guarda en una variable o constante (`validateEmail` en el ejemplo anterior).

Dado que la función envuelta por `useCallback()` ahora solo cambia cuando cambia una de las dependencias, la función devuelta se puede usar como una dependencia para `useEffect()` sin ejecutar ese efecto para todo tipo de cambios de estado o actualizaciones de componentes.

En el caso del ejemplo anterior, la función del efecto solo se ejecutaría cuando cambie `enteredEmail`, porque ese es el único cambio que conducirá a la creación de un nuevo objeto de función `validateEmail`.

Otra razón común para la ejecución innecesaria de efectos es el uso de **objetos como dependencias**, como en este ejemplo:

```javascript
import { useEffect } from 'react'; 

function Error(props) { 
  useEffect( 
    function () { 
      // performing some error logging 
      // in a real app, a HTTP request might be sent to some analytics API 
      console.log('An error occurred!'); 
      console.log(props.message); 
    }, 
    [props] 
  ); 

  return <p>{props.message}</p>; 
} 

export default Error;
```

Este componente `Error` se utiliza en otro componente, el componente `Form`, de esta manera:

```javascript
import { useState } from 'react'; 
import Error from './Error.jsx'; 

function Form() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  const [errorMessage, setErrorMessage] = useState(''); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleSubmitForm(event) { 
    event.preventDefault(); 
    if (!enteredEmail.endsWith('.com')) { 
      setErrorMessage('Only email addresses ending with .com are accepted!'); 
    } 
  } 

  return ( 
    <form onSubmit={handleSubmitForm}> 
      <div> 
        <label>Email</label> 
        <input type="email" onChange={handleUpdateEmail} /> 
      </div> 
      {errorMessage && <Error message={errorMessage} />} 
      <button>Submit</button> 
    </form> 
  ); 
} 

export default Form;
```

El componente `Error` recibe un mensaje de error a través de props (`props.message`) y lo muestra en la pantalla. Además, con la ayuda de `useEffect()`, realiza un registro de errores. En este ejemplo, el error simplemente se envía a la consola de JavaScript. En una aplicación real, el error podría enviarse a alguna API de análisis de datos a través de una solicitud HTTP. De cualquier manera, se realiza un efecto secundario que depende del mensaje de error.

El componente `Form` contiene dos valores de estado, que rastrean la dirección de correo electrónico ingresada así como el estado de error de la entrada. Si se envía un valor de entrada no válido, se establece `errorMessage` y se muestra el componente `Error`.

La parte interesante de este ejemplo es el array de dependencias de `useEffect()` dentro del componente `Error`: contiene el objeto `props` como una dependencia (`props` es siempre un objeto, que agrupa todos los valores de las props). Al utilizar objetos (`props` o cualquier otro objeto; no importa) como dependencias para `useEffect()`, el resultado pueden ser ejecuciones innecesarias de funciones de efectos.

Puedes ver este problema en este ejemplo. Si ejecutas la aplicación e ingresas una dirección de correo electrónico no válida (por ejemplo, `test@test.de`), notarás que las pulsaciones de teclas posteriores en el campo de entrada de correo electrónico harán que el mensaje de error se registre (a través de la función del efecto) con cada pulsación de tecla.

> [!NOTE]
> El código completo se puede encontrar en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/examples/objects-as-dependencies](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/examples/objects-as-dependencies).

**Figura 8.7**: Se registra un nuevo mensaje de error para cada pulsación de tecla.

Esas ejecuciones adicionales pueden ocurrir porque las reevaluaciones de los componentes (es decir, las funciones de los componentes que React vuelve a invocar) producirán objetos de JavaScript completamente nuevos. Incluso si los valores de las propiedades de esos objetos no cambiaron (como en el ejemplo anterior), técnicamente JavaScript crea un objeto completamente nuevo en la memoria. Dado que el efecto depende de todo el objeto, React solo "ve" que hay una nueva versión de ese objeto y, por lo tanto, ejecuta el efecto nuevamente.

En el ejemplo anterior, se crea un nuevo objeto `props` (para el componente `Error`) cada vez que React llama a la función del componente `Form`, incluso si el mensaje de error (el único valor de prop que se establece) no cambió.

En este ejemplo, eso solo es molesto porque satura la consola de JavaScript en las herramientas de desarrollo. Sin embargo, si estuvieras enviando una solicitud HTTP a alguna API de análisis, podría causar problemas de ancho de banda y hacer que la aplicación sea más lenta. Por lo tanto, es mejor que te acostumbres a evitar ejecuciones innecesarias de efectos como regla general.

En el caso de dependencias de objetos, la mejor manera de evitar ejecuciones innecesarias es **simplemente desestructurar el objeto** para que puedas pasar solo aquellas propiedades del objeto como dependencias que son necesarias para el efecto:

```javascript
function Error(props) { 
  const { message } = props; // destructure to extract required properties 

  useEffect( 
    function () { 
      console.log('An error occurred!'); 
      console.log(message); 
    }, 
    // [props] // don't use the entire props object! 
    [message] 
  ); 

  return <p>{message}</p>; 
}
```

En el caso de las props, también podrías desestructurar el objeto directamente en la lista de parámetros de la función del componente:

```javascript
function Error({message}) { 
  // ... 
}
```

Al utilizar este enfoque, te aseguras de que solo los valores de propiedad requeridos se establezcan como dependencias. Por lo tanto, incluso si el objeto se recrea, el valor de la propiedad (en este caso, el valor de la propiedad `message`) es lo único que importa. Si no cambia, la función del efecto no se ejecutará nuevamente.

#### Efectos y código asíncrono
Algunos efectos tratan con código asíncrono (enviar solicitudes HTTP es un ejemplo típico). Al realizar tareas asíncronas en funciones de efectos, hay una regla importante que debes tener en cuenta: **la función del efecto en sí no debe ser asíncrona y no debe devolver una promesa**. Esto no significa que no puedas trabajar con promesas en efectos: simplemente no debes devolver una promesa.

Es posible que desees utilizar `async/await` para simplificar el código asíncrono, pero al hacerlo dentro de una función de efecto, es fácil devolver accidentalmente una promesa. Por ejemplo, el siguiente código funcionaría pero no sigue las mejores prácticas:

```javascript
useEffect(async function () { 
  const fetchedPosts = await fetchPosts(); 
  setLoadedPosts(fetchedPosts); 
}, []);
```

Agregar la palabra clave `async` delante de `function` habilita el uso de `await` dentro de la función, lo que hace que trabajar con código asíncrono (es decir, con promesas) sea más conveniente.

Pero la función de efecto pasada a `useEffect()` solo debe devolver una función normal (la función de limpieza), si es que devuelve algo. No debe devolver una promesa. De hecho, React genera una advertencia al intentar ejecutar código como el fragmento anterior:

**Figura 8.8**: React muestra una advertencia sobre el uso de `async` en una función de efecto.

Para evitar esta advertencia, puedes usar promesas sin `async/await`, de esta manera:

```javascript
useEffect(function () { 
  fetchPosts().then((fetchedPosts) => setLoadedPosts(fetchedPosts)); 
}, []);
```

Esto funciona porque la función del efecto no devuelve la promesa.

Alternativamente, si deseas utilizar `async/await`, puedes crear una **función envoltorio separada dentro de la función del efecto**, que luego se ejecuta en el efecto:

```javascript
useEffect(function () { 
  async function loadData() { 
    const fetchedPosts = await fetchPosts(); 
    setLoadedPosts(fetchedPosts); 
  } 

  loadData(); 
}, []);
```

Al hacer eso, la función del efecto en sí no es asíncrona (no devuelve una promesa), pero aún puedes usar `async/await`.

---

### Sección 6: Reglas de los Hooks

En este capítulo, se introdujeron dos nuevos Hooks: `useEffect()` y `useCallback()`. Ambos Hooks son muy importantes, especialmente `useEffect()`, ya que es un Hook que normalmente usarás mucho. Junto con `useState()` (introducido en el Capítulo 4, *Trabajando con Eventos y Estado*) y `useRef()` (introducido en el Capítulo 7, *Portales y Refs*), ahora tienes un conjunto sólido de Hooks clave de React.

Al trabajar con Hooks de React, hay **dos reglas** (las llamadas *reglas de los Hooks*) que debes seguir:
1. **Llama a los Hooks únicamente en el nivel superior de las funciones de los componentes**. No los llames dentro de sentencias `if`, bucles o funciones anidadas.
2. **Llama a los Hooks únicamente dentro de componentes de React o Custom Hooks** (los Custom Hooks se cubrirán en el Capítulo 12, *Creación de Custom Hooks en React*).

Estas reglas existen porque los Hooks de React no funcionarán como se espera si se usan de una manera no compatible. Afortunadamente, React generará un mensaje de advertencia si violas una de estas reglas; por lo tanto, notarás si lo haces accidentalmente.

---

### Sección 7: Resumen y puntos clave

- Las acciones que no están directamente relacionadas con el proceso principal de una función pueden considerarse **efectos secundarios** (*side effects*).
- Los efectos secundarios pueden ser tareas asíncronas (por ejemplo, enviar una solicitud HTTP), pero también pueden ser sincrónicas (por ejemplo, `console.log()` o acceder al almacenamiento del navegador).
- A menudo se necesitan efectos secundarios para lograr un objetivo determinado, pero es una buena idea separarlos del proceso principal de una función.
- Los efectos secundarios pueden volverse problemáticos si provocan bucles infinitos (debido a los ciclos de actualización entre el efecto y el estado).
- `useEffect()` es un Hook de React que debe usarse para envolver efectos secundarios y ejecutarlos de manera segura.
- `useEffect()` toma una **función de efecto** y un **array de dependencias del efecto**.
- La función del efecto se ejecuta directamente después de invocar la función del componente (no simultáneamente).
- Cualquier valor, variable o función utilizada dentro de un efecto debe agregarse al array de dependencias.
- Las excepciones del array de dependencias son valores externos (definidos fuera de una función de componente), funciones de actualización de estado o valores definidos y utilizados dentro de la función del efecto.
- Si no se especifica ningún array de dependencias, la función del efecto se ejecuta después de cada invocación de la función del componente.
- Si se especifica un array de dependencias vacío, la función del efecto se ejecuta una vez cuando el componente se monta por primera vez (es decir, cuando se crea por primera vez).
- Las funciones de efecto también pueden devolver funciones de limpieza opcionales que se llaman justo antes de que se vuelva a ejecutar una función de efecto (y justo antes de que se elimine un componente del DOM).
- Las funciones de efecto **no deben devolver promesas**.
- Para las dependencias de funciones, `useCallback()` puede ayudar a reducir la cantidad de ejecuciones de efectos.
- Para las dependencias de objetos, la desestructuración puede ayudar a reducir la cantidad de ejecuciones de efectos.

---

### Sección 8: ¿Qué sigue?

El manejo de efectos secundarios es un problema común al crear aplicaciones porque la mayoría de las aplicaciones necesitan algún tipo de efectos secundarios (por ejemplo, enviar una solicitud HTTP) para funcionar correctamente. Por lo tanto, los efectos secundarios no son un problema en sí mismos, pero pueden causar problemas (por ejemplo, bucles infinitos) si se manejan incorrectamente.

Con el conocimiento adquirido en este capítulo, ya sabes cómo manejar los efectos secundarios de manera eficiente con `useEffect()` y los conceptos clave relacionados.

Muchos efectos secundarios se activan debido a la entrada o interacción del usuario, por ejemplo, porque se envió un formulario. El próximo capítulo volverá a revisar el concepto de envíos de formularios explorando la función de **Form Actions** de React.

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/08-effects/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/08-effects/exercises/questions-answers.md):

1. ¿Cómo definirías un efecto secundario?
2. ¿Cuál es un problema potencial que podría surgir con algunos efectos secundarios en los componentes de React?
3. ¿Cómo funciona el Hook `useEffect()`?
4. ¿Qué valores no deben agregarse al array de dependencias de `useEffect()`?
5. ¿Qué valor puede devolver la función del efecto? ¿Y qué tipo de valor no debe devolverse?

---

### Sección 10: Aplica lo aprendido

Ahora que conoces los efectos, puedes agregar funciones aún más interesantes a tus aplicaciones de React. Obtener datos a través de HTTP al renderizar un componente es tan fácil como acceder al almacenamiento del navegador cuando cambia algún estado.

En la siguiente sección, encontrarás una actividad que te permitirá practicar el trabajo con efectos y `useEffect()`. Como siempre, necesitarás emplear algunos de los conceptos cubiertos en capítulos anteriores (como trabajar con el estado).

#### Actividad 8.1: Construcción de un blog básico
En esta actividad, debes agregar lógica a una aplicación de React existente para renderizar una lista de títulos de publicaciones de blog obtenidos de una API web de backend y enviar publicaciones de blog recién agregadas a esa misma API. La API de backend utilizada es [https://jsonplaceholder.typicode.com/](https://jsonplaceholder.typicode.com/), que es una API de prueba que en realidad no almacena ningún dato que le envíes. Siempre devolverá los mismos datos ficticios, pero es perfecta para practicar el envío de solicitudes HTTP.

Como beneficio adicional (*bonus*), también puedes agregar lógica para cambiar el texto del botón de envío mientras la solicitud HTTP para guardar la nueva publicación de blog está en camino.

Utiliza tus conocimientos sobre efectos y solicitudes HTTP del lado del navegador para implementar una solución.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/activities/practice-1-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start`, en este caso) para usar la versión correcta del código.
> Para esta actividad, necesitas saber cómo enviar solicitudes HTTP (GET, POST, etc.) a través de JavaScript (por ejemplo, a través de la función `fetch()` o con la ayuda de una biblioteca de terceros). Si aún no tienes ese conocimiento, este recurso puede ayudarte a comenzar: [http://packt.link/DJ6Hx](http://packt.link/DJ6Hx).

Después de descargar el código y ejecutar `npm install` en la carpeta del proyecto para instalar todas las dependencias requeridas, los pasos de la solución son los siguientes:
1. Envía una solicitud HTTP GET a la API ficticia para obtener publicaciones de blog dentro del componente `App` (cuando el componente se renderiza por primera vez).
2. Muestra las publicaciones de blog de prueba obtenidas en la pantalla.
3. Maneja los envíos de formularios y envía una solicitud HTTP POST (con algunos datos ficticios) a la API de backend de prueba.
4. *Bonus*: Establece el texto del botón en "Saving…" mientras la solicitud está en camino (y en "Save" cuando no lo esté).

El resultado esperado debe ser una interfaz de usuario que se vea así:

**Figura 8.9**: La interfaz de usuario final.

> [!NOTE]
> Encontrarás una solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/08-effects/activities/practice-1).
