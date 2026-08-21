# Parte 1: Fundamentos de React

## Capítulo 9: Manejo de Entradas de Usuario y Formularios con Form Actions

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Describir el propósito de las Form Actions en React.
- Construir y utilizar Form Actions personalizadas para manejar los envíos de formularios.
- Usar el Hook `useActionState()` para gestionar el estado dependiente del formulario.
- Renderizar una interfaz de usuario pendiente durante el envío mediante el Hook `useFormStatus()`.
- Realizar actualizaciones de estado optimistas con el Hook `useOptimistic()`.
- Implementar acciones tanto sincrónicas como asincrónicas.

---

### Sección 1: Introducción

En el Capítulo 4, *Trabajando con Eventos y Estado*, aprendiste cómo manejar los envíos de formularios en aplicaciones de React. Y aunque no hay absolutamente nada de malo con el enfoque mostrado allí (de hecho, es probablemente el enfoque que encontrarás en la mayoría de los proyectos de React), React proporciona una forma alternativa de manejar los envíos de formularios cuando se trabaja en proyectos que utilizan **React versión 19 o posterior**. React 19 introdujo una nueva característica llamada **Actions** (también denominadas *Form Actions* a lo largo de este capítulo) que puede simplificar el proceso de manejar envíos de formularios, extraer la entrada del usuario y proporcionar retroalimentación de validación.

Este capítulo primero revisará los envíos de formularios como se introdujeron en el Capítulo 4 y explorará cómo se puede extraer y validar la entrada del usuario. A partir de entonces, este capítulo presentará las Form Actions y explicará cómo realizar los mismos pasos (manejar el envío, extraer valores y validar valores) utilizando esa función. También aprenderás sobre los Hooks de React relacionados con Actions como `useActionState()`.

---

### Sección 2: Manejo de envíos de formularios sin Actions

Como aprendiste en el Capítulo 4, *Trabajando con Eventos y Estado*, cuando no usas Actions, puedes manejar los envíos de formularios escuchando el evento `submit` a través de la prop `onSubmit` en el elemento `<form>`.

Considera el siguiente fragmento de código de ejemplo:

```javascript
function App() { 
  function handleSubmit(event) { 
    event.preventDefault(); 
    console.log('Submitted!'); 
  } 

  return ( 
    <form onSubmit={handleSubmit}> 
      <p> 
        <label htmlFor="email">Email</label> 
        <input type="email" id="email" /> 
      </p> 
      <p> 
        <label htmlFor="password">Password</label> 
        <input type="password" id="password" /> 
      </p> 
      <p className="actions"> 
        <button>Login</button> 
      </p> 
    </form> 
  ); 
}
```

> [!NOTE]
> Puedes encontrar el ejemplo funcional completo en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/examples/01-form-submission-without-actions](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/examples/01-form-submission-without-actions).

Este código muestra un formulario y maneja su envío a través de la función `handleSubmit()`. Esta función recibe automáticamente un objeto de evento, que se utiliza para evitar el comportamiento predeterminado del navegador de enviar una solicitud HTTP al servidor que aloja el sitio web.

Pero, por supuesto, limitarse a manejar el envío no es demasiado útil. Por lo general, también deseas extraer y utilizar los valores introducidos por el usuario del sitio web.

#### Extracción de entradas de usuario
Cuando se trata de extraer los valores ingresados en un formulario, tienes un par de opciones:
- Realizar el seguimiento de los valores a través del estado (es decir, mediante el uso de `useState()`), como se describe en el Capítulo 4.
- Confiar en las Refs a través de `useRef()`, como se explica en el Capítulo 7, *Portales y Refs*.
- Aprovechar el objeto `event` creado automáticamente.

#### Rastreo de estado
Puedes realizar un seguimiento de los valores ingresados por el usuario a través del estado gestionado por `useState()`, como se explica en el Capítulo 4. Por ejemplo, los valores de entrada del formulario del fragmento de código anterior se pueden rastrear y utilizar en `handleSubmit()`, como se muestra en el siguiente ejemplo:

```javascript
function App() { 
  const [email, setEmail] = useState(''); 
  const [password, setPassword] = useState(''); 

  function handleSubmit(event) { 
    event.preventDefault(); 
    const credentials = { email, password }; 
    console.log(credentials); 
  } 

  function handleEmailChange(event) { 
    setEmail(event.target.value); 
  } 

  function handlePasswordChange(event) { 
    setPassword(event.target.value); 
  } 

  return ( 
    <form onSubmit={handleSubmit}> 
      <p> 
        <label htmlFor="email">Email</label> 
        <input type="email" id="email" value={email} onChange={handleEmailChange} /> 
      </p> 
      <p> 
        <label htmlFor="password">Password</label> 
        <input type="password" id="password" value={password} onChange={handlePasswordChange} /> 
      </p> 
      <p className="actions"> 
        <button>Login</button> 
      </p> 
    </form> 
  ); 
}
```

En este fragmento de código de ejemplo actualizado, el Hook `useState()` se usa para administrar los valores de estado `email` y `password`. Los valores de estado se actualizan con cada pulsación de tecla en los campos de entrada. Como resultado, los últimos valores ingresados están disponibles dentro de `handleSubmit()` cuando se envía el formulario.

Este enfoque funciona bien y se encontrará en muchos proyectos de React. Sin embargo, existen algunas posibles desventajas al usar el estado para rastrear los valores de entrada:
- Dado que el estado se actualiza en cada pulsación de tecla y la función del componente se vuelve a ejecutar cada vez que cambia algún valor de estado, el rendimiento de la aplicación podría verse afectado.
- Cuando se trabaja con formularios más complejos con más campos de entrada, es posible que sea necesario administrar muchos valores de estado diferentes.

