# Parte 2: Construcción de Aplicaciones React Complejas

## Capítulo 17: Entendiendo React Suspense y el Hook `use()`

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Describir el propósito y la funcionalidad de la característica Suspense de React.
- Usar Suspense con RSCs para mostrar contenido alternativo (*fallback*) a un nivel granular.
- Usar Suspense para componentes de cliente a través del Hook `use()` de React.
- Aplicar diferentes estrategias de Suspense para la obtención de datos y el contenido alternativo.

---

### Sección 1: Introducción

En el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, en la sección *Reducción del tamaño de los paquetes mediante división de código (Lazy Loading)*, aprendiste sobre el componente `<Suspense>` de React y cómo se puede utilizar en el contexto de la carga diferida (*lazy loading*) y la división de código (*code splitting*) para mostrar contenido alternativo mientras se descarga un paquete de código.

Como se explicó allí, el propósito del componente Suspense es simplificar el proceso de mostrar contenido alternativo, lo que, a su vez, puede conducir a una mejor experiencia de usuario. Dado que quedarse mirando contenido desactualizado o una página en blanco no es algo que la mayoría de los usuarios aprecien, tener una función incorporada que muestre contenido alternativo resulta muy conveniente.

En este capítulo, aprenderás que el componente Suspense de React no se limita a ser utilizado para la división de código. En su lugar, también se puede utilizar para la obtención de datos (*data fetching*) con el fin de mostrar contenido temporal mientras los datos se están cargando (por ejemplo, desde una base de datos). Aunque, como también aprenderás, Suspense solo se puede utilizar para la obtención de datos si estos se obtienen de una manera determinada.

Además, este capítulo revisitará el Hook `use()`, que se introdujo en el Capítulo 11, *Trabajando con Estado Complejo*. Como aprenderás, además de usarlo para obtener acceso a valores de contexto, este Hook también se puede utilizar en combinación con Suspense.

---

### Sección 2: Mostrar contenido alternativo granular con Suspense

Al obtener datos o descargar un recurso (por ejemplo, un archivo de código), pueden producirse retrasos en la carga: retrasos que pueden dar lugar a una mala experiencia de usuario. Por lo tanto, deberías considerar mostrar algún contenido alternativo temporal mientras esperas el recurso solicitado.

Por esa razón, para simplificar el proceso de renderizar contenido alternativo mientras se espera algún recurso, React ofrece su componente Suspense. Como se mostró en el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, puedes usar el componente Suspense como un contenedor alrededor de elementos de React que obtienen código o datos. Por ejemplo, al usarlo en el contexto de la división de código, puedes mostrar contenido alternativo temporal de la siguiente manera:

```javascript
import { lazy, Suspense, useState } from 'react';

const DateCalculator = lazy(() =>
  import('./components/DateCalculator.jsx')
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
        But you can also open a calculator which calculates the difference
        between two dates.
      </p>
      <button onClick={handleOpenDateCalc}>Open Calculator</button>
      <Suspense fallback={<p>Loading...</p>}>
        {showDateCalc && <DateCalculator />}
      </Suspense>
    </>
  );
}
```

En este ejemplo (que proviene de un proyecto regular de React basado en Vite), el componente Suspense de React se envuelve alrededor del componente `DateCalculator` renderizado condicionalmente. `DateCalculator` se crea con la ayuda de la función `lazy()` de React, que se utiliza para cargar de forma diferida (es decir, bajo demanda) el paquete de código que pertenece a este componente.

Como resultado, todo el resto del contenido de la página se muestra desde el principio. Solo el componente `DateCalculator` mostrado condicionalmente se reemplaza con el contenido alternativo (`<p>Loading...</p>`) mientras se obtiene el código. Por lo tanto, Suspense se utiliza para renderizar código JSX alternativo en un nivel muy granular. En lugar de reemplazar todo el marcado de la página o del componente con contenido temporal, solo se reemplaza una pequeña parte de la interfaz de usuario.

Por supuesto, Suspense proporciona una funcionalidad que también sería excelente tener al obtener datos; después de todo, allí también ocurren retrasos con frecuencia.

#### Uso de Suspense para la obtención de datos con Next.js
Como se explicó en el capítulo anterior, en la sección *Gestión de estados de carga con Next.js*, el proceso de obtención de datos a menudo viene acompañado de tiempos de espera que pueden afectar negativamente la experiencia del usuario. Es por eso que, en esa misma sección, aprendiste que Next.js te permite definir un archivo `loading.js` que contiene algún componente alternativo que se renderiza durante dicho retraso.

Sin embargo, el uso de ese enfoque esencialmente reemplaza toda la página (o el área principal de esa página) con el contenido del componente alternativo de carga. Pero eso no siempre es lo ideal: en su lugar, es posible que desees mostrar contenido alternativo de carga en un nivel más granular al obtener datos.

Afortunadamente, en los proyectos de Next.js, puedes usar Suspense de manera similar a como se muestra en el ejemplo de la sección anterior, envolviéndolo alrededor de componentes que obtienen datos. Dado que Next.js admite la transmisión de respuestas HTTP mediante *streaming*, puede renderizar el resto de la página de inmediato mientras transmite el contenido que depende de los datos obtenidos al lado del cliente una vez que esté disponible. Hasta que los datos se carguen y estén disponibles, Suspense renderizará su contenido alternativo (*fallback*) definido.

