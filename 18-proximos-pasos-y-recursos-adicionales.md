# Parte 2: Construcción de Aplicaciones React Complejas

## Capítulo 18: Próximos Pasos y Recursos Adicionales

### Objetivos de aprendizaje
Al finalizar este capítulo, sabrás lo siguiente:
- Cómo dar el paso de leer el libro a aplicar tus conocimientos.
- Cómo practicar de la mejor manera lo que has aprendido a lo largo de este libro.
- Qué temas de React puedes explorar a continuación.
- Qué paquetes de terceros populares de React podrían merecer una mirada más cercana.

---

### Sección 1: Introducción

Con este libro, has obtenido una exhaustiva (re)introducción a los conceptos clave de React que debes conocer para trabajar con éxito con React, proporcionando orientación tanto teórica como práctica sobre componentes, props, estado, contexto, React Hooks, enrutamiento, React en el lado del servidor y muchos otros conceptos cruciales.

Pero React es más que una simple colección de conceptos e ideas. Impulsa todo un ecosistema de bibliotecas de terceros que ayudan con muchos problemas comunes específicos de React. También existe una enorme comunidad de React que comparte soluciones para problemas comunes o patrones populares.

En este último y breve capítulo, aprenderás sobre algunas de las bibliotecas de terceros más importantes y populares que quizás quieras explorar. También se te presentarán otros excelentes recursos que ayudan a aprender React. Además, este capítulo compartirá algunas recomendaciones sobre cómo proceder y continuar creciendo como desarrollador de React después de terminar este libro.

---

### Sección 2: ¿Cómo deberías continuar?

Utiliza el conocimiento que adquiriste a lo largo de este libro como una base sobre la cual construir. Profundiza en Next.js, explora otras bibliotecas populares de React o aprende más sobre alternativas a React como Angular o Vue. El desarrollo web ofrece una amplia gama de tecnologías, lenguajes, bibliotecas, patrones y conceptos. Y aunque esto a veces puede resultar abrumador, también es un vasto océano de oportunidades para crecer como desarrollador y mejorar en la resolución de problemas complejos.

Pero además de aprender más sobre React y los paquetes relacionados, también es importante aplicar tus conocimientos y practicar lo que has aprendido. No te limites a leer libro tras libro. En su lugar, utiliza tus habilidades recién adquiridas para construir algunos proyectos de demostración.

No tienes que construir el próximo Amazon o TikTok. Hay una razón por la que aplicaciones como esas son construidas por equipos gigantescos. Pero deberías crear pequeños proyectos de demostración que se centren en un par de problemas centrales. Podrías, por ejemplo, crear un sitio web muy básico que permita a los usuarios almacenar y ver sus objetivos diarios, o crear una página básica de *Meetups* donde los visitantes puedan organizar y unirse a eventos.

En pocas palabras: **la práctica es la clave**. Debes aplicar lo aprendido y construir cosas. Porque al construir proyectos de demostración, automáticamente te encontrarás con problemas que tendrás que resolver sin tener una solución a mano. Tendrás que probar diferentes enfoques y buscar en Internet posibles soluciones (o soluciones parciales). En última instancia, así es como más se aprende y cómo desarrollas tus habilidades de resolución de problemas.

No encontrarás una solución para todos los problemas en este libro, pero este libro sí te brinda las herramientas básicas y los bloques de construcción que te ayudarán con esos problemas. Las soluciones se construyen luego combinando estos bloques de construcción y basándose en el conocimiento adquirido a lo largo de este libro.

#### Conviértete en un desarrollador Fullstack de React
Este libro ya cubrió conceptos cruciales para iniciarte en el desarrollo backend basado en React. Los Capítulos 15, 16 y 17 exploraron el renderizado en el servidor (*Server-Side Rendering*), Next.js, componentes y acciones del servidor (*Server Components* y *Server Actions*), y características relacionadas que se necesitarán para crear aplicaciones React *fullstack*.

En consecuencia, profundizar en Next.js podría ser un próximo paso interesante. Con la ayuda de la documentación oficial o cursos en línea como mi curso *Next.js & React – The Complete Guide*, puedes adquirir los conocimientos necesarios para convertirte en un desarrollador React *fullstack*.

