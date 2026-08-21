# Parte 1: Fundamentos de React

## Capítulo 11: Trabajando con Estado Complejo

### Objetivos de aprendizaje
Al finalizar este capítulo, serás capaz de:
- Gestionar el estado entre componentes (*cross-component state*) o incluso a nivel de toda la aplicación (en lugar de solo el estado específico de un componente).
- Distribuir datos a través de múltiples componentes.
- Manejar valores y cambios de estado complejos.

---

### Sección 1: Introducción

El estado es uno de los conceptos centrales que debes comprender (y con el que debes trabajar) para usar React de manera efectiva. Básicamente, cada aplicación de React utiliza (muchos) valores de estado en muchos componentes para presentar una interfaz de usuario dinámica y reactiva.

Desde valores de estado simples que contienen un contador cambiante o valores introducidos por los usuarios, hasta valores de estado más complejos como la combinación de múltiples entradas de formulario o información de autenticación de usuario, el estado está en todas partes. En las aplicaciones de React, normalmente se gestiona con la ayuda del Hook `useState()`.

Sin embargo, una vez que comienzas a crear aplicaciones de React más complejas (por ejemplo, tiendas en línea, paneles de administración y sitios similares), es probable que enfrentes varios desafíos relacionados con el estado. Los valores de estado pueden usarse en el componente A pero modificarse en el componente B, o estar compuestos por múltiples valores dinámicos que pueden cambiar por una amplia variedad de razones (por ejemplo, un carrito de compras en una tienda en línea, que es una combinación de productos donde cada producto tiene una cantidad, un precio y posiblemente otros rasgos que pueden modificarse individualmente).

Puedes manejar todos estos problemas con `useState()`, props y los otros conceptos cubiertos en este libro hasta ahora. Pero notarás que las soluciones basadas únicamente en `useState()` adquieren una complejidad que puede ser difícil de entender y mantener. Es por eso que React tiene más herramientas que ofrecer: herramientas creadas para este tipo de problemas, que este capítulo destacará y analizará.

---

### Sección 2: Un problema con el estado entre componentes

Ni siquiera necesitas crear una aplicación de React muy sofisticada para encontrar un problema común: **el estado que abarca múltiples componentes**.

Por ejemplo, podrías estar creando una aplicación de noticias donde los usuarios puedan marcar ciertos artículos como favoritos. Una interfaz de usuario simple podría verse así:

**Figura 11.1**: Una interfaz de usuario de ejemplo.

Como puedes ver en la figura anterior, la lista de artículos está a la izquierda y un resumen de los artículos guardados se encuentra en la barra lateral a la derecha.

Una solución común es dividir esta interfaz de usuario en múltiples componentes. La lista de artículos, específicamente, probablemente estaría en su propio componente, al igual que la barra lateral de resumen de marcadores.

Sin embargo, en ese escenario, ambos componentes necesitarían acceder al mismo estado compartido, es decir, la lista de artículos marcados como favoritos. El componente de la lista de artículos requeriría acceso para agregar (o eliminar) artículos. El componente de la barra lateral de resumen de marcadores lo requeriría, ya que necesita mostrar los artículos guardados.

El árbol de componentes y el uso del estado para este tipo de aplicación podrían verse así:

**Figura 11.2**: Dos componentes hermanos comparten el mismo estado.

En esta figura, puedes ver que el estado se comparte entre estos dos componentes. También ves que los dos componentes tienen un componente principal compartido (el componente `News`, en este ejemplo).

Dado que el estado es utilizado por dos componentes, no lo administrarías en ninguno de ellos. En su lugar, se eleva (*lifted up*), como se describe en el Capítulo 4, *Trabajando con Eventos y Estado* (en la sección *Elevación del estado / Lifting State Up*). Al elevar el estado, los valores de estado y los punteros a las funciones que manipulan los valores de estado se pasan a los componentes reales que necesitan acceso a través de props.

Esto funciona y es un patrón común. Puedes (y debes) seguir usándolo. ¿Pero qué pasa si un componente que necesita acceso a algún estado compartido está profundamente anidado en otros componentes? ¿Qué pasaría si el árbol de componentes de la aplicación del ejemplo anterior se viera así?

**Figura 11.3**: Un árbol de componentes con múltiples capas de componentes dependientes del estado.

En esta figura, puedes ver que el componente `BookmarkSummary` es un componente profundamente anidado. Entre él y el componente `News` (que gestiona el estado compartido), tienes otros dos componentes: el componente `InfoSidebar` y el componente `BookmarkInformation`. En aplicaciones de React más complejas, tener múltiples niveles de anidamiento de componentes, como en este ejemplo, es muy común.

Por supuesto, incluso con esos componentes adicionales, los valores de estado aún se pueden pasar a través de props. Solo necesitas agregar props a todos los componentes intermedios entre el componente que contiene el estado y el componente que necesita el estado. Por ejemplo, debes pasar el valor de estado `bookmarkedArticles` al componente `InfoSidebar` (a través de props) para que ese componente pueda reenviarlo a `BookmarkInformation`:

```javascript
import BookmarkInformation from '../BookmarkSummary/BookmarkInformation.jsx'; 
import classes from './InfoSidebar.module.css'; 

function InfoSidebar({ bookmarkedArticles }) { 
  return ( 
    <aside className={classes.sidebar}> 
      <BookmarkInformation bookmarkedArticles={bookmarkedArticles} /> 
    </aside> 
  ); 
} 

export default InfoSidebar;
```

El mismo procedimiento se repite dentro del componente `BookmarkInformation`.

> [!NOTE]
> Puedes encontrar el ejemplo completo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/examples/01-cross-cmp-state](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/examples/01-cross-cmp-state).

Este tipo de patrón se llama **Prop Drilling** (perforación o traspaso excesivo de props). *Prop drilling* significa que un valor de estado se pasa a través de múltiples componentes a través de props. Y se pasa a través de componentes que no necesitan el estado en absoluto, excepto para reenviarlo a un componente secundario (como lo están haciendo los componentes `InfoSidebar` y `BookmarkInformation` en el ejemplo anterior).

Como desarrollador, normalmente querrás evitar este patrón porque el *prop drilling* tiene algunas debilidades:
- Los componentes que forman parte del *prop drilling* (como `InfoSidebar` o `BookmarkInformation`) ya no son realmente reutilizables porque cualquier componente que quiera usarlos tiene que proporcionar un valor para la prop de estado reenviada.
- El *prop drilling* también genera una gran cantidad de código repetitivo que debe escribirse (el código para aceptar props y reenviar props).
- Refactorizar componentes requiere más trabajo porque se deben agregar o eliminar props de estado.

