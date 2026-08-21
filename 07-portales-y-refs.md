# Parte 1: Fundamentos de React

## Capítulo 7: Portales y Refs

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Utilizar el acceso directo a elementos del DOM para interactuar con ellos.
- Exponer las funciones y los datos de tus componentes a otros componentes.
- Controlar la posición de los elementos JSX renderizados en el DOM.

---

### Sección 1: Introducción

React.js se trata por completo de construir interfaces de usuario y, en el contexto de este libro, se trata específicamente de construir interfaces de usuario web.

Las interfaces de usuario web se basan, en última instancia, en el Modelo de Objetos del Documento (**DOM**, *Document Object Model*). Puedes usar JavaScript para leer o manipular el DOM. Esto es lo que te permite crear sitios web interactivos: puedes agregar, eliminar o editar elementos del DOM después de que se haya cargado una página. Esto se puede utilizar para agregar o quitar ventanas modales superpuestas (*overlays*) o para leer los valores introducidos en los campos de entrada.

Esto se analizó en el Capítulo 1, *React – Qué es y por qué*, y, como aprendiste allí, React se utiliza para simplificar este proceso. En lugar de manipular el DOM o leer valores de los elementos del DOM manualmente, puedes usar React para describir el estado deseado. React luego se encarga de los pasos necesarios para lograr este estado deseado.

Sin embargo, existen escenarios y casos de uso en los que, a pesar de usar React, aún deseas poder acceder directamente a elementos específicos del DOM; por ejemplo, para leer un valor ingresado por un usuario en un campo de entrada, o si no estás satisfecho con la posición elegida por React para un elemento recién insertado en el DOM.

React proporciona ciertas funcionalidades que te ayudan exactamente en este tipo de situaciones: **Portales (*Portals*)** y **Referencias (*Refs*)**. Aunque manipular directamente el DOM seguirá sin ser una gran idea, estas herramientas, como aprenderás a lo largo de este capítulo, pueden ayudarte a leer los valores de los elementos del DOM o a cambiar la estructura del DOM sin trabajar en contra de React.

---

### Sección 2: Un mundo sin Refs

Considera el siguiente ejemplo: tienes un sitio web que renderiza un campo de entrada solicitando la dirección de correo electrónico de un usuario. Podría verse así:

**Figura 7.1**: Un formulario de ejemplo con un campo de entrada de correo electrónico.

El código para el componente responsable de renderizar el formulario y manejar el valor de la dirección de correo electrónico ingresada podría verse así:

```javascript
function EmailForm() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  console.log(enteredEmail); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleSubmitForm(event) { 
    event.preventDefault(); 
    // could send enteredEmail to a backend server 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmitForm}> 
      <label htmlFor="email">Your email</label> 
      <input type="email" id="email" onChange={handleUpdateEmail} /> 
      <button>Save</button> 
    </form> 
  ); 
}
```

Como puedes ver, este ejemplo utiliza el Hook `useState()`, combinado con el evento `change`, para registrar las pulsaciones de teclas en el campo de entrada de correo electrónico y almacenar el valor ingresado.

Este código funciona bien y no hay nada de malo en tener este tipo de código en tu aplicación. Pero agregar el escuchador de eventos y el estado adicionales, así como agregar la función para actualizar el estado cada vez que se activa el evento `change`, es una cantidad considerable de código repetitivo (*boilerplate*) para una tarea simple: obtener la dirección de correo electrónico ingresada.

El fragmento de código anterior no hace nada más con la dirección de correo electrónico que enviarla. En otras palabras, la única razón para usar el estado `enteredEmail` en el ejemplo es leer el valor ingresado.

Aunque `enteredEmail` solo se requiere en la función `handleSubmitForm()`, React volverá a ejecutar la función del componente `EmailForm` con cada actualización del estado `enteredEmail`, es decir, **con cada pulsación de tecla en el campo `<input>`**. Esto tampoco es ideal, ya que conduce a una gran cantidad de ejecuciones de código innecesarias y, por lo tanto, a posibles problemas de rendimiento.

En escenarios como este, se podría ahorrar bastante código (y quizás rendimiento) si recurrieras a algo de lógica de JavaScript puro (*vanilla JavaScript*):

```javascript
const emailInputEl = document.getElementById('email'); 
const enteredEmailVal = emailInputEl.value;
```

Estas dos líneas de código (que teóricamente podrían fusionarse en una sola línea) te permiten obtener un elemento del DOM y leer el valor almacenado actualmente.

El problema con este tipo de código es que no utiliza React. Y si estás creando una aplicación de React, realmente deberías apegarte a React cuando trabajes con el DOM. No comiences a mezclar tu propio código de JavaScript puro que accede al DOM con el código de React.

Esto puede provocar comportamientos no deseados o errores, especialmente si comienzas a manipular el DOM. Podría provocar errores porque React no estaría al tanto de tus cambios en ese caso; la interfaz de usuario renderizada real no estaría sincronizada con la interfaz de usuario asumida por React. Incluso si solo estás leyendo del DOM, es un buen hábito no comenzar a mezclar métodos de acceso al DOM de JavaScript puro con tu código de React.

Para permitirte obtener elementos del DOM y leer valores sin romper este principio, React te ofrece un concepto especial que puedes usar: las **Refs**.

Ref significa **referencia**, y es una función que te permite almacenar referencias a valores, por ejemplo, a elementos del DOM desde dentro de un componente de React. El código de JavaScript puro anterior haría lo mismo (también te da acceso a un elemento renderizado), pero al usar Refs, puedes obtener acceso sin mezclar código de JavaScript puro en tu código de React.

Las Refs se pueden crear utilizando un Hook especial de React llamado **`useRef()`**.

Este Hook se puede ejecutar para generar un objeto ref:

```javascript
import { useRef } from 'react'; 

function EmailForm() { 
  const emailRef = useRef(null); 
  // other code ... 
};
```

Este objeto Ref generado, `emailRef` en el ejemplo anterior, se establece inicialmente en `null` pero luego se puede asignar a cualquier elemento JSX. Esta asignación se realiza a través de una prop especial (la prop **`ref`**) que es admitida automáticamente por cada elemento JSX:

```javascript
return ( 
  <form className={classes.form} onSubmit={handleSubmitForm}> 
    <label htmlFor="email">Your email</label> 
    <input ref={emailRef} type="email" id="email" /> 
    <button>Save</button> 
  </form> 
);
```

Al igual que la prop `key` introducida en el Capítulo 5, *Renderizado de Listas y Contenido Condicional*, la prop `ref` la proporciona React. La prop `ref` requiere un objeto Ref, es decir, uno que haya sido creado a través de `useRef()`.

En este ejemplo, `useRef()` recibe `null` como valor inicial, ya que técnicamente aún no está asignado al elemento del DOM cuando la función del componente se ejecuta por primera vez. Es solo después de ese ciclo de renderizado inicial del componente cuando se establecerá la conexión. Por lo tanto, después de esta primera ejecución de la función del componente, el valor almacenado en la Ref será el objeto del DOM subyacente del elemento `<input>` en este ejemplo.

Con ese objeto Ref creado y asignado, puedes usarlo para obtener acceso al elemento JSX conectado (al elemento `<input>` en este ejemplo). Solo hay una cosa importante a tener en cuenta: para obtener el elemento conectado, **debes acceder a una propiedad especial `current` en el objeto Ref creado**. Esto es necesario porque React almacena el valor asignado al objeto Ref en un objeto anidado, accesible a través de la propiedad `current`, como se muestra aquí:

```javascript
function handleSubmitForm(event) { 
  event.preventDefault(); 
  const enteredEmail = emailRef.current.value; // .current is mandatory! 
  // could send enteredEmail to a backend server 
};
```

`emailRef.current` produce el objeto del DOM subyacente que se renderizó para el elemento JSX conectado. En este caso, por lo tanto, permite el acceso al objeto del DOM del elemento input. Dado que ese objeto del DOM tiene una propiedad `value`, se puede acceder a esta propiedad `value` sin problemas.

> [!NOTE]
> Para obtener más información sobre este tema, consulta [https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attributes).

Con este tipo de código, puedes leer el valor del elemento del DOM sin tener que usar `useState()` y un escuchador de eventos. Por lo tanto, el código del componente final se vuelve bastante más limpio y conciso:

```javascript
function EmailForm() { 
  const emailRef = useRef(null); 

  function handleSubmitForm(event) { 
    event.preventDefault(); 
    const enteredEmail = emailRef.current.value; 
    // could send enteredEmail to a backend server 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmitForm}> 
      <label htmlFor="email">Your email</label> 
      <input ref={emailRef} type="email" id="email" /> 
      <button>Save</button> 
    </form> 
  ); 
}
```

---

### Sección 3: Refs frente a Estado

Dado que las Refs se pueden utilizar para obtener un acceso rápido y sencillo a los elementos del DOM, la pregunta que podría surgir es si siempre deberías utilizar Refs en lugar de estado.

La respuesta clara a esta pregunta es **"no"**.

Las Refs pueden ser una muy buena alternativa en casos de uso como el que se muestra arriba, cuando necesitas acceso de solo lectura a un elemento. Este suele ser el caso cuando se trata de la entrada del usuario. En general, las Refs pueden reemplazar al estado si solo estás accediendo a algún valor para leerlo cuando se ejecuta alguna función (una función controladora de envío de formulario, por ejemplo). Tan pronto como necesites cambiar valores y esos cambios deban reflejarse en la interfaz de usuario (por ejemplo, renderizando algún contenido condicional), las Refs quedan descartadas.

En el ejemplo anterior, si además de obtener el valor ingresado también quisieras restablecer (es decir, borrar) la entrada de correo electrónico después de enviar el formulario, deberías usar el estado nuevamente. Si bien podrías restablecer el campo de entrada con la ayuda de una Ref, no deberías hacerlo. Estarías comenzando a manipular el DOM directamente, y solo React debería hacer eso, con sus propios métodos internos basados en el código declarativo que le proporcionas.

Debes evitar restablecer la entrada de correo electrónico de esta manera:

```javascript
function EmailForm() { 
  const emailRef = useRef(null); 

  function handleSubmitForm(event) { 
    event.preventDefault(); 
    const enteredEmail = emailRef.current.value; 
    // could send enteredEmail to a backend server 
    emailRef.current.value = ''; // resetting the input value 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmitForm}> 
      <label htmlFor="email">Your email</label> 
      <input ref={emailRef} type="email" id="email" /> 
      <button>Save</button> 
    </form> 
  ); 
}
```

En su lugar, debes restablecerlo utilizando el concepto de estado de React y siguiendo el enfoque declarativo adoptado por React:

```javascript
function EmailForm() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  } 

  function handleSubmitForm(event) { 
    event.preventDefault(); 
    // could send enteredEmail to a backend server 
    // reset by setting the state + using the value prop below 
    setEnteredEmail(''); 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmitForm}> 
      <label htmlFor="email">Your email</label> 
      <input type="email" id="email" onChange={handleUpdateEmail} value={enteredEmail} /> 
      <button>Save</button> 
    </form> 
  ); 
}
```

> [!NOTE]
> Como regla general, simplemente debes intentar evitar escribir código imperativo en proyectos de React. En su lugar, indícale a React cómo debe verse la interfaz de usuario final y deja que React descubra cómo llegar allí.
> Leer valores a través de Refs es una excepción aceptable, pero la manipulación de elementos del DOM (con o sin Refs, por ejemplo, seleccionando directamente nodos del DOM a través de `document.getElementById()` o similar) debe evitarse. Una excepción poco común es un escenario como llamar a `focus()` en un objeto del DOM de un elemento de entrada, porque métodos como `focus()` no suelen causar ningún cambio en el DOM que pueda romper la aplicación de React.

---

### Sección 4: Uso de Refs para más que acceso al DOM

Acceder a elementos del DOM (para leer valores) es uno de los casos de uso más comunes para usar Refs. Como se mostró anteriormente, puede ayudarte a reducir código en ciertas situaciones.

