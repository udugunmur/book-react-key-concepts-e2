# Parte 2: Construcción de Aplicaciones React Complejas

## Capítulo 16: React Server Components y Server Actions

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Crear y utilizar Componentes de Servidor de React (*React Server Components* o RSC).
- Describir cómo (y cuándo) se ejecutan y renderizan en la pantalla los RSC.
- Obtener datos y realizar operaciones asíncronas con la ayuda de los RSC.
- Trazar límites entre cliente y servidor mediante la creación y el uso de componentes de cliente (*client components*).
- Realizar mutaciones de datos del lado del servidor con la ayuda de las Server Actions.
- Actualizar la interfaz de usuario (UI) en respuesta a las Server Actions.

---

### Sección 1: Introducción

En el capítulo anterior, aprendiste que puedes usar el renderizado en el servidor (*Server-Side Rendering* o SSR) para renderizar componentes de React en el servidor. SSR garantiza que los usuarios reciban un documento HTML completamente poblado en su solicitud HTTP inicial, no una estructura de página casi vacía. También te presentaron **Next.js** y aprendiste cómo puedes usar ese framework para crear aplicaciones React que vienen con SSR (y muchas otras características útiles) de fábrica.

Este capítulo se basa en el anterior; específicamente, aprenderás sobre dos características cruciales de React que desbloquea Next.js: **React Server Components (RSCs)** y **Server Actions**.

A lo largo de este capítulo, aprenderás cómo estas dos características ayudan con la obtención y mutación de datos, y por qué no puedes usarlas en todos los proyectos de React, aunque técnicamente sean parte de React y no de Next.js.

> [!NOTE]
> Los RSCs y las Server Actions son características relativamente nuevas de React. Soportarlas en proyectos personalizados de React es complejo, como aprenderás a lo largo de este capítulo.
> Aunque es poco probable, es posible que los conceptos o características relacionados con los RSCs o las Server Actions cambien. También es posible que el soporte para estas características en proyectos personalizados sea más fácil.
> Es por eso que este libro incluye un documento dedicado que rastrea cualquier cambio significativo que debas tener en cuenta: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/main/CHANGELOG.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/main/CHANGELOG.md).

---

### Sección 2: El problema con la obtención de datos en el servidor

Si tienes una aplicación React habilitada para SSR, ya sea habilitándola manualmente (por ejemplo, en un proyecto basado en Vite) o mediante un framework como Next.js, las funciones de tus componentes de React se ejecutan en el servidor. Por lo tanto, cualquier dato requerido por esos componentes debe obtenerse en el servidor.

Pero como se explicó en el capítulo anterior, en la sección *La obtención de datos en el lado del servidor no es trivial*, enviar solicitudes HTTP con la ayuda de `useEffect()` o intentar actualizar la interfaz de usuario mediante `useState()` no funciona cuando se utiliza SSR. En el servidor, React solo llama a las funciones de los componentes una vez: no las vuelve a ejecutar cuando cambia el estado. Tampoco llama a tus funciones de efecto.

Esta es una limitación seria, ya que muchas aplicaciones React necesitan obtener datos de algún backend o base de datos. No poder obtener y renderizar esos datos en el servidor significa que los visitantes del sitio web volverán a recibir documentos HTML incompletos (y esperarán a que los datos se obtengan en el lado del cliente), y los rastreadores de los motores de búsqueda no verán el contenido más importante de la página web.

Esa es una de las razones por las que React introdujo los **RSCs**.

---

### Sección 3: Introducción a los RSC

Los **RSC**, a pesar de su nombre, no son necesariamente componentes que se ejecutan en un servidor. En cambio, su característica definitoria es que **¡las funciones de sus componentes nunca, bajo ninguna circunstancia, se ejecutan en el lado del cliente!**

En consecuencia, los RSC pueden ejecutarse en un servidor, pero también pueden llamarse durante el proceso de compilación, pregenerando componentes en tiempo de compilación (*build time*). Sin embargo, definitivamente no se ejecutarán en el navegador.

**Figura 16.1**: Los RSCs no se pueden llamar desde el lado del cliente.

¿Pero cuál es el propósito de los RSC? ¿Cómo se crean y se usan?

#### Entendiendo los RSC
La idea central detrás de los RSC es que puedes crear componentes que se renderizan fuera del navegador (por ejemplo, en el servidor). Como resultado, estos componentes pueden ejecutar código que no funcionaría en el navegador, por ejemplo, porque se utilizan APIs específicas de Node.js, o código que depende de credenciales (como credenciales de base de datos) que no deben exponerse al cliente.

A diferencia de los componentes "normales" (componentes de cliente) que se renderizan a través de SSR, los RSC se pueden renderizar (en el servidor) después de la carga inicial de la página. Por lo tanto, los RSC no se limitan a renderizar una instantánea inicial de la página. Además, los RSC pueden obtener datos en el lado del servidor. Más adelante en este capítulo, la sección *RSCs frente al Renderizado en el Servidor (SSR)* examinará más de cerca la relación entre los RSC y los componentes "normales" renderizados mediante SSR.

Por lo tanto, los RSC resuelven un problema importante: **te permiten entrelazar el código React del frontend y del backend**. Mientras que en el pasado, antes de los RSC, normalmente tenías que crear aplicaciones web de backend y frontend separadas, ahora puedes crear aplicaciones *fullstack* integradas que combinan código React del lado del servidor y del lado del cliente.

El uso de RSC ofrece, por lo tanto, varias ventajas:
- Crear aplicaciones *fullstack* totalmente integradas donde el backend y el frontend están estrechamente conectados y utilizan el mismo servidor se vuelve mucho más fácil.
- **La obtención asíncrona de datos del lado del servidor dentro de los componentes se vuelve posible**: a diferencia del lado del cliente (o cuando se usa SSR), React te permite usar `async`/`await` y devolver un valor de tipo `Promise` en las funciones de tus componentes.
- Los visitantes del sitio web descargan paquetes de JavaScript del lado del cliente más pequeños, ya que el código de los RSC se omite.
- Ejecutar operaciones computacionalmente pesadas o usar grandes bibliotecas de terceros se vuelve más fácil, ya que las operaciones y su código se pueden externalizar al servidor (o al proceso de compilación).
- El código o las credenciales que no deberían ser accesibles para los usuarios de tu sitio web se pueden mover a los RSC.

Por ejemplo, gracias a los RSC, puedes crear componentes como este:

```javascript
import pg from 'pg'; // pg package (more info: node-postgres.com) 
const { Client } = pg 
const client = new Client({ 
  user: 'username', 
  password: 'your-password', 
  host: 'my.database-server.com', 
  port: 5334, 
  database: 'demo', 
}); 

async function ProductsPage() { 
  await client.connect(); 
  const res = await client.query('SELECT * FROM products'); 
  await client.end(); 
  return ( 
    <ul> 
      {res.rows.map(row => <li key={row.id}>{row.title}</li>)} 
    </ul> 
  ); 
}
```

El componente `ProductsPage` contiene código que se conecta a una base de datos PostgreSQL para obtener datos de productos desde allí.

Sin los RSC, este tipo de componente sería imposible de construir y utilizar: no se te permitiría usar `async`/`await`, el paquete `pg` podría depender de algunas APIs que no están disponibles en el navegador y expondrías las credenciales de tu base de datos en el paquete de código del lado del cliente.

Todas estas cosas están permitidas al compilar RSCs. React explícitamente te permite devolver una Promesa (y por lo tanto usar `async`/`await`) al construir un RSC. Dado que se garantiza que el código nunca terminará en el lado del cliente, conectarse a una base de datos también es seguro.

Por lo tanto, puedes crear fácilmente aplicaciones *fullstack* totalmente integradas donde el código backend y frontend se combinan a la perfección.

Sin embargo, el uso de RSCs es simple y complejo al mismo tiempo, como explicará la siguiente sección.

#### Creación y uso de RSCs
En un proyecto de Next.js que utiliza el App Router, **todos los componentes de React, sin importar si se usan como páginas o están anidados en algún otro componente, son, por defecto, RSCs**.