Por lo tanto, volviendo al ejemplo de la sección *Gestión de estados de carga con Next.js* del Capítulo 16, *React Server Components y Server Actions*, puedes aprovechar Suspense externalizando el código de obtención de datos en un componente `UserGoals` separado:

```javascript
import fs from 'node:fs/promises';

async function fetchGoals() {
  await new Promise((resolve) => setTimeout(resolve, 3000)); // delay
  const goals = await fs.readFile('./data/user-goals.json', 'utf-8');
  return JSON.parse(goals);
}

export default async function UserGoals() {
  const fetchedGoals = await fetchGoals();
  return (
    <ul>
      {fetchedGoals.map((goal) => (
        <li key={goal}>{goal}</li>
      ))}
    </ul>
  );
}
```

Este componente `UserGoals` se puede envolver con Suspense en el componente `GoalsPage` de la siguiente manera:

```javascript
import { Suspense } from 'react';
import UserGoals from '../../components/UserGoals';

export default async function GoalsPage() {
  return (
    <>
      <h1>Top User Goals</h1>
      <Suspense
        fallback={<p id="fallback">Fetching user goals...</p>}
      >
        <UserGoals />
      </Suspense>
    </>
  );
}
```

Este código ahora utiliza el componente Suspense de React para mostrar un párrafo alternativo mientras el componente `UserGoals` obtiene los datos.

> [!NOTE]
> Puedes encontrar el código completo del proyecto de demostración en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/examples/02-data-fetching-suspense](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/examples/02-data-fetching-suspense).

Como resultado, cuando los usuarios navegan a `/goals`, ven inmediatamente el título (el elemento `<h1>`) en combinación con el contenido alternativo. Ya no hay necesidad de un archivo `loading.js` separado.

**Figura 17.1**: El contenido alternativo se muestra como parte de la página de destino, en lugar de reemplazarla por completo.

Sin embargo, la ventaja de usar Suspense en esta situación no es solo que el archivo `loading.js` ya no sea necesario. En su lugar, la obtención de datos y el contenido alternativo ahora se pueden gestionar a un nivel muy granular.

Por ejemplo, en una aplicación de tienda online más compleja, podrías tener un componente como este:

```javascript
function ShopOverviewPage() {
  return (
    <>
      <header>
        <h1>Find your next deal!</h1>
        <MainNavigation />
      </header>
      <main>
        <Suspense fallback={<DailyDealSkeleton />}>
          <DailyDeal />
        </Suspense>
        <section id="search">
          <h2>Looking for something specific?</h2>
          <Search />
        </section>
        <Suspense fallback={<p>Fetching products...</p>}>
          <Products />
        </Suspense>
      </main>
    </>
  );
}
```

En este ejemplo, los elementos `<header>` y `<section id="search">` siempre son visibles y se renderizan de inmediato. Por otro lado, `<DailyDeal />` y `<Products />` solo se renderizan una vez que sus datos han sido obtenidos. Hasta entonces, se muestran sus respectivos contenidos alternativos.

**Figura 17.2**: Los marcadores de posición se muestran inicialmente hasta que los datos cargados se transmiten en streaming y se renderizan en la pantalla.

`<DailyDeal />` y `<Products />` se cargarán y renderizarán independientemente uno del otro, ya que están envueltos por dos bloques Suspense diferentes. En consecuencia, los usuarios verán de inmediato el encabezado y el área de búsqueda, y finalmente verán la oferta del día y los productos, aunque cualquiera de los dos puede cargarse y renderizarse primero.

Lo importante de estos ejemplos es que los componentes envueltos por Suspense son RSCs que usan `async`/`await`. Como aprenderás en la siguiente sección, no todos los componentes de React interactuarán con el componente Suspense. Pero los React Server Components, en proyectos de Next.js, sí lo harán.

#### Uso de Suspense en otros proyectos de React: posible, pero complicado
La sección anterior exploró cómo puedes aprovechar Suspense para la obtención de datos con RSCs en proyectos de Next.js.

Sin embargo, Suspense no es una característica o concepto específico de Next.js; en su lugar, lo proporciona el propio React. En consecuencia, puedes usarlo en cualquier proyecto de React para mostrar contenido alternativo mientras se obtienen los datos.

Al menos, esa es la teoría. Pero resulta que no puedes usarlo con todos los componentes y estrategias de obtención de datos.

#### Suspense no funciona con `useEffect()`
Dado que obtener datos a través de `useEffect()` es una estrategia común, podrías sentir la tentación de usar Suspense en conjunto con este Hook para mostrar contenido alternativo mientras los datos se cargan a través de la función de efecto.

Por ejemplo, el siguiente componente `BlogPosts` utiliza `useEffect()` para cargar y mostrar algunas publicaciones de blog:

```javascript
import { useEffect, useState } from 'react';

function BlogPosts() {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    async function fetchBlogPosts() {
      // simulate slow network
      await new Promise((resolve) => setTimeout(resolve, 3000));
      const response = await fetch(
        'https://jsonplaceholder.typicode.com/posts'
      );
      const posts = await response.json();
      setPosts(posts);
    }
    fetchBlogPosts();
  }, []);

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Podrías envolver este componente con Suspense de la siguiente manera:

```javascript
import { Suspense } from 'react';
import BlogPosts from './components/BlogPosts.jsx';

function App() {
  return (
    <>
      <h1>All posts</h1>
      <Suspense fallback={<p>Fetching blog posts...</p>}>
        <BlogPosts />
      </Suspense>
    </>
  );
}
```

Desafortunadamente, esto no funcionará de la manera prevista. En lugar de mostrar el contenido alternativo, no se renderizará nada mientras se obtienen los datos.

La razón de este comportamiento es que Suspense está diseñado para suspenderse cuando se obtienen datos **durante el proceso de renderizado del componente**, no cuando se obtienen dentro de alguna función de efecto.

Resulta útil recordar cómo funciona `useEffect()` (del Capítulo 8, *Manejo de Efectos Secundarios*): la función de efecto se ejecuta después de que se ejecuta la función del componente, es decir, después de que finaliza el primer ciclo de renderizado del componente.

Como resultado, no puedes usar Suspense para mostrar contenido alternativo cuando obtienes datos mediante `useEffect()`. En su lugar, en esos casos, necesitas gestionar y usar manualmente algún estado de carga en el componente que realiza la obtención de datos (es decir, gestionando manualmente diferentes partes del estado como `isLoading`, por ejemplo, como se explicó y mostró en el Capítulo 11, *Trabajando con Estado Complejo*, en las secciones *Limitaciones de useState()* y *Gestión del estado con useReducer()*).

#### Obtener datos durante el renderizado: la forma incorrecta
Dado que Suspense pretende mostrar contenido alternativo mientras un componente obtiene datos durante su proceso de renderizado, podrías intentar reescribir el componente `BlogPosts` para que se vea así:

```javascript
async function BlogPosts() {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  const response = await fetch(
    'https://jsonplaceholder.typicode.com/posts'
  );
  const posts = await response.json();
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Pero intentar usar este código generará un error en las herramientas de desarrollo del navegador:

**Figura 17.3**: React se queja de los componentes asíncronos en el lado del cliente.

React no admite el uso de `async`/`await` en componentes de cliente. Solo los React Server Components pueden usar esa sintaxis (y por lo tanto devolver promesas). En consecuencia, los proyectos regulares de React, que no están configurados para admitir RSCs, no pueden usar esta solución.

Por supuesto, se te podría ocurrir una solución alternativa (problemática) como esta:

```javascript
function BlogPosts() {
  const [posts, setPosts] = useState([]);
  new Promise(() =>
    setTimeout(() => {
      return fetch('https://jsonplaceholder.typicode.com/posts')
        .then((response) => response.json())
        .then((fetchedPosts) => setPosts(fetchedPosts));
    }, 3000)
  );

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Pero este enfoque ya fue descartado en el Capítulo 8, *Manejo de Efectos Secundarios*, en la sección *¿Cuál es el problema?*: el código crea un bucle infinito.

Por lo tanto, obtener datos como parte del proceso de renderizado de un componente es realmente difícil cuando no se trabaja con RSCs.

#### Conseguir soporte para Suspense es complicado
Dado que Suspense requiere que la obtención de datos ocurra durante el proceso de renderizado, lo cual es difícil de configurar manualmente, la propia documentación de React ([https://react.dev/reference/react/Suspense#displaying-a-fallback-while-content-is-loading](https://react.dev/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)) menciona que *"solo las fuentes de datos habilitadas para Suspense activarán el componente Suspense"*, indicando además que esas fuentes de datos incluyen:
- Obtención de datos con frameworks habilitados para Suspense como Relay y Next.js.
- Carga diferida de código de componentes con `lazy()`.
- Lectura del valor de una Promesa con `use()`.

En la misma página, la documentación oficial destaca que *"la obtención de datos habilitada para Suspense sin el uso de un framework de opinión aún no es compatible"*.

> [!NOTE]
> La documentación puede cambiar con el tiempo, y React también. Pero aunque la redacción exacta pueda diferir en el momento en que leas esto, la forma de usar Suspense y el hecho de que no se pueda usar sin bibliotecas especiales o características como `lazy()`, es muy poco probable que cambien.
> Este capítulo se escribió cuando se lanzó React 19. Puedes visitar el registro de cambios (*changelog*) oficial de este libro para saber si algo significativo ha cambiado desde entonces: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/main/CHANGELOG.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/main/CHANGELOG.md).

Por lo tanto, a menos que planees crear tu propia biblioteca compatible con Suspense, debes limitarte a usar Suspense para la división de código (a través de `lazy()`), usar un framework o biblioteca de terceros que se integre con Suspense, o explorar el uso del Hook `use()`.

Por supuesto, la función `lazy()` (y cómo usarla con Suspense) ya se cubrió en el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, en la sección *Reducción del tamaño de los paquetes mediante división de código (Lazy Loading)*. Pero, ¿qué pasa con las otras dos opciones: las bibliotecas compatibles con Suspense y el Hook `use()`?

#### Uso de Suspense para la obtención de datos con bibliotecas compatibles
Como aprendiste en la sección *Uso de Suspense para la obtención de datos con Next.js*, puedes usar Suspense para la obtención de datos cuando trabajas con Next.js. Pero si bien Next.js es uno de los frameworks de React más populares que admite Suspense, no es la única opción que tienes.

Por ejemplo, **TanStack Query** (anteriormente conocido como React Query) es otra popular biblioteca de terceros que desbloquea Suspense para la obtención de datos. Sin embargo, esta biblioteca, a diferencia de Next.js, no pretende ayudar con la creación de aplicaciones React *fullstack* ni con la ejecución de código en el lado del servidor. En su lugar, TanStack Query es una biblioteca enfocada por completo en ayudar con la obtención de datos en el lado del cliente, las mutaciones de datos y la gestión del estado asíncrono. Dado que se ejecuta en el lado del cliente, también funciona en proyectos de React que no se integran con SSR y RSCs, aunque también puedes usarla en dichos proyectos.

TanStack Query es una biblioteca compleja y rica en características; probablemente podríamos escribir un libro entero sobre ella. Pero el siguiente breve fragmento de código (que proviene de un proyecto basado en Vite, no de un proyecto de Next.js) muestra cómo puedes obtener datos con la ayuda de esa biblioteca:

```javascript
import { useSuspenseQuery } from '@tanstack/react-query';

async function fetchPosts() {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  const response = await fetch('https://jsonplaceholder.typicode.com/posts');
  const posts = await response.json();
  return posts;
}

function BlogPosts() {
  const { data } = useSuspenseQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

En este ejemplo, el componente `BlogPosts` utiliza el Hook `useSuspenseQuery()` de TanStack Query, junto con una función personalizada `fetchPosts()`, para obtener datos mediante una solicitud HTTP. Como implica el nombre del Hook, este se integra con el componente Suspense de React.

Como resultado, el componente `BlogPosts` puede envolverse con Suspense de la siguiente manera:

```javascript
import { Suspense } from 'react';
import BlogPosts from './components/BlogPosts.jsx';

function App() {
  return (
    <>
      <h1>All posts</h1>
      <Suspense fallback={<p>Fetching blog posts...</p>}>
        <BlogPosts />
      </Suspense>
    </>
  );
}
```

Como puedes ver, Suspense se utiliza de la misma manera que se utilizó con `lazy()` o Next.js. Por lo tanto, su funcionalidad y uso no cambian: si lo envuelves alrededor de un componente que se integra con Suspense (como lo hace `BlogPosts` a través del Hook `useSuspenseQuery()` de TanStack Query), Suspense se puede utilizar para mostrar contenido alternativo mientras se lleva a cabo un proceso de obtención de datos.

> [!NOTE]
> Puedes encontrar el proyecto de ejemplo completo en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/examples/05-tanstack-query](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/examples/05-tanstack-query).

Por supuesto, este es solo un ejemplo simple. Puedes hacer mucho más con TanStack Query, y también hay otras bibliotecas que se pueden utilizar en conjunto con Suspense. Solo es importante comprender que existen otras opciones además de Next.js. Pero también es crucial tener en cuenta que no todo el código (y tampoco todas las bibliotecas) funcionará con Suspense.

Además de usar bibliotecas que se integran directamente con Suspense (como TanStack Query a través de su Hook `useSuspenseQuery()`), también puedes usar Suspense para la obtención de datos con la ayuda del Hook incorporado **`use()`** de React.

#### Uso de `use()` con datos durante el renderizado
El Hook `use()` ofrecido por React no se limita a acceder a valores de contexto, como se mostró en el Capítulo 11, *Trabajando con Estado Complejo*; en su lugar, también se puede utilizar para **leer valores de una promesa**.

Por lo tanto, puedes usar el Hook `use()` durante el proceso de renderizado de un componente para extraer y usar el valor de una promesa. `use()` interactuará automáticamente con cualquier componente Suspense contenedor y le informará sobre el estado actual del proceso de obtención de datos (es decir, si la promesa se ha resuelto o no).

El ejemplo de la sección *Obtener datos durante el renderizado: la forma incorrecta* se puede ajustar para usar el Hook `use()` de la siguiente manera:

```javascript
import { use } from 'react';

async function fetchPosts() {
  await new Promise((resolve) => setTimeout(resolve, 3000));
  const response = await fetch(
    'https://jsonplaceholder.typicode.com/posts'
  );
  const posts = await response.json();
  return posts;
}

function BlogPosts() {
  const posts = use(fetchPosts());
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

El componente `BlogPosts` ya no es un componente que usa `async`/`await`. En su lugar, utiliza el Hook importado `use()` para leer el valor de la promesa producida al llamar a `fetchPosts()`.

Como se mencionó, `use()` interactúa con Suspense, por lo que `BlogPosts` se puede envolver con Suspense de esta manera:

```javascript
import { Suspense } from 'react';
import BlogPosts from './components/BlogPosts.jsx';

function App() {
  return (
    <>
      <h1>All posts</h1>
      <Suspense fallback={<p>Fetching blog posts...</p>}>
        <BlogPosts />
      </Suspense>
    </>
  );
}
```

Al ejecutar este código, podría funcionar según lo previsto (dependiendo de la versión de React que estés utilizando), pero es más probable que no produzca ningún resultado o incluso muestre un mensaje de error en las herramientas de desarrollo del navegador:

**Figura 17.4**: El Hook use() solo funciona con promesas creadas por bibliotecas compatibles con Suspense.

Como explica este mensaje de error, el Hook `use()` no está diseñado para usarse con promesas regulares creadas como en el ejemplo anterior. En su lugar, debe usarse en promesas proporcionadas por bibliotecas o frameworks compatibles con Suspense.

> [!NOTE]
> Si deseas ir en contra de la recomendación oficial e intentar construir promesas que admitan `use()` y Suspense, puedes explorar los proyectos de demostración oficiales de Suspense vinculados en la documentación oficial de React ([https://19.react.dev/reference/react/Suspense](https://19.react.dev/reference/react/Suspense)), por ejemplo, este proyecto: [https://codesandbox.io/p/sandbox/strange-black-6j7nnj](https://codesandbox.io/p/sandbox/strange-black-6j7nnj).
> Ten en cuenta que, como se menciona en la documentación, el enfoque utilizado en ese proyecto de demostración utiliza APIs inestables y podría no funcionar con futuras versiones de React.

Por lo tanto, nuevamente, se necesita el soporte de un framework o biblioteca de terceros. No importa si intentas usar Suspense con componentes que obtienen datos como parte del proceso de renderizado con o sin `use()`, terminas necesitando ayuda.

Dicho de otra manera: para aprovechar Suspense, debes obtener datos directamente a través de una biblioteca o framework compatible con Suspense, o debes usar el Hook `use()` en una promesa generada por una biblioteca o framework compatible con Suspense.

Uno de estos frameworks es, una vez más, Next.js. Además de usar Suspense alrededor de los RSCs, como se muestra en la sección *Uso de Suspense para la obtención de datos con Next.js*, también puedes usar Suspense junto con el Hook `use()` en promesas producidas por Next.js.

#### Uso de `use()` con promesas creadas por Next.js
Los proyectos de Next.js son capaces de crear promesas que funcionarán con `use()` y Suspense. Para ser precisos, **cualquier promesa que crees en un RSC y pases a un componente (de cliente) a través de props califica como una promesa que se puede usar con `use()`**.

Considera este código de ejemplo:

```javascript
import fs from 'node:fs/promises';
import UserGoals from '../../components/UserGoals';

async function fetchGoals() {
  await new Promise((resolve) => setTimeout(resolve, 3000)); // delay
  const goals = await fs.readFile('./data/user-goals.json', 'utf-8');
  return JSON.parse(goals);
}

export default function GoalsPage() {
  const fetchGoalsPromise = fetchGoals();
  return (
    <>
      <h1>Top User Goals</h1>
      <UserGoals promise={fetchGoalsPromise} />
    </>
  );
}
```

En este fragmento de código, se crea una promesa llamando a `fetchGoals()` y se almacena en una constante llamada `fetchGoalsPromise`. La promesa creada (`fetchGoalsPromise`) luego se pasa como valor para la prop `promise` al componente `UserGoals`.

Junto con otro componente, este componente `UserGoals` se define en el archivo `UserGoals.js` de la siguiente manera:

```javascript
import { use, Suspense } from 'react';

function Goals({ fetchGoalsPromise }) {
  const goals = use(fetchGoalsPromise);
  return (
    <ul>
      {goals.map((goal) => (
        <li key={goal}>{goal}</li>
      ))}
    </ul>
  );
}

export default function UserGoals({ promise }) {
  return (
    <Suspense fallback={<p id="fallback">Fetching user goals...</p>}>
      <Goals fetchGoalsPromise={promise} />
    </Suspense>
  );
}
```

En este ejemplo de código, el componente `UserGoals` usa Suspense para envolver el componente `Goals`, al cual esencialmente le reenvía el valor recibido de la prop `promise` (a través de la prop `fetchGoalsPromise`). Luego, el componente `Goals` lee ese valor de promesa a través del Hook `use()`.

Dado que la promesa se crea en un RSC (`GoalsPage`) administrado por Next.js, React no se quejará de este código: Next.js crea promesas que funcionan con `use()`. En su lugar, mostrará el contenido alternativo (`<p id="fallback">Fetching user goals...</p>`) mientras se obtienen los datos y renderiza la interfaz de usuario final una vez que los datos han llegado y se han transmitido en streaming al cliente.

Como se explicó anteriormente, cualquier elemento no envuelto por Suspense (es decir, el elemento `<h1>`, en este ejemplo) se mostrará desde el principio.

**Figura 17.5**: El texto alternativo se muestra junto al título mientras se obtienen los datos a través de use().

También vale la pena señalar que tanto `UserGoals` como `Goals` también son RSCs; no obstante, pueden usar el Hook `use()`.

Normalmente, los Hooks no se pueden usar en RSCs, pero el Hook `use()` es especial. Así como se puede usar dentro de declaraciones `if` o bucles (como se explicó en el Capítulo 11, *Trabajando con Estado Complejo*), se puede ejecutar tanto en componentes de servidor como de cliente.

Sin embargo, al trabajar con un componente de servidor, también puedes simplemente usar `async`/`await` en lugar de `use()`. Por lo tanto, el Hook `use()` es realmente útil cuando se trata de leer valores de promesas en componentes de cliente, donde `async`/`await` no está disponible.

#### Uso de `use()` en Client Components
Además de usarlo para acceder al contexto, el Hook `use()` se introdujo para ayudar a **leer valores de promesas en componentes de cliente**, es decir, en situaciones en las que no puedes usar `async`/`await`.

Considera este ejemplo actualizado de objetivos de usuario, donde se gestiona algún estado y se desencadena un efecto secundario:

```javascript
'use client';
import { use, Suspense, useEffect, useState } from 'react';
// sendAnalytics() is a dummy function that just logs to the console
import { sendAnalytics } from '../lib/analytics';

function Goals({ fetchGoalsPromise }) {
  const [mainGoal, setMainGoal] = useState();
  const goals = use(fetchGoalsPromise);

  function handleSetMainGoal(goal) {
    setMainGoal(goal);
  }

  return (
    <ul>
      {goals.map((goal) => (
        <li
          key={goal}
          id={goal === mainGoal ? 'main-goal' : undefined}
          onClick={() => handleSetMainGoal(goal)}
        >
          {goal}
        </li>
      ))}
    </ul>
  );
}

export default function UserGoals({ promise }) {
  useEffect(() => {
    sendAnalytics('user-goals-loaded', navigator.userAgent);
  }, []);

  return (
    <Suspense fallback={<p id="fallback">Fetching user goals...</p>}>
      <Goals fetchGoalsPromise={promise} />
    </Suspense>
  );
}
```

En este ejemplo, el componente `Goals` usa `useState()` para gestionar la información de qué objetivo fue marcado como el objetivo principal por el usuario. Además, el componente `UserGoals` (que usa Suspense) utiliza el Hook `useEffect()` para enviar un evento de analítica una vez que el componente se renderiza (es decir, antes de que se muestre el componente suspendido `Goals`). Debido al uso de todas estas características exclusivas del lado del cliente, se requiere la directiva `'use client'`.

Como resultado, `async`/`await` no se puede usar en los componentes `Goals` y `UserGoals`. Pero dado que el Hook `use()` se puede usar en componentes de cliente, ofrece una posible solución para situaciones como esta. Y, dado que este ejemplo es de una aplicación Next.js, React no se quejará del tipo de promesa que consume `use()`. En su lugar, este código de ejemplo haría que se mostrara el contenido alternativo mientras se obtienen los datos de los objetivos.

---

### Sección 3: Patrones de uso de Suspense

Como has aprendido, el componente Suspense se puede envolver alrededor de componentes que obtienen datos como parte de su proceso de renderizado, siempre que lo hagan de manera compatible.

Por supuesto, en muchos proyectos, es posible que tengas múltiples componentes que obtienen datos y que deberían mostrar contenido alternativo mientras lo hacen. Afortunadamente, puedes usar el componente Suspense tantas veces como sea necesario; incluso puedes combinar múltiples componentes Suspense entre sí.

#### Revelar contenido juntos
Hasta ahora, en todos los ejemplos, Suspense siempre se envolvía alrededor de un solo componente. Pero no hay ninguna regla que te impida envolver Suspense alrededor de múltiples componentes.

Por ejemplo, el siguiente código es válido:

```javascript
function Shop() {
  return (
    <>
      <h1>Welcome to our shop!</h1>
      <Suspense fallback={<p>Fetching shop data...</p>}>
        <DailyDeal />
        <Products />
      </Suspense>
    </>
  );
}
```

En este fragmento de código, la obtención de datos en los componentes `DailyDeal` y `Products` comienza simultáneamente. Dado que ambos componentes están envueltos por un único componente Suspense, el contenido alternativo se muestra hasta que ambos componentes hayan terminado de obtener datos. Por lo tanto, si un componente (por ejemplo, `DailyDeal`) termina después de un segundo y el otro componente (`Products`) tarda cinco segundos, ambos componentes solo se revelarán (y reemplazarán el contenido alternativo) después de cinco segundos.

**Figura 17.6**: Los datos se obtienen en paralelo y el contenido alternativo se muestra mediante Suspense hasta que todos los componentes hayan finalizado.

#### Revelar contenido lo antes posible
Por supuesto, hay situaciones en las que podrías querer mostrar contenido alternativo para múltiples componentes, pero donde no deseas esperar a que todos los componentes terminen de obtener datos antes de mostrar cualquier contenido obtenido.

En tales situaciones, puedes usar Suspense varias veces:

```javascript
function Shop() {
  return (
    <>
      <h1>Welcome to our shop!</h1>
      <Suspense fallback={<p>Fetching daily deal data...</p>}>
        <DailyDeal />
      </Suspense>
      <Suspense fallback={<p>Fetching products data...</p>}>
        <Products />
      </Suspense>
    </>
  );
}
```

En este ejemplo de código ajustado, `DailyDeal` y `Products` están envueltos con dos instancias diferentes del componente Suspense. Por lo tanto, el contenido de cada componente se revelará una vez que esté disponible, independientemente del estado de obtención de datos del otro componente.

**Figura 17.7**: Cada componente reemplaza su contenido alternativo con el contenido final una vez que finaliza su obtención.

#### Anidar contenido suspendido
Además de obtener datos en paralelo, también puedes crear secuencias de carga más complejas con componentes Suspense anidados.

Considera este ejemplo:

```javascript
function Shop() {
  return (
    <>
      <h1>Welcome to our shop!</h1>
      <Suspense fallback={<p>Fetching shop data...</p>}>
        <DailyDeal />
        <Suspense fallback={<p>Fetching products data...</p>}>
          <Products />
        </Suspense>
      </Suspense>
    </>
  );
}
```

En este caso, inicialmente se muestra el párrafo con el texto *Fetching shop data*. Detrás de escena, comienza la obtención de datos en los componentes `DailyDeal` y `Products`.

Una vez que el componente `DailyDeal` termina de obtener datos, se muestra su contenido. Al mismo tiempo, debajo de `DailyDeal`, se renderiza el contenido alternativo del bloque Suspense anidado si el componente `Products` todavía está obteniendo datos.

Finalmente, una vez que `Products` ha recibido sus datos, el contenido alternativo del componente Suspense interno se elimina y el componente `Products` se renderiza en su lugar.

**Figura 17.8**: Los bloques Suspense anidados conducen a la obtención secuencial de datos y a la revelación de contenido.

Por lo tanto, como puedes ver, puedes usar Suspense varias veces. Además, puedes combinar diferentes componentes Suspense de modo que puedas crear exactamente la secuencia de carga y la experiencia de usuario que necesitas.

---

### Sección 4: ¿Deberías obtener datos mediante Suspense o `useEffect()`?

Como aprendiste a lo largo de este capítulo, puedes usar Suspense junto con RSCs, bibliotecas compatibles con Suspense o el Hook `use()` (que también requiere bibliotecas de soporte) para obtener datos y mostrar contenido alternativo mientras se obtienen los datos.

Alternativamente, como se cubrió en el Capítulo 11, *Trabajando con Estado Complejo*, también puedes obtener datos y mostrar manualmente contenido alternativo mediante `useEffect()` y `useState()` o `useReducer()`. En ese caso, esencialmente gestionas el estado que determina si mostrar algún contenido alternativo de carga por tu cuenta; con Suspense, React lo hace por ti.

En consecuencia, depende de ti qué enfoque prefieras. El uso de Suspense puede ahorrarte bastante código, ya que no necesitas gestionar estas diferentes partes del estado manualmente. Combinado con frameworks como Next.js o bibliotecas como TanStack Query, la obtención de datos puede volverse significativamente más fácil que cuando se hace manualmente a través de `useEffect()`. Además, Suspense se integra con RSCs y SSR y, por lo tanto, se puede utilizar para obtener datos en el servidor, a diferencia de `useEffect()`, que no tiene ningún efecto en el lado del servidor.

Sin embargo, si no estás utilizando ninguna biblioteca o framework que admita Suspense o promesas habilitadas para `use()`, no tienes más remedio que recurrir a `useEffect()` (y, por lo tanto, no usar Suspense para la obtención de datos). Esto puede cambiar con futuras versiones de React, ya que podrían proporcionar herramientas que ayuden a crear promesas que funcionen con `use()`. Pero por el momento, es básicamente una decisión entre usar (las bibliotecas adecuadas) y Suspense o no usar bibliotecas y recurrir a `useEffect()`.

---

### Sección 5: Resumen y puntos clave

- El componente **Suspense** se puede utilizar para mostrar contenido alternativo (*fallback*) mientras se obtienen datos o se descarga código.
- Para la obtención de datos, Suspense solo funciona con componentes que obtienen datos a través de **fuentes de datos compatibles con Suspense** durante su proceso de renderizado.
- Bibliotecas y frameworks como **TanStack Query** y **Next.js** admiten el uso de Suspense para la obtención de datos.
- Usando Next.js, puedes envolver Suspense alrededor de componentes de servidor (RSCs) que usan `async`/`await`.
- Alternativamente, Suspense se puede envolver alrededor de componentes que usan el Hook **`use()`** de React para leer el valor de una promesa.
- `use()` solo debe usarse para leer valores de promesas que se resuelven teniendo en cuenta Suspense, por ejemplo, promesas creadas por bibliotecas de terceros compatibles con Suspense.
- Al usar Next.js, las promesas creadas en RSCs y pasadas a componentes (de cliente) a través de props se pueden consumir a través de `use()`.
- El Hook `use()` ayuda a leer valores y usar Suspense en componentes que también necesitan usar características específicas del cliente como `useState()`.
- Suspense se puede envolver alrededor de tantos componentes como sea necesario para obtener datos y mostrar contenido simultáneamente.
- Suspense también se puede anidar para crear secuencias de carga complejas.

---

### Sección 6: ¿Qué sigue?

La función Suspense de React puede ser muy útil, ya que ayuda a mostrar contenido alternativo de forma granular mientras se obtienen códigos o datos. Al mismo tiempo, cuando se trata de la obtención de datos, puede ser complicado usar Suspense, ya que solo funciona con componentes que obtienen datos de la manera correcta (por ejemplo, a través del Hook `use()`, si la promesa pasada al Hook es compatible con Suspense).

Es por eso que este capítulo también exploró cómo usar Suspense y `use()` con Next.js, y cómo ese framework simplifica el proceso de obtención de datos y visualización de contenido alternativo con Suspense y `use()`.

A pesar de la complejidad potencial, Suspense puede ayudar a crear excelentes experiencias de usuario, ya que te permite mostrar fácilmente contenido alternativo mientras un recurso está pendiente.

Este capítulo también concluye la lista de características centrales de React que debes conocer como desarrollador de React. Por supuesto, siempre puedes profundizar más para explorar más patrones y bibliotecas de terceros. El siguiente (y último) capítulo compartirá algunos recursos y posibles próximos pasos en los que podrías profundizar después de terminar este libro.

---

### Sección 7: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se encuentran en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/17-suspense-use/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/17-suspense-use/exercises/questions-answers.md):

1. ¿Cuál es el propósito del componente `Suspense` de React?
2. ¿Cómo necesitan los componentes obtener datos para funcionar con `Suspense`?
3. ¿Cómo se puede utilizar `Suspense` al trabajar con Next.js?
4. ¿Cuál es el propósito del Hook `use()`?
5. ¿Qué tipo de promesas pueden ser leídas por el Hook `use()`?
6. Enumera tres formas de utilizar `Suspense` con múltiples componentes.

---

### Sección 8: Aplica lo aprendido

Con todo el conocimiento recién adquirido sobre Next.js, es hora de aplicarlo a un proyecto de demostración real.

En la siguiente sección, encontrarás una actividad que te permitirá practicar el trabajo con Next.js y Suspense. Como siempre, también necesitarás emplear algunos de los conceptos cubiertos en capítulos anteriores.

#### Actividad 17.1: Implementar Suspense en el Mini Blog
En esta actividad, tu trabajo consiste en basarte en el proyecto terminado de la Actividad 16.1. Allí se construyó un blog muy simple. Ahora, tu tarea es mejorar este blog para mostrar algún contenido alternativo mientras se carga la lista de publicaciones de blog o los detalles de una publicación de blog individual. Para demostrar tus conocimientos, debes obtener datos mediante `async`/`await` en la página de inicio (`/`), y a través del Hook `use()` en la página `/blog/<algun-id>`.

Además, la lista de publicaciones de blog disponibles también debe mostrarse debajo de los detalles de una sola publicación de blog. Por supuesto, mientras se obtienen los datos de esa lista, se debe mostrar algún texto alternativo; sin embargo, ese texto debe mostrarse independientemente del contenido alternativo para los detalles de la publicación del blog.

> [!NOTE]
> Puedes encontrar una instantánea del proyecto inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/activities/practice-1-start). Al descargar este código, siempre descargarás todo el repositorio. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start`, en este caso) para usar la instantánea de código correcta.

