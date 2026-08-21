# Parte 2: Construcción de Aplicaciones React Complejas

## Capítulo 14: Gestión de Datos con React Router

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Utilizar React Router para obtener (*fetch*) o enviar datos sin necesidad de usar `useEffect()` o `useState()`.
- Compartir datos entre diferentes rutas sin utilizar la función Context de React.
- Actualizar la interfaz de usuario en función del estado actual de envío de datos.
- Crear rutas de página y rutas de acción (*action routes*).
- Mejorar la experiencia del usuario difiriendo la carga de datos no críticos (*deferring data loading*).

---

### Sección 1: Introducción

En el capítulo anterior, aprendiste a usar React Router para cargar diferentes componentes para diferentes rutas de URL. Esta es una característica importante, ya que te permite crear sitios web de múltiples páginas mientras sigues usando React.

El enrutamiento es una característica crucial para muchas aplicaciones web, y React Router es, por lo tanto, un paquete muy importante. Pero así como la mayoría de los sitios web necesitan enrutamiento, casi todos los sitios web necesitan **obtener y manipular datos**. Por ejemplo, las solicitudes HTTP en la mayoría de los sitios web se envían para cargar datos (como una lista de productos o publicaciones de blog) o para mutar datos (por ejemplo, para crear un producto o una publicación de blog).

En el Capítulo 8, *Manejo de Efectos Secundarios*, aprendiste que puedes usar el Hook `useEffect()` y varias otras características de React para enviar solicitudes HTTP desde dentro de una aplicación de React. Pero si estás utilizando React Router, obtienes herramientas nuevas y aún más potentes para trabajar con datos.

Este capítulo explorará qué nuevas características están disponibles en React Router y cómo se pueden usar para simplificar el proceso de obtención o envío de datos.

---

### Sección 2: La obtención de datos y el enrutamiento están estrechamente vinculados

Como se mencionó anteriormente, la mayoría de los sitios web necesitan obtener (o enviar) datos, y la mayoría de los sitios web necesitan más de una página. Pero es importante darse cuenta de que estos dos conceptos suelen estar estrechamente relacionados.

Cada vez que un usuario visita una nueva página (como `/posts`), es probable que sea necesario obtener algunos datos. En el caso de una página `/posts`, los datos requeridos probablemente sean una lista de publicaciones de blog que se recuperan de un servidor backend. El componente de React renderizado (como `Posts`) debe, por lo tanto, enviar una solicitud HTTP al servidor backend, esperar la respuesta, manejar la respuesta (así como posibles errores) y, en última instancia, mostrar los datos obtenidos.

Por supuesto, no todas las páginas necesitan obtener datos. Las páginas de aterrizaje (*landing pages*), las páginas "Acerca de nosotros" y las páginas de "Términos y condiciones" probablemente no necesiten obtener datos cuando un usuario las visita. En cambio, los datos de esas páginas suelen ser estáticos; incluso podrían incluirse en el código fuente, ya que no cambian con frecuencia.

Pero muchas páginas sí necesitan obtener datos de un backend cada vez que se cargan, por ejemplo, "Productos", "Noticias", "Eventos" u otras páginas que se actualizan con poca frecuencia, como el "Perfil de usuario".

Y la obtención de datos no lo es todo. La mayoría de los sitios web también contienen funciones que requieren el envío de datos, ya sea una publicación de blog que se puede crear o actualizar, datos de productos que se administran o un comentario de usuario que se puede agregar. Por lo tanto, enviar datos a un backend también es un caso de uso muy común.

Y más allá de las solicitudes, es posible que los componentes también necesiten interactuar con otras APIs del navegador, como `localStorage`. Por ejemplo, es posible que sea necesario obtener la configuración del usuario del almacenamiento local a medida que se carga una determinada página.

Naturalmente, todas estas interacciones ocurren en las páginas. Pero puede que no sea inmediatamente obvio cuán estrechamente vinculados están la obtención y el envío de datos con el enrutamiento.

La mayor parte del tiempo, **los datos se obtienen cuando una ruta se activa**, es decir, cuando un componente (el componente de página) se renderiza por primera vez. Claro, los usuarios también pueden hacer clic en un botón para actualizar los datos, pero mientras esto es opcional, la obtención de datos al cargar la página inicial casi siempre es obligatoria.

Y cuando se trata de enviar datos, también existe una estrecha conexión con el enrutamiento. A primera vista, no queda claro cómo se relaciona porque, si bien tiene sentido obtener datos al cargar la página, es menos probable que necesites enviar datos de inmediato (excepto tal vez datos de seguimiento o análisis).

Pero es muy probable que, **después de enviar datos, desees navegar a una página diferente**, lo que significa que en realidad es al revés: en lugar de iniciar la obtención de datos a medida que se carga una página, deseas cargar una página diferente después de enviar algunos datos. Por ejemplo, después de que un administrador introduce algunos datos de producto y envía el formulario, normalmente debería ser redirigido a una página diferente (por ejemplo, de `/products/new` a la página `/products`).

La conexión entre la obtención de datos, el envío y el enrutamiento se puede resumir, por lo tanto, en los siguientes puntos:
- La **obtención de datos** a menudo debe iniciarse cuando una ruta se activa (si esa página necesita datos).
- Después de **enviar datos**, el usuario a menudo debe ser redirigido a otra ruta.

Debido a que estos conceptos están estrechamente acoplados, React Router proporciona funciones adicionales que simplifican enormemente el proceso de trabajar con datos.

#### Envío de solicitudes HTTP sin React Router
Trabajar con datos no se limita a enviar solicitudes HTTP. Como se mencionó en la sección anterior, es posible que también necesites almacenar o recuperar datos a través de `localStorage` o realizar alguna otra operación a medida que se carga una página. Pero enviar solicitudes HTTP es un escenario especialmente común y, por lo tanto, será el caso de uso principal considerado durante la mayor parte de este capítulo. No obstante, es vital tener en cuenta que lo que aprendas en este capítulo no se limita al envío de solicitudes HTTP.

Como verás, React Router proporciona varias características que ayudan a enviar solicitudes HTTP (o usar otras APIs de obtención y manipulación de datos), pero también puedes enviar solicitudes HTTP (o interactuar con `localStorage` u otras APIs) sin estas características. De hecho, el Capítulo 8, *Manejo de Efectos Secundarios*, ya te enseñó cómo se pueden enviar solicitudes HTTP desde dentro de componentes de React con la ayuda de `useEffect()`.

Al utilizar las capacidades de obtención de datos de React Router, **puedes deshacerte de `useEffect()` y de la gestión manual del estado**.

> [!NOTE]
> Además de volver atrás en este libro, también puedes revisar cómo funciona la obtención de datos con `useEffect()` a través de este ejemplo de código en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/examples/01-data-fetching-classic](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/examples/01-data-fetching-classic).

---

### Sección 3: Carga de datos con React Router

Con React Router, la obtención de datos se puede simplificar en este fragmento de código más corto:

```javascript
import { useLoaderData } from 'react-router-dom'; 

function Posts() { 
  const loadedPosts = useLoaderData(); 
  return ( 
    <main> 
      <h1>Your Posts</h1> 
      <ul className="posts"> 
        {loadedPosts.map((post) => ( 
          <li key={post.id}>{post.title}</li> 
        ))} 
      </ul> 
    </main> 
  ); 
} 

export default Posts; 

export async function loader() { 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/posts' 
  ); 
  if (!response.ok) { 
    throw new Error('Could not fetch posts'); 
  } 
  return response; 
}
```

Lo creas o no, realmente es mucho menos código que en los ejemplos mostrados en el Capítulo 8. En aquel entonces, al usar `useEffect()`, se debían administrar fragmentos de estado separados para manejar los estados de carga y error, así como los datos recibidos. Aunque, para ser justos, el contenido que debería mostrarse en caso de un error falta aquí; está en un archivo separado (que se mostrará más adelante), pero solo agregaría tres líneas de código adicionales.

En el fragmento de código anterior, ves un par de características nuevas que aún no se han cubierto en el libro. La función `loader()` y el Hook `useLoaderData()` son agregados por React Router. Estas características, junto con muchas otras que se explorarán a lo largo de este capítulo, están disponibles a través del paquete React Router.

Con esa biblioteca instalada, puedes configurar una prop adicional **`loader`** en las definiciones de tus rutas. Esta prop acepta una función que será ejecutada por React Router cada vez que se active esta ruta (o una de sus rutas secundarias, si se define):

```javascript
{ path: '/posts', element: <Posts />, loader: () => {...} }
```

Esta función se puede utilizar para realizar cualquier obtención de datos u otras tareas necesarias para mostrar con éxito el componente de página. Por lo tanto, la lógica para obtener los datos necesarios se puede extraer del componente y mover a una función separada.

Dado que muchos sitios web tienen docenas o incluso cientos de rutas, agregar estas funciones de carga en línea (*inline*) en los objetos de definición de rutas conduce rápidamente a definiciones de rutas complejas y confusas. Por esta razón, normalmente agregarás (y exportarás) la función `loader()` en el mismo archivo que contiene el componente que necesita los datos.

