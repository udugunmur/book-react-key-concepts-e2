# Parte 1: Fundamentos de React

## Capítulo 2: Entendiendo los Componentes de React y JSX

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Definir qué son exactamente los componentes.
- Construir y utilizar componentes de manera eficaz.
- Utilizar convenciones de nomenclatura comunes y patrones de código.
- Describir la relación entre los componentes y JSX.
- Escribir código JSX y entender por qué se utiliza.
- Escribir componentes de React sin utilizar código JSX.
- Escribir tus primeras aplicaciones de React.

---

### Sección 1: Introducción

En el capítulo anterior, aprendiste sobre React en general, qué es y por qué deberías considerar usarlo para construir interfaces de usuario. También aprendiste cómo crear proyectos de React con la ayuda de Vite, ejecutando `npm create vite@latest <nombre-de-tu-proyecto>`.

En este capítulo, aprenderás sobre uno de los conceptos y bloques de construcción más importantes de React. Aprenderás que los componentes son bloques de construcción reutilizables que se utilizan para construir interfaces de usuario. Además, se analizará el código JSX en mayor detalle para que puedas utilizar el concepto de componentes y JSX para construir tus primeras aplicaciones básicas de React.

---

### Sección 2: ¿Qué son los componentes?

Un concepto clave de React es el uso de los llamados **componentes**. Los componentes son bloques de construcción reutilizables que se combinan para componer la interfaz de usuario final. Por ejemplo, un sitio web básico podría estar compuesto por una barra lateral que incluye elementos de navegación y una sección principal que incluye elementos para agregar y ver tareas.

**Figura 2.1**: Una pantalla de ejemplo de gestión de tareas con barra lateral y área principal.

Si observas esta página de ejemplo, es posible que puedas identificar varios bloques de construcción (es decir, componentes). Algunos de estos componentes incluso se reutilizan:
- La barra lateral y sus elementos de navegación.
- El área principal de la página.
- En el área principal, el encabezado con el título y la fecha de vencimiento.
- Un formulario para agregar tareas.
- Una lista de tareas.

Ten en cuenta que algunos componentes están anidados dentro de otros componentes; es decir, los componentes también están formados por otros componentes. Esa es una característica clave de React y librerías similares.

#### ¿Por qué componentes?
Sin importar qué página web mires, todas están formadas por bloques de construcción como estos. No es un concepto o idea exclusivo de React. De hecho, el propio HTML "piensa" en componentes si lo miras de cerca. Tienes elementos como `<img>`, `<header>`, `<nav>`, etc., y combinas estos elementos para describir y estructurar el contenido de tu sitio web.

Pero React adopta esta idea de dividir una página web en bloques de construcción reutilizables porque es un enfoque que permite a los desarrolladores trabajar en fragmentos de código pequeños y manejables. Es más fácil y más mantenible que trabajar en un único archivo de código HTML (o React) gigante.

Es por eso que otras librerías —tanto librerías de frontend como React o Angular, así como librerías de backend y motores de plantillas como EJS (*Embedded JavaScript templates*)— también adoptan los componentes (aunque los nombres pueden variar, también encontrarás "parciales" o "includes" como nombres comunes).

> [!NOTE]
> EJS es un popular motor de plantillas para JavaScript. Es especialmente popular para el desarrollo web en el backend con Node.js.

Al trabajar con React, es especialmente importante mantener tu código manejable y trabajar con componentes pequeños y reutilizables porque los componentes de React no son solo colecciones de código HTML. En su lugar, un componente de React también encapsula lógica de JavaScript y, a menudo, también estilos CSS. Para interfaces de usuario complejas, la combinación de marcado (JSX), lógica (JavaScript) y estilos (CSS) podría conducir rápidamente a grandes bloques de código, dificultando el mantenimiento de dicho código. Piensa en un archivo HTML grande que también incluya código JavaScript y CSS. Trabajar en un archivo de código de ese tipo no sería muy agradable.

En resumen, al trabajar en un proyecto de React, trabajarás con muchos componentes. Dividirás tu código en bloques de construcción pequeños y manejables y luego combinarás estos componentes para formar la interfaz de usuario general. Es una característica clave de React.

> [!NOTE]
> Al trabajar con React, debes adoptar esta idea de trabajar con componentes. Pero técnicamente, son opcionales. En teoría, podrías construir páginas web muy complejas con un solo componente. No sería muy divertido ni práctico, pero técnicamente sería posible sin ningún problema.

#### La anatomía de un componente
Los componentes son importantes. Pero, ¿cómo es exactamente un componente de React? ¿Cómo escribes componentes de React por tu cuenta?

Aquí tienes un componente de ejemplo:

```javascript
import { useState } from 'react'; 

function SubmitButton() { 
  const [isSubmitted, setIsSubmitted] = useState(false); 
  
  function handleSubmit() { 
    setIsSubmitted(true); 
  }; 
  
  return ( 
    <button onClick={handleSubmit}> 
      { isSubmitted ? 'Loading…' : 'Submit' } 
    </button> 
  ); 
}; 

export default SubmitButton;
```

Por lo general, almacenarías un fragmento de código como este en un archivo separado (por ejemplo, un archivo llamado `SubmitButton.jsx`, almacenado dentro de una carpeta `/components`, que a su vez reside en la carpeta `/src` de tu proyecto de React) y lo importarías en otros archivos de componentes que necesiten este componente. Se utiliza `.jsx` como extensión ya que el archivo contiene código JSX. Vite impone el uso de `.jsx` como extensión de archivo si estás escribiendo código JSX; no está permitido almacenar dicho código en archivos `.js` en proyectos de Vite (aunque podría funcionar en otras configuraciones de proyectos de React).

El siguiente componente importa el componente definido anteriormente y lo utiliza en su sentencia `return` para renderizar el componente `SubmitButton`:

```javascript
import SubmitButton from './submit-button.jsx'; 

function AuthForm() { 
  return ( 
    <form> 
      <input type="text" /> 
      <SubmitButton /> 
    </form> 
  ); 
}; 

export default AuthForm;
```

Las sentencias `import` que ves en estos ejemplos son sentencias de importación estándar de JavaScript. Teóricamente, en proyectos basados en Vite, podrías omitir la extensión del archivo (`.jsx` en este caso) en la sentencia `import`. Sin embargo, puede ser una buena idea incluir la extensión, ya que coincide con el estándar de JavaScript. No obstante, al importar desde paquetes de terceros (como `useState` del paquete `react`), no se agrega ninguna extensión de archivo: simplemente usas el nombre del paquete. `import` y `export` son palabras clave estándar de JavaScript que ayudan a dividir el código relacionado en múltiples archivos. Elementos como variables, constantes, clases o funciones se pueden exportar a través de `export` o `export default` para que luego puedan usarse en otros archivos después de importarlos allí.

