# Parte 1: Fundamentos de React

## Capítulo 5: Renderizado de Listas y Contenido Condicional

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Mostrar contenido dinámico de forma condicional.
- Renderizar listas de datos y mapear elementos de listas a elementos JSX.
- Optimizar listas para que React pueda actualizar eficientemente la interfaz de usuario cuando sea necesario.

---

### Sección 1: Introducción

En este punto del libro, ya estás familiarizado con varios conceptos clave, incluidos los componentes, las props, el estado y los eventos, con los cuales tienes todas las herramientas esenciales necesarias para construir todo tipo de aplicaciones y sitios web en React. También has aprendido a mostrar valores y resultados dinámicos como parte de la interfaz de usuario.

Sin embargo, hay dos temas relacionados con la visualización de datos dinámicos que aún no se han analizado en profundidad: **mostrar contenido de forma condicional** y **renderizar datos de listas**. Dado que la mayoría (si no todas) las aplicaciones web y sitios web que construyas requerirán al menos uno de estos dos conceptos, es crucial saber cómo trabajar con contenido condicional y datos de listas.

En este capítulo, por lo tanto, aprenderás cómo renderizar y mostrar diferentes elementos de la interfaz de usuario (e incluso secciones enteras de la interfaz de usuario), basados en condiciones dinámicas. Además, aprenderás a mostrar listas de datos (como una lista de tareas pendientes con sus elementos) y renderizar elementos JSX dinámicamente para los elementos que componen una lista. Este capítulo también explorará prácticas recomendadas importantes relacionadas con la visualización de listas y contenido condicional.

---

### Sección 2: ¿Qué son el contenido condicional y los datos de listas?

Antes de profundizar en las técnicas para mostrar contenido condicional o datos de listas, es importante comprender qué se entiende exactamente por estos términos.

El **contenido condicional** significa simplemente cualquier tipo de contenido que solo debe mostrarse bajo ciertas circunstancias. Algunos ejemplos son los siguientes:
- Mensajes o ventanas modales de error que solo deben aparecer si un usuario envía datos incorrectos en un formulario.
- Campos de entrada de formulario adicionales que aparecen una vez que el usuario elige ingresar detalles adicionales (como datos de facturación de empresa).
- Un indicador de carga (*spinner*) que se muestra mientras los datos se envían o se obtienen desde un servidor backend.
- Un menú de navegación lateral que se despliega cuando el usuario hace clic en un botón de menú.

Esta es solo una lista muy corta de algunos ejemplos. Por supuesto, se te podrían ocurrir cientos de ejemplos adicionales. Pero debería quedar claro de qué se tratan todos estos ejemplos al final: elementos visuales o secciones enteras de la interfaz de usuario que solo se muestran si se cumplen ciertas condiciones.

En el primer ejemplo (una ventana modal de error), la condición sería que un usuario ingresó datos incorrectos en un formulario. El contenido mostrado condicionalmente sería entonces la ventana de error.

El contenido condicional es extremadamente común ya que prácticamente todos los sitios web y aplicaciones web tienen algún contenido que es similar o comparable a los ejemplos anteriores.

Además del contenido condicional, muchos sitios web también muestran **listas de datos**. Puede que no siempre sea inmediatamente obvio, pero si lo piensas, prácticamente no hay ningún sitio web que no muestre algún tipo de datos de listas. Nuevamente, aquí hay algunos ejemplos de datos de listas que pueden mostrarse en un sitio:
- Una tienda en línea que muestra una cuadrícula o lista de productos.
- Un sitio de reservas de eventos que muestra una lista de eventos.
- Un carrito de compras que muestra una lista de artículos en el carrito.
- Una página de pedidos que muestra una lista de pedidos.
- Un blog que muestra una lista de artículos, y tal vez una lista de comentarios debajo de una publicación de blog.
- Una lista de elementos de navegación en el encabezado.

Aquí se podría crear una lista interminable de ejemplos. Las listas están en todas partes en la web. Como muestran los ejemplos anteriores, muchos sitios web (probablemente la mayoría) tienen múltiples listas con varios tipos de datos en el mismo sitio.

Toma una tienda en línea, por ejemplo. Aquí tendrías una lista (o una cuadrícula, que en realidad es solo otro tipo de lista) de productos, una lista de artículos del carrito de compras, una lista de pedidos, una lista de elementos de navegación en el encabezado y ciertamente muchas otras listas también. Por eso es importante que sepas cómo mostrar cualquier tipo de lista con cualquier tipo de datos en interfaces de usuario impulsadas por React.

---

### Sección 3: Renderizar contenido condicionalmente

Imagina el siguiente escenario: tienes un botón que, al hacer clic en él, debería mostrar un cuadro de texto adicional, como se muestra aquí:

**Figura 5.1**: Inicialmente, solo aparece el botón en la pantalla.

Después de hacer clic en el botón, se muestra otro cuadro:

**Figura 5.2**: Después de hacer clic en el botón, se revela el cuadro de información.

Este es un ejemplo muy simple, pero no poco realista. Muchos sitios web tienen partes de la interfaz de usuario que funcionan de esta manera. Mostrar información adicional tras hacer clic en un botón (o alguna interacción similar) es un patrón común. Solo piensa en la información nutricional debajo de un plato en un sitio de pedidos de comida o en una sección de preguntas frecuentes (FAQ) donde las respuestas se muestran después de seleccionar una pregunta.

Entonces, ¿cómo se podría implementar este escenario en una aplicación de React?

Si ignoras el requisito de renderizar parte del contenido condicionalmente, el componente general de React podría verse así:

```javascript
function TermsOfUse() { 
  return ( 
    <section> 
      <button>Show Terms of Use Summary</button> 
      <p>By continuing, you accept that we will not indemnify you for any damage or harm caused by our products.</p> 
    </section> 
  ); 
}
```

Este componente no tiene absolutamente ningún código condicional y, por lo tanto, tanto el botón como el cuadro de información adicional se muestran todo el tiempo.

En este ejemplo, ¿cómo se podría mostrar el párrafo con el texto del resumen de los términos de uso condicionalmente (es decir, solo después de hacer clic en el botón)?