Por estas razones, el *prop drilling* solo es aceptable si todos los componentes involucrados solo se utilizan en esta parte específica de la aplicación general de React, y la probabilidad de reutilizarlos o refactorizarlos es baja.

Dado que el *prop drilling* debe evitarse (en la mayoría de las situaciones), React ofrece una alternativa: la **Context API**.

---

### Sección 3: Uso de Context para manejar el estado multicomponente

La característica de **Context** de React es una herramienta que te permite **crear un valor que se puede compartir fácilmente entre tantos componentes como sea necesario, sin utilizar props**.

**Figura 11.4**: React Context se adjunta a los componentes para exponerlo a todos los componentes secundarios, sin *prop drilling*.

El uso de la Context API es un proceso de varios pasos, cuyos pasos se describen aquí:
1. Debes **crear un valor de contexto** que se deba compartir.
2. El contexto debe **proporcionarse** (*provide*) en un componente principal de los componentes que necesitan acceso al objeto de contexto.
3. Los componentes que necesitan acceso (para lectura o escritura) deben **suscribirse al contexto**.

React gestiona el valor del contexto (y sus cambios) internamente y lo distribuye automáticamente a todos los componentes que se hayan suscrito al contexto.

Sin embargo, antes de que cualquier componente pueda suscribirse, el primer paso es crear un objeto de contexto. Esto se hace a través de la función **`createContext()`** de React:

```javascript
import { createContext } from 'react'; 

createContext('Hello Context'); // a context with an initial string value 
createContext({}); // a context with an initial (empty) object as a value
```

Esta función toma un valor inicial que debe compartirse. Puede ser cualquier tipo de valor (por ejemplo, una cadena o un número), pero normalmente es un objeto. Esto se debe a que la mayoría de los valores compartidos son una combinación de los valores reales y las funciones que deberían manipular esos valores. Todas estas cosas se agrupan luego en un solo objeto de contexto.

Por supuesto, el valor de contexto inicial también puede ser un valor vacío (por ejemplo, `null`, `undefined`, una cadena vacía, etc.) si es necesario.

`createContext()` también devuelve un valor: un objeto de contexto que debe almacenarse en una variable (o constante) con mayúscula inicial porque en realidad es un componente de React (y los componentes de React deben comenzar con caracteres en mayúscula).

Aquí se muestra cómo se puede llamar a la función `createContext()` para crear un objeto de contexto para el ejemplo discutido anteriormente en este capítulo:

```javascript
import { createContext } from 'react'; 

const BookmarkContext = createContext({ bookmarkedArticles: [] }); 

export default BookmarkContext;
```

Aquí, el valor inicial es un objeto que contiene la propiedad `bookmarkedArticles`, que contiene un array (vacío). También podrías almacenar solo el array como valor inicial (es decir, `createContext([])`), pero un objeto es mejor ya que se le agregará más contenido más adelante en el capítulo.

Este código normalmente se coloca en un archivo de código de contexto separado (por ejemplo, `bookmark-context.jsx`) que a menudo se almacena en una carpeta llamada `store` (porque esta característica de contexto se puede usar como un almacén de estado central) o `context`. Sin embargo, esto es solo una convención y no es un requisito técnico. Puedes colocar este código en cualquier lugar de tu aplicación de React.

Si el archivo solo contiene el código anterior, puede usar `.js` como extensión de archivo ya que no contiene ningún código JSX. Más adelante en este capítulo, esto cambiará; por lo tanto, ya puedes usar `.jsx` como extensión.

Por supuesto, este valor inicial no es un reemplazo para el estado: es un valor estático que nunca cambia. Pero este fue solo el primero de tres pasos relacionados con el contexto. El siguiente paso es proporcionar el contexto.

#### Proveer y administrar valores de Context
Para utilizar valores de contexto en otros componentes, primero debes proporcionar el valor. Esto se hace utilizando el valor devuelto por `createContext()`.

- Cuando utilizas **React 19 o superior**, esa función genera un componente de React que debe envolver a todos los demás componentes que necesitan acceso al valor del contexto.
- Cuando utilizas una versión anterior de React (es decir, **React 18 o anterior**), el valor devuelto por `createContext()` es en cambio un objeto que contiene una propiedad anidada `Provider`. Esa propiedad luego contiene un componente de React que debe envolver a todos los demás componentes que necesitan acceso al valor del contexto.

De cualquier manera, se trata de envolver los componentes con un componente proveedor de contexto (*context provider*).

En el ejemplo anterior, usando React 19 o superior, el componente `BookmarkContext` devuelto por `createContext()` se podría usar en el componente `News` para envolver tanto a los componentes `Articles` como a `InfoSidebar`:

```javascript
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function News() { 
  return ( 
    <BookmarkContext> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContext> 
  ); 
}
```

O, si utilizas React 18 o inferior:

```javascript
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function News() { 
  return ( 
    <BookmarkContext.Provider> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContext.Provider> 
  ); 
}
```

Sin embargo, este código no funciona porque falta una cosa importante: **el componente espera una prop `value`**, que debe contener el valor de contexto actual que debe distribuirse a los componentes interesados. Si bien proporcionas un valor de contexto inicial (que podría haber estado vacío), también debes informar a React sobre el valor de contexto actual porque, con mucha frecuencia, los valores de contexto cambian (después de todo, a menudo se usan como reemplazo del estado entre componentes).

Por lo tanto, el código se podría modificar de esta manera al usar React 19 o superior:

```javascript
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function News() { 
  const bookmarkCtxValue = { bookmarkedArticles: [] }; // for now, it's the same value as used before, for the initial context 
  return ( 
    <BookmarkContext value={bookmarkCtxValue}> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContext> 
  ); 
}
```

O, si utilizas React 18 o inferior:

```javascript
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function News() { 
  const bookmarkCtxValue = { bookmarkedArticles: [] }; // for now, it's the same value as used before, for the initial context 
  return ( 
    <BookmarkContext.Provider value={bookmarkCtxValue}> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContext.Provider> 
  ); 
}
```

Con este código, se distribuye un objeto con una lista de artículos guardados a los componentes descendientes interesados.

Sin embargo, la lista sigue siendo estática. Pero eso se puede cambiar con una herramienta que ya conoces: el Hook `useState()`. Dentro del componente `News`, puedes usar el Hook `useState()` para administrar la lista de artículos guardados, de esta manera:

```javascript
import { useState } from 'react'; 
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function News() { 
  const [savedArticles, setSavedArticles] = useState([]); 
  const bookmarkCtxValue = { 
    bookmarkedArticles: savedArticles // using the state as a value 
  }; 

  return ( 
    <BookmarkContext value={bookmarkCtxValue}> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContext> 
  ); 
}
```

Con este cambio, el contexto pasa de estático a dinámico. Cada vez que cambie el estado `savedArticles`, el valor del contexto cambiará.

