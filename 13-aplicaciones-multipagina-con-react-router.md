# Parte 2: Construcción de Aplicaciones React Complejas

## Capítulo 13: Aplicaciones Multipágina con React Router

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Construir aplicaciones de página única multipágina (*Multipage Single-Page Applications*) y entender por qué no es una contradicción.
- Usar el paquete React Router para cargar diferentes componentes de React para diferentes rutas de URL.
- Crear rutas estáticas y dinámicas (y entender qué son las rutas en primer lugar).
- Navegar por el sitio web tanto mediante enlaces como mediante comandos programáticos.
- Construir diseños de página anidados (*nested page layouts*).

---

### Sección 1: Introducción

Habiendo trabajado en los primeros doce capítulos de este libro, ahora deberías saber cómo construir componentes de React y aplicaciones web, así como cómo administrar el estado de los componentes y de toda la aplicación, y cómo compartir datos entre componentes (a través de props o contexto).

Pero a pesar de que sabes cómo componer un sitio web de React a partir de múltiples componentes, todos estos componentes se encuentran en la misma página de sitio web única. Claro, puedes mostrar componentes y contenido condicionalmente, pero los usuarios nunca cambiarán a una página diferente. Esto significa que la ruta de la URL nunca cambiará; los usuarios siempre permanecerán en `tu-dominio.com`. Además, en este punto, tus aplicaciones de React no admiten rutas como `tu-dominio.com/products` o `tu-dominio.com/blog/latest`.

> [!NOTE]
> Los Localizadores Uniformes de Recursos (*Uniform Resource Locators* o **URLs**) son referencias a recursos web. Por ejemplo, [https://academind.com/courses](https://academind.com/courses) es una URL que apunta a una página específica del sitio web del autor. En este ejemplo, `academind.com` es el nombre de dominio del sitio web y `/courses` es la ruta a una página específica del sitio web.

Para las aplicaciones de React, puede tener sentido que la ruta del sitio web cargado nunca cambie. Después de todo, en el Capítulo 1, *React – Qué es y por qué*, aprendiste que creas aplicaciones de página única (*Single-Page Applications* o **SPAs**) con React.

Pero aunque tenga sentido, también es una limitación bastante seria.

---

### Sección 2: Una sola página no es suficiente

Tener solo una página significa que los sitios web complejos que normalmente constarían de múltiples páginas (por ejemplo, una tienda en línea con páginas de productos, pedidos y más) se vuelven bastante difíciles de crear con React. Sin múltiples páginas, tienes que recurrir al estado y a los valores condicionales para mostrar contenido diferente en la pantalla.

Pero sin cambiar las rutas de la URL, los visitantes de tu sitio web no pueden compartir enlaces a nada más que a la página de inicio de tu sitio web. Además, cualquier contenido cargado condicionalmente se perderá cuando un nuevo visitante acceda a esa página de inicio. Ese también será el caso si los usuarios simplemente recargan la página en la que se encuentran actualmente. Una recarga obtiene una nueva versión de la página y, por lo tanto, se pierden todos los cambios de estado (y por consiguiente de la interfaz de usuario).

Por estas razones, para la mayoría de los sitios web de React necesitas absolutamente una forma de incluir múltiples páginas (con diferentes rutas de URL) en una sola aplicación de React. Gracias a las características modernas de los navegadores y a un paquete de terceros muy popular, eso es de hecho posible (y es el estándar para la mayoría de las aplicaciones de React).

A través del paquete **React Router**, tu aplicación de React puede escuchar los cambios en la ruta de la URL y mostrar diferentes componentes para diferentes rutas. Por ejemplo, podrías definir las siguientes asignaciones de ruta a componente:
- `<dominio>/` => se carga el componente `<Home />`.
- `<dominio>/products` => se carga el componente `<ProductList />`.
- `<dominio>/products/p1` => se carga el componente `<ProductDetail />`.
- `<dominio>/about` => se carga el componente `<AboutUs />`.

Técnicamente, seguirá siendo una SPA porque todavía solo se envía una página HTML a los usuarios del sitio web. Pero en esa aplicación React de una sola página, el paquete React Router renderiza condicionalmente diferentes componentes en función de las rutas de URL específicas que se visitan. Como desarrollador de la aplicación, no tienes que administrar manualmente este tipo de estado ni renderizar contenido condicionalmente: React Router lo hará por ti. Además, tu sitio web puede manejar diferentes rutas de URL y, por lo tanto, las páginas individuales se pueden compartir o recargar.

---

### Sección 3: Primeros pasos con React Router y definición de rutas

React Router es una biblioteca de React de terceros que se puede instalar en cualquier proyecto de React. Una vez instalada, puedes usar varios componentes en tu código para habilitar las funciones mencionadas anteriormente.

Dentro de tu proyecto de React, el paquete se instala mediante este comando:

```bash
npm install react-router-dom
```

Una vez instalado, puedes importar y utilizar varios componentes (y Hooks) de esa biblioteca.

Para comenzar a admitir múltiples páginas en tu aplicación React, debes configurar el enrutamiento (*routing*) siguiendo los siguientes pasos:
1. Crea diferentes componentes para tus diferentes páginas (por ejemplo, los componentes `Dashboard` y `Orders`).
2. Utiliza la función `createBrowserRouter()` y el componente `RouterProvider` de la biblioteca React Router para habilitar el enrutamiento y definir las rutas que debe admitir la aplicación de React.

En este contexto, el término **enrutamiento (*routing*)** se refiere a que la aplicación de React sea capaz de cargar diferentes componentes para diferentes rutas de URL (por ejemplo, diferentes componentes para las rutas `/` y `/orders`). Una **ruta (*route*)** es una definición que se agrega a la aplicación de React que define la ruta de la URL para la cual se debe renderizar un fragmento JSX predefinido (por ejemplo, el componente `Orders` debe cargarse para la ruta `/orders`).

En una aplicación de React de ejemplo que contiene los componentes `Dashboard` y `Orders`, y donde la biblioteca React Router se instaló a través de `npm install`, puedes habilitar el enrutamiento y la navegación entre estos dos componentes editando el componente raíz (en `src/App.jsx`) de la siguiente manera:

```javascript
import { createBrowserRouter, RouterProvider } from 'react-router-dom'; 
import Dashboard from './routes/Dashboard.jsx'; 
import Orders from './routes/Orders.jsx'; 

const router = createBrowserRouter([ 
  { path: '/', element: <Dashboard /> }, 
  { path: '/orders', element: <Orders /> } 
]); 

function App() { 
  return <RouterProvider router={router} />; 
} 

export default App;
```

> [!NOTE]
> Puedes encontrar el código de ejemplo completo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/01-getting-started-with-routing](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/01-getting-started-with-routing).

En el fragmento de código anterior, se llama a la función `createBrowserRouter()` de React Router para crear un objeto `router` que contiene la configuración de rutas de la aplicación (una lista de rutas disponibles). El array pasado a `createBrowserRouter()` contiene objetos de definición de rutas, donde cada objeto define una ruta (`path`) para la cual debe coincidir la ruta y un elemento (`element`) que debe renderizarse.

El componente `RouterProvider` de React Router se utiliza luego para establecer la configuración del enrutador y definir un lugar para que se rendericen los elementos de la ruta activa.

Puedes pensar en el elemento `<RouterProvider />` como si fuera reemplazado por el contenido definido a través de la propiedad `element` una vez que una ruta se activa. Por lo tanto, la posición del componente `RouterProvider` es importante. En este caso (y probablemente en la mayoría de las aplicaciones de React), es el componente raíz de la aplicación; es decir, React Router debe controlar todo el árbol de componentes de la aplicación.

Si ejecutas la aplicación de React de ejemplo proporcionada (a través de `npm run dev`), verás la siguiente salida en la pantalla:

