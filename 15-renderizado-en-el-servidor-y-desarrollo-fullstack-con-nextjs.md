# Parte 2: Construcción de Aplicaciones React Complejas

## Capítulo 15: Renderizado en el Servidor (SSR) y Desarrollo Fullstack con Next.js

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Describir la diferencia entre React del lado del cliente y React del lado del servidor.
- Determinar qué tipo de aplicación React construir.
- Construir aplicaciones React *fullstack* con la ayuda del framework Next.js.
- Explicar las características clave y ventajas de Next.js.

---

### Sección 1: Introducción

Hasta este punto del libro, has aprendido mucho sobre la creación de aplicaciones React del lado del cliente (*client-side*), es decir, aplicaciones donde el código React (transpilado) se ejecuta en los navegadores de los visitantes de tu sitio web.

Esto tiene sentido porque React se creó originalmente para simplificar el proceso de creación de interfaces de usuario interactivas y reactivas mediante la ejecución de código JavaScript en el lado del cliente. Hasta la fecha, la mayoría de las características de React, incluidas las cubiertas hasta este punto en este libro (por ejemplo, estado, contexto y enrutamiento), existen para cumplir este propósito.

Pero, como aprenderás en este y en los siguientes capítulos, en realidad también puedes **ejecutar código React en el lado del servidor**. Hay ciertas características de React que solo se pueden usar allí, por ejemplo, los componentes de servidor de React (*React Server Components* o RSC), que se tratarán con gran detalle en el Capítulo 16, *React Server Components y Server Actions*.

Este capítulo te iniciará en React en el lado del servidor, explicará brevemente qué es el renderizado en el servidor (*Server-Side Rendering* o **SSR**) y te presentará **Next.js**, un framework *fullstack* popular y rico en funciones para React que te permite combinar código backend y frontend. Aprenderás a crear aplicaciones Next.js y a utilizar funciones centrales de Next.js, como el enrutamiento basado en archivos (*file-based routing*).

---

### Sección 2: ¿Cuál es el problema con las aplicaciones React del lado del cliente?

La gran ventaja de las aplicaciones de página única (*Single-Page Applications* o SPAs) y de React en el lado del cliente es que puedes crear interfaces de usuario web altamente reactivas e interactivas. La interfaz de usuario se puede actualizar casi instantáneamente, se pueden evitar recargas y cambios de página visibles y, por lo tanto, tus usuarios se benefician de una experiencia de usuario similar a la de una aplicación móvil.

Pero esta dependencia del código del lado del cliente (y, por lo tanto, de JavaScript) también tiene posibles desventajas:
1. Si los usuarios deshabilitan JavaScript, el sitio web será prácticamente inutilizable.
2. El documento HTML obtenido inicialmente está casi vacío: la obtención de datos y el renderizado del contenido solo tienen lugar después de esa solicitud y respuesta HTTP inicial.

El primer punto puede no importar demasiado, ya que solo un pequeño subconjunto de todos los usuarios desactivará JavaScript y puedes mostrar un mensaje de advertencia adecuado a través de la etiqueta `<noscript>`.

Pero el segundo punto puede tener consecuencias significativas. Dado que el documento HTML inicial está casi vacío, los usuarios no verán ningún contenido hasta que todo el código JavaScript haya sido descargado y ejecutado. Si bien la mayoría de los usuarios no verán un retraso notable, según el dispositivo y la conexión a Internet del usuario, esto puede demorar hasta unos segundos para algunos visitantes.

Además, los rastreadores de los motores de búsqueda (por ejemplo, el rastreador de Google) no esperarán necesariamente a que se descargue y ejecute todo el código JavaScript del lado del cliente al indexar tu página. Por lo tanto, esos rastreadores pueden ver una página mayormente vacía y, por ende, posicionar mal tu sitio web (o no indexarlo en absoluto).

**Figura 15.1**: El contenido de la página no se encuentra en ninguna parte del código fuente de la página (es decir, en el documento HTML obtenido).

La Figura 15.1 muestra el código fuente de la página (que se puede inspeccionar haciendo clic derecho en el sitio web) de una aplicación React típica. Como puedes ver en esa figura, casi no hay contenido entre las etiquetas `<body>`. El título ("Hello World!") y el texto que aparece debajo no se encuentran en ese código fuente. El contenido falta allí porque no forma parte de la respuesta HTTP inicial. En cambio, es renderizado por el código React transpilado después de que se cargó la página (y después de que ese código se descargó del servidor).

Por supuesto, estas desventajas no importarán en todos los casos. Si estás creando una aplicación interna para una empresa, o una interfaz de usuario que está oculta detrás de un inicio de sesión (y que, por lo tanto, no se indexará de todos modos), o si solo te diriges a usuarios con dispositivos y conexiones a Internet rápidos, es posible que no tengas que preocuparte por estos posibles problemas.

Pero si estás creando un sitio web de cara al público donde la indexación en motores de búsqueda es importante o que puede ser visitado por usuarios con dispositivos o conexiones a Internet lentas, es posible que desees considerar eliminar estas desventajas. Y ahí es precisamente donde el **SSR** puede ayudarte.

---

### Sección 3: Entendiendo el Renderizado en el Servidor (SSR)

Cuando se trabaja con React, **SSR** se refiere al proceso de **renderizar páginas web, y por lo tanto tus componentes de React, en el servidor** que maneja la solicitud HTTP entrante cuando un usuario visita tu sitio web.

Con SSR habilitado, el servidor renderizará tu árbol de componentes de React y, por lo tanto, producirá el código HTML real generado por tus componentes y sus instrucciones JSX. Es este código HTML terminado el que luego se envía de regreso al cliente. Como resultado, los visitantes del sitio web recibirán un archivo HTML que ya no estará vacío, sino que contendrá el contenido real de la página. Los rastreadores de motores de búsqueda también verán ese contenido e indexarán la página en consecuencia.

**Figura 15.2**: React SSR en acción.

Lo mejor de todo es que no pierdes las ventajas del lado del cliente de React porque, al habilitar SSR, ¡React sigue funcionando en el lado del cliente como lo hacía antes! Tomará el control una vez que se haya recibido ese documento HTML inicial y brindará a los usuarios la misma experiencia de SPA que podías ofrecer sin SSR. Aunque, técnicamente, al usar SSR, React se inicializará de manera ligeramente diferente en el cliente: en lugar de volver a renderizar todo el DOM allí, **hidratará (*hydrate*) el contenido de la página que se renderizó en el servidor**. La **hidratación (*hydration*)** significa que React conectará la estructura de tus componentes al código HTML renderizado (que se renderizó en base a esa misma estructura, por supuesto) y lo hará interactivo.