Con los conocimientos adquiridos a lo largo de los capítulos anteriores, especialmente el Capítulo 4, *Trabajando con Eventos y Estado*, ya tienes las habilidades necesarias para mostrar el texto solo después de hacer clic en el botón. El siguiente código muestra cómo se podría reescribir el componente para mostrar el texto completo solo después de hacer clic en el botón:

```javascript
import { useState } from 'react'; 

function TermsOfUse() { 
  const [showTerms, setShowTerms] = useState(false); 

  function handleShowTermsSummary() { 
    setShowTerms(true); 
  } 

  let paragraphText = ''; 

  if (showTerms) { 
    paragraphText = 'By continuing, you accept that we will not indemnify you for any damage or harm caused by our products.'; 
  } 

  return ( 
    <section> 
      <button onClick={handleShowTermsSummary}> 
        Show Terms of Use Summary 
      </button> 
      <p>{paragraphText}</p> 
    </section> 
  ); 
}
```

Partes del código mostrado en este fragmento ya califican como contenido condicional. El valor `paragraphText` se establece condicionalmente, con la ayuda de una sentencia `if` basada en el valor almacenado en el estado `showTerms`.

Sin embargo, el elemento `<p>` en sí no es condicional. Siempre está presente, independientemente de si contiene una oración completa o una cadena vacía. Si abrieras las herramientas de desarrollo del navegador e inspeccionaras esa área de la página, sería visible un elemento de párrafo vacío, como se muestra en la siguiente figura:

**Figura 5.3**: Se renderiza un elemento de párrafo vacío como parte del DOM.

Tener ese elemento `<p>` vacío en el DOM no es lo ideal. Aunque es invisible para el usuario, es un elemento adicional que el navegador debe renderizar. El impacto en el rendimiento será muy probablemente insignificante, pero sigue siendo algo que deberías evitar. Una página web no se beneficia de tener elementos vacíos que no contienen contenido.

Sin embargo, puedes trasladar tus conocimientos sobre valores condicionales (como el texto del párrafo) a **elementos condicionales**. Además de almacenar valores estándar como texto o números en variables, **también puedes almacenar elementos JSX en variables**. Esto es posible porque, como se mencionó en el Capítulo 1, *React – Qué es y por qué*, JSX es solo azúcar sintáctico. Detrás de escena, un elemento JSX es una función estándar de JavaScript que ejecuta React. Además, por supuesto, el valor devuelto por una llamada a una función se puede almacenar en una variable o constante.

Con eso en mente, se podría usar el siguiente código para renderizar todo el párrafo condicionalmente:

```javascript
import { useState } from 'react'; 

function TermsOfUse() { 
  const [showTerms, setShowTerms] = useState(false); 

  function handleShowTermsSummary() { 
    setShowTerms(true); 
  } 

  let paragraph; 

  if (showTerms) { 
    paragraph = <p>By continuing, you accept that we will not indemnify you for any damage or harm caused by our products.</p>; 
  } 

  return ( 
    <section> 
      <button onClick={handleShowTermsSummary}> 
        Show Terms of Use Summary 
      </button> 
      {paragraph} 
    </section> 
  ); 
}
```

En este ejemplo, si `showTerms` es `true`, la variable `paragraph` no almacena texto sino un elemento JSX completo (el elemento `<p>`). En el código JSX devuelto, el valor almacenado en la variable `paragraph` se muestra dinámicamente a través de `{paragraph}`. Si `showTerms` es `false`, `paragraph` almacena el valor `undefined` y no se renderiza nada en el DOM. Por lo tanto, **insertar `null` o `undefined` en el código JSX hace que React no muestre nada**. Pero si `showTerms` es `true`, el párrafo completo se guarda como un valor y se muestra en el DOM.

Así es como se pueden renderizar dinámicamente elementos JSX completos. Por supuesto, no estás limitado a elementos individuales: podrías almacenar estructuras de árbol JSX completas (como elementos JSX múltiples, anidados o hermanos) dentro de variables o constantes. Como regla simple, **cualquier cosa que pueda devolver una función de componente se puede almacenar en una variable**.

#### Diferentes formas de renderizar contenido condicionalmente
En el ejemplo mostrado anteriormente, el contenido se renderiza condicionalmente mediante el uso de una variable, que se establece con la ayuda de una sentencia `if` y luego se muestra dinámicamente en el código JSX. Esta es una técnica común (y perfectamente válida) para renderizar contenido condicionalmente, pero no es el único enfoque que puedes usar.

Alternativamente, también podrías:
1. Utilizar expresiones ternarias.
2. Utilizar los operadores lógicos de JavaScript.
3. Usar cualquier otra forma válida de JavaScript para seleccionar valores condicionalmente.

Las siguientes secciones explorarán cada enfoque en detalle.

#### Utilización de expresiones ternarias
En JavaScript (y en muchos otros lenguajes de programación), puedes utilizar **expresiones ternarias** (también conocidas como operadores ternarios condicionales) como alternativas a las sentencias `if`. Las expresiones ternarias pueden ahorrarte líneas de código, especialmente con condiciones simples donde el objetivo principal es asignar el valor de una variable de forma condicional.

Aquí hay una comparación directa, comenzando primero con una sentencia `if` regular:

```javascript
let a = 1; 
if (someCondition) { 
  a = 2; 
}
```

Aquí está la misma lógica, implementada con una expresión ternaria:

```javascript
const a = someCondition ? 2 : 1;
```

Este es código JavaScript estándar, no específico de React. Sin embargo, es importante comprender esta característica central de JavaScript para comprender cómo se puede utilizar en aplicaciones de React.

Trasladado al ejemplo anterior de React, el contenido del párrafo se podría establecer y mostrar condicionalmente con la ayuda de expresiones ternarias de esta manera:

```javascript
import { useState } from 'react'; 

function TermsOfUse() { 
  const [showTerms, setShowTerms] = useState(false); 

  function handleShowTermsSummary() { 
    setShowTerms(true); 
  } 

  const paragraph = showTerms ? <p>By continuing, you accept that we will not indemnify you for any damage or harm caused by our products.</p> : null; 

  return ( 
    <section> 
      <button onClick={handleShowTermsSummary}> 
        Show Terms of Use Summary 
      </button> 
      {paragraph} 
    </section> 
  ); 
}
```

Como puedes ver, el código general es un poco más corto que antes, cuando se usaba una sentencia `if`. La constante `paragraph` contiene el párrafo (incluido el contenido del texto) o `null`. Se utiliza `null` como valor alternativo porque `null` se puede insertar de forma segura en el código JSX, ya que simplemente hace que no se renderice nada en su lugar.