> [!NOTE]
> Si el concepto de dividir el código en múltiples archivos y usar `import` y `export` es completamente nuevo para ti, es posible que desees consultar primero recursos básicos de JavaScript sobre este tema. Por ejemplo, MDN tiene un excelente artículo que explica los fundamentos, que puedes encontrar en [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules).

Por supuesto, los componentes mostrados en estos ejemplos están muy simplificados y también contienen características sobre las que aún no has aprendido (por ejemplo, `useState()`). Sin embargo, la idea general de tener bloques de construcción independientes que se puedan combinar debería quedar clara.

Al trabajar con React, existen dos formas alternativas de definir componentes:
- **Componentes basados en clases** (*Class-based components* o *class components*): Componentes definidos mediante la palabra clave `class`.
- **Componentes funcionales** (*Functional components* o *function components*): Componentes definidos mediante funciones regulares de JavaScript.

En todos los ejemplos cubiertos en este libro, los componentes se construyen como funciones de JavaScript. Como desarrollador de React, debes utilizar uno de estos dos enfoques, ya que React espera que los componentes sean funciones o clases.

> [!NOTE]
> Hasta finales de 2018, tenías que usar componentes basados en clases para ciertos tipos de tareas, específicamente para componentes que usaban estado internamente (el estado se cubrirá en el Capítulo 4, *Trabajando con Eventos y Estado*). Sin embargo, a finales de 2018 se introdujo un nuevo concepto: los **React Hooks**. Esto te permite realizar todas las operaciones y tareas con componentes funcionales. En consecuencia, aunque React todavía los admite, los componentes basados en clases están en desuso y no se tratan en este libro.

En los ejemplos anteriores, hay un par de cosas destacables adicionales:
- Las funciones de los componentes llevan nombres en mayúsculas (por ejemplo, `SubmitButton`).
- Dentro de las funciones de los componentes, se pueden definir otras funciones "internas" (por ejemplo, `handleSubmit`, escrita típicamente en camelCase).
- Las funciones de los componentes devuelven código similar a HTML (código JSX).
- Funcionalidades como `useState()` se pueden utilizar dentro de las funciones de los componentes.
- Las funciones de los componentes se exportan (a través de `export default`).
- Ciertas características (como `useState` o el componente personalizado `SubmitButton`) se importan mediante la palabra clave `import`.

Las siguientes secciones analizarán más de cerca estos diferentes conceptos que componen los componentes y su código.

#### ¿Qué son exactamente las funciones de componentes?
En React, los componentes son funciones (o clases, pero como se mencionó anteriormente, esas ya no son relevantes).

Una función es una construcción normal de JavaScript, no un concepto exclusivo de React. Es importante tener esto en cuenta. React es una librería de JavaScript y, por consiguiente, utiliza características de JavaScript (como funciones); React no es un lenguaje de programación nuevo.

Al trabajar con React, se pueden utilizar funciones regulares de JavaScript para encapsular código HTML (o, para ser más precisos, JSX) y la lógica de JavaScript que pertenece a ese código de marcado. Sin embargo, depende del código que escribas en una función si califica para ser tratada como un componente de React o no. Por ejemplo, en los fragmentos de código anteriores, la función `handleSubmit` también es una función regular de JavaScript, pero no es un componente de React. El siguiente ejemplo muestra otra función regular de JavaScript que no califica como un componente de React:

```javascript
function calculate(a, b) { 
  return {sum: a + b}; 
};
```

De hecho, una función será tratada como un componente y, por lo tanto, se podrá utilizar como un elemento HTML en el código JSX si devuelve un **valor renderizable** (típicamente código JSX). Esto es muy importante. Solo puedes usar una función como un componente de React en código JSX si es una función que devuelve algo que React pueda renderizar. Técnicamente, el valor devuelto no tiene por qué ser código JSX, pero en la mayoría de los casos lo será. Verás un ejemplo de código no JSX que se devuelve en el Capítulo 7, *Portales y Refs*.

En el fragmento de código donde se definieron las funciones llamadas `SubmitButton` y `AuthForm`, esas dos funciones calificaban como componentes de React porque ambas devolvían código JSX (que es código que React puede renderizar, haciéndolo renderizable). Una vez que una función califica como un componente de React, se puede usar como un elemento HTML dentro del código JSX, tal como `<SubmitButton />` se usó como un elemento HTML (autocerrado).

Al trabajar con JavaScript puro, normalmente llamas a las funciones para ejecutarlas. Con los componentes funcionales, eso es diferente. React llama a estas funciones en tu nombre y, por esa razón, como desarrollador, las usas como elementos HTML dentro de este código JSX.

> [!NOTE]
> Al referirse a valores renderizables, vale la pena señalar que, con diferencia, el tipo de valor más común devuelto o utilizado es, en efecto, código JSX, es decir, marcado definido mediante JSX. Esto tiene sentido porque con JSX puedes definir la estructura similar a HTML de tu contenido e interfaz de usuario.
> Pero además del marcado JSX, hay un par de otros valores clave que también califican como renderizables y que, por lo tanto, podrían ser devueltos por componentes personalizados (en lugar de código JSX). Cabe destacar que también puedes devolver cadenas de texto o números, así como matrices (*arrays*) que contengan elementos JSX, cadenas de texto o números.

---

### Sección 3: ¿Qué hace React con todos estos componentes?

Si sigues el rastro de todos los componentes y sus sentencias `import` y `export` hasta la parte superior, encontrarás una instrucción `root.render(...)` en el script de entrada principal del proyecto de React. Por lo general, este script de entrada principal se encuentra en el archivo `main.jsx`, ubicado en la carpeta `src/` del proyecto. Este método `render()`, que proporciona la librería React (para ser precisos, el paquete `react-dom`), toma un fragmento de código JSX y lo interpreta y ejecuta por ti.

El fragmento completo que encuentras en el archivo de entrada raíz (`main.jsx`) suele verse así:

```javascript
import React from 'react'; 
import ReactDOM from 'react-dom/client'; 
import './index.css'; 
import App from './App.jsx'; 

const root = ReactDOM.createRoot(document.getElementById('root')); 
root.render(<App />);
```

El código exacto que encuentres en tu nuevo proyecto de React puede verse ligeramente diferente.

Puede, por ejemplo, incluir un elemento adicional `<StrictMode>` que envuelve a `<App>`. `<StrictMode>` activa comprobaciones adicionales que pueden ayudar a detectar errores sutiles en tu código de React. Pero también puede provocar comportamientos confusos y mensajes de error inesperados, especialmente al experimentar con React o aprenderlo. Como este libro está principalmente interesado en cubrir las características centrales y los conceptos clave de React, no se utilizará `<StrictMode>`.