Como puedes comprobar si inspeccionas cualquier función de componente de React en un proyecto Next.js, realmente no tienen nada de especial. Parecen componentes de React normales:

```javascript
export default function ServerComponentInfo() { 
  return <p>This is a React Server Component.</p>; 
}
```

Puedes usar `async`/`await` con ellos, pero no es obligatorio. Puedes usar APIs y paquetes del lado del servidor, pero no es obligatorio. Por lo tanto, crear RSCs es simple: después de todo, son simplemente componentes normales.

Lo mismo ocurre con su uso: los usas como siempre has usado los componentes de React, como elementos JSX personalizados:

```javascript
<ServerComponentInfo />
```

Como puedes ver, no podrías decir a simple vista que se trata de un tipo especial de componente. Se crea y se usa como aprendiste a lo largo de todo este libro.

No obstante, todos los demás componentes de todos los demás capítulos de este libro, que se utilizaron en proyectos de React basados en Vite, no eran RSCs: eran componentes regulares o componentes de cliente.

Entonces, ¿qué hace que los componentes en un proyecto Next.js sean especiales? ¿Por qué una característica proporcionada por React está disponible en proyectos Next.js pero no necesariamente en otros proyectos de React (por ejemplo, en proyectos basados en Vite)?

#### Desbloqueando los RSC en proyectos React
Los RSCs son una característica proporcionada por React, no por Next.js. Sin embargo, no todos los proyectos de React pueden usar esta característica.

La razón de esto, y de por qué los RSC están disponibles en los proyectos de Next.js, es el proceso de compilación de Next.js y lo que Next.js hace con estos componentes (y con todo el código del proyecto de React, en realidad) detrás de escena. A un nivel alto, puedes pensar que Next.js hace las siguientes cosas:
1. El flujo de trabajo de compilación y el proceso de empaquetado (*bundling*) **separan los componentes del servidor y del cliente** para garantizar que ningún código de RSC termine en el lado del cliente.
2. Next.js configura **puntos finales de API (*endpoints*)** (es decir, direcciones URL a las que el código del lado del cliente puede enviar solicitudes) que activan las funciones de los componentes RSC en el servidor y devuelven instrucciones que permiten que el código React del lado del cliente actualice la interfaz de usuario.
3. Next.js llama a estos endpoints cuando es necesario, por ejemplo, al navegar a una nueva página.
4. Next.js pasa la respuesta de la API (que contiene estas instrucciones de renderizado) a React, que utiliza las instrucciones devueltas para actualizar la interfaz de usuario según sea necesario.

**Figura 16.2**: El código del componente cliente y servidor está separado; la comunicación ocurre a través de solicitudes HTTP.

Técnicamente, es un poco más complejo que eso, pero para el propósito de este libro y para usar la función, no se requiere una comprensión profunda de los aspectos internos, del mismo modo que no necesitas comprender qué sucede exactamente internamente al usar `useState()`, por ejemplo.

Puedes verificar los puntos mencionados ejecutando un proyecto de demostración de Next.js que se encuentra aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/16-rsc-server-actions/examples/01-rsc-intro](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/16-rsc-server-actions/examples/01-rsc-intro).

Esta aplicación de demostración consta de dos archivos de componentes de página básicos: `app/page.js` y `app/info/page.js`. El componente de la página principal (el componente `Home` dentro de `app/page.js`) genera un componente `ServerComponentInfo`:

```javascript
import ServerComponentInfo from '../components/ServerComponentInfo'; 

export default function Home() { 
  return <ServerComponentInfo />; 
}
```

Ese componente, a su vez, simplemente muestra contenido estático codificado:

```javascript
import Link from 'next/link'; 

export default function ServerComponentInfo() { 
  return ( 
    <div id="rsc-info"> 
      <p>This is a React Server Component.</p> 
      <p><Link href="/info">Learn More</Link></p> 
    </div> 
  ); 
}
```

Tanto el componente `Home` como el `ServerComponentInfo` son RSCs, simplemente porque son componentes en un proyecto Next.js. Como se mencionó anteriormente, todos los componentes en los proyectos Next.js son componentes del servidor por defecto. Si estos componentes fueran parte de un proyecto de React basado en Vite que no está configurado para admitir RSCs, estos componentes serían en cambio componentes "normales" (componentes de cliente).

En el mismo proyecto de demostración, también hay un componente para la página `/info`. Este componente contiene código que no funcionaría en un componente que no sea un RSC:

```javascript
import fs from 'node:fs/promises'; 

export default async function InfoPage() { 
  const info = await fs.readFile('data/rsc-info.json', 'utf-8'); 
  const { summary } = JSON.parse(info); 
  return ( 
    <div id="info-page"> 
      <h1>Understanding React Server Components</h1> 
      <p> 
        {summary} 
      </p> 
    </div> 
  ); 
}
```

Este código no funcionaría en ninguno de los proyectos de React (basados en Vite) que viste antes en este libro por las siguientes razones:
- El componente `InfoPage` utiliza el paquete `fs` de Node para cargar datos de un archivo `rsc-info.json` (que forma parte del proyecto).
- El componente usa `async`/`await`, por lo que devuelve una Promesa que finalmente produce el código JSX (es decir, los elementos de React).

En proyectos que no admiten RSCs, no puedes usar APIs del lado del servidor ya que todo el código se ejecuta en el navegador. Tampoco puedes devolver una Promesa en tus componentes; en componentes que no son RSC, eso no se consideraría un valor de retorno de función de componente válido. Sin embargo, cuando se trabaja con RSCs, ambas cosas están permitidas y son posibles.

Como se mencionó en la sección *Entendiendo los RSC*, el uso de funciones del lado del servidor (como las APIs de Node.js) es algo que se desbloquea porque el componente `InfoPage`, como todos los componentes en los proyectos de Next.js, es un RSC. Para los RSCs, React también admite el uso de `async`/`await`.

En consecuencia, como se esperaba, no encontrarás el código del componente `InfoPage` en los paquetes de código JavaScript del lado del cliente. Puedes verificar esto visitando la página `/info`. Si abres la pestaña Red (*Network*) en las herramientas de desarrollo del navegador y luego recargas la página, verás todas las solicitudes HTTP enviadas al servidor. Esto incluye todas las solicitudes de archivos de código JavaScript que se necesitan en el lado del cliente de esta aplicación React.

**Figura 16.3**: Al visitar /info, se envían solicitudes de CSS, JS y algunos otros archivos al servidor.

Si luego revisas todos los archivos JavaScript solicitados y buscas `rsc-info.json` en los archivos de código descargados, no tendrás ninguna coincidencia en ningún archivo. Esto demuestra que este código, que forma parte de la función del componente `InfoPage`, no termina en ningún paquete de código del lado del cliente.

**Figura 16.4**: El nombre de archivo de la fuente de datos que se incluye en el código RSC no se puede encontrar en el lado del cliente.

¿Cómo aparece entonces en la pantalla el contenido obtenido del archivo `rsc-info.json`?

Esto se responde si utilizas un navegador diferente a Chrome o Edge. Esto es necesario debido a un error en la pestaña Red en las herramientas de desarrollo de Chrome/Edge que hace que la respuesta de una solicitud se oculte bajo ciertas circunstancias.

En su lugar, puedes, por ejemplo, utilizar Firefox para visitar la página raíz (`/`). Allí, haz clic en el enlace visible en la página para navegar a la página `/info`. Al hacerlo, se enviará una nueva solicitud HTTP. Si inspeccionas esa solicitud y su respuesta (en las herramientas de desarrollo del navegador Firefox), verás las **instrucciones RSC serializadas** que devuelve el servidor.

**Figura 16.5**: Se reciben instrucciones serializadas para React del lado del cliente desde el servidor.

Como puedes ver, la respuesta no es contenido HTML. En cambio, es un conjunto de instrucciones serializadas que React traduce a elementos DOM en el lado del cliente (es decir, en el navegador).

Por lo tanto, como puedes ver, crear y usar RSCs es simple, pero preparar el proyecto para manejarlos no lo es. En su lugar, necesitas un proceso de compilación que separe el código del cliente y del servidor, y endpoints de API que invoquen las funciones de los componentes del servidor en el servidor. También necesitas código del lado del cliente que envíe solicitudes a esos endpoints de API cada vez que se deban renderizar los componentes del servidor.

