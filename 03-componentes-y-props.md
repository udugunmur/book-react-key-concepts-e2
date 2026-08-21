# Parte 1: Fundamentos de React

## Capítulo 3: Componentes y Props

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Construir componentes de React reutilizables.
- Utilizar un concepto llamado **props** para hacer que los componentes sean configurables.
- Construir interfaces de usuario flexibles combinando componentes con props.

---

### Sección 1: Introducción

En el capítulo anterior, aprendiste sobre el bloque de construcción clave de cualquier interfaz de usuario basada en React: los **componentes**. Aprendiste por qué son importantes los componentes, cómo se utilizan y cómo puedes construir componentes tú mismo.

También aprendiste sobre **JSX**, que es el marcado similar a HTML que típicamente devuelven las funciones de los componentes. Es este marcado el que define lo que se debe renderizar en la página web final (en otras palabras, qué marcado HTML debe terminar en la página web final que se sirve a los visitantes).

---

### Sección 2: ¿Pueden los componentes hacer más?

Sin embargo, hasta ahora, esos componentes no han sido demasiado útiles. Aunque podías usarlos para dividir el contenido de tu página web en bloques de construcción más pequeños, la reutilización real de estos componentes era bastante limitada. Por ejemplo, cada objetivo de curso que pudieras tener como parte de una lista general de objetivos de curso iría en su propio componente (si decidías dividir el contenido de tu página web en múltiples componentes en primer lugar).

Si lo piensas, esto no es demasiado útil; sería mucho mejor si los diferentes elementos de la lista pudieran compartir un componente común y simplemente configuraras ese único componente con diferente contenido o atributos, exactamente como funciona HTML.

Al escribir código HTML simple y describir contenido con él, utilizas elementos HTML reutilizables y los configuras con diferente contenido o atributos. Por ejemplo, tienes un único elemento HTML `<a>`, pero gracias al atributo `href` y al contenido secundario del elemento, puedes crear una cantidad infinita de elementos de enlace diferentes que apuntan a diferentes recursos, como se muestra en el siguiente fragmento:

```html
<a href="https://google.com">Use Google</a> 
<a href="https://academind.com">Browse Free Tutorials</a>
```

Estos dos elementos utilizan exactamente el mismo elemento HTML (`<a>`), pero dan como resultado enlaces totalmente diferentes que terminarían en la página web (apuntando a dos sitios web totalmente distintos).

Para desbloquear completamente el potencial de los componentes de React, sería muy útil si pudieras configurarlos como elementos HTML normales. Y resulta que puedes hacer exactamente eso con otro concepto clave de React llamado **props**.

---

### Sección 3: Uso de Props en componentes

¿Cómo utilizas las props en tus componentes? ¿Y cuándo las necesitas?

La segunda pregunta se responderá con mayor detalle un poco más adelante. Por el momento, basta con saber que normalmente tendrás algunos componentes que son reutilizables y, por lo tanto, necesitan props, y algunos componentes que son únicos y podrían no necesitar props.

La parte del "cómo" es la más importante en este punto, y esta parte se puede dividir en dos problemas complementarios:
1. Pasar props a los componentes.
2. Consumir props en un componente.

#### Pasar props a los componentes
¿Cómo querrías que funcionaran las props y la configurabilidad de los componentes si diseñaras React desde cero?

Por supuesto, habría una amplia variedad de soluciones posibles, pero hay un gran modelo a seguir que se puede considerar: HTML. Como se mencionó anteriormente, al trabajar con HTML, pasas contenido y configuración entre las etiquetas del elemento o mediante atributos.

Afortunadamente, los componentes de React funcionan exactamente como los elementos HTML cuando se trata de configurarlos. Las props simplemente se pasan como **atributos** (a tu componente) o como **datos secundarios** (*children*) entre las etiquetas del componente, y también puedes mezclar ambos enfoques:

```javascript
<Product id="abc1" price="12.99" />
<FancyLink target="https://some-website.com">Click me</FancyLink>
```

Por esta razón, configurar componentes es bastante sencillo, al menos si los miras desde la perspectiva del consumidor (en otras palabras, cómo los usas en JSX).