Puedes solucionar estos problemas implementando optimizaciones de código, que se analizarán en el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, y administrando el estado como un objeto, como se explica en el Capítulo 11, *Trabajando con Estado Complejo*.

Pero también podrías considerar usar Refs para extraer los valores de entrada.

#### Confiar en Refs
Si estás creando un formulario donde no planeas establecer valores de entrada y donde, en su lugar, solo deseas leer esos valores al enviar el formulario, usar la funcionalidad de Refs de React (introducida en el Capítulo 7) podría tener sentido:

```javascript
function App() { 
  const emailRef = useRef(null); 
  const passwordRef = useRef(null); 

  function handleSubmit(event) { 
    event.preventDefault(); 
    const credentials = { 
      email: emailRef.current.value, 
      password: passwordRef.current.value, 
    }; 
    console.log(credentials); 
  } 

  return ( 
    <form onSubmit={handleSubmit}> 
      <p> 
        <label htmlFor="email">Email</label> 
        <input type="email" id="email" ref={emailRef} /> 
      </p> 
      <p> 
        <label htmlFor="password">Password</label> 
        <input type="password" id="password" ref={passwordRef} /> 
      </p> 
      <p className="actions"> 
        <button>Login</button> 
      </p> 
    </form> 
  ); 
}
```

En este bloque de código, el Hook `useRef()` se utiliza para crear dos Refs que están conectadas a los campos de entrada de correo electrónico y contraseña. Estas Refs se utilizan luego para leer los valores ingresados dentro de `handleSubmit()`.

Al utilizar este enfoque, la función del componente `App` ya no se ejecuta con cada pulsación de tecla. Pero aún tienes que escribir el código donde se crean las Refs a través de `useRef()` y donde se conectan a los elementos JSX a través de la prop `ref`.

Es por eso que podrías considerar confiar en el navegador y en el objeto `event` creado automáticamente (que se recibe en `handleSubmit()`), en lugar de usar las funciones de React para extraer esos valores ingresados.

#### Aprovechar el objeto `event`
En el Capítulo 4, *Trabajando con Eventos y Estado*, aprendiste que el navegador intenta enviar una solicitud HTTP cuando se envía un formulario. Es por eso que se llama a `event.preventDefault()` dentro de `handleSubmit()`: esta llamada a la función garantiza que esta solicitud no se envíe.

Sin embargo, el objeto de evento no solo es útil para evitar ese comportamiento predeterminado. También contiene información importante sobre el evento de envío que ocurrió. Por ejemplo, puedes obtener acceso al objeto del DOM del formulario subyacente (es decir, un objeto JavaScript que describe el elemento `<form>` renderizado, su configuración y su estado actual) a través de `event.currentTarget`.

Esto es muy útil porque puedes pasar ese objeto del DOM del formulario a la función constructora **`FormData`** que proporciona el navegador. Esta interfaz se puede utilizar para extraer los valores de los campos de entrada de un formulario.

El siguiente ejemplo muestra el uso concreto de esta característica:

```javascript
function App() { 
  function handleSubmit(event) { 
    event.preventDefault(); 
    const fd = new FormData(event.currentTarget); 
    const credentials = { 
      email: fd.get('email'), 
      password: fd.get('password'), 
    }; 
    console.log(credentials); 
  } 

  return ( 
    <form onSubmit={handleSubmit}> 
      <p> 
        <label htmlFor="email">Email</label> 
        <input type="email" id="email" name="email" /> 
      </p> 
      <p> 
        <label htmlFor="password">Password</label> 
        <input type="password" id="password" name="password" /> 
      </p> 
      <p className="actions"> 
        <button>Login</button> 
      </p> 
    </form> 
  ); 
}
```

Como puedes ver en el fragmento de código anterior, el objeto de datos del formulario `fd` se construye instanciando `FormData`. Como se mencionó, la interfaz `FormData` la proporciona el navegador; por lo tanto, no es necesario importarla desde React ni desde ninguna otra biblioteca.

Este objeto de datos de formulario ofrece varios métodos que ayudan a acceder a los valores de los campos del formulario, por ejemplo, el método `get()` para extraer el valor de un campo de entrada específico. Para identificar el campo de entrada del cual deseas obtener el valor, el método `get()` requiere el nombre del campo de entrada como argumento. Es por eso que también debes establecer la prop `name` en los elementos de control del formulario (es decir, en los elementos `<input>` en el ejemplo anterior).

Este enfoque tiene la ventaja de que no necesitas ni estado ni Refs; por lo tanto, se debe escribir un poco menos de código. Además, dado que casi no se utilizan funciones de React, este código será menos propenso a romperse debido a posibles cambios futuros en React.

En consecuencia, este enfoque podría parecer la mejor manera de manejar los envíos de formularios. ¿Pero lo es?

#### ¿Qué solución es la mejor?
No existe una forma correcta o incorrecta de manejar los envíos de formularios. Además de las preferencias personales, los requisitos de la aplicación también pueden favorecer un enfoque sobre los demás.

Por ejemplo, si tu aplicación necesita cambiar los valores de entrada, usar únicamente `FormData` como se muestra arriba no sería ideal, ya que tendrías que escribir código imperativo para actualizar un campo de entrada.

Eso es un problema porque, como se explicó en el Capítulo 1, *React – Qué es y por qué*, debes evitar escribir código como este en tus aplicaciones de React:

```javascript
function clearInput() { 
  document.getElementById('email').value = ''; // imperative code :( 
}
```