#### Los RSC y las Server Actions no se pueden utilizar en todos los proyectos
Hasta ahora en este libro, cada vez que se introducía una nueva característica de React, simplemente podías usarla en tu proyecto de React, sin importar si era un proyecto creado y administrado por Vite o cualquier otra herramienta (por ejemplo, `create-react-app`).

Con los RSCs y las Server Actions, esto cambia. Debido a las muchas cosas que se deben hacer detrás de escena (consulta la sección anterior), aunque estas son características proporcionadas por React, no puedes simplemente comenzar a usarlas en cualquier proyecto de React. En su lugar, para desbloquear estas funciones, debes tener un proyecto configurado para admitirlas.

Como resultado, en el momento en que se escribe este libro, los RSCs y las Server Actions realmente **solo se pueden usar con la ayuda de frameworks que integran y admiten activamente estas funciones**, por ejemplo, el framework Next.js.

Por supuesto, técnicamente es posible configurar un proyecto que admita ambas funciones por tu cuenta, pero requiere conocimientos avanzados sobre desarrollo backend y configuración del flujo de trabajo de compilación. En consecuencia, la mayoría de los proyectos de React que necesitan estas funciones dependen de frameworks como Next.js. Dado que la forma en que trabajas con los RSCs y las Server Actions siempre será la misma, sin importar en qué tipo de proyecto los uses, este libro ignorará la parte de configuración personalizada y se centrará en cómo usar estos dos conceptos centrales.

#### RSCs frente al Renderizado en el Servidor (SSR)
A primera vista, el uso de RSCs puede parecer similar a los componentes de React con SSR. Después de todo, ambos conceptos tratan sobre ejecutar código fuera del navegador.

Pero aunque los conceptos suenen similares, son bastante diferentes:
- **SSR** se trata de renderizar un árbol de componentes a HTML cuando se recibe una solicitud. Se trata de crear una **instantánea inicial de la página**, al final. Además, al crear una aplicación web interactiva, una parte vital de SSR es que la instantánea HTML pre-renderizada se **hidrata** en el lado del cliente (consulta el Capítulo 15 y la Figura 15.3). Como resultado, cuando se usa SSR, todo el árbol de componentes con todas sus funciones de componentes se evalúa en el lado del servidor y también en el lado del cliente: no hay división entre el código del lado del servidor y del lado del cliente. Por esa razón, tampoco puedes tener ningún código exclusivo del servidor en tus componentes de React con SSR clásico.
- Con los **RSCs**, eso cambia. El código de las funciones de sus componentes, como se explicó en las secciones anteriores, **nunca termina en el lado del cliente**.

**Figura 16.6**: Los RSCs no se hidratan; en su lugar, su salida se solicita mediante solicitudes HTTP.

Es por eso que un proyecto habilitado para SSR no admite automáticamente los RSCs. Por otro lado, podrías configurar un proyecto que admita RSCs pero que también use SSR para algunos componentes: componentes que deben pre-renderizarse en el servidor pero que también se necesitan en el lado del cliente (por ejemplo, porque agregan interactividad a la página). Este tipo de componentes se explorará en la siguiente sección.

También vale la pena señalar que los RSCs, al igual que los componentes renderizados en el servidor en proyectos SSR, solo se ejecutan una vez por solicitud. Sin embargo, los RSCs, a diferencia de los componentes "normales" renderizados mediante SSR, se pueden ejecutar bajo demanda mientras se ejecuta la aplicación; no se limitan a ser llamados para crear una instantánea inicial de la página.

Sin embargo, hay una pregunta importante: ¿cómo se puede agregar interactividad y, por ejemplo, manejar la entrada del usuario, en aplicaciones de React donde todos los componentes se renderizan en el servidor? Después de todo, la interacción del usuario tiene lugar en el navegador.

#### RSCs frente a Client Components
Los RSCs ofrecen algunas ventajas convincentes (consulta la sección *Entendiendo los RSC*), pero también introducen un problema potencialmente grande: si todo el código del componente "vive" y se ejecuta en el servidor, no hay lugar para la interactividad del lado del cliente.

#### No todos los componentes deberían ser RSCs
Si tienes un componente que necesita administrar algún estado (por ejemplo, un carrito de compras que solo debería mostrarse tras la interacción del usuario), ese estado y la interfaz de usuario deben ser administrados y actualizados por React del lado del cliente. Porque ese era (y es) uno de los principales puntos de venta de React: puedes usarlo para crear interfaces de usuario altamente reactivas e interactivas. Pero este objetivo choca claramente con la idea de los RSCs, donde ningún código de componente llega al navegador y donde los componentes solo se renderizan una vez por solicitud.

Es por eso que React te permite definir los llamados **límites servidor-cliente (*server-client boundaries*)** agregando la directiva **`'use client'`** en la parte superior de los archivos que contienen funciones de componentes que deben ejecutarse en el lado del cliente.

**Figura 16.7**: La directiva 'use client' crea un límite entre el código del lado del servidor y del lado del cliente.

Ya encontraste `'use client'` en el capítulo anterior, en la sección *Resaltar enlaces activos y uso de la directiva "use client"*. En aquel entonces, esta directiva no tenía mucho sentido. Ahora, con tu conocimiento recién adquirido sobre los RSCs, el propósito detrás de esta directiva se volverá más claro.

Con `'use client'` agregado a un archivo de componente, los componentes definidos en ese archivo se convierten en **componentes de cliente (*client components*)**. Los componentes de cliente también se pre-renderizan en el servidor, pero su código también se ejecuta en el lado del cliente: se hidratan, como se explicó en el capítulo anterior. Por lo tanto, a diferencia del código de los componentes de servidor, el código de los componentes de cliente llega al lado del cliente:

```javascript
'use client'; 
import { useState } from 'react'; 

export default function Cart() { 
  const [isVisible, setIsVisible] = useState(false); 

  function handleCartVisibility() { 
    setIsVisible((prevState) => !prevState); 
  } 

  return ( 
    <div id="cart"> 
      <button onClick={handleCartVisibility}> 
        {isVisible ? 'Hide Cart' : 'Show Cart'} 
      </button> 
      {isVisible && <p>Cart Items</p>} 
    </div> 
  ); 
}
```

En este ejemplo, el componente `Cart` es un componente de cliente porque se agrega `'use client'` en la parte superior del archivo. Esto es necesario porque el componente `Cart` usa el Hook `useState()`, que solo funciona en el navegador.

Siempre que agregas la directiva `'use client'` a un archivo de componente, las funciones de los componentes en ese archivo se incluirán en el paquete de código del lado del cliente. Por lo tanto, las funciones de los componentes se pueden (y se van a) ejecutar en el navegador; por lo tanto, puedes usar funciones que dependen de ejecutarse allí, como `useState()` o código que debe ejecutarse tras la entrada del usuario (por ejemplo, si se presionó un `<button>`).

Es por eso que Next.js muestra un error si intentas usar un Hook en un componente que no está marcado como un componente de cliente mediante `'use client'`.

**Figura 16.8**: Next.js se queja del uso del Hook useState() en un RSC.

Este error ocurre porque estás intentando construir algo imposible: un componente que solo se evalúa en el servidor pero que también reacciona a la entrada del usuario y actualiza algún estado. Dado que esto último, como aprendiste en el Capítulo 4, *Trabajando con Eventos y Estado*, normalmente dará como resultado una actualización de la interfaz de usuario, el código debe ejecutarse en el lado del cliente, algo que claramente está en conflicto con el objetivo de ejecutar el código del componente solo en el servidor.

Por lo tanto, **se debe agregar `'use client'` siempre que tengas un componente que deba ejecutarse en el navegador**.

> [!NOTE]
> Por supuesto, no necesitarás agregar la directiva `'use client'` en proyectos que no implementen RSCs. Es por eso que no lo viste en ningún otro proyecto de React en capítulos anteriores.

#### ¡`'use client'` también afecta a los componentes secundarios!
El uso de la directiva `'use client'` en un archivo de componente tiene una implicación muy importante: **todos los componentes anidados se convierten en componentes de cliente también**, incluso si no usas `'use client'` en sus archivos de componentes.