Pero las Refs son más que simples "puentes de conexión con elementos": son objetos que se pueden usar para **almacenar todo tipo de valores**, no solo punteros a objetos del DOM. Por ejemplo, también puedes almacenar cadenas, números o cualquier otro tipo de valor en una Ref:

```javascript
const passwordRetries = useRef(0);
```

Puedes pasar un valor inicial a `useRef()` (`0` en este ejemplo) y luego acceder o cambiar ese valor en cualquier momento dentro del componente al que pertenece la Ref:

```javascript
passwordRetries.current = 1;
```

Sin embargo, aún tienes que usar la propiedad `current` para leer y cambiar el valor almacenado, porque, como se mencionó anteriormente, aquí es donde React almacenará el valor real que pertenece a la Ref.

Esto puede ser útil para almacenar datos que deberían **"sobrevivir" a las reevaluaciones de los componentes**. Como aprendiste en el Capítulo 4, *Trabajando con Eventos y Estado*, React ejecutará las funciones de los componentes cada vez que cambie el estado de un componente. Dado que la función se ejecuta nuevamente, cualquier dato almacenado en variables con ámbito de función se perdería. Considera el siguiente ejemplo:

```javascript
function Counters() { 
  const [counter1, setCounter1] = useState(0); 
  const counterRef = useRef(0); 
  let counter2 = 0; 

  function handleChangeCounters() { 
    setCounter1(1); 
    counter2 = 1; 
    counterRef.current = 1; 
  }; 

  return ( 
    <> 
      <button onClick={handleChangeCounters}>Change Counters</button> 
      <ul> 
        <li>Counter 1: {counter1}</li> 
        <li>Counter 2: {counter2}</li> 
        <li>Counter 3: {counterRef.current}</li> 
      </ul> 
    </> 
  ); 
};
```

En este ejemplo, los contadores 1 y 3 cambiarían a 1 una vez que se hace clic en el botón. Sin embargo, el contador 2 permanecería en cero, a pesar de que la variable `counter2` también cambia a un valor de 1 en `handleChangeCounters`:

**Figura 7.2**: Solo cambiaron dos de los tres valores de contador.

En este ejemplo, se debe esperar que el valor del estado cambie y que el nuevo valor se refleje en la interfaz de usuario actualizada. Esa es toda la idea detrás del estado, después de todo.

Sin embargo, la Ref (`counterRef`) también mantiene su valor actualizado a lo largo de las reevaluaciones del componente. Ese es el comportamiento descrito anteriormente: **las Refs no se restablecen ni se borran cuando la función del componente circundante se ejecuta de nuevo**. La variable estándar de JavaScript (`counter2`) no conserva su valor: aunque se cambia en `handleChangeCounters`, se inicializa una nueva variable cuando la función del componente se ejecuta nuevamente; por lo tanto, el valor actualizado (`1`) se pierde.

En este ejemplo, podría parecer nuevamente que las Refs pueden reemplazar al estado, pero el ejemplo en realidad muestra muy bien por qué ese no es el caso. Intenta reemplazar `counter1` con otra Ref (para que no quede ningún valor de estado en el componente) y hacer clic en el botón:

```javascript
import { useRef } from 'react'; 

function Counters() { 
  const counterRef1 = useRef(0); 
  const counterRef2 = useRef(0); 
  let counter2 = 0; 

  function handleChangeCounters() { 
    counterRef1.current = 1; 
    counter2 = 1; 
    counterRef2.current = 1; 
  } 

  return ( 
    <> 
      <button onClick={handleChangeCounters}>Change Counters</button> 
      <ul> 
        <li>Counter 1: {counterRef1.current}</li> 
        <li>Counter 2: {counter2}</li> 
        <li>Counter 3: {counterRef2.current}</li> 
      </ul> 
    </> 
  ); 
}; 

export default Counters;
```

No cambiará nada en la página porque, si bien se registra el clic en el botón y se ejecuta la función `handleChangeCounters`, no se inicia ningún cambio de estado, y los cambios de estado (iniciados mediante las llamadas a la función de actualización de estado `setXYZ`) son los desencadenantes que hacen que React reevalúe un componente. **Los cambios en los valores de Ref no desencadenan reevaluaciones**.

**Figura 7.3**: Los valores del contador no cambian.

Como puedes ver, cambiar los valores de Ref no hace que las funciones de los componentes se ejecuten nuevamente; el estado, por otro lado, sí lo hace. Sin embargo, si una función de componente se ejecuta nuevamente (debido a un cambio de estado), los valores de Ref se conservan y no se descartan.

Por lo tanto, si tienes datos que deberían sobrevivir a las reevaluaciones de los componentes pero no deberían administrarse como estado (porque los cambios en esos datos no deberían hacer que el componente se reevalúe cuando cambien), podrías usar una Ref:

```javascript
const passwordRetries = useRef(0); 
// later in the component ... 
passwordRetries.current = 1; // changed from 0 to 1 
// later ... 
console.log(passwordRetries.current); // prints 1, even if the component changed
```

Esta no es una función que se utilice con frecuencia, pero puede resultar útil de vez en cuando. En todos los demás casos, utiliza valores de estado normales.

#### Refs en componentes personalizados
Las Refs no solo se pueden usar para acceder a elementos del DOM. También puedes usarlas para acceder a componentes de React, incluidos tus propios componentes.

Esto a veces puede ser útil. Considera este ejemplo: tienes un componente `<Form>` que contiene un componente `<Preferences>` anidado. Este último componente se encarga de mostrar dos casillas de verificación (*checkboxes*), solicitando al usuario sus preferencias de boletín informativo:

**Figura 7.4**: Un formulario de registro en un boletín informativo que muestra dos casillas de verificación para establecer preferencias.

El código del componente `Preferences` podría verse así:

```javascript
function Preferences() { 
  const [wantsNewProdInfo, setWantsNewProdInfo] = useState(false); 
  const [wantsProdUpdateInfo, setWantsProdUpdateInfo] = useState(false); 

  function handleChangeNewProdPref() { 
    setWantsNewProdInfo((prevPref) => !prevPref); 
  } 

  function handleChangeUpdateProdPref() { 
    setWantsProdUpdateInfo((prevPref) => !prevPref); 
  } 

  return ( 
    <div className={classes.preferences}> 
      <label> 
        <input type="checkbox" id="pref-new" checked={wantsNewProdInfo} onChange={handleChangeNewProdPref} /> 
        <span>New Products</span> 
      </label> 
      <label> 
        <input type="checkbox" id="pref-updates" checked={wantsProdUpdateInfo} onChange={handleChangeUpdateProdPref} /> 
        <span>Product Updates</span> 
      </label> 
    </div> 
  ); 
};
```

Como puedes ver, es un componente básico que esencialmente muestra las dos casillas de verificación, agrega algo de estilo y realiza un seguimiento de la casilla de verificación seleccionada a través del estado.

El código del componente `Form` podría verse así:

```javascript
function Form() { 
  function handleSubmit(event) { 
    event.preventDefault(); 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmit}> 
      <div className={classes.formControl}> 
        <label htmlFor="email">Your email</label> 
        <input type="email" id="email" /> 
      </div> 
      <Preferences /> 
      <button>Submit</button> 
    </form> 
  ); 
}
```

Ahora imagina que, al enviar el formulario (dentro de la función `handleSubmit`), se deben restablecer las preferencias (es decir, que ya no se seleccione ninguna casilla de verificación). Además, antes de restablecer, los valores seleccionados deben leerse y usarse en la función `handleSubmit`.

Esto sería sencillo si las casillas de verificación no se hubieran colocado en un componente separado. Si todo el código y el marcado JSX residieran en el componente `Form`, se podría usar el estado en ese componente para leer y cambiar los valores. Pero este no es el caso en este ejemplo, y reescribir el código solo por este problema parece una restricción innecesaria.

Afortunadamente, las Refs pueden ayudar en esta situación.

Puedes **exponer funcionalidades (por ejemplo, funciones o valores de estado) de un componente a otros componentes a través de Refs**. Las Refs se pueden usar esencialmente como un dispositivo de comunicación entre dos componentes, tal como se usaron como un dispositivo de comunicación con un elemento del DOM en las secciones anteriores.

Convenientemente, tus componentes personalizados pueden recibir una `ref` como una prop regular (en **React 19 o superior**):

```javascript
function Preferences(props) { 
  // or function Preferences({ ref }) {} 
  // can use props.ref in here 
  // component code ... 
}; 

export default Preferences;
```

Por lo tanto, podrías usar este componente `Preferences` y pasarle una `ref`:

```javascript
function Form() { 
  const preferencesRef = useRef(null); 
  return <Preferences ref={preferencesRef} />; 
}
```

Es importante tener en cuenta que este código solo funciona cuando se utiliza React 19 o superior. Cuando se trabaja con una versión anterior de React, desafortunadamente no se admite pasar Refs como props regulares a los componentes. En tales proyectos, tendrías que envolver la función del componente que debe recibir una Ref con una función especial llamada **`forwardRef()`** que proporciona React.

Por lo tanto, en proyectos de React que utilicen React 18 o versiones anteriores, para recibir y usar Refs, debes envolver el componente receptor (`Preferences`, en este ejemplo) con `forwardRef()`.

Esto se puede hacer de la siguiente manera:

```javascript
const Preferences = forwardRef((props, ref) => { 
  // component code ... 
}); 

export default Preferences;
```

Esto se ve ligeramente diferente a todos los demás componentes de este libro porque se usa una función flecha en lugar de la palabra clave `function`. Siempre puedes usar funciones flecha en lugar de "funciones normales", pero aquí es útil cambiar ya que hace que envolver la función con `forwardRef()` sea muy fácil. Alternativamente, podrías mantener la palabra clave `function` y envolver la función de esta manera:

```javascript
function Preferences(props, ref) { 
  // component code ... 
}; 

export default forwardRef(Preferences);
```

Depende de ti qué sintaxis prefieres. Ambas funcionan y ambas se usan comúnmente en proyectos de React.

La parte interesante de este código es que la función del componente ahora recibe dos parámetros en lugar de uno. Además de recibir props (lo que las funciones de los componentes siempre hacen), ahora también recibe un parámetro especial `ref`. Y este parámetro solo se recibe porque la función del componente está envuelta con `forwardRef()`.

Este parámetro `ref` contendrá cualquier valor de ref establecido por el componente que utilice el componente `Preferences`. Por ejemplo, el componente `Form` podría establecer un parámetro de referencia en `Preferences` de esta manera:

```javascript
function Form() { 
  const preferencesRef = useRef({}); 

  function handleSubmit(event) { 
    // other code ... 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmit}> 
      <div className={classes.formControl}> 
        <label htmlFor="email">Your email</label> 
        <input type="email" id="email" /> 
      </div> 
      <Preferences ref={preferencesRef} /> 
      <button>Submit</button> 
    </form> 
  ); 
}
```

Nuevamente, se usa `useRef()` para crear un objeto ref (`preferencesRef`), y ese objeto luego se pasa a través de la prop especial `ref` al componente `Preferences`. La Ref creada recibe un valor predeterminado de un objeto vacío (`{}`); es a este objeto al que luego se puede acceder a través de `ref.current`. En el componente `Preferences`, el valor de ref se puede recibir y extraer como una prop normal (React >= 19) o se debe acceder a él con la ayuda de la función `forwardRef()` de React. En ese caso, se recibe a través de este segundo parámetro `ref`, que existe debido a `forwardRef()`.

¿Pero cuál es el beneficio de eso? ¿Cómo se puede utilizar ahora este objeto `preferencesRef` dentro de `Preferences` para permitir la interacción entre componentes?

Dado que `ref` es un objeto que nunca se reemplaza, incluso si el componente en el que se creó a través de `useRef()` se reevalúa, el componente receptor puede asignar propiedades y métodos a ese objeto y el componente creador puede usar estos métodos y propiedades. El objeto ref se utiliza, por tanto, como vehículo de comunicación.