En el proyecto inicial proporcionado, encontrarás funciones para obtener todas las publicaciones de blog y una sola publicación. Estas funciones contienen retrasos artificiales para simular servidores lentos.

Después de descargar el código y ejecutar `npm install` en la carpeta del proyecto para instalar todas las dependencias requeridas, los pasos de la solución son los siguientes:
1. Externaliza la lógica para obtener y mostrar una lista de publicaciones en un componente separado.
2. Utiliza ese componente en la página de inicio y usa el componente `Suspense` de React para mostrar un contenido alternativo adecuado mientras se obtienen las publicaciones del blog.
3. Además, externaliza la lógica para recuperar y renderizar los detalles de una publicación de blog individual en un componente de cliente (!) separado. Muestra ese componente recién creado en la página `/blog/<algun-id>`.
4. Pasa una promesa para obtener los detalles de una publicación a ese componente recién creado y utiliza el Hook `use()` para leer su valor. Además, aprovecha el componente `Suspense` para mostrar algún contenido alternativo.
5. Reutiliza el componente que obtiene y renderiza una lista de publicaciones de blog y muéstralo debajo de los detalles de la publicación de blog en la página `/blog/<algun-id>`. Usa `Suspense` para mostrar algún contenido alternativo, independientemente del estado de obtención de datos de los detalles de la publicación del blog.

La página final debería verse como se muestra en las siguientes capturas de pantalla:

**Figura 17.9**: Se muestra el contenido alternativo mientras se obtienen las publicaciones del blog.

**Figura 17.10**: Se muestra el contenido alternativo mientras se obtienen los detalles de la publicación del blog y la lista de publicaciones del blog.

> [!NOTE]
> Puedes encontrar el código completo para esta actividad y una solución de ejemplo aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/17-suspense-use/activities/practice-1).
