# Parte 1: Fundamentos de React

## Capítulo 4: Trabajando con Eventos y Estado

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Agregar controladores de eventos de usuario (por ejemplo, para reaccionar a clics de botones) en aplicaciones de React.
- Actualizar la interfaz de usuario (UI) mediante un concepto llamado **estado** (*state*).
- Construir interfaces de usuario dinámicas e interactivas reales (es decir, para que ya no sean estáticas).

---

### Sección 1: Introducción

En los capítulos anteriores, aprendiste cómo construir interfaces de usuario con la ayuda de componentes de React. También aprendiste sobre las **props**: un concepto y una funcionalidad que permite a los desarrolladores de React construir y reutilizar componentes configurables.

Estas son características y bloques de construcción importantes de React, pero solo con estas funcionalidades solo podrías construir aplicaciones estáticas de React (es decir, aplicaciones web que nunca cambian). No podrías cambiar ni actualizar el contenido en la pantalla si solo tuvieras acceso a esas características. Tampoco podrías reaccionar a ningún evento del usuario y actualizar la interfaz de usuario en respuesta a tales eventos (por ejemplo, para mostrar una ventana emergente o modal al hacer clic en un botón).

Dicho de otra manera, no podrías construir sitios web y aplicaciones web reales si estuvieras limitado solo a componentes y props.

Por lo tanto, en este capítulo se introduce un concepto completamente nuevo: el **estado** (*state*). El estado es una funcionalidad de React que permite a los desarrolladores actualizar datos internos y desencadenar una actualización de la interfaz de usuario basada en dichos ajustes de datos. Además, aprenderás a reaccionar a eventos de los usuarios, como clics en botones o texto introducido en campos de entrada.

---

### Sección 2: ¿Cuál es el problema?

Como se describió anteriormente, en este punto del libro hay un problema con todas las aplicaciones y sitios de React que podrías estar construyendo: son estáticos. La interfaz de usuario no puede cambiar.

Para comprender un poco mejor este problema, observa un componente típico de React, tal como eres capaz de construirlo hasta este punto del libro:

```javascript
function EmailInput() { 
  return ( 
    <div> 
      <input placeholder="Your email" type="email" /> 
      <p>The entered email address is invalid.</p> 
    </div> 
  ); 
};
```

Este componente puede parecer extraño. ¿Por qué hay un elemento `<p>` que informa al usuario sobre una dirección de correo electrónico incorrecta?

Bueno, el objetivo podría ser mostrar ese párrafo solo si el usuario introdujo una dirección de correo electrónico incorrecta. Es decir, la aplicación web debe esperar a que el usuario comience a escribir y evaluar la entrada del usuario una vez que haya terminado de escribir (es decir, una vez que el campo de entrada pierda el foco). Luego, se debe mostrar el mensaje de error si la dirección de correo electrónico se considera no válida (por ejemplo, un campo de entrada vacío o la falta del símbolo `@`).

Pero en este momento, con las habilidades de React adquiridas hasta ahora, esto es algo que no podrías construir. En su lugar, el mensaje de error siempre se mostraría ya que no hay forma de cambiarlo en función de eventos del usuario y condiciones dinámicas. En otras palabras, esta aplicación de React es estática, no dinámica. La interfaz de usuario no puede cambiar.

Por supuesto, las interfaces de usuario cambiantes y las aplicaciones web dinámicas son cosas que querrás construir. Casi todos los sitios web existentes contienen elementos y funciones dinámicas en su interfaz de usuario. Por lo tanto, ese es el problema que se resolverá en este capítulo.

#### Cómo no resolver el problema
¿Cómo se podría hacer más dinámico el componente mostrado anteriormente?

La siguiente es una solución que se te podría ocurrir (spoiler: el código no funcionará, así que no necesitas intentar ejecutarlo):

```javascript
function EmailInput() { 
  return ( 
    <div> 
      <input placeholder="Your email" type="email" /> 
      <p></p> 
    </div> 
  ); 
}; 

const input = document.querySelector('input'); 
const errorParagraph = document.querySelector('p'); 

function evaluateEmail(event) { 
  const enteredEmail = event.target.value; 
  if (enteredEmail.trim() === '' || !enteredEmail.includes('@')) { 
    errorParagraph.textContent = ' The entered email address is invalid.'; 
  } else { 
    errorParagraph.textContent = ''; 
  } 
}; 

input.addEventListener('blur', evaluateEmail);
```

Este código no funcionará porque no puedes seleccionar elementos del DOM renderizados por React desde el interior del mismo archivo de componente de esta manera. Esto es solo un ejemplo simulado de cómo podrías intentar resolverlo. Dicho esto, podrías colocar el código debajo de la función del componente en algún lugar donde se ejecute correctamente (por ejemplo, dentro de una devolución de llamada `setTimeout()` que se active después de un segundo, permitiendo que la aplicación de React renderice todos los elementos en la pantalla).

Ubicado en el lugar correcto, este código agregará el comportamiento de validación de correo electrónico descrito anteriormente en este capítulo. Tras el evento integrado `blur`, se activa la función `evaluateEmail`. Esta función recibe el objeto de evento como argumento (automáticamente, por el navegador) y, por lo tanto, la función `evaluateEmail` puede obtener el valor introducido a partir de ese objeto de evento mediante `event.target.value`. El valor introducido se puede usar luego en una comprobación `if` para mostrar o eliminar condicionalmente el mensaje de error.