Una desventaja de las expresiones ternarias es que la legibilidad y la comprensibilidad pueden verse afectadas, especialmente cuando se utilizan expresiones ternarias anidadas, como en el siguiente ejemplo:

```javascript
const paragraph = !showTerms ? null : someOtherCondition ? <p>By continuing, you accept that we will not indemnify you for any damage or harm caused by our products.</p> : null;
```

Este código es difícil de leer y aún más difícil de entender. Por esta razón, generalmente debes evitar escribir expresiones ternarias anidadas y recurrir a sentencias `if` en tales situaciones.

Sin embargo, a pesar de estas posibles desventajas, las expresiones ternarias pueden ayudarte a escribir menos código en aplicaciones de React, especialmente cuando las utilizas en línea (*inline*), directamente dentro del código JSX:

```javascript
import { useState } from 'react'; 

function TermsOfUse() { 
  const [showTerms, setShowTerms] = useState(false); 

  function handleShowTermsSummary() { 
    setShowTerms(true); 
  } 

  return ( 
    <section> 
      <button onClick={handleShowTermsSummary}> 
        Show Terms of Use Summary 
      </button> 
      {showTerms ? <p>By continuing, you accept that we will not indemnify you for any damage or harm caused by our products.</p> : null} 
    </section> 
  ); 
}
```

Este es el mismo ejemplo que antes, solo que ahora es aún más corto ya que aquí evitas usar la constante `paragraph` utilizando la expresión ternaria directamente dentro del fragmento JSX. Esto permite un código de componente relativamente limpio, por lo que es bastante común usar expresiones ternarias en el código JSX en aplicaciones de React para aprovechar esto.

#### Uso de los operadores lógicos de JavaScript
Las expresiones ternarias son populares porque te permiten escribir menos código, lo que, cuando se usa en los lugares correctos (y evitando anidar múltiples expresiones ternarias), puede ayudar con la legibilidad general.

Especialmente en aplicaciones de React, en el código JSX a menudo escribirás expresiones ternarias como esta:

```javascript
<div> 
  {showDetails ? <h1>Product Details</h1> : null} 
</div>
```

O bien, como esta:

```javascript
<div> 
  {showTerms ? <p>Our terms of use …</p> : null} 
</div>
```

¿Qué tienen en común estos dos fragmentos?

Son innecesariamente largos porque, en ambos ejemplos, se debe especificar el caso `else` (`: null`), a pesar de que no aporta nada a la interfaz de usuario final. Después de todo, el propósito principal de estas expresiones ternarias es renderizar elementos JSX (`<h1>` y `<p>`, en los ejemplos anteriores). El caso `else` (`: null`) simplemente significa que no se renderiza nada si no se cumplen las condiciones (`showDetails` y `showTerms`).

Es por eso que un patrón diferente es muy popular entre los desarrolladores de React:

```javascript
<div> 
  {showDetails && <h1>Product Details</h1>} 
</div>
```

Esta es la forma más corta posible de lograr el resultado previsto, renderizando solo el elemento `<h1>` y su contenido si `showDetails` es `true`.

Este código utiliza un comportamiento interesante de los operadores lógicos de JavaScript, específicamente del operador `&&` (*AND* lógico). En JavaScript, **el operador `&&` devuelve el segundo valor (es decir, el valor después de `&&`) si el primer valor (es decir, el valor antes de `&&`) es verdadero o verídico (*truthy*, es decir, no es `false`, `undefined`, `null`, `0`, etc.)**. Normalmente, usarías el operador `&&` en sentencias `if` o expresiones ternarias. Sin embargo, al trabajar con React y JSX, puedes aprovechar este comportamiento para mostrar valores verídicos condicionalmente. Esta técnica también se llama **cortocircuito** (*short-circuiting*).

Por ejemplo, el siguiente código mostraría `'Hello'`:

```javascript
console.log(1 === 1 && 'Hello');
```

Este comportamiento se puede utilizar para escribir expresiones muy cortas que comprueban una condición y luego muestran otro valor, como se muestra en el ejemplo anterior.

> [!NOTE]
> Vale la pena señalar que usar `&&` puede generar resultados inesperados si lo usas con valores de condición que no sean booleanos (es decir, si el valor antes de `&&` contiene un valor no booleano). Si `showDetails` fuera `0` en lugar de `false` (por el motivo que fuera), el número `0` se mostraría en la pantalla. Por lo tanto, debes asegurarte de que el valor que actúa como condición produzca `null` o `false` en lugar de valores falsos arbitrarios. Podrías, por ejemplo, forzar una conversión a booleano agregando `!!` (por ejemplo, `!!showDetails`). Eso no es necesario si el valor de tu condición ya contiene `null` o `false`.

#### ¡Sé creativo!
En este punto, has aprendido sobre tres formas diferentes de definir y mostrar contenido condicionalmente (sentencias `if` regulares, expresiones ternarias y el operador `&&`). Sin embargo, el punto más importante es que el código de React es, en última instancia, simplemente código JavaScript estándar. Por lo tanto, cualquier enfoque que seleccione valores condicionalmente funcionará.

Si tiene sentido en tu caso de uso específico y en tu aplicación de React, también podrías tener un componente que seleccione y muestre contenido condicionalmente de esta forma:

```javascript
const languages = { 
  de: 'de-DE', 
  us: 'en-US', 
  uk: 'en-GB' 
}; 

function LanguageSelector({country}) { 
  return <p>Selected Language: {languages[country]}</p> 
}
```

Este componente muestra `'de-DE'`, `'en-US'` o `'en-GB'` según el valor de la prop `country`. Este resultado se logra mediante el uso de la sintaxis de selección dinámica de propiedades de JavaScript. En lugar de seleccionar una propiedad específica a través de la notación de punto (como `person.name`), puedes seleccionar valores de propiedades a través de la notación de corchetes. Con esa notación, puedes pasar un nombre de propiedad específico (`languages['de-DE']`) o una expresión que produzca un nombre de propiedad (`languages[country]`).

Seleccionar valores de propiedad dinámicamente de esta manera es otro patrón común para elegir valores de un mapa de valores. Por lo tanto, es una alternativa a especificar múltiples sentencias `if` o expresiones ternarias.

