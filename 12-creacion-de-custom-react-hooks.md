# Parte 1: Fundamentos de React

## Capítulo 12: Creación de Custom React Hooks

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Crear tus propios React Hooks personalizados (*Custom Hooks*).
- Utilizar Hooks personalizados y predeterminados de React en tus componentes.

---

### Sección 1: Introducción

A lo largo de este libro, se ha hecho referencia repetidamente a una característica clave de React en muchas variaciones diferentes: los **React Hooks**.

Los Hooks potencian casi todas las funcionalidades y conceptos centrales que ofrece React: desde la gestión del estado en un solo componente hasta el acceso al estado entre componentes (contexto) en múltiples componentes. Te permiten acceder a elementos JSX a través de refs y te permiten manejar efectos secundarios dentro de las funciones de los componentes.

Sin los Hooks, el React moderno no funcionaría y sería imposible crear aplicaciones ricas en funciones.

Hasta ahora, solo se han presentado y utilizado los Hooks integrados. Sin embargo, también puedes crear tus propios Hooks personalizados (*Custom Hooks*), o puedes usar Hooks personalizados creados por otros desarrolladores (por ejemplo, mediante el uso de bibliotecas de terceros). En este capítulo, aprenderás por qué querrías hacer esto y cómo funciona.

---

### Sección 2: Introducción a los Custom Hooks

Antes de comenzar a crear Hooks personalizados, es muy importante comprender qué son exactamente los Custom Hooks.

En las aplicaciones de React, los Custom Hooks son **funciones regulares de JavaScript** que cumplen con las siguientes condiciones:
1. El nombre de la función comienza con **`use`** (al igual que todos los Hooks integrados comienzan con `use`: `useState()`, `useReducer()`, etc.).
2. La función **llama a otro Hook de React** (uno integrado o uno personalizado; no importa).
3. La función **no devuelve únicamente código JSX** (de lo contrario, sería esencialmente un componente de React), aunque podría devolver algún código JSX, siempre y cuando no sea el único valor devuelto.

Si una función cumple con estas tres condiciones, puede (y debe) denominarse un **Custom (React) Hook**. Por lo tanto, los Custom Hooks son en realidad funciones normales con nombres especiales (que comienzan con `use`) que llaman a otros Hooks (personalizados o integrados) y que no devuelven (solo) código JSX. Si intentas llamar a un Hook (personalizado o integrado) en algún otro lugar (por ejemplo, fuera de cualquier función o en una función normal que no sea un Hook), es posible que recibas una advertencia (según la configuración de tu proyecto; ver más abajo).

Por ejemplo, la siguiente función utiliza el Hook `useEffect()` pero tiene un nombre que no comienza con `use`. Por lo tanto, no está alineada con la recomendación oficial de nomenclatura:

```javascript
function sendAnalyticsEvent(event) { 
  useEffect(() => { 
    fetch('https://my-analytics-backend.com/events', { 
      method: 'POST', 
      body: JSON.stringify(event) 
    }) 
  }, []); 
}
```

En proyectos que realizan análisis estático de código (*linting*) para verificar violaciones de reglas, este código generaría una advertencia porque esta función no califica como un Custom Hook (debido a su nombre).

**Figura 12.1**: React se queja si llamas a una función de Hook en el lugar equivocado.

Como indica la advertencia, los Hooks, ya sean personalizados o integrados, solo deben llamarse dentro de las funciones de los componentes. Y, aunque el mensaje de advertencia no lo menciona explícitamente, **también se pueden llamar dentro de Custom Hooks**.

Por lo tanto, si la función `sendAnalyticsEvent()` se renombra a `useSendAnalyticsEvent()`, la advertencia desaparece ya que la función ahora califica como un Custom Hook.

Aunque técnicamente no es una regla estricta impuesta por el propio React, es una fuerte recomendación seguir esta convención de nomenclatura.

Poder crear Custom Hooks es una característica extremadamente importante porque significa que **puedes crear funciones reutilizables que no sean componentes y que puedan contener lógica de estado** (a través de `useState()` o `useReducer()`), **manejar efectos secundarios en tus funciones de Custom Hooks reutilizables** (a través de `useEffect()`), **o usar cualquier otro Hook de React**. Con funciones normales que no son Hooks, nada de esto sería posible y, por lo tanto, no podrías externalizar ninguna lógica que involucre un Hook de React en tales funciones.

De esta manera, los Custom Hooks complementan el concepto de los componentes de React:
- Mientras que los componentes de React son **bloques de construcción de UI reutilizables** (que pueden contener lógica con estado),
- Los Custom Hooks son **fragmentos de lógica reutilizables** que se pueden usar en las funciones de tus componentes.