Por lo tanto, si necesitas editar el valor de una entrada, es preferible utilizar el estado (es decir, `useState()`):

```javascript
const [email, setEmail] = useState(''); 
// ... other code 
function clearInput() { 
  setEmail(''); 
} 
// simplified JSX code below 
return ( 
  <form> 
    <input value={email} onChange={event => setEmail(event.target.value)} /> 
  </form> 
);
```

Incluso si no necesitas actualizar ningún campo de entrada, es posible que el objeto de evento y `FormData` por sí solos no sean suficientes.

Por ejemplo, si necesitas acceder a los campos de entrada fuera de `handleSubmit()`, el objeto de evento no está disponible. Como resultado, interactuar con el elemento de formulario y sus elementos secundarios no es posible a través del objeto de evento. En tales escenarios, trabajar con Refs que estén directamente conectadas a los elementos de entrada individuales probablemente simplificará las cosas.

El siguiente ejemplo utiliza una ref para llamar al método integrado `focus()` del elemento `<input>` dentro de una función:

```javascript
const emailRef = useRef(null); 

function showForm() { 
  // other code ... 
  emailRef.current.focus(); 
} 
// simplified JSX code below 
return ( 
  <form> 
    <input ref={emailRef} /> 
  </form> 
);
```

Entonces, como puedes ver, no existe una solución mágica universal. Todas estas características de React y las diferentes formas de manejar los envíos de formularios existen por buenas razones. Puedes mezclarlas y combinarlas según sea necesario; por lo tanto, es útil conocer estas diferentes opciones.

Pero a pesar de que ya existen un par de formas de manejar los envíos de formularios, con React 19, hay una más.

---

### Sección 3: Manejo de envíos de formularios con Actions

React 19 introdujo el concepto de **Actions** (de formulario), un concepto que en realidad consta de dos tipos de acciones: **Client Actions** y **Server Actions**. Ambos tipos de acciones pueden ayudar a manejar los envíos de formularios, pero para el propósito de este capítulo, el término *Form Actions* se utilizará para describir las Client Actions (es decir, acciones de formulario que se ejecutan en el navegador del usuario del sitio web). Las Server Actions se cubrirán por separado en el Capítulo 16, *React Server Components y Server Actions*.

Las Form Actions se introdujeron para simplificar el proceso de manejo de envíos de formularios y extracción de datos, especialmente al crear aplicaciones full-stack con Server Actions. Además, también pueden ser muy útiles cuando se combinan con algunos Hooks nuevos de React, que se analizarán más adelante en este capítulo.

Así es como se puede manejar el envío de un formulario a través de una Form Action del lado del cliente:

```javascript
function App() { 
  function submitAction(formData) { 
    const credentials = { 
      email: formData.get('email'), 
      password: formData.get('password'), 
    }; 
    console.log(credentials); 
  } 

  return ( 
    <form action={submitAction}> 
      <p> 
        <label htmlFor="email">Email</label> 
        <input type="email" id="email" name="email" /> 
      </p> 
      <p> 
        <label htmlFor="password">Password</label> 
        <input type="password" id="password" name="password" /> 
      </p> 
      <p className="actions"> 
        <button>Login</button> 
      </p> 
    </form> 
  ); 
}
```

A primera vista, este ejemplo puede parecer muy similar al fragmento de código donde se usaron el objeto de evento y `currentTarget` para derivar el `FormData`. Pero si miras más de cerca, verás que hay algunas diferencias clave:
- `handleSubmit` pasó a llamarse `submitAction` y acepta un parámetro llamado `formData` en lugar de `event`.
- El elemento `<form>` ya no tiene la prop `onSubmit`; en su lugar, ahora tiene una **prop `action`** que apunta a la función `submitAction`.

El cambio de nombre de la función es opcional; no existe ningún requisito técnico para nombrar esta función `submitAction` o algo similar. Pero cambiar el nombre tiene sentido porque la función ya no maneja directamente el evento de envío. En su lugar, se utiliza como un valor para la prop `action` recién agregada.

Y de eso se trata precisamente la característica de Form Actions de React: **establecer la prop `action` de un elemento `<form>` en una función que React luego invocará en tu nombre cuando se envíe el formulario**. Sin embargo, a diferencia de cuando se usa la prop `onSubmit`, React evitará el comportamiento predeterminado del navegador y creará un objeto de datos de formulario por ti (y pasará ese objeto como argumento a la función de acción).

Ya no tienes que realizar estos pasos manualmente y, como resultado, el envío del formulario se puede manejar con una cantidad mínima de código.

Por supuesto, si necesitas configurar y administrar los valores de entrada manualmente, o si necesitas interactuar con los campos del formulario en algún momento (por ejemplo, para llamar a `focus()`), aún necesitarás trabajar con el estado o con Refs. Pero si solo estás intentando manejar el envío y obtener los valores ingresados, usar la función de Form Actions puede ser muy útil.

Pero las Form Actions no solo son útiles porque requieran menos código.

#### Actions síncronas frente a asíncronas
Las Form Actions del cliente pueden ser **sincrónicas o asincrónicas**, lo que significa que también puedes usar y devolver una promesa en la función de acción. Por lo tanto, también puedes usar `async / await` con esa función.

Por ejemplo, si tienes un formulario en una aplicación que tiene como objetivo almacenar algunos datos de tareas en el almacenamiento del navegador (a través de la API `localStorage`), puedes hacerlo con una acción sincrónica (ya que `localStorage` es una API sincrónica):

```javascript
function storeTaskAction(formData) { 
  const task = { 
    title: formData.get('title'), 
    body: formData.get('body'), 
    dueDate: formData.get('date') 
  }; 
  localStorage.setItem('daily-task', JSON.stringify(task)); 
}
```