En este ejemplo, el componente `Preferences` se podría modificar de esta manera para usar el objeto ref:

```javascript
function Preferences(props) { // wrap with forwardRef() for React < 19 
  const { ref } = props; // Extracting ref prop 
  const [wantsNewProdInfo, setWantsNewProdInfo] = useState(false); 
  const [wantsProdUpdateInfo, setWantsProdUpdateInfo] = useState(false); 

  function handleChangeNewProdPref () { 
    setWantsNewProdInfo((prevPref) => !prevPref); 
  } 

  function handleChangeUpdateProdPref() { 
    setWantsProdUpdateInfo((prevPref) => !prevPref); 
  } 

  function reset() { 
    setWantsNewProdInfo(false); 
    setWantsProdUpdateInfo(false); 
  } 

  ref.current.reset = reset; 
  ref.current.selectedPreferences = { 
    newProductInfo: wantsNewProdInfo, 
    productUpdateInfo: wantsProdUpdateInfo, 
  }; 

  // also return JSX code (has not changed) ... 
};
```

En `Preferences`, tanto los valores de estado como un puntero a una función `reset` recién agregada se almacenan en el objeto ref recibido. Se utiliza `ref.current` ya que el objeto creado por React (al usar `useRef()`) siempre tiene dicha propiedad `current`, y esa propiedad debe usarse para almacenar los valores reales en `ref`.

Dado que `Preferences` y `Form` operan sobre el mismo objeto que está almacenado en el objeto ref, las propiedades y métodos asignados al objeto en `Preferences` también se pueden usar en `Form`:

```javascript
function Form() { 
  const preferencesRef = useRef({}); 

  function handleSubmit(event) { 
    event.preventDefault(); 
    console.log(preferencesRef.current.selectedPreferences); // reading a value 
    preferencesRef.current.reset(); // executing a function stored in Preferences 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmit}> 
      <div className={classes.formControl}> 
        <label htmlFor="email">Your email</label> 
        <input type="email" id="email" /> 
      </div> 
      <Preferences ref={preferencesRef} /> 
      <button>Submit</button> 
    </form> 
  ); 
}
```

Al usar Refs de esta manera, un componente principal (`Form`, en este caso) puede interactuar con algún componente secundario (por ejemplo, `Preferences`) de una **manera imperativa**, lo que significa que se puede acceder a las propiedades y llamar a métodos para manipular el componente secundario (o, para ser precisos, para activar algunas funciones y comportamientos internos dentro del componente secundario).