**Figura 15.3**: Después de recibir el código HTML renderizado, React hidrata el código en el lado del cliente.

En consecuencia, obtendrás lo mejor de ambos mundos: páginas pre-renderizadas no vacías para la solicitud HTTP inicial enviada por el navegador, y una aplicación web altamente reactiva para que el usuario la disfrute.

---

### Sección 4: Añadir SSR a una aplicación React

Es extremadamente importante comprender que las aplicaciones React habilitadas para SSR necesitan ejecutar código en **dos entornos (servidor y navegador)**, mientras que las aplicaciones React del lado del cliente solo dependen del navegador. Por lo tanto, para usar SSR, se debe agregar un entorno del lado del servidor al proyecto React; no basta con ajustar el código React en unos pocos lugares.

Por ejemplo, los proyectos estándar basados en Vite no admiten SSR de fábrica. En consecuencia, si deseas admitir SSR, debes editar la configuración de tu proyecto Vite (y algunos de los archivos de código del proyecto) para permitir la ejecución del código React tanto en el lado del cliente como en el del servidor. Por ejemplo, debes agregar código que maneje las solicitudes HTTP entrantes y active la ejecución del código React en el lado del servidor.

> [!NOTE]
> Habilitar SSR manualmente requiere conocimientos de desarrollo backend y de configuración de procesos de compilación, además del conocimiento de React que necesitas.
> Afortunadamente, como aprenderás a lo largo de este capítulo, a menudo no necesitas pasar por ese proceso de configuración manual. En su lugar, puedes confiar en frameworks como Next.js para que hagan el trabajo pesado por ti.
> Si estás interesado en configurar manualmente SSR en proyectos basados en Vite, la documentación oficial de Vite SSR es un excelente lugar para obtener más información: [https://vitejs.dev/guide/ssr](https://vitejs.dev/guide/ssr).
> Además, puedes explorar el siguiente proyecto de demostración que se configuró de acuerdo con las instrucciones oficiales de Vite SSR: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/02-ssr-enabled](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/02-ssr-enabled).

El hecho de que habilitar manualmente SSR sea un proceso no trivial que requiere conocimientos avanzados de Node.js y desarrollo backend es una de las razones por las que la documentación oficial de React recomienda crear nuevos proyectos de React con la ayuda de frameworks como Next.js (consulta [https://react.dev/learn/start-a-new-react-project](https://react.dev/learn/start-a-new-react-project)).

Pero no es la única razón.

#### La obtención de datos en el lado del servidor no es trivial
Además del proceso de configuración no trivial, los proyectos habilitados para SSR también sufren de otro posible problema: **la obtención de datos en el lado del servidor es difícil**.

Si estás creando una aplicación React que necesita obtener datos en algunos componentes (por ejemplo, con la ayuda de `useEffect()`, como se muestra en el Capítulo 8, *Manejo de Efectos Secundarios*), descubrirás que **los datos no se obtienen cuando el componente se renderiza en el servidor**. En cambio, la obtención de datos solo ocurrirá en el lado del cliente. El marcado HTML renderizado en el lado del servidor no contendrá el contenido que depende de los datos obtenidos.

La razón de este comportamiento es que **las funciones de los componentes de React solo se ejecutan en el servidor una vez**, es decir, solo el primer ciclo de renderizado del componente se realiza en el servidor. Puedes pensar en el SSR como la producción de solo una instantánea (*snapshot*) inicial de la página. Las actualizaciones de estado posteriores se ignoran y, por lo tanto, las funciones de efecto (activadas a través de `useEffect()`) tampoco se ejecutan en el lado del servidor. Como resultado, la obtención de datos que depende de funciones de efecto y actualizaciones de estado posteriores no funcionará en el lado del servidor.

Considera este ejemplo, donde una función de componente `Todos` usa `useEffect()` para obtener algunos datos de tareas pendientes ficticias de `jsonplaceholder.typicode.com`:

```javascript
import { useEffect, useState } from 'react'; 
import { loadTodos, saveTodo } from '../todos.js'; 

function Todos() { 
  const [todos, setTodos] = useState(); 

  useEffect(() => { 
    async function fetchTodos() { 
      // sends HTTP request to jsonplaceholder.typicode.com 
      const todos = await loadTodos(); 
      setTodos(todos); 
    } 
    fetchTodos(); 
  }, []); 

  async function addTodoAction(fd) { 
    const todo = { 
      title: fd.get('title'), 
    }; 
    const savedTodo = await saveTodo(todo); 
    setTodos((prevTodos) => [savedTodo, ...prevTodos]); 
  } 

  return ( 
    <section> 
      <h2>Manage your todos</h2> 
      <form action={addTodoAction}> 
        <input type="text" name="title" /> 
        <button type="submit">Add Todo</button> 
      </form> 
      {(!todos || todos.length === 0) && ( 
        <p>No todos found.</p> 
      )} 
      {todos && todos.length > 0 && ( 
        <ul> 
          {todos.map((todo) => ( 
            <li key={todo.id}>{todo.title}</li> 
          ))} 
        </ul> 
      )} 
    </section> 
  ); 
}
```

> [!NOTE]
> Puedes encontrar el código de ejemplo completo en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/03-ssr-data-fetching](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/03-ssr-data-fetching).

Al ejecutar este código en el servidor, no habrá ningún error. En cambio, la aplicación se ejecutará como se espera y obtendrá las tareas pendientes ficticias del servidor backend.

Sin embargo, el documento HTML que se produce en el servidor no contendrá las tareas pendientes obtenidas. En su lugar, simplemente contendrá el texto alternativo ("No todos found").

**Figura 15.4**: El HTML renderizado no contiene las tareas pendientes reales.

El marcado generado no contiene las tareas pendientes obtenidas porque, como se explicó anteriormente, las funciones de los componentes de React solo se ejecutan una vez en el lado del servidor (y la función pasada a `useEffect()` no se ejecuta en absoluto).

Debido a este comportamiento, no puedes realizar fácilmente operaciones asíncronas y, por ejemplo, obtener datos a través de `useEffect()` en tus componentes de React cuando usas SSR. Por lo tanto, el contenido HTML renderizado en el lado del servidor nunca contendrá esos datos.

Si bien puedes encontrar soluciones alternativas para ese problema (por ejemplo, realizar la operación de obtención de datos en el servidor antes de ejecutar las funciones de los componentes), ese es un problema que resolverán Next.js y un concepto llamado **React Server Components (RSC)**.

---

### Sección 5: Presentando Next.js

Next.js es un framework de React, es decir, un framework que se basa en React y le agrega características y patrones adicionales. Específicamente, Next.js agrega características como enrutamiento basado en archivos, SSR integrado o almacenamiento en caché automático para mejorar el rendimiento. Aunque, lo más importante, desbloquea dos conceptos cruciales de React: **React Server Components (RSC)** y **Server Actions**. Como aprenderás, estas funciones permiten que el código React del lado del servidor realice operaciones asíncronas y, por ejemplo, obtenga y renderice datos en el servidor.

Por lo tanto, Next.js te ahorra el esfuerzo de habilitar manualmente SSR y, además, desbloquea otras funciones potentes que ayudan a obtener datos en el servidor.

> [!NOTE]
> También existen frameworks alternativos de React como Remix/React Router (se fusionaron para incorporar características opcionales de framework React fullstack a React Router) o TanStack Start.
> Next.js no solo existe desde hace mucho tiempo, sino que también es el framework fullstack más popular (medido por uso) en el momento en que se escribió este libro.

Este capítulo te iniciará en Next.js y te proporcionará una breve descripción general de sus conceptos básicos. El próximo capítulo (Capítulo 16, *React Server Components y Server Actions*) se basará en este conocimiento para profundizar aún más.

#### Creación de proyectos Next.js
Para usar Next.js, primero debes crear un proyecto Next.js. Técnicamente, seguirá siendo un proyecto de React, lo que significa que podrás usar características de React como componentes, props, estado, Hooks o JSX. Pero será un proyecto que viene con el paquete `next` instalado y que impone una cierta estructura de carpetas que Next.js necesita. No puedes simplemente instalar Next.js en un proyecto React existente (basado en Vite) y comenzar a usarlo allí; se requerirían ajustes significativos en la configuración y estructura del proyecto. Next.js trae su propio proceso de compilación y no usa Vite bajo el capó. Por lo tanto, crear un proyecto nuevo tiene más sentido.

Para comenzar con un nuevo proyecto de Next.js, debes ejecutar el siguiente comando en el terminal de tu sistema o en el símbolo del sistema (en el lugar de tu sistema donde deseas que se cree la nueva carpeta del proyecto):

```bash
npx create-next-app@latest first-next-app
```

Después de ejecutar este comando, tendrás que tomar un par de decisiones en el terminal (por ejemplo, si deseas usar TypeScript).

Puedes confirmar todas esas opciones simplemente presionando la tecla Intro (*Enter*), aceptando así la opción predeterminada. Sin embargo, debes asegurarte de elegir **No para TypeScript** (a menos que sepas cómo usarlo) y **Yes para App Router**. Puedes encontrar un proyecto inicial (ligeramente limpio) en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/04-nextjs-intro](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/04-nextjs-intro).

