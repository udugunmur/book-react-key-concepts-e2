# Parte 1: Fundamentos de React

## Capítulo 6: Estilos en Aplicaciones React

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Aplicar estilos a elementos JSX mediante la asignación de estilos en línea (*inline styles*) o con la ayuda de clases CSS.
- Establecer estilos en línea y mediante clases, tanto de forma estática como dinámica o condicional.
- Construir componentes reutilizables que permitan la personalización de estilos.
- Utilizar CSS Modules para acotar y limitar el alcance (*scope*) de los estilos a componentes específicos.
- Comprender la idea central detrás de `styled-components`, una biblioteca externa de CSS-in-JS.
- Usar Tailwind CSS para dar estilo a aplicaciones de React.

---

### Sección 1: Introducción

React.js es una biblioteca de JavaScript para el frontend. Esto significa que está enfocada por completo en la construcción de interfaces de usuario (web) y el manejo de la interacción del usuario.

Hasta este punto, este libro ha explorado ampliamente cómo se puede utilizar React para agregar interactividad a una aplicación web. El estado, el manejo de eventos y el contenido dinámico son conceptos clave relacionados con esto.

Por supuesto, los sitios web y las aplicaciones web no se tratan solo de interactividad. Podrías crear una aplicación web increíble que ofrezca funciones interactivas y atractivas y, aun así, podría no ser popular si carece de un aspecto visual atractivo. La presentación es clave, y la web no es una excepción.

Por lo tanto, al igual que todas las demás aplicaciones y sitios web, las aplicaciones y sitios web de React necesitan estilos adecuados, y al trabajar con tecnologías web, las Hojas de Estilo en Cascada (**CSS**, *Cascading Style Sheets*) son el lenguaje de elección.