Y no es solo Next.js: también puedes explorar alternativas como Remix y React Router (que está recibiendo más capacidades *fullstack*) o TanStack Start. Si no te importa tener una experiencia de desarrollo *fullstack* totalmente integrada como la que ofrece Next.js, también puedes aprender más sobre cómo conectar un backend desacoplado a un frontend de React; es decir, puedes aprender a construir y conectar una API de backend separada (REST o GraphQL) con Node.js o cualquier otro lenguaje de backend.

Sin embargo, convertirte en un desarrollador *fullstack* no es algo que estés obligado a hacer. Es una opción, pero según tus preferencias personales o tu función en un equipo, podría no ser la opción adecuada para ti. Solo es importante saber que crear aplicaciones *fullstack* con React es un camino posible que podrías explorar, y que es un camino que se volvió considerablemente más fácil con Next.js y frameworks similares. De cualquier manera, como se mencionó anteriormente, también deberías aplicar tus conocimientos de React y practicar construyendo proyectos de demostración, sin importar si estás profundizando en el desarrollo *fullstack* o no.

#### Problemas interesantes para explorar
Entonces, ¿qué problemas y aplicaciones de demostración podrías explorar e intentar construir?

En general, puedes intentar crear clones (simplificados) de aplicaciones web populares (como una versión muy simplificada de Amazon). En última instancia, tu imaginación es el límite, pero en las siguientes secciones encontrarás detalles y consejos para tres ideas de proyectos y los desafíos que conllevan.

##### Construir un carrito de compras
Un tipo de sitio web muy común es una tienda online (*e-commerce*). Puedes encontrar tiendas online para todo tipo de productos, desde bienes físicos como libros, ropa o muebles, hasta productos digitales como videojuegos o películas, y crear una tienda online de este tipo sería una idea de proyecto y un desafío interesantes.

Por supuesto, las tiendas online vienen con muchas características que no se pueden construir solo con React del lado del cliente. Por ejemplo, todo el proceso de pago es principalmente una tarea de backend donde las solicitudes deben ser manejadas por servidores. La gestión de inventario sería otra característica que tiene lugar en bases de datos y en servidores, y no en los navegadores de los visitantes de tu sitio web. En consecuencia, puedes usar Next.js (o una de las alternativas mencionadas anteriormente en este capítulo) para encargarte de esta funcionalidad de backend y así construir una aplicación React *fullstack*. Pero incluso si no deseas sumergirte en el desarrollo *fullstack*, las tiendas online contienen muchas características que requieren interfaces de usuario interactivas (y, por lo tanto, se benefician del uso de las funciones del lado del cliente de React). Por ejemplo, puedes configurar diferentes páginas que muestren listas de productos disponibles, detalles del producto o el estado actual de un pedido, como aprendiste en el Capítulo 13, *Aplicaciones Multipágina con React Router*. También sueles tener carritos de compras en los sitios web. Crear un carrito de este tipo, combinado con la funcionalidad de agregar y eliminar artículos, utilizaría de manera similar varias características de React; por ejemplo, la gestión del estado, como se explicó en el Capítulo 4, *Trabajando con Eventos y Estado*.

Todo comienza con tener un par de páginas (rutas) para productos de prueba ficticios (que están codificados en el código frontend y no se obtienen de algún backend), detalles de productos y el carrito de compras en sí. El carrito de compras muestra elementos que deben gestionarse a través del estado de toda la aplicación (por ejemplo, a través de contexto, como se cubrió en el Capítulo 11, *Trabajando con Estado Complejo*), ya que los visitantes del sitio web deben poder agregar artículos al carrito desde la página de detalles del producto. También necesitarás una amplia variedad de componentes de React, muchos de los cuales deben ser reutilizables (por ejemplo, los elementos individuales del carrito de compras que se muestran). Tu conocimiento de los componentes y props de React del Capítulo 2, *Entendiendo los Componentes de React y JSX*, y del Capítulo 3, *Componentes y Props*, te ayudará con eso.

El estado del carrito de compras también es un estado no trivial. Una simple lista de productos normalmente no será suficiente, aunque, por supuesto, al menos puedes aplicar tus conocimientos del Capítulo 5, *Renderizado de Listas y Contenido Condicional*. En su lugar, debes verificar si un artículo ya forma parte del carrito o si se agrega por primera vez. Si ya forma parte del carrito, debes actualizar la cantidad del artículo en el carrito. Por supuesto, también deberás asegurarte de que los usuarios puedan eliminar artículos del carrito o reducir la cantidad de un artículo. Y si quieres hacerlo aún más sofisticado, incluso puedes simular cambios de precios que deben tenerse en cuenta al actualizar el estado del carrito de compras.