Por lo tanto, los Custom Hooks te ayudan a reutilizar la lógica compartida entre componentes. Por ejemplo, los Custom Hooks te permiten externalizar la lógica para enviar una solicitud HTTP y manejar los estados relacionados (carga, error, etc.).

#### ¿Por qué crearías Custom Hooks?
En el capítulo anterior (Capítulo 11, *Trabajando con Estado Complejo*), cuando se introdujo el Hook `useReducer()`, se proporcionó un ejemplo en el que el Hook se utilizó para enviar una solicitud HTTP. Aquí está el código final relevante nuevamente:

```javascript
const initialHttpState = { 
  data: null, 
  isLoading: false, 
  error: null, 
}; 

function httpReducer(state, action) { 
  if (action.type === 'FETCH_START') { 
    return { 
      ...state, // copying the existing state 
      isLoading: state.data ? false : true, 
      error: null, 
    }; 
  } 
  if (action.type === 'FETCH_ERROR') { 
    return { 
      data: null, 
      isLoading: false, 
      error: action.payload, 
    }; 
  } 
  if (action.type === 'FETCH_SUCCESS') { 
    return { 
      data: action.payload, 
      isLoading: false, 
      error: null, 
    }; 
  } 
  return initialHttpState; // default value for unknown actions 
} 

function App() { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch( 
        'https://jsonplaceholder.typicode.com/posts' 
      ); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, []); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return ( 
    <> 
      <header> 
        <h1>Complex State Blog</h1> 
        <button onClick={fetchPosts}>Load Posts</button> 
      </header> 
      {httpState.isLoading && <p>Loading...</p>} 
      {httpState.error && <p>{httpState.error}</p>} 
      {httpState.data && <BlogPosts posts={httpState.data} />} 
    </> 
  ); 
};
```

En este ejemplo de código, se envía una solicitud HTTP cada vez que el componente `App` se renderiza por primera vez. La solicitud HTTP obtiene una lista de publicaciones (ficticias). Hasta que finaliza la solicitud, se muestra un mensaje de carga (`<p>Loading…</p>`) al usuario. Si hay un error, se muestra un mensaje de error.

Como puedes ver, se debe escribir una gran cantidad de código para manejar este caso de uso relativamente básico. Y, especialmente en aplicaciones de React más grandes, es muy probable que múltiples componentes necesiten enviar solicitudes HTTP. Probablemente no necesitarán enviar exactamente la misma solicitud a la misma URL (`https://jsonplaceholder.typicode.com/posts`, en este ejemplo), pero definitivamente es posible que diferentes componentes obtengan diferentes datos de diferentes URLs.

Por lo tanto, casi exactamente el mismo código debe escribirse una y otra vez en múltiples componentes. Y no es solo el código para enviar la solicitud HTTP (es decir, la función envuelta por `useCallback()`); en cambio, la gestión del estado relacionada con HTTP (realizada mediante `useReducer()`, en este ejemplo), así como la inicialización de la solicitud mediante `useEffect()`, deben repetirse en todos esos componentes.

Y ahí es donde los Custom Hooks vienen al rescate. Los Custom Hooks te ayudan a evitar esta repetición al permitirte crear "fragmentos de lógica" reutilizables y potencialmente con estado que se pueden compartir entre componentes.

#### Un primer Custom Hook
Antes de explorar escenarios avanzados y resolver el problema de la solicitud HTTP mencionado anteriormente, aquí hay un ejemplo más básico de un primer Custom Hook:

```javascript
import { useState } from 'react'; 

function useCounter() { 
  const [counter, setCounter] = useState(0); 

  function increment() { 
    setCounter(oldCounter => oldCounter + 1); 
  }; 

  function decrement() { 
    setCounter(oldCounter => oldCounter - 1); 
  }; 

  return { counter, increment, decrement }; 
}; 

export default useCounter;
```

Este código se puede almacenar en un archivo llamado `use-counter.js` dentro de una carpeta `hooks/`, aunque ambos nombres dependen totalmente de ti. No hay reglas con respecto al nombre del archivo o de la carpeta (o, en general, al lugar donde almacenas este código). La extensión del archivo es `.js` en lugar de `.jsx` ya que este archivo no contiene código JSX.

Como puedes ver, `useCounter` es una función regular de JavaScript. El nombre de la función comienza con `use` y, por lo tanto, esta función califica como un Custom Hook (lo que significa que no recibirás ningún mensaje de advertencia al usar otros Hooks dentro de ella).