Por lo tanto, esa es la pieza que falta cuando se trata de proporcionar el contexto. Si el contexto debe ser dinámico (y modificable desde dentro de algún componente secundario anidado), el valor del contexto también debe incluir un puntero a la función que desencadena una actualización de estado.

Para el ejemplo anterior, el código se ajusta de la siguiente manera:

```javascript
import { useState } from 'react'; 
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import BookmarkContext from '../../store/bookmark-context.jsx'; 

function News() { 
  const [savedArticles, setSavedArticles] = useState([]); 

  function addArticle(article) { 
    setSavedArticles( 
      (prevSavedArticles) => [...prevSavedArticles, article] 
    ); 
  } 

  function removeArticle(articleId) { 
    setSavedArticles( 
      (prevSavedArticles) => prevSavedArticles 
        .filter( 
          (article) => article.id !== articleId 
        ) 
    ); 
  } 

  const bookmarkCtxValue = { 
    bookmarkedArticles: savedArticles, 
    bookmarkArticle: addArticle, 
    unbookmarkArticle: removeArticle 
  }; 

  return ( 
    <BookmarkContext value={bookmarkCtxValue}> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContext> 
  ); 
}
```

Las siguientes son dos cosas importantes modificadas en este fragmento de código:
- Se agregaron dos nuevas funciones: `addArticle` y `removeArticle`.
- Se agregaron propiedades que apuntan a estas funciones en `bookmarkCtxValue`: los métodos `bookmarkArticle` y `unbookmarkArticle`.

La función `addArticle` agrega un nuevo artículo (que se debe marcar como favorito) al estado `savedArticles`. Se utiliza la forma funcional de actualizar el valor de estado ya que el nuevo valor de estado depende del valor de estado anterior (el artículo guardado se agrega a la lista de artículos ya guardados).

De manera similar, la función `removeArticle` elimina un artículo de la lista `savedArticles` filtrando la lista existente de modo que se mantengan todos los elementos, excepto el que tiene un valor de `id` coincidente.

Si el componente `News` no utilizara la nueva función de contexto, sería un componente que utiliza estado, tal como has visto muchas veces antes en este libro. Pero ahora, al usar la Context API de React, esas capacidades existentes se combinan con una nueva función (el contexto) para crear un valor dinámico y distribuible.

Cualquier componente anidado en los componentes `Articles` o `InfoSidebar` (o sus componentes descendientes) podrá acceder a este valor de contexto dinámico y a los métodos `bookmarkArticle` y `unbookmarkArticle` en el objeto de contexto, sin ningún tipo de *prop drilling*.

> [!NOTE]
> No es obligatorio crear valores de contexto dinámicos. También podrías distribuir un valor estático a componentes anidados. Esto es posible pero es un escenario poco común, ya que la mayoría de las aplicaciones de React normalmente necesitan valores de estado dinámicos que puedan cambiar entre componentes.

#### Uso de Context en componentes anidados
Con el contexto creado y proporcionado, está listo para ser utilizado por componentes que necesitan acceder o cambiar el valor del contexto.

Para hacer que el valor del contexto sea accesible a los componentes anidados dentro del componente de contexto (`BookmarkContext`, en el ejemplo anterior), React ofrece el Hook **`use()`**.

Este Hook, sin embargo, solo está disponible cuando se trabaja con **React 19 o superior**. En proyectos que utilizan versiones anteriores de React, en su lugar usarías el Hook **`useContext()`** para acceder a algún valor de contexto. Ese Hook también sigue siendo compatible con React 19; por lo tanto, puedes usar cualquiera de los dos Hooks para obtener un valor de contexto.

El Hook `use()` es un poco más flexible que el Hook `useContext()` ya que, a diferencia de cualquier otro Hook, en realidad también **se puede usar dentro de sentencias `if` o bucles**. Además, el Hook se puede utilizar para más cosas que obtener acceso a valores de contexto; por lo tanto, verás `use()` nuevamente en el Capítulo 17, *Entendiendo React Suspense y el Hook use()*.

Como se mencionó, al trabajar con React 19, si estás intentando obtener acceso a un valor de contexto, se pueden usar tanto `use()` como `useContext()`. Tanto `use()` como `useContext()` requieren un argumento: el objeto de contexto que se creó mediante `createContext()`, es decir, el valor devuelto por esa función. Como resultado, obtendrás el valor pasado al componente proveedor de contexto (es decir, el valor establecido para su prop `value`). Cuando trabajes con React 19 o superior, dado que `use()` es un poco más flexible y ciertamente requiere escribir un poco menos, puedes ignorar `useContext()` y usar el Hook `use()` para acceder a los valores de contexto.

Para el ejemplo anterior, el valor del contexto se puede utilizar en el componente `BookmarkSummary` de esta manera:

```javascript
import { use } from 'react'; // or import useContext for React < 19 
import BookmarkContext from '../../store/bookmark-context.jsx'; 
import classes from './BookmarkSummary.module.css'; 

function BookmarkSummary() { 
  const bookmarkCtx = use(BookmarkContext); // React < 19: const bookmarkCtx = useContext(BookmarkContext); 
  const numberOfArticles = bookmarkCtx.bookmarkedArticles.length; 

  return ( 
    <> 
      <p className={classes.summary}> 
        {numberOfArticles} articles bookmarked 
      </p> 
      <ul className={classes.list}> 
        {bookmarkCtx.bookmarkedArticles.map((article) => ( 
          <li key={article.id}>{article.title}</li> 
        ))} 
      </ul> 
    </> 
  ); 
} 

export default BookmarkSummary;
```

En este código, `use()` recibe el valor `BookmarkContext`, que se importa desde el archivo `store/bookmark-context.jsx`. Luego devuelve el valor almacenado en el contexto, que es el `bookmarkCtxValue` que se encuentra en el ejemplo de código anterior. Como puedes ver en ese fragmento, `bookmarkCtxValue` es un objeto con tres propiedades: `bookmarkedArticles`, `bookmarkArticle` (un método) y `unbookmarkArticle` (también un método).

Este objeto devuelto se almacena en una constante `bookmarkCtx`. Cada vez que cambie el valor del contexto (porque se ejecuta la función de actualización de estado `setSavedArticles` en el componente `News`), React también volverá a ejecutar este componente `BookmarkSummary` y, por lo tanto, `bookmarkCtx` contendrá el valor de estado más reciente.

Finalmente, en el componente `BookmarkSummary`, se accede a la propiedad `bookmarkedArticles` en el objeto `bookmarkCtx`. Esta lista de artículos luego se utiliza para calcular el número de artículos guardados, generar un breve resumen y mostrar la lista en la pantalla.