#### Consumir props en un componente
¿Cómo puedes obtener acceso a los valores de las props pasados a un componente al escribir el código interno de ese componente?

Imagina que estás construyendo un componente `GoalItem` que es responsable de mostrar un único elemento de objetivo (por ejemplo, un objetivo de curso o de proyecto) que formará parte de una lista general de objetivos.

El marcado JSX del componente padre podría verse así:

```javascript
<ul> 
  <GoalItem /> 
  <GoalItem /> 
  <GoalItem /> 
</ul>
```

Dentro de `GoalItem`, el objetivo sería aceptar diferentes títulos de objetivos para que el mismo componente (`GoalItem`) pueda usarse para mostrar estos diferentes títulos como parte de la lista final que se muestra a los visitantes del sitio web. Quizás el componente también debería aceptar otro dato (por ejemplo, un ID único que se utiliza internamente).

Así es como se podría utilizar el componente `GoalItem` en JSX, como se muestra en el siguiente ejemplo:

```javascript
<ul> 
  <GoalItem id="g1" title="Finish the book!" /> 
  <GoalItem id="g2" title="Learn all about React!" /> 
</ul>
```

Dentro de la función del componente `GoalItem`, el plan probablemente sería mostrar contenido dinámico (en otras palabras, los datos recibidos a través de props) de esta manera:

```javascript
function GoalItem() { 
  return <li>{title} (ID: {id})</li>; 
}
```

Pero esta función de componente no funcionaría. Tiene un problema: `title` e `id` nunca se definen dentro de esa función de componente. Por lo tanto, este código provocaría un error porque estás usando una variable que no fue definida.

Por supuesto, estos no deberían definirse dentro del componente `GoalItem` de todos modos, ya que la idea era hacer que el componente `GoalItem` fuera reutilizable y recibiera diferentes valores de `title` e `id` desde fuera del componente (es decir, desde el componente que renderiza la lista de componentes `<GoalItem>`).

React proporciona una solución para este problema: un valor de parámetro especial que React pasa automáticamente a cada función de componente. Este es un parámetro especial que contiene los datos de configuración adicionales que se establecen en el componente en el código JSX, llamado el **parámetro props**.

La función de componente anterior podría (y debería) reescribirse de la siguiente manera:

```javascript
function GoalItem(props) { 
  return <li>{props.title} (ID: {props.id})</li>; 
}
```

El nombre del parámetro (`props`) depende de ti, pero usar `props` como nombre es una convención porque el concepto general se llama props.

Para comprender este concepto, es importante tener en cuenta que tú no llamas a estas funciones de componentes en ningún otro lugar de tu código y que, en su lugar, React llamará a estas funciones en tu nombre. Y dado que React llama a estas funciones, puede pasarles argumentos adicionales al ejecutarlas.

Este argumento `props` es, en efecto, un argumento adicional de este tipo. React lo pasará a cada función de componente, independientemente de si lo definiste como un parámetro explícito en la definición de la función del componente. Sin embargo, si no definiste ese parámetro `props` en la función de un componente, por supuesto, no podrás trabajar con los datos de las props en ese componente.

Este argumento `props` proporcionado automáticamente siempre contendrá un **objeto** (porque React pasa un objeto como valor para este argumento), y las propiedades de este objeto serán los "atributos" que agregaste a tu componente (como `title` o `id`) dentro del código JSX donde se utiliza el componente.

Es por eso que en este ejemplo del componente `GoalItem`, los datos personalizados se pueden pasar a través de atributos (`<GoalItem id="g1" … />`) y consumirse a través del objeto `props` y sus propiedades (`<li>{props.title}</li>`).

---

### Sección 4: Componentes, Props y Reutilización

Gracias a este concepto de props, los componentes se vuelven verdaderamente reutilizables, en lugar de ser solo teóricamente reutilizables.

Renderizar tres componentes `<GoalItem>` sin ninguna configuración adicional solo podría renderizar el mismo objetivo tres veces, ya que el texto del objetivo (y cualquier otro dato que puedas necesitar) tendría que estar codificado de forma fija (*hardcoded*) en la función del componente.

Al utilizar props como se describió anteriormente, el mismo componente se puede utilizar varias veces con diferentes configuraciones. Eso te permite definir una estructura de marcado general y una lógica una sola vez (en la función del componente), pero luego usarla tantas veces como sea necesario con diferentes configuraciones.