Al configurar las definiciones de rutas, puedes importar el componente y su función de carga y usarlo así:

```javascript
import Posts, { loader as postsLoader } from './components/Posts.jsx'; 
// … other code … 
const router = createBrowserRouter([ 
  { path: '/posts', element: <Posts />, loader: postsLoader } 
]);
```

Asignar un alias (`postsLoader`, en este ejemplo) a la función de carga importada es opcional pero recomendable, ya que es muy probable que tengas múltiples funciones de carga de diferentes componentes, lo que de otro modo provocaría colisiones de nombres.

> [!NOTE]
> Técnicamente, no necesitas nombrar tus funciones `loader`. Podrías usar cualquier nombre y asignarlas como valores para la propiedad `loader` en la definición de la ruta.
> Pero usar `loader` como nombre de función no solo sigue la convención; también tiene la ventaja de que el soporte de carga perezosa integrado de React Router (cubierto en el capítulo anterior) carga la función de carga de forma perezosa cuando es necesario. No puede hacer eso si eliges cualquier otro nombre.

Con este loader definido, **React Router ejecutará la función `loader()` cada vez que se active una ruta**. Para ser precisos, la función `loader()` se llama antes de que se ejecute la función del componente (es decir, antes de que se renderice el componente).

**Figura 14.1**: El componente Posts se renderiza después de que se ejecuta el loader.

Esto también explica por qué el ejemplo del componente `Posts` al comienzo de esta sección no contenía ningún código que manejara ningún estado de carga. Simplemente **no hay estado de carga** ya que una función de componente solo se ejecuta después de que su loader haya finalizado (y los datos estén disponibles). React Router no finalizará la transición de página hasta que la función `loader()` haya terminado su trabajo (aunque, como aprenderás hacia el final de este capítulo, hay una manera de cambiar este comportamiento).

La función `loader()` puede realizar cualquier operación de tu elección (como enviar una solicitud HTTP o acceder al almacenamiento del navegador a través de la API `localStorage`). Dentro de esa función, debes devolver los datos que deben exponerse a la función del componente. También vale la pena señalar que la función `loader()` puede devolver cualquier tipo de datos. También puede devolver un objeto `Promise` que luego se resuelve en cualquier tipo de datos. En ese caso, React Router esperará automáticamente a que se cumpla la Promesa antes de ejecutar la función del componente de ruta relacionada. La función `loader()` puede, por lo tanto, realizar tanto tareas asíncronas como síncronas.

> [!NOTE]
> Es importante comprender que la función `loader()`, al igual que todo el resto del código que compone tu aplicación de React, se ejecuta en el lado del cliente (es decir, en el navegador del visitante del sitio web). Por lo tanto, puedes realizar cualquier acción que se pueda realizar en cualquier otro lugar (por ejemplo, dentro de `useEffect()`) en tu aplicación de React.
> No debes intentar ejecutar código que pertenezca al lado del servidor. Conectarse directamente a una base de datos, escribir en el sistema de archivos o realizar cualquier otra tarea del lado del servidor fallará o introducirá riesgos de seguridad, lo que significa que podrías exponer accidentalmente las credenciales de la base de datos en el lado del cliente.

#### Obtener acceso a los datos cargados
Por supuesto, el componente que pertenece a un loader (es decir, el componente que forma parte de la misma definición de ruta) necesita los datos devueltos por el loader. Es por eso que React Router ofrece un nuevo Hook para acceder a esos datos: el Hook **`useLoaderData()`**.

Cuando se llama dentro de una función de componente, este Hook produce los datos devueltos por el loader que pertenece a la ruta activa. Si los datos devueltos son una Promesa, React Router (como se mencionó anteriormente) esperará automáticamente a que esa Promesa se resuelva y proporcionará los datos resueltos cuando se llame a `useLoaderData()`.

La función `loader()` también puede devolver un objeto de respuesta HTTP (o una Promesa que se resuelve en un objeto `Response`). Este es el caso en el ejemplo anterior porque la función `fetch()` produce una Promesa que se resuelve en un objeto de tipo `Response`. En ese caso, **React Router extrae automáticamente el cuerpo de la respuesta** y proporciona acceso directo a los datos que se adjuntaron a la respuesta (a través de `useLoaderData()`).

> [!NOTE]
> Si se debe devolver una respuesta, el objeto devuelto debe cumplir con la interfaz estándar `Response`, como se define aquí: [https://developer.mozilla.org/en-US/docs/Web/API/Response](https://developer.mozilla.org/en-US/docs/Web/API/Response).
> Devolver respuestas puede resultar extraño al principio. Después de todo, el código `loader()` todavía se ejecuta dentro del navegador (no en un servidor). Por lo tanto, técnicamente no se envió ninguna solicitud y no se debería requerir ninguna respuesta (ya que todo el código se ejecuta en el mismo entorno: el navegador).
> Por esa razón, puedes, pero no tienes que devolver una respuesta; puedes devolver cualquier tipo de valor. React Router simplemente también admite respuestas como uno de los posibles tipos de valor devuelto.

`useLoaderData()` se puede llamar en cualquier componente renderizado por el componente de ruta actualmente activo. Puede ser el componente de ruta en sí (`Posts`, en el ejemplo anterior), pero también puede ser cualquier componente anidado.

Por ejemplo, `useLoaderData()` también se puede utilizar en un componente `PostsList` que se incluye en el componente `Posts` (que tiene un loader agregado a su definición de ruta):

```javascript
import { useLoaderData } from 'react-router-dom'; 

function PostsList() { 
  const loadedPosts = useLoaderData(); 
  return ( 
    <main> 
      <h1>Your Posts</h1> 
      <ul className="posts"> 
        {loadedPosts.map((post) => ( 
          <li key={post.id}>{post.title}</li> 
        ))} 
      </ul> 
    </main> 
  ); 
} 

export default PostsList;
```

Para este ejemplo, el archivo del componente `Posts` se ve así:

```javascript
import PostsList from '../components/PostsList.jsx'; 

function Posts() { 
  return ( 
    <main> 
      <h1>Your Posts</h1> 
      <PostsList /> 
    </main> 
  ); 
} 

export default Posts; 

export async function loader() { 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/posts' 
  ); 
  if (!response.ok) { 
    throw new Error('Could not fetch posts'); 
  } 
  return response; 
}
```

Esto significa que `useLoaderData()` se puede utilizar exactamente en el lugar donde necesitas los datos. La función `loader()` también se puede definir donde quieras, pero debe agregarse a la ruta donde se requieren los datos.

> [!NOTE]
> Dependiendo de la versión de React Router que se esté utilizando, es posible que recibas una advertencia relacionada con la falta de un elemento "No HydrateFallback". Puedes ignorar esta advertencia, ya que solo importa cuando se utiliza renderizado en el servidor (*Server-Side Rendering*).
> También puedes explorar este ejemplo de código en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/examples/02-data-fetching-react-router](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/examples/02-data-fetching-react-router).

#### Carga de datos para rutas dinámicas
Para la mayoría de los sitios web, es poco probable que las rutas estáticas y predefinidas por sí solas sean suficientes para satisfacer tus necesidades. Por ejemplo, si creaste un sitio de blog con rutas exclusivamente estáticas, estarías limitado a una simple lista de publicaciones de blog en `/posts`. Para agregar más detalles sobre una publicación de blog seleccionada en rutas como `/posts/1` o `/posts/2` (para publicaciones con diferentes valores de `id`), deberías incluir rutas dinámicas.

Por supuesto, React Router también admite la obtención de datos con la ayuda de la función `loader()` para rutas dinámicas:

```javascript
{ path: "/posts/:id", element: <PostDetails />, loader: postDetailsLoader }
```

El componente `PostDetails` y su función de carga se pueden implementar de la siguiente manera:

```javascript
import { useLoaderData } from 'react-router-dom'; 

function PostDetails() { 
  const post = useLoaderData(); 
  return ( 
    <div id="post-details"> 
      <h1>{post.title}</h1> 
      <p>{post.body}</p> 
    </div> 
  ); 
} 

export default PostDetails; 

export async function loader({ params, request }) { 
  console.log(request); 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/posts/' + params.id 
  ); 
  if (!response.ok) { 
    throw new Error('Could not fetch post for id ' + params.id); 
  } 
  return response; 
}
```

Si se parece mucho al componente `Posts` de la sección *Carga de datos con React Router*, no es una coincidencia. Dado que la función `loader()` funciona exactamente de la misma manera, solo se utiliza una característica adicional para obtener el valor del segmento de ruta dinámico: un objeto **`params`** que React Router pone a disposición.

> [!NOTE]
> También puedes explorar este ejemplo de código en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/examples/03-dynamic-routes](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/examples/03-dynamic-routes).

Al agregar una función `loader()` a una definición de ruta, React Router llama a esa función cada vez que la ruta se activa, justo antes de que se renderice el componente. Al ejecutar esa función, React Router pasa un objeto que contiene información adicional como argumento a `loader()`.

Este objeto pasado a `loader()` incluye dos propiedades principales:
1. Una propiedad **`request`** que contiene un objeto con más detalles sobre la solicitud que condujo a la activación de la ruta.
2. Una propiedad **`params`** que produce un objeto que contiene un mapa clave-valor de todos los parámetros de ruta dinámicos para la ruta activa.