Dentro de la carpeta del proyecto creado, se puede iniciar un servidor de desarrollo a través de:

```bash
npm run dev
```

Si bien el comando es el mismo que en un proyecto de Vite, el servidor en realidad apuntará a un puerto diferente de forma predeterminada. En lugar de `localhost:5173` (Vite), los proyectos de Next.js usan **`localhost:3000`** para el servidor de desarrollo de vista previa.

Al igual que en un proyecto basado en Vite, debes mantener este proceso en ejecución mientras trabajas en el código del proyecto. El proceso de compilación subyacente recargará y actualizará automáticamente el sitio web de vista previa a medida que realices cambios en tu código.

> [!NOTE]
> Next.js es un framework maduro y establecido que nunca ha dejado de innovar y cambiar.
> A finales de 2022, se introdujo el llamado **App Router** como una nueva forma de estructurar y crear aplicaciones Next.js (el enfoque antiguo ahora se conoce como *Pages Router*). Este libro, por supuesto, cubre el nuevo enfoque de App Router.
> A mediados de 2024 (cuando se escribe esta edición), el enfoque de App Router, a pesar de estar marcado como estable, todavía recibe con frecuencia nuevas funciones y cambios.
> Por lo tanto, aunque es poco probable, los conceptos y el código explicados en este libro pueden cambiar con el tiempo. El proceso de configuración descrito anteriormente también puede cambiar. En tales casos, se agregará una nota (con instrucciones sobre cómo ajustar el código) al documento Changelog en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/main/CHANGELOG.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/main/CHANGELOG.md).

Un proyecto de Next.js recién creado viene con todas sus dependencias instaladas (`npm install` se ejecuta automáticamente como parte del proceso de creación del proyecto) y una estructura de proyecto como esta:
- Una carpeta **`app/`** que contiene archivos relacionados con rutas (consulta la siguiente sección).
- Una carpeta **`public/`** que se puede utilizar para almacenar recursos estáticos que deben servirse sin modificaciones (es decir, sin ser modificados por el proceso de compilación).
- Archivos `jsconfig.json` y `nextjs.config.mjs` para configurar el proyecto y los comportamientos específicos de Next.js.
- `package.json` y `package-lock.json` para administrar las dependencias del proyecto.

Por lo tanto, excepto por la carpeta `app/`, no es muy diferente de la estructura que conoces de Vite. Sin embargo, vale la pena señalar que Next.js, a diferencia de Vite, no impone `.jsx` como extensión de archivo para los archivos JavaScript que contienen código JSX. Puedes usarlo, pero no es obligatorio. Por ejemplo, el proyecto inicial utiliza `page.js` y `layout.js`, no `page.jsx` y `layout.jsx`, a pesar de que estos archivos contienen código JSX.

Al igual que los proyectos basados en Vite, los proyectos de Next.js vienen con un flujo de trabajo de compilación que procesa y transpila tus archivos de código automáticamente al ejecutar el servidor de desarrollo o al compilar para producción (lo cual puedes hacer mediante `npm run build`).

Como prácticamente todas las configuraciones modernas de proyectos de React, los proyectos de Next.js admiten la importación de archivos de estilo (como `globals.css`) o imágenes en archivos JavaScript. También te permite omitir o establecer extensiones de archivo en las importaciones. Además, Next.js también tiene soporte para CSS Modules.

Dicho de otra manera: puedes trabajar en proyectos de Next.js prácticamente de la misma manera que lo haces en proyectos basados en Vite.