Y si eso suena familiar, esa es exactamente la misma idea que se aplica a las funciones regulares de JavaScript (o cualquier otro lenguaje de programación). Defines la lógica una vez y luego puedes llamarla varias veces con diferentes entradas para recibir diferentes resultados. Es lo mismo para los componentes, al menos cuando se adopta este concepto de props.

#### La propiedad especial “children”
Se mencionó anteriormente que React pasa este objeto `props` automáticamente a las funciones de los componentes. Ese es efectivamente el caso y, como se describió, este objeto contiene todos los atributos que estableces en el componente (en JSX) como propiedades.

Pero React no solo empaqueta tus atributos en este objeto; también agrega otra propiedad adicional al objeto `props`: la **propiedad especial `children`** (una propiedad integrada cuyo nombre es fijo, lo que significa que no puedes cambiarlo).

La propiedad `children` contiene un dato muy importante: el contenido que hayas proporcionado entre las etiquetas de apertura y cierre del componente.

Hasta ahora, en los ejemplos mostrados anteriormente, los componentes eran en su mayoría autocerrados. `<GoalItem id="…" title="…" />` no contiene contenido entre las etiquetas del componente. Todos los datos se pasan al componente a través de atributos.

No hay nada de malo en este enfoque. Puedes configurar tus componentes solo con atributos. Pero para algunos datos y algunos componentes, podría tener más sentido y ser más lógico apegarse a las convenciones regulares de HTML, pasando esos datos entre las etiquetas del componente. Y el componente `GoalItem` es en realidad un gran ejemplo.

¿Qué enfoque parece más intuitivo?

```javascript
<GoalItem id="g1" title="Learn React" />
<GoalItem id="g1">Learn React</GoalItem>
```

Podrías determinar que la segunda opción parece un poco más intuitiva y acorde con el HTML estándar porque allí también configurarías un elemento de lista normal de esta manera: `<li id="li1">Some list item</li>`.

Si bien no tienes otra opción al trabajar con elementos HTML regulares (no puedes agregar un atributo `goal` a un `<li>` solo porque quieras), sí tienes una opción al trabajar con React y tus propios componentes. Simplemente depende de cómo consumas las props dentro de la función del componente. Ambos enfoques pueden funcionar, según el código interno del componente.

Aun así, es posible que desees pasar ciertos datos entre las etiquetas de los componentes, y la propiedad especial `children` te permite hacer precisamente eso. Contiene cualquier contenido que definas entre las etiquetas de apertura y cierre del componente. Por lo tanto, en el caso del segundo ejemplo, `children` contendría la cadena `"Learn React"`.

En tu función de componente, puedes trabajar con el valor `children` tal como trabajas con cualquier otro valor de prop:

```javascript
function GoalItem(props) { 
  return <li>{props.children} (ID: {props.id})</li>; 
}
```

#### ¿Qué componentes necesitan Props?
Se mencionó antes, pero es extremadamente importante: **¡las props son opcionales!**

React siempre pasará datos de props a tus componentes, pero no tienes la obligación de trabajar con ese parámetro. Ni siquiera tienes que definirlo en tu función de componente si no planeas trabajar con él.

No existe una regla estricta que defina qué componentes necesitan props y cuáles no. Esto se aprende con la experiencia y simplemente depende del rol de un componente.

Podrías tener un componente `Header` general que muestra un encabezado estático (con un logotipo, título, etc.), y dicho componente probablemente no necesite configuración externa (en otras palabras, ningún "atributo" u otro tipo de datos pasados a él). Podría ser autosuficiente, con todos los valores requeridos codificados directamente en el componente.

Pero también construirás y usarás a menudo componentes como el componente `GoalItem` (es decir, componentes que sí necesitan datos externos para ser útiles). Cada vez que un componente se usa más de una vez en tu aplicación React, hay una alta probabilidad de que utilice props. Sin embargo, lo contrario no es necesariamente cierto: aunque tendrás componentes de un solo uso que no usan props, también tendrás componentes que solo se usan una vez en toda la aplicación y aun así aprovechan las props. Como se mencionó anteriormente, depende del caso de uso exacto y del componente.