Esta función de acción es sincrónica, ya que no devuelve una promesa ni usa `async / await`. Por lo tanto, como puedes ver, todos los ejemplos de Form Actions hasta ahora han utilizado acciones sincrónicas.

Pero si estás trabajando en un proyecto que necesita enviar datos ingresados a un backend a través de una solicitud HTTP, puedes aprovechar la compatibilidad con código asíncrono:

```javascript
async function storeTodoAction(formData) { 
  const todoTitle = formData.get('title'); 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/todos', 
    { 
      method: 'POST', 
      body: JSON.stringify({ title: todoTitle }), 
      headers: { 
        'Content-type': 'application/json; charset=UTF-8', 
      }, 
    } 
  ); 
  const todo = await response.json(); 
  console.log(todo); 
}
```

En este ejemplo, se agrega la palabra clave `async` delante de la función. Esto convierte la función en una función asíncrona que devolverá una promesa.

Esta flexibilidad que ofrece la función de Form Actions de React es muy útil, ya que te permite realizar una amplia variedad de operaciones al enviar formularios. Sin embargo, es importante tener en cuenta que, por ahora, todas estas acciones siempre se ejecutan en el lado del cliente, es decir, en el navegador del visitante del sitio web. Las acciones del lado del servidor se explorarán en el Capítulo 16.

---

### Sección 4: Detrás de escena: las Actions son transiciones

Antes de profundizar en las Form Actions, puede resultar útil echar un vistazo bajo el capó.

Esto se debe a que, técnicamente, las Actions (es decir, tanto las Client Actions como las Server Actions) en React son las llamadas **transiciones** (*transitions*). Para ser precisos, son **transiciones asíncronas**.

Por tanto, la pregunta es: ¿qué es una transición en React?

En una aplicación de React, una transición es un concepto en el que React garantizará que **algunas actualizaciones de estado que potencialmente consumen mucho tiempo no bloqueen las actualizaciones de la interfaz de usuario**.

Las Form Actions pueden considerarse actualizaciones de estado (potencialmente) lentas; por lo tanto, bajo el capó, React las maneja de manera que el resto de la interfaz de usuario siga respondiendo.

Como resultado, **cualquier llamada de actualización de estado que realices dentro de una función de Form Action solo será procesada por React una vez que esa Form Action haya terminado**. Por ejemplo, el siguiente código, probablemente de forma inesperada, solo actualizará la interfaz de usuario después de tres segundos:

```javascript
import { useState } from 'react'; 

function App() { 
  const [error, setError] = useState(null); 

  async function storeTodoAction(formData) { 
    const todoTitle = formData.get('title'); 
    if (!todoTitle || todoTitle.trim() === '') { 
      setError('Title is required.'); // state update BEFORE delay 
    } 
    // 3s delay to simulate a slow process 
    await new Promise((resolve) => setTimeout(resolve, 3000)); 
    console.log('Submission done!'); 
  } 

  return ( 
    <> 
      <form action={storeTodoAction}> 
        <p> 
          <label htmlFor="title">Title</label> 
          <input type="text" id="title" name="title" /> 
        </p> 
        {error && <p className="errors">{error}</p>} 
        <p className="actions"> 
          <button>Store Todo</button> 
        </p> 
      </form> 
    </> 
  ); 
}
```

A pesar de que el estado `error` se actualiza antes de que comience el retardo, React no volverá a ejecutar la función del componente (y, por lo tanto, no actualizará la interfaz de usuario) antes de que la Form Action en su conjunto haya finalizado. Por lo tanto, el mensaje de error solo aparece en la pantalla después de tres segundos.

**Figura 9.1**: El mensaje de error solo aparece con retraso.

> [!NOTE]
> Puedes encontrar el código de ejemplo completo en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/examples/08-transition](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/examples/08-transition).

---

### Sección 5: Gestión de estado basada en envíos de formularios

Al manejar envíos de formularios, es bastante común que también desees actualizar la interfaz de usuario después del envío. Para acciones asíncronas, donde la operación ejecutada puede tardar un par de segundos (dependiendo de la operación, por supuesto), es posible que incluso desees actualizar la interfaz de usuario durante el envío, mostrando algún estado pendiente mientras se procesa el formulario enviado.

React tiene como objetivo ayudarte con ambos requisitos ofreciendo dos Hooks específicos relacionados con las Form Actions: **`useActionState()`** y **`useFormStatus()`**.

#### Actualización del estado de la interfaz de usuario con `useActionState()`
React proporciona un Hook llamado `useActionState()`, que está diseñado para usarse junto con Form Actions, sin importar si trabajas con Client Actions o Server Actions.

El objetivo de este Hook es **ayudarte a actualizar la interfaz de usuario de la aplicación en función del resultado de una Form Action**.

Esto puede, por ejemplo, ser útil para validar valores de entrada de formularios y mostrar un mensaje de error si hay entradas no válidas. Para realizar esta tarea, el Hook `useActionState()` se puede importar desde el paquete `react` y usarse de la siguiente manera:

```javascript
import { useActionState } from 'react'; 

function App() { 
  async function storeTodoAction(prevState, formData) { 
    const todoTitle = formData.get('title'); 
    if (!todoTitle || todoTitle.trim() === '') { 
      return { 
        error: 'Title must not be empty.', 
      }; 
    } 
    // sending HTTP request etc... 
    return { 
      error: null, 
    }; 
  } 

  const [formState, formAction] = useActionState(storeTodoAction, { 
    error: null, 
  }); 

  return ( 
    <form action={formAction}> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      {formState.error && <p className='errors'> 
        {formState.error} 
      </p>} 
      <p className="actions"> 
        <button>Store Todo</button> 
      </p> 
    </form> 
  ); 
}
```