#### Trabajando con rutas basadas en archivos (*File-Based Routes*)
En los proyectos basados en Vite, tienes un alto grado de flexibilidad en lo que respecta a la estructura del proyecto. Dentro de la carpeta `src/`, puedes crear cualquier subcarpeta y archivo de tu elección. Los nombres de esos archivos y carpetas tampoco importan realmente (siempre que sean válidos y usen las extensiones correctas).

Cuando trabajabas con React Router, configurabas las rutas en uno de tus archivos de código JSX y cargabas cualquier componente almacenado en cualquier archivo para cualquier ruta (consulta el Capítulo 13, *Aplicaciones Multipágina con React Router*).

En los proyectos de Next.js, eso es un poco diferente porque **Next.js usa el sistema de archivos para definir rutas**: no configuras rutas en el código. Como resultado, si bien todavía tienes mucha flexibilidad, hay algunas reglas relacionadas con el enrutamiento con respecto a la estructura del proyecto y los nombres de los archivos que se deben seguir; de lo contrario, la aplicación se romperá y no funcionará según lo previsto.

Next.js implementa el enrutamiento basado en archivos a través de su propio enrutador integrado. Este enrutador analiza tu sistema de archivos y deduce las rutas admitidas, sus rutas de URL y qué componentes de React cargar y renderizar en función de la estructura de archivos y carpetas de tu proyecto.

Al utilizar el enfoque de App Router, debes almacenar todos los componentes que deben cargarse como páginas dentro de la carpeta `app/` (o una carpeta anidada) en **archivos llamados `page.js`**. Dado que todos los archivos de componentes de ruta deben llamarse `page.js`, **son los nombres de las carpetas principales los que definen la ruta de URL** para la cual se cargará el componente.

Por ejemplo, podrías tener una estructura de archivos y carpetas como se muestra en la Figura 15.5:

**Figura 15.5**: En Next.js, los archivos page.js contienen componentes de ruta. Los nombres de las carpetas determinan la ruta.

En la Figura 15.5, puedes ver que se definen cuatro rutas a través del sistema de archivos: una ruta raíz (`/`) y las rutas `/about`, `/users` y `/terms/en`. Para cada ruta, el componente almacenado en el `page.js` respectivo se cargará y se renderizará en la pantalla.

Por ejemplo, podrías tener un archivo `app/page.js` como este:

```javascript
export default function Home() { 
  return ( 
    <main> 
      <h1>Hello Next.js World </h1> 
      <p>Build fullstack React applications with ease!</p> 
      <p> 
        Learn more about Next.js in{' '} 
        <a href="https://www.udemy.com/course/nextjs-react-the-complete-guide/"> 
          my course 
        </a>{' '} 
        or the <a href="https://nextjs.org/">official documentation</a>. 
      </p> 
    </main> 
  ); 
}
```

Como puedes ver, en este archivo `page.js` se almacena una función de componente de React normal. El nombre de la función del componente no importa: solo es importante que sea una función de componente que se exporte de forma predeterminada dentro de un archivo llamado `page.js`. Como resultado, el siguiente contenido será visible en la pantalla si un usuario visita `<dominio>/` (o simplemente `<dominio>`, sin la barra diagonal):

**Figura 15.6**: El enrutador de Next.js carga el componente almacenado en el archivo app/page.js y renderiza su contenido.

Por lo tanto, puedes agregar fácilmente tantas rutas como sea necesario, posiblemente anidadas, simplemente creando carpetas, subcarpetas y archivos `page.js`.

#### Renderizado en el servidor con Next.js
Además de proporcionar un enrutador integrado basado en archivos (y muchas otras funciones que se explorarán a lo largo de este y los siguientes capítulos), Next.js tiene otra ventaja crucial: **implementa SSR de fábrica**. No tienes que agregar ningún archivo, cambiar ninguna configuración ni ajustar ningún código para renderizar componentes de React en el servidor; en cambio, funciona desde el principio.

En consecuencia, el componente del archivo `app/page.js` (el componente `Home` en el ejemplo anterior) se evalúa y renderiza en el lado del servidor cuando un usuario visita `<dominio>/`. Es el código HTML terminado el que se envía al navegador. Y, al igual que con los proyectos basados en Vite con SSR personalizado, Next.js también renderiza todos los componentes secundarios que se pueden usar dentro de `page.js` en el servidor.

Además, al crear sitios web con Next.js, sigues creando aplicaciones React. Es por eso que las aplicaciones de Next.js se vuelven interactivas en el lado del cliente una vez que se realiza el SSR. Técnicamente, como también aprenderás en el próximo capítulo (*React Server Components y Server Actions*), se volverán interactivas de una manera diferente a las aplicaciones React habilitadas para SSR basadas en Vite (donde React hidrata el marcado renderizado en el servidor en el cliente), pero en última instancia, los usuarios de tu sitio web tendrán una experiencia de usuario similar a una SPA.

Por lo tanto, si deseas crear una aplicación de React que admita SSR, se recomienda confiar en un framework como Next.js en lugar de configurar SSR manualmente.

Además, podrás utilizar otras funciones útiles, como el sistema de enrutamiento basado en archivos, especialmente porque no se limita a definir rutas a través de archivos `page.js`. Por ejemplo, también simplifica el proceso de definición de diseños (*layouts*).

#### Trabajando con Layouts
Como se mencionó, cuando se trata de enrutamiento, los nombres de los archivos y dónde los almacenas son importantes.

Por ejemplo, también encontrarás un archivo **`layout.js`** junto al archivo `page.js` en la carpeta `app/` del ejemplo anterior.

**Figura 15.7**: Además de un archivo page.js, un archivo de estilos y un favicon, se puede encontrar un archivo layout.js en la carpeta app/.

Al igual que `page.js`, `layout.js` es un nombre de archivo reservado, es decir, Next.js maneja ese archivo de una manera especial.

Este archivo `layout.js` también exporta una función de componente, pero el componente creado no se renderizará para una ruta específica. En su lugar, **se utiliza como un contenedor alrededor de todas las páginas hermanas o anidadas**. Por lo tanto, el archivo `layout.js` se puede utilizar para definir código JSX que se compartirá entre varias páginas.

Dado que está pensado para ser utilizado como un componente contenedor, la función de componente exportada por `layout.js` debe utilizar la prop especial **`children`** (consulta el Capítulo 3, *Componentes y Props*) para definir el lugar donde debe mostrarse el contenido de la página envuelta.

Por ejemplo, podrías usar el archivo `app/layout.js` para definir un diseño global que agregue una barra de navegación encima del contenido `<main>`:

```javascript
export default function RootLayout({ children }) { 
  return ( 
    <html lang="en"> 
      <body> 
        <header> 
          <nav> 
            <ul> 
              <li><a href="/">Home</a></li> 
              <li><a href="/events">Events</a></li> 
            </ul> 
          </nav> 
        </header> 
        <main>{children}</main> 
      </body> 
    </html> 
  ); 
}
```

En este fragmento de código de ejemplo, también vale la pena señalar que el componente `RootLayout` renderiza los elementos `<html>` y `<body>`. En proyectos basados en Vite, eso no es algo que harías. Allí, en su lugar, defines un lugar en el archivo `index.html` donde se debe inyectar el HTML renderizado (a través de la función `createRoot()` expuesta por el paquete `react-dom`; consulta el Capítulo 2, *Entendiendo los Componentes de React y JSX*).

Next.js no depende de dicho archivo `index.html`; en su lugar, te obliga a definir un archivo `layout.js` raíz en el nivel superior de la carpeta `app/`. Es entonces este diseño raíz el que debe definir la estructura general de la página HTML renderizada. Sin embargo, no hay ninguna sección `<head>` en ese archivo, ya que Next.js administrará e inyectará esa sección detrás de escena. Además, Next.js también insertará importaciones de JavaScript y CSS en el documento HTML renderizado.

Puedes agregar más archivos `layout.js` (anidados) si deseas tener diseños anidados que solo envuelvan algunas de tus páginas. Dichos diseños son opcionales; el diseño raíz (`app/layout.js`) es obligatorio, sin embargo.

Con un archivo `layout.js` como el que se muestra en el ejemplo de código anterior, en un proyecto que contiene un archivo `app/page.js` y un archivo `app/events/page.js`, los usuarios del sitio web podrían visitar ambas páginas y ver una navegación compartida.

**Figura 15.8**: A medida que el usuario navega hacia/desde /events, el encabezado compartido persiste.

En la Figura 15.8, el contenido principal (definido por los archivos `page.js`) cambia, pero la navegación compartida (configurada en `layout.js`) persiste.

Si bien compartir marcado JSX es el caso de uso más común para usar diseños, también puedes usarlos para compartir estilos importando un archivo CSS en un archivo `layout.js`:

```javascript
import './globals.css'; 

export default function RootLayout({ children }) { 
  return ( 
    <html lang="en"> 
      Unchanged JSX code… 
    </html> 
  ); 
}
```

En este y en los ejemplos anteriores, la función del componente se llama `RootLayout`: ese nombre no importa, pero debe ser un componente que se exporte por defecto.

Por supuesto, los diseños que se utilizan para compartir una barra de navegación se vuelven aún más útiles si les agregas enlaces funcionales...

#### Gestión de la navegación interna
En el ejemplo de código anterior, se utilizó el elemento `<a>` para crear enlaces entre las diferentes páginas de la aplicación Next.js.

Sin embargo, al igual que otras aplicaciones React, las aplicaciones Next.js se convierten en SPAs una vez que se realiza la carga inicial de la página. Por lo tanto, se desaconseja la creación de enlaces internos a través de etiquetas `<a>` por las mismas razones por las que se desaconsejaba al usar React Router en proyectos de React basados en Vite (compara el Capítulo 13, *Aplicaciones Multipágina con React Router*).

Al igual que React Router, Next.js (que se encarga del enrutamiento en proyectos de Next.js) proporciona un componente especial **`Link`** que debes usar para enlaces internos (en lugar del elemento `<a>`):

```javascript
import Link from 'next/link'; 

export default function RootLayout({ children }) { 
  return ( 
    <html lang="en"> 
      <body> 
        <header> 
          <nav> 
            <ul> 
              <li><Link href="/">Home</Link></li> 
              <li><Link href="/events">Events</Link></li> 
            </ul> 
          </nav> 
        </header> 
        <main>{children}</main> 
      </body> 
    </html> 
  ); 
}
```

Este componente `<Link>` acepta una prop **`href`**, que se establece en la ruta de destino. Internamente, Next.js capturará los clics en los enlaces y actualizará la barra de direcciones del navegador y la interfaz de usuario del sitio web en consecuencia cargando y renderizando los componentes `page.js` requeridos.

#### Resaltar enlaces activos y uso de la directiva "use client"
Si deseas diseñar los enlaces de manera diferente cuando conducen a la página actualmente activa, no encontrarás un componente `NavLink` integrado como ocurre con React Router. En su lugar, debes agregar tu propia lógica configurando la prop `className` del componente `Link` dinámicamente en función de la ruta actualmente activa.

Para averiguar qué ruta está activa actualmente, puedes utilizar el Hook **`usePathname()`** proporcionado por Next.js:

```javascript
import { usePathname } from 'next/navigation'; 
const path = usePathname();
```

Por ejemplo, podrías ajustar el archivo `layout.js` para que se vea así:

```javascript
import Link from 'next/link'; 
import { usePathname } from 'next/navigation'; 
import './globals.css'; 

export default function RootLayout({ children }) { 
  const path = usePathname(); 
  return ( 
    <html lang="en"> 
      <body> 
        <header> 
          <nav> 
            <ul> 
              <li> 
                <Link href="/" className={path === '/' ? 'active' : ''}> 
                  Home 
                </Link> 
              </li> 
              <li> 
                <Link 
                  href="/events" 
                  className={path.startsWith( '/events' ) ? 'active' : ''} 
                > 
                  Events 
                </Link> 
              </li> 
            </ul> 
          </nav> 
        </header> 
        <main>{children}</main> 
      </body> 
    </html> 
  ); 
}
```

Sin embargo, si intentaras ejecutar este código, obtendrías un mensaje de error:

**Figura 15.9**: Next.js se queja del uso de un Hook en un Server Component.

Este mensaje de error suena bastante críptico ya que menciona un *Client Component* y *Server Components*. Ambos son conceptos cruciales de React que se explorarán en el próximo capítulo (*React Server Components y Server Actions*).

Para el capítulo actual, es suficiente conocer la solución para este problema, que consiste en agregar la directiva **`"use client"`** en la parte superior del archivo `app/layout.js`:

```javascript
"use client"; 
import Link from 'next/link'; 
import { usePathname } from 'next/navigation'; 
import './globals.css'; 

export default function RootLayout({ children }) { 
  const path = usePathname(); 
  // return JSX code 
}
```