El objeto `request` no importa para este ejemplo y se analizará en la siguiente sección. Pero el objeto `params` contiene una propiedad `id` que contiene el valor del ID de la publicación para la cual se carga la ruta. La propiedad se llama `id` porque, en la definición de la ruta, se eligió `/posts/:id` como ruta. Si se hubiera elegido un nombre de marcador de posición diferente, una propiedad con ese nombre habría estado disponible en `params` (por ejemplo, para `/posts/:postId`, esto sería `params.postId`). Este comportamiento es similar al objeto `params` producido por `useParams()`, como se explicó en el Capítulo 13, *Aplicaciones Multipágina con React Router*.

Con la ayuda del objeto `params` y el ID de la publicación, se puede incluir el ID de publicación adecuado en la URL de solicitud saliente (para la solicitud `fetch()`) y, por lo tanto, se pueden cargar los datos correctos de la publicación desde la API del backend. Una vez que llegan los datos, React Router renderizará el componente `PostDetails` y expondrá la publicación cargada a través del Hook `useLoaderData()`.

#### Loaders, solicitudes y código del lado del cliente
En la sección anterior, aprendiste sobre un objeto `request` que se proporciona a la función `loader()`. Obtener un objeto `request` de este tipo puede resultar confuso porque React Router es una biblioteca del lado del cliente: todo el código se ejecuta en el navegador, no en un servidor. Por lo tanto, ninguna solicitud debería llegar a la aplicación de React (ya que las solicitudes HTTP se envían del cliente al servidor, no entre funciones de JavaScript en el lado del cliente).

Y, de hecho, no se envía ninguna solicitud a través de HTTP. En su lugar, React Router crea un objeto de solicitud a través de la interfaz `Request` integrada del navegador para usarlo como un "vehículo de datos". Esta solicitud no se envía a través de HTTP, pero se utiliza como un valor para la propiedad `request` en el objeto de datos que se pasa a tu función `loader()`.