**Figura 13.1**: Se carga el contenido del componente Dashboard.

El contenido del componente `Dashboard` se muestra en la pantalla si visitas `localhost:5173`. Ten en cuenta que el contenido de la página visible no está definido en el componente `App` (en el fragmento de código compartido anteriormente). En su lugar, solo se agregaron dos definiciones de rutas: una para la ruta `/` (es decir, para `localhost:5173/` o simplemente `localhost:5173`, sin la barra diagonal final; se maneja de la misma manera) y otra para la ruta `/orders` (`localhost:5173/orders`).

> [!NOTE]
> `localhost` es una dirección local que normalmente se utiliza para el desarrollo. Cuando despliegues tu aplicación de React (es decir, la subas a un servidor web), recibirás un dominio diferente o asignarás un dominio personalizado. De cualquier manera, no será `localhost` después del despliegue.
> La parte posterior a `localhost` (`:5173`) define el puerto de red al que se enviará la solicitud. Sin la información de puerto adicional, los puertos 80 o 443 (como puertos HTTP(S) predeterminados) se utilizan automáticamente. Durante el desarrollo, sin embargo, estos no son los puertos que deseas. En su lugar, normalmente usarías puertos como 5173, 8000 u 8080, ya que normalmente no están ocupados por ningún otro proceso del sistema y, por lo tanto, se pueden usar de manera segura. Los proyectos creados a través de Vite normalmente usan el puerto 5173.

Dado que `localhost:5173` se carga de forma predeterminada (al ejecutar `npm run dev`), la primera definición de ruta (`{ path: '/', element: <Dashboard /> }`) se activa. Esta ruta está activa porque su ruta (`'/'`) coincide con la ruta de `localhost:5173` (ya que es lo mismo que `localhost:5173/`).

Como resultado, React Router renderiza el código JSX definido a través de `element` en lugar del componente `<RouterProvider>`. En este caso, esto significa que se muestra el contenido del componente `Dashboard` porque el valor de la propiedad `element` de esta definición de ruta es `<Dashboard />`. Es bastante común usar componentes individuales (como `<Dashboard />`, en este ejemplo), pero puedes establecer cualquier contenido JSX como un valor para la propiedad `element`.

En el ejemplo anterior, no se muestra ninguna página compleja. En su lugar, solo aparece texto en la pantalla. Sin embargo, esto cambiará más adelante en este capítulo.

Pero se vuelve interesante si cambias manualmente la URL de solo `localhost:5173` a `localhost:5173/orders` en la barra de direcciones del navegador. En cualquiera de los capítulos anteriores, esto no habría cambiado el contenido de la página. Pero ahora, con el enrutamiento habilitado y las rutas adecuadas definidas, el contenido de la página sí cambia, como se muestra:

**Figura 13.2**: Para /orders, se muestra el contenido del componente Orders.

Una vez que la URL cambia, el contenido del componente `Orders` se muestra en la pantalla. Nuevamente es solo un texto básico en este primer ejemplo, pero muestra que se renderiza un código diferente para diferentes rutas de URL.

Sin embargo, este ejemplo básico tiene una falla importante (además del contenido bastante aburrido de la página). En este momento, los usuarios deben ingresar las URLs manualmente. Pero, por supuesto, no es así como se utilizan normalmente los sitios web.

#### Añadir navegación de página
Para permitir que los usuarios cambien entre diferentes páginas del sitio web sin editar la barra de direcciones del navegador manualmente, los sitios web normalmente contienen enlaces, típicamente agregados a través del elemento HTML `<a>` (el elemento de anclaje), de esta manera:

```html
<a href="/orders">Past Orders</a>
```

Para este ejemplo, la navegación en la página podría agregarse modificando el código del componente `Dashboard` de la siguiente manera:

```javascript
function Dashboard() { 
  return ( 
    <> 
      <h1>The "Dashboard" route component</h1> 
      <p>Go to the <a href="/orders">Orders page</a>.</p> 
      {/* <p> elements omitted */} 
    </> 
  ); 
} 

export default Dashboard;
```

En este fragmento de código, se ha agregado un enlace a la ruta `/orders`. Por lo tanto, los visitantes del sitio web ahora ven esta página:

**Figura 13.3**: Se ha agregado un enlace de navegación.

Cuando los usuarios del sitio web hacen clic en este enlace, son dirigidos a la ruta `/orders` y el contenido del componente `Orders` se muestra en la pantalla.

Este enfoque funciona pero tiene un defecto importante: **el sitio web se recarga cada vez que un usuario hace clic en el enlace**. Puedes notar que se recarga porque el icono de actualización del navegador cambia brevemente a una cruz cada vez que haces clic en un enlace.

Esto sucede porque el navegador envía una nueva solicitud HTTP al servidor cada vez que se hace clic en un enlace. Aunque el servidor siempre devuelve la misma página HTML única, la página se recarga durante ese proceso (debido a la nueva solicitud HTTP que se envió).

Si bien eso no es un problema en esta página de demostración simple, sería un problema si tuvieras algún estado compartido (por ejemplo, un estado en toda la aplicación administrado a través de contexto) que no deba restablecerse durante un cambio de página. Además, cada nueva solicitud lleva tiempo y obliga al navegador a descargar todos los recursos del sitio web (por ejemplo, archivos de scripts) nuevamente. Aunque esos archivos puedan estar en caché, este es un paso innecesario que puede afectar el rendimiento del sitio web.

El siguiente componente de ejemplo `App`, ligeramente ajustado, ilustra el problema del restablecimiento del estado:

```javascript
import { useState } from 'react'; 
import { createBrowserRouter, RouterProvider } from 'react-router-dom'; 
import Dashboard from './routes/Dashboard.jsx'; 
import Orders from './routes/Orders.jsx'; 

const router = createBrowserRouter([ 
  { path: '/', element: <Dashboard /> }, 
  { path: '/orders', element: <Orders /> }, 
]); 

function App() { 
  const [counter, setCounter] = useState(0); 

  function handleIncCounter() { 
    setCounter((prevCounter) => prevCounter + 1); 
  } 

  return ( 
    <> 
      <p> 
        <button onClick={handleIncCounter}>Increase Counter</button> 
      </p> 
      <p>Current Counter: <strong>{counter}</strong></p> 
      <RouterProvider router={router} /> 
    </> 
  ); 
} 

export default App;
```

> [!NOTE]
> El código para este ejemplo se puede encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/03-naive-navigation-problem](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/03-naive-navigation-problem).

En este ejemplo, se agregó un contador simple al componente `App`. Dado que `<RouterProvider>` se renderiza en ese mismo componente, debajo del contador, el componente `App` no debería reemplazarse cuando un usuario visita una página diferente (en su lugar, es `<RouterProvider>` el que debería reemplazarse, no todo el código JSX del componente `App`).

Al menos, esa es la teoría. Pero, como puedes ver en la siguiente captura de pantalla, el estado del contador se pierde cada vez que se hace clic en algún enlace:

**Figura 13.4**: El estado del contador se reinicia al cambiar de página.

En la captura de pantalla, puedes ver que el contador se establece inicialmente en 3 (porque se hizo clic en el botón tres veces). Después de navegar de `Dashboard` a la página `Orders` (haciendo clic en el enlace de la página Orders), el contador cambia a 0.

Eso sucede porque la página se recarga debido a la solicitud HTTP que envía el navegador.

Para solucionar este problema y evitar esta recarga de página no deseada, debes evitar el comportamiento predeterminado del navegador. En lugar de enviar una nueva solicitud HTTP, la dirección URL del navegador simplemente debe actualizarse (de `localhost:5173` a `localhost:5173/orders`) y el componente de destino (`Orders`) debe cargarse. Por lo tanto, para el usuario del sitio web, parecería como si se hubiera cargado una página diferente. Pero detrás de escena, es solo el documento de la página (el DOM) el que se actualizó.