Esto es técnicamente necesario ya que el código JSX de los componentes de cliente se vuelve a evaluar, y todos los componentes personalizados utilizados allí se vuelven a ejecutar cada vez que se llama nuevamente a la función del componente de cliente (por ejemplo, debido a algún cambio de estado); eso es algo que aprendiste en el Capítulo 10, *Detrás de Escena de React y Oportunidades de Optimización*.

Como resultado, todos los componentes anidados dentro de un componente de cliente deben ser componentes de cliente ellos mismos, ya que su código de lo contrario no estaría disponible en el lado del cliente.

**Figura 16.9**: Los componentes secundarios de los componentes de cliente también se convierten en componentes de cliente.

Para mantener el paquete de código del cliente pequeño y con un rendimiento óptimo, generalmente es una buena idea **maximizar el número de componentes de servidor y, por lo tanto, minimizar el número de componentes de cliente**. Dado que los componentes anidados de los componentes de cliente se convierten en componentes de cliente automáticamente, debes intentar mover el límite servidor-cliente (es decir, el uso de `'use client'`) **lo más abajo posible en el árbol de componentes**. Idealmente, solo las hojas de tu árbol de componentes usan Hooks de React o manejan la entrada del usuario. Dicho de otra manera: solo usa `'use client'` cuando sea necesario e intenta afectar la menor cantidad de componentes posible con él.

**Figura 16.10**: La mayoría de los componentes son RSCs.

La Figura 16.10 muestra un árbol de componentes de ejemplo donde solo un pequeño subconjunto de todos los componentes son componentes de cliente.

La pregunta por lo tanto es: ¿cómo se puede combinar y optimizar el uso de componentes de servidor y de cliente en proyectos de React que admiten RSCs?

#### Combinación de RSCs y Client Components
Normalmente, terminarás con proyectos de React donde la mayoría de los componentes no necesitan ser componentes de cliente (por lo tanto, deberían ser RSCs), pero donde algunas funciones de componentes sí necesitan ejecutarse en el navegador (es decir, sí necesitan `'use client'`).

Puedes pensar en `'use client'` como el punto en el árbol de componentes donde el tipo de componente cambia de componente de servidor a componente de cliente (consulta la Figura 16.9 y la Figura 16.10).

Por esa razón, React te permite combinar ambos tipos de componentes en el mismo proyecto, aunque debes seguir un par de reglas importantes:
1. Los **componentes de servidor pueden importar y renderizar componentes de cliente** (es decir, emitir un componente de cliente en su código JSX).
2. Los **componentes de cliente no deben importar y renderizar directamente componentes de servidor** que dependan de características del lado del servidor.
3. Los **componentes de cliente pueden renderizar implícitamente componentes de servidor a través de props** (por ejemplo, a través de la prop `children`).

Para hacer estas reglas un poco menos abstractas, cada caso se mostrará con un ejemplo concreto.

#### Renderizar Client Components dentro de Server Components
Puedes usar componentes de cliente en el código JSX de los componentes de servidor sin problemas.

Considera el siguiente componente de ejemplo `UserTodos`, que permite a los usuarios administrar un array de tareas pendientes que se almacena localmente a través de `localStorage`:

```javascript
'use client'; 
import { useEffect, useRef, useState } from 'react'; 

export default function UserTodos() { 
  const todoRef = useRef(null); 
  const [todos, setTodos] = useState([]); 

  useEffect(() => { 
    const storedTodos = localStorage.getItem('todos'); 
    setTodos(storedTodos ? JSON.parse(storedTodos) : []); 
  }, []); 

  function handleAddTodo(event) { 
    event.preventDefault(); 
    const todo = todoRef.current.value.trim(); 
    const newTodo = { 
      id: new Date().getTime(), 
      text: todo, 
    }; 
    setTodos((prevTodos) => [...prevTodos, newTodo]); 
    const storedTodos = localStorage.getItem('todos'); 
    localStorage.setItem( 
      'todos', 
      JSON.stringify( 
        storedTodos ? [...JSON.parse(storedTodos), newTodo] : [newTodo] 
      ) 
    ); 
  } 

  return ( 
    <> 
      <form onSubmit={handleAddTodo}> 
        <input type="text" placeholder="Your to-do" ref={todoRef} /> 
        <button type="submit">Add</button> 
      </form> 
      <ul> 
        {todos.map((todo) => ( 
          <li key={todo.id}>{todo.text}</li> 
        ))} 
      </ul> 
    </> 
  ); 
}
```

Dado que se utilizan `localStorage` (una API del navegador), refs, estado (`todos` a través de `useState()`) y escuchadores de eventos (`submit` a través de `onSubmit`), este debe ser un componente de cliente. Es por eso que se agrega `'use client'` en la parte superior del archivo.

Sin embargo, este componente se puede utilizar en un componente de servidor sin problemas:

```javascript
import UserTodos from '../components/UserTodos'; 

export default function Home() { 
  return ( 
    <main> 
      <h1>Manage your to-dos with ease!</h1> 
      <UserTodos /> 
    </main> 
  ); 
}
```

Eso es posible porque los componentes de cliente también se pueden renderizar en el servidor; simplemente no son exclusivos de ese entorno (a diferencia de los RSCs, que sí lo son). Dicho de otra manera: los componentes de cliente se renderizan en el servidor como se renderizaban todos los componentes en proyectos SSR que no admiten RSCs. Se renderiza una instantánea inicial en la primera solicitud y, a partir de entonces, React del lado del cliente toma el control e hidrata el componente.

> [!NOTE]
> En el ejemplo anterior, los datos se cargan desde `localStorage` a través de `useEffect()`. Esto se hace para garantizar que el código se ejecute en el cliente. Dado que `localStorage` no está disponible en el servidor, acceder a él sin envolverlo con `useEffect()` causaría un error.
> Dado que `useEffect()` se ignora en el servidor, es una forma segura de utilizar APIs exclusivas del navegador.

#### Renderizar Server Components dentro de Client Components
Como ya se mencionó en la sección *¡`'use client'` también afecta a los componentes secundarios!*, no puedes importar componentes de servidor dentro de componentes de cliente y renderizarlos allí si dependen de funciones exclusivas del servidor.

Sin embargo, en muchas situaciones, no obtendrás ningún error. Por ejemplo, podrías tener un componente `Cart` del lado del cliente definido de la siguiente manera:

```javascript
'use client'; 
import { useState } from 'react'; 
import CartItem from './CartItem'; 

export default function Cart() { 
  const [isVisible, setIsVisible] = useState(false); 

  function handleCartVisibility() { 
    setIsVisible((prevState) => !prevState); 
  } 

  return ( 
    <div id="cart"> 
      <button onClick={handleCartVisibility}> 
        {isVisible ? 'Hide Cart' : 'Show Cart'} 
      </button> 
      {isVisible && ( 
        <ul> 
          <CartItem title={'Book'} quantity={1} /> 
          <CartItem title={'Pen'} quantity={2} /> 
          <CartItem title={'Pencil'} quantity={5} /> 
        </ul> 
      )} 
    </div> 
  ); 
}
```

A diferencia de `Cart`, la función del componente `CartItem` podría ser un componente de servidor (es decir, no está marcado mediante `'use client'`):

```javascript
export default function CartItem({ title, quantity }) { 
  return ( 
    <li> 
      <article> 
        <h2>{title}</h2> 
        <p>Quantity: {quantity}</p> 
      </article> 
    </li> 
  ); 
}
```

Este código funciona porque el componente que solía ser un componente de servidor (`CartItem`) **simplemente se convierte en un componente de cliente** una vez que se importa y se usa en un archivo de componente de cliente.

Sin embargo, te encontrarás con un mensaje de error si intentas importar y usar un componente de servidor que utiliza características específicas de componentes de servidor, como una API de Node.js o `async`/`await`.

Por ejemplo, el siguiente componente `DynamicCartItem` ajustado intenta utilizar el paquete `fs` de Node para cargar un elemento del carrito desde un archivo:

```javascript
import fs from 'node:fs/promises'; 

export default async function DyncamicCartItem({ id }) { 
  const data = await fs.readFile(`data/cart.json`, 'utf8'); 
  const storedCart = JSON.parse(data); 
  const cartItem = storedCart.find((item) => item.id === id); 
  return ( 
    <li> 
      <article> 
        <h2>{cartItem.title}</h2> 
        <p>Quantity: {cartItem.quantity}</p> 
      </article> 
    </li> 
  ); 
}
```

Importar y usar este componente en el componente `Cart` causará un error.

Intentar ejecutar este código provocará que se muestre un mensaje de error en la pantalla porque React no puede convertir automáticamente `DynamicCartItem` en un componente de cliente (debido al uso de funciones exclusivas de RSC). Por lo tanto, se quejará de algún código del lado del servidor (por ejemplo, alguna API de Node.js) que estás intentando usar en el lado del cliente.

**Figura 16.11**: React se queja del uso de una API de Node.js en el navegador.

Por lo tanto, en situaciones como esta, deberás reestructurar tu aplicación para terminar con una combinación de componentes válida nuevamente. Por ejemplo, **pasando componentes de servidor como props a componentes de cliente**, en lugar de importarlos y renderizarlos directamente.

#### Renderizado de Server Components a través de props
No puedes importar y usar componentes de servidor que realicen alguna operación exclusiva del lado del servidor (como usar APIs de Node.js) en componentes de cliente.

Pero puedes cambiar el código de tu componente de cliente para que no importe ni use directamente el componente de servidor. En su lugar, puedes esperar recibir un componente de servidor como una prop, por ejemplo, a través de la prop especial **`children`** sobre la que aprendiste en el Capítulo 3, *Componentes y Props*:

```javascript
'use client'; 
import { useState } from 'react'; 

export default function Cart({ children }) { 
  const [isVisible, setIsVisible] = useState(false); 

  function handleCartVisibility() { 
    setIsVisible((prevState) => !prevState); 
  } 

  return ( 
    <div id="cart"> 
      <button onClick={handleCartVisibility}> 
        {isVisible ? 'Hide Cart' : 'Show Cart'} 
      </button> 
      {isVisible && <ul>{children}</ul>} 
    </div> 
  ); 
}
```

Este componente `Cart` ajustado sigue siendo un componente de cliente. Sin embargo, dado que ya no importa ni renderiza directamente el componente de servidor `DynamicCartItem`, React no tiene problemas.

En su lugar, el componente `DynamicCartItem` ahora se importa y se genera en el componente `Home` de la siguiente manera:

```javascript
import DyncamicCartItem from '../components/DynamicCartItem'; 
import Cart from '../components/Cart'; 

export default function Home() { 
  return ( 
    <> 
      <header> 
        <Cart> 
          <DyncamicCartItem id={1} /> 
          <DyncamicCartItem id={2} /> 
          <DyncamicCartItem id={3} /> 
        </Cart> 
      </header> 
      <main> 
        <h1>Some dummy app</h1> 
      </main> 
    </> 
  ); 
}
```

Los elementos `DynamicCartItem` se pasan como valor para la prop `children` al componente `Cart`.

Esto puede resultar poco intuitivo al principio, pero es vital comprender que **esto funciona porque los componentes `DynamicCartItem` ahora se renderizan como parte de otro componente de servidor: el componente `Home`**. Es el resultado de ese proceso de renderizado lo que luego se pasa como valor al componente `Cart`. Por lo tanto, ese componente no incluye el componente `DynamicCartItem` en su parte del árbol de componentes. En su lugar, tanto `Cart` como `DynamicCartItem` son hijos directos del componente `Home`.

El árbol de componentes general de la aplicación se vería así:

**Figura 16.12**: DynamicCartItem y Cart son ambos componentes secundarios directos del componente Home.

Aunque, en la interfaz de usuario terminada, pueda parecer que `DynamicCartItem` es un hijo de `Cart`, técnicamente no lo es.

Es clave comprender que envolver un componente con otro componente (`<Cart><DynamicCartItem /></Cart>`) conduce a una estructura de árbol de componentes diferente a renderizar un componente dentro de otro componente.

Por lo tanto, este es un patrón que puede resultar útil en situaciones en las que podrías necesitar incluir un componente de servidor en un componente de cliente.

En general, puedes combinar RSCs y componentes de cliente según sea necesario. Además, Next.js también proporciona algunas características adicionales que pueden ayudar con los RSCs y la obtención de datos a través de RSCs.

#### Obtención avanzada de datos con Next.js
Como se mencionó anteriormente, en la sección *Entendiendo los RSC*, la obtención de datos a través de RSCs ofrece varias ventajas en comparación con la obtención de datos en componentes de cliente: no tienes que usar `useEffect()` para enviar solicitudes HTTP a APIs de backend separadas, puedes conectarte directamente a una base de datos, puedes usar `async`/`await`, etc. Por lo tanto, se recomienda absolutamente obtener datos a través de RSCs siempre que sea posible.

Cuando se trabaja con Next.js, la obtención de datos basada en RSC se vuelve aún más fácil porque Next.js ayuda a mostrar contenido alternativo mientras esperas que lleguen los datos.

#### Gestión de estados de carga con Next.js
Cuando trabajas con Next.js (con App Router), puedes definir archivos **`loading.js`** dentro de la carpeta `app/` para configurar componentes que se renderizarán mientras los componentes del servidor hermanos o anidados están cargando datos. Next.js determina si un componente está cargando datos o no comprobando si devuelve una Promesa que aún no se ha resuelto.

> [!NOTE]
> El próximo capítulo profundizará aún más en el manejo de los estados de carga y la visualización de contenido alternativo. Explorará la función `Suspense` de React, que permite una gestión granular del estado de carga a medida que los datos se transmiten en streaming.

Considera este componente de ejemplo `GoalsPage`, que obtiene datos de un archivo:

```javascript
import fs from 'node:fs/promises'; 

async function fetchGoals() { 
  await new Promise((resolve) => setTimeout(resolve, 3000)); // delay 
  const goals = await fs.readFile('./data/user-goals.json', 'utf-8'); 
  return JSON.parse(goals); 
} 

export default async function GoalsPage() { 
  const fetchedGoals = await fetchGoals(); 
  return ( 
    <> 
      <h1>Top User Goals</h1> 
      <ul> 
        {fetchedGoals.map((goal) => ( 
          <li key={goal}>{goal}</li> 
        ))} 
      </ul> 
    </> 
  ); 
}
```

La función (`fetchGoals()`) que realiza la obtención de datos real tiene un retraso incorporado para simular una conexión de red o base de datos lenta.

Sin un archivo `loading.js` agregado al proyecto, el usuario se quedará mirando una página en blanco o desactualizada durante un par de segundos antes de que se renderice la página solicitada.

**Figura 16.13**: Después de hacer clic en el enlace, la nueva página tarda tres segundos en cargarse.

Este comportamiento se produce porque la nueva página aún no está lista y no se puede renderizar porque todavía está obteniendo datos.

Para mejorar la experiencia del usuario, se puede agregar un archivo `loading.js` junto al archivo lento `app/goals/page.js` (o, si es necesario, en alguna carpeta principal, ya que `loading.js` también mostrará su contenido para las rutas secundarias).

Dentro del archivo recién creado `app/goals/loading.js`, se crea un componente de React normal. Al igual que todos los componentes en los proyectos de Next.js, este es un RSC por defecto:

```javascript
export default function LoadingGoals() { 
  return <p id="fallback">Loading user goals, please wait...</p>; 
}
```

El nombre del componente (`LoadingGoals`) no importa. Pero este componente ahora garantiza que el texto alternativo `Loading user goals, please wait...` se muestre en la pantalla mientras el usuario espera que `GoalsPage` se cargue y se renderice.

**Figura 16.14**: El contenido alternativo de carga se muestra mientras la página está en transición.

Por supuesto, puedes mostrar cualquier contenido alternativo de tu elección: no tiene por qué ser un texto simple como en este ejemplo.

Por lo tanto, cuando trabajas con Next.js, agregar archivos `loading.js` para definir componentes alternativos puede mejorar enormemente la experiencia de los usuarios de tu sitio web.