> [!NOTE]
> Para obtener más información sobre la interfaz `Request` integrada, visita [https://developer.mozilla.org/en-US/docs/Web/API/Request](https://developer.mozilla.org/en-US/docs/Web/API/Request).

Este objeto `request` será innecesario en muchas funciones de carga, pero hay escenarios ocasionales en los que puedes extraer información útil de ese objeto, información que podría ser necesaria en el loader para obtener los datos correctos.

Por ejemplo, puedes usar el objeto `request` y su propiedad `url` para obtener acceso a cualquier parámetro de búsqueda (*query parameters*) que pueda incluirse en la URL de la página actualmente activa:

```javascript
export async function loader({ request }) { 
  // e.g. for localhost:5173/posts?sort=desc 
  const sortDirection = new URL(request.url).searchParams.get('sort'); 
  // Fetch sorted posts, based on local 'sort' query param value 
  const response = await fetch( 
    'https://example.com/posts?sorting=' + sortDirection 
  ); 
  return response; 
}
```

En este fragmento de código, el valor de `request` se utiliza para obtener el valor de un parámetro de consulta que se utiliza en la URL de la aplicación de React. Ese valor luego se utiliza en una solicitud saliente.

Sin embargo, es vital que tengas en cuenta que el código dentro de tu función `loader()`, al igual que todo tu otro código de React, siempre se ejecuta en el lado del cliente. Si, en cambio, deseas ejecutar código en un servidor (y, por ejemplo, obtener datos en el lado del servidor), debes utilizar el renderizado en el servidor (*Server-Side Rendering* o SSR) o algún framework de React que implemente SSR, como Next.js. SSR y Next.js se cubrirán en el próximo capítulo (Capítulo 15, *Renderizado en el Servidor (SSR) y Desarrollo Fullstack con Next.js*) y en los capítulos posteriores.

---

### Sección 4: Revisión de Layouts

React Router admite el concepto de **rutas de diseño (*layout routes*)**. Se trata de rutas que contienen otras rutas y las renderizan como elementos secundarios anidados. Como recordarás, este concepto se introdujo en el Capítulo 13, *Aplicaciones Multipágina con React Router*.

Convenientemente, las rutas de diseño también se pueden utilizar para **compartir datos entre rutas anidadas**. Considera este sitio web de ejemplo:

**Figura 14.2**: Un sitio web con un encabezado, una barra lateral y contenido principal.

Este sitio web tiene un encabezado con una barra de navegación, una barra lateral que muestra una lista de publicaciones disponibles y un área principal que muestra la publicación de blog seleccionada actualmente.

Este ejemplo incluye dos rutas de diseño que están anidadas entre sí:
1. La **ruta de diseño raíz (`Root`)**, que incluye la barra de navegación superior que se comparte en todas las páginas.
2. Una **ruta de diseño de publicaciones (`PostsLayout`)**, que incluye la barra lateral y el contenido principal de sus rutas secundarias (por ejemplo, los detalles de una publicación seleccionada).

El código de definiciones de rutas se ve así:

```javascript
const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, // main layout, adds navigation bar 
    children: [ 
      { index: true, element: <Welcome /> }, 
      { 
        path: '/posts', 
        element: <PostsLayout />, // posts layout, adds posts sidebar 
        loader: postsLoader, 
        children: [ 
          { index: true, element: <Posts /> }, 
          { path: ':id', element: <PostDetails />, loader: postDetailsLoader }, 
        ], 
      }, 
    ], 
  }, 
]);
```

Con esta configuración, tanto el componente `<Posts />` como el `<PostDetails />` se renderizan junto a la barra lateral (ya que la barra lateral forma parte del elemento `<PostsLayout />`).

La parte interesante es que la ruta `/posts` (es decir, la ruta de diseño) carga los datos de las publicaciones, ya que tiene asignado el `postsLoader`, por lo que el archivo del componente `PostsLayout` se ve así:

```javascript
import { Outlet, useLoaderData } from 'react-router-dom'; 
import PostsList from '../components/PostsList.jsx'; 

function PostsLayout() { 
  const loadedPosts = useLoaderData(); 
  return ( 
    <div id="posts-layout"> 
      <nav> 
        <PostsList posts={loadedPosts} /> 
      </nav> 
      <main> 
        <Outlet /> 
      </main> 
    </div> 
  ); 
} 

export default PostsLayout; 

export async function loader() { 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/posts' 
  ); 
  if (!response.ok) { 
    throw new Error('Could not fetch posts'); 
  } 
  return response; 
}
```

Dado que las rutas de diseño también son rutas normales, puedes agregar funciones `loader()` y usar `useLoaderData()` tal como lo harías en cualquier otra ruta. Sin embargo, debido a que las rutas de diseño se activan para múltiples rutas secundarias, sus datos también se muestran para diferentes rutas. En el ejemplo anterior, la lista de publicaciones de blog siempre se muestra en el lado izquierdo de la pantalla, sin importar si un usuario visita `/posts` o `/posts/10`:

**Figura 14.3**: Se utilizan el mismo diseño y datos para diferentes rutas secundarias.

En esta captura de pantalla, el diseño y los datos utilizados no cambian a medida que se activan diferentes rutas secundarias. React Router también evita volver a obtener datos innecesariamente (para los datos de la lista de publicaciones de blog) a medida que cambias entre rutas secundarias. Es lo suficientemente inteligente como para darse cuenta de que el diseño circundante no ha cambiado.

#### Reutilización de datos entre rutas
Las rutas de diseño no solo te ayudan a compartir componentes y marcado: también te permiten cargar y compartir datos entre una ruta de diseño y sus rutas secundarias.

Por ejemplo, el componente `PostDetails` (es decir, el componente que se renderiza para la ruta `/posts/:id`) necesita los datos de una sola publicación, y esos datos se pueden recuperar a través de un loader adjunto a la ruta `/posts/:id`:

```javascript
export async function loader({ params }) { 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/posts/' + params.id 
  ); 
  if (!response.ok) { 
    throw new Error('Could not fetch post for id ' + params.id); 
  } 
  return response; 
}
```

Este ejemplo se analizó anteriormente en este capítulo en la sección *Carga de datos para rutas dinámicas*. Este enfoque es adecuado, pero en algunas situaciones se puede evitar esta solicitud HTTP adicional. Por ejemplo, la siguiente configuración de ruta se puede simplificar y se puede evitar el `postDetailsLoader` adicional en la ruta secundaria:

```javascript
const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, // main layout, adds navigation bar 
    children: [ 
      { index: true, element: <Welcome /> }, 
      { 
        path: '/posts', 
        element: <PostsLayout />, // posts layout, adds posts sidebar 
        loader: postsLoader, 
        children: [ 
          { index: true, element: <Posts /> }, 
          { path: ':id', element: <PostDetails />, loader: postDetailsLoader // can be removed }, 
        ], 
      }, 
    ], 
  }, 
]);
```

En este ejemplo, la ruta `PostsLayout` ya obtiene una lista de todas las publicaciones. Ese componente de diseño también está activo para la ruta `PostDetails`. En tal escenario, obtener una sola publicación es innecesario, ya que todos los datos ya se han obtenido para la lista de publicaciones. Por supuesto, se requeriría un loader `postDetailsLoader` específico para la ruta secundaria `PostDetails` si la solicitud de la lista de publicaciones (por `postsLoader` en la ruta `PostsLayout`) no produjera todos los datos requeridos por `PostDetails`.

Pero si todos los datos están disponibles, React Router te permite acceder a los datos del loader de un componente de ruta principal a través del Hook **`useRouteLoaderData()`**.

Este Hook se puede utilizar de la siguiente manera:

```javascript
const posts = useRouteLoaderData('posts');
```

`useRouteLoaderData()` requiere un identificador de ruta como argumento. Requiere un identificador asignado a la ruta antecesora que contiene los datos que deben reutilizarse. Puedes asignar dicho identificador a través de la propiedad **`id`** a tus rutas como parte del código de definiciones de rutas:

```javascript
const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, // main layout, adds navigation bar 
    children: [ 
      { index: true, element: <Welcome /> }, 
      { 
        path: '/posts', 
        id: 'posts', // the id value is up to you 
        element: <PostsLayout />, // posts layout, adds posts sidebar 
        loader: postsLoader, 
        children: [ 
          { index: true, element: <Posts /> }, 
          { path: ':id', element: <PostDetails />, // details loader was removed }, 
        ], 
      }, 
    ], 
  }, 
]);
```

El Hook `useRouteLoaderData()` luego devuelve los mismos datos que produce `useLoaderData()` en esa ruta a la que agregaste el `id`. En este ejemplo, proporcionaría una lista de publicaciones de blog.

En `PostDetails`, este Hook se puede utilizar de la siguiente manera:

```javascript
import { useParams, useRouteLoaderData } from 'react-router-dom'; 

function PostDetails() { 
  const params = useParams(); 
  const posts = useRouteLoaderData('posts'); 
  const post = posts.find((post) => post.id.toString() === params.id); 
  return ( 
    <div id="post-details"> 
      <h1>{post.title}</h1> 
      <p>{post.body}</p> 
    </div> 
  ); 
} 

export default PostDetails;
```

El Hook `useParams()` se utiliza para obtener acceso al valor del parámetro de ruta dinámico, y el método `find()` se utiliza en la lista de publicaciones para identificar una sola publicación con una propiedad `id` coincidente. En este ejemplo, evitarías enviar una solicitud HTTP innecesaria al reutilizar datos que ya están disponibles.

Por lo tanto, el `postDetailsLoader` que formaba parte de la definición de la ruta `/posts/:id` se puede eliminar.

---

### Sección 5: Manejo de errores

En el primer ejemplo al principio de este capítulo (donde la solicitud HTTP se envió con la ayuda de `useEffect()`), el código no solo manejaba el caso de éxito sino también los posibles errores. En todos los ejemplos basados en React Router desde entonces, se ha omitido el manejo de errores. El manejo de errores no se discutió hasta este punto porque, si bien React Router juega un papel importante en el manejo de errores, es vital obtener primero una comprensión sólida de cómo funciona React Router en general y cómo ayuda con la obtención de datos. Pero, por supuesto, los errores no siempre se pueden evitar y definitivamente no deben ignorarse.

Afortunadamente, el manejo de errores también es muy sencillo y fácil cuando se utilizan las capacidades de datos de React Router. Puedes configurar una propiedad **`errorElement`** en las definiciones de tus rutas y definir el elemento que debe renderizarse cuando ocurre un error:

```javascript
// ... other imports 
import Error from './components/Error.jsx'; 

const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, 
    errorElement: <Error />, 
    children: [ 
      { index: true, element: <Welcome /> }, 
      { 
        path: '/posts', 
        id: 'posts', 
        element: <PostsLayout />, 
        loader: postsLoader, 
        children: [ 
          { index: true, element: <Posts /> }, 
          { path: ':id', element: <PostDetails /> }, 
        ], 
      }, 
    ], 
  }, 
]);
```

Esta propiedad `errorElement` se puede configurar en cualquier definición de ruta de tu elección, o incluso en múltiples definiciones de ruta simultáneamente. **React Router renderizará el `errorElement` de la ruta más cercana al lugar donde se lanzó el error**.

En el fragmento anterior, sin importar qué ruta produjera un error, siempre se mostraría el `errorElement` de la ruta raíz (ya que esa es la única definición de ruta con un `errorElement`). Pero si también agregaras un `errorElement` a la ruta `/posts`, y la ruta `:id` produjera un error, se mostraría en pantalla el `errorElement` de la ruta `/posts`, de la siguiente manera:

```javascript
const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, 
    errorElement: <Error />, // for all errors not handled elsewhere 
    children: [ 
      { index: true, element: <Welcome /> }, 
      { 
        path: '/posts', 
        id: 'posts', 
        element: <PostsLayout />, // used if /posts or /posts/:id throws an error 
        errorElement: <PostsError />, // handles /posts related errors 
        loader: postsLoader, 
        children: [ 
          { index: true, element: <Posts /> }, 
          { path: ':id', element: <PostDetails /> }, 
        ], 
      }, 
    ], 
  }, 
]);
```

Esto te permite, como desarrollador, configurar un control de errores detallado y granular.

Dentro del componente utilizado como valor para el `errorElement`, puedes obtener acceso al error que se lanzó a través del Hook **`useRouteError()`**:

```javascript
import { useRouteError } from 'react-router-dom'; 

function Error() { 
  const error = useRouteError(); 
  return ( 
    <> 
      <h1>Oh no!</h1> 
      <p>An error occurred</p> 
      <p>{error.message}</p> 
    </> 
  ); 
} 

export default Error;
```

Con esta solución de manejo de errores simple pero efectiva, React Router te permite evitar administrar los estados de error tú mismo. En su lugar, simplemente defines un elemento estándar de React (a través de la prop `element`) que debe mostrarse cuando las cosas van bien y un `errorElement` que se mostrará si las cosas salen mal.

---

### Sección 6: Avanzando hacia el envío de datos

Hasta ahora, has aprendido mucho sobre la obtención de datos. Pero como se mencionó anteriormente en este capítulo, React Router también ayuda con el envío de datos.

Considera el siguiente componente de ejemplo:

```javascript
function NewPost() { 
  return ( 
    <form id="post-form"> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      <p> 
        <label htmlFor="text">Text</label> 
        <textarea id="text" name="text" rows={3} /> 
      </p> 
      <button>Save Post</button> 
    </form> 
  ); 
} 

export default NewPost;
```

Este componente renderiza un elemento `<form>` que permite a los usuarios ingresar los detalles de una nueva publicación. Debido a la siguiente configuración de ruta, el componente se muestra cada vez que la ruta `/posts/new` se activa:

```javascript
const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, 
    errorElement: <Error />, 
    children: [ 
      { index: true, element: <Welcome /> }, 
      { 
        path: '/posts', 
        id: 'posts', 
        element: <PostsLayout />, 
        loader: postsLoader, 
        children: [ 
          { index: true, element: <Posts /> }, 
          { path: ':id', element: <PostDetails /> }, 
          { path: 'new', element: <NewPost /> }, 
        ], 
      }, 
    ], 
  }, 
]);
```

Sin las funciones relacionadas con datos de React Router, podrías manejar el envío de formularios de esta manera:

```javascript
function NewPost() { 
  const navigate = useNavigate(); 

  async function submitAction(formData) { 
    const enteredTitle = formData.get('title'); 
    const enteredText = formData.get('text'); 
    const postData = { title: enteredTitle, text: enteredText }; 

    await fetch('https://jsonplaceholder.typicode.com/posts', { 
      method: 'POST', 
      body: JSON.stringify(postData), 
      headers: {'Content-Type': 'application/json'} 
    }); 

    navigate('/posts'); 
  } 

  return ( 
    <form action={submitAction}> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      <p> 
        <label htmlFor="text">Text</label> 
        <textarea id="text" rows={3} name="text" /> 
      </p> 
      <button>Save Post</button> 
    </form> 
  ); 
}
```

Al igual que antes al obtener datos, esto requiere que se agregue una buena cantidad de código y lógica a la función del componente: debes extraer manualmente los datos enviados, enviar la solicitud HTTP y navegar a una página diferente después de recibir una respuesta HTTP.

Además, es posible que también debas gestionar el estado de carga y los posibles errores (excluidos en el ejemplo anterior).

Nuevamente, React Router ofrece ayuda: donde se puede agregar una función `loader()` para manejar la carga de datos, se puede definir una función **`action()`** para manejar el envío de datos.

Al usar la nueva función `action()`, el componente de ejemplo anterior se ve así:

```javascript
import { Form, redirect } from 'react-router-dom'; 

function NewPost() { 
  return ( 
    <Form method="post" id="post-form"> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title"/> 
      </p> 
      <p> 
        <label htmlFor="text">Text</label> 
        <textarea id="text" rows={3} name="text" /> 
      </p> 
      <button>Save Post</button> 
    </Form> 
  ); 
} 

export default NewPost; 

export async function action({ request }) { 
  const formData = await request.formData(); 
  const enteredTitle = formData.get('title'); 
  const enteredText = formData.get('text'); 
  const postData = { title: enteredTitle, text: enteredText }; 

  await fetch('https://jsonplaceholder.typicode.com/posts', { 
    method: 'POST', 
    body: JSON.stringify(postData), 
    headers: { 'Content-Type': 'application/json' }, 
  }); 

  return redirect('/posts'); 
}
```

Este código puede tener una longitud similar, pero tiene la ventaja de mover toda la lógica de envío de datos fuera de la función del componente a una función especial `action()`.

Además de la adición de la función `action()`, el fragmento de código de ejemplo incluye los siguientes cambios y características importantes:
- Un componente `<Form>` que se utiliza en lugar de `<form>`.
- La prop `method` se establece en `<Form>` (en `"post"`).
- Los datos enviados se extraen como `FormData` llamando a `request.formData()`.
- El usuario es redirigido a través de una función `redirect()` recién agregada (en lugar de `useNavigate()` y `navigate()`).

#### Trabajando con `action()` y datos de formulario
Al igual que `loader()`, `action()` es una función especial que se puede agregar a las definiciones de rutas, de la siguiente manera:

```javascript
import NewPost, { action as newPostAction } from './components/NewPost.jsx'; 
// ... 
{ path: 'new', element: <NewPost />, action: newPostAction },
```

Con la prop `action` configurada en una definición de ruta, la función asignada se llama automáticamente cada vez que se envía un `<Form>` (¡no `<form>`!) que apunta a esta ruta. `Form` es un componente proporcionado por React Router que debe usarse en lugar del elemento predeterminado `<form>`.

Internamente, `Form` utiliza el elemento `<form>` predeterminado pero evita el comportamiento predeterminado del navegador de crear y enviar una solicitud HTTP al enviar el formulario. En su lugar, React Router crea un objeto `FormData` y llama a la función `action()` definida para la ruta a la que se dirige el `<Form>`, pasándole un objeto `request`, basado en la interfaz integrada `Request`. El objeto de solicitud pasado contiene los datos del formulario generados por React Router. Más adelante en este capítulo, en la sección *Controlar qué `<Form>` activa qué Action*, aprenderás a controlar qué función `action()` de qué ruta ejecutará React Router.

> [!NOTE]
> Manejar el envío de formularios con la ayuda de "acciones" puede sonar familiar: el Capítulo 9, *Manejo de Entradas de Usuario y Formularios con Form Actions*, analizó un concepto similar.
> Pero mientras que el Capítulo 9 analizó una característica integrada en React (que no estaba relacionada ni dependía del enrutamiento), este capítulo explora un concepto central de React Router.
> En última instancia, puedes usar cualquiera de los dos enfoques para manejar el envío de formularios. O no usar ninguno de los dos y manejar el evento `submit` manualmente a través de `onSubmit`.
> Pero al usar enrutamiento con React Router, a menudo obtendrás un código más limpio y conciso que se integra perfectamente con otras características de enrutamiento como redirecciones al usar el componente `<Form>` y la función `action()` de React Router.

El objeto de datos del formulario que se crea al llamar a `request.formData()` incluye todos los valores de entrada del formulario ingresados en el formulario enviado. Para registrarse, un elemento de entrada como `<input>`, `<select>` o `<textarea>` debe tener asignado el atributo `name`. Los valores establecidos para esos atributos `name` se pueden utilizar más adelante para extraer los datos ingresados.

El objeto `request` (que contiene los datos del formulario) recibido por la función `action()` es creado por React Router cuando se envía el formulario.

El componente `Form` define el método HTTP del objeto de solicitud. Al configurar la prop `method` de `Form` en `"get"` (el valor predeterminado) o `"post"`, controlas lo que sucede cuando se envía el formulario:
- Al configurar `method="get"` (o cuando no se configura ningún método), se producirá una navegación de URL normal, tal como si se hiciera clic en un enlace a una determinada ruta. En ese caso, los valores del formulario ingresados se codificarán como parámetros de búsqueda de la URL (*query parameters*).
- Para activar una función `action()`, el `method` de `<Form>` debe establecerse en `"post"`.

Sin embargo, es importante comprender que la solicitud no se envía a través de HTTP ya que `action()`, al igual que `loader()` o la función del componente, todavía se ejecuta en el navegador y no en un servidor.

La función `action()` luego recibe un objeto con una propiedad `request` que contiene el objeto de solicitud creado con los datos del formulario incluidos. Este objeto de solicitud se puede utilizar para extraer los valores introducidos en los campos de entrada del formulario de la siguiente manera:

```javascript
export async function action({ request }) { 
  const formData = await request.formData(); 
  const postData = Object.fromEntries(formData); 
  // ... 
}
```

El método integrado `formData()` produce una Promesa que se resuelve en un objeto `FormData` que ofrece un método `get()` que se puede utilizar para obtener un valor ingresado por su identificador (es decir, por el valor del atributo `name` establecido en el elemento de entrada). Por ejemplo, el valor ingresado en `<input name="title">` se podría recuperar mediante `formData.get('title')`.

Alternativamente, puedes seguir el enfoque elegido en el fragmento de código anterior y convertir el objeto `formData` en un objeto clave-valor simple a través de `Object.fromEntries(formData)`. Este objeto (`postData`, en el ejemplo anterior) contiene los nombres establecidos en los elementos de entrada del formulario como propiedades y los valores ingresados como valores para esas propiedades (lo que significa que `postData.title` produciría el valor ingresado en `<input name="title">`).

> [!NOTE]
> React Router también admite los otros verbos HTTP principales (`"patch"`, `"put"` y `"delete"`), y configurar `method` en uno de estos verbos de hecho también activará la función `action()`.
> Esto puede resultar útil cuando se trabaja con varios formularios que deberían activar el mismo `action()`. Al usar diferentes métodos, puedes usar una sola acción para ejecutar código diferente según el valor extraído de `request.method` dentro de la función `action()`.
> Pero vale la pena señalar que el uso de métodos distintos de `'get'` y `'post'` no está en línea con el estándar HTML. Por lo tanto, React Router podría eliminar el soporte para estos métodos en el futuro.
> Por lo tanto, cuando trabajes con múltiples formularios que activan la misma acción, una solución más estable puede ser incluir un campo de entrada oculto con un identificador único (por ejemplo, `<input type="hidden" name="_method" value="DELETE">`). Este valor luego se puede extraer y usar (por ejemplo, en una declaración `if`) en la función `action()`.

Los datos extraídos luego se pueden utilizar para cualquier operación de tu elección. Podría ser un paso de validación adicional o una solicitud HTTP enviada a alguna API de backend, donde los datos pueden almacenarse en una base de datos o archivo:

```javascript
export async function action({ request }) { 
  const formData = await request.formData(); 
  const postData = Object.fromEntries(formData); 

  await fetch('https://jsonplaceholder.typicode.com/posts', { 
    method: 'POST', 
    body: JSON.stringify(postData), 
    headers: { 'Content-Type': 'application/json' }, 
  }); 

  return redirect('/posts'); 
}
```

Finalmente, una vez realizados todos los pasos previstos, la función `action()` debe devolver un valor: cualquier valor de cualquier tipo, pero al menos `null`. **No se permite no devolver nada** (es decir, omitir la declaración `return`). Aunque, al igual que con la función `loader()`, también puedes devolver una respuesta, por ejemplo, una respuesta de redirección como esta:

```javascript
export async function action({ request }) { 
  // action logic … 
  return new Response("", { status: 302, headers: { Location: '/posts' } }); 
}
```

De hecho, para las acciones, es muy probable que desees navegar a una página diferente una vez que se haya realizado la acción (por ejemplo, una vez que se haya enviado una solicitud HTTP a una API). Esto puede ser necesario para alejar al usuario de la página de entrada de datos a una página que muestre todas las entradas de datos disponibles (por ejemplo, de `/posts/new` a `/posts`).

Para simplificar este patrón común, React Router proporciona una función **`redirect()`** que produce un objeto de respuesta que hace que React Router cambie a una ruta diferente. Por lo tanto, puedes devolver el resultado de llamar a `redirect()` en tu función `action()` para asegurarte de que el usuario navegue a una página diferente. Es el equivalente a llamar a `navigate()` (a través de `useNavigate()`) al manejar envíos de formularios manualmente:

```javascript
export async function action({ request }) { 
  // action logic … 
  return redirect('/posts') 
}
```

En este fragmento, se utiliza la función `redirect()` de React Router en lugar de construir manualmente un objeto `Response`.

#### Devolver datos en lugar de redirigir
Como se mencionó, tus funciones `action()` pueden devolver cualquier cosa. No tienes que devolver un objeto de respuesta. Si bien es bastante común devolver una respuesta de redirección, es posible que ocasionalmente desees devolver algunos datos sin procesar.

Un escenario en el que quizás no desees redirigir al usuario es después de validar la entrada del usuario. Dentro de la función `action()`, antes de enviar los datos ingresados a alguna API, es posible que desees validar primero los valores proporcionados. Si se detecta un valor no válido (como un título vacío), normalmente se logra una excelente experiencia de usuario manteniendo al usuario en la ruta con el `<Form>`. Los valores ingresados por el usuario no deben borrarse ni perderse; en cambio, el formulario debe actualizarse para presentar información útil sobre errores de validación al usuario. Esta información se puede pasar desde `action()` a la función del componente para que se pueda mostrar allí (por ejemplo, junto a los campos de entrada del formulario).

En situaciones como esta, puedes devolver un valor "normal" (es decir, no una respuesta de redirección) desde tu función `action()`:

```javascript
export async function action({ request }) { 
  const formData = await request.formData(); 
  const postData = Object.fromEntries(formData); 

  let validationErrors = []; 

  if (postData.title.trim().length === 0) { 
    validationErrors.push('Invalid post title provided.') 
  } 
  if (postData.text.trim().length === 0) { 
    validationErrors.push('Invalid post text provided.') 
  } 

  if (validationErrors.length > 0) { 
    return validationErrors; 
  } 

  await fetch('https://jsonplaceholder.typicode.com/posts', { 
    method: 'POST', 
    body: JSON.stringify(postData), 
    headers: { 'Content-Type': 'application/json' }, 
  }); 

  return redirect('/posts'); 
}
```

En este ejemplo, se devuelve un array `validationErrors` si los valores de título o texto ingresados están vacíos.

Los datos devueltos por una función `action()` se pueden utilizar en el componente de ruta (o en cualquier otro componente anidado) a través del Hook **`useActionData()`**:

```javascript
import { Form, redirect, useActionData } from 'react-router-dom'; 

function NewPost() { 
  const validationErrors = useActionData(); 

  return ( 
    <Form method="post" id="post-form"> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      <p> 
        <label htmlFor="text">Text</label> 
        <textarea id="text" name="text" rows={3} /> 
      </p> 
      <ul> 
        {validationErrors && validationErrors.map((err) => <li key={err}>{err}</li>)} 
      </ul> 
      <button>Save Post</button> 
    </Form> 
  ); 
}
```

`useActionData()` funciona de manera muy similar a `useLoaderData()`, pero a diferencia de `useLoaderData()`, no se garantiza que produzca ningún dato. Esto se debe a que mientras que las funciones `loader()` siempre se llaman antes de que se renderice el componente de ruta, la función `action()` solo se llama una vez que se envía el `<Form>`.

En este ejemplo, `useActionData()` se utiliza para obtener acceso a los `validationErrors` devueltos por `action()`. Si `validationErrors` es verdadero (es decir, no es `undefined`), el array se asignará a una lista de elementos de error que se muestran al usuario:

**Figura 14.4**: Los errores de validación se muestran debajo de los campos de entrada.

La función `action()` es, por lo tanto, bastante versátil en el sentido de que puedes usarla para realizar una acción y redirigir a otra página, así como para realizar más de una operación y devolver diferentes valores para diferentes casos de uso.

#### Controlar qué `<Form>` activa qué Action
Anteriormente en este capítulo, en la sección *Trabajando con `action()` y datos de formulario*, aprendiste que cuando se usa `<Form>` en lugar de `<form>`, React Router ejecutará la función `action()` de destino. ¿Pero a qué función `action()` se dirige `<Form>`?

Por defecto, es la función `action()` asignada a la ruta que también renderiza el formulario (ya sea directamente o mediante algún componente descendiente). Considera esta definición de ruta:

```javascript
{ path: '/posts/new', element: <NewPost />, action: newPostAction }
```

Con esta definición, la función `newPostAction()` se activaría cada vez que se envíe cualquier `<Form>` dentro del componente `NewPost` (o cualquier componente anidado).

En muchos casos, este comportamiento predeterminado es exactamente lo que deseas. Pero también puedes dirigirte a funciones `action()` definidas en otras rutas configurando la prop `action` en `<Form>` con la ruta de la ruta que contiene la `action()` que debe ejecutarse:

```javascript
// form rendered in a component that belongs to /posts 
<Form method="post" action="/save-data"> 
  ... 
</Form>
```

Este formulario haría que React Router ejecutara la acción que pertenece a la ruta `/save-data`, a pesar de que el componente `<Form>` se pueda renderizar como parte de un componente que pertenece a una ruta diferente (por ejemplo, `/posts`).

Sin embargo, vale la pena señalar que dirigirse a una ruta diferente conducirá a una transición de página a la ruta de esa ruta, incluso si tu acción no devuelve una respuesta de redirección. En una sección posterior de este capítulo, titulada *Obtención y envío de datos detrás de escena*, aprenderás cómo se puede evitar ese comportamiento.

#### Reflejar el estado de navegación actual
Después de enviar un formulario, la función `action()` que se activa puede necesitar algún tiempo para realizar todas las operaciones previstas. El envío de solicitudes HTTP a APIs en particular puede demorar hasta unos pocos segundos.

Por supuesto, no es una gran experiencia de usuario si el usuario no recibe ningún comentario sobre el estado actual de envío de datos. No queda claro de inmediato si ocurrió algo después de hacer clic en el botón de envío.

Por esa razón, es posible que desees mostrar una animación de carga o actualizar el texto del botón mientras se ejecuta la función `action()`. De hecho, una forma común de proporcionar comentarios a los usuarios es deshabilitar el botón de envío y cambiar su texto de la siguiente manera:

**Figura 14.5**: El botón de envío está atenuado (*disabled*).

Puedes obtener el estado actual de React Router (es decir, si actualmente está en transición a otra ruta o ejecutando una función `action()`) a través del Hook **`useNavigation()`**. Este Hook proporciona un objeto `navigation` que contiene varias piezas de información relacionada con el enrutamiento.

Lo más importante es que este objeto tiene una propiedad **`state`** que produce una cadena que describe el estado de navegación actual. Esta propiedad se establece en uno de los siguientes tres valores posibles:
- **`submitting`**: Si se está ejecutando actualmente una función `action()`.
- **`loading`**: Si se está ejecutando actualmente una función `loader()` (por ejemplo, debido a una respuesta `redirect()`).
- **`idle`**: Si no se está ejecutando ninguna función `action()` o `loader()` en este momento.

Por lo tanto, puedes usar esta propiedad `state` para averiguar si React Router está navegando actualmente a una página diferente o ejecutando una acción. Por consiguiente, el botón de envío se puede actualizar como se muestra en la captura de pantalla anterior a través de este código:

```javascript
import { Form, redirect, useActionData, useNavigation } from 'react-router-dom'; 

function NewPost() { 
  const validationErrors = useActionData(); 
  const navigation = useNavigation(); 

  const isSubmitting = navigation.state !== 'idle'; 

  return ( 
    <Form method="post" id="post-form"> 
      <p> 
        <label htmlFor="title">Title</label> 
        <input type="text" id="title" name="title" /> 
      </p> 
      <p> 
        <label htmlFor="text">Text</label> 
        <textarea id="text" name="text" rows={3} /> 
      </p> 
      <ul> 
        {validationErrors && validationErrors.map((err) => <li key={err}>{err}</li>)} 
      </ul> 
      <button disabled={isSubmitting}> 
        {isSubmitting ? 'Saving...' : 'Save Post'} 
      </button> 
    </Form> 
  ); 
}
```

En este ejemplo, la constante `isSubmitting` es `true` si el estado de navegación actual es distinto de `'idle'`. Esta constante se utiliza luego para deshabilitar el botón de envío (a través del atributo `disabled`) y ajustar el texto del botón.

#### Envío de formularios de forma programática
En algunos casos, no querrás activar instantáneamente un `action()` cuando se envía un formulario, por ejemplo, si necesitas pedir confirmación al usuario primero, como cuando se activan acciones que eliminan o actualizan datos.

Para tales escenarios, React Router te permite enviar un formulario (y por lo tanto activar una función `action()`) mediante programación. En lugar de utilizar el componente `Form` proporcionado por React Router, manejas el envío del formulario manualmente utilizando el elemento `<form>` predeterminado. Como parte de tu código, puedes usar una función `submit()` proporcionada por el Hook **`useSubmit()`** de React Router para activar `action()` manualmente una vez que estés listo para ello.

Considera este ejemplo:

```javascript
import { 
  redirect, 
  useParams, 
  useRouteLoaderData, 
  useSubmit, 
} from 'react-router-dom'; 

function PostDetails() { 
  const params = useParams(); 
  const posts = useRouteLoaderData('posts'); 
  const post = posts.find((post) => post.id.toString() === params.id); 
  const submit = useSubmit(); 

  function handleSubmit(event) { 
    event.preventDefault(); 
    const proceed = window.confirm('Are you sure?'); 

    if (proceed) 
    { 
      submit( 
        { message: 'Your submitted data, if needed' }, 
        { 
          method: 'post', 
        } 
      ); 
    } 
  } 

  return ( 
    <div id="post-details"> 
      <h1>{post.title}</h1> 
      <p>{post.body}</p> 
      <form onSubmit={handleSubmit}> 
        <button>Delete</button> 
      </form> 
    </div> 
  ); 
} 

export default PostDetails; 

// action must be added to route definition! 
export async function action({ request }) { 
  const formData = await request.formData(); 
  console.log(formData.get('message')); 
  console.log(request.method); 
  return redirect('/posts'); 
}
```

En este ejemplo, la `action()` se activa manualmente mediante el envío programático de datos a través de la función `submit()` proporcionada por `useSubmit()`. Este enfoque es necesario ya que de otro modo sería imposible pedir confirmación al usuario (a través del método `window.confirm()` del navegador).

Debido a que los datos se envían mediante programación, se debe utilizar el elemento `<form>` predeterminado y el evento de envío debe manejarse manualmente. Como parte de este proceso, el comportamiento predeterminado del navegador de enviar una solicitud HTTP también debe evitarse manualmente.

Por lo general, es preferible utilizar `<Form>` en lugar del envío programático. Pero en ciertas situaciones, como en el ejemplo anterior, poder controlar el envío de formularios manualmente puede resultar muy útil.

#### Obtención y envío de datos detrás de escena
También hay situaciones en las que es posible que necesites activar una acción o cargar datos **sin causar una transición de página**.

Un botón de "Me gusta" (*Like*) sería un ejemplo. Cuando se hace clic en él, se debe activar un proceso en segundo plano (como almacenar información sobre el usuario y la publicación que le gustó), pero el usuario no debe ser dirigido a una página diferente:

**Figura 14.6**: Un botón de Me gusta debajo de una publicación.

Para lograr este comportamiento, podrías envolver el botón en un `<Form>` y, al final de la función `action()`, simplemente redirigir de regreso a la página que ya está activa.

Pero técnicamente, esto todavía conduciría a una acción de navegación adicional. Por lo tanto, se ejecutarían las funciones `loader()` y podrían ocurrir otros posibles efectos secundarios (la posición de desplazamiento actual podría perderse, por ejemplo). Por esa razón, es posible que desees evitar este tipo de comportamiento.

Afortunadamente, React Router ofrece una solución: el Hook **`useFetcher()`**, que produce un objeto que contiene un método `submit()`. A diferencia de la función `submit()` proporcionada por `useSubmit()`, el método `submit()` producido por `useFetcher()` está diseñado para **activar acciones (o funciones `loader()`) sin iniciar una transición de página**.

Un botón de Me gusta, como se describió anteriormente, se puede implementar de la siguiente manera (con la ayuda de `useFetcher()`):

```javascript
import { 
  // ... other imports 
  useFetcher, 
} from 'react-router-dom'; 
import { FaHeart } from 'react-icons/fa'; 

function PostDetails() { 
  // ... other code & logic 
  const fetcher = useFetcher(); 

  function handleLikePost() { 
    fetcher.submit(null, { 
      method: 'post', 
      action: `/posts/${post.id}/like`, // targeting an action on another route 
    }); 
  } 

  return ( 
    <div id="post-details"> 
      <h1>{post.title}</h1> 
      <p>{post.body}</p> 
      <div className="actions"> 
        <button className="icon-btn" onClick={handleLikePost}> 
          <FaHeart /> <span>Like this post</span> 
        </button> 
        <form onSubmit={handleSubmit}> 
          <button>Delete</button> 
        </form> 
      </div> 
    </div> 
  ); 
}
```

El objeto `fetcher` devuelto por `useFetcher()` tiene varias propiedades. Por ejemplo, también contiene propiedades que brindan información sobre el estado actual de la acción o el cargador activado (incluidos los datos que se hayan devuelto).

Pero este objeto también incluye dos métodos importantes:
- **`load()`**: Para activar la función `loader()` de una ruta (por ejemplo, `fetcher.load('/route-path')`).
- **`submit()`**: Para activar una función `action()` con los datos y la configuración proporcionados.

En el fragmento de código anterior, se llama al método `submit()` para activar la acción definida en la ruta `/posts/<post-id>/like`. Sin `useFetcher()` (es decir, al usar `useSubmit()` o `<Form>`), React Router cambiaría a la ruta de la ruta seleccionada al activar su acción. Con `useFetcher()`, esto se evita y la acción de esa ruta se puede llamar desde dentro de otra ruta (lo que significa que la acción definida para `/posts/<post-id>/like` se llama mientras la ruta `/posts/<post-id>` está activa).

Esto también te permite definir **rutas que no renderizan ningún elemento** (es decir, en las que no hay un componente de página) y, en su lugar, solo contienen una función `loader()` o `action()`. Por ejemplo, el archivo de ruta `/posts/<post-id>/like` (`pages/like.js`) se ve así:

```javascript
// there is no component function in this file! 
export function action({ params }) { 
  console.log('Triggered like action.'); 
  console.log(`Liking post with id ${params.id}.`); 
  // Do anything else 
  // May return data or response, including redirect() if needed 
  return null; // something must be returned, even if it's just null 
}
```

Como se menciona en el fragmento de código, se puede devolver cualquier dato en esta acción. Pero debes devolver al menos `null`; evitar la declaración `return` y no devolver nada no está permitido y causará un error.

Se registra como una ruta de la siguiente manera:

```javascript
import { action as likeAction } from './pages/like.js'; 
// ... 
{ path: ':id/like', action: likeAction },
```

Esto funciona porque esta `action()` solo se activa a través del método `submit()` proporcionado por `useFetcher()`. `<Form>` y la función `submit()` producida por `useSubmit()` iniciarían en cambio una transición de ruta a `/posts/<post-id>/like`. Sin la propiedad `element` establecida en la definición de la ruta, esta transición conduciría a una página vacía, como se muestra aquí:

**Figura 14.7**: Se muestra una página vacía (anidada), junto con un mensaje de advertencia.

Debido a la flexibilidad adicional que ofrece, `useFetcher()` puede ser muy útil al crear interfaces de usuario altamente interactivas. No está pensado como un reemplazo de `useSubmit()` o `<Form>`, sino más bien como una herramienta adicional para situaciones en las que no se requiere ni se desea una transición de ruta.

#### Diferir la carga de datos (*Deferring Data Loading*)
Hasta este punto del capítulo, todos los ejemplos de obtención de datos han asumido que una página solo debería mostrarse una vez que se hayan obtenido todos sus datos. Es por eso que nunca se administró ningún estado de carga (y, por lo tanto, no se mostró ningún contenido alternativo de carga).

En muchas situaciones, este es exactamente el comportamiento que deseas, ya que a menudo no tiene sentido mostrar un indicador de carga durante una fracción de segundo para luego reemplazarlo con los datos reales de la página.

Pero también hay situaciones en las que el comportamiento opuesto podría ser deseable, por ejemplo, si sabes que una determinada página tardará bastante en cargar sus datos (posiblemente debido a una consulta de base de datos compleja que debe ejecutarse en el backend) o si tienes una página que carga diferentes fragmentos de datos y algunos fragmentos son mucho más lentos que otros.

En tales escenarios, puede tener sentido renderizar el componente de página aunque todavía falten algunos datos. React Router también admite este caso de uso al permitirte **diferir la carga de datos (*defer data loading*)**, lo que, a su vez, permite que el componente de la página se renderice antes de que los datos estén disponibles.

Diferir la carga de datos es tan simple como **devolver una promesa desde el loader** (en lugar de usar `await` dentro de él):

```javascript
// ... other imports 
export async function loader() { 
  return { posts: getPosts() }; 
}
```

En este ejemplo, `getPosts()` es una función que devuelve una Promesa (lenta):

```javascript
async function getPosts() { 
  const response = await fetch( 
    'https://jsonplaceholder.typicode.com/posts' 
  ); 
  await wait(3); // utility function, simulating a slow response 
  if (!response.ok) { 
    throw new Error('Could not fetch posts'); 
  } 
  const data = await response.json(); 
  return data; 
}
```

React Router te permite devolver promesas directamente. Al hacerlo, puedes esperar los valores reales producidos por esas promesas en el código del lado del cliente.

Dentro de la función del componente donde se usa `useLoaderData()`, también debes usar un nuevo componente proporcionado por React Router: el componente **`Await`**. Se utiliza de esta manera:

```javascript
import { Suspense } from 'react'; 
import { Await } from 'react-router-dom'; 
// ... other imports 

function PostsLayout() { 
  const data = useLoaderData(); 

  return ( 
    <div id="posts-layout"> 
      <nav> 
        <Suspense fallback={<p>Loading posts...</p>}> 
          <Await resolve={data.posts}> 
            {(loadedPosts) => <PostsList posts={loadedPosts} />} 
          </Await> 
        </Suspense> 
      </nav> 
      <main> 
        <Outlet /> 
      </main> 
    </div> 
  ); 
}
```

El elemento `<Await>` toma una prop **`resolve`** que recibe un valor de tipo Promesa de los datos del loader. Está envuelto por el componente `<Suspense>` proporcionado por React.

El valor pasado a `resolve` es una Promesa que se almacenó en el objeto devuelto por la función `loader()`. Allí, se utilizó una clave llamada `posts` para contener esa Promesa. El valor de esa clave fue la Promesa devuelta por `getPosts()`. Es esta Promesa la que se pasa como valor a `resolve` a través de `<Await resolve={data.posts}>`. Si se utilizara un nombre de clave diferente (por ejemplo, `blogPosts`), ese nombre de clave tendría que referenciarse al configurar `resolve` (por ejemplo, `<Await resolve={data.blogPosts}>`).

`Await` espera automáticamente a que la Promesa se resuelva antes de llamar a la función que se pasa a `<Await>` como hijo (es decir, la función pasada entre las etiquetas de apertura y cierre de `<Await>`). Esta función es ejecutada por React Router una vez que los datos de la operación diferida están disponibles. Por lo tanto, dentro de esa función, se recibe `loadedPosts` como parámetro y se pueden renderizar los elementos finales de la interfaz de usuario.

El componente `Suspense` que se utiliza como envoltorio alrededor de `<Await>` define algún contenido alternativo que se renderiza mientras los datos diferidos aún no están disponibles. En el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, el componente `Suspense` se utilizó para mostrar contenido alternativo hasta que se descargara el código faltante. Ahora, se utiliza para acortar el tiempo hasta que los datos requeridos estén disponibles.

Como se muestra en la Figura 14.8, al devolver una Promesa (y usar `<Await>`) de esta manera, otras partes del sitio web que no se cargan a través de `<Await>` ya se renderizan y muestran mientras se esperan los datos de las publicaciones.

**Figura 14.8**: Los detalles de la publicación ya son visibles mientras se carga la lista de publicaciones.

Otra gran ventaja de devolver una Promesa y esperarla en el código del lado del cliente es que puedes combinar fácilmente múltiples procesos de obtención y controlar qué procesos deben diferirse y cuáles no. Por ejemplo, una ruta puede estar obteniendo diferentes fragmentos de datos. Si solo un proceso tiende a ser lento, puedes diferir solo el lento de la siguiente manera:

```javascript
export async function loader() { 
  return { 
    posts: getPosts(), // slow operation => deferred 
    userData: await getUserData() // fast operation => NOT deferred 
  }; 
}
```

En este ejemplo, `getUserData()` no se difiere porque se agrega la palabra clave `await` delante de él. Por lo tanto, JavaScript espera a que esa Promesa (la Promesa devuelta por `getUserData()`) se resuelva antes de regresar de `loader()`. Por lo tanto, el componente de ruta se renderiza una vez que finaliza `getUserData()` pero antes de que termine `getPosts()`.

---

### Sección 7: Resumen y puntos clave

- React Router puede ayudarte con la obtención y el envío de datos.
- Puedes registrar funciones **`loader()`** para tus rutas, lo que hace que la obtención de datos se inicialice a medida que una ruta se activa.
- Las funciones `loader()` devuelven datos (o respuestas que envuelven datos) a los que se puede acceder a través de **`useLoaderData()`** en las funciones de tus componentes.
- Los datos de los loaders se pueden utilizar entre componentes a través de **`useRouteLoaderData()`**.
- También puedes registrar funciones **`action()`** en tus rutas que se activan tras el envío de formularios.
- Para activar funciones `action()`, debes usar el componente **`<Form>`** de React Router o enviar datos mediante programación a través de **`useSubmit()`** o **`useFetcher()`**.
- **`useFetcher()`** se puede utilizar para cargar o enviar datos sin iniciar una transición de ruta.
- Al obtener datos lentos, puedes devolver promesas sin usar `await` en el `loader()` para **diferir la carga** de algunos o todos los datos de una ruta (utilizando `<Await>` y `<Suspense>`).

---

### Sección 8: ¿Qué sigue?

Obtener y enviar datos son tareas extremadamente comunes, especialmente cuando se crean aplicaciones de React más complejas.

Por lo general, esas tareas están estrechamente conectadas con las transiciones de ruta, y React Router es la herramienta perfecta para manejar este tipo de operaciones. Es por eso que el paquete React Router ofrece potentes capacidades de gestión de datos que simplifican enormemente estos procesos.

En este capítulo, aprendiste cómo React Router te ayuda a obtener o enviar datos y qué funciones avanzadas te ayudan a manejar escenarios de manipulación de datos tanto básicos como más complejos.

Por lo tanto, este capítulo concluye la lista de características centrales de React Router que necesitas conocer.

Los próximos capítulos explorarán las capacidades del lado del servidor de React y cómo puedes crear aplicaciones *fullstack* con React, cargar datos en un servidor y utilizar el framework **Next.js**.

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se encuentran en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/exercises/questions-answers.md):