A lo largo de este libro, verás muchos ejemplos y ejercicios que te ayudarán a obtener una comprensión más profunda de cómo construir componentes y usar props.

#### Cómo manejar múltiples Props
Como se muestra en los ejemplos anteriores, no estás limitado a una sola prop por componente. De hecho, puedes pasar y usar tantas props como tu componente necesite, sin importar si son 1 o 100 (o más) props.

Una vez que creas componentes con más de dos o tres props, puede surgir una nueva pregunta: ¿tienes que agregar todas esas props individualmente (en otras palabras, como atributos separados), o puedes pasar menos atributos que contengan datos agrupados, como matrices (*arrays*) u objetos?

Y efectivamente, puedes hacerlo. React te permite pasar arrays y objetos como valores de props también. De hecho, **¡cualquier valor válido de JavaScript se puede pasar como un valor de prop!**

Esto te permite decidir si deseas tener un componente con 20 props individuales ("atributos") o solo una prop "grande". Aquí tienes un ejemplo de cómo el mismo componente se configura de dos maneras diferentes:

```javascript
<Product title="A book" price={29.99} id="p1" /> 

// o bien:
const productData = {title: 'A book', price: 29.99, id: 'p1'}; 
<Product data={productData} />
```

Por supuesto, el componente también debe adaptarse internamente (en otras palabras, en la función del componente) para esperar props individuales o agrupadas. Pero como tú eres el desarrollador, esa es, por supuesto, tu elección.

Dentro de la función del componente, también puedes hacerte la vida más fácil.

No hay nada de malo en acceder a los valores de las props a través de `props.XYZ`, pero si tienes un componente que recibe múltiples props, repetir `props.XYZ` una y otra vez puede volverse engorroso y hacer que el código sea un poco más difícil de leer.

Puedes utilizar una característica predeterminada de JavaScript para mejorar la legibilidad: la **desestructuración de objetos** (*object destructuring*).

La desestructuración de objetos te permite extraer valores de un objeto y asignarlos a variables o constantes en un solo paso:

```javascript
const user = {name: 'Max', age: 29}; 
const {name, age} = user; // <-- desestructuración de objetos en acción 
console.log(name); // muestra 'Max'
```

Por lo tanto, puedes usar esta sintaxis para extraer todos los valores de las props y asignarlos a variables con el mismo nombre directamente al inicio de tu función de componente:

```javascript
function Product({title, price, id}) { 
  // desestructuración en acción 
  // … 
  // title, price, id ahora están disponibles como variables dentro de esta función 
}
```

No tienes la obligación de utilizar esta sintaxis, pero puede hacerte la vida más fácil.

> [!NOTE]
> Para obtener más información sobre la desestructuración de objetos, MDN es un excelente lugar para profundizar. Puedes acceder a esto en: [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).

#### Propagación de Props (*Spreading Props*)
Imagina que estás construyendo un componente personalizado que debería actuar como un "envoltorio" (*wrapper*) alrededor de algún otro componente, tal vez un componente integrado.

Por ejemplo, podrías estar construyendo un componente `Link` personalizado que debería devolver un elemento estándar `<a>` con algún estilo o lógica personalizada añadida:

```javascript
function Link({children}) { 
  return <a target="_blank" rel="noopener noreferrer">{children}</a>; 
};
```

Este componente de ejemplo muy simple devuelve un elemento `<a>` preconfigurado. Este componente `Link` personalizado configura el elemento de anclaje para que las páginas nuevas siempre se abran en una nueva pestaña. En lugar del elemento estándar `<a>`, podrías usar este componente `Link` en tu aplicación React para obtener ese comportamiento directamente para todos tus enlaces.

Pero este componente personalizado tiene un problema: es un envoltorio alrededor de un elemento central, pero al crear tu propio componente, eliminas la configurabilidad de ese elemento central. Si fueras a usar este componente `Link` en tu aplicación, ¿cómo establecerías la prop `href` para configurar el destino del enlace?

Podrías intentar lo siguiente:

```javascript
<Link href="https://some-site.com">Click here</Link>
```

Sin embargo, este código de ejemplo no funcionaría porque `Link` no acepta ni utiliza una prop `href`.