Además, en general, puedes utilizar cualquier enfoque que funcione en JavaScript estándar, porque React es, después de todo, solo JavaScript estándar en su núcleo.

#### ¿Qué enfoque es el mejor?
Se han discutido varias formas de configurar y mostrar contenido condicionalmente, pero ¿cuál es el mejor enfoque?

Eso realmente depende de ti (y, si corresponde, de tu equipo). Se han destacado las ventajas y desventajas más importantes, pero en última instancia, es tu decisión. Si prefieres las expresiones ternarias, no hay nada de malo en elegirlas sobre el operador lógico `&&`, por ejemplo.

También dependerá del problema exacto que estés intentando resolver. Si tienes un mapa de valores (como una lista de países y sus códigos de idioma), optar por la selección dinámica de propiedades en lugar de múltiples sentencias `if` podría ser preferible. Por otro lado, si tienes una sola condición de verdadero/falso (como `age > 18`), usar una sentencia `if` estándar o el operador lógico `&&` podría ser lo mejor.

#### Configurar etiquetas de elementos de forma condicional
Mostrar contenido condicionalmente es un escenario muy común. Pero a veces, también querrás elegir el **tipo de etiqueta HTML que se mostrará condicionalmente**. Por lo general, este será el caso cuando crees componentes cuya tarea principal sea envolver y mejorar los componentes integrados.

Aquí tienes un ejemplo:

```javascript
function Button({isButton, config, children}) { 
  if (isButton) { 
    return <button {...config}>{children}</button>; 
  } 
  return <a {...config}>{children}</a>; 
};
```

Este componente `Button` comprueba si el valor de la prop `isButton` es verídico y, si ese es el caso, devuelve un elemento `<button>`. Se espera que la prop `config` sea un objeto de JavaScript, y el operador de propagación estándar de JavaScript (`...`) se utiliza luego para agregar todos los pares clave-valor del objeto `config` como props al elemento `<button>`. Si `isButton` no es verídico (tal vez porque no se proporcionó ningún valor para `isButton`, o porque el valor es `false`), la condición `else` se activa. En lugar de un elemento `<button>`, se devuelve un elemento `<a>`.

> [!NOTE]
> Usar el operador de propagación (`...`) para traducir las propiedades de un objeto (pares clave-valor) en props de componentes es otro patrón común de React (y se introdujo en el Capítulo 3, *Componentes y Props*). El operador de propagación no es un operador específico de React, pero usarlo para este propósito especial sí lo es.
> Al propagar un objeto como `{link: 'https://some-url.com', isButton: false}` en un elemento `<a>` (a través de `<a {...obj}>`), el resultado sería el mismo que si todas las props se hubieran establecido individualmente (es decir, `<a link="https://some-url.com" isButton={false}>`).

Este patrón es particularmente popular en situaciones en las que construyes componentes envoltorios personalizados que envuelven un componente central común (por ejemplo, `<button>`, `<input>` o `<a>`) para agregar ciertos estilos o comportamientos, mientras permites que el componente se use de la misma manera que el componente integrado (es decir, puedes establecer todas las props predeterminadas).

El componente `Button` del ejemplo anterior devuelve dos elementos JSX totalmente diferentes, según el valor de la prop `isButton`. Esta es una excelente manera de verificar una condición y devolver contenido diferente (es decir, contenido condicional).

Sin embargo, al utilizar un comportamiento especial de React, este componente podría escribirse con aún menos código:

```javascript
function Button({isButton, config, children}) { 
  const Tag = isButton ? 'button' : 'a'; 
  return <Tag {...config}>{children}</Tag>; 
};
```

El comportamiento especial es que **los nombres de las etiquetas se pueden almacenar (como valores de cadena) en variables o constantes, y esas variables o constantes se pueden usar como elementos JSX en el código JSX** (siempre que el nombre de la variable o constante comience con un carácter en mayúscula, como todos tus componentes personalizados).

La constante `Tag` en el ejemplo anterior almacena la cadena `'button'` o `'a'`. Como comienza con un carácter en mayúscula (`Tag`, en lugar de `tag`), se puede usar como un componente personalizado dentro de fragmentos de código JSX. React acepta esto como un componente, aunque no sea una función de componente. Esto se debe a que se almacena el nombre de una etiqueta de elemento HTML estándar, por lo que React puede renderizar el componente integrado apropiado. El mismo patrón también se podría utilizar con componentes personalizados. En lugar de almacenar valores de cadena, almacenarías punteros a tus funciones de componentes personalizados:

```javascript
import MyComponent from './my-component.jsx'; 
import MyOtherComponent from './my-other-component.jsx'; 

const Tag = someCondition ? MyComponent : MyOtherComponent;
```

Este es otro patrón útil que puede ayudar a ahorrar código y, por lo tanto, conduce a componentes más limpios.

---

### Sección 4: Renderizado de Listas de Datos

Además de mostrar datos condicionales, a menudo trabajarás con listas de datos que deben mostrarse en una página. Como se mencionó anteriormente en este capítulo, algunos ejemplos son listas de productos, transacciones y elementos de navegación.

Normalmente, en las aplicaciones de React, dichos datos de lista se reciben como un array de valores. Por ejemplo, un componente podría recibir un array de productos a través de props (pasado al componente desde dentro de otro componente que podría estar obteniendo esos datos de alguna API de backend):

```javascript
function ProductsList({products}) { 
  // … todo! 
};
```

En este ejemplo, el array `products` podría verse así:

```javascript
const products = [ 
  {id: 'p1', title: 'A Book', price: 59.99}, 
  {id: 'p2', title: 'A Carpet', price: 129.49}, 
  {id: 'p3', title: 'Another Book', price: 39.99}, 
];
```

Sin embargo, estos datos no se pueden mostrar directamente así. En cambio, el objetivo suele ser traducirlos a una lista de elementos JSX adecuada. Por ejemplo, el resultado deseado podría ser el siguiente:

```html
<ul> 
  <li> 
    <h2>A Book</h2> 
    <p>$59.99</p> 
  </li> 
  <li> 
    <h2>A Carpet</h2> 
    <p>$129.49</p> 
  </li> 
  <li> 
    <h2>Another Book</h2> 
    <p>$39.99</p> 
  </li> 
</ul>
```