Afortunadamente, no tienes que implementar la lógica para esto por tu cuenta. En su lugar, la biblioteca React Router expone un componente especial **`Link`** que debe usarse en lugar del elemento de anclaje `<a>`.

Para usar este nuevo componente, el código en `src/routes/Dashboard.jsx` debe ajustarse de esta manera:

```javascript
import { Link } from 'react-router-dom'; 

function Dashboard() { 
  return ( 
    <> 
      <h1>The "Dashboard" route component</h1> 
      <p>Go to the <Link to="/orders">Orders page</Link>.</p> 
      <p> 
        This component could display the user dashboard of some web shop. 
      </p> 
      <p>It's just a dummy example here, but you get the point.</p> 
      <p> 
        It's worth noting, that it's a regular React component that's activated by React Router because of the active route configuration. 
      </p> 
    </> 
  ); 
} 

export default Dashboard;
```

> [!NOTE]
> El código para este ejemplo se puede encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/04-react-router-navigation](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/04-react-router-navigation).

Dentro de este ejemplo actualizado, se utiliza el nuevo componente `Link`. Ese componente requiere una prop **`to`**, que se utiliza para definir la ruta de la URL que se debe cargar.

Al usar este componente en lugar del elemento de anclaje `<a>`, el estado del contador ya no se reinicia. Esto se debe a que React Router ahora evita el comportamiento predeterminado del navegador (es decir, la recarga no deseada de la página descrita anteriormente) y muestra el contenido correcto de la página.

Bajo el capó, el componente `Link` todavía renderiza el elemento `<a>` integrado. Pero React Router lo controla e implementa el comportamiento descrito anteriormente.

El componente `Link` es, por lo tanto, el componente predeterminado que debe usarse para enlaces internos. Para enlaces externos, se debe utilizar el elemento estándar `<a>`, ya que el enlace apunta fuera del sitio web y, por lo tanto, no hay ningún estado que preservar ni recarga de página que evitar.

#### Trabajando con Layouts y rutas anidadas
La mayoría de los sitios web requieren alguna forma de navegación en todo el sitio (y por lo tanto enlaces de navegación) u otras secciones de página que deben compartirse en algunas o todas las rutas.

Considera el sitio web de ejemplo anterior con las rutas `/` y `/orders`. El sitio web de ejemplo también se beneficiaría de tener una barra de navegación superior que permita a los usuarios cambiar entre la página de inicio (es decir, la ruta `Dashboard`) y la página `Orders`.

Por lo tanto, `App.jsx` podría ajustarse para tener una barra de navegación superior dentro de un `<header>` encima de `<RouterProvider>`:

```javascript
import { createBrowserRouter, RouterProvider, Link } from 'react-router-dom'; 
import Dashboard from './routes/Dashboard.jsx'; 
import Orders from './routes/Orders.jsx'; 

const router = createBrowserRouter([ 
  { path: '/', element: <Dashboard /> }, 
  { path: '/orders', element: <Orders /> }, 
]); 

function App() { 
  return ( 
    <> 
      <header> 
        <nav> 
          <ul> 
            <li> 
              <Link to="/">My Dashboard</Link> 
            </li> 
            <li> 
              <Link to="/orders">Past Orders</Link> 
            </li> 
          </ul> 
        </nav> 
      </header> 
      <RouterProvider router={router} /> 
    </> 
  ); 
} 

export default App;
```

Pero si intentas ejecutar esta aplicación, verás una página en blanco y encontrarás un mensaje de error en la consola de JavaScript en las herramientas de desarrollo del navegador.

**Figura 13.5**: React Router parece quejarse de algo.

El mensaje de error es un poco críptico, pero el problema es que el código anterior **intenta usar `<Link>` fuera de un componente controlado por React Router**.

Solo los componentes cargados a través de `<RouterProvider>` están controlados por React Router; por lo tanto, las características de React Router, como su componente `Link`, solo se pueden usar en componentes de ruta (o sus componentes descendientes).

Por lo tanto, configurar la navegación principal dentro del componente `App` (que no es cargado por React Router) no funciona.

Para envolver o mejorar múltiples componentes de ruta con algún componente compartido y marcado JSX, debes definir una nueva ruta que envuelva las rutas existentes. Dicha ruta también se denomina a veces **ruta de diseño (*layout route*)**, ya que se puede utilizar para proporcionar un diseño compartido. Las rutas envueltas por esta ruta se denominarían **rutas anidadas (*nested routes*)**.

Una ruta de diseño se define como cualquier otra ruta dentro del array de definiciones de rutas. Luego se convierte en una ruta de diseño envolviendo otras rutas a través de una propiedad especial **`children`** que acepta React Router. Esa propiedad `children` recibe un array de rutas anidadas: rutas secundarias de la ruta principal envolvente.

Aquí está el código de definición de ruta ajustado para esta aplicación de ejemplo:

```javascript
import Root from './routes/Root.jsx'; 
import Dashboard from './routes/Dashboard.jsx'; 
import Orders from './routes/Orders.jsx'; 

const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, 
    children: [ 
      { index: true, element: <Dashboard /> }, 
      { path: '/orders', element: <Orders /> }, 
    ], 
  }, 
]);
```

En este fragmento de código actualizado, se define una nueva ruta de diseño raíz (`Root`), una ruta que registra las rutas existentes (los componentes `Dashboard` y `Orders`) como rutas secundarias. Por lo tanto, esta configuración permite que el componente `Root` esté activo simultáneamente con el componente de ruta `Dashboard` u `Orders`.

También puedes notar que la ruta `Dashboard` ya no tiene una propiedad `path`. En su lugar, ahora tiene una propiedad **`index`**, que se establece en `true`. Esa propiedad `index` es una propiedad que se puede usar cuando se trabaja con rutas anidadas. Le dice a React Router qué ruta anidada activar (y por lo tanto qué componente cargar) si la ruta de la ruta principal coincide exactamente.

En este ejemplo, cuando la ruta `/` está activa (es decir, si un usuario visita `<dominio>/`), se renderizarán los componentes `Root` y `Dashboard`. Para `<dominio>/orders`, `Root` y `Orders` se volverán visibles.

El componente `Root` es un componente recién agregado en este ejemplo. Es un componente estándar (como `Dashboard` u `Orders`) con una característica especial: **define el lugar donde se deben insertar los componentes de la ruta secundaria a través de un componente especial `Outlet`** que proporciona React Router:

```javascript
import { Link, Outlet } from 'react-router-dom'; 

function Root() { 
  return ( 
    <> 
      <header> 
        <nav> 
          <ul> 
            <li> 
              <Link to="/">My Dashboard</Link> 
            </li> 
            <li> 
              <Link to="/orders">Past Orders</Link> 
            </li> 
          </ul> 
        </nav> 
      </header> 
      <Outlet /> 
    </> 
  ); 
} 

export default Root;
```

El marcador de posición `<Outlet />` es necesario ya que React Router debe saber dónde renderizar los componentes de ruta de las rutas pasadas a la propiedad `children`.

> [!NOTE]
> Puedes encontrar el código de ejemplo completo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/05-layouts-nested-routes](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/05-layouts-nested-routes).

Dado que el componente `Root` en sí también es renderizado por React Router, ahora es un componente que tiene acceso a la etiqueta `<Link>`. Por lo tanto, este componente `Root` se puede utilizar para compartir marcado común (como el `<header>` de navegación) en todas las rutas anidadas.

**Figura 13.6**: Se muestra una barra de navegación compartida en la parte superior (para todas las rutas).

Por lo tanto, las rutas anidadas y las rutas de diseño (o rutas envolventes) son características cruciales que ofrece React Router.

