# Parte 1: Fundamentos de React

## Capítulo 1: React – Qué es y por qué

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Describir qué es React y por qué deberías usarlo.
- Comparar React con proyectos web construidos únicamente con JavaScript.
- Explicar la diferencia entre código imperativo y declarativo.
- Diferenciar entre aplicaciones de página única (*Single-Page Applications* o SPAs) y aplicaciones multipágina.
- Crear nuevos proyectos de React.

---

### Sección 1: Introducción

React.js (o simplemente React, como también se le conoce y como se le denominará durante la mayor parte de este libro) es una de las librerías de JavaScript para frontend más populares —quizás incluso la más popular, según la encuesta a desarrolladores de Stack Overflow de 2023—. Actualmente es utilizada por más del 5% de los 1.000 sitios web más importantes y, en comparación con otras librerías y frameworks populares de JavaScript para frontend como Angular, React lidera por un margen enorme al observar métricas clave como las descargas semanales de paquetes a través de npm, que es una herramienta comúnmente utilizada para descargar y gestionar paquetes de JavaScript.

Aunque ciertamente es posible escribir buen código de React sin comprender completamente cómo funciona internamente y por qué lo estás utilizando, es muy probable que puedas aprender conceptos avanzados más rápido y evitar errores si intentas comprender las herramientas con las que trabajas, así como las razones para elegir una herramienta determinada en primer lugar.

Por lo tanto, antes de considerar cualquier cosa sobre sus conceptos e ideas centrales o revisar código de ejemplo, primero debes entender qué es realmente React y por qué existe. Esto te ayudará a comprender cómo funciona React internamente y por qué ofrece las características que tiene.

Si ya sabes por qué estás usando React, por qué se utilizan soluciones como React en general en lugar de JavaScript puro (*vanilla JavaScript*, es decir, JavaScript sin ningún framework o librería, más sobre esto en la siguiente sección), y cuál es la idea detrás de React y su sintaxis, puedes, por supuesto, saltarte esta sección y avanzar directamente a los capítulos más orientados a la práctica más adelante en este libro.

Pero si solo crees que lo sabes y no estás 100% seguro, definitivamente deberías leer este capítulo primero.

---

### Sección 2: ¿Qué es React?