¿Cómo se puede lograr esta transformación?

Nuevamente, es una buena idea ignorar a React por un momento y encontrar una manera de transformar datos de listas con JavaScript estándar. Una forma posible de lograr esto sería usar un bucle `for…of`, como se muestra:

```javascript
const transformedProducts = []; 

for (const product of products) { 
  transformedProducts.push(product.title); 
}
```

En este ejemplo, la lista de objetos de productos (`products`) se transforma en una lista de títulos de productos (es decir, una lista de valores de cadena). Esto se logra recorriendo todos los elementos de producto en `products` y extrayendo solo la propiedad `title` de cada producto. Este valor de la propiedad `title` se inserta luego en el nuevo array `transformedProducts`.

Se puede utilizar un enfoque similar para transformar la lista de objetos en una lista de elementos JSX:

```javascript
const productElements = []; 

for (const product of products) { 
  productElements.push(( 
    <li> 
      <h2>{product.title}</h2> 
      <p>${product.price}</p> 
    </li> 
  )); 
}
```

La primera vez que ves código como este, puede parecer un poco extraño. Pero ten en cuenta que el código JSX se puede usar en cualquier lugar donde se puedan usar valores regulares de JavaScript (es decir, números, cadenas, objetos, etc.). Por lo tanto, también puedes insertar un valor JSX en un array de valores. Dado que es código JSX, también puedes mostrar contenido dinámicamente en esos elementos JSX (como `<h2>{product.title}</h2>`).

Este código es válido y es un primer paso importante para mostrar datos de listas. Pero es solo el primer paso, ya que los datos actuales se transformaron pero todavía no son devueltos por un componente.

¿Cómo se puede devolver entonces un array de elementos JSX de este tipo?

La respuesta es que **se puede devolver sin ningún truco o código especial**. JSX en realidad acepta valores de array como valores mostrados dinámicamente.

Puedes mostrar el array `productElements` de esta manera:

```javascript
return ( 
  <ul> 
    {productElements} 
  </ul> 
);
```

Al insertar un array de elementos JSX en código JSX, todos los elementos JSX dentro de ese array se muestran uno al lado del otro. Por lo tanto, los dos fragmentos siguientes producirían el mismo resultado:

```javascript
return ( 
  <div> 
    {[<p>Hi there</p>, <p>Another item</p>]} 
  </div> 
); 

return ( 
  <div> 
    <p>Hi there</p> 
    <p>Another item</p> 
  </div> 
);
```

Con esto en mente, el componente `ProductsList` podría escribirse de la siguiente manera:

```javascript
function ProductsList({products}) { 
  const productElements = []; 

  for (const product of products) { 
    productElements.push(( 
      <li> 
        <h2>{product.title}</h2> 
        <p>${product.price}</p> 
      </li> 
    )); 
  } 

  return ( 
    <ul> 
      {productElements} 
    </ul> 
  ); 
};
```

Este es un enfoque posible para mostrar datos de listas. Como se explicó anteriormente en este capítulo, se trata de utilizar funciones estándar de JavaScript y combinarlas con JSX.

Sin embargo, no es necesariamente la forma más común de mostrar datos de listas en aplicaciones de React. En la mayoría de los proyectos, encontrarás una solución diferente.

#### Mapeo de datos de listas (*Mapping List Data*)
Mostrar datos de listas con bucles `for` funciona, como puedes ver en los ejemplos anteriores. Sin embargo, al igual que con las sentencias `if` y las expresiones ternarias, puedes reemplazar los bucles `for` con una sintaxis alternativa para escribir menos código y mejorar la legibilidad de los componentes.

JavaScript ofrece un método de array integrado que se puede utilizar para transformar elementos de un array: el método **`map()`**. `map()` es un método predeterminado al que se puede llamar en cualquier array de JavaScript. Acepta una función como parámetro y ejecuta esa función para cada elemento del array. El valor devuelto por esta función debe ser el valor transformado. Luego, `map()` combina todos estos valores transformados devueltos en un nuevo array que es devuelto por `map()`.

Podrías usar `map()` de esta manera:

```javascript
const users = [ 
  {id: 'u1', name: 'Max', age: 35}, 
  {id: 'u2', name: 'Anna', age: 32} 
]; 

const userNames = users.map(user => user.name); 
// userNames = ['Max', 'Anna']
```

En este ejemplo, se utiliza `map()` para transformar el array de objetos de usuario en un array de nombres de usuario (es decir, un array de valores de cadena).

El método `map()` a menudo puede producir el mismo resultado que un bucle `for` pero con menos código.

Por lo tanto, `map()` también se puede usar para generar un array de elementos JSX y el componente `ProductsList` de antes podría reescribirse de la siguiente manera:

```javascript
function ProductsList({products}) { 
  const productElements = products.map(product => ( 
    <li> 
      <h2>{product.title}</h2> 
      <p>${product.price}</p> 
    </li> 
  )); 

  return ( 
    <ul> 
      {productElements} 
    </ul> 
  ); 
};
```

Esto ya es más corto que el ejemplo anterior del bucle `for`. Sin embargo, al igual que con las expresiones ternarias, el código se puede acortar aún más moviendo la lógica directamente al código JSX:

```javascript
function ProductsList({products}) { 
  return ( 
    <ul> 
      {products.map(product => ( 
        <li> 
          <h2>{product.title}</h2> 
          <p>${product.price}</p> 
        </li> 
      ))} 
    </ul> 
  ); 
};
```

Dependiendo de la complejidad de la transformación (es decir, la complejidad del código ejecutado dentro de la función interna que se pasa al método `map()`), por razones de legibilidad, es posible que desees considerar no utilizar este enfoque en línea (como cuando se asignan elementos de un array a una estructura JSX compleja o cuando se realizan cálculos adicionales como parte del proceso de mapeo). En última instancia, esto se reduce a la preferencia y el juicio personal.

Debido a que es muy conciso, **el uso del método `map()` (ya sea con la ayuda de una variable o constante adicional, o directamente en línea en el código JSX) es el enfoque estándar de facto para mostrar datos de listas en aplicaciones de React y JSX en general**.

#### Actualización de listas
Imagina que tienes una lista de datos mapeados a elementos JSX y que en algún momento se agrega un nuevo elemento a la lista. O considera un escenario en el que tienes una lista donde dos elementos intercambian lugares (es decir, la lista se reordena). ¿Cómo se pueden reflejar tales actualizaciones en el DOM?