1. ¿Cómo se relacionan la obtención y el envío de datos con el enrutamiento?
2. ¿Cuál es el propósito de las funciones `loader()`?
3. ¿Cuál es el propósito de las funciones `action()`?
4. ¿Cuál es la diferencia entre `<Form>` y `<form>`?
5. ¿Cuál es la diferencia entre `useSubmit()` y `useFetcher()`?
6. ¿Cuál es la idea detrás de devolver promesas en lugar de esperarlas (`await`) en un `loader()`?

---

### Sección 10: Aplica lo aprendido

Aplica tus conocimientos sobre enrutamiento, combinados con la manipulación de datos, a la siguiente actividad.

#### Actividad 14.1: Una aplicación de tareas pendientes (*To-Dos App*)
En esta actividad, tu tarea es crear una aplicación web básica de lista de tareas pendientes que permita a los usuarios administrar sus tareas diarias. La página terminada debe permitir a los usuarios agregar elementos de tareas pendientes, actualizar elementos de tareas pendientes, eliminar elementos de tareas pendientes y ver una lista de elementos de tareas pendientes.

Se deben admitir las siguientes rutas:
- `/`: La página principal, responsable de cargar y mostrar una lista de tareas pendientes.
- `/new`: Una página, abierta como un modal encima de la página principal, que permite a los usuarios agregar una nueva tarea pendiente.
- `/:id`: Una página, también abierta como un modal encima de la página principal, que permite a los usuarios actualizar o eliminar una tarea pendiente seleccionada.