También vale la pena señalar que puedes agregar tantos niveles de anidamiento de rutas como requiera tu aplicación; no estás restringido a tener solo una ruta de diseño que envuelva rutas secundarias.

#### De Link a NavLink
En una navegación compartida, a menudo deseas resaltar el enlace que condujo a la página actualmente activa. Por ejemplo, si un usuario hizo clic en el enlace `Past Orders` (y por lo tanto navega a `/orders`), ese enlace debería cambiar su apariencia (por ejemplo, su color).

Considera el ejemplo anterior (Figura 13.6): allí, en la barra de navegación superior, no es inmediatamente obvio si el usuario está en la página `Dashboard` o en la página `Orders`. Por supuesto, la dirección URL y el contenido principal de la página cambian, pero los elementos de navegación no se ajustan visualmente.

Para comprobar este punto, compara la captura de pantalla anterior con la siguiente:

**Figura 13.7**: El enlace de navegación resaltado "Past Orders" está subrayado y cambia de color.

En esta versión del sitio web, queda claro de inmediato que el usuario se encuentra en la página "Orders" ya que el enlace de navegación `Past Orders` está resaltado. Son cosas sutiles como esta las que hacen que los sitios web sean más intuitivos y, en última instancia, pueden generar una mayor participación de los usuarios.

¿Pero cómo se puede lograr esto?

Para hacer esto, no usarías el componente `Link`, sino un componente alternativo especial que ofrece `react-router-dom`: el componente **`NavLink`**:

```javascript
import { NavLink, Outlet } from 'react-router-dom'; 

function Root() { 
  return ( 
    <> 
      <header> 
        <nav> 
          <ul> 
            <li> 
              <NavLink to="/">My Dashboard</NavLink> 
            </li> 
            <li> 
              <NavLink to="/orders">Past Orders</NavLink> 
            </li> 
          </ul> 
        </nav> 
      </header> 
      <Outlet /> 
    </> 
  ); 
} 

export default Root;
```

El componente `NavLink` se utiliza casi exactamente igual que el componente `Link`. Lo envuelves alrededor de algún texto (el título del enlace) y defines la ruta de destino a través de la prop `to`. Sin embargo, el componente `NavLink` tiene algunas características adicionales relacionadas con el estilo que el componente `Link` normal no tiene.

Para ser precisos, el componente `NavLink` **aplica de forma predeterminada una clase CSS llamada `active` al elemento de anclaje renderizado cuando el enlace está activo**.

**Figura 13.8**: El elemento `<a>` renderizado recibió una clase CSS "active".

En caso de que desees aplicar un nombre de clase CSS diferente o estilos en línea cuando un enlace se activa, `NavLink` también te permite hacerlo. Esto se debe a que las props `className` y `style` de `NavLink` se comportan de manera ligeramente diferente a como lo hacen en otros elementos: además de aceptar valores de cadena (`className`) u objetos de estilo (`style`), **ambas props también aceptan funciones** que React Router llamará automáticamente ante cada acción de navegación. Por ejemplo, se podría usar el siguiente código para garantizar que se aplique una determinada clase o estilo CSS:

```javascript
<NavLink 
  className={({ isActive }) => isActive ? 'loaded' : ''} 
  style={({ isActive }) => isActive ? { color: 'red' } : undefined}
> 
  Some Link 
</NavLink>
```

En el fragmento de código anterior, tanto `className` como `style` aprovechan la función que ejecutará React Router. Esta función recibe automáticamente un objeto como argumento de entrada, un objeto creado y proporcionado por React Router que contiene una propiedad `isActive`. React Router establece `isActive` en `true` si el enlace conduce a la ruta actualmente activa, y en `false` en caso contrario.

Por lo tanto, puedes devolver cualquier nombre de clase CSS u objeto de estilo que elijas en esas funciones. React Router los aplicará luego al elemento `<a>` renderizado.

> [!NOTE]
> Puedes encontrar el código final de este ejemplo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/06-navlinks](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/06-navlinks).

Una nota importante es que `NavLink` considerará que una ruta está activa si su ruta coincide con la ruta de la URL actual o si su ruta comienza con la ruta de la URL actual. Por ejemplo, si tuvieras una ruta `/blog/all-posts`, un componente `NavLink` que apunte solo a `/blog` se consideraría activo si la ruta actual es `/blog/all-posts` (porque la ruta de esa ruta comienza con `/blog`). Si no deseas este comportamiento, puedes agregar la prop especial **`end`** al componente `NavLink`, de la siguiente manera:

```javascript
<NavLink 
  to="/blog" 
  style={({ isActive }) => isActive ? { color: 'red' } : undefined} 
  end
> 
  Blog 
</NavLink>
```

Con esta prop especial agregada, este `NavLink` solo se consideraría activo si la ruta actual es exactamente `/blog`; para `/blog/all-posts`, el enlace no estaría activo.

Una excepción a esa regla son los enlaces a solo `/`. Dado que todas las rutas técnicamente comienzan con esta "ruta vacía", React Router de forma predeterminada solo considera `<NavLink to="/">` como activo si el usuario se encuentra actualmente en `<dominio>/`. Para otras rutas (por ejemplo, `/orders`), `<NavLink to="/">` no se marcaría como activo.

`NavLink` es siempre la opción preferida cuando el estilo de un enlace depende de la ruta actualmente activa. Para todos los demás enlaces internos, usa `Link`. Para enlaces externos, `<a>` es el elemento de elección.

#### Componentes de ruta frente a componentes "normales"
Vale la pena mencionar y señalar que, en los ejemplos anteriores, los componentes `Dashboard` y `Orders` eran componentes normales de React. Podrías usar estos componentes en cualquier lugar de tu aplicación de React, no solo como valores para la propiedad `element` de una definición de ruta.

Sin embargo, los dos componentes son especiales porque ambos están almacenados en la carpeta `src/routes` en el directorio del proyecto. No están almacenados en la carpeta `src/components`, que se utilizó para componentes a lo largo de este libro.

Sin embargo, eso no es algo obligatorio. De hecho, los nombres de las carpetas dependen totalmente de ti. Estos dos componentes podrían almacenarse en `src/components`. También podrías almacenarlos en una carpeta `src/elements`. Pero usar `src/routes` es bastante común para componentes que se usan exclusivamente para enrutamiento. Las alternativas populares son `src/screens`, `src/views` y `src/pages`.

Si tu aplicación incluye otros componentes que no se utilizan como elementos de enrutamiento, aún los almacenarías en `src/components` (es decir, en una ruta diferente). Esto no es una regla estricta ni un requisito técnico, pero ayuda a mantener organizados tus proyectos de React. Dividir tus componentes en múltiples carpetas hace que sea más fácil comprender rápidamente qué componentes cumplen qué propósitos en el proyecto.

En el proyecto de ejemplo mencionado anteriormente, puedes, por ejemplo, refactorizar el código de modo que el código de navegación se almacene en un componente separado (por ejemplo, un componente `MainNavigation`, almacenado en `src/components/shared/MainNavigation.jsx`). El código del archivo del componente se ve así:

```javascript
import { NavLink } from 'react-router-dom'; 
import classes from './MainNavigation.module.css'; 

function MainNavigation() { 
  return ( 
    <header className={classes.header}> 
      <nav> 
        <ul> 
          <li> 
            <NavLink 
              to="/" 
              className={({ isActive }) => 
                isActive ? classes.active : undefined 
              } 
              end 
            > 
              My Dashboard 
            </NavLink> 
          </li> 
          <li> 
            <NavLink 
              to="/orders" 
              className={({ isActive }) => 
                isActive ? classes.active : undefined 
              } 
            > 
              Past Orders 
            </NavLink> 
          </li> 
        </ul> 
      </nav> 
    </header> 
  ); 
} 

export default MainNavigation;
```

