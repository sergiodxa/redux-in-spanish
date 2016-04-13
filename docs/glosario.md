# Glosario
Este es un glosario de los términos principales en Redux, junto a su tipo de dato. Los tipos están documentados usando la [notación Flow](http://flowtype.org/docs/quick-reference.html).

## Estado
```
type State = any
```
Estado (también llamado árbol de estado) es un termino general, pero en la API de Redux normalmente se refiere al valor de estado único que es manejado por el Store y devuelto por [`getState()`](./api/store.html#getState). Representa el estado de tu aplicación de Redux, normalmente es un objeto con muchas anidaciones.

Por convención, el estado a nivel superior es un objeto o algún tipo de colección llave-valor como un Map, pero técnicamente puede ser de cualquier tipo. Aun así, debes hacer tu mejor esfuerzo en mantener el estado serializable. No pongas nada dentro que no puedas fácilmente convertirlo a un JSON.

## Acción
```
type Action = object
```
Una acción es un objeto plano (POJO - Plan Old JavaScript Object) que representa una intención de modificar el estado. Las acciones son la única forma en que los datos llegan al store. Cualquier dato, ya sean eventos de UI, callbacks de red, u otros recursos como WebSockets eventualmente van a ser despachados como acciones.

Las acciones deben tener un campo `type` que indica el tipo de acción a realizar. Los tipos pueden ser definidos como constantes e importados desde otro módulo Es mejor usar strings como tipos en vez de [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) ya que los strings son serializables.

Aparte del `type`, la estructura de una acción depende de vos. Si estás interesado, revisa [Flux Standard Action](https://github.com/acdlite/flux-standard-action) para recomendaciones de como deberías estar estructurado una acción.

Revisa [acción asíncrona](#acción-asíncrona) debajo.

## Reducer
```
type Reducer<S, A> = (state: S, action: A) => S
```
Un *reducer* (también llamado *función reductora*) es una función que acepta una acumulación y una valor y devuelve una nueva acumulación. Son usados para reducir una colección de valores a un único valor.

Los reducers no son únicos de Redux — son un concepto principal de la programación funcional. Incluso muchos lenguajes no funcionales, como JavaScript, tienen una API para reducción. En JavaScript, es [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce).

En Redux, el valor acumulado es el árbol de estado, y los valores que están siendo acumulados son acciones. Los reducers calculan el nuevo estado en base al anterior estado y la acción. Deben ser *funciones puras* — funciones que devuelven el mismo valor dados los mismos argumentos. Deben estar libres de efectos secundarios. Esto es lo que permite características increíbles como hot reloading y time travel.

Los reducers son el concepto más importante en Redux.

*No hagas peticiones a APIs en los reducers.*

## Función despachadora
```
type BaseDispatch = (a: Action) => Action
type Dispatch = (a: Action | AsyncAction) => any
```
La *función despachadora* (o simplemente *función dispatch*) es una función que acepta una acción o una [acción asíncrona](#acción-asíncrona); entonces puede o no despachar una o más acciones al store.

Debemos distinguir entre una función despachadora en general y la función base [`dispatch`](./api/Store.md#dispatch) provista por la instancia del store sin ningún middleware.

La función base dispatch *siempre* envía síncronamente acciones al reducer del store, junto al estado anterior devuelto por el store, para calcular el nuevo estado. Espera que las acciones sean objetos planos listos para ser consumidos por el reducer.

Los [middlewares](#middleware) envuelven la función dispatch base. Le permiten a la función dispatch manejar [acciones asíncronas](#acción-asíncrona) además de las acciones. Un middleware puede transformar, retrasar, ignorar o interpretar de cualquier forma una acción o acción asíncrona antes de pasarla al siguiente middleware. Lea más abajo para más información.

## Creador de acciones
```
type ActionCreator = (...args: any) => Action | AsyncAction
```
Un *creador de acciones* es, simplemente, una función que devuelve una acción. No confunda los dos términos — otra vez, una acción es un pedazo de información, y los creadores de acciones son fabricas que crean esas acciones.

Llamar un creador de acciones solo produce una acción, no la despacha. Necesitas llama al método [`dispatch`](./api/Store.md#dispatch) del store para causar una modificación. Algunas veces decimos *creador de acciones conectado*, esto es una función que ejecuta un creador de acciones e inmediatamente despacha el resultado a una instancia del store específica.

Si un creador de acciones necesita leer el estado actual, hacer una llamada al API, o causar un efecto secundario, como una transición de rutas, debe retornas una [acción asíncrona](#acción-asíncrona) en vez de una acción.

## Acción asíncrona
```
type AsyncAction = any
```
Una *acción asíncrona* es un valor que es enviado a una función despachadora, pero todavía no esta listo para ser consumido por el reducer. Debe ser transformada por un [middleware](#middleware) en una acción (o una serie de acciones) antes de ser enviada a la función [`dispatch()`](./api/Store.md#dispatch) base. Las acciones asíncronas pueden ser de diferentes tipos, dependiendo del middleware que uses. Normalmente son primitivos asíncronos como una promesa o un thunk, que no son enviados inmediatamente a un reducer, pero despachan una acción cuando una operación se completa.

## Middleware
```
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```
Un middleware es una función de orden superior que toma una [función despachadora](#función-despachadora) y devuelve una nueva función despachadora. A menudo convierten [acciones asíncronas](#acción-asíncrona) en acciones.

Los middlewares son combinables usando funciones. Son útiles para registrar acciones, realizar efectos secundarios como ruteo, o convertir una llamada asíncrona a una API en una serie de acciones síncronas.

Revisa [`applyMiddleware(...middlewares)`](./api/apply-middleware.md) para más detalles de los middlewares.

## Store
```
type Store = {
  dispatch: Dispatch
  getState: () => State
  subscribe: (listener: () => void) => () => void
  replaceReducer: (reducer: Reducer) => void
}
```
Un store es un objeto que mantiene el árbol de estado de la aplicación.

Solo debe haber un único store en una aplicación de Redux, ya que la composición ocurre en los reducers.

- [dispatch(action)](./api/Store.md#dispatch) es la función dispatch base descrita arriba.
- [getState()](./api/Store.md#getState) devuelve el estado actual de la aplicación.
- [subscribe(listener)](./api/Store.md#subscribe) registra una función para que se ejecute en cada cambio de estado.
- [replaceReducer(nextReducer)](./api/Store.md#replaceReducer) puede ser usada para implementar hot reloading y code splitting. Normalmente no la vas a usar.

Revisa la [referencia API del Store](./api/Store.md) completa para más detalles.

## Creador de store
```
type StoreCreator = (reducer: Reducer, initialState: ?State) => Store
```
Un creador de store es una función que crea un store de Redux. Al igual que la función despachante, debemos separar un creador de stores base, [`createStore(reducer, initialState)`](api/create-store.md) exportado por Redux, por los creadores de store devueltos por los potenciadores de store.

## Potenciador de store
```
type StoreEnhancer = (next: StoreCreator) => StoreCreator
```
Un potenciador de store es una función de orden superior que toma un creador de store y devuelve una versión potenciada del creador de store. Es similar a un middleware ya que te permite alterar la interfaz de un store de manera combinable.

Los potenciadores de store son casi el mismo concepto que los componentes de orden superior de React, que ocasionalmente se los llamada "potenciadores de componentes".

Debido a que el store no es una instancia, sino una colección de funciones en un objeto plano, es posible crear copias fácilmente y modificarlas sin modificar el store original. Hay un ejemplo en la documentación de [`compose`](./api/compose.md) demostrándolo.

Normalmente nunca vas a escribir un potenciador de store, pero capaz uses el que provee las [herramientas de desarrollo](https://github.com/gaearon/redux-devtools). Es lo que permite que el time travel sea posible sin que la aplicación se entere de que esta ocurriendo. Curiosamente, la forma de [implementar middleware en Redux](./api/apply-middleware.md) es un potenciador de store.