La buena noticia es que React se encargará de eso por ti si la actualización se realiza de manera **con estado** (*stateful*, es decir, utilizando el concepto de estado de React, como se explica en el Capítulo 4, *Trabajando con Eventos y Estado*).

Sin embargo, hay un par de aspectos importantes a tener en cuenta al actualizar listas (con estado).

Aquí hay un ejemplo simple que no funcionaría según lo previsto:

```javascript
import { useState } from 'react'; 

function Todos() { 
  const [todos, setTodos] = useState(['Learn React', 'Recommend this book']); 

  function handleAddTodo() { 
    todos.push('A new todo'); 
  }; 

  return ( 
    <div> 
      <button onClick={handleAddTodo}>Add Todo</button> 
      <ul> 
        {todos.map(todo => <li>{todo}</li>)} 
      </ul> 
    </div> 
  ); 
};
```

Inicialmente, se mostrarían dos elementos de tareas pendientes en la pantalla (`<li>Learn React</li>` y `<li>Recommend this book</li>`). Pero una vez que se hace clic en el botón y se ejecuta `handleAddTodo`, el resultado esperado de mostrar otro elemento de tareas pendientes no se materializará.

Esto se debe a que ejecutar `todos.push('A new todo')` actualizará el array `todos`, pero React no lo notará. Recuerda que solo debes actualizar el estado a través de la función de actualización de estado devuelta por `useState()`; de lo contrario, React no reevaluará la función del componente.

Entonces, ¿qué tal este código?:

```javascript
function handleAddTodo() { 
  setTodos(todos.push('A new todo')); 
};
```

Esto también es incorrecto porque la función de actualización de estado (`setTodos`, en este caso) debe recibir el nuevo estado (es decir, el estado que debe establecerse) como argumento. Sin embargo, el método `push()` no devuelve el array actualizado; en su lugar, muta el array existente en el lugar. Incluso si `push()` devolviera el array actualizado, seguiría siendo incorrecto utilizar el código anterior, porque los datos se cambiarían (mutarían) detrás de escena antes de que se ejecutara la función de actualización de estado. Dado que los arrays son objetos y, por lo tanto, tipos de datos por referencia, técnicamente los datos se cambiarían antes de informar a React sobre ese cambio. Siguiendo las mejores prácticas de React, esto debe evitarse.

Por lo tanto, al actualizar un array (o, como nota al margen, un objeto en general), debes realizar esta actualización de manera **inmutable** (es decir, sin cambiar el array u objeto original). En su lugar, se debe crear un nuevo array u objeto. Este nuevo array puede basarse en el array antiguo y contener todos los datos antiguos, así como cualquier dato nuevo o actualizado.

Por lo tanto, el array `todos` debe actualizarse de esta manera:

```javascript
function handleAddTodo() { 
  setTodos(curTodos => [...curTodos, 'A new todo']); 
  // alternativa: Usar concat() en lugar del operador de propagación:
  // concat(), a diferencia de push(), devuelve un nuevo array 
  // setTodos(curTodos => curTodos.concat('A new todo')); 
};
```

Al usar `concat()` o un nuevo array, combinado con el operador de propagación, se proporciona un array completamente nuevo a la función de actualización de estado. Ten en cuenta también que se pasa una función a la función de actualización de estado ya que el nuevo estado depende del estado anterior.

Al actualizar un valor de estado de array (o cualquier objeto) de esta manera, React puede captar esos cambios. Por lo tanto, React reevaluará la función del componente y aplicará los cambios necesarios al DOM.

> [!NOTE]
> La **inmutabilidad** no es un concepto exclusivo de React, pero sigue siendo clave en las aplicaciones de React. Al trabajar con estados y valores de referencia (es decir, objetos y arrays), la inmutabilidad es sumamente importante para garantizar que React pueda captar los cambios y que no se realicen cambios de estado "invisibles" (es decir, no reconocidos por React).
> Hay diferentes formas de actualizar objetos y arrays de forma inmutable, pero un enfoque popular es crear nuevos objetos o arrays y luego usar el operador de propagación (`...`) para fusionar los datos existentes en esos nuevos arrays u objetos.

---

### Sección 5: Un problema con los elementos de la lista

Si estás siguiendo el proceso con tu propio código y muestras datos de lista como se describe en las secciones anteriores, es posible que hayas notado que React genera una advertencia en la consola de las herramientas de desarrollo del navegador:

**Figura 5.4**: React a veces genera una advertencia sobre la falta de claves únicas (*keys*).

React se queja de la falta de **claves (*keys*)**.

Para comprender esta advertencia y la idea detrás de las claves, es útil explorar un caso de uso específico y un problema potencial con ese escenario. Supón que tienes un componente de React que es responsable de mostrar una lista de elementos, tal vez una lista de tareas pendientes. Además, asume que esos elementos de la lista se pueden reordenar y que la lista se puede editar de otras formas (por ejemplo, se pueden agregar nuevos elementos, se pueden actualizar o eliminar elementos existentes, etc.). En otras palabras, la lista no es estática.

Considera este ejemplo de interfaz de usuario, en el que se agrega un nuevo elemento a una lista de tareas pendientes:

**Figura 5.5**: Una lista se actualiza insertando un nuevo elemento en la parte superior.

En la figura anterior, puedes ver la lista renderizada inicialmente (1), que luego se actualiza después de que un usuario ingresa y envía un nuevo valor de tarea pendiente (2). Se agrega un nuevo elemento de tarea pendiente en la parte superior de la lista (es decir, como primer elemento de la lista) (3).

> [!NOTE]
> El código fuente de ejemplo para esta aplicación de demostración se puede encontrar en: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/examples/02-keys](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/examples/02-keys).

Si trabajas en esta aplicación y abres las herramientas de desarrollo del navegador (y luego la consola de JavaScript), verás la advertencia de "claves faltantes" que se mencionó anteriormente. Esta aplicación también ayuda a comprender de dónde proviene esta advertencia.

En Chrome DevTools, navega a la pestaña *Elements* y selecciona uno de los elementos de tareas pendientes o la lista de tareas pendientes vacía (es decir, el elemento `<ul>`). Una vez que agregas un nuevo elemento de tareas pendientes, cualquier elemento del DOM que se haya insertado o actualizado se resalta en Chrome en la pestaña *Elements* (parpadeando brevemente):