Dentro de `useCounter()`, se administra un estado de contador a través de `useState()`. El estado se modifica mediante dos funciones anidadas (`increment` y `decrement`), y el estado, así como las funciones, son devueltos por `useCounter` (agrupados en un objeto de JavaScript).

> [!NOTE]
> La sintaxis utilizada para agrupar `counter`, `increment` y `decrement` utiliza una característica estándar de JavaScript: los nombres abreviados de propiedades (*shorthand property names*).
> Si el nombre de una propiedad en un objeto coincide literalmente con el nombre de la variable cuyo valor se asigna a la propiedad, puedes usar esta notación más corta.
> En lugar de escribir `{ counter: counter, increment: increment, decrement: decrement }`, puedes usar la notación abreviada `{ counter, increment, decrement }` que se muestra en el fragmento anterior.

Este Custom Hook se puede almacenar en un archivo separado (por ejemplo, en una carpeta `hooks` dentro del proyecto de React, como `src/hooks/use-counter.js`). A partir de entonces, se puede usar en cualquier componente de React y puedes usarlo en tantos componentes de React como sea necesario.

Por ejemplo, los siguientes dos componentes (`Demo1` y `Demo2`) podrían usar este Hook `useCounter` de la siguiente manera:

```javascript
import useCounter from './hooks/use-counter.js'; 

function Demo1() { 
  const { counter, increment, decrement } = useCounter(); 

  return ( 
    <> 
      <p>{counter}</p> 
      <button onClick={increment}>Inc</button> 
      <button onClick={decrement}>Dec</button> 
    </> 
  ); 
}; 

function Demo2() { 
  const { counter, increment, decrement } = useCounter(); 

  return ( 
    <> 
      <p>{counter}</p> 
      <button onClick={increment}>Inc</button> 
      <button onClick={decrement}>Dec</button> 
    </> 
  ); 
}; 

function App() { 
  return ( 
    <main> 
      <Demo1 /> 
      <Demo2 /> 
    </main> 
  ); 
}; 

export default App;
```

> [!NOTE]
> Encontrarás el código de ejemplo completo en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/examples/01-first-hook](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/examples/01-first-hook).

Los componentes `Demo1` y `Demo2` ejecutan `useCounter()` dentro de las funciones de sus componentes. La función `useCounter()` se llama como una función normal porque es una función estándar de JavaScript.

Dado que el Hook `useCounter` devuelve un objeto con tres propiedades (`counter`, `increment` y `decrement`), `Demo1` y `Demo2` utilizan la desestructuración de objetos para almacenar los valores de las propiedades en constantes locales. Estos valores luego se utilizan en el código JSX para mostrar el valor del contador y conectar los dos elementos `<button>` a las funciones `increment` y `decrement`.

Después de presionar los botones un par de veces cada uno, la interfaz de usuario resultante podría verse así:

**Figura 12.2**: Dos contadores independientes.

En esta captura de pantalla, también puedes ver un comportamiento muy interesante e importante de los Custom Hooks: **si el mismo Custom Hook con estado se usa en múltiples componentes, cada componente obtiene su propio estado**. El estado del contador no se comparte. El componente `Demo1` gestiona su propio estado de contador (a través del Custom Hook `useCounter()`), y lo mismo ocurre con el componente `Demo2`.

---

### Sección 3: Custom Hooks: Una característica flexible

Los dos estados independientes de `Demo1` y `Demo2` muestran una característica muy importante de los Custom Hooks: **los utilizas para compartir lógica, no estado**. Si necesitaras compartir el estado entre componentes, lo harías con React Context (consulta el capítulo anterior).

Al utilizar Hooks, cada componente utiliza su propia "instancia" (o "versión") de ese Hook. Siempre es la misma lógica, pero cualquier estado o efecto secundario manejado por un Hook se maneja por componente.

También vale la pena señalar que los Custom Hooks pueden tener estado, pero no es obligatorio. Pueden gestionar el estado mediante `useState()` o `useReducer()`, pero también podrías crear Custom Hooks que solo manejen efectos secundarios (sin ninguna gestión de estado).

Solo hay una cosa que debes hacer implícitamente en los Custom Hooks: **debes usar algún otro Hook de React (personalizado o integrado)**. Esto se debe a que si no incluyes ningún otro Hook, no habría necesidad de crear un Custom Hook en primer lugar. Un Custom Hook es solo una función regular de JavaScript (con un nombre que comienza con `use`) con la que puedes usar otros Hooks. Si no necesitas usar ningún otro Hook, simplemente puedes crear una función normal de JavaScript con un nombre que no comience con `use`.