Sin embargo, este libro no trata sobre CSS. No te explicará ni enseñará cómo usar CSS, ya que existen recursos dedicados y más completos para ello (por ejemplo, las guías gratuitas de CSS en [https://developer.mozilla.org/en-US/docs/Learn/CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS)). Pero este capítulo te enseñará cómo **combinar código CSS con JSX y conceptos de React**, como el estado y las props. Aprenderás a agregar estilos a tus elementos JSX, estilizar componentes personalizados y hacer que los estilos de esos componentes sean configurables. Este capítulo también te enseñará cómo establecer estilos de forma dinámica y condicional, y explorará bibliotecas externas populares, como `styled-components` y Tailwind CSS, que se pueden utilizar para dar estilo.

---

### Sección 2: ¿Cómo funcionan los estilos en aplicaciones React?

Hasta este punto, las aplicaciones y ejemplos presentados en este libro solo han tenido un estilo mínimo. Pero al menos tenían algún estilo básico, en lugar de no tener ningún estilo.

¿Pero cómo se agregó ese estilo? ¿Cómo se pueden agregar estilos a los elementos de la interfaz de usuario (como los elementos del DOM) al usar React?

La respuesta corta es: "Exactamente igual que en aplicaciones que no son de React". Puedes agregar estilos y clases de CSS a los elementos JSX tal como lo harías con los elementos HTML normales. Y en tu código CSS, puedes usar todas las características y selectores que conoces de CSS. No hay cambios específicos de React que debas hacer al escribir código CSS.

Los ejemplos de código utilizados hasta ahora (es decir, las actividades u otros ejemplos alojados en GitHub) siempre utilizaron estilos CSS regulares, con la ayuda de selectores de CSS, para aplicar algunos estilos básicos a la interfaz de usuario final. Esas reglas de CSS se definieron en un archivo `index.css`, que forma parte de cada proyecto de React recién creado (al usar Vite para la creación de proyectos, como se muestra en el Capítulo 1, *React – Qué es y por qué*).

Por ejemplo, aquí está el archivo `index.css` utilizado en la Actividad 5.2 del capítulo anterior (Capítulo 5, *Renderizado de Listas y Contenido Condicional*):

```css
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;700&family=Rubik:ital,wght@0,300..900;1,300..900&display=swap'); 

body { 
  margin: 0; 
  padding: 3rem; 
  font-family: 'Poppins', sans-serif; 
  -webkit-font-smoothing: antialiased; 
  -moz-osx-font-smoothing: grayscale; 
  text-align: center; 
  background-color: #dff8fb; 
  color: #212324; 
} 

button { 
  padding: 0.5rem 1rem; 
  font-family: 'Rubik', sans-serif; 
  font-size: 1rem; 
  border: none; 
  border-radius: 4px; 
  background-color: #212324; 
  color: #fff; 
  cursor: pointer; 
} 

button:hover { 
  background-color: #3f3e40; 
} 

ul { 
  max-width: 35rem; 
  list-style-type: none; 
  padding: 0; 
  margin: 2rem auto; 
} 

li { 
  margin: 1rem 0; 
  padding: 1rem; 
  background-color: #5ef0fd; 
  border: 2px solid #212324; 
  border-radius: 4px; 
}
```

El código CSS real y su significado no son importantes (como se mencionó, este libro no trata sobre CSS). Sin embargo, lo importante es el hecho de que este código no contiene ningún código JavaScript o React en absoluto. Como se mencionó, el código CSS que escribes es totalmente independiente del hecho de que estés usando React en tu aplicación.

La pregunta más interesante es: ¿cómo se aplica realmente ese código a la página web renderizada? ¿Cómo se importa en esa página?

Normalmente, esperarías importaciones de archivos de estilo (a través de `<link href="…">`) dentro de los archivos HTML que se sirven. Dado que las aplicaciones de React normalmente se tratan de crear aplicaciones de una sola página (*Single-Page Applications*, consulta el Capítulo 1, *React – Qué es y por qué*), solo tienes un archivo HTML: el archivo `index.html`. Pero si inspeccionas ese archivo, no encontrarás ninguna importación `<link href="…">` que apunte al archivo `index.css` (solo algún otro elemento `<link>` que importa un favicon), como puedes ver en la siguiente captura de pantalla:

**Figura 6.1**: La sección `<head>` del archivo `index.html` no contiene ninguna importación `<link>` que apunte al archivo `index.css`.

¿Cómo se importan y aplican entonces los estilos definidos en `index.css`?

Encontrarás una declaración de importación en el archivo de entrada raíz (este es el archivo `main.jsx` en proyectos generados a través de Vite):

```javascript
import React from 'react'; 
import ReactDOM from 'react-dom/client'; 
import App from './App.jsx'; 
import './index.css'; 

ReactDOM.createRoot(document.getElementById('root')).render( 
  <React.StrictMode> 
    <App /> 
  </React.StrictMode>, 
);
```

La declaración `import './index.css';` hace que se importe el archivo CSS y que el código CSS definido se aplique a la página web renderizada.

Vale la pena señalar que este **no es un comportamiento estándar de JavaScript**. No puedes importar archivos CSS en JavaScript, al menos no si solo estás usando JavaScript puro (*vanilla JavaScript*).

CSS funciona de esta manera en las aplicaciones de React porque el código se transpila antes de cargarse en el navegador. Por lo tanto, no encontrarás esa declaración de importación en el código JavaScript final que se ejecuta en el navegador. En su lugar, durante el proceso de transpilación, el transpilador identifica la importación de CSS, la elimina del archivo JavaScript e inyecta el código CSS (o un enlace apropiado al archivo CSS potencialmente empaquetado y optimizado) en el archivo `index.html`.

Puedes confirmar esto inspeccionando el contenido del Modelo de Objetos del Documento (DOM) renderizado de la página web cargada en el navegador.

Para hacerlo, selecciona la pestaña *Elements* en las herramientas de desarrollo en Chrome, como se muestra a continuación:

**Figura 6.2**: Los elementos `<style>` de CSS inyectados se pueden encontrar en el DOM en tiempo de ejecución.

Puedes definir cualquier estilo que desees aplicar a tus elementos HTML (es decir, a tus elementos JSX en tus componentes) directamente dentro del archivo `index.css`, o en cualquier otro archivo CSS importado por el archivo `index.css`.

También puedes agregar declaraciones de importación de CSS adicionales, apuntando a otros archivos CSS, al archivo `main.jsx` o a cualquier otro archivo JavaScript (incluidos los archivos que almacenan componentes). Sin embargo, es importante tener en cuenta que **los estilos CSS son siempre globales**. No importa si importas un archivo CSS en `main.jsx` o en un archivo JavaScript específico de un componente, los estilos definidos en ese archivo CSS se aplicarán globalmente.

Eso significa que los estilos definidos en un archivo `goal-list.css`, que pueden importarse en un archivo `GoalList.jsx`, aún podrían afectar a los elementos JSX definidos en un componente totalmente diferente. Más adelante en este capítulo, aprenderás sobre técnicas que te permiten evitar conflictos accidentales de estilo e implementar el acotamiento de ámbito (*scoping*) de estilos.

#### Uso de estilos en línea (*Inline Styles*)
Puedes usar archivos CSS para definir estilos CSS globales y usar diferentes selectores de CSS para apuntar a diferentes elementos JSX (o grupos de elementos).

Pero a pesar de que normalmente se desaconseja, también puedes establecer estilos en línea directamente en elementos JSX a través de la prop `style`.

> [!NOTE]
> Si te preguntas por qué se desaconsejan los estilos en línea, la siguiente discusión en Stack Overflow proporciona muchos argumentos en contra de los estilos en línea: [https://stackoverflow.com/questions/2612483/whats-so-bad-about-in-line-css](https://stackoverflow.com/questions/2612483/whats-so-bad-about-in-line-css).

Establecer estilos en línea en código JSX funciona de la siguiente manera:

```javascript
function TodoItem() { 
  return <li style={{color: 'red', fontSize: '18px'}}>Learn React!</li>; 
};
```

En este ejemplo, la prop `style` se agrega al elemento `<li>` (todos los elementos JSX admiten la prop `style`), y tanto la propiedad `color` como la propiedad de tamaño (`fontSize`) del texto se establecen a través de CSS.

Este enfoque difiere de lo que usarías para establecer estilos en línea al trabajar solo con HTML (en lugar de JSX). Al usar HTML plano, establecerías estilos en línea de esta manera:

```html
<li style="color: red; font-size: 18px">Learn React!</li>
```

La diferencia es que **la prop `style` espera recibir un objeto de JavaScript que contenga la configuración de estilo**, no una cadena simple de texto. Esto es algo que debe tenerse en cuenta, ya que, como se mencionó anteriormente, los estilos en línea normalmente no se usan con tanta frecuencia.

Dado que el objeto de estilo es un objeto y no una cadena simple, se pasa como un valor entre llaves, tal como un array, un número o cualquier otro valor que no sea una cadena deba establecerse entre llaves (cualquier cosa entre comillas dobles o simples se trata como un valor de cadena). Por lo tanto, vale la pena señalar que el ejemplo anterior no utiliza ningún tipo de sintaxis especial de "doble llave" y, en su lugar, utiliza un par de llaves para rodear el valor que no es una cadena y otro par para rodear los datos del objeto.

Dentro del objeto de estilo, se puede establecer cualquier propiedad de estilo CSS admitida por el elemento del DOM subyacente. Los nombres de las propiedades son los definidos para el elemento HTML (es decir, los mismos nombres de propiedades CSS a los que podrías apuntar y establecer con JavaScript puro al mutar un elemento HTML).

Al establecer estilos en código JavaScript (como con la prop `style` mostrada anteriormente), se deben utilizar los **nombres de propiedades CSS de JavaScript**. Esos nombres son similares a los nombres de propiedades CSS que usarías en código CSS, pero no son exactamente iguales. Las diferencias ocurren para los nombres de propiedades que constan de varias palabras (por ejemplo, `font-size`). Al apuntar a tales propiedades en JavaScript, se debe usar la **notación camelCase** (`fontSize` en lugar de `font-size`), ya que las propiedades de JavaScript no pueden contener guiones. Alternativamente, podrías envolver el nombre de la propiedad entre comillas (`'font-size'`).

> [!NOTE]
> Puedes encontrar más información sobre la propiedad `style` de los elementos HTML y los nombres de propiedades CSS de JavaScript aquí: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style).

#### Establecer estilos mediante clases CSS
Como se mencionó, el uso de estilos en línea generalmente se desaconseja y, por lo tanto, se prefieren los estilos CSS definidos en archivos CSS (o entre etiquetas `<style>` en la sección `<head>` del documento).

En esos bloques de código CSS, puedes escribir código CSS regular y usar selectores de CSS para aplicar estilos CSS a ciertos elementos. Podrías, por ejemplo, aplicar estilo a todos los elementos `<li>` de una página (sin importar qué componente los haya renderizado) de esta manera:

```css
li { 
  color: red; 
  font-size: 18px; 
}
```

Siempre que este código se agregue a la página (porque el archivo CSS en el que está definido se importa en `main.jsx`, por ejemplo), se aplicará el estilo.

Con bastante frecuencia, los desarrolladores buscan apuntar a elementos específicos o grupos de elementos. En lugar de aplicar algún estilo a todos los elementos `<li>` de una página, el objetivo podría ser apuntar solo a los elementos `<li>` que forman parte de una lista específica. Considera esta estructura HTML que se renderiza en la página (puede estar dividida en varios componentes, pero esto no importa aquí):

```html
<nav> 
  <ul> 
    <li><a href="…">Home</a></li> 
    <li><a href="…">New Goals</a></li> 
  </ul> 
</nav> 
... 
<h2>My Course Goals</h2> 
<ul> 
  <li>Learn React!</li> 
  <li>Master React!</li> 
</ul>
```

En este ejemplo, lo más probable es que los elementos de la lista de navegación no reciban el mismo estilo que los elementos de la lista de objetivos del curso (y viceversa).

Normalmente, este problema se resolvería con la ayuda de clases CSS y el selector de clases. Podrías ajustar el código HTML de esta manera:

```html
<nav> 
  <ul> 
    <li><a href="…">Home</a></li> 
    <li><a href="…">New Goals</a></li> 
  </ul> 
</nav> 
... 
<h2>My Course Goals</h2> 
<ul> 
  <li class="goal-item">Learn React!</li> 
  <li class="goal-item">Master React!</li> 
</ul>
```

El siguiente código CSS luego apuntaría solo a los elementos de la lista de objetivos del curso pero no a los elementos de la lista de navegación:

```css
.goal-item { 
  color: red; 
  font-size: 18px; 
}
```

Este enfoque casi funciona también en aplicaciones de React.

Sin embargo, si intentas agregar clases CSS a elementos JSX, como se muestra en el ejemplo anterior, te encontrarás con una advertencia en las herramientas de desarrollo del navegador:

**Figura 6.3**: Una advertencia generada por React.

Como se ilustra en la figura anterior, no debes agregar `class` como prop y, en su lugar, debes usar **`className`**. De hecho, si cambias `class` por `className` como nombre de prop, la advertencia desaparecerá y se aplicarán los estilos CSS de clase. Por lo tanto, el código JSX adecuado se ve así:

```javascript
<ul> 
  <li className="goal-item">Learn React!</li> 
  <li className="goal-item">Master React!</li> 
</ul>
```

¿Pero por qué React sugiere que uses `className` en lugar de `class`?

Es similar a usar `htmlFor` en lugar de `for` cuando se trabaja con elementos `<label>` (como se discutió en el Capítulo 4, *Trabajando con Eventos y Estado*). Al igual que `for`, **`class` es una palabra clave reservada en JavaScript** y, por lo tanto, se usa `className` como nombre de prop en su lugar.

#### Establecer estilos dinámicamente
Con los estilos en línea y las clases CSS (y los estilos CSS globales en general), existen varias formas de aplicar estilos a los elementos. Hasta ahora, todos los ejemplos han mostrado estilos estáticos, es decir, estilos que nunca cambiarán una vez que se haya cargado una página.

Pero aunque la mayoría de los elementos de la página no cambian sus estilos después de que se carga una página, normalmente también tienes algunos elementos a los que se les debe dar estilo de forma dinámica o condicional. Aquí tienes algunos ejemplos:
- Una aplicación de tareas pendientes donde las diferentes prioridades de las tareas reciben diferentes colores.
- Un formulario de entrada donde los elementos de formulario no válidos deben resaltarse después de un envío de formulario no exitoso.
- Un juego basado en la web donde los jugadores pueden elegir colores para sus avatares.

En tales casos, aplicar estilos estáticos no es suficiente y se deben usar estilos dinámicos en su lugar. Establecer estilos dinámicamente es sencillo. Nuevamente, se trata simplemente de aplicar los conceptos clave de React cubiertos anteriormente (más importante aún, los relacionados con el establecimiento de valores dinámicos del Capítulo 2, *Entendiendo los Componentes de React y JSX*, y del Capítulo 4, *Trabajando con Eventos y Estado*).

Aquí hay un ejemplo donde el color de un párrafo se establece dinámicamente según el color que un usuario introduce en un campo de entrada:

```javascript
function ColoredText() { 
  const [enteredColor, setEnteredColor] = useState(''); 

  function handleUpdateTextColor(event) { 
    setEnteredColor(event.target.value); 
  }; 

  return ( 
    <> 
      <input type="text" onChange={handleUpdateTextColor}/> 
      <p style={{color: enteredColor}}>This text's color changes dynamically!</p> 
    </> 
  ); 
};
```

El texto ingresado en el campo `<input>` se almacena en el estado `enteredColor`. Este estado se utiliza luego para establecer la propiedad CSS `color` del elemento `<p>` dinámicamente. Esto se logra pasando un objeto de estilo, con la propiedad `color` establecida en el valor `enteredColor`, como valor para la prop `style` del elemento `<p>`. El color del texto del párrafo se establece, por lo tanto, dinámicamente en el valor ingresado por el usuario (asumiendo que los usuarios ingresan valores de color CSS válidos en el campo `<input>`).

No estás limitado a estilos en línea; las clases CSS también se pueden establecer dinámicamente, como en el siguiente fragmento:

```javascript
function TodoPriority() { 
  const [chosenPriority, setChosenPriority] = useState('low-prio'); 

  function handleChoosePriority(event) { 
    setChosenPriority(event.target.value); 
  }; 

  return ( 
    <> 
      <p className={chosenPriority}>Chosen Priority: {chosenPriority}</p> 
      <select onChange={handleChoosePriority}> 
        <option value="low-prio">Low</option> 
        <option value="high-prio">High</option> 
      </select> 
    </> 
  ); 
};
```

En este ejemplo, el estado `chosenPriority` alternará entre `low-prio` y `high-prio`, según la selección del menú desplegable. El valor del estado se muestra luego como texto dentro del párrafo y también se usa como un nombre de clase CSS dinámico, aplicado al elemento `<p>`. Para que esto tenga algún efecto visual, por supuesto, debe haber clases CSS `low-prio` y `high-prio` definidas en algún archivo CSS o bloque `<style>`. Por ejemplo, considera el siguiente código en `index.css`:

```css
.low-prio { 
  background-color: blue; 
  color: white; 
} 

.high-prio { 
  background-color: red; 
  color: white; 
}
```

#### Estilos condicionales
Estrechamente relacionados con los estilos dinámicos están los estilos condicionales. De hecho, en última instancia, son solo un caso especial de estilos dinámicos. En los ejemplos anteriores, los valores de estilo en línea y los nombres de clase se establecieron como iguales a los valores elegidos o ingresados por el usuario.

Sin embargo, también puedes derivar estilos o nombres de clase dinámicamente en función de diferentes condiciones, como se muestra aquí:

```javascript
function TextInput({isValid, isRecommended, ...props}) { 
  let cssClass = 'input-default'; 

  if (isRecommended) { 
    cssClass = 'input-recommended'; 
  } 

  if (!isValid) { 
    cssClass = 'input-invalid'; 
  } 

  return <input className={cssClass} {...props} /> 
};
```

En este ejemplo, se construye un componente envoltorio alrededor del elemento estándar `<input>`. (Para obtener más información sobre los componentes envoltorios, consulta el Capítulo 3, *Componentes y Props*). El propósito principal de este componente envoltorio es establecer algunos estilos predeterminados para el elemento `<input>` envuelto. El componente envoltorio está diseñado para proporcionar un elemento de entrada preestilizado que se pueda utilizar en cualquier lugar de la aplicación. De hecho, proporcionar elementos preestilizados es uno de los casos de uso más comunes y populares para crear componentes envoltorios.

En este ejemplo concreto, los estilos predeterminados se aplican mediante clases CSS. Si el valor de la prop `isValid` es `true` y el valor de la prop `isRecommended` es `false`, la clase CSS `input-default` se aplicará al elemento `<input>`, ya que ninguna de las dos sentencias `if` se activa.

Si `isRecommended` es `true` (pero `isValid` no es falso), se aplicaría la clase CSS `input-recommended`. Si `isValid` es `false`, se agrega la clase `input-invalid` en su lugar. Por supuesto, las clases CSS deben definirse en algunos archivos CSS importados (por ejemplo, en `index.css`).

Los estilos en línea se pueden establecer de manera similar, como se muestra en el siguiente fragmento:

```javascript
function TextInput({isValid, isRecommended, ...props}) { 
  let bgColor = 'black'; 

  if (isRecommended) { 
    bgColor = 'blue'; 
  } 

  if (!isValid) { 
    bgColor = 'red'; 
  } 

  return <input style={{backgroundColor: bgColor}} {...props} /> 
};
```

En este ejemplo, el color de fondo del elemento `<input>` se establece condicionalmente, según los valores recibidos a través de las props `isValid` e `isRecommended`.

#### Combinar múltiples clases CSS dinámicas
En los ejemplos anteriores, se establecía un máximo de una clase CSS dinámicamente a la vez. Sin embargo, no es raro encontrar escenarios en los que se deben fusionar y agregar múltiples clases CSS derivadas dinámicamente a un elemento.

Considera el siguiente ejemplo:

```javascript
function ExplanationText({children, isImportant}) { 
  const defaultClasses = 'text-default text-expl'; 
  return <p className={defaultClasses}>{children}</p>; 
}
```

Aquí, se agregan dos clases CSS a `<p>` simplemente combinándolas en una sola cadena. Alternativamente, podrías agregar directamente una cadena con las dos clases de esta manera:

```javascript
return <p className="text-default text-expl">{children}</p>;
```

Este código funcionará, pero ¿qué sucede si el objetivo también es agregar otro nombre de clase a la lista de clases, según el valor de la prop `isImportant` (que se ignora en el ejemplo anterior)?

Reemplazar la lista de clases predeterminada es fácil, como has aprendido:

```javascript
function ExplanationText({children, isImportant}) { 
  let cssClasses = 'text-default text-expl'; 
  if (isImportant) { 
    cssClasses = 'text-important'; 
  } 
  return <p className={cssClasses}>{children}</p>; 
}
```

¿Pero qué pasa si el objetivo no es reemplazar la lista de clases predeterminadas? ¿Qué pasa si se debe agregar `text-important` como una clase a `<p>`, además de `text-default` y `text-expl`?

La prop `className` espera recibir un valor de cadena, por lo que pasar un array de clases no es una opción directa. Sin embargo, simplemente puedes fusionar varias clases en una sola cadena, y hay un par de formas diferentes de hacerlo:
- **Concatenación de cadenas**:
  ```javascript
  cssClasses = cssClasses + ' text-important';
  ```
- **Uso de literales de plantilla (*template literals*)**:
  ```javascript
  cssClasses = `${cssClasses} text-important`;
  ```
- **Unir un array (*join*)**:
  ```javascript
  cssClasses = [cssClasses, 'text-important'].join(' ');
  ```

Todos estos ejemplos podrían usarse dentro de la sentencia `if` (`if (isImportant)`) para agregar condicionalmente la clase `text-important`, según el valor de la prop `isImportant`. Los tres enfoques, así como las variaciones de estos enfoques, funcionarán porque todos estos enfoques producen una cadena. En general, cualquier enfoque que produzca una cadena se puede utilizar para generar valores para `className`.

#### Fusionar múltiples objetos de estilo en línea
Al trabajar con estilos en línea, en lugar de clases CSS, también puedes fusionar varios objetos de estilo. La principal diferencia es que no produces una cadena con todos los valores sino, más bien, un objeto con todos los valores de estilo combinados.

Esto se puede lograr utilizando técnicas estándar de JavaScript para fusionar varios objetos en un solo objeto. La técnica más popular implica el uso del operador de propagación (*spread operator*), como se muestra en este ejemplo:

```javascript
function ExplanationText({children, isImportant}) { 
  let defaultStyle = { 
    color: 'black' 
  }; 

  if (isImportant) { 
    defaultStyle = { 
      ...defaultStyle, 
      backgroundColor: 'red' 
    }; 
  } 

  return <p style={defaultStyle}>{children}</p>; 
}
```

Aquí observarás que `defaultStyle` es un objeto con una propiedad `color`. Si `isImportant` es `true`, se reemplaza con un objeto que contiene todas las propiedades que tenía antes (a través del operador de propagación, `...defaultStyle`), así como la propiedad `backgroundColor`.

> [!NOTE]
> Para obtener más información sobre la función y el uso del operador de propagación, consulta el Capítulo 5, *Renderizado de Listas y Contenido Condicional*.

#### Construcción de componentes con estilos personalizables
Como ya sabes, los componentes se pueden reutilizar. Esto se ve respaldado por el hecho de que se pueden configurar mediante props. El mismo componente se puede utilizar en diferentes lugares de una página con diferentes configuraciones para producir una salida diferente.

Dado que los estilos se pueden establecer tanto de forma estática como dinámica, también puedes hacer que el estilo de tus componentes sea personalizable. Los ejemplos anteriores ya muestran dicha personalización en acción; por ejemplo, la prop `isImportant` se utilizó en el ejemplo anterior para agregar condicionalmente un color de fondo rojo a un párrafo. El componente `ExplanationText`, por lo tanto, ya permite la personalización indirecta del estilo a través de la prop `isImportant`.

Además de esta forma de personalización, también podrías crear componentes que acepten props que ya contengan nombres de clases CSS u objetos de estilo. Por ejemplo, el siguiente componente envoltorio acepta una prop `className` que se fusiona con una clase CSS predeterminada (`btn`):

```javascript
function Button({children, config, className}) { 
  return <button {...config} className={`btn ${className}`}>{children}</button>; 
};
```

Este componente podría utilizarse en otro componente de la siguiente manera:

```javascript
<Button config={{onClick: doSomething}} className="btn-alert">Click me!</Button>
```

Si se usa de esta manera, el elemento `<button>` final recibiría tanto la clase `btn` como la clase `btn-alert`.

No tienes que usar `className` como nombre de prop; se puede usar cualquier nombre, ya que es tu componente. Sin embargo, no es una mala idea usar `className` porque así puedes mantener tu modelo mental de establecer clases CSS a través de `className` (para componentes integrados, no tendrás esa opción).

En lugar de fusionar los valores de las props con nombres de clases CSS u objetos de estilo predeterminados, puedes sobrescribir los valores predeterminados. Esto te permite crear componentes que vienen con algún estilo listo para usar sin imponer ese estilo de manera obligatoria:

```javascript
function Button({children, config, className}) { 
  let cssClasses = 'btn'; 

  if (className) { 
    cssClasses = className; 
  } 

  return <button {...config} className={cssClasses}>{children}</button>; 
};
```

Puedes ver cómo todos los diferentes conceptos tratados a lo largo de este libro se unen aquí: las props permiten la personalización, los valores se pueden establecer, intercambiar y cambiar de forma dinámica y condicional y, por lo tanto, se pueden crear componentes altamente reutilizables y configurables.

#### Personalización con opciones de configuración fijas
Además de exponer props como `className` o `style`, que se fusionan con otras clases o estilos definidos dentro de una función de componente, también puedes crear componentes que apliquen diferentes estilos o nombres de clase basados en otros valores de props.

Esto se ha mostrado en los ejemplos anteriores donde se utilizaron props como `isValid` o `isImportant` para aplicar ciertos estilos condicionalmente. Esta forma de aplicar estilos podría denominarse, por tanto, "estilo indirecto" (aunque este no es un término oficial).

Ambos enfoques pueden destacar en diferentes circunstancias. Para los componentes envoltorios, por ejemplo, aceptar props `className` o `style` (que se pueden fusionar con otros estilos dentro del componente) permite que el componente se use exactamente como un componente integrado (por ejemplo, como el componente que envuelve). El estilo indirecto, por otro lado, puede ser muy útil si deseas crear componentes que proporcionen un par de variaciones predefinidas.

Un buen ejemplo es un cuadro de texto que proporciona dos temas integrados que se pueden seleccionar a través de una prop específica:

**Figura 6.4**: Un `TextBox` recibe estilo según el valor de la prop "mode".

El código para el componente `TextBox` podría verse así:

```javascript
function TextBox({children, mode}) { 
  let cssClasses; 

  if (mode === 'alert') { 
    cssClasses = 'box-alert'; 
  } else if (mode === 'info') { 
    cssClasses = 'box-info'; 
  } 

  return <p className={cssClasses}>{children}</p>; 
};
```

Este componente `TextBox` siempre produce un elemento de párrafo. Si la prop `mode` se establece en cualquier valor que no sea `'alert'` o `'info'`, el párrafo no recibe ningún estilo especial. Pero si `mode` es igual a `'alert'` o `'info'`, se agregan clases CSS específicas al párrafo.

Este componente, por lo tanto, no permite el estilo directo a través de alguna prop `className` o `style` que se fusionaría, pero sí ofrece diferentes variaciones o temas que se pueden configurar con la ayuda de una prop específica (la prop `mode` en este caso).

---

### Sección 3: El problema de los estilos no acotados (sin ámbito local)

Si consideras los diferentes ejemplos con los que has tratado hasta ahora en este capítulo, hay un caso de uso específico que ocurre con bastante frecuencia: **los estilos son relevantes solo para un componente específico**.

Por ejemplo, en el componente `TextBox` de la sección anterior, `'box-alert'` y `'box-info'` son clases CSS que probablemente solo sean relevantes para este componente específico y su marcado. Si a cualquier otro elemento JSX de la aplicación se le aplicara una clase `'box-alert'` (aunque eso fuera poco probable), probablemente no debería tener el mismo estilo que el elemento `<p>` en el componente `TextBox`.

Los estilos de diferentes componentes podrían entrar en conflicto entre sí y sobrescribirse mutuamente porque los estilos no están acotados (*scoped*, es decir, restringidos) a un componente específico. Los estilos CSS son siempre globales, a menos que se utilicen estilos en línea (lo cual se desaconseja, como se mencionó anteriormente).

Al trabajar con bibliotecas basadas en componentes como React, esta falta de acotamiento es un problema común. Es fácil escribir estilos conflictivos a medida que aumentan el tamaño y la complejidad de las aplicaciones (o, en otras palabras, a medida que se agregan más y más componentes a la base de código de una aplicación React).

Es por eso que los miembros de la comunidad de React han desarrollado varias soluciones para este problema. Las siguientes son tres de las soluciones más populares:
1. **CSS Modules** (admitido de forma predeterminada en proyectos de React creados con Vite).
2. **Styled Components** (utilizando una biblioteca de terceros llamada `styled-components`).
3. **Tailwind CSS** (una biblioteca de CSS muy popular).

#### Estilos acotados con CSS Modules
**CSS Modules** es el nombre de un enfoque en el que los archivos CSS individuales se vinculan a archivos JavaScript específicos y a los componentes definidos en esos archivos. Este enlace se establece transformando los nombres de las clases CSS, de modo que cada archivo JavaScript reciba sus propios nombres de clases CSS únicos. Esta transformación se realiza automáticamente como parte del flujo de trabajo de compilación del código. Por lo tanto, una configuración de proyecto determinada debe admitir CSS Modules realizando la transformación de nombres de clase CSS descrita. Los proyectos creados a través de Vite admiten CSS Modules de forma predeterminada.

**Figura 6.5**: CSS Modules en acción. Los nombres de las clases CSS se transforman en nombres únicos durante el flujo de compilación.

Los CSS Modules se habilitan y utilizan nombrando los archivos CSS de una manera muy específica y claramente definida: **`<cualquiercosa>.module.css`**. `<cualquiercosa>` es cualquier valor de tu elección, pero la parte `.module` delante de la extensión del archivo es obligatoria, ya que indica (al flujo de compilación del proyecto) que este archivo CSS debe transformarse de acuerdo con el enfoque de CSS Modules.

Por lo tanto, los archivos CSS nombrados de esta manera deben importarse a los componentes de una manera específica:

```javascript
import classes from './file.module.css';
```

Esta sintaxis de importación es diferente de la sintaxis de importación mostrada al comienzo de esta sección para `index.css`:

```javascript
import './index.css';
```

Al importar archivos CSS como se muestra en el segundo fragmento, el código CSS simplemente se fusiona en el archivo `index.html` y se aplica globalmente. Al usar CSS Modules en su lugar (primer fragmento de código), los nombres de clases CSS definidos en el archivo CSS importado se transforman de modo que sean únicos para el archivo JS que importa el archivo CSS.

Dado que los nombres de las clases CSS se transforman y, por lo tanto, ya no son iguales a los nombres de clase que definiste en el archivo CSS, importas un objeto (`classes`, en el ejemplo anterior) desde el archivo CSS. Este objeto expone todos los nombres de clases CSS transformados bajo claves que coinciden con los nombres de clases CSS definidos por ti en el archivo CSS. Los valores de esas propiedades son los nombres de clase transformados (como cadenas).

Aquí hay un ejemplo completo, comenzando con un archivo CSS específico del componente (`TextBox.module.css`):

```css
.alert { 
  padding: 1rem; 
  border-radius: 6px; 
  background-color: #f9bcb5; 
  color: #480c0c; 
} 

.info { 
  padding: 1rem; 
  border-radius: 6px; 
  background-color: #d6aafa; 
  color: #410474; 
}
```

El archivo JavaScript (`TextBox.jsx`) para el componente al que debe pertenecer el código CSS se ve así:

```javascript
import classes from './TextBox.module.css'; 

function TextBox({ children, mode }) { 
  let cssClasses; 

  if (mode === 'alert') { 
    cssClasses = classes.alert; 
  } else if (mode === 'info') { 
    cssClasses = classes.info; 
  } 

  return <p className={cssClasses}>{children}</p>; 
} 

export default TextBox;
```

> [!NOTE]
> El código de ejemplo completo también se puede encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/examples/01-css-modules-intro](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/examples/01-css-modules-intro).

Si inspeccionas el elemento de texto renderizado en las herramientas de desarrollo del navegador, notarás que el nombre de la clase CSS aplicada al elemento `<p>` no coincide con el nombre de clase especificado en el archivo `TextBox.module.css`:

**Figura 6.6**: El nombre de la clase CSS se transforma debido al uso de CSS Modules.

Este es el caso porque, como se describió anteriormente, el nombre de la clase se transformó durante el proceso de compilación para que fuera único. Si cualquier otro archivo CSS, importado por otro archivo JavaScript, definiera una clase con el mismo nombre (`info` en este caso), los estilos no chocarían ni se sobrescribirían entre sí, ya que los nombres de clase interfirientes se transformarían en diferentes nombres de clase antes de aplicarse a los elementos del DOM.

De hecho, en el ejemplo proporcionado en GitHub, puedes encontrar otra clase CSS `info` definida en el archivo `index.css`:

```css
.info { 
  border: 5px solid red; 
}
```

Ese archivo todavía se importa en `main.jsx` y, por lo tanto, sus estilos se aplican globalmente a todo el documento. No obstante, los estilos `.info` claramente no afectan al `<p>` renderizado por `TextBox` (no hay borde rojo alrededor del cuadro de texto en la Figura 6.6). No están afectando a ese elemento porque ya no tiene una clase `info`; la clase fue renombrada `_info_1mtzh_8` por el flujo de compilación (aunque el nombre que veas diferirá, ya que contiene un elemento aleatorio).

También vale la pena señalar que el archivo `index.css` todavía se importa en `main.jsx`, como se muestra al principio de este capítulo. La declaración de importación no se cambia a `import classes from './index.css';`, ni el archivo CSS se llama `index.module.css`.

Ten en cuenta también que puedes utilizar CSS Modules para acotar estilos a componentes y también puedes mezclar el uso de CSS Modules con archivos CSS normales, que se importan a archivos JavaScript sin utilizar CSS Modules (es decir, sin acotamiento).

Otro aspecto importante del uso de CSS Modules es que **solo puedes usar selectores de clase CSS** (es decir, en tus archivos `.module.css`) porque CSS Modules depende de las clases CSS. Puedes escribir selectores que combinen clases con otros selectores, como `input.invalid`, pero no puedes agregar selectores que no usen clases en absoluto en tus archivos `.module.css`. Por ejemplo, los selectores `input { ... }` o `#some-id { ... }` no funcionarían aquí.

CSS Modules es una forma muy popular de acotar estilos a componentes (de React) y se utilizará en muchos ejemplos durante el resto de este libro.

#### La biblioteca styled-components
La biblioteca `styled-components` es una solución denominada **CSS-in-JS**. Las soluciones CSS-in-JS tienen como objetivo eliminar la separación entre el código CSS y el código JavaScript fusionándolos en el mismo archivo. Los estilos de los componentes se definirían justo al lado de la lógica del componente. Depende de la preferencia personal si favoreces la separación (como se impone mediante el uso de archivos CSS) o mantener los dos lenguajes juntos.

Dado que `styled-components` es una biblioteca de terceros que no está preinstalada en proyectos de React recién creados, debes instalar esta biblioteca como primer paso si deseas utilizarla. Esto se puede hacer a través de npm (que se instaló automáticamente junto con Node.js en el Capítulo 1, *React – Qué es y por qué*):

```bash
npm install styled-components
```

La biblioteca `styled-components` proporciona esencialmente componentes envoltorios alrededor de todos los componentes centrales integrados (es decir, alrededor de `p`, `a`, `button`, `input`, etc.). Expone todos estos componentes envoltorios como *plantillas etiquetadas* (*tagged templates*): funciones de JavaScript que no se llaman como funciones regulares sino que se ejecutan agregando comillas invertidas (un literal de plantilla) justo después del nombre de la función, por ejemplo, ``doSomething`text data` ``.

> [!NOTE]
> Las plantillas etiquetadas pueden resultar confusas cuando las ves por primera vez, especialmente porque es una característica de JavaScript que no se usa con demasiada frecuencia. Es muy probable que no hayas trabajado con ellas muy seguido. Es aún más probable que nunca antes hayas creado una plantilla etiquetada personalizada. Puedes obtener más información sobre las plantillas etiquetadas en esta excelente documentación en MDN en [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates).

Aquí hay un componente que importa y usa `styled-components` para establecer y acotar el estilo:

```javascript
import styled from 'styled-components'; 

const Button = styled.button` 
  background-color: #370566; 
  color: white; 
  border: none; 
  padding: 1rem; 
  border-radius: 4px; 
`; 

export default Button;
```

Este componente no es una función de componente sino, más bien, una constante que almacena el valor devuelto al ejecutar la plantilla etiquetada `styled.button`. Esa plantilla etiquetada devuelve una función de componente que produce un elemento `<button>`. Los estilos pasados a través de la plantilla etiquetada (es decir, dentro del literal de plantilla) se aplican a ese elemento de botón devuelto. Puedes ver esto si inspeccionas el botón en las herramientas de desarrollo del navegador:

**Figura 6.7**: El elemento de botón renderizado recibe los estilos de componente definidos.

En la Figura 6.7, también puedes ver cómo la biblioteca `styled-components` aplica tus estilos al elemento. Extrae tus definiciones de estilo de la cadena de la plantilla etiquetada y las inyecta en un elemento `<style>` en la sección `<head>` del documento. Los estilos inyectados se aplican luego a través de un selector de clase generado (y nombrado) por la biblioteca `styled-components`. Finalmente, la biblioteca agrega el nombre de clase CSS generado automáticamente al elemento (`<button>`, en este caso).

Los componentes expuestos por la biblioteca `styled-components` propagan cualquier prop adicional que pases a un componente sobre el componente central envuelto. Además, cualquier contenido insertado entre las etiquetas de apertura y cierre también se inserta entre las etiquetas del componente envuelto.

Por eso el `Button` creado anteriormente se puede usar así sin agregarle ninguna lógica adicional:

```javascript
import Button from './components/button.jsx'; 

function App() { 
  function handleClick() { 
    console.log('This button was clicked!'); 
  } 

  return <Button onClick={handleClick}>Click me!</Button>; 
} 

export default App;
```

> [!NOTE]
> El código de ejemplo completo se puede encontrar en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/examples/02-styled-components-intro](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/examples/02-styled-components-intro).