En este fragmento de código, el componente `NavLink` se ajusta para asignar una clase CSS llamada `active` a cualquier enlace que pertenezca a la ruta actualmente activa. Esto es necesario cuando se usan CSS Modules ya que los nombres de clase cambian durante el proceso de compilación, como se discutió en el Capítulo 6, *Estilos en Aplicaciones React*. Además de eso, es esencialmente el mismo código de menú de navegación que el utilizado anteriormente en este capítulo.

Este componente `MainNavigation` luego se puede importar y usar en el archivo `Root.jsx` de esta manera:

```javascript
import { Outlet } from 'react-router-dom'; 
import MainNavigation from '../components/shared/MainNavigation.jsx'; 

function Root() { 
  return ( 
    <> 
      <MainNavigation /> 
      <Outlet /> 
    </> 
  ); 
} 

export default Root;
```

Importar y usar el componente `MainNavigation` da como resultado un componente `Root` más limpio y, al mismo tiempo, conserva la misma funcionalidad que antes.

Estos cambios muestran cómo puedes combinar componentes de enrutamiento que solo se usan para enrutamiento (`Dashboard` y `Orders`) y componentes que se usan fuera del enrutamiento (`MainNavigation`).

> [!NOTE]
> Puedes encontrar el código final de este ejemplo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/07-routing-and-normal-cmp](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/07-routing-and-normal-cmp).

Pero incluso con esas mejoras de marcado y estilo, la aplicación de demostración todavía sufre un problema importante: **solo admite rutas estáticas y predefinidas**. Pero, para la mayoría de los sitios web, ese tipo de rutas no son suficientes.

---

### Sección 4: De rutas estáticas a rutas dinámicas

Hasta ahora, todos los ejemplos han tenido dos rutas: `/` para el componente `Dashboard` y `/orders` para el componente `Orders`. Pero, por supuesto, puedes agregar tantas rutas como sea necesario. Si tu sitio web consta de 20 páginas diferentes, puedes (y debes) agregar 20 definiciones de rutas a tu componente `App`.

En la mayoría de los sitios web, sin embargo, también tendrás algunas rutas que no se pueden definir manualmente, porque no todas las rutas y sus rutas de acceso exactas se conocen de antemano.

Considera el ejemplo anterior, enriquecido con componentes adicionales y algunos datos ficticios:

**Figura 13.9**: Una lista de elementos de pedidos.

> [!NOTE]
> Puedes encontrar el código de este ejemplo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/08-dynamic-routes-problem](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/08-dynamic-routes-problem). En el código, notarás que se agregaron muchos componentes nuevos y archivos de estilo. Sin embargo, el código no utiliza ninguna característica nueva: simplemente se utiliza para mostrar una interfaz de usuario más realista y generar algunos datos ficticios.

En la captura de pantalla anterior, Figura 13.9, puedes ver una lista de pedidos generada en la página `Past Orders` (es decir, por el componente `Orders`).

En el código subyacente, cada elemento de pedido se envuelve con un componente `Link` para que se pueda cargar una página separada con más detalles para cada elemento:

```javascript
function OrdersList() { 
  return ( 
    <ul className={classes.list}> 
      {orders.map((order) => ( 
        <li key={order.id}> 
          <Link to='/orders'><OrderItem order={order} /></Link> 
        </li> 
      ))} 
    </ul> 
  ); 
}
```

En este fragmento de código, la ruta para el componente `Link` se establece en `/orders`. Sin embargo, ese no es el valor final que debería asignarse. En su lugar, este ejemplo destaca un problema importante: si bien es la misma ruta y componente que debe cargarse para cada elemento de pedido (es decir, algún componente que muestre datos detallados sobre el pedido seleccionado), el contenido exacto que genera ese componente depende del elemento de pedido que se seleccionó. Es la misma ruta y componente con datos diferentes.

Fuera del enrutamiento, usarías props para reutilizar el mismo componente con diferentes datos. Pero con el enrutamiento, no se trata solo del componente: también debes admitir diferentes rutas, porque los datos detallados para diferentes pedidos deben cargarse a través de diferentes rutas (por ejemplo, `/orders/o1`, `/orders/o2`, etc.). De lo contrario, volverías a tener URLs que no se pueden compartir ni recargar.

Por lo tanto, la ruta debe incluir no solo algún identificador estático (como `/orders`), sino también un **valor dinámico que sea diferente para cada elemento de pedido**. Para tres elementos de pedidos con valores de `id` `o1`, `o2` y `o3`, el objetivo podría ser admitir las rutas `/orders/o1`, `/orders/o2` y `/orders/o3`.

Por esta razón, se podrían agregar las siguientes tres definiciones de rutas:

```javascript
{ path: '/orders/o1', element: <OrderDetail id="o1" /> }, 
{ path: '/orders/o2', element: <OrderDetail id="o2" /> }, 
{ path: '/orders/o3', element: <OrderDetail id="o3" /> }
```

Pero esta solución tiene un defecto importante: agregar todas estas rutas manualmente requiere una enorme cantidad de trabajo. Y ese ni siquiera es el mayor problema: normalmente ni siquiera conoces todos los valores de antemano. En este ejemplo, cuando se realiza un nuevo pedido, se tendría que agregar una nueva ruta. Pero no puedes ajustar el código fuente de tu sitio web cada vez que un visitante realiza un pedido.

Claramente, se necesita una solución mejor. React Router ofrece esa solución mejorada ya que admite **rutas dinámicas**.

Las rutas dinámicas se definen como otras rutas, excepto que, al definir sus valores de `path`, deberás incluir **uno o más segmentos de ruta dinámicos con identificadores de tu elección**.

La definición de la ruta `OrderDetail` se ve así:

```javascript
{ path: '/orders/:id', element: <OrderDetail /> }
```

Han cambiado tres cosas clave:
1. Es solo una definición de ruta en lugar de una lista (posiblemente) infinita de definiciones.
2. `path` contiene un **segmento de ruta dinámico (`:id`)**.
3. `OrderDetail` ya no recibe una prop `id`.

La sintaxis `:id` es una sintaxis especial admitida por React Router. Cada vez que un segmento de una ruta comienza con dos puntos, React Router lo trata como un segmento dinámico. Eso significa que se reemplazará con un valor diferente en la ruta de la URL real. Para la ruta `/orders/:id`, las rutas `/orders/o1`, `/orders/o2` y `/orders/abc` coincidirían y, por lo tanto, activarían la ruta.

Por supuesto, no tienes que usar `:id`. Puedes usar cualquier identificador de tu elección. Para el ejemplo anterior, `:orderId`, `:order` u `:oid` también tendrían sentido.

El identificador ayudará a tu aplicación a acceder a los datos correctos dentro del componente de página que debe cargarse para la ruta dinámica (es decir, el componente de ruta `OrderDetail` en los fragmentos de código de ejemplo anteriores). Es por eso que se eliminó la prop `id` de `OrderDetail` en el último fragmento de código. Dado que solo se define una ruta, solo se podría pasar un valor de `id` específico a través de props, lo cual no resolvería el problema. Por lo tanto, se debe utilizar una forma diferente de cargar datos específicos del pedido.

#### Extracción de parámetros de ruta
En el ejemplo anterior, cuando un usuario del sitio web visita `/orders/o1` o `/orders/o2` (o la misma ruta para cualquier otro ID de pedido), se carga el componente `OrderDetail`. Este componente luego debería mostrar más información sobre el pedido específico que se seleccionó (es decir, el pedido cuyo ID está codificado en la ruta de la URL).

Por cierto, ese no es solo el caso de este ejemplo; puedes pensar en muchos otros tipos de sitios web también. También podrías tener, por ejemplo, una tienda en línea con rutas para productos (`/products/p1`, `/products/p2`, etc.), o un blog de viajes donde los usuarios puedan visitar publicaciones de blog individuales (`/blog/post1`, `/blog/post2`, etc.).