> [!NOTE]
> Todo el código anterior que trata sobre el evento `blur` (como `addEventListener`) y el objeto de evento, incluido el código en la comprobación `if`, es código JavaScript estándar. No es específico de React de ninguna manera.
> Si tienes dificultades con este código que no es de React, se recomienda encarecidamente que consultes primero más recursos sobre JavaScript puro (como las guías en el sitio web de MDN en [https://developer.mozilla.org/en-US/docs/Web/JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)).

¿Pero qué tiene de malo este código si funcionara en algunos lugares del código general de la aplicación?

**¡Es código imperativo!** Eso significa que estás escribiendo instrucciones paso a paso sobre lo que debe hacer el navegador. No estás declarando el estado final deseado; en su lugar, estás describiendo una forma de llegar allí; y no estás utilizando React.

Ten en cuenta que React se trata de controlar la interfaz de usuario y que escribir código de React se trata de escribir código **declarativo**, en lugar de código imperativo. Vuelve a revisar el Capítulo 2, *Entendiendo los Componentes de React y JSX*, si eso te suena completamente nuevo.

Podrías lograr tu objetivo introduciendo este tipo de código, pero estarías trabajando en contra de React y su filosofía (la filosofía de React es que declaras tus estados finales deseados y dejas que React descubra cómo llegar allí). Un claro indicador de esto es el hecho de que te verías obligado a encontrar el lugar adecuado para este tipo de código para que funcione.

Este no es un problema filosófico y no es solo una regla rígida y extraña que debas seguir. En cambio, al trabajar contra React de esta manera, te harás la vida innecesariamente difícil como desarrollador. No estás utilizando las herramientas que te proporciona React ni estás dejando que React descubra cómo lograr el estado deseado de la interfaz de usuario.

Eso no solo significa que dedicas tiempo a resolver problemas que no tendrías que resolver; también significa que estás desaprovechando posibles optimizaciones que React podría realizar internamente. Es muy probable que tu solución no solo te genere más trabajo (es decir, más código), sino que también dé lugar a un resultado con errores y un rendimiento subóptimo.

El ejemplo mostrado anteriormente es simple. Piensa en sitios web y aplicaciones web más complejos, como tiendas en línea, sitios de alquiler vacacional o aplicaciones web como Google Docs. Allí, podrías tener docenas o cientos de funciones y elementos dinámicos en la interfaz de usuario. Administrarlos todos con una mezcla de código React y código JavaScript estándar se convertirá rápidamente en una pesadilla.

#### Una solución incorrecta pero mejor
El enfoque ingenuo discutido anteriormente no funciona bien. Te obliga a descubrir cómo hacer que el código se ejecute correctamente (por ejemplo, envolviendo partes de él en alguna llamada a `setTimeout()` para retrasar la ejecución) y hace que tu código quede disperso por todas partes (es decir, dentro de las funciones de los componentes de React, fuera de esas funciones y tal vez también en archivos totalmente no relacionados). ¿Qué tal una solución que adopte React, como esta?:

```javascript
function EmailInput() { 
  let errorMessage = ''; 

  function evaluateEmail(event) { 
    const enteredEmail = event.target.value; 
    if (enteredEmail.trim() === '' || !enteredEmail.includes('@')) { 
      errorMessage = ' The entered email address is invalid.'; 
    } else { 
      errorMessage = ''; 
    } 
  }; 

  const input = document.querySelector('input'); 
  input.addEventListener('blur', evaluateEmail); 

  return ( 
    <div> 
      <input placeholder="Your email" type="email" /> 
      <p>{errorMessage}</p> 
    </div> 
  ); 
};
```

Este código nuevamente no funcionaría (aunque técnicamente sea código JavaScript válido). La selección de elementos JSX no funciona de esta manera. No funciona porque `document.querySelector('input')` se ejecuta antes de que se renderice nada en el DOM (cuando la función del componente se ejecuta por primera vez). Nuevamente, tendrías que retrasar la ejecución de ese código hasta que termine el primer ciclo de renderizado (por lo tanto, estarías trabajando una vez más en contra de React).

Pero aunque todavía no funcione, está más cerca de la solución correcta.

Está más cerca de la implementación ideal porque adopta React mucho más de lo que lo hizo el primer intento de solución. Todo el código está contenido en la función del componente al que pertenece. El mensaje de error se maneja mediante una variable `errorMessage` que se muestra como parte del código JSX.

La idea detrás de esta posible solución es que el componente de React que controla una determinada funcionalidad o elemento de la interfaz de usuario también es responsable de su estado y sus eventos. ¡Es posible que identifiques aquí dos palabras clave importantes de este capítulo!

Este enfoque definitivamente va en la dirección correcta, pero aún no funcionaría por dos razones:
1. La selección del elemento JSX `<input>` mediante `document.querySelector('input')` fallaría.
2. Incluso si se pudiera seleccionar el campo de entrada, la interfaz de usuario no se actualizaría como se espera.

Estos dos problemas se resolverán a continuación, lo que finalmente conducirá a una implementación que adopta plenamente React y sus características. La próxima solución evitará mezclar código de React y código que no sea de React. Como verás, el resultado será un código más sencillo en el que tendrás que hacer menos trabajo (es decir, escribir menos código).

#### Mejorar la solución reaccionando adecuadamente a los eventos
En lugar de mezclar código imperativo de JavaScript como `document.querySelector('input')` con código específico de React, debes adoptar plenamente React y sus características.

Dado que escuchar eventos y desencadenar acciones en función de ellos es un requisito extremadamente común, React tiene una solución integrada: puedes adjuntar escuchadores de eventos directamente a los elementos JSX a los que pertenecen.

El ejemplo anterior se reescribiría de la siguiente manera:

```javascript
function EmailInput() { 
  let errorMessage = ''; 

  function evaluateEmail(event) { 
    const enteredEmail = event.target.value; 
    if (enteredEmail.trim() === '' || !enteredEmail.includes('@')) { 
      errorMessage = 'The entered email address is invalid.'; 
    } else { 
      errorMessage = ''; 
    } 
  }; 

  return ( 
    <div> 
      <input placeholder="Your email" type="email" onBlur={evaluateEmail} /> 
      <p>{errorMessage}</p> 
    </div> 
  ); 
};
```

Este código todavía no actualizará la interfaz de usuario, pero al menos el evento se maneja correctamente.

Se agregó la prop `onBlur` al elemento `input` integrado. Esta prop la pone a disposición React, al igual que todos estos elementos HTML base (como `<input>` y `<p>`) son puestos a disposición como componentes por React. De hecho, todos estos componentes HTML integrados vienen con sus atributos HTML estándar como props de React (además de algunas props adicionales, como la prop de manejo de eventos `onBlur`).

React expone todos los eventos estándar que se pueden conectar a elementos del DOM como props `onXYZ` (donde `XYZ` es el nombre del evento, como `blur` o `click`, comenzando con un carácter en mayúscula). Puedes reaccionar al evento `blur` agregando la prop `onBlur`. Podrías escuchar un evento de clic a través de la prop `onClick`. Ya captas la idea.

> [!NOTE]
> Para obtener más información sobre los eventos estándar, consulta [https://developer.mozilla.org/en-US/docs/Web/Events#event_listing](https://developer.mozilla.org/en-US/docs/Web/Events#event_listing).

Estas props requieren valores para cumplir su función. Para ser precisos, necesitan un **puntero a la función** que debe ejecutarse cuando ocurra el evento. En el ejemplo anterior, la prop `onBlur` recibe un puntero a la función `evaluateEmail` como valor.

> [!NOTE]
> Hay una diferencia sutil entre `evaluateEmail` y `evaluateEmail()`. El primero es un puntero a la función; el segundo ejecuta la función inmediatamente (y produce el valor devuelto, si lo hay). Nuevamente, esto no es algo específico de React, sino un concepto estándar de JavaScript. Si no está claro, este recurso lo explica con mayor detalle: [https://developer.mozilla.org/en-US/docs/Web/Events#event_listing](https://developer.mozilla.org/en-US/docs/Web/Events#event_listing).

Al utilizar estas props de eventos, el código de ejemplo anterior ahora finalmente se ejecutará sin arrojar ningún error. Podrías verificar esto agregando una declaración `console.log('Hello');` dentro de la función `evaluateEmail`. Esto mostrará el texto `'Hello'` en la consola de las herramientas de desarrollo de tu navegador cada vez que el campo de entrada pierda el foco:

```javascript
function EmailInput() { 
  let errorMessage = ''; 

  function evaluateEmail(event) { 
    console.log('Hello'); 
    const enteredEmail = event.target.value; 
    if (enteredEmail.trim() === '' || !enteredEmail.includes('@')) { 
      errorMessage = 'The entered email address is invalid.'; 
    } else { 
      errorMessage = ''; 
    } 
  }; 

  return ( 
    <div> 
      <input placeholder="Your email" type="email" onBlur={evaluateEmail} /> 
      <p>{errorMessage}</p> 
    </div> 
  ); 
};
```

En la consola del navegador, esto se ve de la siguiente manera:

**Figura 4.1**: Mostrar texto en la consola del navegador al quitar el foco del campo de entrada.

Este es definitivamente un paso más hacia la mejor implementación posible, pero todavía no producirá el resultado deseado de actualizar el contenido de la página de forma dinámica.

---

### Sección 3: Actualizar el estado correctamente

A estas alturas, ya comprendes cómo configurar correctamente los escuchadores de eventos y ejecutar funciones tras determinados eventos. Lo que falta es una funcionalidad que obligue a React a actualizar la interfaz de usuario visible en la pantalla y el contenido que se muestra a los usuarios de la aplicación.

Ahí es donde entra en juego el concepto de **estado** (*state*) de React. Al igual que las props, el estado es un concepto clave de React, pero mientras que las props se tratan de recibir datos externos dentro de un componente, el estado se trata de **administrar y actualizar datos internos**. Lo más importante es que, **siempre que se actualiza el estado, React actualiza las partes de la interfaz de usuario que se ven afectadas por ese cambio de estado**.

Así es como se utiliza el estado en React (por supuesto, el código se explicará en detalle a continuación):

```javascript
import { useState } from 'react'; 

function EmailInput() { 
  const [errorMessage, setErrorMessage] = useState(''); 

  function evaluateEmail(event) { 
    const enteredEmail = event.target.value; 
    if (enteredEmail.trim() === '' || !enteredEmail.includes('@')) { 
      setErrorMessage('The entered email address is invalid.'); 
    } else { 
      setErrorMessage(''); 
    } 
  }; 

  return ( 
    <div> 
      <input placeholder="Your email" type="email" onBlur={evaluateEmail} /> 
      <p>{errorMessage}</p> 
    </div> 
  ); 
};
```

En comparación con el código de ejemplo discutido anteriormente en este capítulo, este código no parece muy diferente. Pero hay una diferencia clave: el uso del Hook `useState()`.

Los **Hooks** son otro concepto clave de React. Son funciones especiales que solo se pueden utilizar dentro de componentes de React (o dentro de otros Hooks personalizados, como se cubrirá en el Capítulo 12, *Creación de Custom Hooks en React*). Los Hooks añaden características y comportamientos especiales a los componentes de React en los que se utilizan. Por ejemplo, el Hook `useState()` permite que un componente (y, por lo tanto, implícitamente React) establezca y gestione un estado que está vinculado a este componente. React proporciona varios Hooks integrados y no todos se centran en la gestión del estado. Aprenderás sobre otros Hooks y sus propósitos a lo largo de este libro.

El Hook `useState()` es un Hook extremadamente importante y de uso común, ya que te permite administrar datos dentro de un componente que, al actualizarse, le indican a React que actualice la interfaz de usuario en consecuencia.

Esa es la idea central detrás de la gestión del estado y este concepto de estado: **el estado son datos que, cuando cambian, deben obligar a React a reevaluar un componente y actualizar la interfaz de usuario si es necesario**.

Usar Hooks como `useState()` es bastante sencillo: los importas desde `'react'` y luego los llamas como una función dentro de la función de tu componente. Los llamas como una función porque, como se mencionó, los Hooks de React son funciones, solo que funciones especiales (desde la perspectiva de React).

#### Una mirada más cercana a useState()
¿Cómo funciona exactamente el Hook `useState()` y qué hace internamente?

Al llamar a `useState()` dentro de una función de componente, registras algunos datos en React. Es un poco como definir una variable o constante en JavaScript puro. Pero hay algo especial: React rastreará el valor registrado internamente y, cada vez que lo actualices, React **reevaluará la función del componente** en la que se registró el estado.

React hace esto comprobando si los datos utilizados en el componente han cambiado. Lo más importante es que React valida si la interfaz de usuario debe cambiar debido a los datos modificados (por ejemplo, porque se muestra un valor dentro del código JSX). Si React determina que la interfaz de usuario debe cambiar, actualiza el DOM real en los lugares donde se necesita una actualización (por ejemplo, cambiando un texto que se muestra en la pantalla). Si no se necesita ninguna actualización, React finaliza la reevaluación del componente sin actualizar el DOM.

El funcionamiento interno de React se analizará en gran detalle en el Capítulo 10, *Tras Bambalinas de React y Oportunidades de Optimización*.

Todo el proceso comienza llamando a `useState()` dentro de un componente. Esto crea un valor de estado (que será almacenado y administrado por React) y lo vincula a un componente específico. Se registra un valor de estado inicial simplemente pasándolo como valor de parámetro a `useState()`. En el ejemplo anterior, se registra una cadena vacía (`''`) como primer valor:

```javascript
const [errorMessage, setErrorMessage] = useState('');
```

Como puedes ver en el ejemplo, `useState()` no solo acepta un valor de parámetro. También devuelve un valor: **un array con exactamente dos elementos**.

El ejemplo anterior utiliza la **desestructuración de arrays**, que es una característica estándar de JavaScript que permite a los desarrolladores recuperar valores de un array y asignarlos inmediatamente a variables o constantes. En el ejemplo, los dos elementos que componen el array devuelto por `useState()` se extraen de ese array y se almacenan en dos constantes (`errorMessage` y `setErrorMessage`). Sin embargo, no estás obligado a utilizar la desestructuración de arrays cuando trabajas con React o `useState()`.

También podrías escribir el código de esta manera:

```javascript
const stateData = useState(''); 
const errorMessage = stateData[0]; 
const setErrorMessage = stateData[1];
```

Esto funciona perfectamente, pero al utilizar la desestructuración de arrays, el código queda un poco más conciso. Es por eso que normalmente ves la sintaxis que utiliza la desestructuración de arrays al explorar aplicaciones y ejemplos de React. Tampoco tienes que usar constantes; las variables (a través de `let`) también estarían bien. Sin embargo, como verás a lo largo de este capítulo y el resto del libro, las variables no se reasignarán directamente, por lo que usar constantes tiene sentido (pero no es obligatorio de ninguna manera).

> [!NOTE]
> Si la desestructuración de arrays o la diferencia entre variables y constantes te resulta completamente nueva, se recomienda encarecidamente que repases los conceptos básicos de JavaScript antes de continuar con este libro (consulta [http://packt.link/3B8Ct](http://packt.link/3B8Ct) para la desestructuración de arrays, [https://packt.link/hGjqL](https://packt.link/hGjqL) para información sobre la variable `let` y [https://packt.link/TdPPS](https://packt.link/TdPPS) para obtener orientación sobre el uso de `const`).

Como se mencionó anteriormente, `useState()` devuelve un array con exactamente dos elementos. Siempre serán exactamente dos elementos y siempre exactamente el mismo tipo de elementos:
1. El primer elemento es siempre el **valor de estado actual**.
2. El segundo elemento es una **función** que puedes llamar para establecer el estado en un nuevo valor.

¿Pero cómo funcionan juntos estos dos valores (el valor de estado y la función de actualización de estado)? ¿Qué hace React con ellos internamente? ¿Cómo utiliza React estos dos elementos del array para actualizar la interfaz de usuario?

#### Una mirada bajo el capó de React
React administra los valores de estado por ti, en un almacenamiento interno al que tú, como desarrollador, no puedes acceder directamente. Dado que a menudo necesitas acceder a un valor de estado (por ejemplo, alguna dirección de correo electrónico introducida, como en el ejemplo anterior), React proporciona una forma de leer los valores de estado: el primer elemento en el array devuelto por `useState()`. El primer elemento del array devuelto contiene el valor del estado actual. Por lo tanto, puedes usar este elemento en cualquier lugar donde necesites trabajar con el valor de estado (por ejemplo, en el código JSX para mostrarlo allí).

Además, a menudo también necesitas actualizar el estado, por ejemplo, porque un usuario introdujo una nueva dirección de correo electrónico. Como tú no administras el valor del estado directamente, React te proporciona una función a la que puedes llamar para informarle sobre el nuevo valor de estado. Ese es el segundo elemento en el array devuelto.

En el ejemplo mostrado anteriormente, llamas a `setErrorMessage('Error!')` para establecer el valor del estado `errorMessage` en una nueva cadena (`'Error!'`).

¿Pero por qué se gestiona así? ¿Por qué no usar simplemente una variable estándar de JavaScript que puedas asignar y reasignar según sea necesario?

Porque **React debe ser informado cada vez que haya un cambio de estado que afecte la interfaz de usuario**. De lo contrario, la interfaz de usuario visible no cambia en absoluto, incluso en los casos en que debería hacerlo. React no rastrea las variables regulares ni los cambios en sus valores, por lo que no tienen influencia en el estado de la interfaz de usuario.

La función de actualización de estado expuesta por React (ese segundo elemento del array devuelto por `useState()`) desencadena un efecto interno de actualización de la interfaz de usuario. Esta función de actualización de estado hace más que establecer un nuevo valor: también le informa a React que un valor de estado ha cambiado y que, por lo tanto, la interfaz de usuario podría necesitar una actualización.

Por lo tanto, cada vez que llamas a `setErrorMessage('Error!')`, React no solo actualiza el valor que almacena internamente; también verifica la interfaz de usuario y la actualiza cuando es necesario. Las actualizaciones de la interfaz de usuario pueden implicar desde simples cambios de texto hasta la eliminación y adición completas de varios elementos del DOM. ¡Todo es posible allí!

React determina la nueva interfaz de usuario de destino volviendo a ejecutar (también llamado **reevaluar**) cualquier función de componente que se vea afectada por un cambio de estado. Eso incluye la función del componente que ejecutó la función `useState()` que devolvió la función de actualización de estado que se llamó. Pero también incluye cualquier componente secundario, ya que una actualización en un componente principal podría dar lugar a nuevos datos de estado que también utilizan algunos componentes secundarios (el valor de estado podría pasarse a los componentes secundarios a través de props).

Si necesitas una representación visual de cómo encaja todo esto, considera el siguiente diagrama:

**Figura 4.2**: Flujo de actualización de estado de React.

Es importante comprender y tener en cuenta que React volverá a ejecutar (reevaluar) una función de componente si se llama a una función de actualización de estado en la función de componente o en alguna función de componente principal. Esto también explica por qué el valor de estado devuelto por `useState()` (es decir, el primer elemento del array) puede ser una constante, aunque puedas asignar nuevos valores llamando a la función de actualización de estado (el segundo elemento del array). Dado que se vuelve a ejecutar toda la función del componente, `useState()` también se vuelve a llamar (porque se vuelve a ejecutar todo el código de la función del componente) y, por lo tanto, React devuelve un nuevo array con dos nuevos elementos. El primer elemento del array sigue siendo el valor de estado actual.

Sin embargo, como la función del componente se llamó debido a una actualización de estado, el valor de estado actual es ahora el valor actualizado.

Esto puede ser un poco complicado de asimilar, pero es así como funciona React internamente. Al final, se trata simplemente de que React llama a las funciones de los componentes varias veces, tal como cualquier función de JavaScript se puede llamar varias veces.

#### Convenciones de nomenclatura
El Hook `useState()` se utiliza normalmente en combinación con la desestructuración de arrays, de esta forma:

```javascript
const [enteredEmail, setEnteredEmail] = useState('');
```

Pero al utilizar la desestructuración de arrays, los nombres de las variables o constantes (`enteredEmail` y `setEnteredEmail`, en este caso) dependen de ti, como desarrollador. Por lo tanto, una pregunta válida es cómo deberías nombrar estas variables o constantes. Afortunadamente, existe una convención clara cuando se trata de React y `useState()`, y estos nombres de variables o constantes:
- El primer elemento (es decir, el valor de estado actual) debe nombrarse de manera que describa de qué se trata el valor del estado. Algunos ejemplos serían `enteredEmail`, `userEmail`, `providedEmail`, simplemente `email` o nombres similares. Debes evitar nombres genéricos como `a` o `value`, o nombres engañosos como `setValue` (que suena como si fuera una función, pero no lo es).
- El segundo elemento (es decir, la función de actualización de estado) debe nombrarse de manera que quede claro que es una función y que hace lo que hace. Algunos ejemplos serían `setEnteredEmail` o `setEmail`. En general, la convención para esta función es nombrarla **`setXYZ`**, donde `XYZ` es el nombre que elegiste para el primer elemento, la variable del valor de estado actual (ten en cuenta, sin embargo, que comienzas con un carácter en mayúscula, como en `setEnteredEmail`, no `setenteredEmail`).

#### Tipos de valores de estado permitidos
Gestionar direcciones de correo electrónico introducidas (o entradas de usuario en general) es de hecho un caso de uso y un ejemplo común para trabajar con el estado. Sin embargo, no estás limitado a este escenario ni a este tipo de valor.

En el caso de la entrada de usuario introducida, a menudo tratarás con valores de cadena de texto (*string*) como direcciones de correo electrónico, contraseñas, publicaciones de blog o valores similares. Pero **cualquier tipo de valor de JavaScript válido** se puede gestionar con la ayuda de `useState()`. Podrías, por ejemplo, gestionar la suma total de varios artículos del carrito de compras (es decir, un número) o un valor booleano (por ejemplo, "¿Confirmó el usuario los términos de uso?").

Además de gestionar tipos de valores primitivos, también puedes almacenar y actualizar tipos de datos de referencia como **objetos y arrays**.

> [!NOTE]
> Si la diferencia entre tipos de datos primitivos y por referencia no te queda del todo clara, se recomienda encarecidamente que profundices en este concepto central de JavaScript a través del siguiente enlace antes de continuar con este libro: [https://academind.com/tutorials/reference-vs-primitive-values](https://academind.com/tutorials/reference-vs-primitive-values).

React te brinda la flexibilidad de administrar todos estos tipos de valores como estado. Incluso puedes cambiar el tipo de valor en tiempo de ejecución (tal como puedes hacerlo en JavaScript puro). Está perfectamente bien almacenar un número como valor de estado inicial y actualizarlo a una cadena en un momento posterior.

Al igual que con JavaScript estándar, debes asegurarte, por supuesto, de que tu programa maneje este comportamiento adecuadamente, aunque técnicamente no hay nada de malo en cambiar de tipo.

---

### Sección 4: Trabajando con Múltiples Valores de Estado

Al construir cualquier cosa que no sean aplicaciones web o interfaces de usuario muy simples, necesitarás múltiples valores de estado. Tal vez los usuarios no solo puedan ingresar su correo electrónico sino también un nombre de usuario o su dirección. Tal vez también necesites rastrear algún estado de error o guardar artículos del carrito de compras. Tal vez los usuarios puedan hacer clic en un botón de "Me gusta" cuyo estado deba guardarse y reflejarse en la interfaz de usuario. Hay muchos valores que cambian con frecuencia y cuyos cambios deben reflejarse en la interfaz de usuario.

Considera este escenario concreto: tienes un componente que necesita administrar tanto el valor ingresado por un usuario en un campo de entrada de correo electrónico como el valor que se insertó en un campo de contraseña. Cada valor debe capturarse una vez que un campo pierde el foco.

Dado que tienes dos campos de entrada que contienen valores diferentes, tienes dos valores de estado: el correo electrónico ingresado y la contraseña ingresada. Aunque podrías usar ambos valores juntos en algún momento (por ejemplo, para iniciar la sesión de un usuario), los valores no se proporcionan simultáneamente. Además, es posible que también necesites que cada valor sea independiente, ya que lo usas para mostrar posibles mensajes de error (por ejemplo, "contraseña demasiado corta") mientras el usuario ingresa datos.

Escenarios como este son muy comunes y, por lo tanto, también puedes administrar múltiples valores de estado con el Hook `useState()`. Hay dos formas principales de hacerlo:
1. Usar múltiples fragmentos de estado (*state slices*, múltiples valores de estado).
2. Usar un único objeto de estado grande y fusionado.

#### Uso de múltiples fragmentos de estado (*State Slices*)
Puedes administrar múltiples valores de estado (también llamados a menudo fragmentos de estado o *state slices*) simplemente llamando a `useState()` varias veces en la función de tu componente.

Para el ejemplo descrito anteriormente, una función de componente (simplificada) podría verse así:

```javascript
function LoginForm() { 
  const [enteredEmail, setEnteredEmail] = useState(''); 
  const [enteredPassword, setEnteredPassword] = useState(''); 

  function handleUpdateEmail(event) { 
    setEnteredEmail(event.target.value); 
  }; 

  function handleUpdatePassword(event) { 
    setEnteredPassword(event.target.value); 
  }; 

  // A continuación, las props se dividen en varias líneas para una mejor legibilidad. 
  // Esto está permitido al usar JSX, al igual que en HTML estándar.
  return ( 
    <form> 
      <input type="email" placeholder="Your email" onBlur={handleUpdateEmail} /> 
      <input type="password" placeholder="Your password" onBlur={handleUpdatePassword} /> 
    </form> 
  ); 
};
```

En este ejemplo, se administran dos fragmentos de estado llamando a `useState()` dos veces. Por lo tanto, React registra y administra dos valores de estado internamente. Estos dos valores se pueden leer y actualizar de forma independiente el uno del otro.

> [!NOTE]
> En el ejemplo, las funciones que se activan tras los eventos comienzan con `handle` (`handleUpdateEmail` y `handleUpdatePassword`). Esta es una convención utilizada por algunos desarrolladores de React: las funciones de control de eventos comienzan con `handle…` para dejar en claro que estas funciones manejan ciertos eventos (desencadenados por el usuario). Esta no es una convención obligatoria; las funciones también podrían haberse llamado `updateEmail`, `updatePassword`, `emailUpdateHandler`, `passwordUpdateHandler` o cualquier otra cosa. Si el nombre es significativo y sigue una convención coherente, es una opción válida.

Puedes registrar tantos fragmentos de estado (llamando a `useState()` varias veces) como necesites en un componente. Podrías tener un valor de estado, pero también podrías tener docenas o incluso cientos. Por lo general, sin embargo, solo tendrás un par de fragmentos de estado por componente, ya que deberías intentar dividir los componentes más grandes (que podrían estar haciendo muchas cosas diferentes) en múltiples componentes más pequeños para mantenerlos manejables.

La ventaja de administrar múltiples valores de estado de esta manera es que puedes actualizarlos de forma independiente. Si el usuario ingresa una nueva dirección de correo electrónico, solo necesitas actualizar ese valor de estado del correo electrónico. El valor del estado de la contraseña no importa para esos efectos.

Una posible desventaja podría ser que múltiples fragmentos de estado —y, por lo tanto, múltiples llamadas a `useState()`— conducen a muchas líneas de código que podrían sobrecargar tu componente. Sin embargo, como se mencionó anteriormente, normalmente deberías intentar dividir los componentes grandes en múltiples componentes más pequeños de todos modos.

Aun así, existe una alternativa a la gestión de múltiples valores de estado de esta manera: también puedes gestionar un único objeto de valor de estado fusionado.

#### Gestión de objetos de estado fusionados
En lugar de llamar a `useState()` para cada fragmento de estado individual, puedes optar por un único objeto de estado grande que combine todos los diferentes valores de estado:

```javascript
function LoginForm() { 
  const [userData, setUserData] = useState({ 
    email: '', 
    password: '' 
  }); 

  function handleUpdateEmail(event) { 
    setUserData({ 
      email: event.target.value, 
      password: userData.password 
    }); 
  }; 

  function handleUpdatePassword(event) { 
    setUserData({ 
      email: userData.email, 
      password: event.target.value 
    }); 
  }; 

  // ... código omitido, porque el código JSX devuelto es el mismo que antes 
};
```

En este ejemplo, `useState()` se llama solo una vez (es decir, solo hay un fragmento de estado) y el valor inicial pasado a `useState()` es un objeto de JavaScript. El objeto contiene dos propiedades: `email` y `password`. Los nombres de las propiedades dependen de ti, pero deben describir los valores que se almacenarán en ellas.

`useState()` todavía devuelve un array con exactamente dos elementos. Que el valor inicial sea un objeto no cambia nada al respecto. El primer elemento del array devuelto ahora es simplemente un objeto en lugar de una cadena (como lo era en los ejemplos mostrados anteriormente). Como se mencionó antes, se puede usar cualquier tipo de valor de JavaScript válido al trabajar con `useState()`. Los tipos de valores primitivos, como cadenas o números, se pueden usar del mismo modo que usarías tipos de valores por referencia, como objetos o arrays (que, técnicamente, también son objetos).

La función de actualización de estado (`setUserData`, en el ejemplo anterior) sigue siendo una función creada por React que puedes llamar para establecer el estado en un nuevo valor. Además, no tendrías que volver a establecerlo como un objeto, aunque normalmente ese es el valor predeterminado. No cambias los tipos de valores al actualizar el estado a menos que tengas una buena razón para hacerlo (aunque, técnicamente, puedes cambiar a un tipo diferente en cualquier momento).

> [!NOTE]
> En el ejemplo anterior, la forma en que se utiliza la función de actualización de estado no es del todo correcta. Funcionaría, pero viola las mejores prácticas recomendadas. Más adelante en este capítulo aprenderás por qué es así y cómo deberías usar la función de actualización de estado en su lugar.

Al administrar objetos de estado como se muestra en el ejemplo anterior, hay una cosa crucial que debes tener en cuenta: **siempre debes establecer todas las propiedades que contiene el objeto, incluso las que no cambiaron**. Esto es necesario porque, al llamar a la función de actualización de estado, le indicas a React qué nuevo valor de estado debe almacenarse internamente.

Por lo tanto, cualquier valor que pases como argumento a la función de actualización de estado **sobrescribirá por completo el valor almacenado previamente**. Si proporcionas un objeto que contiene solo las propiedades que cambiaron, todas las demás propiedades se perderán, ya que el objeto de estado anterior se reemplaza por el nuevo, que contiene menos propiedades.

Esta es una trampa común y, por lo tanto, algo a lo que debes prestar atención. Por esta razón, en el ejemplo mostrado anteriormente, la propiedad que no cambia se establece en el valor del estado anterior (por ejemplo, `email: userData.email`, donde `userData` es la instantánea del estado actual y el primer elemento del array devuelto por `useState()`, mientras se establece `password` en `event.target.value`).

Depende totalmente de ti si prefieres administrar un valor de estado (es decir, un objeto que agrupa varios valores) o varios fragmentos de estado (es decir, varias llamadas a `useState()`). No existe una forma correcta o incorrecta y ambos enfoques tienen sus ventajas y desventajas.

Sin embargo, vale la pena señalar que, por lo general, deberías intentar dividir los componentes grandes en componentes más pequeños. Así como las funciones normales de JavaScript no deberían hacer demasiado trabajo en una sola función (se considera una buena práctica tener funciones separadas para diferentes tareas), los componentes también deberían centrarse en una o pocas tareas por componente. En lugar de tener un componente `<App />` gigante que maneje múltiples formularios, autenticación de usuarios y un carrito de compras directamente en un solo componente, sería preferible dividir el código de ese componente en múltiples componentes más pequeños que luego se combinan para construir la aplicación general.

Al seguir ese consejo, la mayoría de los componentes no deberían tener demasiado estado para administrar de todos modos, ya que administrar muchos valores de estado es un indicador de que un componente está haciendo demasiado trabajo. Es por eso que podrías terminar usando unos pocos fragmentos de estado por componente, en lugar de grandes objetos de estado.

#### Actualizar el estado basado en el estado anterior correctamente
Al aprender sobre los objetos como valores de estado, aprendiste que es fácil sobrescribir (y perder) datos accidentalmente porque podrías establecer el nuevo estado en un objeto que contenga solo las propiedades que cambiaron, no las que no cambiaron. Por eso, al trabajar con objetos o arrays como valores de estado, es importante agregar siempre las propiedades y elementos existentes al nuevo valor de estado.

Además, en general, establecer un valor de estado en un nuevo valor que se basa (al menos parcialmente) en el estado anterior es una tarea común. Podrías establecer `password` en `event.target.value` pero también establecer `email` en `userData.email` para asegurarte de que la dirección de correo electrónico almacenada no se pierda al actualizar una parte del estado general.

Sin embargo, ese no es el único escenario en el que el nuevo valor del estado podría basarse en el anterior. Otro ejemplo sería un componente de contador, por ejemplo, un componente como este:

```javascript
function Counter() { 
  const [counter, setCounter] = useState(0); 

  function handleIncrement() { 
    setCounter(counter + 1); 
  }; 

  return ( 
    <> 
      <p>Counter Value: {counter}</p> 
      <button onClick={handleIncrement}>Increment</button> 
    </> 
  ); 
};
```

En este ejemplo, se registra un controlador de eventos de clic para `<button>` (a través de la prop `onClick`). Con cada clic, el valor del estado `counter` se incrementa en 1.

Este componente funcionaría, pero el código mostrado en el fragmento de ejemplo en realidad **está violando una práctica recomendada importante**: las actualizaciones de estado que dependen de algún estado anterior deben realizarse con la ayuda de una **función que se pasa a la función de actualización de estado**. Para ser precisos, el ejemplo debería reescribirse de la siguiente manera:

```javascript
function Counter() { 
  const [counter, setCounter] = useState(0); 

  function handleIncrement() { 
    setCounter(function(prevCounter) { 
      return prevCounter + 1; 
    }); 
    // alternativamente, se podrían usar funciones flecha de JS:
    // setCounter(prevCounter => prevCounter + 1); 
  }; 

  return ( 
    <> 
      <p>Counter Value: {counter}</p> 
      <button onClick={handleIncrement}>Increment</button> 
    </> 
  ); 
};
```

Esto puede parecer un poco extraño. Podría parecer que ahora se pasa una función como nuevo valor de estado a la función de actualización de estado (es decir, el número almacenado en `counter` se reemplaza por una función). Pero, en efecto, ese no es el caso.

Técnicamente, se pasa una función como argumento a la función de actualización de estado, pero React no almacenará esa función como el nuevo valor de estado. En su lugar, al recibir una función como nuevo valor de estado en la función de actualización de estado, **React llamará a esa función por ti y le pasará el último valor de estado garantizado**. Por lo tanto, debes proporcionar una función que acepte al menos un parámetro: el valor de estado anterior (`prevCounter`). React pasará este valor a la función automáticamente cuando la ejecute internamente.

La función también debe devolver un valor: el nuevo valor de estado que debe ser almacenado por React. Además, dado que la función recibe el valor del estado anterior, ahora puedes derivar el nuevo valor del estado basándote en el valor del estado anterior (por ejemplo, sumándole el número 1, pero aquí se podría realizar cualquier operación).

¿Por qué es necesario esto si la aplicación también funcionaba bien antes de este cambio? Es necesario porque, en aplicaciones e interfaces de usuario de React más complejas, React podría estar procesando muchas actualizaciones de estado simultáneamente, potencialmente activadas desde diferentes fuentes en diferentes momentos.

Si no se utiliza el enfoque comentado, es posible que el orden de las actualizaciones de estado no sea el esperado y se puedan introducir errores en la aplicación. Incluso si sabes que tu caso de uso no se verá afectado y la aplicación cumple su función sin problemas, se recomienda adherirse a esta práctica recomendada y pasar una función a la función de actualización de estado si el nuevo estado depende del estado anterior.

Con este nuevo conocimiento en mente, echa otro vistazo a un ejemplo de código anterior:

```javascript
function LoginForm() { 
  const [userData, setUserData] = useState({ 
    email: '', 
    password: '' 
  }); 

  function handleUpdateEmail(event) { 
    setUserData({ 
      email: event.target.value, 
      password: userData.password 
    }); 
  }; 

  function handleUpdatePassword(event) { 
    setUserData({ 
      email: userData.email, 
      password: event.target.value 
    }); 
  }; 

  // ... código omitido, porque el código JSX devuelto es el mismo que antes 
};
```

¿Puedes detectar el error en este código?

No es un error técnico; el código se ejecutará bien y la aplicación funcionará como se espera. Pero, no obstante, hay un problema con este código: viola la práctica recomendada explicada. En el fragmento de código, el estado en ambas funciones controladoras se actualiza haciendo referencia a la instantánea del estado actual a través de `userData.password` y `userData.email`, respectivamente.

El fragmento de código debería reescribirse de la siguiente manera:

```javascript
function LoginForm() { 
  const [userData, setUserData] = useState({ 
    email: '', 
    password: '' 
  }); 

  function handleUpdateEmail(event) { 
    setUserData(prevData => ({ 
      email: event.target.value, 
      password: prevData.password 
    })); 
  }; 

  function handleUpdatePassword(event) { 
    setUserData(prevData => ({ 
      email: prevData.email, 
      password: event.target.value 
    })); 
  }; 

  // ... código omitido, porque el código JSX devuelto es el mismo que antes 
  // userData no se utiliza activamente aquí, por lo que podrías recibir una advertencia 
  // al respecto. Simplemente ignórala o comienza a usar userData 
  // (por ejemplo, mediante console.log(userData)) 
};
```

Al pasar una función flecha como argumento a `setUserData`, permites que React llame a esa función. React hará esto automáticamente y proporcionará el estado anterior (`prevData`) de manera garantizada. El valor devuelto (el objeto que almacena el correo electrónico o la contraseña actualizados y el correo electrónico o la contraseña almacenados actualmente) se establece luego como el nuevo estado. El resultado, en este caso, puede ser el mismo que antes, pero ahora el código cumple con las mejores prácticas recomendadas.

> [!NOTE]
> En el ejemplo anterior, se utilizó una función flecha en lugar de una función regular. Ambos enfoques están bien. Puedes utilizar cualquiera de los dos tipos de función; el resultado será el mismo.

En resumen, **siempre debes pasar una función a la función de actualización de estado si el nuevo estado depende del estado anterior**. De lo contrario, si el nuevo estado depende de algún otro valor (por ejemplo, la entrada directa del usuario), pasar directamente el nuevo valor de estado como argumento de la función es perfectamente válido y recomendado.

#### Enlace bidireccional (*Two-Way Binding*)
Hay un uso especial del concepto de estado de React que vale la pena analizar: el **enlace bidireccional** (*two-way binding*).

El enlace bidireccional es un concepto que se utiliza si tienes una fuente de entrada (típicamente un elemento `<input>`) que establece algún estado tras la entrada del usuario (por ejemplo, tras el evento `change`) y muestra la entrada al mismo tiempo.

Aquí tienes un ejemplo:

```javascript
function NewsletterField() { 
  const [email, setEmail] = useState(''); 

  function handleUpdateEmail(event) { 
    setEmail(event.target.value); 
  }; 

  return ( 
    <> 
      <input 
        type="email" 
        placeholder="Your email address" 
        value={email} 
        onChange={handleUpdateEmail} 
      /> 
    </> 
  ); 
};
```

En comparación con los otros fragmentos y ejemplos de código, la diferencia aquí es que el componente no solo almacena la entrada del usuario (tras el evento `change`, en este caso), sino que el valor introducido también se muestra en el elemento `<input>` (a través de la prop estándar `value`).

Esto puede parecer un bucle infinito, pero React se encarga de esto y garantiza que no se convierta en uno. En cambio, esto es lo que comúnmente se conoce como enlace bidireccional, ya que **un valor se establece y se lee desde la misma fuente**.

Quizás te preguntes por qué se comenta esto aquí, pero es importante saber que es perfectamente válido escribir código como este. Además, este tipo de código podría ser necesario si no solo deseas establecer un valor (en este caso, el valor del correo electrónico) tras la entrada del usuario en el campo `<input>`, sino también desde otras fuentes. Por ejemplo, podrías tener un botón en el componente que, al hacer clic en él, deba borrar la dirección de correo electrónico ingresada.

Podría verse así:

```javascript
function NewsletterField() { 
  const [email, setEmail] = useState(''); 

  function handleUpdateEmail(event) { 
    setEmail(event.target.value); 
  }; 

  function handleClearInput() { 
    setEmail(''); // restablece la entrada de correo electrónico (de nuevo a una cadena vacía) 
  }; 

  return ( 
    <> 
      <input 
        type="email" 
        placeholder="Your email address" 
        value={email} 
        onChange={handleUpdateEmail} 
      /> 
      <button onClick={handleClearInput}>Reset</button> 
    </> 
  ); 
};
```

En este ejemplo actualizado, la función `handleClearInput` se ejecuta cuando se hace clic en `<button>`. Dentro de la función, el estado `email` se restablece a una cadena vacía. Sin el enlace bidireccional, el estado se actualizaría, pero el cambio no se reflejaría en el elemento `<input>`. Allí, el usuario seguiría viendo su última entrada. El estado reflejado en la interfaz de usuario (el sitio web) y el estado administrado internamente por React serían diferentes: un error que debes evitar por completo.

---

### Sección 5: Derivar Valores a partir del Estado

Como probablemente ya puedas intuir, el estado es un concepto clave en React. El estado te permite administrar datos que, cuando cambian, obligan a React a reevaluar un componente y, en última instancia, la interfaz de usuario.

Como desarrollador, puedes usar valores de estado en cualquier lugar de tu componente (y en tus componentes secundarios, pasándoles el estado a través de props). Podrías, por ejemplo, repetir lo que introdujo un usuario de esta manera:

```javascript
function Repeater() { 
  const [userInput, setUserInput] = useState(''); 

  function handleChange(event) { 
    setUserInput(event.target.value); 
  }; 

  return ( 
    <> 
      <input type="text" onChange={handleChange} /> 
      <p>You entered: {userInput}</p> 
    </> 
  ); 
};
```

Este componente puede no ser demasiado útil, pero funcionará y utiliza estado.

A menudo, para hacer cosas más útiles, necesitarás utilizar un valor de estado como base para **derivar un nuevo valor** (a menudo más complejo). Por ejemplo, en lugar de simplemente repetir lo que ingresó el usuario, podrías contar la cantidad de caracteres ingresados y mostrar esa información al usuario:

```javascript
function CharCounter() { 
  const [userInput, setUserInput] = useState(''); 

  function handleChange(event) { 
    setUserInput(event.target.value); 
  }; 

  const numChars = userInput.length; 

  return ( 
    <> 
      <input type="text" onChange={handleChange} /> 
      <p>Characters entered: {numChars}</p> 
    </> 
  ); 
};
```

Observa la adición de la nueva constante `numChars` (también podría ser una variable mediante `let`). Esta constante se deriva del estado `userInput` accediendo a la propiedad `length` en el valor de cadena que se almacena en el estado `userInput`.

¡Esto es importante! No estás limitado a trabajar únicamente con valores de estado directos. Puedes administrar algún valor clave como estado (es decir, el valor que cambiará) y derivar otros valores basados en ese valor de estado, como, en este caso, la cantidad de caracteres ingresados por el usuario. De hecho, esto es algo que harás con frecuencia como desarrollador de React.

Quizás también te preguntes por qué `numChars` es una constante y está fuera de la función `handleChange`. Después de todo, esa es la función que se ejecuta tras la entrada del usuario (es decir, con cada pulsación de tecla que realiza el usuario).

Recuerda lo que aprendiste sobre cómo maneja React el estado internamente: cuando llamas a la función de actualización de estado (`setUserInput`, en este caso), **React reevaluará el componente al que pertenece el estado**. Esto significa que React volverá a llamar a la función del componente `CharCounter`. Por lo tanto, todo el código de esa función se ejecuta nuevamente.

**Figura 4.3**: El valor `numChars` se deriva del estado cuando la función del componente se ejecuta nuevamente.

React vuelve a ejecutar las funciones de los componentes para determinar cómo debería verse la interfaz de usuario después de la actualización del estado; y, si detecta alguna diferencia en comparación con la interfaz de usuario renderizada actualmente, React actualizará la interfaz de usuario del navegador (es decir, el DOM) en consecuencia. De lo contrario, no pasará nada.

Dado que React llama a la función del componente nuevamente, `useState()` producirá su array de valores (valor de estado actual y función de actualización de estado). El valor del estado actual será el estado en el que se configuró cuando se llamó a `setUserInput`. Por lo tanto, este nuevo valor `userInput` se puede usar para realizar otros cálculos en cualquier lugar de la función del componente, como derivar `numChars` accediendo a la propiedad `length` de `userInput`.

Por eso `numChars` puede ser una constante: para esta ejecución específica del componente, no se reasignará. Solo se podría derivar un nuevo valor cuando la función del componente se vuelva a ejecutar en el futuro (es decir, si se vuelve a llamar a `setUserInput`). En ese caso, se creará una constante `numChars` completamente nueva (y la anterior se descartará).

---

### Sección 6: Trabajo con Formularios y Envío de Formularios

El estado se utiliza habitualmente cuando se trabaja con formularios y entradas de usuario. De hecho, la mayoría de los ejemplos de este capítulo trataron sobre alguna forma de entrada del usuario.

Hasta este punto, todos los ejemplos se centraron en escuchar eventos de usuario que están adjuntos directamente a elementos de entrada individuales. Eso tiene sentido porque a menudo querrás escuchar eventos como pulsaciones de teclas o un campo que pierde el foco. Especialmente al agregar validación de entrada (es decir, verificar los valores ingresados), es posible que desees usar eventos de entrada para brindar a los usuarios del sitio web comentarios útiles mientras escriben.

Pero también es muy común reaccionar al **envío general del formulario** (*form submission*). Por ejemplo, el objetivo podría ser combinar la entrada de varios campos de entrada y enviar los datos a algún servidor backend. ¿Cómo podrías lograr esto? ¿Cómo puedes escuchar y reaccionar al envío de un formulario?

Puedes hacer todas estas cosas con la ayuda de eventos estándar de JavaScript y las props de control de eventos correspondientes proporcionadas por React. Específicamente, se puede agregar la prop `onSubmit` a los elementos `<form>` para asignar una función que debe ejecutarse una vez que se envía un formulario. Para luego manejar el envío con React y JavaScript, debes asegurarte de que el navegador no haga lo predeterminado: generar (y enviar) una solicitud HTTP automáticamente.

Al igual que en JavaScript puro, esto se puede lograr llamando al método `preventDefault()` en el objeto de evento generado automáticamente.

Aquí tienes un ejemplo completo:

```javascript
function NewsletterSignup() { 
  const [email, setEmail] = useState(''); 
  const [agreed, setAgreed] = useState(false); 

  function handleUpdateEmail(event) { 
    // aquí se podría agregar la validación del correo electrónico
    setEmail(event.target.value); 
  }; 

  function handleUpdateAgreement(event) { 
    setAgreed(event.target.checked); // checked es una propiedad booleana predeterminada de JS
  }; 

  function handleSignup(event) { 
    event.preventDefault(); // evita el comportamiento predeterminado del navegador de enviar una solicitud HTTP
    const userData = {userEmail: email, userAgrees: agreed}; 
    // doWhateverYouWant(userData); 
  }; 

  return ( 
    <form onSubmit={handleSignup}> 
      <div> 
        <label htmlFor="email">Your email</label> 
        <input type="email" id="email" onChange={handleUpdateEmail}/> 
      </div> 
      <div> 
        <input type="checkbox" id="agree" onChange={handleUpdateAgreement}/> 
        <label htmlFor="agree">Agree to terms and conditions</label> 
      </div> 
    </form> 
  ); 
};
```

Este fragmento de código maneja el envío del formulario a través de la función `handleSignup()` que está asignada a la prop integrada `onSubmit`. La entrada del usuario aún se obtiene con la ayuda de dos fragmentos de estado (`email` y `agreed`), que se actualizan tras los eventos `change` de las entradas.

> [!NOTE]
> En el ejemplo de código anterior, es posible que hayas notado una nueva prop que no se usó antes en este libro: `htmlFor`. Esta es una prop especial integrada en React y los elementos JSX centrales que proporciona. Se puede agregar a elementos `<label>` para establecer el atributo `for` para estos elementos. La razón por la que se llama `htmlFor` en lugar de simplemente `for` es que, como se explicó anteriormente en el libro, JSX parece HTML pero no es HTML: es JavaScript por debajo. En JavaScript, `for` es una palabra clave reservada para bucles `for`. Para evitar problemas, la prop se denomina `htmlFor`.

El uso de `onSubmit` (combinado con `preventDefault()`) para manejar envíos de formularios es una forma muy común de lidiar con entradas de usuario y formularios en React. Pero cuando trabajas en proyectos que usan **React 19 o superior**, también puedes usar una forma alternativa para manejar envíos de formularios: puedes usar una funcionalidad de React llamada **Form Actions**, que se cubrirá con gran detalle en el Capítulo 9, *Manejo de Entradas de Usuario y Formularios con Form Actions*.

---

### Sección 7: Elevación del Estado (*Lifting State Up*)

Aquí hay un escenario y problema común: tienes dos componentes en tu aplicación de React y un cambio o evento en el componente A debería cambiar el estado en el componente B. Para que esto sea menos abstracto, considera el siguiente ejemplo simple:

```javascript
function SearchBar() { 
  const [searchTerm, setSearchTerm] = useState(''); 

  function handleUpdateSearchTerm(event) { 
    setSearchTerm(event.target.value); 
  }; 

  return <input type="search" onChange={handleUpdateSearchTerm} />; 
}; 

function Overview() { 
  return <p>Currently searching for {searchTerm}</p>; 
}; 

function App() { 
  return ( 
    <> 
      <SearchBar /> 
      <Overview /> 
    </> 
  ); 
};
```

En este ejemplo, el componente `Overview` debería mostrar el término de búsqueda ingresado. Sin embargo, el término de búsqueda se administra en realidad en otro componente: el componente `SearchBar`. En este ejemplo simple, los dos componentes podrían, por supuesto, fusionarse en un solo componente y el problema se resolvería. Pero es muy probable que al construir aplicaciones más realistas, te enfrentes a escenarios similares pero con componentes mucho más complejos. Dividir los componentes en partes más pequeñas se considera una buena práctica, ya que mantiene manejables los componentes individuales.

Tener múltiples componentes que dependen de algún fragmento de estado compartido es, por lo tanto, un escenario al que te enfrentarás con frecuencia cuando trabajes con React.

Este problema se puede resolver **elevando el estado** (*lifting state up*). Al elevar el estado, el estado no se administra en ninguno de los dos componentes que lo usan —ni en `Overview`, que lee el estado, ni en `SearchBar`, que lo establece—, sino en un **componente ancestro compartido**. Para ser precisos, se administra en el **ancestro compartido más cercano**. Recuerda que los componentes están anidados entre sí y, por lo tanto, al final se construye un "árbol de componentes" (con el componente `App` como componente raíz).

**Figura 4.4**: Un árbol de componentes de ejemplo.

En el ejemplo de código simple anterior, el componente `App` es el componente ancestro más cercano (y, en este caso, el único) tanto de `SearchBar` como de `Overview`. Si la aplicación estuviera estructurada como se muestra en la figura, con el estado establecido en uno de los componentes `Product` y utilizado en `Cart`, `Products` sería el componente ancestro más cercano.

El estado se eleva utilizando props en los componentes que necesitan manipular (es decir, establecer) o leer el estado, y registrando el estado en el componente ancestro que comparten los otros dos componentes. Aquí está el ejemplo actualizado de antes:

```javascript
function SearchBar({onUpdateSearch}) { 
  return <input type="search" onChange={onUpdateSearch} />; 
}; 

function Overview({currentTerm}) { 
  return <p>Currently searching for {currentTerm}</p>; 
}; 

function App() { 
  const [searchTerm, setSearchTerm] = useState(''); 

  function handleUpdateSearchTerm(event) { 
    setSearchTerm(event.target.value); 
  }; 

  return ( 
    <> 
      <SearchBar onUpdateSearch={handleUpdateSearchTerm} /> 
      <Overview currentTerm={searchTerm} /> 
    </> 
  ); 
};
```

En realidad, el código no cambió mucho; principalmente se reubicó un poco. El estado ahora se administra dentro del ancestro compartido y componente `App`, y los otros dos componentes obtienen acceso a él a través de props.

Están sucediendo tres cosas clave en este ejemplo:
1. El componente `SearchBar` recibe una prop llamada `onUpdateSearch`, cuyo valor es una función: una función creada en el componente `App` y pasada a `SearchBar` desde `App`.
2. La prop `onUpdateSearch` se establece luego como un valor para la prop `onChange` en el elemento `<input>` dentro del componente `SearchBar`.
3. El estado `searchTerm` (es decir, su valor actual) se pasa desde `App` a `Overview` a través de una prop llamada `currentTerm`.

Los dos primeros puntos pueden resultar confusos. Pero ten en cuenta que, en JavaScript, las funciones son objetos de primera clase y valores normales. Puedes almacenar funciones en variables y, al usar React, pasar funciones como valores para props. De hecho, ya pudiste ver eso en acción al principio de este capítulo: al introducir eventos y manejo de eventos, se proporcionaron funciones como valores para todas estas props `onXYZ` (`onChange`, `onBlur`, etc.).

En este fragmento de código, se pasa una función como valor para una prop personalizada (es decir, una prop esperada en un componente creado por ti, no integrado en React). La prop `onUpdateSearch` espera una función como valor porque la prop misma se usa luego como un valor para la prop `onChange` en el elemento `<input>`.

La prop se llama `onUpdateSearch` para dejar en claro que espera una función como valor y que se conectará a un evento. Sin embargo, se podría haber elegido cualquier nombre; no tiene que empezar con `on`. Pero es una convención común nombrar de esta manera las props que esperan funciones como valores y que están destinadas a conectarse a eventos.

Por supuesto, `updateSearch` no es un evento predeterminado, pero dado que la función se llamará efectivamente tras el evento `change` del elemento `<input>`, la prop actúa como un evento personalizado.

Con esta estructura, el estado se elevó al componente `App`. Este componente registra y administra el estado. Sin embargo, también expone la función de actualización de estado (indirectamente, en este caso, ya que está envuelta por la función `handleUpdateSearchTerm`) al componente `SearchBar`. También proporciona el valor de estado actual (`searchTerm`) al componente `Overview` a través de la prop `currentTerm`.

Dado que los componentes secundarios y descendientes también son reevaluados por React cuando el estado cambia en un componente, los cambios en el componente `App` también harán que se reevalúen los componentes `SearchBar` y `Overview`. Por lo tanto, se captará el nuevo valor de prop para `searchTerm` y React actualizará la interfaz de usuario.

No se necesitan nuevas funciones de React para esto: es solo una combinación de estado y props. Sin embargo, dependiendo de cómo se conecten estas características y dónde se utilicen, se pueden lograr patrones de aplicación tanto simples como más complejos.

---

### Sección 8: Resumen y puntos clave

- Los controladores de eventos se pueden agregar a elementos JSX a través de props `on[NombreDelEvento]` (por ejemplo, `onClick`, `onChange`).
- Se puede ejecutar cualquier función tras eventos (del usuario).
- Para obligar a React a reevaluar componentes y (posiblemente) actualizar la interfaz de usuario renderizada, se debe utilizar el **estado**.
- El estado se refiere a datos administrados internamente por React, y se puede definir un valor de estado a través del Hook `useState()`.
- Los Hooks de React son funciones de JavaScript que agregan características especiales a los componentes de React (por ejemplo, la función de estado en este capítulo).
- `useState()` siempre devuelve un array con exactamente dos elementos:
  1. El primer elemento es el **valor de estado actual**.
  2. El segundo elemento es una **función para establecer el estado en un nuevo valor** (la función de actualización de estado).
- Al establecer el estado en un nuevo valor que depende del valor anterior, se debe pasar una función a la función de actualización de estado. Esta función recibe el estado anterior como parámetro (que React proporcionará automáticamente) y devuelve el nuevo estado que se debe establecer.
- Cualquier valor válido de JavaScript se puede establecer como estado, además de valores primitivos como cadenas o números, lo que también incluye valores de referencia como objetos y arrays.
- Si el estado necesita cambiar debido a algún evento que ocurre en otro componente, debes **elevar el estado** y administrarlo en un nivel superior compartido (es decir, un componente ancestro común).

---

### Sección 9: ¿Qué sigue?

El estado es un bloque de construcción extremadamente importante porque te permite crear aplicaciones verdaderamente dinámicas. Con este concepto clave establecido, el próximo capítulo se centrará en utilizar el estado (y otros conceptos aprendidos hasta ahora) para **renderizar contenido condicionalmente y renderizar listas de contenido**.

Estas son tareas comunes que se requieren en casi cualquier interfaz de usuario o aplicación web que estés creando, sin importar si se trata de mostrar una superposición de advertencia o mostrar una lista de productos. El próximo capítulo te ayudará a agregar tales características a tus aplicaciones de React.

---

### Sección 10: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/04-state-events/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/04-state-events/exercises/questions-answers.md):

1. ¿Qué "problema" resuelve el estado?
2. ¿Cuál es la diferencia entre props y estado?
3. ¿Cómo se registra el estado en un componente?
4. ¿Qué valores proporciona el Hook `useState()`?
5. ¿Cuántos valores de estado se pueden registrar para un solo componente?
6. ¿El estado también afecta a otros componentes (además del componente en el que se registró)?
7. ¿Cómo se debe actualizar el estado si el nuevo estado depende del estado anterior?
8. ¿Cómo se puede compartir el estado entre múltiples componentes?

---

### Sección 11: Aplica lo aprendido

Con los nuevos conocimientos adquiridos en este capítulo, finalmente puedes crear interfaces de usuario y aplicaciones de React verdaderamente dinámicas. En lugar de limitarte a contenidos y páginas estáticos y codificados de forma fija, ahora puedes utilizar el estado para establecer y actualizar valores y obligar a React a reevaluar los componentes y la interfaz de usuario.

Aquí encontrarás una actividad que te permitirá aplicar todos los conocimientos, incluidos estos nuevos conocimientos sobre el estado, que has adquirido hasta este punto.

#### Actividad 4.1: Construcción de una calculadora simple
En esta actividad, construirás una calculadora muy básica que permite a los usuarios sumar, restar, multiplicar y dividir dos números entre sí.

Los pasos son los siguientes:
1. Construye la interfaz de usuario utilizando componentes de React. Asegúrate de construir cuatro componentes separados para las cuatro operaciones matemáticas, aunque se pueda reutilizar mucho código.
2. Recopila la entrada del usuario y actualiza el resultado cada vez que el usuario ingresa un valor en uno de los dos campos de entrada relacionados.
3. Ten en cuenta que al trabajar con números y obtener esos números de la entrada del usuario, deberás asegurarte de que los valores ingresados se traten como números y no como cadenas.

El resultado final y la interfaz de usuario de la calculadora deberían verse así:

**Figura 4.5**: Interfaz de usuario de la calculadora.

> [!NOTE]
> El estilo, por supuesto, variará. Para obtener el mismo estilo que se muestra en la captura de pantalla, usa mi proyecto inicial preparado, que puedes encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/04-state-events/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/04-state-events/activities/practice-1-start).
> Analiza el archivo `index.css` en ese proyecto para determinar cómo estructurar tu código JSX para aplicar los estilos.
> Encontrarás la solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/04-state-events/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/04-state-events/activities/practice-1).

#### Actividad 4.2: Mejora de la calculadora
En esta actividad, te basarás en la Actividad 4.1 para hacer que la calculadora construida allí sea un poco más compleja. El objetivo es reducir la cantidad de componentes y construir un solo componente en el que los usuarios puedan seleccionar la operación matemática a través de un elemento desplegable (*drop-down*). Además, el resultado debe mostrarse en un componente diferente, es decir, no en el componente donde se recopila la entrada del usuario.

Los pasos son los siguientes:
1. Elimina tres de los cuatro componentes de la actividad anterior y utiliza un solo componente para todas las operaciones matemáticas.
2. Agrega un elemento desplegable (elemento `<select>`) a ese componente restante (entre las dos entradas) y agrégale las cuatro operaciones matemáticas como opciones (elementos `<option>`).
3. Usa el estado para recopilar tanto los números ingresados por el usuario como la operación matemática elegida a través del menú desplegable (depende de ti si prefieres un solo objeto de estado o múltiples fragmentos de estado).
4. Muestra el resultado en otro componente (Pista: elige un buen lugar para registrar y administrar el estado).

El resultado y la interfaz de usuario de la calculadora deberían verse así:

**Figura 4.6**: Interfaz de usuario de la calculadora mejorada.

> [!NOTE]
> Encontrarás la solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/04-state-events/activities/practice-2](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/04-state-events/activities/practice-2).