Aunque se omite aquí, el modo estricto se cubrirá en el Capítulo 10, *Tras Bambalinas de React y Oportunidades de Optimización*. Si deseas obtener más información al respecto ahora mismo, puedes profundizar en la documentación oficial: [https://react.dev/reference/react/StrictMode](https://react.dev/reference/react/StrictMode). Solo ten en cuenta que algunos de los efectos provocados por el modo estricto serán más fáciles de entender después de haber leído más partes de este libro.

Para seguir el libro sin confusiones, es una buena idea limpiar el archivo `main.jsx` recién creado para que se parezca al fragmento de código anterior.

El método `createRoot()` le indica a React que cree un nuevo punto de entrada, que se utilizará para inyectar la interfaz de usuario generada en el documento HTML real que se servirá a los visitantes del sitio web. Por lo tanto, el argumento pasado a `createRoot()` es un puntero a un elemento del DOM que se encuentra en `index.html`, la única página que se servirá a los visitantes del sitio web.

En muchos casos, se utiliza `document.getElementById('root')` como argumento. Este método integrado de JavaScript puro produce una referencia a un elemento del DOM que ya forma parte del documento `index.html`. Por lo tanto, como desarrollador, debes asegurarte de que dicho elemento con el valor de atributo `id` proporcionado (`root`, en este ejemplo) exista en el archivo HTML en el que se carga el script de la aplicación de React. En un proyecto de React predeterminado creado mediante `npm create vite@latest`, este será el caso: puedes encontrar un elemento `<div id="root">` en el archivo `index.html` en la carpeta raíz del proyecto.

Este archivo `index.html` es un archivo relativamente vacío que solo actúa como un contenedor para la aplicación de React. React solo necesita un punto de entrada (definido mediante `createRoot()`), que se utilizará para adjuntar la interfaz de usuario generada al sitio web mostrado. El archivo HTML y su contenido, como resultado, no definen directamente el contenido del sitio web. En cambio, el archivo solo sirve como punto de partida para la aplicación de React, lo que le permite a React tomar el control y gestionar la interfaz de usuario real.

Una vez definido el punto de entrada raíz, se puede llamar a un método llamado `render()` en el objeto raíz creado a través de `createRoot()`:

```javascript
root.render(<App />);
```

Este método `render()` le indica a React qué contenido (es decir, qué componente de React) se debe inyectar en ese punto de entrada raíz. En la mayoría de las aplicaciones de React, este es un componente llamado `App`. React generará entonces las instrucciones de manipulación del DOM apropiadas para reflejar el marcado definido mediante JSX en el componente `App` en la página web real.

Este componente `App` es una función de componente que se importa desde algún otro archivo. En un proyecto de React predeterminado, la función del componente `App` se define y exporta en un archivo `App.jsx`, que también se encuentra en la carpeta `src/`.

Este componente, que se pasa a `render()` (típicamente `<App />`), también se denomina **componente raíz** (*root component*) de la aplicación de React. Es el componente principal que se renderiza en el DOM. Todos los demás componentes están anidados en el código JSX de ese componente `App` o en el código JSX de componentes descendientes aún más anidados. Puedes pensar en todos estos componentes construyendo un **árbol de componentes** (*component tree*) que es evaluado por React y traducido en instrucciones reales de manipulación del DOM.

**Figura 2.2**: Los componentes anidados de React forman un árbol de componentes.

> [!NOTE]
> Como se mencionó en el capítulo anterior, React se puede utilizar en varias plataformas. Con el paquete `react-native`, podría usarse para crear aplicaciones móviles nativas para iOS y Android. El paquete `react-dom`, que proporciona el método `createRoot()` (y por lo tanto, implícitamente, el método `render()`), se enfoca en el navegador. Proporciona el "puente" entre las capacidades de React y las instrucciones del navegador que se requieren para dar vida a la interfaz de usuario (descrita a través de componentes JSX y React) en el navegador. Si compilas para diferentes plataformas, se requieren reemplazos para `ReactDOM.createRoot()` y `render()` (y, por supuesto, tales alternativas existen).

De cualquier manera, no importa si usas una función de componente como un elemento HTML dentro del código JSX de otros componentes o la usas como un elemento HTML que se pasa como argumento al método `render()`, React se encarga de interpretar y ejecutar la función de componente en tu nombre.

Por supuesto, este no es un concepto nuevo. En JavaScript, las funciones son objetos de primera clase, lo que significa que puedes pasar funciones como argumentos a otras funciones. Esto es básicamente lo que sucede aquí, solo que con el giro adicional de usar esta sintaxis JSX, que no es una característica predeterminada de JavaScript.

React ejecuta estas funciones de componentes por ti y traduce el código JSX devuelto en instrucciones del DOM. Para ser precisos, React recorre la estructura de marcado JSX devuelta y profundiza en cualquier otro componente personalizado que pueda usarse en ese código JSX hasta que termina con código JSX que solo está compuesto por elementos HTML nativos e integrados (técnicamente, no es realmente HTML, pero eso se discutirá más adelante en este capítulo).

Toma estos dos componentes como ejemplo:

```javascript
function Greeting() { 
  return <p>Welcome to this book!</p>; 
}; 

function App() { 
  return ( 
    <div> 
      <h2>Hello World!</h2> 
      <Greeting /> 
    </div> 
  ); 
}; 

const root = ReactDOM.createRoot(document.getElementById('root')); 
root.render(<App />);
```

El componente `App` utiliza el componente `Greeting` dentro de su código JSX. React recorrerá toda la estructura de marcado JSX y deducirá este código JSX final:

```javascript
root.render(( 
  <div> 
    <h2>Hello World!</h2> 
    <p>Welcome to this book!</p> 
  </div> 
), document.getElementById('root'));
```

Este código indicará a React y a ReactDOM que realicen las siguientes operaciones en el DOM:
1. Crear un elemento `<div>`.
2. Dentro de ese `<div>`, crear dos elementos secundarios: `<h2>` y `<p>`.
3. Establecer el contenido de texto del elemento `<h2>` en `'Hello World!'`.
4. Establecer el contenido de texto del elemento `<p>` en `'Welcome to this book!'`.
5. Insertar el `<div>`, con sus hijos, en el elemento del DOM preexistente que tiene el ID `'root'`.

Esto es un poco simplificado, pero puedes pensar en React manejando componentes y código JSX como se describió anteriormente.

> [!NOTE]
> En realidad, React no trabaja con código JSX internamente. Es simplemente más fácil de usar como desarrollador. Más adelante en este capítulo, aprenderás en qué se transforma el código JSX y cómo es el código real con el que trabaja React.

#### Componentes integrados (*Built-In Components*)
Como se mostró en los ejemplos anteriores, puedes crear tus propios componentes personalizados creando funciones que devuelvan código JSX. Y de hecho, esa es una de las cosas principales que harás todo el tiempo como desarrollador de React: crear funciones de componentes, muchas funciones de componentes.

Pero, en última instancia, si fusionaras todo el código JSX en un solo fragmento grande de código JSX, como se muestra en el último ejemplo, terminarías con un bloque de código JSX que incluye solo elementos HTML estándar como `<div>`, `<h2>`, `<p>`, etc.

Al usar React, no creas elementos HTML completamente nuevos que el navegador pueda mostrar y manejar directamente. En su lugar, creas componentes que solo funcionan dentro del entorno de React. Antes de que lleguen al navegador, React los evalúa y los "traduce" a instrucciones de JavaScript que manipulan el DOM (como `document.append(…)`).

Pero ten en cuenta que todo este código JSX es una característica que no forma parte del propio lenguaje JavaScript. Es básicamente azúcar sintáctico (*syntactical sugar*, es decir, una simplificación con respecto a la sintaxis del código) proporcionado por la librería React y la configuración del proyecto que estás utilizando para escribir código React. Por lo tanto, elementos como `<div>`, cuando se usan en código JSX, tampoco son elementos HTML normales porque no estás escribiendo código HTML. Puede parecerlo, pero está dentro de un archivo `.jsx` y no es marcado HTML; en su lugar, es este código JSX especial. Es importante tener esto presente.

En consecuencia, estos elementos `<div>` y `<h2>` que ves en todos estos ejemplos también son solo componentes de React al final. Pero no son componentes construidos por ti, sino que son proporcionados por React (o, para ser precisos, por ReactDOM).

Al trabajar con React, siempre terminas con estas primitivas: estas funciones de componentes integradas que luego se traducen en instrucciones del navegador que generan y agregan o eliminan elementos normales del DOM. La idea detrás de la creación de componentes personalizados es agrupar estos elementos para que termines con bloques de construcción reutilizables que se puedan usar para construir la interfaz de usuario general. Pero, al final, esta interfaz de usuario está formada por elementos HTML normales.

> [!NOTE]
> Dependiendo de tu nivel de conocimiento sobre desarrollo web frontend, es posible que hayas oído hablar de una función web llamada *Web Components*. La idea detrás de esta característica es que puedes construir elementos HTML completamente nuevos con JavaScript puro.
> Como se mencionó, React no utiliza esta característica; no construyes nuevos elementos HTML personalizados con React.

#### Convenciones de nomenclatura
Todas las funciones de componentes que puedes encontrar en este libro llevan nombres como `SubmitButton`, `AuthForm` o `Greeting`.

Por lo general, puedes nombrar tus funciones de React como quieras, al menos en el archivo donde las estás definiendo. Pero es una convención común utilizar la convención de nomenclatura **PascalCase**, en la que el primer carácter está en mayúscula y varias palabras se agrupan en una sola palabra (`SubmitButton` en lugar de `Submit Button`), donde cada "subpalabra" comienza con otro carácter en mayúscula.

En el lugar donde defines la función de tu componente, es solo una convención de nomenclatura, no una regla estricta. Sin embargo, es una **regla estricta** en el lugar donde usas las funciones de los componentes, es decir, en el código JSX donde incrustas tus propios componentes personalizados.

No puedes usar tu propia función de componente personalizado como un componente de esta manera:

```javascript
<greeting />
```

React te obliga a usar un carácter inicial en mayúscula para los nombres de tus propios componentes personalizados cuando los uses en código JSX. Esta regla existe para darle a React una forma clara y fácil de distinguir los componentes personalizados de los componentes integrados como `<div>`, etc. React solo necesita mirar el carácter inicial para determinar si es un elemento integrado o un componente personalizado.

Además de los nombres de las funciones de los componentes en sí, también es importante comprender las convenciones de nomenclatura de archivos. Los componentes personalizados se almacenan normalmente en archivos independientes que residen dentro de una carpeta `src/components/`. Sin embargo, esto no es una regla estricta. La ubicación exacta así como el nombre de la carpeta dependen de ti, pero debe estar en algún lugar dentro de la carpeta `src/`. Sin embargo, usar una carpeta llamada `components/` es el estándar.

Mientras que es el estándar usar PascalCase para las funciones de los componentes, no hay un valor predeterminado general con respecto a los nombres de archivo. Algunos desarrolladores también prefieren PascalCase para los nombres de archivo; y, de hecho, en proyectos de React nuevos, creados como se describe en este libro, el componente `App` se puede encontrar dentro de un archivo llamado `App.jsx`. No obstante, también encontrarás muchos proyectos de React donde los componentes se almacenan en archivos que siguen la convención de nomenclatura **kebab-case** (todo en minúsculas y varias palabras se combinan en una sola palabra mediante un guion). Con esta convención, las funciones de los componentes podrían almacenarse en archivos llamados `submit-button.jsx`, por ejemplo.

En última instancia, depende de ti (y de tu equipo) qué convención de nomenclatura de archivos deseas seguir. En este libro, se utilizará **PascalCase** para los nombres de archivo.

---

### Sección 4: JSX vs HTML vs Vanilla JavaScript

Como se mencionó anteriormente, los proyectos de React suelen contener mucho código JSX. La mayoría de los componentes personalizados devolverán fragmentos de código JSX. Puedes ver esto en todos los ejemplos compartidos hasta ahora, y lo verás en básicamente todos los proyectos de React que explores, sin importar si estás usando React para el navegador u otras plataformas como `react-native`.

¿Pero qué es exactamente este código JSX? ¿En qué se diferencia de HTML? ¿Y cómo se relaciona con JavaScript puro?

JSX es una característica que no forma parte de JavaScript puro. Sin embargo, lo que puede resultar confuso es que tampoco forma parte directamente de la librería React.

En cambio, JSX es azúcar sintáctico proporcionado por el flujo de trabajo de compilación que forma parte del proyecto general de React. Cuando inicias el servidor web de desarrollo mediante `npm run dev` o compilas la aplicación de React para producción (es decir, para su despliegue) a través de `npm run build`, inicias un proceso que transforma este código JSX nuevamente en instrucciones estándar de JavaScript. Como desarrollador, no ves esas instrucciones finales, pero React, la librería, las recibe y las evalúa.

Entonces, ¿en qué se transforma el código JSX?

En los proyectos modernos de React, se transforma en un código bastante complejo y poco intuitivo que se parece a esto:

```javascript
function Ld() { 
  return St.jsx('p', { children: 'Welcome to this book!' }); 
}
```

Por supuesto, este código no es muy amigable para el desarrollador. No es el tipo de código que tú escribirías. En su lugar, es el código producido por Vite (es decir, por el proceso de compilación subyacente) para que lo ejecute el navegador.

Pero, en teoría, podrías escribir código como este en lugar de usar JSX, si, por alguna razón, quisieras evitar escribir código JSX. React tiene un método integrado que puedes usar en lugar de JSX: puedes usar el método `createElement(…)` de React.

Aquí tienes un ejemplo concreto, primero en JSX:

```javascript
function Greeting() { 
  return <p>Hello World!</p>; 
};
```

En lugar de usar JSX, también podrías escribir el código de este componente de la siguiente manera:

```javascript
function Greeting() { 
  return React.createElement('p', {}, 'Hello World!'); 
};
```

`createElement()` es un método integrado en la librería React. Le indica a React que cree un elemento de párrafo con `'Hello World!'` como contenido secundario (es decir, como contenido interno anidado). Este elemento de párrafo se crea internamente primero (a través de un concepto llamado DOM virtual, que se discutirá más adelante en el libro, en el Capítulo 10, *Tras Bambalinas de React y Oportunidades de Optimización*). A partir de entonces, una vez creados todos los elementos para todos los elementos JSX, el DOM virtual se traduce en instrucciones reales de manipulación del DOM que son ejecutadas por el navegador.

> [!NOTE]
> Se ha mencionado anteriormente que React (en el navegador) es en realidad una combinación de dos paquetes: `react` y `react-dom`.
> Con la introducción de `React.createElement(…)`, ahora es más fácil explicar cómo funcionan juntos estos dos paquetes: React crea este DOM virtual internamente y luego lo pasa al paquete `react-dom`. Este paquete luego genera las instrucciones reales de manipulación del DOM que deben ejecutarse para actualizar la página web de modo que se muestre la interfaz de usuario deseada allí.
> Como se mencionó, esto se cubrirá con mayor detalle en el Capítulo 10.

El valor del parámetro intermedio (`{}` en el ejemplo) es un objeto de JavaScript que puede contener una configuración adicional para el elemento que se va a crear.

Aquí tienes un ejemplo donde este argumento intermedio se vuelve importante:

```javascript
function Advertisement() { 
  return <a href="https://my-website.com">Visit my website</a>; 
};
```

Esto se transformaría en lo siguiente:

```javascript
function Advertisement() { 
  return React.createElement( 
    'a', 
    { href: ' https://my-website.com ' }, 
    'Visit my website' 
  ); 
};
```

El último argumento que se pasa a `React.createElement(…)` es el contenido secundario del elemento, es decir, el contenido que debe estar entre las etiquetas de apertura y cierre del elemento. Para elementos JSX anidados, se generarían llamadas anidadas a `React.createElement(…)`:

```javascript
function Alert() { 
  return ( 
    <div> 
      <h2>This is an alert!</h2> 
    </div> 
  ); 
};
```

Esto se transformaría así:

```javascript
function Alert() { 
  return React.createElement( 
    'div', 
    {}, 
    React.createElement('h2', {}, 'This is an alert!') 
  ); 
};
```

#### Uso de React sin JSX
Dado que todo el código JSX se transforma de todos modos en estas llamadas a métodos nativos de JavaScript, en realidad puedes crear aplicaciones e interfaces de usuario con React sin utilizar JSX.

Puedes omitir JSX por completo si lo deseas. En lugar de escribir código JSX en tus componentes y en todos los lugares donde se espera JSX, simplemente puedes llamar a `React.createElement(…)`.

Por ejemplo, los dos fragmentos siguientes producirán exactamente la misma interfaz de usuario en el navegador:

```javascript
function App() { 
  return ( 
    <p>Please visit my <a href="https://my-blog-site.com">Blog</a></p> 
  ); 
};
```

El fragmento anterior será en última instancia lo mismo que el siguiente:

```javascript
function App() { 
  return React.createElement( 
    'p', 
    {}, 
    [ 
      'Please visit my ', 
      React.createElement( 
        'a', 
        { href: 'https://my-blog-site.com' }, 
        'Blog' 
      ) 
    ] 
  ); 
};
```

Por supuesto, otra cuestión es si querrías hacer esto. Como puedes ver en este ejemplo, es mucho más engorroso depender únicamente de `React.createElement(…)`. Terminas escribiendo mucho más código y las estructuras de elementos profundamente anidadas conducirán a un código que puede volverse casi imposible de leer.

Por eso, por lo general, los desarrolladores de React usan JSX. Es una gran característica que hace que construir interfaces de usuario con React sea mucho más agradable. Pero es importante entender que no es HTML ni una característica de JavaScript puro, sino que es azúcar sintáctico que se transforma en llamadas a funciones detrás de escena.

#### Los elementos JSX se tratan como valores regulares de JavaScript
Debido a que JSX es simplemente azúcar sintáctico que se transforma, hay un par de conceptos y reglas notables que debes tener en cuenta:
- Los elementos JSX son simplemente valores regulares de JavaScript (funciones, para ser precisos) al final.
- Las mismas reglas que se aplican a todos los valores de JavaScript también se aplican a los elementos JSX.
- Como resultado, en un lugar donde solo se espera un valor (por ejemplo, después de la palabra clave `return`), **solo debes tener un elemento JSX**.

Este código causaría un error:

```javascript
function App() { 
  return ( 
    <p>Hello World!</p> 
    <p>Let's learn React!</p> 
  ); 
};
```

El código puede parecer válido al principio, pero en realidad es incorrecto. En este ejemplo, estarías devolviendo dos valores en lugar de solo uno. Eso no está permitido en JavaScript.

Por ejemplo, el siguiente código que no es de React también sería inválido:

```javascript
function calculate(a, b) { 
  return ( 
    a + b 
    a - b 
  ); 
};
```

No puedes devolver más de un valor, independientemente de cómo lo escribas.

Por supuesto, puedes devolver una matriz (*array*) o un objeto. Por ejemplo, este código sería válido:

```javascript
function calculate(a, b) { 
  return [ 
    a + b, 
    a - b 
  ]; 
};
```

Sería válido porque solo devuelves un valor: un array. Este array contiene múltiples valores, como suelen contener los arrays. Eso estaría bien y lo mismo ocurriría si usaras código JSX:

```javascript
function App() { 
  return [ 
    <p>Hello World!</p>, 
    <p>Let's learn React!</p> 
  ]; 
};
```

Este tipo de código estaría permitido ya que estás devolviendo un único array con dos elementos dentro de él. Los dos elementos son elementos JSX en este caso, pero como se mencionó anteriormente, los elementos JSX son solo valores regulares de JavaScript. Por lo tanto, puedes usarlos en cualquier lugar donde se esperen valores.

Sin embargo, cuando trabajes con JSX, no verás este enfoque de array con demasiada frecuencia, simplemente porque puede resultar molesto acordarse de envolver los elementos JSX entre corchetes. Además, se parece menos a HTML, lo que de alguna manera frustra el propósito y la idea central detrás de JSX (se inventó para permitir a los desarrolladores escribir código HTML dentro de archivos JavaScript).

En su lugar, si se requieren elementos hermanos, como en estos ejemplos, se utiliza un tipo especial de componente envoltorio: un **React fragment** (fragmento de React). Ese es un componente integrado que sirve para permitirte devolver o definir elementos JSX hermanos:

```javascript
function App() { 
  return ( 
    <> 
      <p>Hello World!</p> 
      <p>Let's learn React!</p> 
    </> 
  ); 
};
```

Este elemento especial `<> … </>` está disponible en la mayoría de los proyectos modernos de React (por ejemplo, los creados a través de Vite), y puedes pensar en él como una envoltura de tus elementos JSX con un array detrás de escena. Alternativamente, también puedes usar `<React.Fragment> … </React.Fragment>`. Dado que es posible que algunos proyectos de React no admitan la sintaxis abreviada `<> … </>`, este componente integrado siempre está disponible.

Los paréntesis (`()`) que se envuelven alrededor del código JSX en todos estos ejemplos son necesarios para permitir un formato multilínea limpio. Técnicamente, podrías poner todo tu código JSX en una sola línea, pero sería bastante ilegible. Para dividir los elementos JSX en varias líneas, tal como sueles hacer con el código HTML normal en archivos `.html`, necesitas esos paréntesis: le indican a JavaScript dónde comienza y termina el valor devuelto.

Dado que los elementos JSX son valores regulares de JavaScript (al menos después de ser traducidos por el proceso de compilación), también puedes usar elementos JSX en todos los lugares donde se pueden usar valores.

Hasta ahora, ese ha sido el caso de todas estas sentencias `return`, pero también puedes almacenar elementos JSX en variables o pasarlos como argumentos a otras funciones:

```javascript
function App() { 
  const content = <p>Stored in a variable!</p>; // this works! 
  return content; 
};
```

Esto será importante una vez que te sumerjas en conceptos un poco más avanzados como contenido condicional o repetido, algo que se cubrirá en el Capítulo 5, *Renderizado de Listas y Contenido Condicional*.

#### Los elementos JSX deben tener una etiqueta de cierre
Otra regla importante relacionada con los elementos JSX es que siempre deben tener una etiqueta de cierre. Por lo tanto, los elementos JSX deben ser **autocerrados** (*self-closing*) si no hay contenido entre las etiquetas de apertura y cierre:

```javascript
function App() { 
  return <img src="some-image.png" />; 
};
```

En HTML normal, no necesitarías esa barra diagonal al final. En su lugar, el HTML estándar admite elementos vacíos o *void elements* (es decir, `<img src="…">`). Puedes agregar esa barra inclinada allí también, pero no es obligatoria.

Al trabajar con JSX, estas barras diagonales son **obligatorias** si tu elemento no contiene ningún contenido secundario.

---

### Sección 5: Más allá del contenido estático

Hasta ahora, en todos estos ejemplos, el contenido que se devolvía era estático. Era contenido como `<p>Hello World!</p>`, que por supuesto es un contenido que nunca cambia. Siempre mostrará un párrafo que dice: 'Hello World!'.

Pero la mayoría de los sitios web, por supuesto, necesitan mostrar contenido dinámico que pueda cambiar (por ejemplo, debido a la entrada del usuario). Del mismo modo, te resultará difícil encontrar muchos sitios web sin imágenes.

Por lo tanto, como desarrollador de React, es importante saber cómo mostrar contenido dinámico (y qué significa realmente "contenido dinámico") y cómo mostrar imágenes en una aplicación de React.

#### Mostrar contenido dinámico
En este punto del libro, todavía no tienes ninguna herramienta para hacer que el contenido sea verdaderamente dinámico. Para ser precisos, React requiere el concepto de **estado** (que se cubrirá en el Capítulo 4, *Trabajando con Eventos y Estado*) para cambiar el contenido que se muestra (por ejemplo, tras la entrada del usuario o algún otro evento).

Sin embargo, dado que este capítulo trata sobre JSX, vale la pena sumergirse en la sintaxis para mostrar contenido dinámico, aunque todavía no sea verdaderamente interactivo:

```javascript
function App() { 
  const userName = 'Max'; 
  return <p>Hi, my name is {userName}!</p>; 
};
```

Técnicamente, este ejemplo todavía produce una salida estática ya que `userName` nunca cambia, pero ya puedes ver la sintaxis para mostrar contenido dinámico como parte del código JSX. Utilizas llaves de apertura y cierre (`{…}`) con una expresión de JavaScript (como el nombre de una variable o constante, como es el caso aquí) entre esas llaves.

Puedes colocar cualquier expresión válida de JavaScript entre esas llaves. Por ejemplo, también puedes llamar a una función (por ejemplo, `{getMyName()}`) o hacer cálculos simples en línea (por ejemplo, `{1 + 1}`).

Sin embargo, no puedes agregar declaraciones complejas como bucles o sentencias `if` entre esas llaves. Nuevamente, se aplican las reglas estándar de JavaScript. Estás mostrando un valor (potencialmente) dinámico y, por lo tanto, se permite cualquier cosa que produzca un único valor en ese lugar. Sin embargo, vale la pena señalar que algunos tipos de valores no se pueden usar para mostrar un valor en JSX. Por ejemplo, intentar mostrar un objeto de JavaScript directamente en JSX provocará un error.

También vale la pena señalar que no estás limitado a mostrar contenido dinámico entre las etiquetas de los elementos. En su lugar, también puedes establecer valores dinámicos para los atributos:

```javascript
function App() { 
  const userName = 'Max'; 
  return <input type="text" value={userName} />; 
};
```

#### Renderizado de imágenes
La mayoría de los sitios web no muestran únicamente texto plano. En su lugar, a menudo también es necesario representar imágenes.

Por supuesto, cuando trabajas con React, puedes usar el elemento `<img />` predeterminado como en cualquier otro proyecto web. Pero hay dos cosas importantes a tener en cuenta al mostrar imágenes en proyectos de React:
1. `<img />` debe ser una etiqueta de cierre automático (*self-closing*).
2. Al mostrar imágenes locales almacenadas dentro de la carpeta `src/`, debes importarlas en tus archivos `.jsx`.

Como se explicó anteriormente en la sección sobre el cierre de elementos JSX, no puedes tener elementos JSX vacíos sin ninguna etiqueta de cierre.

Además, al mostrar imágenes almacenadas localmente (es decir, imágenes almacenadas en la carpeta `src/` del proyecto, no en algún servidor remoto), normalmente no estableces una ruta de cadena fija hacia la imagen en tu código.

Podrías estar acostumbrado a mostrar imágenes de esta forma:

```html
<img src="assets/images/wave.jpg">
```

Pero los proyectos de React (por ejemplo, cuando se crean con Vite) implican algún tipo de proceso de compilación. En la mayoría de los proyectos, la estructura final del proyecto que se desplegará en un servidor se verá bastante diferente de la estructura del proyecto en la que trabajas durante el desarrollo.

Siendo ese el caso, si almacenas una imagen en la carpeta `src/assets` en un proyecto de React basado en Vite y la usas como ruta (`<img src="src/assets/my-image.jpg" />`), la imagen no se cargará en el sitio web desplegado. No se cargará allí porque la estructura de carpetas desplegable ya no contendrá una carpeta `src/assets`.

De hecho, puedes hacerte una idea de la estructura de carpetas lista para producción ejecutando `npm run build`. Esto compilará el proyecto para el despliegue y producirá una nueva carpeta `dist` en el directorio de tu proyecto. Es el contenido de esa carpeta `dist` el que se desplegará en algún servidor. Si inspeccionas esa carpeta, no encontrarás una carpeta `src` allí.

**Figura 2.3**: La carpeta `dist` contiene una estructura diferente.

Dicho de otra manera: no puedes saber de antemano la ruta exacta de una imagen almacenada localmente. Es por eso que debes importar el archivo de imagen en tu archivo `.jsx`. Como resultado, obtendrás un valor de cadena que contendrá la ruta real (que funcionará en producción). Este valor se puede establecer luego como un valor dinámico para el atributo `src` del elemento `<img />`:

```javascript
import myImage from './assets/my-image.png'; 

function App() { 
  return <img src={myImage} />; 
};
```

Esto puede parecer extraño al principio, pero es un código que funcionará en prácticamente todos los proyectos de React. Detrás de escena, esta importación es analizada por el proceso de compilación subyacente. Luego, la sentencia de importación se elimina y la ruta de la imagen se codifica de forma fija en el código de salida listo para producción (es decir, el código que se almacena en la carpeta `dist`).

Sin embargo, hay una excepción importante: si almacenas un archivo de imagen (o, en realidad, cualquier recurso) en la carpeta `public/` de tu proyecto, puedes hacer referencia directa a su ruta.

Por ejemplo, un archivo de imagen `demo.jpg` almacenado en `public/images/demo.jpg` se puede renderizar y mostrar de la siguiente manera:

```javascript
function App() { 
  return <img src="/images/demo.jpg" />; 
};
```

Esto funciona porque el contenido de la carpeta `public/` simplemente se copia en la carpeta `dist/`. A diferencia de la carpeta `src/` y sus archivos anidados, los archivos de la carpeta `public/` omiten el paso de transpilación.

Ten en cuenta que el nombre de la carpeta pública en sí no forma parte de las rutas a las que se hace referencia: es `src="/images/demo.jpg"`, no `src="/public/images/demo.jpg"`.

¿Qué enfoque deberías utilizar entonces? ¿Almacenar imágenes en `src/` o en `public/`?
- Para la mayoría de las imágenes, `src/` es una opción sensata, ya que el paso de preprocesamiento asigna un nombre de archivo único a cada archivo importado. Como resultado, los archivos se pueden almacenar en caché de manera más eficiente una vez que se implementa la aplicación.
- Cualquier archivo importado en el archivo raíz `index.html`, o archivos donde el nombre del archivo nunca debe cambiar (por ejemplo, porque también está referenciado por alguna otra aplicación que se ejecuta en otro servidor) normalmente debe ir a la carpeta `public/`.

Por lo tanto, en la mayoría de los casos, al mostrar imágenes que están almacenadas localmente en tu proyecto, debes guardarlas en la carpeta `src/` y luego importarlas a tus archivos JSX. Al usar imágenes almacenadas en algún servidor remoto, utilizarías en su lugar la URL completa de la imagen:

```javascript
function App() { 
  return <img src="https://some-server.com/my-image.jpg" />; 
};
```

---

### Sección 6: ¿Cuándo deberías dividir componentes?

A medida que trabajes con React y aprendas más y más sobre él, y a medida que te sumerjas en proyectos de React más desafiantes, es muy probable que te surja una pregunta muy común: **¿Cuándo debo dividir un solo componente de React en varios componentes independientes?**

Como se mencionó anteriormente en este capítulo, React se basa en componentes y, por lo tanto, es muy común tener docenas, cientos o incluso miles de componentes de React en un solo proyecto.

Cuando se trata de dividir un componente de React individual en varios componentes más pequeños, no existe una regla estricta que debas seguir. Como se mencionó anteriormente, podrías poner todo el código de tu interfaz de usuario en un solo componente grande. Alternativamente, podrías crear un componente personalizado independiente para cada elemento HTML y fragmento de contenido que tengas en tu interfaz de usuario. Probablemente ninguno de los dos enfoques sea el ideal. En su lugar, una buena regla general es **crear un componente de React independiente para cada entidad de datos identificable**.

Por ejemplo, si estás mostrando una lista de tareas pendientes (*to-do*), podrías identificar dos entidades principales: el elemento de tarea individual y la lista general. En este caso, podría tener sentido crear dos componentes separados en lugar de escribir un componente más grande.

La ventaja de dividir tu código en múltiples componentes es que los componentes individuales se mantienen manejables porque hay menos código por componente y por archivo de componente.

Sin embargo, cuando se trata de dividir componentes en múltiples componentes, surge un nuevo problema: **¿Cómo haces que tus componentes sean reutilizables y configurables?**

```javascript
import Todo from './todo.jsx'; 

function TodoList() { 
  return ( 
    <ul> 
      <Todo /> 
      <Todo /> 
    </ul> 
  ); 
};
```

En este ejemplo, todas las tareas pendientes serían exactamente iguales porque usamos el mismo componente `<Todo />`, que no se puede configurar. Es posible que desees hacerlo configurable agregando atributos personalizados (`<Todo text="Learn React!" />`) o pasando contenido entre las etiquetas de apertura y cierre (`<Todo>Learn React!</Todo>`).

Y, por supuesto, React admite esto. En el próximo capítulo, aprenderás sobre un concepto clave llamado **props**, que te permite hacer que tus componentes sean configurables de esta manera.

---

### Sección 7: Resumen y puntos clave

- React adopta los componentes: bloques de construcción reutilizables que se combinan para definir la interfaz de usuario final.
- Los componentes deben devolver contenido renderizable: típicamente, código JSX que define el código HTML que debe producirse al final.
- React proporciona muchos componentes integrados: además de componentes especiales como `<> … </>`, obtienes componentes para todos los elementos HTML estándar.
- Para permitir que React distinga los componentes personalizados de los componentes integrados, los nombres de los componentes personalizados deben comenzar con letras mayúsculas cuando se usan en código JSX (normalmente se usa la convención PascalCase).
- JSX no es HTML ni una característica estándar de JavaScript; en su lugar, es azúcar sintáctico proporcionado por los flujos de trabajo de compilación que forman parte de todos los proyectos de React.
- Podrías reemplazar el código JSX con llamadas a `React.createElement(…)`, pero dado que esto conduce a un código significativamente más difícil de leer, normalmente se evita.
- Al usar elementos JSX, no debes tener elementos hermanos en lugares donde se esperan valores únicos (por ejemplo, directamente después de la palabra clave `return`).
- Los elementos JSX siempre deben cerrarse automáticamente si no hay contenido entre las etiquetas de apertura y cierre.
- El contenido dinámico se puede mostrar mediante llaves (por ejemplo, `<p>{someText}</p>`).
- Las imágenes se pueden representar haciendo referencia a sus rutas (si se almacenan de forma remota o en la carpeta `public/`) o importando los archivos de imagen a los archivos JSX y mostrándolos con la sintaxis de contenido dinámico.
- En la mayoría de los proyectos de React, divides el código de tu interfaz de usuario en docenas o cientos de componentes, que luego se exportan e importan para combinarse nuevamente.

---

### Sección 8: ¿Qué sigue?

En este capítulo aprendiste mucho sobre componentes y JSX. El próximo capítulo se basa en este conocimiento clave y explica cómo puedes hacer que los componentes sean reutilizables haciéndolos configurables.

Antes de continuar, también puedes practicar lo que has aprendido hasta este punto respondiendo las preguntas y realizando los ejercicios a continuación.

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con las respuestas de ejemplo que se encuentran aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/02-components-jsx/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/02-components-jsx/exercises/questions-answers.md).