Además de obtener datos, muchas aplicaciones de React también necesitan cambiar datos en algún momento.

---

### Sección 4: De la obtención de datos a las mutaciones de datos

En este punto, has aprendido mucho sobre RSCs, componentes de cliente y cómo pueden (y no pueden) trabajar juntos. En la sección *Entendiendo los RSC*, también aprendiste sobre algunas ventajas que ofrecen los RSCs.

Por supuesto, es posible que también desees cambiar datos, no solo cargarlos y mostrarlos.

#### Manejo de mutaciones de datos con Server Actions
React no solo brinda soporte para RSCs: también te permite agregar las llamadas **Server Actions** a tus aplicaciones.

Las Server Actions se basan en la misma idea que las acciones de formulario del cliente (*client form actions*), que se introdujeron y explicaron en el Capítulo 9, *Manejo de Entradas de Usuario y Formularios con Form Actions*. Sin embargo, las Server Actions, como su nombre lo indica, **se ejecutarán en el lado del servidor**, no en el lado del cliente.

Por lo tanto, puedes usar Server Actions para recuperar la entrada del usuario enviada en el servidor y procesarla allí. Por ejemplo, podrías almacenar los datos enviados en un archivo o base de datos.

En consecuencia, las Server Actions son un componente importante cuando se busca crear aplicaciones React *fullstack* totalmente integradas. Por lo general, la obtención de datos por sí sola no es suficiente, por lo que existe la función Server Actions. Al tener ambas, RSCs y Server Actions, puedes obtener y mutar datos en el servidor, al mismo tiempo que habilitas experiencias de usuario interactivas del lado del cliente donde sea necesario.

#### Desbloquear Server Actions en proyectos React
Al igual que los RSCs, no puedes usar Server Actions en todos los proyectos de React. En su lugar, se requiere una configuración de proyecto especial para usar esta función. Por ejemplo, los proyectos de Next.js admiten Server Actions de fábrica (cuando usan el App Router). Al igual que con los RSCs, puedes pensar que Next.js hace lo siguiente:
1. El flujo de trabajo de compilación y el proceso de empaquetado separan el código que pertenece a las Server Actions para que no termine en el paquete del lado del cliente.
2. Next.js configura endpoints de API que activan las funciones de Server Action y responden con cualquier valor de retorno definido en esas funciones.
3. Next.js llama a estos endpoints cuando es necesario (por ejemplo, al enviar un formulario conectado a una Server Action, como se muestra en la siguiente sección).

Por lo tanto, las Server Actions, al igual que los RSCs, pueden ser difíciles de soportar en proyectos personalizados que no usan Next.js. Es absolutamente posible crear proyectos personalizados que brinden soporte tanto para Server Actions como para RSCs, pero no es trivial.

Afortunadamente, usar Server Actions (en proyectos que las admiten) no es complicado.

#### Definición y activación de Server Actions
Como se mencionó en la sección *Manejo de mutaciones de datos con Server Actions*, las Server Actions son muy similares a las acciones de formulario del cliente que ya conoces del Capítulo 9.

Pero hay dos diferencias clave que deben tenerse en cuenta al crear una Server Action:
1. Una función de Server Action **debe ser asíncrona** (es decir, debe usar `async`/`await`). No hay Server Actions síncronas.
2. Dentro de la función de Server Action, al principio del cuerpo de la función, debes agregar la directiva **`'use server'`**.

Por lo tanto, una Server Action válida se puede definir y utilizar en un componente de la siguiente manera:

```javascript
export default function UserFeedback() { 
  async function saveFeedback(formData) { 
    'use server'; 
    const feedback = formData.get('feedback'); 
    console.log(feedback); 
  } 

  return ( 
    <form action={saveFeedback}> 
      <p> 
        <label htmlFor="feedback">Your feedback</label> 
        <textarea id="feedback" name="feedback" rows={3} /> 
      </p> 
      <p><button>Submit</button></p> 
    </form> 
  ); 
}
```

Como puedes ver, además del hecho de que debe ser asíncrona y de que utiliza la directiva `'use server'`, esta función de acción se parece a las que viste en el Capítulo 9: recibe un objeto `formData` que React proporcionará cuando se envíe el formulario, y estableces la función de acción como un valor para la prop `action` en un elemento `<form>`.

Como se mencionó en la sección anterior, si buscaras este código en los archivos de código descargados por el navegador, no lo encontrarías: este código realmente solo se ejecuta en el lado del servidor.

> [!NOTE]
> El componente `UserFeedback` del ejemplo anterior es un RSC.
> Si lo piensas, esto podría ser extraño: después de todo, este componente maneja la entrada e interacción del usuario. ¿Por qué funciona entonces sin `'use client'`?
> Porque las Server Actions (vinculadas a la prop `action` del `<form>`) son especiales. React admite explícitamente este patrón dentro de los RSCs. `'use client'` es de hecho obligatorio para cualquier otro tipo de manejo de entrada del usuario (por ejemplo, si confías en las props `onSubmit` u `onChange`). Pero vincular Server Actions a través de la prop `action` está permitido.
> Además, es importante comprender que la directiva `'use server'` solo existe para marcar acciones como Server Actions. No puedes usarla, por ejemplo, para marcar componentes como componentes de servidor.

Por supuesto, la Server Action del ejemplo anterior actualmente solo registra la entrada en la consola. Una acción más realista probablemente almacenaría esos datos en algún lugar y redirigiría al usuario a otra página.

#### Manejo de la entrada del usuario y actualización de la interfaz de usuario
Considera esta versión actualizada del ejemplo anterior:

```javascript
import { storeFeedback } from '../lib/feedback-db'; 

function UserFeedback() { 
  async function saveFeedback(formData) { 
    'use server'; 
    const feedback = formData.get('feedback'); 
    storeFeedback(feedback); 
  } 

  return ( 
    <form action={saveFeedback}> 
      <p> 
        <label htmlFor="feedback">Your feedback</label> 
        <textarea id="feedback" name="feedback" rows={3} /> 
      </p> 
      <p><button>Submit</button></p> 
    </form> 
  ); 
}
```

La Server Action `saveFeedback()` ahora almacena el feedback extraído a través de la función `storeFeedback()`.

Esta función se define de la siguiente manera:

```javascript
import fs from 'node:fs/promises'; 

export async function storeFeedback(text) { 
  const storedFeedback = await fs.readFile('data/user-feedback.json'); 
  const feedback = JSON.parse(storedFeedback); 
  feedback.push({ id: new Date().getTime(), text }); 
  await fs.writeFile( 
    'data/user-feedback.json', 
    JSON.stringify(feedback) 
  ); 
}
```

En una aplicación real, los datos podrían almacenarse en una base de datos. Aquí, en este ejemplo simple, se almacenan simplemente en un archivo `user-feedback.json` que forma parte del proyecto general de Next.js.

Como puedes ver, del mismo modo que puedes acceder directamente a un archivo o base de datos desde dentro de un RSC, puedes editar directamente un archivo o enviar una consulta a la base de datos desde dentro de una Server Action.

También puedes actualizar la interfaz de usuario navegando mediante programación al usuario a una página diferente a partir de entonces. En una aplicación Next.js, puedes usar la función **`redirect()`** proporcionada por Next.js para activar dicha acción de navegación, por ejemplo, justo después de almacenar el texto de feedback enviado:

```javascript
import { redirect } from 'next/navigation'; 
import { storeFeedback } from '../lib/feedback-db'; 

export default function UserFeedback() { 
  async function saveFeedback(formData) { 
    'use server'; 
    const feedback = formData.get('feedback'); 
    await storeFeedback(feedback); 
    redirect('/thanks') 
  } 

  // same JSX code as before, hence omitted 
}
```

Este es un patrón muy común al crear aplicaciones *fullstack*, ya que a menudo deseas dirigir a los usuarios de tu sitio web a una página diferente una vez que hayan enviado los datos.

Pero también puedes usar un patrón diferente y actualizar la interfaz de usuario que contiene el formulario, según el envío del formulario.

