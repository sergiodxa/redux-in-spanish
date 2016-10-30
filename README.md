# <a href='http://es.redux.js.org'><img src='https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67' height='60'></a>

Redux es un contenedor predecible del estado de aplicaciones JavaScript.

Te ayuda a escribir aplicaciones que se comportan de manera consistente, corren en distintos ambientes (cliente, servidor y nativo), y son fáciles de probar. Además de eso, provee una gran experiencia de desarrollo, gracias a [edición en vivo combinado con un depurador sobre una línea de tiempo](https://github.com/gaearon/redux-devtools).

Puedes usar Redux combinado con [React](https://facebook.github.io/react/), o cual cualquier otra librería de vistas. Es muy pequeño (2kB) y no tiene dependencias.


> Aprende Redux con su creador (en inglés):
> [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) (30 vídeos gratuitos)

## Testimonios
> [“Love what you’re doing with Redux”](https://twitter.com/jingc/status/616608251463909376)
> Jing Chen, creador de Flux

> [“I asked for comments on Redux in FB's internal JS discussion group, and it was universally praised. Really awesome work.”
Bill Fisher, author of Flux document](https://twitter.com/fisherwebdev/status/616286955693682688)
> Bill Fisher, autor de la documentación de Flux

> [“It's cool that you are inventing a better Flux by not doing Flux at all.”](https://twitter.com/andrestaltz/status/616271392930201604)
> André Staltz, creador de Cycle

## Experiencia de desarrollo
Escribí Redux mientras trabajaba en mi charla de React Europe llamada ["Hot Reloading with Time Travel"](https://www.youtube.com/watch?v=xsSnOQynTHs). Mi meta era crear una librería de manejo de estado con una API mínima, pero completamente predecible, de forma que fuese posible implementar logging, hot reloading, time travel, aplicaciones universales/isomórficas, grabar y repetir.

## Influencias
Redux evoluciona las ideas de [Flux](http://facebook.github.io/flux/), pero evitando su complejidad tomando cosas de [Elm](https://github.com/evancz/elm-architecture-tutorial/).

Ya sea que los hayas usado o no, solo toma unos minutos para empezar a usar Redux.

## Instalación
Para instalar la versión estable:
```bash
npm i -S redux
```
Normalmente también vas a querer usar la [conexión a React](https://github.com/rackt/react-redux) y las [herramientas de desarrollo](https://github.com/gaearon/redux-devtools).
```bash
npm i -S react-redux
npm i -D redux-devtools
```
Esto asumiendo que estas usando [npm](https://www.npmjs.com/) como administrador de paquetes con un empaquetador de módulos como [Webpack](http://webpack.github.io/) o [Browserify](http://browserify.org/) para usar [módulos de CommonJS](http://webpack.github.io/docs/commonjs.html).

Si todavía no usas [npm](https://www.npmjs.com/) o algún empaquetador de módulos moderno, capaz prefiera el paquete en [UMD](https://github.com/umdjs/umd) que define `Redux` como un objeto global, puedes usar una desde [cdnjs](https://cdnjs.com/libraries/redux). *No* recomendamos este enfoque para ninguna aplicación seria, ya que la mayoría de las librerías complementarias a Redux está solo disponibles en [npm](https://www.npmjs.com/).

## El Gist
Todo el estado de tu aplicación esta almacenado en un único árbol dentro de un único *store*.
La única forma de cambiar el árbol de estado es emitiendo una *acción*, un objeto describiendo que ocurrió.

Para especificar como las acciones transforman el árbol de estado, usas *reducers* puros.

¡Eso es todo!

```javascript
import { createStore } from 'redux';

/**
 * Esto es un reducer, una función pura con el formato (state, action) => newState.
 * Describe como una acción transforma el estado en el nuevo estado.
 *
 * La forma del estado depende de tí: puede ser un primitivo, un array, un objeto, o incluso una estructura de datos de Immutable.js. La única parte importante es que no debes modificar el objeto del estado, en vez de eso devolver una nuevo objeto si el estado cambió.
 *
 * En este ejemplo, usamos `switch` y strings, pero puedes usar cualquier forma que desees si tiene sentido para tu proyecto.
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// Creamos un store de Redux almacenando el estado de la aplicación.
// Su API es { subscribe, dispatch, getState }.
let store = createStore(counter);

// Puedes suscribirte manualmente a los cambios, o conectar tu vista directamente
store.subscribe(() => {
  console.log(store.getState())
});

// La única forma de modificar el estado interno es despachando acciones.
// Las acciones pueden ser serializadas, registradas o almacenadas luego para volver a ejecutarlas.
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

En vez de modificar el estado directamente, especificas las modificaciones que quieras que ocurren con objetos planos llamados *acciones*. Entonces escribes una función especial llamada *reducer* que decide como cada acción transforma el estado de la aplicación.

Si vienes de Flux, hay una única diferencia importante que necesitas entender. Redux no tiene Dispatcher o soporta múltiples stores. En cambio, hay un único store con una única función reductora. Cuando tu aplicación crezca, en vez de agregar más stores, divides tu reducer principal en varios reducers pequeños que operan de forma independiente en distintas partes del árbol de estado. Esto es exactamente como si solo hubiese un componente principal en una aplicación de React, pero esta compuesta de muchos componentes pequeños.

Esta arquitectura puede parecer una exageración para una aplicación de contador, pero los hermoso de este patrón es los bien que escala en aplicaciones grandes y complejas. También permite herramientas de desarrollo muy poderosas, ya que es posible registrar cada modificación que las acciones causan. Podrías registrar la sesión de un usuario y reproducirlas simplemente ejecutando las mismas acciones.

## Aprende Redux con su Creador (en inglés)
[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) es un curso de 30 vídeos explicados por Dan Abramov, autor de Redux. Esta diseñado para complementar las partes "Básicas" de la documentación mientras incluye ideas adicionales sobre inmutabilidad, pruebas, buenas prácticas de Redux, y como usar Redux con React.**Este curso es gratuito y siempre lo va a ser**.

> [“Great course on egghead.io by @dan_abramov - instead of just showing you how to use #redux, it also shows how and why redux was built!”](https://twitter.com/sandrinodm/status/670548531422326785)
> Sandrino Di Mattia

> [“Plowing through @dan_abramov 'Getting Started with Redux' - its amazing how much simpler concepts get with video.”](https://twitter.com/chrisdhanaraj/status/670328025553219584)
> Chris Dhanaraj

> [“This video series on Redux by @dan_abramov on @eggheadio is spectacular!”](https://twitter.com/eddiezane/status/670333133242408960)
> Eddie Zaneski

> [“Come for the name hype. Stay for the rock solid fundamentals. (Thanks, and great job @dan_abramov and @eggheadio!)”](https://twitter.com/danott/status/669909126554607617)
> Dan

> [“This series of videos on Redux by @dan_abramov is repeatedly blowing my mind - gunna do some serious refactoring”](https://twitter.com/gelatindesign/status/669658358643892224)
> Laurence Roberts

Así que ¿Qué estas esperando?

### **[¡Mira los 30 vídeos gratuitos!](https://egghead.io/series/getting-started-with-redux)**

Sí se gusto el curso, considera soportar a Egghead [comprando una suscripción](https://egghead.io/pricing). Los suscriptos tiene acceso al código fuentes de los ejemplos en cada uno los vídeos, así como un montón de lecciones avanzados en otros temas, incluyendo JavaScript en profundidad, React, Angular, y más. Muchas [profesores de Egghead](https://egghead.io/instructors) son también autores de librerías de código abierto, así que comprando una suscripción es una buena forma de ayudarlos por todo el trabajo que hicieron.

## Documentación
- [Introducción](docs/introduccion/README.md)
- [Básico](docs/basico/README.md)
- [Avanzado](docs/avanzado/README.md)
- [Recetas](docs/recetas/README.md)
- [Solución de problemas](docs/solucion-de-problemas.md)
- [Glosario](docs/glosario.md)
- [Referencia API](docs/api/README.md)

## Ejemplos
- [Contador](https://github.com/rackt/redux/tree/master/examples/counter)
- [TodoMVC](https://github.com/rackt/redux/tree/master/examples/todomvc)
- [Todos con Deshacer](https://github.com/rackt/redux/tree/master/examples/todos-with-undo)
- [Asíncrono](https://github.com/rackt/redux/tree/master/examples/async)
- [Universal](https://github.com/rackt/redux/tree/master/examples/universal)
- [Mundo real](https://github.com/rackt/redux/tree/master/examples/real-world)
- [Carrito de compra](https://github.com/rackt/redux/tree/master/examples/shopping-cart)
- [Vista de árbol](https://github.com/rackt/redux/tree/master/examples/tree-view)

Si eres nuevo en el ecosistema de NPM y tiene problemas iniciando un proyecto, o no estas seguro de donde pegar el gist de arriba, revisa [simples-redux-example](https://github.com/jackielii/simplest-redux-example) que usa React junto a React y Browserify.

## Discusiones
Únete al canal [#redux](https://discord.gg/0ZcbPKXt5bZ6au5t) de la comunidad en Discord [Reactiflux](http://www.reactiflux.com/).

## Agradecimientos
- [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial) por una gran introducción al modelo de actualización de estado con reducers;
- [Turning the database inside-out](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) por explotarme la mente;
- [Developing ClojureScript with Figwheel](https://www.youtube.com/watch?v=j-kj2qwJa_E) por convencerme que la re-evaluaciǿn debe "simplemente funcionar";
- [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) por Hot Module Replacement;
- [Flummox](https://github.com/acdlite/flummox) por enseñarme un enfoque a Flux sin boilerplate or singletons;
- [disto](https://github.com/threepointone/disto) por la prueba de concepto de hot reloadable Stores;
- [NuclearJS](https://github.com/optimizely/nuclear-js) por probar que esta arquitectua puede tener buen rendimiento;
- [Om](https://github.com/omcljs/om) por popularizar la idea de un único Store;
- [Cycle](https://github.com/cyclejs/cycle-core) por demostrar que tan común una función puede ser la mejor herramienta;
- [React](https://github.com/facebook/react) por la innovación pragmática.

Agradecimientos especiales a [Jamie Paton](http://jdpaton.github.io/) por liberar `redux` como nombre de módulo en NPM.

## Registro de cambios
Este proyecto se adhiere al [Versionamiento Semántico](http://semver.org/).

Cada lanzamiento, junto a sus instrucciones de migración, están documentados en la página de Github de [Lanzamientos](https://github.com/rackt/redux/releases).

## Patrocinadores
El trabajo con Redux fue [financiado por la comunidad](https://www.patreon.com/reactdx).

Algunos de las compañías más destacadas que hicieron esto posible:

- [Webflow](https://github.com/webflow)
- [Chess iX](http://www.chess-ix.com/)

[Ve la lista completa de patrocinadores de Redux.](docs/PATRONS.md)

## Licencia
MIT