1. ¿Cuál es la idea detrás del uso de componentes?
2. ¿Cómo puedes crear un componente de React?
3. ¿Qué convierte a una función normal en una función de componente de React?
4. ¿Qué reglas centrales debes tener en cuenta con respecto a los elementos JSX?
5. ¿Cómo gestionan el código JSX React y ReactDOM?

---

### Sección 10: Aplica lo aprendido

Con este y el capítulo anterior, tienes todo el conocimiento necesario para crear un proyecto de React y completarlo con algunos primeros componentes básicos.

A continuación, encontrarás tus dos primeras actividades prácticas para este libro.

#### Actividad 2.1: Creación de una aplicación React para presentarte
Supón que estás creando tu página de portafolio personal y, como parte de esa página, deseas mostrar información básica sobre ti (por ejemplo, tu nombre o edad). Podrías usar React y construir un componente de React que muestre este tipo de información, como se describe en la siguiente actividad.

El objetivo es crear una aplicación de React como aprendiste en el capítulo anterior (es decir, crearla mediante `npm create vite@latest <nombre-del-proyecto>` y ejecutar `npm run dev` para iniciar el servidor de desarrollo) y editar el archivo `App.jsx` de modo que muestres información básica sobre ti. Podrías, por ejemplo, mostrar tu nombre completo, dirección, puesto de trabajo u otro tipo de información. Al final, depende de ti qué contenido deseas mostrar y qué elementos HTML elijas.