#### Server Actions y `useActionState()`
Quizás recuerdes el Hook **`useActionState()`** del Capítulo 9, *Manejo de Entradas de Usuario y Formularios con Form Actions*. Este Hook se puede utilizar para derivar algún estado del componente a partir de una acción (de formulario). Ese estado, a su vez, se puede utilizar para actualizar la interfaz de usuario en función del resultado de la acción.

Dado que una Server Action es un tipo especial de acción de formulario, puedes usar ese mismo Hook para actualizar la interfaz de usuario en función de tu Server Action y sus valores devueltos.

Por ejemplo, podrías intentar usar `useActionState()` en el componente `UserFeedback` de esta manera:

```javascript
import { useActionState } from 'react'; 
import { redirect } from 'next/navigation'; 
import { storeFeedback } from '../lib/feedback-db'; 
import FeedbackForm from './FeedbackForm'; 

export default function UserFeedback() { 
  async function saveFeedback(prevState, formData) { 
    'use server'; 
    const feedback = formData.get('feedback'); 
    if (!feedback || feedback.trim() === '') { 
      return { error: 'Please provide some feedback!' }; 
    } 
    await storeFeedback(feedback); 
    redirect('/thanks'); 
  } 

  const [formState, formAction] = useActionState(saveFeedback, { 
    error: null, 
  }); 

  return ( 
    <form action={formAction}> 
      <p> 
        <label htmlFor="feedback">Your feedback</label> 
        <textarea id="feedback" name="feedback" rows={3} /> 
      </p> 
      {formState.error && <p id="error">{formState.error}</p>} 
      <p> 
        <button>Submit</button> 
      </p> 
    </form> 
  ); 
}
```

Sin embargo, el uso de este código causaría un error:

**Figura 16.15**: React se queja del uso de un Hook en un RSC.

Es un mensaje de error que ya conoces de la sección *No todos los componentes deberían ser RSCs* y de la Figura 16.8. React no permite el uso de Hooks en RSCs, y `UserFeedback` es un RSC.

La solución, por supuesto, es sencilla: simplemente agrega la directiva `'use client'` en la parte superior del archivo `UserFeedback.js`:

```javascript
'use client'; 
import { useActionState } from 'react'; 
import { redirect } from 'next/navigation'; 
import { storeFeedback } from '../lib/feedback-db'; 
import FeedbackForm from './FeedbackForm'; 

export default function UserFeedback() { 
  // component code didn't change, hence omitted 
}
```

Pero con este cambio aplicado, encontrarás otro mensaje de error:

**Figura 16.16**: React ahora se queja del uso de 'use server' y 'use client' en el mismo archivo.

Este mensaje de error ocurre porque el archivo del componente `UserFeedback` está usando actualmente las directivas `'use client'` y `'use server'`, en diferentes lugares, pero en el mismo archivo.

Dicho de otra manera: **solo puedes definir una Server Action (y por lo tanto usar `'use server'`) dentro de un RSC, no dentro de un componente de cliente**.

Una posible solución para este problema es mover el formulario de feedback y el Hook `useActionState()` a un nuevo componente que se utilizará como componente secundario de `UserFeedback`. La función de Server Action se puede pasar a través de props a ese componente recién agregado.

Por ejemplo, puedes crear un componente `FeedbackForm` que se vea así:

```javascript
'use client'; 
import { useActionState } from 'react'; 

export default function FeedbackForm({action}) { 
  const [formState, formAction] = useActionState(action, { 
    error: null, 
  }); 

  return ( 
    <form action={formAction}> 
      <p> 
        <label htmlFor="feedback">Your feedback</label> 
        <textarea id="feedback" name="feedback" rows={3} /> 
      </p> 
      {formState.error && <p id="error">{formState.error}</p>} 
      <p> 
        <button>Submit</button> 
      </p> 
    </form> 
  ); 
}
```

Este componente `FeedbackForm` espera una prop `action`, que luego se pasa como un valor a `useActionState()`. En consecuencia, el componente `FeedbackForm` se puede utilizar en el componente `UserFeedback` de la siguiente manera:

```javascript
import { redirect } from 'next/navigation'; 
import { storeFeedback } from '../lib/feedback-db'; 
import FeedbackForm from './FeedbackForm'; 

export default function UserFeedback() { 
  async function saveFeedback(prevState, formData) { 
    'use server'; 
    const feedback = formData.get('feedback'); 
    if (!feedback || feedback.trim() === '') { 
      return { error: 'Please provide some feedback!' }; 
    } 
    await storeFeedback(feedback); 
    redirect('/thanks'); 
  } 

  return <FeedbackForm action={saveFeedback} />; 
}
```

Si ejecutaras este código, la aplicación funcionaría sin ningún problema. Entonces, nuevamente, al igual que con los RSCs, todo se reduce a crear una estructura de componentes que funcione.

Esta es una forma absolutamente válida de resolver este problema. Pero si prefieres no dividir el componente `UserFeedback` en varios componentes y externalizar el formulario en `FeedbackForm`, también existe otra solución posible.

#### Almacenamiento de Server Actions en archivos separados
Puedes definir Server Actions directamente dentro de los RSCs. Como aprendiste en el capítulo anterior, también puedes pasarlas a través de props.

Como alternativa, React también permite **almacenarlas en archivos separados**. Hacerlo te permite crear componentes más limpios, ya que el código de la Server Action se mueve fuera de las funciones de los componentes. Además, React permite importar una Server Action almacenada en un archivo separado directamente dentro de un archivo de componente de cliente.

Teniendo en cuenta los ejemplos de código anteriores, podrías mover la Server Action `saveFeedback()` a un archivo separado `actions/feedback.js` en la carpeta de tu proyecto Next.js (los nombres de archivos y carpetas dependen totalmente de ti). En ese archivo, puedes mover la directiva `'use server'` fuera de la Server Action y **colocarla en la parte superior del archivo**:

```javascript
'use server'; 
import { redirect } from 'next/navigation'; 
import { storeFeedback } from '../lib/feedback-db'; 

export async function saveFeedback(prevState, formData) { 
  const feedback = formData.get('feedback'); 
  if (!feedback || feedback.trim() === '') { 
    return { error: 'Please provide some feedback!' }; 
  } 
  await storeFeedback(feedback); 
  redirect('/thanks'); 
}
```

Agregar la directiva `'use server'` en la parte superior del archivo te permite crear múltiples funciones de Server Action en ese mismo archivo. Luego puedes exportarlas y usarlas en cualquier otro archivo en el que se puedan necesitar.

Por ejemplo, puedes importar la acción `saveFeedback()` en el componente `UserFeedback`, que ahora ya no necesita el componente secundario separado `FeedbackForm`. Dado que las Server Actions almacenadas externamente se pueden importar en archivos de componentes de cliente sin problemas, el archivo final `UserFeedback.js` se ve así:

```javascript
'use client'; 
import { saveFeedback } from '../actions/feedback'; 
import { useActionState } from 'react'; 

export default function UserFeedback() { 
  const [formState, formAction] = useActionState(saveFeedback, { 
    error: null, 
  }); 

  return ( 
    <form action={formAction}> 
      <p> 
        <label htmlFor="feedback">Your feedback</label> 
        <textarea id="feedback" name="feedback" rows={3} /> 
      </p> 
      {formState.error && <p id="error">{formState.error}</p>} 
      <p> 
        <button>Submit</button> 
      </p> 
    </form> 
  ); 
}
```

Por lo tanto, almacenar Server Actions en archivos separados no solo conduce a componentes más limpios, sino que también puede ayudar a evitar refactorizaciones innecesarias de componentes.

Sin embargo, no importa qué enfoque elijas: puedes usar Server Actions para manejar envíos de formularios en el servidor. Junto con los RSCs, puedes crear aplicaciones *fullstack* que combinan a la perfección el código del lado del cliente y del lado del servidor.

---

### Sección 5: Resumen y puntos clave