Por supuesto, podrías ajustar la función del componente `Link` para que se use una prop `href`:

```javascript
function Link({children, href}) { 
  return <a href={href} target="_blank" rel="noopener noreferrer">{children}</a>; 
};
```

¿Pero qué pasa si también deseas asegurarte de que se pueda agregar la prop `download` si es necesario?

Es cierto que siempre puedes aceptar más y más props (y pasarlas al elemento `<a>` dentro de tu componente), pero esto reduce la reutilización y mantenibilidad de tu componente personalizado.

Una mejor solución es utilizar el **operador de propagación** estándar de JavaScript (es decir, el operador `...` o *spread operator*) y el soporte que React ofrece para ese operador cuando se trabaja con componentes.

Por ejemplo, el siguiente código de componente es válido:

```javascript
function Link({children, config}) { 
  return <a {...config} target="_blank" rel="noopener noreferrer">{children}</a>; 
};
```

En este ejemplo, se espera que `config` sea un objeto de JavaScript (es decir, una colección de pares clave-valor). El operador de propagación (`...`), cuando se usa en código JSX sobre un elemento JSX, convierte ese objeto en múltiples props.

Considera este valor de configuración de ejemplo:

```javascript
const config = { href: 'https://some-site.com', download: true };
```

En este caso, al propagarlo en `<a>` (es decir, `<a {...config}>`), el resultado sería el mismo que si hubieras escrito este código:

```html
<a href="https://some-site.com" download={true}>
```

Un patrón alternativo y más común utiliza otra característica de JavaScript: la **propiedad rest** (*rest property*). Ese es un patrón de JavaScript que te permite agrupar propiedades que no han sido desestructuradas en un nuevo objeto (que luego solo contiene esas propiedades restantes).

```javascript
function Link({children, ...props}) { 
  return <a {...props} target="_blank" rel="noopener noreferrer">{children}</a>; 
};
```

En este ejemplo, al desestructurar las props, solo se desestructura la prop `children`; las demás se almacenan en un nuevo objeto llamado `props`. La sintaxis es muy similar a la sintaxis del operador de propagación: usas tres puntos (`...`). Pero aquí, usas el operador delante de la propiedad que debe contener todas las propiedades restantes. Por lo tanto, es el lugar donde usas ese operador lo que define lo que hace.

Luego puedes usar esa propiedad rest (`props` en el ejemplo) como cualquier otro objeto. En el ejemplo anterior, se usa nuevamente para propagar sus propiedades como props en el elemento `<a>`.

El uso de este patrón te permite utilizar el componente `Link` de una manera más natural, donde no tienes que crear y usar un objeto de configuración separado:

```javascript
<Link href="https://google.com">Can you google that for me?</Link>
```

Estos comportamientos y patrones se pueden usar para construir componentes reutilizables que aún deben mantener la configurabilidad del elemento principal que puedan estar envolviendo. Esto te ayuda a evitar largas listas de props predefinidas y aceptadas, y mejora la reutilización de los componentes.

#### Cadenas de Props / Prop Drilling (*Prop Chains / Prop Drilling*)
Hay un último fenómeno que vale la pena señalar al aprender sobre props: **prop drilling** o **cadenas de props** (*prop chains*).

Es un problema con el que todo desarrollador de React se topará en algún momento. Ocurre cuando construyes una aplicación de React un poco más compleja que contiene múltiples capas de componentes anidados que necesitan enviarse datos entre sí.

Por ejemplo, asume que tienes un componente `NavItem` que debería mostrar un enlace de navegación. Dentro de ese componente, podrías tener otro componente anidado, `AnimatedLink`, que muestra el enlace real (tal vez con algunos estilos de animación agradables).

El componente `NavItem` podría verse así:

```javascript
function NavItem(props) { 
  return <div><AnimatedLink target={props.target} text="Some text" /></div>; 
}
```

Y `AnimatedLink` podría definirse así:

```javascript
function AnimatedLink(props) { 
  return <a href={props.target}>{props.text}</a>; 
}
```

En este ejemplo, la prop `target` se pasa a través del componente `NavItem` hacia el componente `AnimatedLink`. El componente `NavItem` debe aceptar la prop `target` porque debe transmitirse a `AnimatedLink`.