Al ejecutar esta aplicación de ejemplo, los usuarios verán mensajes de error de validación si hay una entrada no válida.

**Figura 9.2**: Se muestra un mensaje de error al enviar un campo de entrada vacío.

Están sucediendo varias cosas en este ejemplo de código:
1. La función de Form Action se modificó para aceptar **dos parámetros** en lugar de solo uno: un estado anterior (`prevState`) y los datos enviados (`formData`).
2. La Form Action ahora también **devuelve un valor**: un objeto con una clave llamada `error` que contiene un mensaje de error o `null`.
3. Se importa y utiliza el Hook `useActionState()`: recibe la función de Form Action (`storeTodoAction`) como primer argumento, y algún objeto de estado inicial (`{error: null}` en este caso) como segundo argumento.
4. El Hook `useActionState()` también devuelve un valor: un array del cual se desestructuran dos elementos (`formState` y `formAction`).
5. El `formAction` desestructurado reemplaza a `storeTodoAction` como un valor para la prop `action` del elemento `<form>`.
6. `formState` se utiliza para mostrar condicionalmente el valor almacenado en la clave `error` de `formState`.

Entonces, como puedes ver, `useActionState()` es un Hook que espera una función de Form Action (sincrónica o asincrónica) como primer argumento y un estado inicial como segunda entrada. Ese estado inicial es necesario para tener algún estado disponible si el formulario aún no se ha enviado. Después del envío del formulario, el estado inicial será reemplazado por nuevos valores de estado devueltos por la función de Form Action.

Dado que el propósito de `useActionState()` es proporcionar algún valor de estado que se pueda usar para actualizar (partes de) la interfaz de usuario, ese estado derivado se expone a través del valor devuelto por `useActionState()`:

```javascript
const [formState, formAction] = useActionState(storeTodoAction, { 
  error: null, 
} );
```

Ese valor devuelto es un array con exactamente **tres elementos**, en el siguiente orden:
1. **El valor del estado actual**, que es el estado inicial (si el formulario aún no se ha enviado) o el valor de estado devuelto por la función de Form Action.
2. **Una función de Form Action actualizada**, que es esencialmente tu función de acción envuelta por React. Esto es necesario para que React obtenga acceso al valor devuelto por tu función de acción (que es el nuevo estado).
3. **Un valor booleano** que indica si el formulario se está enviando actualmente o no (`pending`). Este tercer elemento no se utiliza en el ejemplo de código anterior y se analizará en la sección *Gestión del estado de interfaz de usuario pendiente con useActionState()* de este capítulo.

Por lo tanto, al usar `useActionState()`, ya no vinculas tu función de acción directamente a la prop `action` del elemento `<form>`. En su lugar, utilizas la función de acción creada por `useActionState()`, es decir, utilizas la función de acción que envuelve a tu función de acción.

Al usar `useActionState()`, también debes ajustar tu función de Form Action porque React llamará a tu función con **dos argumentos** en lugar de solo uno: el estado anterior y los datos del formulario enviados:

```javascript
async function storeTodoAction(prevState, formData) { 
  // ... 
}
```

El estado del formulario anterior se pasa a tu función de acción para que puedas usarlo para derivar tu nuevo estado a partir de él (junto con los datos del formulario enviados). En el ejemplo anterior, este no es el caso: el parámetro de estado anterior no se usa allí. No obstante, debe aceptarse como parámetro.

Sin embargo, ese no es el único cambio realizado en la función de Form Action. En su lugar, ahora también debe devolver un nuevo valor de estado que luego `useActionState()` expondrá a la función del componente (a través del primer elemento en el array devuelto por `useActionState()`):

```javascript
async function storeTodoAction(prevState, formData) { 
  // ... 
  return { error: 'Title must not be empty.' }; 
}
```

Ese valor de estado puede ser cualquier cosa: una cadena, un número, un array, un objeto, etc. En el ejemplo de código anterior, es un objeto con una clave llamada `error` que contiene `null` o un mensaje de error en forma de cadena de texto.

Cada vez que se envía el formulario y, por lo tanto, la función de Form Action se ejecuta o devuelve un valor, `useActionState()` hará que React vuelva a ejecutar la función del componente circundante. Por lo tanto, el estado actualizado se pone a disposición. Si eso te suena similar a `useState()`, ¡tienes razón! `useActionState()` es esencialmente como `useState()`, optimizado para derivar el estado a partir de acciones.

`useActionState()` es, por lo tanto, definitivamente un Hook importante, aunque en realidad no se limita a exponer únicamente los valores devueltos por tus acciones a las funciones de los componentes.

#### Gestión del estado de interfaz de usuario pendiente con `useActionState()`
Considera un escenario en el que tienes una Form Action que tarda un par de segundos en finalizar su operación. Por ejemplo, podrías tener una acción que envía una solicitud a un servidor lento o a través de una conexión a Internet lenta. En tales escenarios, es posible que desees actualizar la interfaz de usuario durante el envío del formulario para mostrarle al usuario que algo está sucediendo.

En el siguiente ejemplo, se llama a una función llamada `saveTodo()` desde dentro de la Form Action. Esa función agrega un retraso deliberado de tres segundos para simular una red o un servidor lento:

```javascript
async function saveTodo(todo) { 
  // dummy function that simulates a slow backend which manages todos 
  await new Promise((resolve) => setTimeout(resolve, 3000)); // delay 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/todos', 
    { 
      method: 'POST', 
      body: JSON.stringify(todo), 
      headers: { 
        'Content-type': 'application/json; charset=UTF-8', 
      }, 
    } 
  ); 
  const fetchedTodo = await response.json(); 
  console.log(fetchedTodo); 
} 

function App() { 
  async function storeTodoAction(prevState, formData) { 
    const todoTitle = formData.get('title'); 
    if (!todoTitle || todoTitle.trim() === '') { 
      return { 
        error: 'Title must not be empty.', 
      }; 
    } 
    await saveTodo({ title: todoTitle }); 
    return { 
      error: null, 
    }; 
  } 
  // same code as before, hence omitted 
}
```

Al utilizar Form Actions, como en este ejemplo, actualizar la interfaz de usuario mientras se gestiona el envío del formulario es relativamente fácil porque `useActionState()` expone un **tercer elemento** en su array devuelto: un valor booleano que indica si la acción se está ejecutando actualmente o no (`pending`).

Por lo tanto, el ejemplo anterior se puede ajustar de la siguiente manera para aprovechar ese valor booleano:

```javascript
function App() { 
  async function storeTodoAction(prevState, formData) { 
    // same code as before, hence omitted 
  } 

  const [formState, formAction, pending] = useActionState( 
    storeTodoAction, 
    { 
      error: null, 
    } 
  ); 

  return ( 
    <form action={formAction}> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      {formState.error && <p className="errors">{formState.error}</p>} 
      <p className="actions"> 
        <button disabled={pending}> 
          {pending ? 'Saving' : 'Store'} Todo 
        </button> 
      </p> 
    </form> 
  ); 
}
```

El elemento `pending` se recupera del array mediante desestructuración y luego se utiliza para deshabilitar el `<button>` y actualizar el texto del botón.

Como resultado, la interfaz de usuario cambia una vez que se envía el formulario, hasta que termina después de tres segundos (en este caso, debido al retraso agregado en la función `saveTodo()` anteriormente).

**Figura 9.3**: El botón está deshabilitado y muestra el texto alternativo `Saving Todo` durante el envío del formulario.

#### Manejo del estado de interfaz de usuario pendiente con `useFormStatus()`
El elemento `pending` devuelto por `useActionState()` es una forma simple y directa, pero no la única, de actualizar la interfaz de usuario mientras se ejecuta una Form Action.

React también ofrece un Hook **`useFormStatus()`** que proporciona información sobre el estado actual de envío del formulario. Para ser precisos, es el paquete **`react-dom`** (¡no `react`!) el que exporta este Hook `useFormStatus()`.

A diferencia de `useActionState()`, `useFormStatus()` **debe llamarse en algún componente anidado que esté envuelto por el elemento `<form>`** cuyo estado de envío te interesa.

Podrías, por ejemplo, crear un componente `SubmitButton` que se defina y utilice como se muestra en este fragmento de código:

```javascript
import { useFormStatus } from 'react-dom'; 
import { saveTodo } from './todos.js'; 

function SubmitButton() { 
  const { pending } = useFormStatus(); 

  return ( 
    <button disabled={pending}> 
      {pending ? 'Saving' : 'Store'} Todo 
    </button> 
  ); 
} 

function App() { 
  async function storeTodoAction(formData) { 
    const todo = { title: formData.get('title') }; 
    await saveTodo(todo); 
  } 

  return ( 
    <form action={storeTodoAction}> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      <p className="actions"> 
        <SubmitButton /> 
      </p> 
    </form> 
  ); 
}
```

En este ejemplo, el código real para enviar la tarea pendiente a un servidor backend se extrae en una función `saveTodo()` separada que se almacena en un archivo `todos.js`. Esa función contiene el mismo código que se mostró en ejemplos anteriores (es decir, envía una solicitud HTTP a JSONPlaceholder). Además, se elimina `useActionState()` para que el código sea un poco más corto y sencillo nuevamente. Sin embargo, puedes utilizar absolutamente `useActionState()` junto con `useFormStatus()`. Por ejemplo, podrías usar `useActionState()` para generar errores de validación mientras administras el estado deshabilitado del botón de envío a través de `useFormStatus()` en un componente anidado separado.

`useFormStatus()` se importa desde `react-dom` y se llama dentro de la función del componente `SubmitButton`. Devuelve un objeto que contiene una propiedad `pending` que produce un valor booleano.

Como se mencionó antes, `useFormStatus()` no se puede usar en el componente donde se renderiza el elemento `<form>`. En su lugar, debe usarse en un componente anidado; por eso el componente `<SubmitButton>` se coloca entre las etiquetas `<form>`.

Además de `pending`, el objeto devuelto por `useFormStatus()` también contiene otras tres propiedades:
- **`data`**: Un objeto `FormData` que contiene los datos con los que se envió el `<form>` principal (es decir, el mismo tipo de datos que recibe la función de Form Action).
- **`method`**: Un valor de cadena que es `'get'` o `'post'`, que refleja el valor en el que se estableció la prop `method` en el elemento `<form>`. Por defecto, es `'get'`.
- **`action`**: Un puntero a la función de Form Action que está conectada al `<form>`.

Si solo te importa el estado `pending`, puedes, por supuesto, usar `useActionState()` o `useFormStatus()`. Trabajar con `useActionState()` tiene la ventaja de que no es necesario crear ningún componente anidado independiente. Por otro lado, crear un componente adicional de este tipo y confiar en `useFormStatus()` puede resultar útil si tienes varios formularios en la página; luego podrías, por ejemplo, reutilizar `<SubmitButton>` en todos esos formularios.

---

