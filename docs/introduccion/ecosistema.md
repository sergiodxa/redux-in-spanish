# Ecosistema
Redux es una librería pequeña, pero su API fue elegida cuidadosamente para permitir un ecosistema de herramientas y extensiones.

Para una lista completa de todo lo relacionado a Redux, recomendamos [Awesome Redux](https://github.com/xgrommx/awesome-redux). Contiene ejemplos, boilerplates, middlewares, librerías utilitarias y más.

En esta página solo mostramos algunas de ellas que los colaboradores de Redux han seleccionado personalmente ¡No dejes que eso te desaliente a probar el resto! El ecosistema esta creciendo muy rápido, y tenemos un tiempo limitado para ver y probar todo. Considera estas como "elegidas por el staff", y no dudes en enviar un PR si haces algo increible con Redux.

## Aprendiendo Redux (en español)
- [Introducción a Redux.js](https://medium.com/react-redux/introducci%C3%B3n-a-redux-js-8bdf4fe0751e) - Introducción a conceptos de Redux.js
- [Combinando React.js y Redux.js](https://medium.com/react-redux/combinando-react-js-y-redux-js-7b45a9dc39ac) - Explicación de como usar conjuntamente estas dos tecnologías.
- [Middlewares en Redux.js](https://medium.com/react-redux/middlewares-en-redux-js-88081fcd6c91) - Explicación de como hacer middlewares propios para Redux.js
- [Pruebas unitarias en Redux.js](https://medium.com/react-redux/pruebas-unitarias-en-redux-js-d7285c013123) - Ejemplos de como hacer pruebas a nuestro código de Redux.js.
- [Ruteo en aplicaciones de Redux y React.js](https://medium.com/react-redux/ruteo-en-aplicaciones-de-redux-y-react-js-d62de452bf1b) - Explicación de como manejar las rutas de una aplicación hecha con Redux y React.js.
- [Estructura de archivos Ducks para Redux.js](https://medium.com/react-redux/estructura-de-archivos-ducks-para-redux-js-36bb56a70cb3#.z7qq00d55) - Buena práctica de como organizar creadores de acciones, reducers y tipos de acciones en módulos.
- [Glosario de términos de Redux](https://medium.com/react-redux/glosario-de-términos-de-redux-c2bca005ca69) - Colección de términos usados en Redux junto a su explicación.

## Aprendiendo Redux (en inglés)
### Vídeos
- [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) - Aprende los conceptos básicos de Redux directamente de su creador (30 vídeos gratuitos)

### Aplicaciones de ejemplo
- [SoundRedux](https://github.com/andrewngu/sound-redux) - Un cliente de SoundCloud hecho con Redux
- [Shopping Cart (Flux Comparison)](https://github.com/voronianski/flux-comparison/tree/master/redux) - Un ejemplo de carrito de completo de [Flux Comparison](https://github.com/voronianski/flux-comparison)

### Tutoriales y artículos
- [Redux Tutorial](https://github.com/happypoulp/redux-tutorial) — Aprende a usar Redux paso a paso
- [Redux Egghead Course Notes](https://github.com/tayiorbeii/egghead.io_redux_course_notes) — Notas del [curso en vídeo de Egghead](https://egghead.io/series/getting-started-with-redux)
- [What the Flux?! Let's Redux.](https://blog.andyet.com/2015/08/06/what-the-flux-lets-redux) — Una introducción a Redux
- [A cartoon intro to Redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6) — Una explicación visual del flujo de datos de Redux
- [Understanding Redux](http://www.youhavetolearncomputers.com/blog/2015/9/15/a-conceptual-overview-of-redux-or-how-i-fell-in-love-with-a-javascript-state-container) — Aprende los conceptos básicos de Redux
- [Handcrafting an Isomorphic Redux Application (With Love)](https://medium.com/@bananaoomarang/handcrafting-an-isomorphic-redux-application-with-love-40ada4468af4) — Una guía para crear aplicaciones universales con rutas y data fetching
- [Full-Stack Redux Tutorial](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html) — Una guía completa de desarrollo TDD con Redux, React e Immutable
- [Understanding Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a#.l033pyr02) — Una guía en profundidad de como implementar middlewares de Redux
- [A Simple Way to Route with Redux](http://jlongster.com/A-Simple-Way-to-Route-with-Redux) — Una introducción a Redux Simple Router

### Charlas
- [Live React: Hot Reloading and Time Travel ](http://youtube.com/watch?v=xsSnOQynTHs) — Ve como las restricciones impuestas por Redux hacen hot reloading con time travel fácil
- [Cleaning the Tar: Using React within the Firefox Developer Tools](https://www.youtube.com/watch?v=qUlRpybs7_c) — Aprender como gradualmente migrar una aplicación MVC existente a Redux
- [Redux: Simplifying Application State](https://www.youtube.com/watch?v=okdC5gcD-dM) — Una introducción a la arquitectura de Redux

## Usando Redux
### Conexiones
- [react-redux](https://github.com/gaearon/react-redux) — React
- [ng-redux](https://github.com/wbuchwalter/ng-redux) — Angular
- [ng2-redux](https://github.com/wbuchwalter/ng2-redux) — Angular 2
- [backbone-redux](https://github.com/redbooth/backbone-redux) — Backbone
- [redux-falcor](https://github.com/ekosz/redux-falcor) — Falcor
- [deku-redux](https://github.com/troch/deku-redux) — Deku

### Middleware
- [redux-thunk](http://github.com/gaearon/redux-thunk) — La forma más fácil de trabajar con acciones asíncronas
- [redux-promise](https://github.com/acdlite/redux-promise) — Middleware de promesas compatible con [FSA](https://github.com/acdlite/flux-standard-action)
- [redux-rx](https://github.com/acdlite/redux-rx) — Utilidades de RxJS para Redux, inlcuye un middleware para Observables
- [redux-logger](https://github.com/fcomb/redux-logger) — Registra cada acción de Redux y el siguiente estado
- [redux-immutable-state-invariant](https://github.com/leoasis/redux-immutable-state-invariant) — Advierto sobre modificaciones al estado en desarrollo
- [redux-analytics](https://github.com/markdalgleish/redux-analytics) — Middleware de analiticas para Redux
- [redux-gen](https://github.com/weo-edu/redux-gen) — Middleware de generadores para Redux
- [redux-saga](https://github.com/yelouafi/redux-saga) — Un modelo alternativo para efectos secundarios en aplicaciones de Redux

### Ruteo
- [react-router-redux](https://github.com/rackt/react-router-redux) — Manten React Router y Redux sincronizados fácilmente
- [redux-router](https://github.com/acdlite/redux-router) — Conecta Redux con React Router

### Componentes
- [redux-form](https://github.com/erikras/redux-form) — Mantén el estado de los formularios de React en Redux
- [react-redux-form](https://github.com/davidkpiano/react-redux-form) — Crea formularios facilmente en React con Redux

### Potenciadores
- [redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) — Customiza batching y debouncing a los suscriptos al store
- [redux-history-transitions](https://github.com/johanneslumpe/redux-history-transitions) — Transiciones basadas en History en acciones arbitrarias
- [redux-optimist](https://github.com/ForbesLindesay/redux-optimist) — Aplicac acciones que luego pueden ser revertidas
- [redux-undo](https://github.com/omnidan/redux-undo) — Historial de acciones y deshacer/rehacer sin esfuerzo
- [redux-ignore](https://github.com/omnidan/redux-ignore) — Ignora acciones de Redux
- [redux-recycle](https://github.com/omnidan/redux-recycle) — Reinicia el estado de Redux en ciertas acciones
- [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) — Despacha varias acciones con una sola notificación a los suscriptos
- [redux-search](https://github.com/treasure-data/redux-search) — Automáticamente indexa recursos en un web worker y busca en ellos sin bloquear la aplicación
- [redux-electron-store](https://github.com/samiskin/redux-electron-store) — Sincroniza el store de Redux entre varios procesos de Electron
- [redux-loop](https://github.com/raisemarketplace/redux-loop) — Ejecuta acciones de forma secuencial retornandolas desde tus reducers

### Utilidades
- [reselect](https://github.com/faassen/reselect) — Selector de datos derivados eficiente inspirado por NuclearJS
- [normalizr](https://github.com/gaearon/normalizr) — Normaliza respuestas de API anidades para consumirlas facilmente desde un reducer
- [redux-actions](https://github.com/acdlite/redux-actions) — Disminuye el boilerplate al escribir reducers y creadores de acciones
- [redux-act](https://github.com/pauldijou/redux-act) — Una librería opinionada para crear acciones y reducers
- [redux-transducers](https://github.com/acdlite/redux-transducers) — Utilidades de transductores para Redux
- [redux-immutable](https://github.com/gajus/redux-immutable) — Usado para crear un equivalente a la funcíon `combineReducers` que funcione con un estado de Immutable.js
- [redux-tcomb](https://github.com/gcanti/redux-tcomb) — Estado y acciones immutable y tipadas
- [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) — Simula un store de Redux para probar tu aplicación

### Herramientas de desarrollo
- [Redux DevTools](http://github.com/gaearon/redux-devtools) — Un logger de acciones con una UI para time travel, hot reloading y manejo de errores para reducers, [mostrada por primera vez en React Europe](https://www.youtube.com/watch?v=xsSnOQynTHs)
- [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension) — Una extensión de Chrome que envuelve las Redux DevTools y provee funcionalidades adicionales

### Convenciones de la Comunidad
- [Flux Standard Action](https://github.com/acdlite/flux-standard-action) — Un estándar de acciones de Flux amigable con humanos
- [Canonical Reducer Composition](https://github.com/gajus/canonical-reducer-composition) — Un estándar opinionado para combinar reducers anidados
- [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux) — Una propuesta para empaquetar reducers, acciones y sus tipos

### Traducciones
- [Redux](http://redux.js.org/) — Inglés
- [中文文档](http://camsong.github.io/redux-in-chinese/) — Chino
- [繁體中文文件](https://github.com/chentsulin/redux) — Chino Trandicional
- [Redux in Russian](https://github.com/rajdee/redux-in-russian) — Ruso

## Más
[Awesome Redux](https://github.com/xgrommx/awesome-redux) es una lista más completa de repositorios relacionados con Redux.