React es una librería de JavaScript, y si echas un vistazo a la página web oficial (el sitio web oficial y la documentación de React están disponibles en: [https://react.dev/](https://react.dev/)), descubrirás que sus creadores la definen como: *"La librería para interfaces de usuario web y nativas"*.

¿Pero qué significa esto?

Primero, es importante entender que React es una librería de JavaScript. Como lector de este libro, sabes qué es JavaScript y por qué se utiliza en el navegador. JavaScript te permite añadir interactividad a tu sitio web ya que, con JavaScript, puedes reaccionar a los eventos del usuario y manipular la página después de que se haya cargado. Esto es extremadamente valioso, ya que te permite construir interfaces de usuario web (UIs) altamente interactivas.

¿Pero qué es una "librería" y cómo ayuda React en la construcción de interfaces de usuario?

Aunque se pueden tener discusiones filosóficas sobre qué es una librería (y en qué se diferencia de un framework), la definición pragmática de una librería es que se trata de una colección de funcionalidades que puedes usar en tu código para lograr resultados que normalmente requerirían más código y trabajo por tu parte. Las librerías pueden ayudarte a escribir código más conciso y posiblemente menos propenso a errores, además de permitirte implementar ciertas características con mayor rapidez.

React es precisamente una librería de este tipo: se enfoca en proporcionar funcionalidades que te ayudan a crear interfaces de usuario interactivas y reactivas. De hecho, React va más allá de las interfaces web (es decir, sitios web cargados en navegadores). También puedes crear aplicaciones nativas para dispositivos móviles con React y React Native, que es otra librería que utiliza React por debajo. Los conceptos de React cubiertos en este libro siguen siendo aplicables independientemente de la plataforma de destino elegida. Sin embargo, los ejemplos se centrarán en React para navegadores web. No obstante, sin importar a qué plataforma te dirijas, crear interfaces de usuario interactivas solo con JavaScript puede volverse rápidamente muy complejo y abrumador.

---

### Sección 3: El problema con “Vanilla JavaScript”

*Vanilla JavaScript* es un término comúnmente utilizado en el desarrollo web para referirse a JavaScript sin ningún framework ni librería. Esto significa que escribes todo el código JavaScript por tu cuenta, sin recurrir a ninguna librería o framework que proporcione funcionalidades de utilidad adicionales. Cuando trabajas con JavaScript puro, evitas especialmente el uso de grandes frameworks o librerías de frontend como React o Angular.

Usar JavaScript puro generalmente tiene la ventaja de que los visitantes de un sitio web tienen que descargar menos código JavaScript (ya que los frameworks y librerías principales suelen tener un tamaño considerable y pueden agregar fácilmente más de 50 KB de código JavaScript adicional que debe descargarse).

La desventaja de depender de JavaScript puro es que tú, como desarrollador, debes implementar todas las funcionalidades desde cero por tu cuenta. Esto puede ser propenso a errores y consumir mucho tiempo. Por lo tanto, especialmente las interfaces de usuario y los sitios web más complejos pueden volverse muy difíciles de gestionar rápidamente con JavaScript puro.

React simplifica la creación y gestión de dichas interfaces de usuario al pasar de un enfoque imperativo a uno declarativo. Aunque esta es una frase atractiva, puede resultar difícil de comprender si no has trabajado antes con React o frameworks similares. Para entender la idea detrás de los "enfoques imperativos frente a declarativos" y por qué querrías usar React en lugar de solo JavaScript puro, es útil dar un paso atrás y evaluar cómo funciona JavaScript puro.

Veamos un breve fragmento de código que muestra cómo podrías manejar las siguientes acciones de la interfaz de usuario con JavaScript puro:
1. Añadir un escuchador de eventos (*event listener*) a un botón para escuchar eventos de clic.
2. Reemplazar el texto de un párrafo con un nuevo texto una vez que se haga clic en el botón.

```javascript
const buttonElement = document.querySelector('button'); 
const paragraphElement = document.querySelector('p'); 

function updateTextHandler() { 
  paragraphElement.textContent = 'Text was changed!'; 
} 

buttonElement.addEventListener('click', updateTextHandler);
```

Este ejemplo se mantiene deliberadamente simple, por lo que probablemente no parezca abrumador. Es solo un ejemplo básico para mostrar cómo se escribe generalmente el código con JavaScript puro (más adelante se analizará un ejemplo más complejo). Pero aunque este ejemplo sea sencillo de asimilar, trabajar con JavaScript puro alcanzará rápidamente sus límites en interfaces de usuario ricas en funciones, y el código para manejar adecuadamente diversas interacciones de los usuarios también se volverá más complejo. El código puede crecer significativamente en poco tiempo, por lo que mantenerlo puede convertirse en un verdadero desafío.

En el ejemplo anterior, el código está escrito con JavaScript puro y, en consecuencia, de forma **imperativa**. Esto significa que escribes instrucción tras instrucción y describes detalladamente cada paso que debe darse.

El código mostrado anteriormente se podría traducir en estas instrucciones más legibles para los humanos:
1. Buscar un elemento HTML de tipo `button` para obtener una referencia al primer botón de la página.
2. Crear una constante (es decir, un contenedor de datos) llamada `buttonElement` que almacene la referencia a ese botón.
3. Repetir el paso 1, pero obteniendo una referencia al primer elemento que sea de tipo `p`.
4. Almacenar la referencia del elemento de párrafo en una constante llamada `paragraphElement`.
5. Añadir un escuchador de eventos a `buttonElement` que escuche los eventos de clic (`click`) y active la función `updateTextHandler` cada vez que ocurra dicho clic.
6. Dentro de la función `updateTextHandler`, utilizar `paragraphElement` para establecer su `textContent` a `"Text was changed!"`.

¿Ves cómo cada paso que debe realizarse está claramente definido y escrito en el código?

Esto no debería ser una sorpresa, porque así es como funcionan la mayoría de los lenguajes de programación: defines una serie de pasos que deben ejecutarse en orden. Es un enfoque muy lógico porque el orden de ejecución del código no debería ser aleatorio ni impredecible.

Sin embargo, al trabajar con interfaces de usuario, este enfoque imperativo no es el ideal. De hecho, puede volverse engorroso rápidamente porque, como desarrollador, debes añadir una gran cantidad de instrucciones que, a pesar de aportar poco valor directo de negocio, no pueden omitirse. Necesitas escribir todas las instrucciones del Modelo de Objetos del Documento (*Document Object Model* o DOM) que permiten a tu código interactuar con elementos, agregar elementos, manipularlos, etc.

Tu lógica de negocio principal (por ejemplo, derivar y definir el texto real que debe mostrarse después de un clic) a menudo constituye solo una pequeña fracción del código total. Al controlar y manipular interfaces de usuario web con JavaScript, una gran parte (a menudo la mayoría) de tu código está compuesta frecuentemente por instrucciones del DOM, escuchadores de eventos, operaciones con elementos HTML y gestión del estado de la interfaz de usuario.

Como resultado, terminas describiendo técnicamente todos los pasos necesarios para interactuar con la interfaz de usuario, además de todos los pasos necesarios para derivar los datos de salida (es decir, el estado final deseado de la interfaz de usuario).

> [!NOTE]
> Este libro asume que estás familiarizado con el DOM. En pocas palabras, el DOM es el "puente" entre tu código JavaScript y el código HTML del sitio web con el que deseas interactuar. A través de la API integrada del DOM, JavaScript puede crear, insertar, manipular, eliminar y leer elementos HTML y su contenido.
> Puedes aprender más sobre el DOM en este artículo: [https://academind.com/tutorials/what-is-the-dom](https://academind.com/tutorials/what-is-the-dom).

Las interfaces web modernas son a menudo bastante complejas, con una gran cantidad de interactividad detrás de escena. Es posible que tu sitio web deba escuchar la entrada del usuario en un campo de texto, enviar esos datos a un servidor para validarlos, mostrar un mensaje de retroalimentación de validación en la pantalla y desplegar una ventana modal de error si se envían datos incorrectos.

El ejemplo del clic en el botón no es un caso complejo en general, pero el código de JavaScript puro para implementar un escenario real con estas características puede resultar abrumador. Terminas con múltiples operaciones de selección, inserción y manipulación del DOM, así como varias líneas de código que no hacen nada más que gestionar escuchadores de eventos. Además, mantener el DOM actualizado sin introducir errores puede ser una pesadilla, ya que debes asegurarte de actualizar el elemento del DOM correcto con el valor correcto en el momento adecuado.

> [!NOTE]
> El código completo y funcional se puede encontrar en GitHub en: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/01-what-is-react/examples/example-1/vanilla-javascript](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/01-what-is-react/examples/example-1/vanilla-javascript).

Si observas el código de JavaScript en el repositorio enlazado, probablemente podrás imaginar cómo se vería una interfaz de usuario más compleja.

**Figura 1.1**: Un archivo de código JavaScript de ejemplo que contiene más de 100 líneas de código para una interfaz de usuario bastante trivial.

Este archivo de JavaScript de ejemplo contiene aproximadamente 110 líneas de código. Incluso después de minificarlo (minificar significa que el código se acorta automáticamente, por ejemplo, reemplazando nombres de variables largos por otros más cortos y eliminando espacios en blanco redundantes; en este caso, mediante [https://www.toptal.com/developers/javascript-minifier](https://www.toptal.com/developers/javascript-minifier)) y dividir el código en varias líneas después (para contar las líneas de código brutas), todavía tiene alrededor de 80 líneas de código. Son 80 líneas completas de código para una interfaz de usuario simple con solo funcionalidad básica. La lógica de negocio real (es decir, validar entradas, determinar si se deben mostrar ventanas superpuestas y definir el texto de salida) solo representa una pequeña fracción de la base de código total: entre 20 y 30 líneas de código en este caso (alrededor de 20 después de minificar).

Eso es aproximadamente un **75% del código dedicado exclusivamente a la interacción pura con el DOM**, la gestión del estado del DOM y tareas repetitivas similares.

Como puedes ver con estos ejemplos y números, controlar todos los elementos de la interfaz de usuario y sus diferentes estados (por ejemplo, si un cuadro de información es visible o no) es una tarea desafiante, e intentar crear tales interfaces solo con JavaScript a menudo conduce a un código complejo que incluso puede contener errores.

Por eso el enfoque imperativo, en el que debes definir y escribir cada paso individual, tiene sus límites en situaciones como esta. Esta es la razón por la que React proporciona funcionalidades de utilidad que te permiten escribir código de manera diferente: con un **enfoque declarativo**.

> [!NOTE]
> Este no es un artículo científico y el ejemplo anterior no pretende ser un estudio riguroso. Dependiendo de cómo cuentes las líneas y de qué tipo de código consideres "lógica de negocio principal", obtendrás porcentajes más altos o más bajos. Sin embargo, el mensaje clave no cambia: una gran cantidad de código (en este caso, la mayor parte) trata sobre el DOM y su manipulación, no sobre la lógica real que define tu sitio web y sus características clave.

---

### Sección 4: React y el código declarativo

Volviendo al primer fragmento de código simple mostrado anteriormente, aquí está ese mismo código, esta vez utilizando React:

```javascript
import { useState } from 'react'; 

function App() { 
  const [outputText, setOutputText] = useState('Initial text'); 

  function updateTextHandler() { 
    setOutputText('Text was changed!'); 
  } 

  return ( 
    <> 
      <button onClick={updateTextHandler}> 
        Click to change text 
      </button> 
      <p>{outputText}</p> 
    </> 
  ); 
}
```

Este fragmento realiza las mismas operaciones que el primero hecho solo con JavaScript puro:
1. Añadir un escuchador de eventos a un botón para escuchar eventos de clic (ahora con sintaxis específica de React: `onClick={…}`).
2. Reemplazar el texto de un párrafo con un nuevo texto una vez que ocurre el clic en el botón.

Sin embargo, este código se ve totalmente diferente: parece una mezcla de JavaScript y HTML. De hecho, React utiliza una extensión de sintaxis llamada **JSX** (es decir, JavaScript extendido para incluir una sintaxis similar a XML). Por el momento, basta con entender que este código JSX funcionará gracias a un paso de preprocesamiento (o transpilación) que forma parte del flujo de trabajo de construcción de todo proyecto de React.

El preprocesamiento significa que ciertas herramientas, que forman parte de los proyectos de React, analizan y transforman el código antes de que sea desplegado. Esto permite usar una sintaxis exclusiva para el desarrollo como JSX, que no funcionaría en el navegador y por esa razón se transforma a JavaScript estándar antes del despliegue. (Obtendrás una introducción detallada a JSX en el Capítulo 2, *Entendiendo los Componentes de React y JSX*).

Además, el fragmento anterior contiene una característica específica de React: el **Estado (*State*)**. El estado se discutirá con mayor detalle más adelante en el libro (el Capítulo 4, *Trabajando con Eventos y Estado*, se centrará en el manejo de eventos y estados con React). Por el momento, puedes pensar en este estado como una variable que, al cambiar, hace que React actualice automáticamente la interfaz de usuario en el navegador.

Lo que ves en el ejemplo anterior es el **"enfoque declarativo"** utilizado por React: escribes tu lógica de JavaScript (por ejemplo, funciones que eventualmente deberían ejecutarse) y combinas esa lógica con el código HTML que debería activarla o que se ve afectado por ella. No escribes las instrucciones para seleccionar ciertos elementos del DOM o cambiar el contenido de texto de algunos elementos. En cambio, con React y JSX, te enfocas en tu lógica de negocio en JavaScript y defines la salida HTML deseada que finalmente se debería alcanzar. Esta salida puede, y típicamente contendrá, valores dinámicos que se derivan dentro de tu código JavaScript principal.

En el ejemplo anterior, `outputText` es un estado gestionado por React. En el código, la función `updateTextHandler` se activa al hacer clic, y el valor del estado `outputText` se actualiza a una nueva cadena (`'Text was changed!'`) con la ayuda de la función `setOutputText`. Los detalles exactos de lo que sucede aquí se explorarán en el Capítulo 4.

La idea general es que el valor del estado cambia y, dado que se está referenciando en el último párrafo (`<p>{outputText}</p>`), React muestra el valor actual del estado en ese lugar específico del DOM real (y, por lo tanto, en la página web real). React mantendrá actualizado el párrafo y, por ende, cada vez que `outputText` cambie, React seleccionará este elemento de párrafo nuevamente y actualizará su `textContent` de forma automática.

Este es el enfoque declarativo en acción. Como desarrollador, no necesitas preocuparte por los detalles técnicos (por ejemplo, seleccionar el párrafo y actualizar su `textContent`). En su lugar, delegas este trabajo a React. Solo necesitas concentrarte en los estados finales deseados donde el objetivo es simplemente mostrar el valor actual de `outputText` en un lugar específico de la página. El trabajo de React es hacer todo lo que ocurre "tras bambalinas" para llegar a ese resultado.

Resulta que este fragmento de código no es más corto que el de JavaScript puro; de hecho, es incluso un poco más largo. Pero eso ocurre únicamente porque este primer fragmento se mantuvo deliberadamente simple y conciso. En tales casos, React agrega un poco de código base extra. Si esa fuera toda tu interfaz de usuario, usar React no tendría mucho sentido. Nuevamente, este fragmento fue elegido porque nos permite ver las diferencias de un vistazo. Las cosas cambian si observas el ejemplo más complejo de JavaScript puro anterior y lo comparas con su alternativa en React.

> [!NOTE]
> El código referenciado se puede encontrar en GitHub en: [vanilla-javascript](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/01-what-is-react/examples/example-1/vanilla-javascript) y [reactjs](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/01-what-is-react/examples/example-1/reactjs), respectivamente.

**Figura 1.2**: El fragmento de código anterior ahora implementado mediante React.

Todavía no es corto porque todo el código JSX (es decir, la salida HTML) está incluido dentro del archivo JavaScript. Si ignoras prácticamente todo el lado derecho (ya que el HTML tampoco era parte de los archivos de JavaScript puro), el código de React se vuelve mucho más conciso. Sin embargo, lo más importante es que si observas detenidamente todo el código de React (también en el primer fragmento más corto), notarás que **no hay absolutamente ninguna operación para seleccionar elementos del DOM, crear o insertar elementos en el DOM, o editar elementos del DOM**.

Esta es la idea central de React. No escribes todos los pasos e instrucciones individuales; en su lugar, te enfocas en la "visión general" y en los estados finales deseados del contenido de tu página. Con React, puedes fusionar tu código JavaScript y tu marcado sin tener que lidiar con las instrucciones de bajo nivel para interactuar con el DOM, como seleccionar elementos mediante `document.getElementById()` u operaciones similares.

El uso de este enfoque declarativo en lugar del enfoque imperativo con JavaScript puro te permite a ti, como desarrollador, concentrarte en tu lógica de negocio principal y en los diferentes estados de tu código HTML. No necesitas definir todos los pasos individuales que deben tomarse (como "agregar un escuchador de eventos", "seleccionar un párrafo", etc.), y esto simplifica enormemente el desarrollo de interfaces de usuario complejas.

> [!NOTE]
> Vale la pena enfatizar que React no es una gran solución si estás trabajando en una interfaz de usuario sumamente simple. Si puedes resolver un problema con unas pocas líneas de código de JavaScript puro, probablemente no haya una razón de peso para integrar React en el proyecto.

Al ver el código de React por primera vez, puede parecer muy extraño y poco familiar. No es a lo que estás acostumbrado en JavaScript. Aun así, sigue siendo JavaScript, solo que enriquecido con la funcionalidad de JSX y varias utilidades específicas de React (como el estado). Puede resultar menos confuso si recuerdas que normalmente defines tu interfaz de usuario (es decir, tu contenido y su estructura) con HTML. Allí tampoco escribes instrucciones paso a paso, sino que creas una estructura de árbol anidada con etiquetas HTML. Expresas tu contenido, el significado de los diferentes elementos y la jerarquía de tu interfaz de usuario mediante el uso de diferentes elementos HTML y etiquetas anidadas.

Si tienes esto en cuenta, el enfoque "tradicional" (con JavaScript puro) de manipular la interfaz de usuario debería parecer bastante extraño. ¿Por qué empezarías a definir instrucciones de bajo nivel como "insertar un elemento de párrafo debajo de este botón y establecer su texto en `<algún texto>`" si no haces eso en HTML en absoluto? React, al final, recupera esa sintaxis similar a HTML, que es mucho más conveniente a la hora de definir contenido y estructura. Con React, puedes escribir código JavaScript dinámico codo a codo con el código de la interfaz de usuario (es decir, el código HTML) que se ve afectado por él o que está relacionado con él.

---

### Sección 5: Cómo React manipula el DOM

Como se mencionó anteriormente, al escribir código de React, normalmente lo escribes mezclando HTML con código JavaScript mediante la extensión de sintaxis JSX.

Es importante señalar que el código JSX no se ejecuta de esta forma directamente en los navegadores. En su lugar, debe preprocesarse antes del despliegue. El código JSX debe transformarse en código JavaScript estándar antes de enviarse a los navegadores. El próximo capítulo analizará en detalle JSX y en qué se transforma exactamente. Por el momento, simplemente ten presente que el código JSX debe ser transformado.

Sin embargo, vale la pena saber que el código resultante de transformar JSX tampoco contendrá instrucciones directas del DOM. En su lugar, el código transformado ejecutará varios métodos y funciones de utilidad que están integrados en React (es decir, aquellos que proporciona el paquete de React que debe añadirse a todo proyecto). Internamente, React crea una estructura abstracta en forma de árbol similar al DOM llamada **DOM virtual** (*Virtual DOM*) que refleja el estado actual de la interfaz de usuario. Este libro examina detalladamente este DOM virtual abstracto y cómo funciona React internamente en el Capítulo 10, *Tras Bambalinas de React y Oportunidades de Optimización*.

Es por esto que React (la librería) divide su lógica principal en dos paquetes principales:
1. El paquete principal `react`.
2. El paquete `react-dom`.

El paquete principal `react` es la librería de JavaScript de terceros que debe importarse a un proyecto para utilizar las características de React (como JSX o el estado). Es este paquete el que crea el DOM virtual y deduce el estado actual de la interfaz de usuario. Pero también necesitas el paquete `react-dom` en tu proyecto si deseas manipular el DOM del navegador con React.

El paquete `react-dom`, específicamente la parte `react-dom/client`, actúa como un "puente de traducción" entre tu código de React, el DOM virtual generado internamente y el navegador con su DOM real que necesita ser actualizado. Es el paquete `react-dom` el que generará las instrucciones reales del DOM que seleccionarán, actualizarán, eliminarán y crearán elementos en el DOM.

Esta división existe porque también puedes usar React en otros entornos. Una alternativa muy popular y conocida al DOM (es decir, al navegador) es **React Native**, que permite a los desarrolladores crear aplicaciones móviles nativas con la ayuda de React. Con React Native, también incluyes el paquete `react` en tu proyecto, pero en lugar de `react-dom`, utilizarías el paquete `react-native`. En este libro, "React" se refiere tanto al paquete `react` como a los paquetes puente (como `react-dom`).

> [!NOTE]
> Como se mencionó anteriormente, este libro se centra en React en sí mismo. Los conceptos explicados aquí se aplican tanto a navegadores web y sitios web como a dispositivos móviles. Sin embargo, todos los ejemplos se centrarán en la web y en `react-dom`, ya que esto evita introducir complejidad adicional.

---

### Sección 6: Introducción a las SPAs (Single-Page Applications)

React se puede utilizar para simplificar la creación de interfaces de usuario complejas y existen dos formas principales de hacerlo:
1. Gestionar partes específicas de un sitio web (por ejemplo, una caja de chat en la esquina inferior izquierda).
2. Gestionar la página completa y todas las interacciones de usuario que ocurren en ella.

Ambos enfoques son viables, pero el escenario más popular y común es el segundo: usar React para gestionar toda la página web en lugar de solo fragmentos aislados. Este enfoque es más popular porque la mayoría de los sitios web con interfaces complejas tienen múltiples elementos dinámicos en sus páginas. La complejidad aumentaría si comenzaras a usar React para algunas partes del sitio web sin usarlo en otras áreas. Por esta razón, es muy común gestionar el sitio web completo con React.

Esto ni siquiera se detiene después de usar React en una página específica del sitio. De hecho, React se puede utilizar para manejar cambios en la ruta de la URL y actualizar las partes de la página que deben modificarse para reflejar la nueva página que se debe cargar. Esta funcionalidad se llama **enrutamiento** (*routing*) y los paquetes de terceros como `react-router-dom` (consulta el Capítulo 13, *Aplicaciones Multipágina con React Router*), que se integran con React, te permiten crear un sitio web donde toda la interfaz de usuario se controla a través de React.

Un sitio web que no solo utiliza React para partes de sus páginas, sino para todas las subpáginas y para el enrutamiento, a menudo se construye como una **SPA** (*Single-Page Application* o Aplicación de Página Única) porque es común crear proyectos de React que contienen un único archivo HTML (típicamente llamado `index.html`), que se utiliza para cargar inicialmente el código JavaScript de React. A partir de ese momento, la librería de React y tu código toman el control absoluto de la interfaz de usuario. Esto significa que toda la interfaz se crea y administra mediante JavaScript a través de React y tu código.

Dicho esto, también se está volviendo cada vez más popular crear aplicaciones full-stack con React, donde el código del frontend y del backend se fusionan. Frameworks modernos de React como Next.js simplifican el proceso de construcción de tales aplicaciones web. Si bien los conceptos centrales son los mismos sin importar qué tipo de aplicación se construya, este libro explorará el desarrollo de aplicaciones full-stack con React en mayor detalle en el Capítulo 15 (*Renderizado en el Servidor y Desarrollo Fullstack con Next.js*), Capítulo 16 (*React Server Components y Server Actions*) y Capítulo 17 (*Entendiendo React Suspense y el Hook use()*).

En última instancia, este libro te prepara para trabajar con React en todo tipo de proyectos, ya que los bloques de construcción fundamentales y los conceptos clave son siempre los mismos.

---

### Sección 7: Creación de un proyecto React con Vite

Para trabajar con React, el primer paso es la creación de un proyecto. La documentación oficial recomienda utilizar un framework como Next.js. Pero aunque esto puede tener sentido para aplicaciones web complejas, resulta abrumador para comenzar con React y para explorar sus conceptos fundamentales. Next.js y otros frameworks introducen sus propios conceptos y sintaxis. Como resultado, aprender React puede volverse frustrante rápidamente, ya que puede ser difícil distinguir las características propias de React de las características del framework. Además, no todas las aplicaciones de React necesitan construirse como aplicaciones web full-stack; por consiguiente, usar un framework como Next.js podría añadir una complejidad innecesaria.

Por esta razón, los proyectos de React basados en **Vite** han surgido como una alternativa muy popular. Vite es una herramienta de compilación y desarrollo de código abierto que se puede utilizar para crear y ejecutar proyectos de desarrollo web basados en todo tipo de librerías y frameworks (React es solo una de las muchas opciones).

Vite crea proyectos que vienen con un proceso de compilación preconfigurado e integrado que, en el caso de los proyectos de React, se encarga de la transpilación del código JSX. También proporciona un servidor web de desarrollo que se ejecuta localmente en tu sistema y te permite previsualizar la aplicación React mientras trabajas en ella.

Necesitas una configuración de proyecto como esta porque los proyectos de React normalmente usan características como JSX, que no funcionarían en el navegador sin una transformación previa del código. Por lo tanto, como se mencionó anteriormente, se requiere un paso de preprocesamiento.

Para crear un proyecto con Vite, debes tener instalado **Node.js** (preferiblemente la versión más reciente o la versión LTS más reciente). Puedes obtener el instalador oficial de Node.js para todos los sistemas operativos en [https://nodejs.org/](https://nodejs.org/). Una vez que hayas instalado Node.js, también obtendrás acceso al comando integrado `npm`, que puedes usar para utilizar el paquete Vite para crear un nuevo proyecto de React.

Puedes ejecutar el siguiente comando dentro de tu símbolo del sistema (Windows), bash (Linux) o terminal (macOS). Solo asegúrate de navegar (mediante `cd`) a la carpeta en la que deseas crear tu nuevo proyecto:

```bash
npm create vite@latest my-react-project
```

Una vez ejecutado, este comando te pedirá que elijas el framework o librería que deseas utilizar para este nuevo proyecto. Debes seleccionar **React** y luego **JavaScript**.

Este comando creará una nueva subcarpeta con una estructura básica de proyecto de React (es decir, con varios archivos y carpetas) en el lugar donde lo ejecutaste. Debes ejecutarlo en una ruta de tu sistema donde tengas permisos completos de lectura y escritura y donde no haya conflictos con archivos del sistema u otros proyectos.

Vale la pena señalar que el comando de creación de proyectos no instala las dependencias requeridas, como los paquetes de la librería React. Por esa razón, debes navegar a la carpeta creada en tu terminal o símbolo del sistema (mediante `cd my-react-project`) e instalar estos paquetes ejecutando el siguiente comando:

```bash
npm install
```

Una vez que la instalación finalice con éxito, el proceso de configuración del proyecto estará completo.

Para ver la aplicación React creada, puedes iniciar un servidor de desarrollo en tu máquina a través de este comando:

```bash
npm run dev
```

Esto invoca un script proporcionado por Vite, que levantará un servidor web en ejecución local que preprocesa, compila y aloja tu SPA impulsada por React (por defecto en `localhost:5173`). Por lo tanto, mientras trabajas en el código, normalmente mantienes este servidor de desarrollo en funcionamiento, ya que te permite previsualizar y probar los cambios en el código.

Lo mejor de todo es que este servidor de desarrollo local actualizará automáticamente el sitio web cada vez que guardes cambios en el código, lo que te permitirá previsualizarlos de forma casi instantánea.

Puedes detener este servidor siempre que hayas terminado por el día presionando `Ctrl + C` en la terminal o símbolo del sistema donde ejecutaste `npm run dev`.

Cuando estés listo para volver a trabajar en el proyecto, puedes reiniciar el servidor mediante `npm run dev`.

> [!NOTE]
> En caso de que encuentres algún problema al crear un proyecto de React, también puedes descargar y utilizar el siguiente proyecto inicial: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/01-what-is-react/react-starting-project](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/01-what-is-react/react-starting-project). Es un proyecto creado mediante Vite que se puede utilizar exactamente de la misma manera que si se hubiera creado con el comando anterior.
> Al usar este proyecto inicial (o, de hecho, cualquier instantánea de código alojada en GitHub perteneciente a este libro), debes ejecutar `npm install` en la carpeta del proyecto antes de ejecutar `npm run dev`.

La estructura exacta del proyecto (es decir, los nombres de archivos y carpetas) puede variar con el tiempo, pero generalmente todo nuevo proyecto de React basado en Vite contiene algunos archivos y carpetas clave:
- Una carpeta `src/`, que contiene los archivos de código fuente principales del proyecto:
  - Un archivo `main.jsx`, que es el archivo de script de entrada principal que se ejecutará primero.
  - Un archivo `App.jsx`, que contiene el componente raíz de la aplicación (aprenderás más sobre componentes en el próximo capítulo).
  - Varios archivos de estilos (`*.css`), que son importados por los archivos de JavaScript.
  - Una carpeta `assets/` que se puede utilizar para almacenar imágenes u otros recursos que se usarán en tu código de React.
- Una carpeta `public/`, que contiene archivos estáticos que formarán parte del sitio web final (por ejemplo, un favicon).
- Un archivo `index.html`, que es la única página HTML de este sitio web.
- `package.json` y `package-lock.json` son archivos que listan y definen las dependencias de terceros de tu proyecto:
  - Dependencias de producción como `react` o `react-dom`.
  - Dependencias de desarrollo como `eslint` para comprobaciones automáticas de calidad del código.
- Otros archivos de configuración del proyecto (por ejemplo, `.gitignore` para gestionar el seguimiento de archivos en Git).
- Una carpeta `node_modules/`, que contiene el código real de los paquetes de terceros instalados.

Es importante destacar que `App.jsx` y `main.jsx` utilizan `.jsx` como extensión de archivo, no `.js`. Esta es una extensión requerida por Vite para archivos que no solo contienen JavaScript estándar sino también código JSX. Cuando trabajes en un proyecto de Vite, la mayoría de tus archivos de proyecto usarán consecuentemente `.jsx` como extensión.

Casi todo el código específico de React se escribirá en el archivo `App.jsx` o en archivos de componentes personalizados que se añadirán al proyecto. Exploraremos los componentes en el próximo capítulo.

> [!NOTE]
> `package.json` es el archivo en el que realmente gestionas los paquetes y sus versiones. `package-lock.json` se crea automáticamente (por Node.js) y fija las versiones exactas de dependencias y subdependencias, mientras que `package.json` solo especifica rangos de versiones. Puedes aprender más sobre estos archivos y versiones de paquetes en [https://docs.npmjs.com/](https://docs.npmjs.com/).
> El código de las dependencias del proyecto se almacena en la carpeta `node_modules`. Esta carpeta puede llegar a ser muy grande, ya que contiene el código de todos los paquetes instalados y sus dependencias. Por esa razón, normalmente no se incluye cuando los proyectos se comparten con otros desarrolladores o se suben a GitHub. El archivo `package.json` es todo lo que necesitas: al ejecutar `npm install`, la carpeta `node_modules` se recreará localmente.

---

### Sección 8: Resumen y puntos clave

- React es una librería, aunque en realidad es una combinación de dos paquetes principales: `react` y `react-dom`.
- Aunque es posible construir interfaces de usuario no triviales sin React, simplemente usar JavaScript puro para hacerlo puede ser engorroso, propenso a errores y difícil de mantener.
- React simplifica la creación de interfaces de usuario complejas al proporcionar una forma **declarativa** de definir los estados finales deseados de la interfaz.
- **Declarativo** significa que defines el contenido y la estructura objetivo de la interfaz, combinados con diferentes estados (por ejemplo, "¿está abierto o cerrado un modal?"), y dejas en manos de React la tarea de determinar las instrucciones adecuadas del DOM.
- El paquete `react` en sí deduce los estados de la interfaz de usuario y gestiona un DOM virtual. Es un "puente", como `react-dom` o `react-native`, el que traduce este DOM virtual en instrucciones reales para la interfaz de usuario (DOM).
- Con React puedes construir **SPAs**, lo que significa que React se utiliza para controlar toda la interfaz de usuario en todas las páginas, así como el enrutamiento entre ellas.
- También puedes usar React en combinación con frameworks como Next.js para crear aplicaciones web **full-stack** donde el código del servidor y del cliente están conectados.
- Los proyectos de React se pueden crear con la ayuda del paquete **Vite**, que proporciona una carpeta de proyecto configurada y un servidor de desarrollo con vista previa en vivo.

---

### Sección 9: ¿Qué sigue?

En este punto, deberías tener una comprensión básica de qué es React y por qué podrías considerar usarlo, especialmente para construir interfaces de usuario no triviales. Aprendiste cómo crear nuevos proyectos de React con Vite y ahora estás listo para profundizar en React y en las características clave reales que ofrece.

En el próximo capítulo aprenderás sobre un concepto llamado **componentes**, que son los bloques de construcción fundamentales de las aplicaciones de React. Aprenderás cómo se usan los componentes para componer interfaces de usuario y por qué son necesarios en primer lugar. El próximo capítulo también profundizará en JSX y explorará cómo se transforma en código JavaScript normal y qué tipo de código podrías escribir como alternativa a JSX.

---

### Sección 10: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con las respuestas de ejemplo que se encuentran aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/01-what-is-react/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/01-what-is-react/exercises/questions-answers.md).

1. ¿Qué es React?
2. ¿Qué ventaja ofrece React sobre los proyectos con JavaScript puro?
3. ¿Cuál es la diferencia entre código imperativo y declarativo?
4. ¿Qué es una Aplicación de Página Única (*Single-Page Application* o SPA)?
5. ¿Cómo puedes crear nuevos proyectos de React y por qué necesitas una configuración de proyecto como esta?