Si aún no existen elementos de tareas pendientes, se debe mostrar un mensaje informativo adecuado en la página `/`. Si los usuarios intentan visitar `/:id` con un ID de tarea no válido, se debe mostrar un modal de error.

> [!NOTE]
> Para esta actividad, no hay una API de backend que puedas utilizar. En su lugar, utiliza `localStorage` para gestionar los datos de las tareas pendientes. Ten en cuenta que las funciones `loader()` y `action()` se ejecutan en el lado del cliente y, por lo tanto, pueden utilizar cualquier API del navegador, incluida `localStorage`.
> Encontrarás implementaciones de ejemplo para agregar, actualizar, eliminar y obtener elementos de tareas pendientes de `localStorage` en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/activities/practice-1/src/data/todos.js](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/activities/practice-1/src/data/todos.js).
> Además, no te confundas con las páginas que se abren como modales encima de otras páginas: en última instancia, son simplemente páginas anidadas, diseñadas como superposiciones modales. En caso de que te quedes atascado, puedes utilizar el componente contenedor `Modal` de ejemplo que se encuentra en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/activities/practice-1/src/components/Modal.jsx](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/activities/practice-1/src/components/Modal.jsx).
> Para esta actividad, puedes escribir todos los estilos CSS por tu cuenta si así lo deseas. Pero si deseas centrarte en la lógica de React y JavaScript, también puedes utilizar el archivo CSS terminado de la solución en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/activities/practice-1/src/index.css](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/14-routing-data/activities/practice-1/src/index.css).
> Si usas ese archivo, exploralo cuidadosamente para asegurarte de comprender qué IDs o clases CSS podrían necesitar agregarse a ciertos elementos JSX de tu solución.