También tienes mucha flexibilidad con respecto a la lógica dentro del Hook, sus parámetros y el valor que devuelve. Con respecto a la lógica del Hook, puedes agregar tanta lógica como sea necesaria. Puedes administrar ningún estado o múltiples valores de estado. Puedes incluir otros Custom Hooks o solo usar Hooks integrados. Puedes administrar múltiples efectos secundarios, trabajar con refs o realizar cálculos complejos. No hay restricciones sobre lo que se puede hacer en un Custom Hook.

#### Custom Hooks y parámetros
También puedes aceptar y utilizar parámetros en tus funciones de Custom Hooks. Por ejemplo, el Hook `useCounter` de la sección *Un primer Custom Hook* se puede ajustar para tomar un valor de contador inicial y valores separados por los cuales el contador debe incrementarse o decrementarse, como se muestra en el siguiente fragmento:

```javascript
import { useState } from 'react'; 

function useCounter(initialValue, incVal, decVal) { 
  const [counter, setCounter] = useState(initialValue); 

  function increment() { 
    setCounter(oldCounter => oldCounter + incVal); 
  }; 

  function decrement() { 
    setCounter(oldCounter => oldCounter - decVal); 
  }; 

  return { counter, increment, decrement }; 
}; 

export default useCounter;
```

En este ejemplo ajustado, el parámetro `initialValue` se utiliza para establecer el estado inicial a través de `useState(initialValue)`. Los parámetros `incVal` y `decVal` se utilizan en las funciones `increment` y `decrement` para cambiar el estado del contador con diferentes valores.

Por supuesto, una vez que se utilizan parámetros en un Custom Hook, se deben proporcionar valores de parámetros adecuados cuando se llama al Custom Hook en una función de componente (o en otro Custom Hook). Por lo tanto, el código de los componentes `Demo1` y `Demo2` también debe ajustarse, por ejemplo, de esta manera:

```javascript
function Demo1() { 
  const { counter, increment, decrement } = useCounter(1, 2, 1); 

  return ( 
    <> 
      <p>{counter}</p> 
      <button onClick={increment}>Inc</button> 
      <button onClick={decrement}>Dec</button> 
    </> 
  ); 
}; 

function Demo2() { 
  const { counter, increment, decrement } = useCounter(0, 1, 2); 

  return ( 
    <> 
      <p>{counter}</p> 
      <button onClick={increment}>Inc</button> 
      <button onClick={decrement}>Dec</button> 
    </> 
  ); 
};
```

> [!NOTE]
> También puedes encontrar este código en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/examples/02-parameters](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/examples/02-parameters).

Ahora, ambos componentes pasan diferentes valores de parámetros a la función del Hook `useCounter`. Por lo tanto, pueden reutilizar el mismo Hook y su lógica interna de forma dinámica.

#### Custom Hooks y valores de retorno
Como se muestra con `useCounter`, los Custom Hooks pueden devolver valores. Y esto es importante: **pueden devolver valores, pero no es obligatorio**. Si creas un Custom Hook que solo maneja algunos efectos secundarios (a través de `useEffect()`), no tienes que devolver ningún valor (porque probablemente no haya ningún valor que deba devolverse).

Pero si necesitas devolver un valor, tú decides qué tipo de valor deseas devolver. Podrías devolver un solo número o una cadena. Si tu Hook debe devolver múltiples valores (como lo hace `useCounter`), puedes agrupar estos valores en un array o un objeto. También puedes devolver arrays que contienen objetos o viceversa. En resumen, puedes devolver cualquier cosa: después de todo, es una función normal de JavaScript.

Algunos Hooks integrados como `useState()` y `useReducer()` devuelven arrays (con un número fijo de elementos). `useRef()`, por otro lado, devuelve un objeto (que siempre tiene una propiedad `current`). `useEffect()` no devuelve nada. Por lo tanto, tus Hooks pueden devolver lo que desees.

Por ejemplo, el Hook `useCounter` anterior se podría reescribir para devolver un array en su lugar:

```javascript
import { useState } from 'react'; 

function useCounter(initialValue, incVal, decVal) { 
  const [counter, setCounter] = useState(initialValue); 

  function increment() { 
    setCounter((oldCounter) => oldCounter + incVal); 
  } 

  function decrement() { 
    setCounter((oldCounter) => oldCounter - decVal); 
  } 

  return [counter, increment, decrement]; 
} 

export default useCounter;
```

Para usar los valores devueltos, los componentes `Demo1` y `Demo2` deben cambiar de la desestructuración de objetos a la desestructuración de arrays, de la siguiente manera:

```javascript
function Demo1() { 
  const [counter, increment, decrement] = useCounter(1, 2, 1); 

  return ( 
    <> 
      <p>{counter}</p> 
      <button onClick={increment}>Inc</button> 
      <button onClick={decrement}>Dec</button> 
    </> 
  ); 
} 

function Demo2() { 
  const [counter, increment, decrement] = useCounter(0, 1, 2); 

  return ( 
    <> 
      <p>{counter}</p> 
      <button onClick={increment}>Inc</button> 
      <button onClick={decrement}>Dec</button> 
    </> 
  ); 
}
```

Los dos componentes se comportan como antes, por lo que puedes decidir qué estructura de valor de retorno prefieres.

> [!NOTE]
> Este código terminado también se puede encontrar en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/examples/03-return-values](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/examples/03-return-values).

---

### Sección 4: Un ejemplo más complejo

Los ejemplos anteriores fueron deliberadamente sencillos. Ahora que los conceptos básicos de los Custom Hooks están claros, tiene sentido sumergirse en un ejemplo un poco más avanzado y realista.

Considera el ejemplo de solicitud HTTP del comienzo de este capítulo:

```javascript
const initialHttpState = { 
  data: null, 
  isLoading: false, 
  error: null, 
}; 

function httpReducer(state, action) { 
  if (action.type === 'FETCH_START') { 
    return { 
      ...state, // copying the existing state 
      isLoading: state.data ? false : true, 
      error: null, 
    }; 
  } 
  if (action.type === 'FETCH_ERROR') { 
    return { 
      data: null, 
      isLoading: false, 
      error: action.payload, 
    }; 
  } 
  if (action.type === 'FETCH_SUCCESS') { 
    return { 
      data: action.payload, 
      isLoading: false, 
      error: null, 
    }; 
  } 
  return initialHttpState; // default value for unknown actions 
} 

function App() { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch( 
        'https://jsonplaceholder.typicode.com/posts' 
      ); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, []); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return ( 
    <> 
      <header> 
        <h1>Complex State Blog</h1> 
        <button onClick={fetchPosts}>Load Posts</button> 
      </header> 
      {httpState.isLoading && <p>Loading...</p>} 
      {httpState.error && <p>{httpState.error}</p>} 
      {httpState.data && <BlogPosts posts={httpState.data} />} 
    </> 
  ); 
};
```

En ese ejemplo, toda la lógica de `useReducer()` (incluida la función reductora, `httpReducer`) y la llamada a `useEffect()` se pueden externalizar en un Custom Hook. El resultado sería un componente `App` muy limpio y un Hook reutilizable que también podría usarse en otros componentes.

#### Construcción de una primera versión del Custom Hook
Este Custom Hook podría llamarse `useFetch` (ya que obtiene datos) y podría almacenarse en `hooks/use-fetch.js`. Por supuesto, tanto el nombre del Hook como la ruta de almacenamiento del archivo dependen de ti. Así es como podría verse la primera versión de `useFetch`:

```javascript
import { useCallback, useEffect, useReducer } from 'react'; 

const initialHttpState = { 
  data: null, 
  isLoading: false, 
  error: null, 
}; 

function httpReducer(state, action) { 
  // same reducer code as before 
} 

function useFetch() { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch( 
        'https://jsonplaceholder.typicode.com/posts' 
      ); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, []); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 
} 

export default useFetch;
```

Ten en cuenta que esta no es la versión final.

En esta primera versión, el Hook `useFetch` contiene la lógica de `useReducer()` y `useEffect()`. Vale la pena señalar que la función `httpReducer` se crea fuera de `useFetch`. Esto garantiza que la función no se vuelva a crear innecesariamente cuando `useFetch()` se vuelva a ejecutar (lo que sucederá a menudo, ya que se llama cada vez que se reevalúa el componente que usa este Hook). Por lo tanto, la función `httpReducer` solo se creará una vez (durante toda la vida útil de la aplicación), y esa misma instancia de función será compartida por todos los componentes que usen `useFetch`.

Dado que `httpReducer` es una **función pura** (es decir, siempre produce nuevos valores de retorno que se basan puramente en los valores de los parámetros), compartir esta instancia de función es seguro y no causará ningún error inesperado. Si `httpReducer` almacenara o manipulara valores que no se basan en las entradas de la función, debería crearse dentro de `useFetch`. De esta manera, evitas que múltiples componentes manipulen y utilicen accidentalmente valores compartidos.