De manera similar, `BookmarkContext` se puede utilizar mediante `use()` en el componente `Articles`:

```javascript
import { use } from 'react'; // other imports 

function Articles() { 
  const bookmarkCtx = use(BookmarkContext); // or: const bookmarkCtx = useContext(BookmarkContext) 

  return ( 
    <ul> 
      {dummyArticles.map((article) => { 
        const isBookmarked = bookmarkCtx.bookmarkedArticles.some( 
          (bArticle) => bArticle.id === article.id 
        ); 
        // default icon: Empty bookmark icon, because not bookmarked 
        let buttonIcon = <FaRegBookmark />; 
        if (isBookmarked) { 
          buttonIcon = <FaBookmark />; 
        } 
        return ( 
          <li key={article.id}> 
            <h2>{article.title}</h2> 
            <p>{article.description}</p> 
            <button>{buttonIcon}</button> 
          </li> 
        ); 
      })} 
    </ul> 
  ); 
}
```

En este componente, el contexto se utiliza para determinar si un artículo determinado está marcado actualmente como favorito o no (esta información es necesaria para cambiar el icono y la funcionalidad del botón).

Así es como se pueden leer los valores de contexto (ya sean estáticos o dinámicos) en los componentes. Por supuesto, también se pueden modificar, como se explica en la siguiente sección.

#### Cambio de Context desde componentes anidados
La función de contexto de React se utiliza a menudo para compartir datos entre múltiples componentes sin utilizar props. Por lo tanto, también es bastante común que algunos componentes deban manipular esos datos. Por ejemplo, el valor de contexto para un carrito de compras debe poder ajustarse desde el componente que muestra los productos (porque probablemente tengan un botón "Agregar al carrito").

Sin embargo, para cambiar los valores de contexto desde dentro de un componente anidado, no puedes simplemente sobrescribir el valor de contexto almacenado. El siguiente código no funcionaría según lo previsto:

```javascript
const bookmarkCtx = use(BookmarkContext); // Note: This does NOT work 
bookmarkCtx.bookmarkedArticles = []; // setting the articles to an empty array
```

Este código no funciona. Del mismo modo que no deberías intentar actualizar el estado simplemente asignando un nuevo valor, no puedes actualizar los valores de contexto asignando un nuevo valor. Es por eso que se agregaron dos métodos (`bookmarkArticle` y `unbookmarkArticle`) al valor de contexto en la sección *Proveer y administrar valores de Context*. Estos dos métodos apuntan a funciones que desencadenan actualizaciones de estado (a través de la función de actualización de estado proporcionada por `useState()`).

Por lo tanto, en el componente `Articles`, donde los artículos se pueden marcar o desmarcar mediante clics de botones, se debe llamar a estos métodos:

```javascript
// This code is part of the Article component function 
// default action => bookmark article, because not bookmarked yet 
let buttonAction = () => bookmarkCtx.bookmarkArticle(article); 
// default button icon: Empty bookmark icon, because not bookmarked 
let buttonIcon = <FaRegBookmark />; 
if (isBookmarked) { 
  buttonAction = () => bookmarkCtx.unbookmarkArticle(article.id); 
  buttonIcon = <FaBookmark />; 
}
```

Los métodos `bookmarkArticle` y `unbookmarkArticle` se llaman dentro de funciones anónimas que se almacenan en una variable `buttonAction`. Esa variable se asigna a la prop `onClick` del `<button>`.

Con este código, el valor del contexto se puede modificar exitosamente. Gracias a los pasos dados en la sección anterior (*Uso de Context en componentes anidados*), cada vez que se actualiza el valor del contexto, se refleja automáticamente en la interfaz de usuario.

> [!NOTE]
> El código de ejemplo terminado se puede encontrar en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/examples/02-cross-cmp-state-with-context](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/examples/02-cross-cmp-state-with-context).

---

### Sección 4: Uso eficiente de la Context API

Ser capaz de crear, proporcionar, acceder y cambiar el contexto es importante: en última instancia, son estas cosas las que te permiten utilizar la Context API de React en tus aplicaciones. Pero a medida que tus aplicaciones (y por lo tanto probablemente también tus valores de contexto) se vuelven más complejas, también es importante configurar y administrar tu contexto de manera eficiente, por ejemplo, obteniendo el soporte adecuado del IDE.

#### Obtener un mejor autocompletado de código
En la sección *Uso de Context para manejar el estado multicomponente*, se creó un objeto de contexto mediante `createContext()`. Esa función recibió un valor de contexto inicial: un objeto que contiene una propiedad `bookmarkedArticles`, en el ejemplo anterior.

En este ejemplo, el valor de contexto inicial no es demasiado importante. No se utiliza a menudo porque se sobrescribe con un nuevo valor dentro del componente `News` de todos modos. Sin embargo, dependiendo del Entorno de Desarrollo Integrado (**IDE**) que estés utilizando, puedes obtener un mejor autocompletado de código al definir un valor de contexto inicial que tenga la misma forma y estructura que el valor de contexto final que se administrará en otros componentes de React.

Por lo tanto, dado que se agregaron dos métodos al valor de contexto en la sección *Proveer y administrar valores de Context*, esos métodos también deberían agregarse al valor de contexto inicial en `store/bookmark-context.jsx`:

```javascript
const BookmarkContext = createContext({ 
  bookmarkedArticles: [], 
  bookmarkArticle: () => {}, 
  unbookmarkArticle: () => {} 
}); 

export default BookmarkContext;
```

Los dos métodos se agregan como funciones vacías que no hacen nada porque la lógica real se establece en el componente `News`. Los métodos solo se agregan a este valor de contexto inicial para proporcionar un mejor autocompletado en el IDE. Por lo tanto, este paso es opcional.

#### ¿Context o elevar el estado (*Lifting State Up*)?
En este punto, ahora tienes dos herramientas para administrar el estado entre componentes:
- Puedes **elevar el estado**, como se describió anteriormente en el libro (en el Capítulo 4, *Trabajando con Eventos y Estado*, en la sección *Elevación del estado / Lifting State Up*).
- Alternativamente, puedes usar la **Context API** de React, como se explicó en este capítulo.

¿Cuál de los dos enfoques deberías utilizar en cada escenario?

En última instancia, depende de ti cómo gestiones esto, pero hay algunas reglas sencillas que puedes seguir:
- **Eleva el estado** si solo necesitas compartir el estado a través de uno o dos niveles de anidamiento de componentes.
- **Utiliza la Context API** si tienes cadenas largas de componentes (es decir, anidamiento profundo de componentes) con estado compartido. Una vez que comiences a usar mucho *prop drilling*, es hora de considerar la función de contexto de React.
- **Utiliza también la Context API** si tienes un árbol de componentes relativamente plano pero deseas reutilizar componentes (es decir, no deseas usar props para pasar el estado a los componentes).