Como puedes ver, esta tienda online ficticia extremadamente simple ya ofrece bastante complejidad. Por supuesto, como se mencionó anteriormente, también podrías agregar funcionalidad de backend y almacenar productos ficticios en una base de datos. Si lo deseas, puedes profundizar en Next.js para crear una aplicación *fullstack* más compleja basada en React. Esto te permite aplicar los conocimientos adquiridos en el Capítulo 15, *Renderizado en el Servidor (SSR) y Desarrollo Fullstack con Next.js*, y en el Capítulo 16, *React Server Components y Server Actions*.

##### Construir el sistema de autenticación de una aplicación (registro e inicio de sesión de usuarios)
Muchos sitios web permiten a los usuarios registrarse o iniciar sesión. Para muchos sitios web, la autenticación de usuarios es obligatoria antes de realizar ciertas tareas. Por ejemplo, debes crear una cuenta de Google antes de subir vídeos a YouTube o usar Gmail (y muchos otros servicios de Google). De manera similar, normalmente se necesita una cuenta antes de realizar cursos online de pago o comprar videojuegos (digitales) online. Tampoco puedes realizar operaciones bancarias online sin haber iniciado sesión. Y esa es solo una lista corta; se podrían agregar muchos más ejemplos, pero entiendes la idea. La autenticación de usuarios es necesaria por una amplia variedad de razones en muchos sitios web.

Y en aún más sitios web, está disponible de forma opcional. Por ejemplo, es posible que puedas pedir productos como invitado, pero te beneficias de ventajas adicionales al crear una cuenta (por ejemplo, puedes realizar un seguimiento de tu historial de pedidos o acumular puntos de recompensa).

Por supuesto, crear tu propia versión de YouTube es un desafío demasiado grande como para ser un buen proyecto de práctica. Hay una razón por la que Google tiene miles de desarrolladores en nómina. Sin embargo, puedes identificar y clonar características individuales, como la autenticación de usuarios.

Construye tu propio sistema de autenticación de usuarios con React. Asegúrate de que los usuarios puedan registrarse e iniciar sesión. Agrega algunas páginas de ejemplo (rutas) a tu sitio web y encuentra una manera de hacer que algunas páginas solo estén disponibles para usuarios que hayan iniciado sesión. Puede que estos objetivos no parezcan gran cosa, pero en realidad te enfrentarás a bastantes desafíos en el camino: desafíos que te obligarán a encontrar soluciones para problemas completamente nuevos.

Si bien podrías simplemente usar alguna lógica ficticia (del lado del cliente) en el código de tu aplicación React para simular solicitudes HTTP que se envían a tus servidores detrás de escena, también podrías agregar un backend de demostración real en su lugar. Ese backend necesitaría almacenar cuentas de usuario en una base de datos, validar solicitudes de inicio de sesión y devolver tokens de autenticación que informen al frontend de React sobre el estado de autenticación actual de un usuario. En tu aplicación React, estas solicitudes HTTP se tratarían como efectos secundarios, como se cubrió en el Capítulo 8, *Manejo de Efectos Secundarios*.

Nuevamente, si deseas utilizar un backend real, también deberás sumergirte en el desarrollo backend y crear una aplicación independiente del lado del servidor o usar Next.js (o cualquier framework React *fullstack* similar). Alternativamente, también puedes usar servicios como Firebase, Supabase, Auth0 o uno de los muchos otros servicios que proporcionan backends de autenticación para aplicaciones frontend. De cualquier manera, puedes explorar cómo conectar tu aplicación React a dicho backend.

Como puedes ver, esta idea de proyecto "simple" (o, más bien, idea de característica) presenta muchos desafíos y requerirá que te bases en tus conocimientos de React y encuentres soluciones para una amplia variedad de problemas.

##### Construir un sitio web de gestión de eventos
Si primero construyeras tu propio sistema de carrito de compras y te iniciaras en la autenticación de usuarios, podrías dar un paso más y crear un sitio web más complejo que combine estas características (y ofrezca características nuevas y adicionales).