### Sección 6: Realización de actualizaciones optimistas (*Optimistic Updates*)

Además de `useActionState()` y `useFormStatus()`, React ofrece un último Hook importante relacionado con formularios y Form Actions: el Hook **`useOptimistic()`**.

La idea detrás de este Hook es que puedes usarlo para **mostrar una interfaz de usuario temporal y optimista mientras se realiza una Form Action asíncrona** (que puede tardar un par de segundos). "Optimista" significa que puedes usar este Hook para renderizar una interfaz de usuario que normalmente solo existiría después de que finalice el envío del formulario (por ejemplo, una lista de tareas pendientes que ya incluye la tarea recién enviada).

El siguiente código de ejemplo administra una lista de tareas pendientes con la ayuda de un elemento `<form>` y una Form Action, pero sin usar `useOptimistic()`:

```javascript
import { useFormStatus } from 'react-dom'; 
import { useState } from 'react'; 

let storedTodos = []; 

export async function saveTodo(todo) { 
  // dummy function that simulates a slow backend which manages todos 
  await new Promise((resolve) => setTimeout(resolve, 3000)); 
  const newTodo = { ...todo, id: new Date().getTime() }; 
  storedTodos = [...storedTodos, newTodo]; 
  return storedTodos; 
} 

function SubmitButton() { 
  // same as before, didn't change, hence omitted here 
} 

function App() { 
  const [todos, setTodos] = useState(storedTodos); 

  async function storeTodoAction(formData) { 
    const todo = { title: formData.get('title') }; 
    const updatedTodos = await saveTodo(todo); // takes 3s 
    setTodos(updatedTodos); 
  } 

  return ( 
    <> 
      <form action={storeTodoAction}> 
        <p> 
          <label htmlFor="title">Title</label> 
          <input type="text" id="title" name="title" /> 
        </p> 
        <p className="actions"> 
          <SubmitButton /> 
        </p> 
      </form> 
      <div id="todos"> 
        <h2>My Todos</h2> 
        {todos.length === 0 && <p>No todos found.</p>} 
        {todos.length > 0 && ( 
          <ul> 
            {todos.map((todo) => ( 
              <li key={todo.id}>{todo.title}</li> 
            ))} 
          </ul> 
        )} 
      </div> 
    </> 
  ); 
}
```

En este ejemplo, dado que la función `saveTodo()` vuelve a tener un retraso deliberado incorporado de tres segundos, el usuario del sitio web ve la lista de tareas desactualizada hasta que se completa el proceso de envío del formulario.

**Figura 9.4**: Sin la actualización optimista, las actualizaciones de la interfaz de usuario se retrasan.

Por tanto, la experiencia del usuario se puede mejorar introduciendo el Hook `useOptimistic()`.

Este Hook requiere dos argumentos y devuelve un array con exactamente dos elementos:

```javascript
const [optimisticState, addOptimistic] = useOptimistic( 
  state, 
  updateFunction 
);
```

- **`state`** (el primer argumento) es el estado del componente que debe estar activo inicialmente o si no hay ninguna Form Action pendiente.
- **`updateFunction`** (el segundo argumento) es una función definida por ti que controla cómo se debe actualizar el estado de manera optimista.
- **`optimisticState`** es el estado actualizado optimistamente que estará activo durante la ejecución de la Form Action.
- **`addOptimistic`** activa la `updateFunction` y te permite pasarle un valor a esa función.

Aplicado al ejemplo anterior, `useOptimistic()` se puede utilizar para gestionar un array de tareas alternativo y actualizado optimistamente que estará activo mientras se ejecute la Form Action. A partir de entonces, el estado regular volverá a estar activo (y actualizará la interfaz de usuario en consecuencia):

```javascript
import { useOptimistic } from 'react'; 
import { saveTodo, getTodos } from './todos.js'; 
import { useState } from 'react'; 

function SubmitButton() { 
  // same code as before, hence omitted 
} 

function App() { 
  const loadedTodos = getTodos(); // initial fetch 
  const [todos, setTodos] = useState(loadedTodos); 

  const [optimisticTodos, addOptimisticTodo] = useOptimistic( 
    todos, 
    (currentState, optimisticValue) => { 
      return [...currentState, { ...optimisticValue, id: 'temp' }]; 
    } 
  ); 

  async function storeTodoAction(formData) { 
    const todo = { title: formData.get('title') }; 
    addOptimisticTodo(todo); 
    const updatedTodos = await saveTodo(todo); 
    setTodos(updatedTodos); 
  } 

  return ( 
    <form action={storeTodoAction}> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      <p className="actions"> 
        <SubmitButton /> 
      </p> 
    </form> 
    <div id="todos"> 
      <h2>My Todos</h2> 
      {optimisticTodos.length === 0 && <p>No todos found.</p>} 
      {optimisticTodos.length > 0 && ( 
        <ul> 
          {optimisticTodos.map((todo) => ( 
            <li key={todo.id}>{todo.title}</li> 
          ))} 
        </ul> 
      )} 
    </div> 
  ); 
}
```

Como puedes ver en este ejemplo, el estado `optimisticTodos` ahora se usa en el código JSX. El valor almacenado en esa constante es el estado `todos` normal (administrado por `useState()`), si la Form Action `storeTodoAction()` no se está ejecutando, o es el array derivado por la función pasada a `useOptimistic()` (como segundo argumento).

**Figura 9.5**: Con `useOptimistic()`, la interfaz de usuario se actualiza inmediatamente después del envío.