#### Externalización de la lógica de Context en componentes separados
Con los pasos explicados anteriormente, tienes todo lo necesario para administrar el estado entre componentes mediante el contexto.

Pero hay un patrón que puedes considerar para administrar el estado y el valor del contexto dinámico: **crear un componente separado para proporcionar (y administrar) el valor del contexto**.

En el ejemplo anterior, se utilizó el componente `News` para proporcionar el contexto y administrar su valor (dinámico, basado en estado). Si bien esto funciona, tus componentes pueden volverse innecesariamente complejos si tienen que lidiar con la gestión del contexto. Por lo tanto, crear un componente dedicado y separado para eso puede generar un código más fácil de entender y mantener.

Para el ejemplo anterior, eso significa que, dentro del archivo `store/bookmark-context.jsx`, podrías crear un componente `BookmarkContextProvider` que se vea así:

```javascript
export function BookmarkContextProvider({ children }) { 
  const [savedArticles, setSavedArticles] = useState([]); 

  function addArticle(article) { 
    setSavedArticles( 
      (prevSavedArticles) => [...prevSavedArticles, article] 
    ); 
  } 

  function removeArticle(articleId) { 
    setSavedArticles((prevSavedArticles) => prevSavedArticles.filter((article) => article.id !== articleId) 
    ); 
  } 

  const bookmarkCtxValue = { 
    bookmarkedArticles: savedArticles, 
    bookmarkArticle: addArticle, 
    unbookmarkArticle: removeArticle, 
  }; 

  return ( 
    <BookmarkContext value={bookmarkCtxValue}> 
      {children} 
    </BookmarkContext> 
  ); 
}
```

Este componente contiene toda la lógica relacionada con la gestión de una lista de artículos guardados a través del estado. Crea el mismo valor de contexto que antes (un valor que contiene la lista de artículos, así como dos métodos para actualizar esa lista).

Sin embargo, el componente `BookmarkContextProvider` hace una cosa adicional: utiliza la prop especial `children` (cubierta en el Capítulo 3, *Componentes y Props*, en la sección *La Prop especial "children"*) para envolver lo que se pase entre las etiquetas del componente `BookmarkContextProvider` con `BookmarkContext`.

Esto permite el uso del componente `BookmarkContextProvider` en el componente `News`, de la siguiente manera:

```javascript
import Articles from '../Articles/Articles.jsx'; 
import InfoSidebar from '../InfoSidebar/InfoSidebar.jsx'; 
import { BookmarkContextProvider } from '../../store/bookmark-context.jsx'; 

function News() { 
  return ( 
    <BookmarkContextProvider> 
      <Articles /> 
      <InfoSidebar /> 
    </BookmarkContextProvider> 
  ); 
} 

export default News;
```

En lugar de administrar todo el valor del contexto, el componente `News` ahora simplemente importa el componente `BookmarkContextProvider` y envuelve ese componente alrededor de `Articles` e `InfoSidebar`. El componente `News`, por lo tanto, es más limpio y conciso.

> [!NOTE]
> Este patrón es totalmente opcional. No es una práctica recomendada oficial ni produce ningún beneficio de rendimiento. Es solo un patrón que puede ayudar a mantener las funciones de tus componentes limpias y concisas.
> También vale la pena mencionar que existe un patrón relacionado para consumir contexto. Sin embargo, ese patrón se basa en la creación de un Custom Hook de React, un concepto que se cubrirá en el próximo capítulo. Por lo tanto, el patrón de consumo de contexto mencionado también se cubrirá en el próximo capítulo.

#### Combinación de múltiples contextos
Especialmente en aplicaciones de React más grandes y con más funciones, es posible (y bastante probable) que necesites trabajar con múltiples valores de contexto que probablemente no estén relacionados entre sí. Por ejemplo, una tienda en línea podría usar un contexto para administrar el carrito de compras, otro contexto para el estado de autenticación del usuario y otro valor de contexto para rastrear los análisis de la página.

React admite totalmente casos de uso como este. Puedes crear, administrar, proporcionar y usar tantos valores de contexto como sea necesario. Puedes administrar múltiples valores (relacionados o no relacionados) en un solo contexto o usar múltiples contextos. Puedes proporcionar múltiples contextos en el mismo componente o en componentes diferentes. Depende totalmente de ti y de los requisitos de tu aplicación.

También puedes usar múltiples contextos en el mismo componente (lo que significa que puedes llamar a `use()` o `useContext()` múltiples veces, con diferentes valores de contexto).

---

### Sección 5: Limitaciones de `useState()`

Hasta ahora en este capítulo, se ha explorado la complejidad del estado entre componentes. Pero la gestión del estado también puede resultar desafiante en escenarios donde algún estado solo se utiliza dentro de un único componente.

`useState()` es una gran herramienta para la gestión del estado en la mayoría de los escenarios (por supuesto, ahora mismo, también es la única herramienta que se ha cubierto). Por lo tanto, `useState()` debería ser tu opción predeterminada para gestionar el estado. Pero `useState()` puede alcanzar sus límites si necesitas **derivar un nuevo valor de estado que se basa en el valor de otra variable de estado**, como en este ejemplo:

```javascript
setIsLoading(fetchedPosts ? false : true);
```

Este breve fragmento está tomado de un componente donde se envía una solicitud HTTP para obtener algunas publicaciones de blog:

```javascript
function App() { 
  const [fetchedPosts, setFetchedPosts] = useState(null); 
  const [isLoading, setIsLoading] = useState(false); 
  const [error, setError] = useState(); 

  const fetchPosts = useCallback(async function fetchPosts() { 
    setIsLoading(fetchedPosts ? false : true); 
    try { 
      const response = await fetch( 
        'https://jsonplaceholder.typicode.com/posts' 
      ); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      setIsLoading(false); 
      setError(null); 
      setFetchedPosts(posts); 
    } catch (error) { 
      setIsLoading(false); 
      setError(error.message); 
      setFetchedPosts(null); 
    } 
  }, []); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return ( 
    <> 
      {isLoading && <p>Loading...</p>} 
      {error && <p>{error}</p>} 
      {fetchedPosts && <BlogPosts posts={fetchedPosts} />} 
    </> 
  ); 
}
```

> [!NOTE]
> Encontrarás el código de ejemplo completo en GitHub en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/examples/04-complex-usestate](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/examples/04-complex-usestate).

Al iniciar la solicitud, un valor de estado `isLoading` (responsable de mostrar un indicador de carga en la pantalla) debe establecerse en `true` solo si no se obtuvieron datos antes. Si los datos se obtuvieron antes (es decir, `fetchedPosts` no es `null`), esos datos aún deberían mostrarse en la pantalla, en lugar de algún indicador de carga.