Una de esas ideas de proyectos sería un sitio de gestión de eventos. Este es un sitio web en el que los usuarios pueden crear cuentas y, una vez que hayan iniciado sesión, eventos. Todos los visitantes pueden luego explorar estos eventos y registrarse en ellos. Dependería de ti si el registro como invitado (sin crear una cuenta primero) es posible o no.

También es tu elección si deseas agregar lógica de backend (es decir, un servidor que maneje solicitudes y almacene usuarios y eventos en una base de datos) o simplemente almacenar todos los datos en tu aplicación React (a través del estado de toda la aplicación). Si no agregas un backend, todos los datos se perderán cada vez que se recargue la página y no podrás ver los eventos creados por otros usuarios en otras máquinas, pero aún podrás practicar todas estas características clave de React.

Hay muchas características de React que son necesarias para este tipo de sitio web de prueba: componentes reutilizables, páginas (rutas), estado específico de componentes y de toda la aplicación, manejo y validación de entradas de usuario, visualización de datos condicionales y de listas, y mucho más.

Nuevamente, esta no es una lista exhaustiva de ejemplos. Puedes construir lo que quieras. Sé creativo y experimenta, porque solo dominarás React si lo usas para resolver problemas.

#### Bibliotecas comunes y populares de React
No importa qué tipo de aplicación React estés creando, encontrarás muchos problemas y desafíos en el camino. Desde el manejo y la validación de la entrada del usuario hasta el envío de solicitudes HTTP, las aplicaciones complejas conllevan muchos desafíos.

Puedes resolver todos los desafíos por tu cuenta e incluso escribir todo el código (React) necesario por tu cuenta. Y para practicar, esto podría ser una buena idea. Pero a medida que construyes aplicaciones cada vez más complejas, puede tener sentido externalizar ciertos problemas.