La idea detrás de este primer ejercicio es que practiques la creación de proyectos y el trabajo con código JSX.

Los pasos son los siguientes:
1. Crea un nuevo proyecto de React mediante `npm create vite@latest <proyecto>`. Alternativamente, puedes usar la instantánea del proyecto inicial proporcionada aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-1-start).
2. Edita el archivo `App.jsx` en la carpeta `/src` del proyecto creado y devuelve código JSX con los elementos HTML que elijas para mostrar información básica sobre ti. Puedes usar los estilos en el archivo `index.css` de la instantánea del proyecto inicial para aplicar algunos estilos.
3. Además, almacena una imagen en la carpeta `src/assets` y muéstrala en el componente `App`.

Al final deberías obtener un resultado como este:

**Figura 2.4**: El resultado final de la actividad: información del usuario mostrada en la pantalla.

> [!NOTE]
> Por supuesto, el estilo variará. Para obtener el mismo estilo que se muestra en la captura de pantalla, usa mi proyecto inicial preparado, que puedes encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-1-start).
> Analiza el archivo `index.css` en ese proyecto para determinar cómo estructurar tu código JSX para aplicar los estilos.
> Encontrarás una solución de ejemplo en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/02-components-jsx/activities/practice-1/SOLUTION-INSTRUCTIONS.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/02-components-jsx/activities/practice-1/SOLUTION-INSTRUCTIONS.md).
> Además de las instrucciones enlazadas, también encontrarás el código de la solución de ejemplo finalizado en la carpeta del proyecto que contiene el archivo `SOLUTION-INSTRUCTIONS.md`.
> Sin embargo, antes de explorar esta solución, deberías intentar resolver esta tarea por tu cuenta. Incluso si tu resultado difiere de la solución de ejemplo, o si no logras crear una aplicación funcional, aprenderás más al menos intentándolo porque, como siempre en la vida, solo la práctica hace al maestro.