A primera vista, este código puede no parecer problemático. Pero en realidad viola una regla importante relacionada con `useState()`: **no debes hacer referencia al estado actual para establecer un nuevo valor de estado**. Si necesitas hacerlo, deberías utilizar la forma funcional de la función de actualización de estado (consulta la sección *Actualización correcta del estado en función del estado anterior* del Capítulo 4, *Trabajando con Eventos y Estado*).

Sin embargo, en el ejemplo anterior, esta solución no funcionará. Si cambias a la forma funcional de actualización de estado, solo obtienes acceso al valor actual del estado que estás intentando actualizar. No obtienes acceso (seguro) al valor actual de algún otro estado. En el ejemplo anterior, se hace referencia a otro estado (`fetchedPosts` en lugar de `isLoading`). Por lo tanto, debes violar la regla mencionada.

Esta infracción también tiene consecuencias reales (en este ejemplo). El siguiente fragmento de código forma parte de una función llamada `fetchPosts`, que está envuelta con `useCallback()`:

```javascript
const fetchPosts = useCallback(async function fetchPosts() { 
  setIsLoading(fetchedPosts ? false : true); 
  setError(null); 
  try { 
    const response = await fetch( 
      'https://jsonplaceholder.typicode.com/posts' 
    ); 
    if (!response.ok) { 
      throw new Error('Failed to fetch posts.'); 
    } 
    const posts = await response.json(); 
    setIsLoading(false); 
    setError(null); 
    setFetchedPosts(posts); 
  } catch (error) { 
    setIsLoading(false); 
    setError(error.message); 
    setFetchedPosts(null); 
  } 
}, []);
```

Esta función envía una solicitud HTTP y cambia múltiples valores de estado según el estado de la solicitud.

`useCallback()` se utiliza para evitar un bucle infinito relacionado con `useEffect()` (consulta el Capítulo 8, *Manejo de Efectos Secundarios*, para obtener más información sobre `useEffect()`, bucles infinitos y `useCallback()` como solución). Normalmente, `fetchedPosts` debería agregarse como una dependencia al array de dependencias pasado como segundo argumento a la función `useCallback()`. Sin embargo, en este ejemplo, esto no se puede hacer porque `fetchedPosts` se modifica dentro de la función envuelta por `useCallback()`, y el valor del estado, por lo tanto, no es solo una dependencia sino que también se modifica activamente. Esto provoca un bucle infinito.

Como resultado, se muestra una advertencia en la terminal y no se logra el comportamiento previsto de no mostrar el indicador de carga si los datos se obtuvieron antes:

**Figura 11.5**: En la terminal se muestra una advertencia sobre la dependencia faltante.

Problemas como el que se acaba de describir son comunes si tienes múltiples valores de estado relacionados que dependen entre sí.

Una posible solución sería pasar de múltiples secciones de estado individuales (`fetchedPosts`, `isLoading` y `error`) a un único valor de estado combinado (es decir, a un objeto). Eso aseguraría que todos los valores de estado se agrupen y, por lo tanto, se pueda acceder a ellos de forma segura cuando se utiliza la forma funcional de actualización de estado. El código de actualización de estado podría verse así:

```javascript
setHttpState(prevState => ({ 
  fetchedPosts: prevState.fetchedPosts, 
  isLoading: prevState.fetchedPosts ? false : true, 
  error: null 
}));
```

Esta solución funcionaría. Sin embargo, terminar con objetos de estado cada vez más complejos (y anidados), administrados a través de `useState()`, normalmente no es deseable ya que puede dificultar la gestión del estado y sobrecargar el código de tu componente.

Por eso React ofrece una alternativa a `useState()`: el Hook **`useReducer()`**.

---

### Sección 6: Gestión del estado con `useReducer()`

Al igual que `useState()`, `useReducer()` es un Hook de React. Y al igual que `useState()`, es un Hook que puede desencadenar reevaluaciones de funciones de componentes. Pero, por supuesto, funciona de manera ligeramente diferente; de lo contrario, sería un Hook redundante.

`useReducer()` es un Hook pensado para usarse en la **gestión de objetos de estado complejos**. Rara vez (probablemente nunca) lo usarás para administrar valores simples de cadenas o números.

Este Hook toma dos argumentos principales:
1. Una **función reductora** (*reducer function*).
2. Un **valor de estado inicial**.

Esto plantea una pregunta importante: ¿qué es una función reductora?

#### Comprensión de las funciones reductoras (*Reducer Functions*)
En el contexto de `useReducer()`, una función reductora es una función que recibe dos parámetros:
1. El valor del estado actual.
2. Una acción que fue despachada (*dispatched*).

Además de recibir argumentos, una función reductora también **debe devolver un valor**: el nuevo estado. Se llama función reductora porque reduce el estado antiguo (combinado con una acción) a un nuevo estado.

Para que todo esto sea un poco más fácil de entender y analizar, el siguiente fragmento de código muestra cómo se usa `useReducer()` junto con dicha función reductora:

```javascript
const initialHttpState = { 
  data: null, 
  isLoading: false, 
  error: null, 
}; 

function httpReducer(state, action) { 
  if (action.type === 'FETCH_START') { 
    return { 
      ...state, // copying the existing state 
      isLoading: state.data ? false : true, 
      error: null, 
    }; 
  } 
  if (action.type === 'FETCH_ERROR') { 
    return { 
      data: null, 
      isLoading: false, 
      error: action.payload, 
    }; 
  } 
  if (action.type === 'FETCH_SUCCESS') { 
    return { 
      data: action.payload, 
      isLoading: false, 
      error: null, 
    }; 
  } 
  return initialHttpState; // default value for unknown actions 
} 

function App() { 
  useReducer(httpReducer, initialHttpState); // more component code, not relevant for this snippet / explanation 
}
```

En la parte inferior de este fragmento, puedes ver que se llama a `useReducer()` dentro de la función del componente `App`. Al igual que todos los Hooks de React, debe llamarse dentro de funciones de componentes u otros Hooks. También puedes ver los dos argumentos que se mencionaron anteriormente (la función reductora y el valor del estado inicial) pasados a `useReducer()`.

`httpReducer` es la función reductora. La función toma dos argumentos (`state`, que es el estado anterior, y `action`, que es la acción despachada) y devuelve diferentes objetos de estado para diferentes tipos de acción.

Esta función reductora se encarga de todas las posibles actualizaciones de estado. Por lo tanto, toda la lógica de actualización de estado se externaliza del componente (ten en cuenta que `httpReducer` se define fuera de la función del componente).

Pero el componente debe, por supuesto, poder activar las actualizaciones de estado definidas. Ahí es donde las acciones adquieren importancia.