En todos estos casos, la pregunta es: ¿cómo se accede a los datos que se deben cargar para el identificador específico (por ejemplo, el ID) que se incluye en la ruta de la URL? Dado que siempre se carga el mismo componente, necesitas una forma de identificar dinámicamente el pedido, el producto o la publicación del blog cuyos datos detallados se deben obtener.

Una posible solución sería el uso de props. Cada vez que creas un componente que debe ser reutilizable pero configurable y dinámico, puedes usar props para aceptar diferentes valores. Por ejemplo, el componente `OrderDetail` podría aceptar una prop `id` y luego, dentro del cuerpo de la función del componente, cargar los datos para ese ID de pedido específico.

Sin embargo, como se mencionó en la sección anterior, esta no es una solución viable cuando se carga el componente mediante enrutamiento. Ten en cuenta que el componente `OrderDetail` se crea al definir la ruta:

```javascript
{ path: '/orders/:id', element: <OrderDetail /> }
```

Dado que el componente se crea al definir la ruta en el componente `App`, no puedes pasar ningún valor de prop dinámico específico del ID.

Afortunadamente, eso no es necesario. React Router te ofrece una solución que te permite extraer los datos codificados en la ruta de la URL desde dentro del componente que se muestra en la pantalla (cuando la ruta se activa): el Hook **`useParams()`**.

Este Hook se puede utilizar para obtener acceso a los parámetros de ruta de la ruta actualmente activa. Los **parámetros de ruta (*route parameters*)** son simplemente los valores dinámicos codificados en la ruta de la URL (`id`, en el caso de este ejemplo de `OrderDetail`).

Dentro del componente `OrderDetail`, `useParams()` se puede utilizar para extraer el ID de pedido específico y cargar los datos de pedido adecuados, de la siguiente manera:

```javascript
import { useParams } from 'react-router-dom'; 
import Details from '../components/orders/Details.jsx'; 
import { getOrderById } from '../data/orders.js'; 

function OrderDetail() { 
  const params = useParams(); 
  const orderId = params.id; // orderId is "o1", "o2" etc. 
  const order = getOrderById(orderId); 
  return <Details order={order} />; 
} 

export default OrderDetail;
```

Como puedes ver en este fragmento, `useParams()` devuelve un objeto que contiene todos los parámetros de ruta de la ruta actualmente activa como propiedades. Dado que la ruta de acceso se definió como `/orders/:id`, el objeto `params` contiene una propiedad `id`. El valor de esa propiedad es entonces el valor real codificado en la ruta de la URL (por ejemplo, `o1`). Si eliges un nombre de identificador diferente en la definición de la ruta (por ejemplo, `/orders/:orderId` en lugar de `/orders/:id`), ese nombre de propiedad debe usarse para acceder al valor en el objeto `params` (es decir, acceder a `params.orderId`).

> [!NOTE]
> Puedes encontrar el código completo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/09-dynamic-routes](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/09-dynamic-routes).

Al usar parámetros de ruta, puedes crear fácilmente rutas dinámicas que conducen a la carga de diferentes datos. Pero, por supuesto, definir rutas y manejar la activación de rutas no es tan útil si no tienes enlaces que conduzcan a rutas dinámicas.

#### Creación de enlaces dinámicos
Como se mencionó anteriormente en este capítulo (en la sección *Añadir navegación de página*), los visitantes del sitio web deberían poder hacer clic en enlaces que luego los lleven a las diferentes páginas que componen el sitio web general; es decir, esos enlaces deberían activar las diversas rutas definidas con la ayuda de React Router.

Como se explicó en las secciones *Añadir navegación de página* y *De Link a NavLink*, para enlaces internos (es decir, enlaces que conducen a rutas definidas dentro de la aplicación de React), se utilizan los componentes `Link` o `NavLink`.

Entonces, para rutas estáticas como `/orders`, los enlaces se crean así:

```javascript
<Link to="/orders">Past Orders</Link> // or use <NavLink> instead
```

Al crear un enlace a una ruta dinámica como `/orders/:id`, simplemente puedes crear un enlace como este:

```javascript
<Link to="/orders/o1">Past Orders</Link>
```

Este enlace específico carga el componente `OrderDetail` para el pedido con el ID `o1`.

Crear el enlace de la siguiente manera sería incorrecto:

```javascript
<Link to="/orders/:id">Past Orders</Link>
```

La sintaxis del segmento de ruta dinámico (`:id`) solo se utiliza al definir la ruta, no al crear un enlace. El enlace tiene que apuntar a un recurso específico (un pedido específico, en este caso).

Sin embargo, crear enlaces a pedidos específicos como se mostró anteriormente no es muy práctico. Así como no tendría sentido definir todas las rutas dinámicas individualmente (consulta la sección *De rutas estáticas a rutas dinámicas*), no tiene sentido crear los enlaces respectivos manualmente.

Siguiendo con el ejemplo de los pedidos, tampoco hay necesidad de crear enlaces así, ya que ya tienes una lista de pedidos que se muestra en una página (el componente `Orders`, en este caso). Del mismo modo, podrías tener una lista de productos en una tienda online. En todos estos casos, los elementos individuales (pedidos, productos, etc.) deben ser clickeables y conducir a páginas de detalles con más información.

**Figura 13.10**: Una lista de elementos de pedidos clickeables.

Por lo tanto, los enlaces se pueden generar dinámicamente al renderizar la lista de elementos JSX. En el caso del ejemplo de pedidos, el código se ve así:

```javascript
function OrdersList() { 
  return ( 
    <ul className={classes.list}> 
      {orders.map((order) => ( 
        <li key={order.id}> 
          <Link to={`/orders/${order.id}`}> 
            <OrderItem order={order} /> 
          </Link> 
        </li> 
      ))} 
    </ul> 
  ); 
}
```

En este ejemplo de código, el valor de la prop `to` se establece dinámicamente igual a una cadena que incluye el valor `order.id`. Por lo tanto, cada elemento de la lista recibe un enlace único que conduce a una página de detalles diferente. O, para ser precisos, el enlace siempre conduce al mismo componente pero con un valor de ID de pedido diferente, cargando así diferentes datos de pedido.