Afortunadamente, React cuenta con un ecosistema rico y vibrante que ofrece paquetes de terceros que resuelven todo tipo de problemas comunes. Aquí hay una breve lista no exhaustiva de bibliotecas de terceros populares que pueden ser útiles:
- **TanStack Query**: Una biblioteca muy popular que ayuda con la obtención, almacenamiento en caché y gestión de datos en aplicaciones React ([https://tanstack.com/query/latest](https://tanstack.com/query/latest)).
- **Framer Motion**: Una biblioteca específica de React que te permite crear e implementar animaciones potentes y visualmente agradables en tus aplicaciones React ([https://www.framer.com/motion/](https://www.framer.com/motion/)).
- **React Hook Form**: Una biblioteca que simplifica el proceso de manejo y validación de entradas de usuario ([https://react-hook-form.com/](https://react-hook-form.com/)).
- **Formik**: Otra biblioteca popular que ayuda con el manejo y validación de entradas de formularios ([https://formik.org/](https://formik.org/)).
- **Axios**: Una biblioteca general de JavaScript que simplifica el proceso de enviar solicitudes HTTP y manejar respuestas ([https://axios-http.com/](https://axios-http.com/)).
- **Redux**: En el pasado, esta era una biblioteca esencial de React. Hoy en día, todavía puede ser importante, ya que puede simplificar enormemente la gestión del estado (complejo) entre componentes o de toda la aplicación ([https://redux.js.org/](https://redux.js.org/)).
- **Zustand**: Si necesitas una biblioteca adicional que ayude a administrar el estado en aplicaciones React, también puedes explorar Zustand, una alternativa muy popular a Redux ([https://zustand-demo.pmnd.rs/](https://zustand-demo.pmnd.rs/)).

Esta es solo una lista corta de algunas bibliotecas útiles y populares. Dado que hay una cantidad infinita de desafíos potenciales, también podrías compilar una lista infinita de bibliotecas. Los motores de búsqueda y Stack Overflow (un foro de mensajes para desarrolladores) son tus amigos a la hora de encontrar más bibliotecas que resuelvan otros problemas.

#### Uso de TypeScript
También puedes considerar el uso de **TypeScript**, en lugar de JavaScript puro, para tus proyectos de React.

TypeScript es un superconjunto de JavaScript que agrega tipado fuerte y estricto. Como resultado, el uso de TypeScript puede ayudarte a detectar y evitar ciertos errores relacionados con valores faltantes o tipos de valores incorrectos.

Puedes comenzar con TypeScript para React con la ayuda de la documentación oficial ([https://react.dev/learn/typescript](https://react.dev/learn/typescript)) o cursos o tutoriales en línea dedicados.

#### Otros recursos
Como se mencionó, React tiene un ecosistema altamente vibrante, y no solo en lo que respecta a bibliotecas de terceros. También encontrarás miles de artículos de blog que discuten todo tipo de mejores prácticas, patrones, ideas y soluciones a posibles problemas. Buscar las palabras clave adecuadas (como *React form validation with Hooks*) casi siempre generará artículos interesantes o bibliotecas útiles.

También encontrarás muchos cursos en línea de pago, como el curso *React – The Complete Guide* en [https://www.udemy.com/course/react-the-complete-guide-incl-redux/](https://www.udemy.com/course/react-the-complete-guide-incl-redux/), y tutoriales gratuitos en YouTube.

La documentación oficial es otro excelente lugar para explorar, ya que contiene análisis detallados de temas centrales, así como más artículos de tutoriales: [https://react.dev/](https://react.dev/).

#### Más allá de React para aplicaciones web
Este libro se centró en el uso de React para crear sitios web. Esto se debió a un par de razones. La primera es que React, históricamente, se creó para simplificar el proceso de construcción de interfaces de usuario web complejas, y React impulsa cada día más y más sitios web. Es una de las bibliotecas de desarrollo web del lado del cliente más utilizadas y es más popular que nunca.

Pero también tiene sentido aprender a usar React para el desarrollo web porque no necesitas herramientas adicionales: solo un editor de texto y un navegador.

Dicho esto, React también se puede utilizar para crear interfaces de usuario fuera del navegador y de los sitios web. Con **React Native** e **Ionic for React**, tienes dos proyectos y bibliotecas muy populares que utilizan React para crear aplicaciones móviles nativas para iOS y Android.

Por lo tanto, después de aprender todos estos elementos esenciales de React, tiene mucho sentido explorar también estos proyectos. Elige algunos cursos de React Native o Ionic (o usa la documentación oficial) para aprender cómo puedes usar todos los conceptos de React cubiertos en este libro para crear aplicaciones móviles nativas reales que se puedan distribuir a través de las tiendas de aplicaciones de cada plataforma.

React se puede utilizar para crear todo tipo de interfaces de usuario interactivas para diversas plataformas. Ahora que has terminado este libro, tienes las herramientas necesarias para construir tu próximo proyecto con React, sin importar a qué plataforma se dirija.

---

### Sección 3: Palabras finales

Con todos los conceptos discutidos a lo largo de este libro, así como los recursos adicionales y puntos de partida para profundizar, estás bien preparado para crear aplicaciones web ricas en funciones y altamente intuitivas con React.

No importa si se trata de un blog simple o de una solución compleja de Software como Servicio (*Software-as-a-Service*), ahora conoces los conceptos clave de React que necesitas para crear una aplicación web impulsada por React que a tus usuarios les encantará.

Espero que hayas aprovechado al máximo este libro. Por favor, comparte cualquier comentario que tengas, por ejemplo, a través de X (`@maxedapps`) o enviando un correo electrónico a `customercare@packt.com`.

---

### Sección 4: Únete a nosotros en Discord

Lee este libro junto a otros usuarios, expertos en IA y el propio autor.
Haz preguntas, proporciona soluciones a otros lectores, charla con el autor a través de sesiones de *Ask Me Anything*, y mucho más.
Escanea el código QR o visita el enlace para unirte a la comunidad:
[https://packt.link/ReactKeyConcepts2e](https://packt.link/ReactKeyConcepts2e)

---

### Sección 5: ¿Por qué suscribirse?

- Pasa menos tiempo aprendiendo y más tiempo programando con eBooks y vídeos prácticos de más de 4.000 profesionales de la industria.
- Mejora tu aprendizaje con Planes de Habilidades (*Skill Plans*) creados especialmente para ti.
- Obtén un eBook o vídeo gratuito cada mes.
- Búsqueda completa para un fácil acceso a información vital.
- Copia y pega, imprime y guarda contenido en marcadores.

En [www.packt.com](https://www.packt.com/), también puedes leer una colección de artículos técnicos gratuitos, suscribirte a una variedad de boletines informativos gratuitos y recibir descuentos y ofertas exclusivas en libros y eBooks de Packt.