Para completar la actividad, realiza los siguientes pasos:
1. Crea un nuevo proyecto de React e instala el paquete React Router.
2. Crea componentes (con el contenido que se muestra en las capturas de pantalla a continuación) que se cargarán para las tres páginas requeridas. Además, agrega enlaces (o navegación programática) entre estas páginas.
3. Habilita el enrutamiento y agrega las definiciones de ruta para las tres páginas.
4. Crea funciones `loader()` para cargar (y usar) todos los datos necesarios para las páginas individuales.
5. Agrega funciones `action()` para agregar, actualizar y eliminar tareas pendientes.
   - *Consejo*: Si necesitas enviar varios formularios para diferentes acciones desde la misma página, puedes incluir un campo de entrada oculto que establezca algún valor que puedas verificar en tu función de acción, por ejemplo, `<input type="hidden" name="_method" value="DELETE">`. Alternativamente, también puedes configurar `<Form method="delete">` (o configurarlo en `"patch"`, `"put"` u otros verbos HTTP) y verificar `request.method` en tu función `action()`.
6. Agrega control de errores en caso de que falle la carga o el guardado de datos.

Las páginas terminadas deberían verse así:

**Figura 14.9**: La página principal que muestra una lista de tareas pendientes.

**Figura 14.10**: La página /new, abierta como un modal, que permite a los usuarios agregar una nueva tarea pendiente.

**Figura 14.11**: La página /:id, también abierta como un modal, que permite a los usuarios editar o eliminar una tarea pendiente.

**Figura 14.12**: Un mensaje de información, que se muestra si no se encontraron tareas pendientes.

> [!NOTE]
> El código completo y la solución para esta actividad se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/14-routing-data/activities/practice-1).