> [!NOTE]
> En este fragmento de código (que se puede encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/10-dynamic-links](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/10-dynamic-links)), la cadena se crea como una plantilla literal (*template literal*). Esa es una característica estándar de JavaScript que simplifica la creación de cadenas que incluyen valores dinámicos.
> Puedes obtener más información sobre las plantillas literales en MDN en [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

#### Navegación programática
En la sección anterior, así como anteriormente en este capítulo, la navegación del usuario se habilitó agregando enlaces al sitio web. De hecho, los enlaces son la forma predeterminada de agregar navegación a un sitio web. Pero hay escenarios donde se requiere navegación programática en su lugar.

**Navegación programática (*programmatic navigation*)** significa que se debe cargar una nueva página a través de código JavaScript (en lugar de usar un enlace). Este tipo de navegación generalmente se requiere si la página activa cambia en respuesta a alguna acción, por ejemplo, tras el envío de un formulario.

Si tomas el ejemplo del envío de un formulario, normalmente querrás extraer y guardar los datos enviados. Pero a partir de entonces, a veces será necesario redirigir al usuario a una página diferente. Por ejemplo, no tiene sentido mantener al usuario en una página de Pago después de procesar los datos de la tarjeta de crédito ingresados: es posible que desees redirigir al usuario a una página de Éxito en su lugar.

En el ejemplo analizado a lo largo de este capítulo, la página `Past Orders` podría incluir un campo de entrada que permita a los usuarios ingresar directamente un ID de pedido y cargar los datos del pedido respectivo después de hacer clic en el botón `Find`.

**Figura 13.11**: Un campo de entrada que se puede utilizar para cargar rápidamente un pedido específico.

En este ejemplo, el ID de pedido ingresado se procesa y valida primero antes de enviar al usuario a la página de detalles respectiva. Si el ID proporcionado no es válido, se muestra un mensaje de error. El código se ve así:

```javascript
import orders, { getOrdersSummaryData } from '../../data/orders.js'; 
import classes from './OrdersSummary.module.css'; 

function OrdersSummary() { 
  const { quantity, total } = getOrdersSummaryData(); 
  const formattedTotal = new Intl.NumberFormat('en-US', { 
    style: 'currency', 
    currency: 'USD', 
  }).format(total); 

  function findOrderAction(formData) { 
    const orderId = formData.get('order-id'); 
    const orderExists = orders.some((order) => order.id === orderId); 
    if (!orderExists) { 
      alert('Could not find an order for the entered id.'); 
      return; 
    } 
  } 

  return ( 
    <div className={classes.row}> 
      <p className={classes.summary}> 
        {formattedTotal} | {orders.length} Orders | {quantity} Products 
      </p> 
      <form className={classes.form} action={findOrderAction}> 
        <input 
          type="text" 
          name="order-id" 
          placeholder="Enter order id" 
          aria-label="Find an order by id." 
        /> 
        <button>Find</button> 
      </form> 
    </div> 
  ); 
} 

export default OrdersSummary;
```

El fragmento de código aún no incluye el código que realmente desencadenará el cambio de página, pero muestra cómo se leen y validan las entradas del usuario.

Por lo tanto, este es un escenario perfecto para el uso de la navegación programática. No se puede utilizar un enlace aquí ya que desencadenaría inmediatamente un cambio de página, sin permitirte validar primero la entrada del usuario (al menos no después de hacer clic en el enlace).

La biblioteca React Router también admite la navegación programática para casos como este. Puedes importar y utilizar el Hook especial **`useNavigate()`** para obtener acceso a una función de navegación que se puede utilizar para activar una acción de navegación (es decir, un cambio de página):

```javascript
import { useNavigate } from 'react-router-dom'; 

const navigate = useNavigate(); 
navigate('/orders'); // programmatic alternative to <Link to="/orders">
```

Por lo tanto, el componente `OrdersSummary` anterior se puede ajustar de la siguiente manera para usar este nuevo Hook:

```javascript
function OrdersSummary() { 
  const navigate = useNavigate(); 
  const { quantity, total } = getOrdersSummaryData(); 
  const formattedTotal = new Intl.NumberFormat('en-US', { 
    style: 'currency', 
    currency: 'USD', 
  }).format(total); 

  function findOrderAction(formData) { 
    const orderId = formData.get('order-id'); 
    const orderExists = orders.some((order) => order.id === orderId); 
    if (!orderExists) { 
      alert('Could not find an order for the entered id.'); 
      return; 
    } 
    navigate(`/orders/${orderId}`); 
  } 

  // returned JSX code did not change, hence omitted 
}
```

Vale la pena señalar que el valor pasado a `navigate()` es una cadena construida dinámicamente. La navegación programática admite tanto rutas estáticas como dinámicas.

> [!NOTE]
> El código para este ejemplo se puede encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/11-programmatic-navigation](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/11-programmatic-navigation).

---

### Sección 5: Redirecciones

Hasta ahora, todas las opciones de navegación exploradas (enlaces y navegación programática) redirigen hacia adelante (*forward*) a un usuario a una página específica.

En la mayoría de los casos, ese es el comportamiento previsto. Pero en algunos casos, el objetivo es **redirigir (*redirect*) a un usuario en lugar de reenviarlo**.

La diferencia es sutil pero importante:
- Cuando un usuario es **reenviado hacia adelante**, puede usar los botones de navegación del navegador (Atrás y Adelante) para volver a la página anterior o avanzar a la página de la que vino.
- Para las **redirecciones**, eso no es posible. Siempre que un usuario es redirigido a una página específica (en lugar de reenviado), no puede usar el botón Atrás para regresar a la página anterior.

Redirigir a los usuarios puede, por ejemplo, ser útil para garantizar que los usuarios no puedan regresar a una página de inicio de sesión después de autenticarse con éxito.

Al utilizar React Router, el comportamiento predeterminado es reenviar a los usuarios. Pero puedes cambiar fácilmente a la redirección agregando la prop especial **`replace`** a los componentes `Link` (o `NavLink`), de la siguiente manera:

```javascript
<Link to="/success" replace>Confirm Checkout</Link>
```

Al utilizar la navegación programática, puedes pasar un segundo argumento opcional a la función `navigate()`. El valor de ese segundo parámetro debe ser un objeto que puede contener una propiedad `replace` que debe establecerse en `true` si deseas redirigir a los usuarios:

```javascript
navigate('/dashboard', { replace: true });
```

Poder redirigir o reenviar a los usuarios te permite crear aplicaciones web altamente intuitivas que ofrecen la mejor experiencia de usuario posible para diferentes escenarios.

#### Manejo de rutas no definidas
Las secciones anteriores de este capítulo han asumido que tienes rutas predefinidas a las que deben poder acceder los visitantes del sitio web. ¿Pero qué pasa si un visitante introduce una URL que simplemente no es compatible?

Por ejemplo, el sitio web de demostración utilizado a lo largo de este capítulo admite las rutas `/`, `/orders` y `/orders/<algun-id>`. Pero no admite `/home`, `/products/p1`, `/abc` ni ninguna otra ruta que no sea una de las rutas definidas.

Para mostrar una página personalizada de No encontrado (*Not Found* / 404), puedes definir una **ruta comodín (*catch all*)** con una ruta especial: la ruta **`*`**:

```javascript
{ path: '*', element: <NotFound /> }
```

Al agregar esta ruta al final de la lista de definiciones de rutas en el componente `App`, el componente `NotFound` se mostrará en la pantalla cuando ninguna otra ruta coincida con la ruta de URL ingresada o generada.

#### Carga perezosa (*Lazy Loading*)
En el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, aprendiste sobre la carga perezosa (*lazy loading*), una técnica que se puede utilizar para cargar ciertas partes del código de la aplicación de React solo cuando sea necesario.

La división de código (*code-splitting*) tiene mucho sentido si algunos componentes se cargarán condicionalmente y es posible que no se necesiten en absoluto. Por lo tanto, el enrutamiento es un escenario perfecto para la carga perezosa. Cuando las aplicaciones tienen múltiples rutas, es posible que un usuario nunca visite algunas rutas. Incluso si se visitan todas las rutas, no es necesario descargar todo el código de todas las rutas de la aplicación (es decir, de sus componentes) al inicio cuando se carga la aplicación. En su lugar, tiene sentido descargar únicamente el código de las rutas individuales cuando realmente se activan.

Afortunadamente, React Router tiene **soporte integrado para carga perezosa y división de código basada en rutas**. Proporciona una propiedad **`lazy`** que se puede agregar a una definición de ruta. Esa propiedad espera una función que importe dinámicamente el archivo cargado de forma perezosa (que contiene el componente que debe renderizarse). React Router luego se encarga del resto; por ejemplo, **no necesitas envolver `Suspense` alrededor de ningún componente**:

```javascript
import { createBrowserRouter, RouterProvider } from 'react-router-dom'; 
import Root from './routes/Root.jsx'; 
import Dashboard from './routes/Dashboard.jsx'; 
// Removed static imports of Orders.jsx and OrderDetail.jsx 

const router = createBrowserRouter([ 
  { 
    path: '/', 
    element: <Root />, 
    children: [ 
      { index: true, element: <Dashboard /> }, 
      { path: '/orders', lazy: () => import('./routes/Orders.jsx') }, 
      { path: '/orders/:id', lazy: () => import('./routes/OrderDetail.jsx') }, 
    ], 
  }, 
]); 

function App() { 
  return <RouterProvider router={router} />; 
} 

export default App;
```