Puedes hacer más con la biblioteca `styled-components`. Por ejemplo, puedes establecer estilos de forma dinámica y condicional. Sin embargo, este libro no trata principalmente sobre esa biblioteca. Es solo una de las muchas alternativas a los CSS Modules. Por lo tanto, se recomienda que explores la documentación oficial de `styled-components` si deseas obtener más información, la cual puedes encontrar en [https://styled-components.com/](https://styled-components.com/).

#### Uso de la biblioteca Tailwind CSS para estilos
Acotar estilos con la ayuda de CSS Modules o la biblioteca `styled-components` es una técnica muy útil y popular.

Pero no importa qué enfoque uses, debes escribir todo el código CSS por tu cuenta. Por lo tanto, por supuesto, necesitas saber CSS.

¿Pero qué pasa si no lo sabes? ¿O si simplemente no te gusta escribir código CSS?

En ese caso, puedes utilizar una de las muchas bibliotecas y marcos de CSS disponibles, por ejemplo, el marco Bootstrap CSS o la biblioteca **Tailwind CSS**. Tailwind se ha convertido en una solución de estilos muy popular para proyectos de React (para desarrolladores que no quieren escribir código CSS personalizado).

Ten en cuenta que Tailwind es una biblioteca de CSS que en realidad no está enfocada exclusivamente en React. En cambio, puedes usar Tailwind en cualquier proyecto web para dar estilo a tu código HTML, sin importar qué biblioteca o marco de JavaScript (si corresponde) se esté utilizando allí.

Pero Tailwind es una opción común para las aplicaciones de React, ya que su filosofía central combina muy bien con el modelo enfocado en componentes de React. Esto se debe a que, al usar Tailwind para dar estilo, normalmente compones estilos generales aplicando muchas clases CSS pequeñas a elementos JSX individuales:

```javascript
function App() { 
  return ( 
    <main className="bg-gray-200 text-gray-900 h-screen p-12 text-center"> 
      <h1 className="font-bold text-4xl">Tailwind CSS is amazing!</h1> 
      <p className="text-gray-600"> 
        It may take a while to get used to it. But it's great for people who don't want to write custom CSS code. 
      </p> 
    </main> 
  ); 
} 

export default App;
```

Al encontrar por primera vez código que utiliza Tailwind CSS, la larga lista de clases CSS puede parecer intimidante y caótica. Pero al trabajar con Tailwind, normalmente te acostumbras rápidamente.

Además, el enfoque de Tailwind ofrece muchas ventajas:
- No necesitas aprender CSS en detalle: comprender la sintaxis de Tailwind, que es menos compleja que escribir CSS desde cero, es suficiente.
- Compones estilos combinando clases CSS, de forma similar a como compones interfaces de usuario a partir de componentes en React.
- No tienes que alternar entre archivos JSX y CSS.
- Los cambios de estilo se pueden aplicar y probar muy rápidamente.

Como puedes ver en el fragmento de código anterior, la idea central de Tailwind esencialmente es que proporciona muchas clases CSS combinables que cada una hace muy poco. Por ejemplo, la clase `bg-gray-200` simplemente establece el color de fondo en un cierto tono de gris, y nada más.

Por lo tanto, es la combinación de todas esas clases CSS lo que logra una apariencia determinada, y Tailwind CSS ofrece muchas de estas clases que puedes usar y combinar. Encontrarás una lista completa en la documentación oficial en [https://tailwindcss.com/docs/utility-first](https://tailwindcss.com/docs/utility-first).

Al trabajar con Tailwind en proyectos de React, puedes crear componentes de React no solo para reutilizar lógica o marcado JSX, sino también estilos:

```javascript
function Item({ children }) { 
  return <li className='p-1 my-2 bg-stone-100'>{children}</li>; 
} 

function App() { 
  return ( 
    <main className="bg-gray-200 text-gray-900 h-screen p-12 text-center"> 
      <h1 className="font-bold text-4xl">Tailwind CSS is amazing!</h1> 
      <p className="text-gray-600"> 
        It may take a while to get used to it. But it's great for people who don't want to write custom CSS code. 
      </p> 
      <section className="mt-10 border border-gray-600 max-w-3xl mx-auto p-4 rounded-md bg-gray-300"> 
        <h2 className="font-bold text-xl">Tailwind CSS Advantages</h2> 
        <ul className="mt-4"> 
          <Item>No CSS knowledge required</Item> 
          <Item>Compose styles by combining "small" CSS classes</Item> 
          <Item> Never leave your JSX code - no need to fiddle around in extra CSS files </Item> 
          <Item>Quickly test and apply changes</Item> 
        </ul> 
      </section> 
    </main> 
  ); 
} 

export default App;
```

En este ejemplo, el componente `Item` está diseñado para reutilizar los estilos de Tailwind aplicados al elemento `<li>`.

> [!NOTE]
> También puedes encontrar este proyecto de ejemplo en GitHub: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/examples/03-tailwind](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/examples/03-tailwind).

Si planeas usar Tailwind en tu proyecto de React, debes instalarlo como primer paso. Las instrucciones de instalación detalladas para una amplia variedad de configuraciones de proyectos se pueden encontrar en la documentación oficial; esto incluye instrucciones para proyectos Vite: [https://tailwindcss.com/docs/guides/vite](https://tailwindcss.com/docs/guides/vite).

El proceso de instalación no es tan simple como simplemente importar un archivo CSS, pero no obstante es relativamente sencillo. Requiere un par de pasos de configuración, ya que Tailwind necesita conectarse al proceso de compilación del proyecto para analizar tus archivos JSX y producir código CSS que contenga todos los nombres de clase y reglas de estilo utilizados.

Además de ofrecer muchos estilos de utilidad que se pueden combinar, Tailwind también ofrece muchas oportunidades de personalización y opciones de configuración. Por lo tanto, se podrían escribir libros enteros solo sobre Tailwind. Sin embargo, por supuesto, no es de eso de lo que trata este libro. Por lo tanto, si estás interesado en usar Tailwind en tus proyectos de React, la documentación oficial de Tailwind (consulta los enlaces anteriores) es un excelente lugar para aprender más.

#### Uso de otras bibliotecas y marcos de CSS o JavaScript
Obviamente, se reduce a preferencias personales si deseas escribir código CSS personalizado (potencialmente acotado con CSS Modules o `styled-components`) o si deseas trabajar con bibliotecas CSS de terceros, como Tailwind CSS. No hay una opción correcta o incorrecta, y verás todo tipo de enfoques utilizados en diferentes proyectos de React.

Las opciones presentadas en este capítulo tampoco son exhaustivas; también hay otros tipos de bibliotecas de CSS y JavaScript:
- **Bibliotecas de utilidad** que resuelven problemas de CSS muy específicos, independientemente de si las estás usando en un proyecto de React (por ejemplo, `Animate.css`, que ayuda a agregar animaciones).
- **Otros marcos o bibliotecas de CSS** que proporcionan una amplia variedad de clases CSS preconstruidas que se pueden aplicar a los elementos para lograr rápidamente un aspecto determinado (por ejemplo, `Bootstrap`).
- **Bibliotecas de JavaScript** que ayudan con el estilo o aspectos de estilo específicos como las animaciones (por ejemplo, `Framer Motion`).

Algunas bibliotecas y marcos tienen extensiones específicas de React o admiten específicamente React, pero eso no significa que no puedas usar bibliotecas que no tengan esto.

---

### Sección 4: Resumen y puntos clave

- Se puede utilizar CSS estándar para dar estilo a los componentes de React y los elementos JSX.
- Los archivos CSS generalmente se importan directamente en archivos JavaScript, lo cual es posible gracias al proceso de compilación del proyecto, que extrae el código CSS y lo inyecta en el documento (el archivo HTML).
- Como alternativa a los estilos CSS globales (con selectores de elementos, id, clases u otros), se pueden usar estilos en línea (*inline styles*) para aplicar estilos a elementos JSX.
- Al usar clases CSS para dar estilo, debes usar la prop `className` (no `class`).
- Los estilos se pueden establecer de forma estática y dinámica o condicional con la misma sintaxis que se utiliza para inyectar otros valores dinámicos o condicionales en el código JSX: un par de llaves `{}`.
- Se pueden crear componentes personalizados altamente configurables estableciendo estilos (o clases CSS) según los valores de las props, o fusionando los valores de las props recibidos con otros estilos o cadenas de nombres de clases.
- Al usar solo CSS, el choque de nombres de clases CSS puede ser un problema.
- Los **CSS Modules** resuelven este problema transformando los nombres de clase en nombres únicos (por componente) como parte del flujo de compilación.
- Alternativamente, se pueden utilizar bibliotecas de terceros como `styled-components`. Esta biblioteca es una solución de CSS-in-JS que también tiene la ventaja o desventaja (según tu preferencia) de eliminar la separación entre el código JS y CSS.
- **Tailwind CSS** es otra opción de estilo popular para proyectos de React: es una biblioteca que te permite componer estilos combinando muchas clases CSS pequeñas.
- También se pueden utilizar otras bibliotecas o marcos de CSS; React no impone ninguna restricción al respecto.

---

### Sección 5: ¿Qué sigue?

Con el tema de estilos cubierto, ahora eres capaz de crear interfaces de usuario no solo funcionales sino también visualmente atractivas. Incluso si a menudo trabajas con diseñadores web dedicados o expertos en CSS, normalmente necesitas poder configurar y asignar estilos (dinámicamente) que se te entregan.

Siendo el estilo un concepto general que es relativamente independiente de React, el próximo capítulo volverá a sumergirse en características y temas más específicos de React. Aprenderás sobre **portales y refs**, que son dos conceptos clave integrados en React. Descubrirás qué problemas resuelven estos conceptos y cómo se utilizan las dos funciones.

---

### Sección 6: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/06-styling/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/06-styling/exercises/questions-answers.md).

1. ¿Con qué lenguaje se definen los estilos para los componentes de React?
2. ¿Qué diferencia importante, en comparación con los proyectos sin React, debe tenerse en cuenta cuando se trata de asignar clases a los elementos?
3. ¿Cómo se pueden asignar estilos de forma dinámica y condicional?
4. ¿Qué significa "acotamiento" (*scoping*) en el contexto de los estilos?
5. ¿Cómo se podrían acotar los estilos a los componentes? Explica brevemente al menos un concepto que ayude con el acotamiento.

---

### Sección 7: Aplica lo aprendido

Ahora no solo eres capaz de crear interfaces de usuario interactivas, sino también de dar estilo a esos elementos de la interfaz de usuario de formas atractivas. Puedes configurar y cambiar esos estilos dinámicamente o según condiciones.

En esta sección, encontrarás dos actividades que te permitirán aplicar tus conocimientos recién adquiridos en combinación con lo que aprendiste en capítulos anteriores.

#### Actividad 6.1: Proporcionar información sobre la validez de la entrada tras el envío del formulario
En esta actividad, construirás un formulario básico que permite a los usuarios ingresar una dirección de correo electrónico y una contraseña. La entrada proporcionada en cada campo de entrada se valida y el resultado de la validación se almacena (para cada campo de entrada individual).

El objetivo de esta actividad es agregar algunos estilos generales de formulario y algunos estilos condicionales que se activen una vez que se haya enviado un formulario no válido. Los estilos exactos dependen de ti, pero para resaltar los campos de entrada no válidos, se debe cambiar el color de fondo del campo de entrada afectado, así como su color de borde y el color del texto de la etiqueta relacionada.

Los pasos son los siguientes:
1. Crea un nuevo proyecto de React y agrégale un componente de formulario.
2. Muestra el componente de formulario en el componente raíz del proyecto.
3. En el componente de formulario, muestra un formulario que contenga dos campos de entrada: uno para ingresar una dirección de correo electrónico y otro para ingresar una contraseña.
4. Agrega etiquetas a los campos de entrada.
5. Almacena los valores ingresados y verifica su validez tras el envío del formulario (puedes ser creativo al formar tu propia lógica de validación).
6. Elige las clases CSS apropiadas del archivo `index.css` proporcionado (alternativamente, también puedes escribir tus propias clases).
7. Agrégalas a los campos de entrada no válidos y a sus etiquetas una vez que se hayan enviado valores no válidos.

La interfaz de usuario final debería verse así:

**Figura 6.8**: La interfaz de usuario final con valores de entrada no válidos resaltados en rojo.

Dado que este libro no trata sobre CSS y es posible que no seas un experto en CSS, puedes usar el archivo `index.css` de la solución y concentrarte en la lógica de React para aplicar las clases CSS adecuadas a los elementos JSX.

> [!NOTE]
> Todos los archivos de código utilizados para esta actividad y una solución completa se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/activities/practice-1).

#### Actividad 6.2: Uso de CSS Modules para el acotamiento de estilos
En esta actividad, tomarás la aplicación final creada en la Actividad 6.1 y la ajustarás para usar CSS Modules. El objetivo es migrar todos los estilos específicos del componente a un archivo CSS específico del componente, que utiliza CSS Modules para el acotamiento del estilo.

Por lo tanto, la interfaz de usuario final se ve igual que en la actividad anterior. Sin embargo, los estilos estarán acotados al componente `Form` para que los nombres de clase en conflicto no interfieran con el estilo.

Los pasos son los siguientes:
1. Termina la actividad anterior o toma el código terminado de GitHub.
2. Identifica los estilos que pertenecen específicamente al componente `Form` y muévelos a un nuevo archivo CSS específico del componente.
3. Cambia los selectores de CSS a selectores de nombres de clase y agrega clases a los elementos JSX según sea necesario (esto se debe a que CSS Modules requiere selectores de nombres de clase).
4. Usa el archivo CSS específico del componente como se explica a lo largo de este capítulo y asigna las clases CSS a los elementos JSX apropiados.

> [!NOTE]
> Todos los archivos de código utilizados para esta actividad y una solución completa se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/activities/practice-2](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/06-styling/activities/practice-2).