El uso del Hook `useOptimistic()` puede, por lo tanto, ayudar a crear una excelente experiencia de usuario donde tu aplicación proporciona retroalimentación instantánea, incluso si algunos procesos lentos aún se están ejecutando en segundo plano. Dado que el estado optimista temporal siempre se reemplazará con el estado regular (es decir, el estado `todos`) una vez finalizado el envío del formulario, **tampoco existe el riesgo de mostrar una interfaz de usuario incorrecta**. Si una operación falla, React reemplazará automáticamente la interfaz de usuario temporalmente incorrecta por la correcta cuando vuelva a utilizar el estado regular.

---

### Sección 7: Resumen y puntos clave

- Los envíos de formularios se pueden manejar escuchando manualmente el evento `submit` a través de la prop `onSubmit`.
- Alternativamente, se pueden utilizar **Form Actions**, es decir, funciones vinculadas a la prop `action` de un elemento `<form>`.
- Al manejar el envío de formularios manualmente (a través de `onSubmit`), puedes extraer los valores de los campos del formulario con la ayuda del estado (`useState()`), Refs (`useRef()`) o creando un objeto `FormData` a partir de `event.currentTarget`.
- Al usar Form Actions, se pasa automáticamente un objeto de datos de formulario (`FormData`) con los valores de entrada de los campos del formulario a la función de acción como parámetro.
- El Hook `useActionState()` se puede utilizar para administrar el estado dependiente del formulario (por ejemplo, mensajes de error de validación).
- `useActionState()` también proporciona un valor booleano `pending` que se puede utilizar para actualizar la interfaz de usuario mientras se procesa la Form Action.
- En los componentes anidados (anidados dentro de `<form>`), se puede llamar al Hook `useFormStatus()` para obtener y utilizar información sobre el estado de envío del formulario principal.
- Para proporcionar actualizaciones rápidas de la interfaz de usuario, incluso cuando se trata de procesos lentos en segundo plano (por ejemplo, solicitudes HTTP lentas), el Hook `useOptimistic()` puede ayudar.

---

### Sección 8: ¿Qué sigue?

Tratar con formularios y manejar las entradas del usuario es una tarea muy común en la mayoría de las aplicaciones web. Las aplicaciones de React, por supuesto, no son una excepción.

Es por eso que React ofrece una amplia variedad de enfoques y patrones posibles que puedes usar para manejar los envíos de formularios y extraer la entrada del usuario. Este capítulo exploró y comparó las dos formas principales de hacer esto: usar la prop `onSubmit` o confiar en las Form Actions (solo disponibles a partir de React 19).

Como se explicó y demostró a lo largo de este capítulo, ambos enfoques son válidos y tienen sus casos de uso. Las preferencias personales así como los requisitos de la aplicación son importantes e influirán en tu decisión.

En este punto del libro, ya conoces todos los conceptos clave de React que necesitas para crear aplicaciones web ricas en funcionalidades. El próximo capítulo mirará detrás de escena de React y explorará cómo funciona internamente. También aprenderás sobre algunas técnicas de optimización comunes que pueden hacer que tus aplicaciones sean más eficientes.

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/09-form-actions/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/09-form-actions/exercises/questions-answers.md):

1. ¿Qué es una "Form Action"?
2. ¿Cómo puedes acceder a la entrada del usuario dentro de una Form Action?
3. ¿Cuál es el propósito del Hook `useActionState()` y cómo se usa?
4. ¿Cuál es el propósito del Hook `useFormStatus()` y cómo se usa?
5. ¿Cuál es la diferencia entre `useActionState()` y `useFormStatus()`?
6. ¿Cuál es el propósito del Hook `useOptimistic()` y cómo se usa?

---

### Sección 10: Aplica lo aprendido

Con las Form Actions en tu repertorio de herramientas de React, tienes otra forma poderosa de manejar los envíos de formularios y extraer las entradas del usuario.

En la siguiente sección, encontrarás una actividad que te permitirá practicar el trabajo con Form Actions y los Hooks relacionados con formularios proporcionados por React. Como siempre, también necesitarás emplear algunos de los conceptos cubiertos en capítulos anteriores (como trabajar con estado o mostrar listas).

#### Actividad 9.1: Gestión de un formulario de comentarios
En esta actividad, tu trabajo consiste en desarrollar sobre una aplicación de formulario de comentarios básica existente y manejar los envíos de formularios con la ayuda de Form Actions. Como parte de esta actividad, debes validar el título y el texto de comentarios enviados y mostrar mensajes de error si se envían valores vacíos. También debes actualizar la lista de elementos de comentarios enviados de manera optimista y deshabilitar el botón de envío mientras la Form Action está en curso.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/activities/practice-1-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start`, en este caso) para usar la versión correcta del código.

Después de descargar el código y ejecutar `npm install` en la carpeta del proyecto para instalar todas las dependencias requeridas, los pasos de la solución son los siguientes:
1. Reemplaza la función controladora `onSubmit` existente con una Form Action; luego, limpia y elimina cualquier código que ya no sea necesario.
2. Deshabilita el botón de envío del formulario mientras se procesa la Form Action.
3. Valida la entrada del usuario y genera los mensajes de error correspondientes con la ayuda del Hook `useActionState()`.
4. Actualiza la lista de elementos de comentarios enviados de manera optimista utilizando el Hook `useOptimistic()`.

El resultado esperado debe parecerse a las siguientes capturas de pantalla:

**Figura 9.6**: Durante el envío del formulario, el botón está deshabilitado, pero el elemento enviado aparece al instante.

**Figura 9.7**: Al enviar valores no válidos, se muestran los mensajes de error correspondientes.

> [!NOTE]
> Encontrarás una solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/09-form-actions/activities/practice-1).