> [!NOTE]
> En este ejemplo, la función reductora se crea fuera de la función del componente. También podrías crearla dentro de la función del componente, pero eso no es recomendable. Si creas la función reductora dentro de la función del componente, técnicamente se recreará cada vez que se ejecute la función del componente. Esto afecta el rendimiento innecesariamente ya que la función reductora no necesita acceso a ningún valor de la función del componente (estado o props).

#### Despacho de acciones (*Dispatching Actions*)
El código mostrado anteriormente está incompleto. Al llamar a `useReducer()` en una función de componente, no solo toma dos argumentos; en su lugar, el Hook también devuelve un valor: un array con exactamente dos elementos (al igual que `useState()`, aunque los elementos son diferentes).

`useReducer()` debería, por lo tanto, usarse de esta manera (en el componente `App`):

```javascript
const [httpState, dispatch] = useReducer( 
  httpReducer, 
  initialHttpState 
);
```

En este fragmento, se utiliza la desestructuración de arrays para almacenar los dos elementos (¡y siempre son exactamente dos!) en dos constantes diferentes: `httpState` y `dispatch`.

- El **primer elemento** en el array devuelto (`httpState`, en este caso) es el valor de estado devuelto por la función reductora. Se actualiza (lo que significa que React llama a la función del componente) cada vez que la función reductora se ejecuta nuevamente. El elemento se llama `httpState` en este ejemplo porque contiene el valor del estado, que está relacionado con una solicitud HTTP en esta instancia. Dicho esto, cómo nombres el elemento en tu caso depende de ti.
- El **segundo elemento** (`dispatch`, en el ejemplo) es una función. Es una función a la que se puede llamar para desencadenar una actualización de estado (es decir, para ejecutar la función reductora nuevamente). Cuando se ejecuta, la función `dispatch` debe recibir un argumento: el valor de acción que estará disponible dentro de la función reductora (a través del segundo argumento de la función reductora).

Así es como se puede usar `dispatch` en un componente:

```javascript
dispatch({ type: 'FETCH_START' });
```

El elemento se llama `dispatch` en el ejemplo porque es una función utilizada para despachar acciones a la función reductora. Al igual que antes, el nombre depende de ti, pero `dispatch` es un nombre comúnmente elegido.

La forma y estructura de ese valor de acción también dependen totalmente de ti, pero a menudo se establece en un objeto que contiene una propiedad `type`. La propiedad `type` se utiliza en la función reductora para realizar diferentes acciones para diferentes tipos de acciones. Por lo tanto, `type` actúa como un identificador de acción. Puedes ver cómo se utiliza la propiedad `type` dentro de la función `httpReducer`:

```javascript
function httpReducer(state, action) { 
  if (action.type === 'FETCH_START') { 
    return { 
      ...state, // copying the existing state 
      isLoading: state.data ? false : true, 
      error: null, 
    }; 
  } 
  if (action.type === 'FETCH_ERROR') { 
    return { 
      data: null, 
      isLoading: false, 
      error: action.payload, 
    }; 
  } 
  if (action.type === 'FETCH_SUCCESS') { 
    return { 
      data: action.payload, 
      isLoading: false, 
      error: null, 
    }; 
  } 
  return initialHttpState; // default value for unknown actions 
}
```

Puedes agregar tantas propiedades al objeto de acción como sea necesario. En el ejemplo anterior, algunas actualizaciones de estado acceden a `action.payload` para extraer algunos datos adicionales del objeto de acción. Dentro de un componente, pasarías datos junto con la acción de esta manera:

```javascript
dispatch({ type: 'FETCH_SUCCESS', payload: posts });
```

Nuevamente, el nombre de la propiedad (`payload`) depende de ti, pero pasar datos adicionales junto con la acción te permite realizar actualizaciones de estado que dependen de los datos generados por la función del componente.

Aquí está el código final completo para toda la función del componente `App`:

```javascript
// code for httpReducer etc. did not change 
function App() { 
  const [httpState, dispatch] = useReducer( 
    httpReducer, 
    initialHttpState 
  ); 

  // Using useCallback() to prevent an infinite loop in useEffect() 
  const fetchPosts = useCallback(async function fetchPosts() { 
    dispatch({ type: 'FETCH_START' }); 
    try { 
      const response = await fetch( 
        'https://jsonplaceholder.typicode.com/posts' 
      ); 
      if (!response.ok) { 
        throw new Error('Failed to fetch posts.'); 
      } 
      const posts = await response.json(); 
      dispatch({ type: 'FETCH_SUCCESS', payload: posts }); 
    } catch (error) { 
      dispatch({ type: 'FETCH_ERROR', payload: error.message }); 
    } 
  }, []); 

  useEffect( 
    function () { 
      fetchPosts(); 
    }, 
    [fetchPosts] 
  ); 

  return ( 
    <> 
      <header> 
        <h1>Complex State Blog</h1> 
        <button onClick={fetchPosts}>Load Posts</button> 
      </header> 
      {httpState.isLoading && <p>Loading...</p>} 
      {httpState.error && <p>{httpState.error}</p>} 
      {httpState.data && <BlogPosts posts={httpState.data} />} 
    </> 
  ); 
}
```

En este fragmento de código, puedes ver cómo se despachan diferentes acciones (con diferentes propiedades `type` y, a veces, `payload`). También puedes ver que el valor de `httpState` se utiliza para mostrar diferentes elementos de la interfaz de usuario según el estado (por ejemplo, `<p>Loading…</p>` se muestra si `httpState.isLoading` es `true`).

---

### Sección 7: Resumen y puntos clave

- La gestión del estado puede tener sus desafíos, especialmente cuando se trata de un estado entre componentes (o en toda la aplicación) o valores de estado complejos.
- El estado entre componentes se puede gestionar **elevando el estado** (*lifting state up*) o utilizando la **Context API** de React.
- La Context API suele ser preferible si realizas mucho *prop drilling* (reenvío de valores de estado a través de props a través de múltiples capas de componentes).
- Al utilizar la Context API, utilizas `createContext()` para crear un nuevo objeto de contexto.
- El objeto de contexto creado es un componente que debe envolver la parte del árbol de componentes que debe obtener acceso al contexto.
- Cuando se trabaja con React 18 o versiones anteriores, el objeto de contexto en sí no es un componente, sino un objeto que ofrece una propiedad anidada `Provider` que es un componente.
- Los componentes pueden acceder al valor del contexto a través de los Hooks `use()` (con React 19 o superior) o `useContext()`.
- Para gestionar valores de estado complejos, **`useReducer()`** puede ser una buena alternativa a `useState()`.
- `useReducer()` utiliza una función reductora que convierte el estado actual y una acción despachada en un nuevo valor de estado.
- `useReducer()` devuelve un array con exactamente dos elementos: el valor de estado y una función `dispatch`, que se utiliza para despachar acciones.