De eso se trata el *prop drilling* / cadenas de props: reenvías una prop desde un componente que realmente no la necesita hacia otro componente que sí la necesita.

Tener algo de *prop drilling* en tu aplicación no es necesariamente malo y definitivamente puedes aceptarlo. Sin embargo, si terminas con cadenas de props más largas (en otras palabras, múltiples componentes intermediarios de paso), puedes utilizar una solución que se discutirá en el Capítulo 11, *Trabajando con Estado Complejo*.

---

### Sección 5: Resumen y puntos clave

- Las props son un concepto clave de React que hace que los componentes sean configurables y, por lo tanto, reutilizables.
- React recopila y pasa automáticamente las props a las funciones de los componentes.
- Tú decides (componente por componente) si deseas utilizar los datos de las props (un objeto) o no.
- Las props se pasan a los componentes como atributos o, a través de la prop especial `children`, entre las etiquetas de apertura y cierre.
- Puedes utilizar características de JavaScript como la desestructuración, la propiedad rest o el operador de propagación (*spread*) para escribir código conciso y flexible.
- Dado que tú estás escribiendo el código, depende de ti cómo deseas pasar los datos a través de las props: ¿entre las etiquetas o como atributos? ¿Un único atributo agrupado o muchos atributos de un solo valor? Depende de ti.

---

### Sección 6: ¿Qué sigue?

Las props te permiten hacer que los componentes sean configurables y reutilizables. Aun así, siguen siendo bastante estáticos. Los datos y, por lo tanto, el resultado de la interfaz de usuario no cambian. No puedes reaccionar a eventos del usuario como clics en botones.

Pero el verdadero poder de React solo se vuelve visible una vez que agregas **eventos** (y reacciones a ellos).

En el próximo capítulo, aprenderás cómo puedes agregar escuchadores de eventos cuando trabajas con React, y aprenderás cómo puedes reaccionar a los eventos y cambiar el estado (invisible y visible) de tu aplicación.

---

### Sección 7: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con las respuestas de ejemplo que se encuentran en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/03-components-props/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/03-components-props/exercises/questions-answers.md):

1. ¿Qué "problema" resuelven las props?
2. ¿Cómo se pasan las props a los componentes?
3. ¿Cómo se consumen las props dentro de una función de componente?
4. ¿Qué opciones existen para pasar (múltiples) props a los componentes?

---

### Sección 8: Aplica lo aprendido

Con este y los capítulos anteriores, ahora tienes suficiente conocimiento básico para construir componentes verdaderamente reutilizables.

A continuación, encontrarás una actividad que te permitirá aplicar todos los conocimientos, incluidos los nuevos conocimientos sobre props, que has adquirido hasta ahora.

#### Actividad 3.1: Creación de una aplicación para mostrar tus objetivos para este libro
Esta actividad se basa en la Actividad 2.2 (*Creación de una aplicación React para registrar tus objetivos para este libro*) del capítulo anterior. Si seguiste los pasos allí, puedes usar tu código existente y mejorarlo agregando props. Alternativamente, también puedes usar la solución proporcionada como punto de partida que está accesible en el siguiente enlace: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-2](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/02-components-jsx/activities/practice-2).

El objetivo de esta actividad es construir componentes `GoalItem` reutilizables que se puedan configurar mediante props. Cada componente `GoalItem` debe recibir y mostrar un título de objetivo y un breve texto de descripción, con información adicional sobre el objetivo.

Los pasos son los siguientes:
1. Completa la segunda actividad del capítulo anterior.
2. Reemplaza los componentes de elementos de objetivos codificados de forma fija por un nuevo componente configurable.
3. Renderiza múltiples componentes de objetivos con diferentes títulos (a través de props).
4. Establece la descripción de texto detallada para cada objetivo entre las etiquetas de apertura y cierre del componente de objetivo.

La interfaz de usuario final podría verse así:

**Figura 3.1**: El resultado final: múltiples objetivos mostrados uno debajo del otro.

> [!NOTE]
> Puedes encontrar una solución de ejemplo completa aquí: [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/03-components-props/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/03-components-props/activities/practice-1).