`"use client"` es una directiva, es decir, una instrucción que "le dice" a React y Next.js que este archivo debe manejarse de una manera especial. Agregarlo eliminará el mensaje de error que se muestra en la Figura 15.9, lo que permitirá aplicar estilos a `Link` en función de la ruta. Como se mencionó, el impacto concreto de esta directiva se explorará en el próximo capítulo.

Siempre que planees **usar un Hook en un componente en un proyecto Next.js, se debe agregar la directiva `"use client"`**, sin importar si es un Hook proporcionado por React o por Next.js.

> [!NOTE]
> Quizás te preguntes por qué se requiere `"use client"` para los componentes que usan Hooks. Después de todo, esta directiva no era necesaria cuando se usaba SSR en proyectos basados en Vite.
> La razón es que Next.js técnicamente no usa SSR clásico tal como se introdujo al comienzo de esta sección. En su lugar, Next.js (cuando usa el App Router) usa una característica de React llamada **React Server Components**. Esta característica crucial se explorará con gran detalle en el próximo capítulo. Allí también aprenderás por qué exactamente se necesita `"use client"` en algunos componentes.

#### Creación y uso de componentes regulares
El componente `Link` mencionado en las secciones anteriores es un componente ofrecido por Next.js. Pero, por supuesto, también puedes crear tus propios componentes; después de todo, sigue siendo una aplicación de React.

Además de los componentes que se exponen como páginas (`page.js`) o diseños (`layout.js`), puedes crear y utilizar funciones de componentes en cualquier archivo (con cualquier nombre) de tu elección.

Por ejemplo, puedes agregar una carpeta `components/` junto a la carpeta `app/` y agregar un archivo `MainNavigation.js` en ella. Este archivo luego puede contener un nuevo componente `MainNavigation` que devuelve el código JSX relacionado con la navegación:

```javascript
'use client'; 
import Link from 'next/link'; 
import { usePathname } from 'next/navigation'; 

export default function MainNavigation() { 
  const path = usePathname(); 
  return ( 
    <header> 
      <nav> 
        <ul> 
          <li> 
            <Link href="/" className={path === '/' ? 'active' : ''}> 
              Home 
            </Link> 
          </li> 
          <li> 
            <Link 
              href="/events" 
              className={path === '/events' ? 'active' : ''} 
            > 
              Events 
            </Link> 
          </li> 
        </ul> 
      </nav> 
    </header> 
  ); 
}
```

Ten en cuenta que `"use client"` debe agregarse en la parte superior de este archivo `MainNavigation.js` ya que el Hook `usePathname()` se usa en la función del componente.

Con el código movido a este componente `MainNavigation` recién agregado, dentro del archivo `layout.js`, **se puede eliminar `"use client"`** ya que el Hook `usePathname()` ya no se usa directamente en ese archivo. Se usa en un componente secundario (dentro de `<MainNavigation/>`), pero a React no le importa esto.

Por lo tanto, el archivo `layout.js` actualizado se ve así:

```javascript
import './globals.css'; 
import MainNavigation from '../components/MainNavigation'; 

export default function RootLayout({ children }) { 
  return ( 
    <html lang="en"> 
      <body> 
        <MainNavigation /> 
        <main>{children}</main> 
      </body> 
    </html> 
  ); 
}
```

Gracias a la creación, externalización y uso del componente personalizado `MainNavigation`, el archivo `layout.js` actualizado vuelve a contener una función de componente limpia y concisa.

> [!NOTE]
> Con la excepción de los archivos relacionados con rutas, depende totalmente de ti cómo estructuras tu proyecto Next.js y cómo nombras tus archivos.
> Como se mencionó, puedes almacenar componentes personalizados (que no sean de página) en una carpeta `components/` (o una carpeta con cualquier otro nombre de tu elección) en cualquier lugar que elijas. Puedes colocar esa carpeta `components/` dentro de la carpeta `app/` o en la carpeta raíz del proyecto.
> También puedes no usar una carpeta `components/` en absoluto y, en su lugar, almacenar componentes en archivos ubicados junto a tus archivos `page.js`. Porque si un archivo no se llama `page.js`, no se tratará como una página, por lo que no hay peligro de crear accidentalmente rutas que no deseas en tu proyecto. Si tienes un archivo `app/components/MainNavigation.js` pero ningún archivo `app/components/page.js`, no habrá una ruta `/components`. Los archivos que no se llamen `page.js` (o uno de los otros nombres de archivo reservados; consulta la próxima sección *Otras convenciones de nombres de archivo*) simplemente serán ignorados por Next.js (para fines de enrutamiento).
> Encontrarás más información e ideas sobre la organización de proyectos Next.js en la documentación oficial: [https://nextjs.org/docs/app/building-your-application/routing/colocation](https://nextjs.org/docs/app/building-your-application/routing/colocation).

#### Manejo de rutas dinámicas
Como aprendiste en el Capítulo 13, *Aplicaciones Multipágina con React Router*, en la sección *De rutas estáticas a rutas dinámicas*, muchas aplicaciones de React también necesitan admitir rutas dinámicas.

Por ejemplo, es posible que desees permitir que tus usuarios visiten `/events/e1` para ver los detalles de un evento con ID `e1` y `/events/e2` para un evento con ID `e2` (y así sucesivamente).

Este es un requisito tan común que Next.js, por supuesto, lo admite. Puedes agregar rutas dinámicas en una aplicación Next.js creando una carpeta (en algún lugar de la carpeta `app/`) cuyo **nombre esté entre corchetes**, por ejemplo, `app/events/[eventId]`. Por supuesto, todavía necesitas un archivo `page.js` en esa carpeta para crear realmente una ruta.

La parte entre corchetes (`eventId`, en este ejemplo) depende totalmente de ti. Pero los corchetes le indican a Next.js que estás configurando una ruta dinámica.

El nombre de la carpeta entre corchetes actúa como un **identificador** que se puede utilizar para recuperar el valor concreto codificado en la URL (por ejemplo, para recuperar `e1` en `/events/e1`).

Cada componente que se utiliza como página (o diseño) recibe una prop **`params`** que Next.js establece automáticamente. Si es una página o diseño en una carpeta de ruta dinámica o en alguna carpeta secundaria anidada, la prop `params` contendrá una Promesa que se resuelve en un objeto que contiene los identificadores elegidos (como `eventId`) como claves y los valores de ruta de URL concretos (como `e1`) como valores para esas claves. Dado que `params` contiene una Promesa, **se debe usar `await` sobre ella para obtener acceso al objeto subyacente**.

Por ejemplo, el archivo `app/events/[eventId]/page.js` garantizaría que el componente exportado dentro del archivo `page.js` se renderice para visitas a `/events/e1`, `/events/e2`, etc. Este componente de página puede luego mostrar los detalles del evento con la ayuda del siguiente código:

```javascript
// getEventById is a custom dummy function to load event data 
import { getEventById } from '@/lib/events'; 

export default async function EventDetailsPage({ params }) { 
  // params.eventId exists because of folder name => [eventId] 
  const { eventId } = await params; 
  const event = getEventById(eventId); 
  return ( 
    <div id="event-details"> 
      <header> 
        <img src={`/${event.image}`} alt={event.title} /> 
        <h1>{event.title}</h1> 
        <p> 
          {event.location} | {event.date} 
        </p> 
      </header> 
      <p>{event.description}</p> 
      <p> 
        <button>Register</button> 
      </p> 
    </div> 
  ); 
}
```

En este ejemplo, la prop `params` proporcionada automáticamente se utiliza para obtener acceso al `eventId` codificado en la URL. Si se utilizara algún otro identificador además de `eventId` en el nombre de la carpeta, ese nombre alternativo se utilizaría para acceder al valor de la ruta (por ejemplo, para `[id]/page.js`, accederías a `(await params).id`).

Como resultado, los usuarios pueden visitar esta ruta dinámica y explorar los detalles del evento para un ID de evento elegido.

**Figura 15.10**: Los detalles del evento se cargan y se muestran para /events/e1.

> [!NOTE]
> Puedes encontrar el código de ejemplo completo, incluido el archivo `lib/events.js`, en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/08-nextjs-dynamic-routes](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/examples/08-nextjs-dynamic-routes).