Sin embargo, esta versión del Hook `useFetch` tiene dos grandes problemas:
1. Actualmente, **no se devuelve ningún valor**. Por lo tanto, los componentes que usan este Hook no tendrán acceso a los datos obtenidos ni al estado de carga.
2. **La URL de la solicitud HTTP está codificada de forma fija (*hardcoded*)** dentro de `useFetch`. Como resultado, todos los componentes que usen este Hook enviarán el mismo tipo de solicitud a la misma URL.

Por lo tanto, para mejorar este Hook, se deben abordar estos dos problemas, comenzando con el primero.

#### Hacer que el Hook sea útil devolviendo valores
El primer problema se puede resolver devolviendo los datos obtenidos (o `undefined`, si aún no se obtuvieron datos), el valor del estado de carga y el valor del error. Dado que estos valores son exactamente los valores que componen el objeto `httpState` devuelto por `useReducer()`, `useFetch` simplemente puede devolver todo ese objeto `httpState`, como se muestra aquí:

```javascript
// httpReducer function and initial state did not change, 
// hence omitted here 
function useFetch() { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch( 
        'https://jsonplaceholder.typicode.com/posts' 
      ); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, []); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return httpState; 
}
```

Lo único que cambió en este fragmento de código es la última línea de la función `useFetch`. Con `return httpState`, el estado administrado por `useReducer()` (y por lo tanto por la función `httpReducer`) es devuelto por el Custom Hook.

Con ese primer problema solucionado, el siguiente paso es hacer que el Hook sea más reutilizable.

#### Mejorar la reutilización aceptando un parámetro de entrada
Para solucionar el segundo problema (es decir, la URL fija), se debe agregar un parámetro a `useFetch`:

```javascript
// httpReducer function and initial state did not change, hence omitted here 
function useFetch(url) { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch(url); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, [url]); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return httpState; 
}
```

En este fragmento, se agregó el parámetro `url` a `useFetch`. Este valor de parámetro se utiliza luego dentro del bloque `try` al llamar a `fetch(url)`. Ten en cuenta que `url` también se agregó como una dependencia al array de dependencias de `useCallback()`.

Dado que `useCallback()` se envuelve alrededor de la función de obtención (para evitar bucles infinitos por parte de `useEffect()`), cualquier valor externo utilizado dentro de `useCallback()` debe agregarse a su array de dependencias. Dado que `url` es un valor externo (lo que significa que no está definido dentro de la función envuelta), debe agregarse. Esto también tiene sentido lógicamente: si el parámetro `url` cambiara (es decir, si el componente que usa `useFetch` lo cambia), se debería enviar una nueva solicitud HTTP.

Esta versión final del Hook `useFetch` ahora se puede usar en todos los componentes para enviar solicitudes HTTP a diferentes URLs y usar los valores de estado HTTP según lo requieran los componentes.

Por ejemplo, el componente `App` puede usar `useFetch` de esta manera:

```javascript
import BlogPosts from './components/BlogPosts.jsx'; 
import useFetch from './hooks/use-fetch.js'; 

function App() { 
  const { data, isLoading, error } = useFetch( 
    'https://jsonplaceholder.typicode.com/posts' 
  ); 

  return ( 
    <> 
      <header> 
        <h1>Complex State Blog</h1> 
      </header> 
      {isLoading && <p>Loading...</p>} 
      {error && <p>{error}</p>} 
      {data && <BlogPosts posts={data} />} 
    </> 
  ); 
} 

export default App;
```

El componente importa y llama a `useFetch()` (con la URL apropiada como argumento) y utiliza la desestructuración de objetos para obtener las propiedades `data`, `isLoading` y `error` del objeto `httpState`. Estos valores luego se utilizan en el código JSX.

Por supuesto, el Hook `useFetch` también podría devolver un puntero a la función `fetchPosts` (además de `httpState`) para permitir que componentes como el componente `App` activen manualmente una nueva solicitud, como se muestra aquí:

```javascript
// httpReducer function and initial state did not change, hence omitted here 
function useFetch(url) { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch(url); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, [url]); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return [ 
    httpState, 
    fetchPosts 
  ]; 
}
```

En este ejemplo, se modificó la sentencia `return`. En lugar de devolver solo `httpState`, `useFetch` ahora devuelve un array que contiene el objeto `httpState` y un puntero a la función `fetchPosts`. Alternativamente, `httpState` y `fetchPosts` podrían haberse fusionado en un objeto (en lugar de un array).

En el componente `App`, `useFetch` ahora se podría usar de esta manera:

```javascript
import BlogPosts from './components/BlogPosts.jsx'; 
import useFetch from './hooks/use-fetch.js'; 

function App() { 
  const [{ data, isLoading, error }, fetchPosts] = useFetch( 
    'https://jsonplaceholder.typicode.com/posts' 
  ); 

  return ( 
    <> 
      <header> 
        <h1>Complex State Blog</h1> 
        <button onClick={fetchPosts}>Load Posts</button> 
      </header> 
      {isLoading && <p>Loading...</p>} 
      {error && <p>{error}</p>} 
      {data && <BlogPosts posts={data} />} 
    </> 
  ); 
} 

export default App;
```

El componente `App` utiliza la desestructuración de arrays y objetos combinada para extraer los valores devueltos (y los valores anidados en el objeto `httpState`). Luego se utiliza un elemento `<button>` recién agregado para activar la función `fetchPosts`.

Este ejemplo muestra de manera efectiva cómo los Custom Hooks pueden dar lugar a funciones de componentes mucho más concisas al permitir una fácil reutilización de la lógica, con o sin estado o efectos secundarios.

Además, los Hooks también pueden habilitar algunos patrones interesantes, por ejemplo, relacionados con la Context API de React.

---

### Sección 5: Uso de Custom Hooks para el acceso a Context

Como se insinuó en el capítulo anterior, en la sección *Externalización de la lógica de Context en componentes separados*, puedes usar Custom Hooks para mejorar el proceso de consumo de valores de contexto en los componentes.

Por ejemplo, si proporcionas algún contexto llamado `BookmarkContext` (por ejemplo, a través de un componente `<BookmarkContextProvider>`), puedes acceder a este valor de contexto dentro de los componentes de la siguiente manera:

```javascript
import { use } from 'react'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function BookmarkSummary() { 
  const bookmarkCtx = use(BookmarkContext); 
  // other component code, including returned JSX code 
}
```

Sin embargo, en lugar de acceder directamente al valor del contexto de esta manera, también podrías crear el siguiente Custom Hook (por ejemplo, almacenado en un archivo `store/use-bookmark-context.js`):

```javascript
import { use } from 'react'; 
import BookmarkContext from './bookmark-context.jsx'; 

function useBookmarkContext() { 
  const bookmarkCtx = use(BookmarkContext); 
  return bookmarkCtx; 
} 

export default useBookmarkContext;
```

Pero, por supuesto, este Hook no proporciona realmente ninguna ventaja en comparación con consumir directamente el valor del contexto en un componente a través de `use()`.

Eso cambia una vez que enriqueces este Custom Hook con una lógica más útil, por ejemplo, con el **manejo de errores si se usa en un lugar donde el contexto no está disponible**:

```javascript
function useBookmarkContext() { 
  const bookmarkCtx = use(BookmarkContext); 
  if(!bookmarkCtx) { 
    throw new Error('BookmarkContext must be provided!') 
  } 
  return bookmarkCtx; 
}
```

Este Hook luego se puede utilizar en tus componentes para obtener el valor del contexto de esta manera:

```javascript
import useBookmarkContext from '../../store/use-bookmark-context.js'; 

function BookmarkSummary() { 
  const bookmarkCtx = useBookmarkContext(); 
  // other component code, including returned JSX code 
}
```

Por lo tanto, este no es solo otro ejemplo de un Custom Hook, sino también **un patrón común que debes conocer**. Es un patrón que se utiliza en muchos proyectos de React, ya que garantiza que no intentes utilizar accidentalmente el valor del contexto en un lugar donde no es accesible (es decir, en un componente que no está envuelto por `BookmarkContextProvider`).

Por supuesto, no es un patrón obligatorio. Pero es algo que podrías considerar usar para obtener un error temprano si intentas acceder a tu contexto en el lugar equivocado. Si estás distribuyendo una biblioteca que expone algún contexto, es un patrón especialmente útil, ya que advierte a los usuarios de tu biblioteca en caso de que olviden proporcionar el contexto.

---

### Sección 6: Resumen y puntos clave

- Puedes crear **Custom Hooks** para externalizar y reutilizar lógica que depende de otros Hooks integrados o personalizados.
- Los Custom Hooks son **funciones regulares de JavaScript** con nombres que comienzan con `use`.
- Los Custom Hooks pueden llamar a cualquier otro Hook.
- Por lo tanto, los Custom Hooks pueden, por ejemplo, gestionar el estado o realizar efectos secundarios.
- Todos los componentes pueden usar Custom Hooks simplemente llamándolos como cualquier otro Hook (integrado).
- Cuando múltiples componentes usan el mismo Custom Hook, cada componente recibe su propia "instancia" (es decir, su propio valor de estado, etc.).
- Dentro de los Custom Hooks, puedes aceptar cualquier valor de parámetro y devolver cualquier valor de tu elección.