**Figura 5.6**: Los elementos actualizados del DOM se resaltan en Chrome DevTools.

La parte interesante es que no solo parpadea el elemento de tareas pendientes recién agregado (es decir, el elemento `<li>` recién insertado). En su lugar, **todos los elementos `<li>` existentes, que reflejan elementos de tareas pendientes existentes que no se modificaron, son resaltados por Chrome**. Esto implica que todos estos otros elementos `<li>` también se actualizaron en el DOM, a pesar de que no había necesidad de esa actualización. Los elementos ya existían antes y su contenido (el texto de la tarea) no cambió.

Por alguna razón, React parece destruir los nodos del DOM existentes (es decir, los elementos `<li>` existentes), solo para recrearlos inmediatamente después. Esto sucede con cada nuevo elemento de tareas pendientes que se agrega a la lista. Como puedes imaginar, esto no es muy eficiente y puede causar problemas de rendimiento en aplicaciones más complejas que podrían estar renderizando docenas o cientos de elementos en múltiples listas.

Esto sucede porque React no tiene forma de saber que solo se debe insertar un nodo en el DOM. No puede determinar que todos los demás nodos del DOM deben permanecer intactos porque React solo recibió un valor de estado completamente nuevo: un nuevo array lleno de nuevos objetos de JavaScript. Incluso si el contenido de esos objetos no cambió, técnicamente siguen siendo objetos nuevos (nuevos valores en la memoria).

Como desarrollador, sabes cómo funciona tu aplicación y que el contenido del array de tareas no cambió mucho en realidad. Pero React no lo sabe. Por lo tanto, React determina que todos los elementos de la lista existentes (elementos `<li>`) deben descartarse y reemplazarse por elementos nuevos que reflejen los nuevos datos proporcionados como parte de la actualización del estado. Por eso, todos los nodos del DOM relacionados con la lista se actualizan (es decir, se destruyen y se recrean) en cada actualización del estado.

#### ¡Las Keys al rescate!
El problema descrito anteriormente es extremadamente común. La mayoría de las actualizaciones de listas son actualizaciones incrementales, no cambios masivos. Pero React no puede saber si ese es el caso para tu uso específico y tu lista.

Por eso React utiliza el concepto de **keys** al trabajar con datos de listas y renderizar elementos de listas. Las *keys* son simplemente **valores de identificación únicos** que se pueden (y se deben) adjuntar a los elementos JSX al renderizar datos de listas. Las claves ayudan a React a identificar elementos que se renderizaron previamente y no cambiaron. Al permitir la identificación única de todos los elementos de la lista, las claves también ayudan a React a mover los elementos del DOM (elementos de la lista) de manera eficiente.

Las claves se agregan a los elementos JSX a través de la prop especial integrada **`key`** que es aceptada por cada componente:

```javascript
<li key={todo.id}>{todo.text}</li>
```

Esta prop especial se puede agregar a todos los componentes, ya sean integrados o personalizados. No necesitas aceptar ni manejar la prop `key` de ninguna manera en tus componentes personalizados; React lo hará por ti automáticamente.

La prop `key` requiere un valor que sea único para cada elemento de la lista. **Ningún par de elementos de la lista debe tener la misma clave**. Además, las buenas claves están directamente adjuntas a los datos subyacentes que componen el elemento de la lista. Por lo tanto, los índices de los elementos de la lista son malas claves porque el índice no está adjunto a los datos del elemento de la lista: si reordenas los elementos en una lista, los índices siguen siendo los mismos (un array siempre comienza con el índice 0, seguido de 1, etc.), pero los datos han cambiado de posición.

Considera el siguiente ejemplo:

```javascript
const hobbies = ['Sports', 'Cooking']; 
const reversed = hobbies.reverse(); // ['Cooking', 'Sports']
```

En este ejemplo, `'Sports'` tiene el índice `0` en el array `hobbies`. En el array invertido, su índice sería `1` (porque ahora es el segundo elemento). En este caso, si se usara el índice como clave, los datos no estarían vinculados a él.

Las buenas claves son valores de ID únicos, de modo que cada ID pertenece exactamente a un valor. Si ese valor se mueve o se elimina, su ID debe moverse o desaparecer con ese valor.

Encontrar buenos valores de ID no suele ser un gran problema, ya que la mayoría de los datos de listas se obtienen de bases de datos de todos modos. No importa si se trata de productos, pedidos, usuarios o artículos del carrito de compras: todos son datos que normalmente se almacenarían en una base de datos. Este tipo de datos ya tiene valores de ID únicos, ya que siempre se cuenta con algún tipo de criterio de identificación único al almacenar datos en bases de datos.

A veces, incluso los valores mismos se pueden usar como claves. Considera el siguiente ejemplo:

```javascript
const hobbies = ['Sports', 'Cooking'];
```

Los pasatiempos son valores de cadena y no hay ningún valor de ID único asociado a los pasatiempos individuales. Cada pasatiempo es un valor primitivo (una cadena). Sin embargo, en casos como este, normalmente no tendrás valores duplicados, ya que no tiene sentido que un pasatiempo se incluya más de una vez en un array como este. Por lo tanto, los valores mismos califican como buenas claves:

```javascript
hobbies.map(hobby => <li key={hobby}>{hobby}</li>);
```

En los casos en que no puedas usar los valores en sí y no haya otro valor de clave posible, puedes generar valores de ID únicos directamente en el código de tu aplicación de React. Como último recurso, también puedes recurrir al uso de índices; pero ten en cuenta que esto puede provocar errores inesperados y efectos secundarios si reordenas los elementos de la lista.

Con las claves agregadas a los elementos de la lista, React puede identificar todos los elementos correctamente. Cuando el estado del componente cambia, puede identificar los elementos JSX que ya se renderizaron antes. Por lo tanto, esos elementos ya no se destruyen ni se recrean.

Puedes confirmar esto abriendo nuevamente las herramientas de desarrollo del navegador para verificar qué elementos del DOM se actualizan tras los cambios en los datos de la lista subyacente:

**Figura 5.7**: De varios elementos de la lista, solo un elemento del DOM se actualiza.