Por supuesto, cuando se trabaja con rutas dinámicas, normalmente también se necesitan enlaces a esas rutas dinámicas en algunas partes de la aplicación. Por lo tanto, en este ejemplo, el archivo `app/events/page.js` contiene código que renderiza dinámicamente una lista de elementos de eventos, donde cada elemento tiene un enlace a su página de detalles:

```javascript
import Link from 'next/link'; 
import { getEvents } from '@/lib/events'; 

export default function EventsPage() { 
  const events = getEvents(); 
  return ( 
    <div id="events"> 
      <h2>Browse available events</h2> 
      <ul> 
        {events.map((event) => ( 
          <li key={event.id}> 
            <img src={event.image} alt={event.title} /> 
            <div> 
              <h2>{event.title}</h2> 
              <p>{event.description}</p> 
              <p> 
                <Link href={`/events/${event.id}`}>Explore Event</Link> 
              </p> 
            </div> 
          </li> 
        ))} 
      </ul> 
    </div> 
  ); 
}
```

Hacer clic en estos enlaces llevará a los usuarios a la página de detalles del evento para el ID de evento específico.

> [!NOTE]
> Las rutas estáticas, las rutas dinámicas y las rutas anidadas son los tipos de rutas más importantes que necesitas conocer cuando trabajas con Next.js. Las usarás para la mayoría de tus rutas.
> Además, Next.js también ofrece otros tipos de rutas y características (más avanzadas y específicas) que vale la pena explorar si decides profundizar en Next.js: [https://nextjs.org/docs/app/building-your-application/routing](https://nextjs.org/docs/app/building-your-application/routing).

Además de los diferentes tipos de rutas que se habilitan mediante el uso de los nombres de carpeta adecuados, Next.js también ofrece nombres de archivo reservados adicionales.

#### Otras convenciones de nombres de archivo
Next.js no solo ofrece una variedad de tipos de rutas y características relacionadas con el enrutamiento: también ofrece más nombres de archivos reservados además de `page.js` y `layout.js`.

Por lo tanto, al trabajar con Next.js App Router, también debes tener en cuenta que existen los siguientes nombres de archivo reservados:
- Los archivos **`loading.js`** se pueden agregar junto a los archivos `page.js` y `layout.js` o encima de ellos para definir componentes que deben mostrarse mientras el componente de página (o diseño) está obteniendo datos.
- Los archivos **`error.js`** se pueden agregar en los mismos lugares que los archivos `loading.js` para renderizar componentes de respaldo de error en caso de que una de las páginas hermanas o secundarias lance un error.
- Los archivos **`not-found.js`** se pueden agregar para mostrar contenido alternativo en caso de que un visitante del sitio web intente cargar una ruta o recurso inexistente.
- Los archivos **`route.js`** se pueden agregar para configurar rutas que no renderizan componentes sino que devuelven datos (por ejemplo, en formato JSON).

Puedes obtener más información sobre estos tipos de archivos y aún más convenciones de nombres de archivos en la documentación oficial: [https://nextjs.org/docs/app/building-your-application/routing#file-conventions](https://nextjs.org/docs/app/building-your-application/routing#file-conventions).

También verás algunos de estos tipos de archivos en acción en el próximo capítulo.

---

### Sección 6: Profundizando en Next.js

En este punto, tienes una base sólida de Next.js pero, como se mencionó en la sección anterior, puedes profundizar en Next.js con la ayuda de la documentación oficial.

Allí, además de aprender más sobre enrutamiento, tipos de rutas y nombres de archivos, también puedes explorar cómo Next.js ayuda con el almacenamiento en caché, los estilos o la gestión de metadatos de la página. Dado que este libro trata principalmente sobre React en sí, y no sobre Next.js, cubrir todos estos temas aquí sobrecargaría rápidamente este libro.

Es por eso que este capítulo se centró en establecer una base sólida de React SSR y Next.js. Los conceptos esenciales cubiertos a lo largo de este capítulo ayudarán a comprender las características más avanzadas de React y Next.js, como los React Server Components en el próximo capítulo. Además, gracias a estos fundamentos, también podrás aprender rápidamente más sobre Next.js con la ayuda de la documentación oficial o libros o cursos dedicados a Next.js.

---

### Sección 7: Resumen y puntos clave

- De forma predeterminada, las aplicaciones React basadas en Vite (como la mayoría de las aplicaciones React que no usan Next.js o un framework similar) solo admiten el renderizado del lado del cliente.
- Sin SSR, se envía un archivo `index.html` relativamente vacío al cliente.
- Esto puede provocar malas experiencias de usuario (si los usuarios ven una página vacía durante un período prolongado) o una clasificación subóptima en los motores de búsqueda.
- Puedes habilitar SSR ajustando manualmente los proyectos de React (código y proceso de compilación) para admitir la ejecución de funciones de componentes en el lado del servidor.
- Para evitar el trabajo de configuración manual de SSR y aprovechar muchos otros beneficios, puedes utilizar frameworks como **Next.js**.
- Los proyectos de Next.js vienen con soporte SSR integrado y se pueden crear mediante el comando `npx create-next-app`.
- El Next.js moderno utiliza el enfoque **App Router**, que aprovecha un directorio **`app/`** que se utiliza para configurar rutas con la ayuda del sistema de archivos.
- Dentro de `app/`, defines páginas creando carpetas que contienen archivos **`page.js`** (por ejemplo, `app/about/page.js` agrega soporte para una ruta `/about`).
- Para compartir código JSX (y lógica o estilos) entre páginas, puedes agregar archivos **`layout.js`**.
- Next.js también ofrece otros nombres de archivo reservados para manejar el contenido alternativo que se muestra mientras se cargan datos (`loading.js`) o para manejar errores (`error.js`).
- Puedes vincular páginas a través del componente **`Link`** de Next.js (`next/link`).
- Al usar Hooks de React (como `useState()`) o de Next.js (como `usePathname()`), debes agregar la directiva **`"use client"`** en la parte superior del archivo que usa el Hook.
- Además de las páginas estáticas (como `app/events/page.js` o `app/about/page.js`), también puedes configurar páginas dinámicas encerrando el nombre de una carpeta entre corchetes (por ejemplo, `app/events/[eventId]/page.js`).
- Los valores de los parámetros de ruta dinámicos se pueden extraer en el componente de página cargado mediante la prop especial **`params`** (usando `await params`) que Next.js establece en el componente.
- Las operaciones asíncronas pueden ser problemáticas al usar SSR clásico, o al menos no se pueden ejecutar en componentes que se renderizan en el servidor, lo que obliga al código del lado del cliente a realizarlas. Al menos cuando no se utilizan React Server Components.

---

### Sección 8: ¿Qué sigue?

En este punto, has aprendido mucho sobre SSR en aplicaciones React y sobre Next.js. Eres capaz de crear proyectos de Next.js, definir rutas, renderizar componentes de página, agregar navegación y trabajar con rutas dinámicas.

También aprendiste que Next.js viene con SSR integrado. Por lo tanto, todos los componentes de React (integrados y personalizados, de página y no de página) se renderizan en el servidor cuando un visitante del sitio web envía una solicitud.

Sin embargo, el Next.js moderno no se detiene allí: en lugar de la configuración clásica de SSR introducida al principio de este capítulo, los proyectos de Next.js que usan el App Router ayudan con la obtención asíncrona de datos en el lado del servidor al desbloquear la función **React Server Components** de React. ¡Son esa característica, y las **Server Actions**, las que se explorarán con gran detalle en el próximo capítulo!

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se encuentran en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/15-ssr-next-intro/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/15-ssr-next-intro/exercises/questions-answers.md):

1. ¿Qué dos ventajas principales puede ofrecer el SSR?
2. ¿Cuáles son algunas posibles desventajas o debilidades del SSR?
3. ¿Cómo ayuda Next.js con el SSR?
4. ¿Cómo se configuran las rutas en Next.js (al usar el "App Router")?
5. ¿Qué tiene de especial un componente de página en Next.js?
6. ¿Cuál es el propósito de los componentes de diseño (*layout*) en Next.js?
7. ¿Dónde puedes almacenar componentes de React que no sean de página (y que no sean de diseño) en un proyecto de Next.js?
8. ¿Cuándo y dónde necesitas agregar la directiva `"use client"`?

---

### Sección 10: Aplica lo aprendido

Con todo el conocimiento recién adquirido sobre Next.js, es hora de aplicarlo a un proyecto de demostración real: una aplicación de demostración que se renderizará en el servidor.

En la siguiente sección, encontrarás una actividad que te permitirá practicar el trabajo con Next.js. Como siempre, también necesitarás emplear algunos de los conceptos cubiertos en capítulos anteriores.

#### Actividad 15.1: Migración de una aplicación React Router basada en Vite
En esta actividad, tu trabajo consiste en aprovechar la aplicación basada en Vite de la Actividad 13.1. Esa aplicación se creó con Vite y React Router. Tu trabajo es migrarla de Vite y React Router a Next.js.

Por lo tanto, debes crear un nuevo proyecto de Next.js (usando el App Router) y reconstruir la misma aplicación en ese proyecto.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/activities/practice-1-start). Al descargar este código, siempre descargarás todo el repositorio. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start`, en este caso) para usar la instantánea de código correcta.
> Dado que tu tarea es migrar el proyecto que se creó en la Actividad 13.1, es posible que también desees utilizar el código terminado de esa actividad. Puedes encontrarlo aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/13-routing/activities/practice-1).

Después de descargar el código y ejecutar `npm install` en la carpeta del proyecto para instalar todas las dependencias requeridas, los pasos de la solución son los siguientes:
1. Si creaste un nuevo proyecto de Next.js (es decir, si no estás utilizando la instantánea inicial proporcionada), limpia los archivos `layout.js` y `page.js` para eliminar todo excepto las funciones de los componentes.
2. Crea dos nuevas rutas: una ruta `/products` y una ruta `/products/<algun-id>`.
3. Migra el archivo `data.js` al proyecto Next.js (por ejemplo, a una carpeta `lib/` en la carpeta raíz del proyecto).
4. Actualiza los componentes de la página para cargar y mostrar los datos proporcionados por el archivo `data.js`.
5. Crea una nueva carpeta `components/` y migra (copia) el componente `MainNavigation` a esta carpeta.
6. Actualiza el componente `MainNavigation` (y cualquier otro componente que lo necesite) para usar el componente `Link` de Next.js.
7. Resalta los enlaces activos con la ayuda del Hook `usePathname()`; ¡no olvides la directiva `"use client"`!
8. Migra los estilos del archivo `index.css` al archivo `globals.css`. Asegúrate de que el archivo se importe en el archivo de diseño raíz (`app/layout.js`).

El resultado esperado debería verse como se muestra en las siguientes capturas de pantalla:

**Figura 15.11**: El contenido de la página de inicio.

**Figura 15.12**: El contenido de la página /products.

**Figura 15.13**: El contenido de la página /products/<some-id>.

> [!NOTE]
> Encontrarás el código completo para esta actividad y una solución de ejemplo aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/15-ssr-next-intro/activities/practice-1).