> [!NOTE]
> React también proporciona un Hook `useImperativeHandle()` que se puede utilizar para exponer datos o funciones desde componentes personalizados.
> Técnicamente, no necesitas usar este Hook, como demuestran los ejemplos anteriores: puedes comunicarte entre componentes a través de Refs sin ningún Hook adicional.
> Sin embargo, es posible que desees considerar el uso de `useImperativeHandle()` ya que manejará escenarios como valores de ref faltantes (es decir, si no se proporciona ningún valor de ref). Puedes obtener más información sobre el uso de este Hook en la documentación oficial: [https://react.dev/reference/react/useImperativeHandle](https://react.dev/reference/react/useImperativeHandle).

#### Componentes controlados frente a no controlados
Pasar Refs a componentes personalizados (a través de props o `forwardRef()`) es un método que se puede utilizar para permitir que los componentes `Form` y `Preferences` trabajen juntos. Pero aunque al principio pueda parecer una solución elegante, normalmente **no debería ser tu solución predeterminada** para este tipo de problema.

El uso de Refs, como se muestra en el ejemplo anterior, genera al final **código más imperativo**. Es código imperativo porque en lugar de definir el estado deseado de la interfaz de usuario a través de JSX (lo cual sería declarativo), se agregan instrucciones individuales paso a paso en JavaScript.

Si revisas el Capítulo 1, *React – Qué es y por qué* (la sección *El problema de JavaScript puro*), verás que código como `preferencesRef.current.reset()` (del ejemplo anterior) se parece mucho a instrucciones como `buttonElement.addEventListener(…)` (ejemplo del Capítulo 1). Ambos ejemplos utilizan código imperativo y deben evitarse por las razones mencionadas en el Capítulo 1 (escribir instrucciones paso a paso conduce a una microgestión ineficiente y a menudo a un código innecesariamente complejo).

Dentro del componente `Form`, se invoca la función `reset()` de `Preferences`. Por lo tanto, el código describe la acción deseada que se debe realizar (en lugar del resultado esperado). Normalmente, al trabajar con React debes esforzarte por describir el estado (de la interfaz de usuario) deseado en su lugar. Recuerda que al trabajar con React debes escribir código declarativo, en lugar de código imperativo.

Cuando usas Refs para leer o manipular datos como se muestra en las secciones anteriores de este capítulo, estás construyendo los llamados **componentes no controlados** (*uncontrolled components*). Los componentes se consideran "no controlados" porque React no controla directamente el estado de la interfaz de usuario. En su lugar, los valores se leen de otros componentes o del DOM. Por lo tanto, es el DOM el que controla el estado (por ejemplo, un estado como el valor ingresado por un usuario en un campo de entrada).

Como desarrollador de React, debes intentar minimizar el uso de componentes no controlados. Está perfectamente bien usar Refs para ahorrar algo de código si solo necesitas recopilar algunos valores ingresados. Pero tan pronto como la lógica de tu interfaz de usuario se vuelva más compleja (por ejemplo, si también deseas borrar la entrada del usuario), deberías optar por **componentes controlados** (*controlled components*).

Hacerlo es bastante sencillo: **un componente pasa a estar controlado tan pronto como React gestiona el estado**. En el caso del componente `EmailForm` del comienzo de este capítulo, el enfoque del componente controlado se mostró antes de que se introdujeran las Refs. El uso de `useState()` para almacenar la entrada del usuario (y actualizar el estado con cada pulsación de tecla) significaba que React tenía el control total del valor introducido.

Para el ejemplo anterior, los componentes `Form` y `Preferences`, cambiar a un enfoque de componente controlado podría verse así:

```javascript
function Preferences({newProdInfo, prodUpdateInfo, onUpdateInfo}) { 
  return ( 
    <div className={classes.preferences}> 
      <label> 
        <input type="checkbox" id="pref-new" checked={newProdInfo} onChange={onUpdateInfo.bind(null, 'pref-new')} /> 
        <span>New Products</span> 
      </label> 
      <label> 
        <input type="checkbox" id="pref-updates" checked={prodUpdateInfo} onChange={onUpdateInfo.bind(null, 'pref-updates')} /> 
        <span>Product Updates</span> 
      </label> 
    </div> 
  ); 
};
```

En este ejemplo, el componente `Preferences` deja de administrar el estado de las casillas de verificación y, en su lugar, recibe props de su componente principal (el componente `Form`).

Se utiliza `bind()` en la prop `onUpdateInfo` (que recibirá una función como valor) para preconfigurar la función para su ejecución futura. `bind()` es un método predeterminado de JavaScript al que se puede llamar en cualquier función de JavaScript para controlar qué argumentos se pasarán a esa función una vez que se invoque en el futuro.

> [!NOTE]
> Puedes obtener más información sobre esta función de JavaScript en [https://academind.com/tutorials/function-bind-event-execution](https://academind.com/tutorials/function-bind-event-execution).

El componente `Form` ahora gestiona los estados de las casillas de verificación, aunque no contenga directamente los elementos de las casillas de verificación. Pero ahora comienza a controlar el componente `Preferences` y su estado interno, convirtiendo a `Preferences` en un componente controlado en lugar de uno no controlado:

```javascript
function Form() { 
  const [wantsNewProdInfo, setWantsNewProdInfo] = useState(false); 
  const [wantsProdUpdateInfo, setWantsProdUpdateInfo] = useState(false); 

  function handleUpdateProdInfo(selection) { 
    // using one shared update handler function is optional 
    // you could also use two separate functions (passed to Preferences) as props 
    if (selection === 'pref-new') { 
      setWantsNewProdInfo((prevPref) => !prevPref); 
    } else if (selection === 'pref-updates') { 
      setWantsProdUpdateInfo((prevPref) => !prevPref); 
    } 
  } 

  function reset() { 
    setWantsNewProdInfo(false); 
    setWantsProdUpdateInfo(false); 
  } 

  function handleSubmit(event) { 
    event.preventDefault(); 
    // state values can be used here 
    reset(); 
  } 

  return ( 
    <form className={classes.form} onSubmit={handleSubmit}> 
      <div className={classes.formControl}> 
        <label htmlFor="email">Your email</label> 
        <input type="email" id="email" /> 
      </div> 
      <Preferences newProdInfo={wantsNewProdInfo} prodUpdateInfo={wantsProdUpdateInfo} onUpdateInfo={handleUpdateProdInfo} /> 
      <button>Submit</button> 
    </form> 
  ); 
}
```

`Form` gestiona el estado de selección de las casillas de verificación, incluido el restablecimiento del estado a través de la función `reset()`, y pasa los valores de estado gestionados (`wantsNewProdInfo` y `wantsProdUpdateInfo`), así como la función `handleUpdateProdInfo` que actualiza los valores de estado, a `Preferences`. El componente `Form` ahora controla el componente `Preferences`.

Si revisas los dos fragmentos de código anteriores, notarás que el código final vuelve a ser puramente declarativo. En todos los componentes, el estado se administra y se usa para declarar la interfaz de usuario esperada.

Se considera una buena práctica optar por componentes controlados en la mayoría de los casos. Sin embargo, si solo estás extrayendo algunos valores ingresados por el usuario, usar Refs y crear un componente no controlado es absolutamente aceptable.

---

### Sección 5: React y dónde terminan los elementos en el DOM

Dejando de lado el tema de las Refs, existe otra característica importante de React que puede ayudar a influir en la interacción (indirecta) con el DOM: los **Portales (*Portals*)**.

Al construir interfaces de usuario, a veces es necesario mostrar elementos y contenido de forma condicional. Esto ya se cubrió en el Capítulo 5, *Renderizado de Listas y Contenido Condicional*. Al renderizar contenido condicional, React inyectará ese contenido en el lugar del DOM donde se encuentra el componente general (en el que está definido el contenido condicional).

Por ejemplo, al mostrar un mensaje de error condicional debajo de un campo de entrada, ese mensaje de error se encuentra justo debajo del campo de entrada en el DOM:

**Figura 7.5**: El elemento de mensaje de error del DOM se ubica justo debajo del `<input>` al que pertenece.

Este comportamiento tiene sentido. De hecho, sería bastante desconcertante que React comenzara a insertar elementos del DOM en lugares aleatorios. Pero en algunos escenarios, es posible que prefieras que un elemento (condicional) del DOM se inserte en un lugar diferente en el DOM; por ejemplo, cuando trabajas con elementos superpuestos como los diálogos de error (*error dialogs*).

En el ejemplo anterior, podrías agregar lógica para garantizar que se presente un cuadro de diálogo de error al usuario si el formulario se envía con una dirección de correo electrónico no válida. Esto se podría implementar con una lógica similar al mensaje de error "Invalid email address!", y por lo tanto, el elemento de diálogo también se inyectaría dinámicamente en el DOM:

**Figura 7.6**: El diálogo de error y su fondo (*backdrop*) se inyectan en el DOM.

En esta captura de pantalla, el cuadro de diálogo de error se abre como una superposición sobre un elemento de fondo (*backdrop*), que a su vez se agrega para que actúe como una superposición sobre el resto de la interfaz de usuario.

> [!NOTE]
> La apariencia se maneja completamente mediante CSS, y puedes ver el proyecto completo (incluidos los estilos) aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/examples/05-portals-problem](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/examples/05-portals-problem).

Este ejemplo funciona y se ve bien. Sin embargo, hay margen de mejora.

Semánticamente, no tiene mucho sentido que los elementos de superposición se inyecten en algún lugar anidado en el DOM junto al elemento `<input>`. Tendría más sentido que los elementos superpuestos estuvieran más cerca de la raíz del DOM (en otras palabras, que fueran elementos secundarios directos de `<div id="root">` o incluso de `<body>`), en lugar de ser elementos secundarios de `<form>`. Y no es solo un problema semántico: si la aplicación de ejemplo contiene otros elementos superpuestos, esos elementos podrían entrar en conflicto entre sí, de la siguiente manera:

**Figura 7.7**: El elemento `<footer>` en la parte inferior es visible por encima del fondo.

En este ejemplo, el elemento `<footer>` en la parte inferior ("An example project") no queda oculto ni atenuado por el fondo (*backdrop*) que pertenece al diálogo de error. La razón de esto es que el pie de página también tiene algún estilo CSS adjunto que lo convierte en una superposición de facto (debido al uso de `position: fixed` y `left` + `bottom` en sus estilos CSS).

Como solución a este problema, podrías modificar algunos estilos CSS y, por ejemplo, usar la propiedad CSS `z-index` para controlar los niveles de superposición. Sin embargo, sería una solución más limpia si los elementos superpuestos (es decir, el fondo `<div>` y los elementos de error `<dialog>`) se insertaran en el DOM en un lugar diferente, por ejemplo, al final del elemento `<body>` (pero como hijos directos de `<body>`).

Y ese es exactamente el tipo de problema que los **React Portals** te ayudan a resolver.

#### Portales al rescate
Un Portal, en el mundo de React, es una característica que te permite **indicarle a React que inserte un elemento del DOM en un lugar diferente de donde normalmente se insertaría**.

Teniendo en cuenta el ejemplo mostrado anteriormente, esta función de portal se puede utilizar para indicarle a React que no inserte el error `<dialog>` y el fondo `<div>` que pertenece al diálogo dentro del elemento `<form>`, sino que inserte esos elementos al final del elemento `<body>`.

Para utilizar esta característica de portal, primero debes definir un lugar en el que se puedan insertar los elementos (un "gancho de inyección" o *injection hook*). Esto se puede hacer en el archivo HTML que pertenece a la aplicación React (es decir, `index.html`). Allí, puedes agregar un nuevo elemento (por ejemplo, un elemento `<div>`) en algún lugar del elemento `<body>`:

```html
<body> 
  <div id="root"></div> 
  <div id="dialogs"></div> 
  <script type="module" src="/src/main.jsx"></script> 
</body>
```

En este caso, se agrega un elemento `<div id="dialogs">` en la sección `<body>`, después del elemento `<div id="root">` para garantizar que los componentes (y sus estilos) insertados en ese elemento se evalúen al final. Esto garantizará que sus estilos tengan una mayor prioridad y que los elementos superpuestos insertados en `<div id="dialogs">` no queden tapados por otro contenido anterior en el DOM. Sería posible agregar y usar múltiples puntos de inyección, pero para este ejemplo, solo se necesita uno. También puedes utilizar elementos HTML distintos de los elementos `<div>`.

Con el archivo `index.html` ajustado, se le puede indicar a React que renderice ciertos elementos JSX (es decir, componentes) en un punto de inyección específico a través de la función **`createPortal()`** de `react-dom`:

```javascript
import { createPortal } from 'react-dom'; 
import classes from './ErrorDialog.module.css'; 

function ErrorDialog({ onClose }) { 
  return createPortal( 
    <> 
      <div className={classes.backdrop}></div> 
      <dialog className={classes.dialog} open> 
        <p> 
          This form contains invalid values. Please fix those errors before submitting the form again! 
        </p> 
        <button onClick={onClose}>Okay</button> 
      </dialog> 
    </>, 
    document.getElementById('dialogs') 
  ); 
} 

export default ErrorDialog;
```

Dentro de este componente `ErrorDialog`, que se renderiza condicionalmente por otro componente (el componente `EmailForm`, cuyo código de ejemplo está disponible en GitHub), el código JSX devuelto se envuelve mediante `createPortal()`. `createPortal()` toma dos argumentos:
1. El código JSX que debe renderizarse en el DOM.
2. Un puntero al elemento en `index.html` donde se debe inyectar el contenido.

En este ejemplo, el `<div id="dialogs">` recién agregado se selecciona a través de `document.getElementById('dialogs')`. Por lo tanto, `createPortal()` garantiza que el código JSX generado por `ErrorDialog` se renderice en ese lugar del documento HTML:

**Figura 7.8**: Los elementos superpuestos se insertan en `<div id="dialogs">`.

En esta captura de pantalla, puedes ver que los elementos superpuestos (el fondo `<div>` y el `<dialog>` de error) efectivamente se insertan en el elemento `<div id="dialogs">`, en lugar de en el elemento `<form>` (como ocurría antes).

Como resultado de este cambio, el `<footer>` ya no se superpone al fondo del cuadro de diálogo de error sin necesidad de realizar ningún cambio en el código CSS. Semánticamente, la estructura final del DOM también tiene más sentido, ya que normalmente esperarías que los elementos de superposición estén más cerca del nodo raíz del DOM.

Aun así, usar esta función de portal es opcional. El mismo resultado visual (aunque no la estructura del DOM) se podría haber logrado cambiando algunos estilos CSS. No obstante, aspirar a una estructura del DOM limpia es un objetivo que vale la pena, y evitar código CSS innecesariamente complejo tampoco es una mala práctica.

---

### Sección 6: Resumen y puntos clave

- Las **Refs** se pueden utilizar para obtener acceso directo a elementos del DOM o para almacenar valores que no se restablecerán ni cambiarán cuando se reevalúe el componente circundante.
- Utiliza este acceso directo solo para leer valores, no para manipular elementos del DOM (deja que React haga esto en su lugar).
- Los componentes que obtienen acceso al DOM a través de Refs, en lugar del estado y otras características de React, se consideran **componentes no controlados** (porque React no tiene el control directo).
- Prefiere **componentes controlados** (usando estado y un enfoque estrictamente declarativo) sobre componentes no controlados a menos que estés realizando tareas muy simples como leer un valor de entrada introducido.
- Usando Refs, también puedes exponer características de tus propios componentes para que puedan ser utilizadas imperativamente.
- Puedes establecer y usar una prop `ref` en componentes personalizados cuando trabajas con **React 19 o superior**.
- Cuando utilices React < 19, se debe utilizar la función `forwardRef()` de React para recibir Refs en componentes personalizados.
- Los **Portales** se pueden utilizar para indicarle a React que renderice elementos JSX en un lugar diferente del DOM de donde normalmente lo haría.

---

### Sección 7: ¿Qué sigue?

En este punto del libro, has conocido muchas herramientas y conceptos clave que se pueden utilizar para crear interfaces de usuario interactivas y atractivas. Gracias a las Refs, puedes leer valores del DOM sin usar estado (evitando así reevaluaciones innecesarias de componentes) o gestionar valores que persisten a través de las actualizaciones de componentes. Gracias a los Portales, puedes controlar exactamente dónde se inserta el marcado del componente en el DOM.

Como resultado, obtienes nuevas herramientas que se pueden utilizar para ajustar con precisión tu aplicación de React. Es posible que puedas mejorar el rendimiento (evitando reevaluaciones de componentes) o mejorar la estructura y la semántica de tus elementos del DOM. En última instancia, es la combinación de todas estas herramientas lo que te permite crear aplicaciones web atractivas, interactivas y con un buen rendimiento con React.

Pero, como aprenderás en el próximo capítulo, React tiene aún más conceptos básicos útiles que ofrecer: por ejemplo, una forma de manejar **efectos secundarios** (*side effects*).

El próximo capítulo explorará qué son exactamente los efectos secundarios, por qué necesitan un manejo especial y cómo React te ayuda con eso.

---

### Sección 8: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/07-portals-refs/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/07-portals-refs/exercises/questions-answers.md).

1. ¿Cómo pueden ayudar las Refs con el manejo de la entrada del usuario en formularios?
2. ¿Qué es un componente no controlado?
3. ¿Qué es un componente controlado?
4. ¿Cuándo no deberías usar Refs?
5. ¿Cuál es la idea principal detrás de los portales?

---

### Sección 9: Aplica lo aprendido

Con este conocimiento recién adquirido sobre Refs y Portales, es nuevamente momento de practicar lo que has aprendido.

A continuación, encontrarás dos actividades que te permitirán practicar el trabajo con Refs y Portales. Como siempre, por supuesto, también necesitarás algunos de los conceptos cubiertos en capítulos anteriores (por ejemplo, trabajar con estado).

#### Actividad 7.1: Extraer valores de entrada del usuario
En esta actividad, debes agregar lógica a un componente de React existente para extraer valores de un formulario. El formulario contiene un campo de entrada y un menú desplegable, y debes asegurarte de que, tras el envío del formulario, ambos valores se lean y, para el propósito de esta aplicación de demostración, se muestren en la consola del navegador.

Utiliza tus conocimientos sobre Refs y componentes no controlados para implementar una solución sin usar el estado de React.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-1-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start` en este caso) para usar la versión correcta del código.

Después de descargar el código y ejecutar `npm install` en la carpeta del proyecto (para instalar todas las dependencias requeridas), los pasos de la solución son los siguientes:
1. Crea dos Refs, una para cada elemento de entrada que deba leerse (campo de entrada y menú desplegable).
2. Conecta las Refs a los elementos de entrada.
3. En la función controladora de envío, accede a los elementos del DOM conectados a través de las Refs y lee los valores actualmente ingresados o seleccionados.
4. Muestra los valores en la consola del navegador.

El resultado esperado (interfaz de usuario) debería verse así:

**Figura 7.9**: La consola de las herramientas de desarrollo del navegador muestra los valores seleccionados.

> [!NOTE]
> Encontrarás todos los archivos de código utilizados para esta actividad, así como la solución, en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-1).

#### Actividad 7.2: Agregar un menú lateral (*Side Drawer*)
En esta actividad, conectarás un componente `SideDrawer` ya existente con un botón en la barra de navegación principal para abrir el menú lateral (es decir, mostrarlo) cada vez que se haga clic en el botón. Después de que se abra el menú lateral, un clic en el fondo (*backdrop*) debería cerrar el menú nuevamente.

Además de implementar la lógica general descrita anteriormente, tu objetivo será garantizar un posicionamiento adecuado en el DOM final para que ningún otro elemento se superponga sobre el `SideDrawer` (sin editar ningún código CSS). El `SideDrawer` tampoco debe estar anidado en ningún otro componente o elemento JSX.

> [!NOTE]
> Esta actividad incluye código inicial, que se puede encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-2-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-2-start).

Después de descargar el código y ejecutar `npm install` para instalar todas las dependencias requeridas, los pasos de la solución son los siguientes:
1. Agrega lógica para mostrar u ocultar condicionalmente el componente `SideDrawer` en el componente `MainNavigation`.
2. Agrega un gancho de inyección (*injection hook*) para el menú lateral en el documento HTML.
3. Utiliza la función de portal de React para renderizar los elementos JSX de `SideDrawer` en el gancho recién agregado.

La interfaz de usuario final debería verse y comportarse de la siguiente manera:

**Figura 7.10**: Un clic en el botón de menú abre el menú lateral.

Al hacer clic en el botón del menú, se abre el menú lateral. Si se hace clic en el fondo (*backdrop*) detrás del menú lateral, debería cerrarse nuevamente.

La estructura final del DOM (con el menú lateral abierto) debería verse así:

**Figura 7.11**: Los elementos relacionados con el menú lateral se insertan en un lugar separado en el DOM.

Los elementos del DOM relacionados con el menú lateral (el `<div>` de fondo y `<aside>`) se insertan en un nodo del DOM separado (`<div id="drawer">`).

> [!NOTE]
> Encontrarás todos los archivos de código utilizados para esta actividad, así como la solución, en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-2](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/07-portals-refs/activities/practice-2).