---

### Sección 7: ¿Qué sigue?

Los Custom Hooks son una característica clave de React, ya que te ayudan a escribir componentes más limpios y a reutilizar la lógica (con estado) en ellos. Especialmente al crear aplicaciones de React más complejas (que constan de docenas o incluso cientos de componentes), los Custom Hooks pueden dar lugar a un código enormemente más fácil de gestionar.

Combinados con componentes, props, estado (a través de `useState()` o `useReducer()`), efectos secundarios y todos los demás conceptos cubiertos en este y en capítulos anteriores, ahora tienes una base muy sólida que te permite crear aplicaciones de React listas para producción. Por lo tanto, ahora estás preparado para sumergirte en conceptos más avanzados de React, así como en paquetes de terceros cruciales que debes conocer.

Por ejemplo, la mayoría de las aplicaciones de React no constan de una sola página; en su lugar, al menos en la mayoría de los sitios web, los usuarios deberían poder cambiar entre varias páginas. Por ejemplo, una tienda en línea tiene una lista de productos, páginas de detalles de productos, una página de carrito de compras y muchas otras páginas.

El próximo capítulo, por lo tanto, explorará cómo puedes crear tales aplicaciones de múltiples páginas con React y el popular paquete de terceros **React Router**.

---

### Sección 8: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/12-custom-hooks/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/12-custom-hooks/exercises/questions-answers.md):

1. ¿Cuál es la definición de un Custom Hook?
2. ¿Qué característica especial se puede utilizar dentro de un Custom Hook?
3. ¿Qué sucede cuando múltiples componentes utilizan el mismo Custom Hook?
4. ¿Cómo se pueden hacer más reutilizables los Custom Hooks?

---

### Sección 9: Aplica lo aprendido

Aplica tus conocimientos sobre Custom Hooks a problemas prácticos.

#### Actividad 12.1: Crear un Custom Hook de entrada de teclado
En esta actividad, tu tarea es refactorizar un componente proporcionado para que sea más conciso y ya no contenga ninguna lógica de estado o de efectos secundarios. En su lugar, debes crear un Custom Hook que contenga esa lógica. Este Hook luego podría usarse potencialmente en otras áreas de la aplicación de React también.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/activities/practice-1-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start`, en este caso) para usar la versión correcta del código.

El proyecto proporcionado también utiliza muchas de las funciones cubiertas en capítulos anteriores. Tómate tu tiempo para analizarlo y comprender el código proporcionado. Esta es una gran práctica y te permite ver muchos conceptos clave en acción.

Una vez que hayas descargado el código y ejecutado `npm install` en la carpeta del proyecto para instalar todas las dependencias requeridas, puedes iniciar el servidor de desarrollo mediante `npm run dev`. Como resultado, al visitar `localhost:5173`, deberías ver la siguiente interfaz de usuario:

**Figura 12.3**: El proyecto inicial en ejecución.

Para completar la actividad, los pasos de la solución son los siguientes:
1. Crea un nuevo archivo de Custom Hook (por ejemplo, en la carpeta `src/hooks`) y crea una función de Hook en ese archivo.
2. Mueve la lógica de efectos secundarios y de gestión de estado a esa nueva función de Hook.
3. Haz que el Custom Hook sea más reutilizable aceptando y utilizando un parámetro que controle qué teclas están permitidas.
4. Devuelve el estado gestionado por el Custom Hook.
5. Utiliza el Custom Hook y su valor devuelto en el componente `App`.

La interfaz de usuario debe ser la misma una vez que hayas completado la actividad, pero el código del componente `App` debe cambiar. Después de finalizar la actividad, `App` debería contener únicamente este código:

```javascript
function App() { 
  const pressedKey = useKeyEvent(['s', 'c', 'p']); // this is your Hook! 
  let output = ''; 

  if (pressedKey === 's') { 
    output = ''; 
  } else if (pressedKey === 'c') { 
    output = ''; 
  } else if (pressedKey === 'p') { 
    output = ''; 
  } 

  return ( 
    <main> 
      <h1>Press a key!</h1> 
      <p> 
        Supported keys: <kbd>s</kbd>, <kbd>c</kbd>, <kbd>p</kbd> 
      </p> 
      <p id="output">{output}</p> 
    </main> 
  ); 
}
```

> [!NOTE]
> Todos los archivos de código utilizados para esta actividad, y una solución de ejemplo, se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/12-custom-hooks/activities/practice-1).