- React admite dos funciones especiales del lado del servidor: **RSCs** y **Server Actions**.
- Ambas características no están disponibles en proyectos de React a menos que el proyecto esté configurado específicamente para admitirlas; por lo general, necesitarás usar un framework que admita estas características (por ejemplo, **Next.js**).
- Los **RSC** son componentes que **nunca se renderizan en el lado del cliente**; en su lugar, pueden renderizarse en el servidor (iniciados mediante solicitudes HTTP) o durante el proceso de compilación.
- Los RSC devuelven instrucciones de renderizado que son interpretadas por React en el lado del cliente.
- Dado que los RSC nunca se ejecutan en el lado del cliente, puedes utilizar APIs y funciones exclusivas del servidor en ellos.
- React también permite que los RSC devuelvan valores de tipo `Promise`, por lo que puedes usar `async`/`await` y obtener datos de forma asíncrona sin problemas en los RSC.
- Para crear sitios web interactivos donde la interfaz de usuario pueda cambiar después de renderizarse, puedes marcar los componentes como **componentes de cliente** mediante la directiva **`'use client'`**.
- Solo los componentes de cliente pueden usar Hooks como `useState()` o configurar escuchadores de eventos.
- Los componentes de cliente también se pre-renderizan en el servidor, pero a diferencia de los RSCs, también pueden ejecutarse en el lado del cliente.
- Puedes importar y usar componentes de cliente dentro de RSCs.
- Al importar componentes de servidor en componentes de cliente, los componentes de servidor se convierten automáticamente en componentes de cliente, si es posible.
- Si un RSC no se puede convertir en un componente de cliente (por ejemplo, porque usa `async`/`await` o APIs de Node), deberás reestructurar el árbol de componentes (por ejemplo, pasándolo a través de props como `children`).
- Puedes pasar componentes de servidor a componentes de cliente (sin convertirlos) a través de props.
- React ayuda a gestionar los envíos de formularios en el servidor mediante **Server Actions**.
- Las Server Actions funcionan como las acciones de cliente (consulta el Capítulo 9), pero deben ser asíncronas (`async`/`await`) y utilizar la directiva **`'use server'`**.
- Puedes definir Server Actions dentro de RSCs o en archivos separados; en este último caso, puedes mover la directiva `'use server'` a la parte superior del archivo para definir múltiples Server Actions en el mismo archivo.

---

### Sección 6: ¿Qué sigue?

En este capítulo, aprendiste sobre los RSCs y las Server Actions. Aprendiste que crearlos y usarlos es relativamente sencillo, pero que darles soporte en proyectos no lo es; de ahí que frameworks como Next.js se utilicen comúnmente para aprovechar estas características.

Este capítulo te dio una idea de cómo funcionan los RSCs y las Server Actions detrás de escena, y qué ventajas ofrecen estas características. A lo largo de este capítulo, también aprendiste sobre los componentes de cliente y cómo combinar componentes de servidor y de cliente. Finalmente, se analizaron las Server Actions y se mostraron diferentes formas de definirlas y utilizarlas.

El próximo capítulo se basará en este capítulo y explorará cómo la funcionalidad **`Suspense`** de React puede ayudar a mostrar contenido alternativo mientras se transmiten los datos obtenidos mediante *streaming*.

---

### Sección 7: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se encuentran en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/16-rsc-server-actions/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/16-rsc-server-actions/exercises/questions-answers.md):

1. ¿Cuál es la característica definitoria de los React Server Components?
2. ¿Cuáles son los problemas que resuelven los React Server Components?
3. ¿Cómo se crean y utilizan los React Server Components en proyectos Next.js?
4. ¿Por qué los React Server Components y las Server Actions no se pueden utilizar en todos los proyectos de React?
5. ¿Cuál es la diferencia clave entre el Renderizado en el Servidor (SSR) y los React Server Components (RSC)?
6. ¿Cuál es el propósito de la directiva `'use client'`?
7. ¿Cómo afecta la directiva `'use client'` a los componentes secundarios?
8. ¿Cuáles son las reglas para combinar componentes de servidor y componentes de cliente?
9. ¿Cómo puedes manejar los estados de carga en Next.js mientras obtienes datos con RSCs?
10. ¿Qué son las Server Actions en React y en qué se diferencian de las acciones de cliente?
11. ¿Cómo puedes activar una Server Action?
12. ¿Cómo puedes actualizar la interfaz de usuario después de una Server Action?
13. ¿Se pueden definir Server Actions en archivos separados?

---

### Sección 8: Aplica lo aprendido

Con todo el conocimiento recién adquirido sobre Next.js, es hora de aplicarlo a un proyecto de demostración real: una aplicación de demostración que se renderizará en el servidor.

En la siguiente sección, encontrarás una actividad que te permitirá practicar el trabajo con Next.js. Como siempre, también necesitarás emplear algunos de los conceptos cubiertos en capítulos anteriores.

#### Actividad 16.1: Construir un Mini Blog
En esta actividad, tu trabajo consiste en crear un sitio web de blog muy simple (con Next.js) que permita a los usuarios crear y ver publicaciones de blog. Cada publicación de blog debe constar de un título, una fecha y un cuerpo de texto. Se debe renderizar una lista de títulos y fechas de publicaciones de blog en la página de inicio (`/`); al hacer clic en una publicación, los usuarios deben ser llevados a la página de detalles (`/blog/<algun-id>`), que muestra los datos completos de la publicación del blog. Una página `/blog/new` debe mostrar un formulario que se puede utilizar para crear una nueva publicación.

Las publicaciones deben almacenarse en un archivo `posts.json` (que simplemente puede almacenar un array de objetos de publicación). Después de crear una nueva publicación, los usuarios deben ser redirigidos a la página de detalles de esa publicación. Si los usuarios dejan vacío el campo del título o del cuerpo (o ambos), se debe mostrar un mensaje de error debajo del formulario.

> [!NOTE]
> Puedes encontrar una instantánea del proyecto inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/16-rsc-server-actions/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/16-rsc-server-actions/activities/practice-1-start). Al descargar este código, siempre descargarás todo el repositorio. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start`, en este caso) para usar la instantánea de código correcta.
> En el proyecto inicial proporcionado, puedes explorar el archivo `globals.css` para hacerte una idea de los elementos y la estructura de elementos que quizás desees utilizar para aprovechar los estilos proporcionados. Por supuesto, también puedes configurar y utilizar tus propios estilos.

Después de descargar el código y ejecutar `npm install` en la carpeta del proyecto para instalar todas las dependencias requeridas, los pasos de la solución son los siguientes:
1. Agrega tres nuevos archivos `page.js` (y una estructura de carpetas adecuada) para las tres páginas: `/`, `/blog/new` y `/blog/<algun-id>`.
2. Agrega un nuevo archivo `posts.json` en una carpeta `data/` en la carpeta raíz del proyecto. Este archivo debe almacenar inicialmente un array vacío.
3. Muestra un `<form>` con campos de entrada de título y cuerpo en la página `/blog/new`.
4. Crea una nueva Server Action en un archivo separado e impórtala y "conéctala" al `<form>`. La Server Action debe recuperar el título y el cuerpo del texto ingresados, crear un nuevo objeto (que también incluye un ID y una instantánea de la fecha de creación) y almacenar esos datos en el archivo `posts.json`. Los datos deben almacenarse de modo que las publicaciones de blog existentes no se pierdan.
5. Actualiza la Server Action para implementar la validación de entradas y mostrar los resultados de la validación encima del botón de envío.
6. Obtén las publicaciones de blog en la página de inicio y muestra una lista de publicaciones de blog (título y fecha). Cada publicación debe ser clickeable y llevar al usuario a la página de detalles.
7. En la página de detalles, obtén y muestra los detalles de la publicación del blog (utilizando el ID).
8. Finalmente, redirige al usuario a la página de detalles adecuada desde dentro de la Server Action, después de que se cree una publicación de blog.

La página final debería verse como se muestra en las siguientes capturas de pantalla:

**Figura 16.17**: La página de inicio, que muestra una lista de publicaciones de blog.

**Figura 16.18**: La página /blog/new, a la espera de la entrada del usuario.

**Figura 16.19**: La página /blog/<some-id> que muestra los detalles de la publicación del blog.

> [!NOTE]
> Puedes encontrar el código completo para esta actividad y una solución de ejemplo aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/16-rsc-server-actions/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/16-rsc-server-actions/activities/practice-1).