---

### Sección 8: ¿Qué sigue?

Ser capaz de gestionar valores de estado tanto simples como complejos de manera eficiente es importante. Este capítulo introdujo dos herramientas cruciales que ayudan con esa tarea.

Con los Hooks `use()`, `useContext()` y `useReducer()` de la Context API, se introdujeron tres nuevos Hooks de React. Combinados con todos los demás Hooks cubiertos hasta ahora en el libro, estos marcan los últimos Hooks de React que necesitarás en tu trabajo diario como desarrollador de React.

Sin embargo, como desarrollador de React, no estás limitado a los Hooks integrados: también puedes crear tus propios Hooks. El próximo capítulo finalmente explorará cómo funciona eso y por qué querrías crear **Custom Hooks** en primer lugar.

---

### Sección 9: ¡Pon a prueba tus conocimientos!

Pon a prueba tus conocimientos sobre los conceptos tratados en este capítulo respondiendo a las siguientes preguntas. Luego puedes comparar tus respuestas con los ejemplos que se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/11-complex-state/exercises/questions-answers.md](https://github.com/mschwarzmueller/book-react-key-concepts-e2/blob/11-complex-state/exercises/questions-answers.md):

1. ¿Qué problema se puede solucionar con la Context API de React?
2. ¿Qué tres pasos principales se deben seguir al utilizar la Context API?
3. ¿Cuándo se prefiere `useReducer()` sobre `useState()`?
4. Al trabajar con `useReducer()`, ¿cuál es el papel de las acciones?

---

### Sección 10: Aplica lo aprendido

Aplica tus conocimientos sobre la Context API y el Hook `useReducer()` a algunos problemas reales.

#### Actividad 11.1: Migración de una aplicación a la Context API
En esta actividad, tu tarea es mejorar un proyecto de React existente. Actualmente, la aplicación está construida sin la Context API y, por lo tanto, el estado entre componentes se gestiona elevando el estado. En este proyecto, el *prop drilling* es la consecuencia en algunos componentes. Por lo tanto, el objetivo es ajustar la aplicación de modo que se utilice la Context API para la gestión del estado entre componentes.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-1-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-1-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-1-start` en este caso) para usar la versión correcta del código.

El proyecto proporcionado también utiliza muchas de las funciones cubiertas en capítulos anteriores. Tómate tu tiempo para analizarlo y comprender el código proporcionado. Esta es una gran práctica y te permite ver muchos conceptos clave en acción.

Una vez que hayas descargado el código y ejecutado `npm install` en la carpeta del proyecto (para instalar todas las dependencias requeridas), puedes iniciar el servidor de desarrollo mediante `npm run dev`. Como resultado, al visitar `localhost:5173`, deberías ver la siguiente interfaz de usuario:

**Figura 11.6**: El proyecto inicial en ejecución.

Para completar la actividad, los pasos son los siguientes:
1. Crea un nuevo contexto para los elementos del carrito.
2. Crea un componente `Provider` para el contexto y maneja todos los cambios de estado relacionados con el contexto allí.
3. Proporciona el contexto (con la ayuda del componente `Provider`) y asegúrate de que todos los componentes que necesitan acceso al contexto tengan acceso.
4. Elimina la lógica anterior (donde se elevó el estado).
5. Utiliza el contexto en todos los componentes que necesitan acceder a él.

La interfaz de usuario debe ser la misma que la que se muestra en la Figura 11.6 una vez que hayas completado la actividad. Asegúrate de que la interfaz de usuario funcione exactamente como lo hacía antes de implementar las funciones de contexto de React.

> [!NOTE]
> Todos los archivos de código utilizados para esta actividad, y la solución, se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-1](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-1).

#### Actividad 11.2: Reemplazo de `useState()` por `useReducer()`
En esta actividad, tu tarea es reemplazar los Hooks `useState()` en el componente `Form` con `useReducer()`. Utiliza solo una única función reductora (y por lo tanto solo una llamada a `useReducer()`) y fusiona todos los valores de estado relevantes en un solo objeto de estado.

> [!NOTE]
> Puedes encontrar el código inicial para esta actividad en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-2-start](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-2-start). Al descargar este código, siempre descargarás el repositorio completo. Asegúrate de navegar luego a la subcarpeta con el código inicial (`activities/practice-2-start` en este caso) para usar la versión correcta del código.

El proyecto proporcionado también utiliza muchas de las funciones cubiertas en capítulos anteriores. Tómate tu tiempo para analizarlo y comprender el código proporcionado. Esta es una gran práctica y te permite ver muchos conceptos clave en acción.

Una vez que hayas descargado el código y ejecutado `npm install` en la carpeta del proyecto (para instalar todas las dependencias requeridas), puedes iniciar el servidor de desarrollo mediante `npm run dev`. Como resultado, al visitar `localhost:5173`, deberías ver la siguiente interfaz de usuario:

**Figura 11.7**: El proyecto inicial en ejecución.

En el proyecto inicial proporcionado, los usuarios obtienen uno de tres resultados al hacer clic en el botón `Submit`:
- Si uno o ambos campos de entrada no recibieron ninguna entrada, un mensaje de error les indica a los usuarios que completen el formulario.
- Si los usuarios ingresaron valores en ambos campos de entrada, pero al menos una de las entradas contiene un valor no válido, se muestra un mensaje de error diferente.
- Si los usuarios ingresaron valores válidos en ambos campos de entrada, los valores ingresados se imprimen en la consola de JavaScript de las herramientas de desarrollo.

Para completar la actividad, los pasos de la solución son los siguientes:
1. Elimina (o comenta) la lógica existente en el componente `Form` que utiliza el Hook `useState()` para la gestión del estado.
2. Agrega una función reductora que maneje dos acciones (correo electrónico cambiado y contraseña cambiada) y que también devuelva un valor predeterminado.
3. Actualiza el objeto de estado según el tipo de acción despachada (y la carga útil / *payload*, si es necesario).
4. Utiliza la función reductora con el Hook `useReducer()`.
5. Despacha las acciones apropiadas (con los datos apropiados) en el componente `Form`.
6. Utiliza el valor del estado donde sea necesario.

La interfaz de usuario debe ser la misma que la que se muestra en la Figura 11.7 una vez que hayas terminado la actividad. Asegúrate de que la interfaz de usuario funcione exactamente como lo hacía antes de implementar las funciones de contexto de React.

> [!NOTE]
> Todos los archivos de código utilizados para esta actividad, y la solución, se pueden encontrar en [https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-2](https://github.com/mschwarzmueller/book-react-key-concepts-e2/tree/11-complex-state/activities/practice-2).