En este ejemplo, tanto la ruta `/orders` como la ruta `/orders/:id` están configuradas para cargar sus respectivos componentes de forma perezosa.

Para que el código anterior funcione, hay un ajuste importante que debes aplicar a los archivos de componentes de ruta al utilizar este soporte integrado de carga perezosa: **debes reemplazar la exportación de función de componente predeterminada (`export default SomeComponent`) con una exportación nombrada donde la función del componente se llame `Component`**.

Por ejemplo, el código del componente `Orders` debe modificarse para que se vea así:

```javascript
import OrdersList from '../components/orders/OrdersList.jsx'; 
import OrdersSummary from '../components/orders/OrdersSummary.jsx'; 

function Orders() { 
  return ( 
    <> 
      <OrdersSummary /> 
      <OrdersList /> 
    </> 
  ); 
} 

export const Component = Orders; // named export as "Component"
```

En este fragmento de código, la función del componente `Orders` se exporta como `Component`. Este nombre es obligatorio ya que React Router busca una función de componente llamada `Component` al activar una ruta cargada de forma perezosa.

> [!NOTE]
> El código para este ejemplo se puede encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/12-lazy-loading](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/examples/12-lazy-loading).

Como se explicó en el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*, agregar carga perezosa puede mejorar considerablemente el rendimiento de tu aplicación de React. Siempre debes considerar el uso de la carga perezosa, pero no debes usarla para todas las rutas. Sería especialmente ilógico para las rutas que se garantiza que se cargarán temprano, por ejemplo. En el ejemplo anterior, no tendría mucho sentido cargar de forma perezosa el componente `Dashboard` ya que esa es la ruta predeterminada (con una ruta de `/`).

Pero las rutas que no se garantiza que se visiten (o al menos no inmediatamente después de que se carga el sitio web) son excelentes candidatas para la carga perezosa.

---

### Sección 6: Resumen y puntos clave

- El enrutamiento (*routing*) es una característica clave para muchas aplicaciones de React.
- Con el enrutamiento, los usuarios pueden visitar múltiples páginas a pesar de estar en una SPA.
- El paquete más común que ayuda con el enrutamiento es la biblioteca **React Router** (`react-router-dom`).
- Las rutas se definen con la ayuda de la función `createBrowserRouter()` y el componente `RouterProvider` (normalmente en el componente `App` o en el archivo `main.jsx`, pero puedes hacerlo en cualquier lugar).
- Los objetos de definición de ruta generalmente se configuran con una propiedad `path` (para la cual la ruta debe activarse) y `element` (el contenido que debe mostrarse).
- El contenido y el marcado se pueden compartir entre múltiples rutas mediante la configuración de **rutas de diseño (*layout routes*)**, es decir, rutas que envuelven a otras rutas anidadas.
- Los usuarios pueden navegar entre rutas cambiando manualmente la ruta de la URL, haciendo clic en enlaces o mediante la navegación programática.
- Los enlaces internos (es decir, los enlaces que conducen a rutas de aplicaciones definidas por ti) deben crearse a través de los componentes `Link` o `NavLink`, mientras que los enlaces a recursos externos utilizan el elemento estándar `<a>`.
- La navegación programática se activa a través de la función `navigate()`, que produce el Hook `useNavigate()`.
- Puedes definir rutas estáticas y dinámicas: las rutas estáticas son las predeterminadas, mientras que las **rutas dinámicas** son rutas donde la ruta (en la definición de la ruta) contiene un segmento dinámico (indicado por dos puntos, por ejemplo, `:id`).
- Los valores reales de los segmentos de ruta dinámicos se pueden extraer a través del Hook `useParams()`.
- Puedes utilizar la **carga perezosa (*lazy loading*)** para cargar código específico de la ruta solo cuando el usuario realmente visita la ruta.

---

### Sección 7: ¿Qué sigue?

El enrutamiento es una característica que React no admite de fábrica, pero que sigue siendo importante para la mayoría de las aplicaciones de React. Por eso se incluye en este libro y por eso existe la biblioteca React Router. El enrutamiento es un concepto crucial que completa tu conocimiento sobre las ideas y conceptos más esenciales de React, permitiéndote crear aplicaciones de React tanto simples como complejas.

El próximo capítulo se basa en este capítulo y profundiza aún más en React Router, explorando sus capacidades de obtención y manipulación de datos.

---

### Sección 8: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/exercises/questions-answers.md):

1. ¿En qué se diferencia el enrutamiento de cargar contenido condicionalmente?
2. ¿Cómo se definen las rutas?
3. ¿Cómo deberías agregar enlaces a diferentes rutas a tus páginas?
4. ¿Cómo se pueden agregar rutas dinámicas (por ejemplo, detalles de uno de muchos productos) a tu aplicación?
5. ¿Cómo se pueden extraer los valores de los parámetros de ruta dinámicos (por ejemplo, para cargar datos de productos)?
6. ¿Cuál es el propósito de las rutas anidadas?

---

### Sección 9: Aplica lo aprendido

Aplica tus conocimientos sobre enrutamiento a las siguientes actividades.

#### Actividad 13.1: Creación de un sitio web básico de tres páginas
En esta actividad, tu tarea es crear un primer borrador muy básico para un nuevo sitio web de tienda en línea. El sitio web debe admitir tres páginas principales:
- Una página de bienvenida.
- Una página de descripción general de productos que muestra una lista de productos disponibles.
- Una página de detalles del producto, que permite a los usuarios explorar los detalles del producto.

Otras personas agregarán el estilo, el contenido y los datos finales del sitio web, pero tú debes proporcionar algunos datos ficticios y un estilo predeterminado. También debes agregar una barra de navegación principal compartida en la parte superior e implementar la carga perezosa basada en rutas.

Las páginas terminadas deberían verse así:

**Figura 13.12**: La página de bienvenida.

**Figura 13.13**: Una página que muestra algunos marcadores de posición de productos ficticios.

**Figura 13.14**: La página final de detalles del producto con algunos datos y estilos de marcador de posición.

> [!NOTE]
> Para esta actividad, puedes, por supuesto, escribir todos los estilos CSS por tu cuenta. Pero si deseas centrarte en la lógica de React y JavaScript, también puedes utilizar el archivo CSS terminado de la solución en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/13-routing/activities/practice-1/src/index.css](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/13-routing/activities/practice-1/src/index.css).
> Si usas ese archivo, exploralo cuidadosamente para asegurarte de comprender qué IDs o clases CSS podrían necesitar agregarse a ciertos elementos JSX de tu solución. También puedes utilizar los datos ficticios de la solución en lugar de crear tus propios datos de productos ficticios. Encontrarás los datos para esto en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/13-routing/activities/practice-1/src/data/products.js](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/13-routing/activities/practice-1/src/data/products.js).

Para completar la actividad, los pasos de la solución son los siguientes:
1. Crea un nuevo proyecto de React e instala el paquete React Router.
2. Crea los componentes (con el contenido que se muestra en la captura de pantalla anterior) que se cargarán para las tres páginas requeridas.
3. Habilita el enrutamiento y agrega las definiciones de ruta para las tres páginas.
4. Agrega una barra de navegación principal que sea visible para todas las páginas.
5. Agrega todos los enlaces necesarios y asegúrate de que los enlaces de la barra de navegación reflejen si una página está activa o no.
6. Implementa la carga perezosa (para las rutas donde tenga sentido).

> [!NOTE]
> El código completo y la solución para esta actividad se pueden encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/activities/practice-1).