Después de agregar claves, al actualizar el estado de la lista, solo el nuevo elemento del DOM se resalta en Chrome DevTools. Los otros elementos son (correctamente) ignorados por React.

---

### Sección 6: Resumen y puntos clave

- Al igual que cualquier otro valor de JavaScript, los elementos JSX se pueden configurar y cambiar dinámicamente, según diferentes condiciones.
- El contenido se puede definir condicionalmente mediante sentencias `if`, expresiones ternarias, el operador lógico AND (`&&`) o de cualquier otra forma que funcione en JavaScript.
- Hay varias formas de manejar el contenido condicional: cualquier enfoque que funcione en JavaScript puro también se puede usar en aplicaciones de React.
- Los arrays con elementos JSX se pueden insertar en el código JSX y harán que los elementos del array se muestren como elementos hermanos del DOM.
- Los datos de listas se pueden convertir en arrays de elementos JSX mediante bucles `for`, el método `map()` o cualquier otro enfoque de JavaScript que conduzca a una conversión similar.
- Usar el método `map()` es la forma más común de convertir datos de listas en listas de elementos JSX.
- Se deben agregar claves (a través de la prop `key`) a los elementos JSX de la lista para ayudar a React a actualizar el DOM de manera eficiente.

---

### Sección 7: ¿Qué sigue?

Con el contenido condicional y las listas, ahora tienes todas las herramientas clave necesarias para construir interfaces de usuario tanto simples como más complejas con React. Puedes ocultar y mostrar elementos o grupos de elementos según sea necesario, y puedes renderizar y actualizar dinámicamente listas de elementos para mostrar listas de productos, pedidos o usuarios.

Por supuesto, eso no es todo lo necesario para construir interfaces de usuario realistas. Agregar lógica para cambiar el contenido dinámicamente es una cosa, pero la mayoría de las aplicaciones web también necesitan estilos CSS que deben aplicarse a varios elementos del DOM. Este libro no trata sobre CSS, pero el próximo capítulo explorará cómo se pueden diseñar y estilizar las aplicaciones de React. Especialmente cuando se trata de configurar y cambiar estilos dinámicamente o delimitar estilos a componentes específicos, existen varios conceptos específicos de React que deberían resultarle familiares a todo desarrollador de React.

---

### Sección 8: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/05-lists-conditional-code/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/05-lists-conditional-code/exercises/questions-answers.md):

1. ¿Qué es el "contenido condicional"?
2. Nombra al menos dos formas diferentes de renderizar elementos JSX condicionalmente.
3. ¿Qué enfoque elegante se puede utilizar para definir etiquetas de elementos de forma condicional?
4. ¿Cuál es una posible desventaja de utilizar únicamente expresiones ternarias (para contenido condicional)?
5. ¿Cómo se pueden renderizar listas de datos como elementos JSX?
6. ¿Por qué se deben agregar claves (*keys*) a los elementos de la lista renderizados?
7. Da un ejemplo de una clave buena y una mala.

---

### Sección 9: Aplica lo aprendido

Ahora puedes utilizar tus conocimientos de React para modificar interfaces de usuario dinámicas de diversas formas. Además de poder cambiar los valores de texto y los números que se muestran, ahora también puedes ocultar o mostrar elementos completos (o bloques de elementos) y mostrar listas de datos.

En las siguientes secciones, encontrarás dos actividades que te permitirán aplicar tus conocimientos recién adquiridos (combinados con los conocimientos adquiridos en los otros capítulos del libro).

#### Actividad 5.1: Mostrar un mensaje de error condicional
En esta actividad, construirás un formulario básico que permite a los usuarios ingresar su dirección de correo electrónico. Tras el envío del formulario, la entrada del usuario debe validarse y las direcciones de correo electrónico no válidas (para simplificar, aquí nos referimos a las direcciones de correo electrónico que no contienen ningún signo `@`) deberían hacer que se muestre un mensaje de error debajo del formulario. Cuando las direcciones de correo electrónico no válidas se vuelven válidas, los mensajes de error potencialmente visibles deben eliminarse nuevamente.

Realiza los siguientes pasos para completar esta actividad:
1. Construye una interfaz de usuario que contenga un formulario con una etiqueta, un campo de entrada (de tipo texto, para facilitar la introducción de direcciones de correo electrónico incorrectas para fines de demostración) y un botón de envío que haga que se envíe el formulario.
2. Recopila la dirección de correo electrónico ingresada y muestra un mensaje de error debajo del formulario si la dirección de correo electrónico no contiene ningún signo `@`.

La interfaz de usuario final debería verse y funcionar como se muestra aquí:

**Figura 5.8**: La interfaz de usuario final de esta actividad.

> [!NOTE]
> El estilo, por supuesto, variará. Para obtener el mismo estilo que se muestra en la captura de pantalla, usa mi proyecto inicial preparado, que puedes encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-1-start).
> Analiza el archivo `index.css` en ese proyecto para determinar cómo estructurar tu código JSX para aplicar los estilos.
> Puedes encontrar la solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-1).

#### Actividad 5.2: Mostrar una lista de productos
En esta actividad, construirás una interfaz de usuario donde se mostrará una lista de productos (simulados) en la pantalla. La interfaz también debe contener un botón que, al hacer clic en él, agregue otro elemento nuevo (simulado) a la lista existente de productos.

Realiza los siguientes pasos para completar esta actividad:
1. Agrega una lista de objetos de productos simulados (cada objeto debe tener un ID, título y precio) a un componente de React y agrega código para mostrar estos elementos de productos como elementos JSX.
2. Agrega un botón a la interfaz de usuario. Al hacer clic en él, el botón debe agregar un nuevo objeto de producto a la lista de datos de productos. Esto debería hacer que la interfaz de usuario se actualice y muestre una lista actualizada de elementos de productos.

La interfaz de usuario final debería verse y funcionar como se muestra aquí:

**Figura 5.9**: La interfaz de usuario final de esta actividad.

> [!NOTE]
> El estilo, por supuesto, variará. Para obtener el mismo estilo que se muestra en la captura de pantalla, usa mi proyecto inicial preparado, que puedes encontrar aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-2-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-2-start).
> Analiza el archivo `index.css` en ese proyecto para determinar cómo estructurar tu código JSX para aplicar los estilos.
> Puedes encontrar la solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-2](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/05-lists-conditional-code/activities/practice-2).