#### Actividad 2.2: Creación de una aplicación React para registrar tus objetivos para este libro
Supón que estás agregando una nueva sección a tu sitio de portafolio, donde planeas realizar un seguimiento de tu progreso de aprendizaje. Como parte de esta página, planeas definir y mostrar tus objetivos principales para este libro (por ejemplo, "Aprender sobre las funciones clave de React", "Hacer todos los ejercicios", etc.).

El objetivo de esta actividad es crear otro proyecto nuevo de React en el que agregues múltiples componentes nuevos. Cada objetivo estará representado por un componente independiente, y todos estos componentes de objetivos se agruparán en otro componente que enumera todos los objetivos principales. Además, puedes agregar un componente de encabezado adicional que contenga el título principal de la página web.

Los pasos para completar esta actividad son los siguientes:
1. Crea un nuevo proyecto de React mediante `npm create vite@latest <proyecto>`, o usa la instantánea inicial del proyecto proporcionada aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-2-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-2-start).
2. Dentro del nuevo proyecto, crea una carpeta `components` que contenga varios archivos de componentes (tanto para los objetivos individuales como para la lista de objetivos y el encabezado de la página).
3. Dentro de los diferentes archivos de componentes, define y exporta múltiples funciones de componentes (`FirstGoal`, `SecondGoal`, `ThirdGoal`, etc.) para los diferentes objetivos (un componente por archivo).
4. Además, define un componente para la lista general de objetivos (`GoalList`) y otro componente para el encabezado de la página (`Header`).
5. En los componentes de objetivos individuales, devuelve código JSX con el texto del objetivo y una estructura de elementos HTML adecuada para albergar este contenido.
6. En el componente `GoalList`, importa y muestra los componentes de objetivos individuales.
7. Importa y muestra los componentes `GoalList` y `Header` en el componente raíz `App` (reemplaza el código JSX existente).
8. Aplica el estilo que elijas. También puedes usar el archivo `index.css` que forma parte de la instantánea del proyecto inicial como inspiración.

Al final deberías obtener el siguiente resultado:

**Figura 2.5**: La salida final de la página, mostrando una lista de objetivos.

También encontrarás una solución de ejemplo para esta actividad en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/02-components-jsx/activities/practice-2/SOLUTION-INSTRUCTIONS.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/02-components-jsx/activities/practice-2/SOLUTION-INSTRUCTIONS.md).
Como antes, además de las instrucciones enlazadas, también encontrarás el código de la solución de ejemplo finalizado en la carpeta del proyecto que contiene el archivo `SOLUTION-INSTRUCTIONS.md`.
