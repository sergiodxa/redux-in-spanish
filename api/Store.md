# Store

El store contiene todo el [árbol de estado](../glosario.md#estado) de tu aplicación.
La única forma de cambiar el estado que contiene es despachando una [acción](../glosario.md#acción].

El store no es una clase. Es solo un objeto con unos pócos métodos.
Para crearlo, pasa tu principal [función reductora](../glosario.md#reducer) a [`createStore`](./create-store.md).

>##### Una nota para usuarios de Flux

> Si vienes de Flux, hay una importante diferencias que necesitas entender.
Redux no tiene un Dispatcher o soporta múltiples stores. **En cambio, hay un único store con una única [función reductora](../glosario.md#reducer).** Mientras tu aplicación crece, en vez de añadir stores, puedes dividir tu reducer en varios reducer independientes que operan en diferentes partes del árbolde estado. Puedes usar función como [`combineReducers`](./combine-reducers.md) para combinarlo. Esto es similar a como hay un único componente raíz en aplicaciones de React, pero esta compuesto de muchos componentes pequeños.

### Métodos del Store(#metodos-store)

- [`getState()`](#getState)
- [`dispatch(action)`](#dispatch)
- [`subscribe(listener)`](#subscribe)
- [`replaceReducer(nextReducer)`](#replaceReducer)

## Métodos del Store

### <a id='getState'></a>[`getState()`](#getState)

Regresa el actual árbol de estado de tu aplicación.
Es igual al último valor regresado por los reducers del store.

#### Regresa

*(any)*: El actual árbol de estado de tu aplicación.

---

### <a id='dispatch'></a>[`dispatch(action)`](#dispatch)

Despacha una acciǿn. Esta es la única forma de realizar un cambio de estado.

La función función reductora es ejecutada por el resultado de [`getState()`](#getState) y el `action` indicado de forma síncrona. El valor devuelto es considerado el siguiente estado. Va a ser devuelto por [`getState()`](#getState) desde ahora, y las funciones escuchando los cambios van a ser inmediatamente notificadas.

>##### Una nota para usuarios de Flux
>Si intentas llamar a `dispatch` desde dentro de un [reducer](../glosario.md#reducer), va a tirar un error diciendo "Los reducers no deben despachar acciones." Esto es similar al error de Flux de "No se puede despachar en medio de un despacheo", pero no causa los problemas asociados con este. En Flux, despachar esta prohibido mientras los Store están manejando las acciones y emitiendo las actualizaciones. Esto es una lástima porque hace imposible despachar acciones desde el ciclo de vida del componente u otras partes benignas.

>En Redux, los suscriptores se ejecutan luego de que el reducer principal devuelve el nuevo estado, así *quizás* quieras despachar dentro de ellos. Solo no puedes despachar dentro de reducers porque no pueden tener efectos secundarios. Si quieres causar un efecto secundario en respuesta a una acción, el mejor lugar para hacer eso es [creadores de acciones](../glosario.md#creador-de-acciones) asíncronas.

#### Argumentos

1. `action` (*Objeto*<sup>†</sup>): Un objeto plano describiendo las cambios necesarios para tu aplicación. Las acciones son la única forma de enviar datos al store, así que cualquier dato, ya sea de eventos de la UI, callbacks de peticiones, o cualquier otra fuente como WebSocket va a necesitar eventualmente ser despachada como acciones. Las acciones deben tener un campo `type` que indique el tipo de acción que se esta realizando. Los tipos pueden ser definidos como constantes e importados desde otros módulos. Es mejor usar strings para el `type` en vez de [Symbols](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Symbol) porque los strings son serializables. Aparte de `type`, la estructura de los objetos de acciones son controlador por tí. Si estás interesado, revisa [Flux Standard Action](https://github.com/acdlite/flux-standard-action) para recomendaciones de como armas acciones.

#### Regresa

(Objeto<sup>†</sup>): La acción despachada (revisa las notas).

#### Notas

<sup>†</sup> La implementación "vainilla" del store que obtienes al llamar [`createStore`](./createStore.md) solo soporta objetos planos como acciones y las maneja inmediatamento en los reducers.

De todas formas, si envuelves [`createStore`](./create-store.md) con [`applyMiddleware`](./apply-middleware.md), el middleware puede interpretar acciones de otra forma, y proveer soporta para despachar [acciones asíncronas](../glosario.md#acción-asíncrona). Las acciones asíncronas normalmente son Promises, Observables, o thunks.

Los middlewares son creador por la comunidad y no vienen incluidos en Redux por defecto. Necesitas explícitamente instalar paquetes como [redux-thunk](https://github.com/gaearon/redux-thunk) o [redux-promise](https://github.com/acdlite/redux-promise) para usarlos. Probablemente también crees tus propios middlewares.

Para aprender como describir llamadas asíncronas a un API, leer el estado actual dentro de creadores de acciones o realizar efectos secundarios, mira los ejemplos de [`applyMiddleware`](./applyMiddleware.md).

#### Ejemplo

```js
import { createStore } from 'redux'
let store = createStore(todos, [ 'Use Redux' ])

function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

store.dispatch(addTodo('Read the docs'))
store.dispatch(addTodo('Read about the middleware'))
```

---

### <a id='subscribe'></a>[`subscribe(listener)`](#subscribe)

Agregar una función que escucha los cambios. Va a ser ejecutada cada vez que una acción es despachada y algunas partes del árbol de estado puedan potencialmente haber cambiado. Probablemente quieras llamar a [`getState()`](#getState) para leer el árbol de estado actual dentro del callback.

Probablemente quieras llamar a [`dispatch()`](#dispatch) desde el callback, siguiente las siguientes advertencias:

1. Las suscripciones son ejecutadas justo después de cada llamada de [`dispatch()`](#dispatch). Si te suscribes o desuscribes mientras un listener esta siendo invocado, no va a tener ningún efecto en el [`dispatch()`](#dispatch) actualmente en progreso. De todas formas, la siguiente llamada a [`dispatch()`](#dispatch), ya sea anidada o no, va a usar la más reciente versión de la lista de suscripciones.

2. Los listeners no deben esperar ver todos los cambios de estado, ya que el estado puede haber sido actualizado multiples veces durante los [`dispatch()`](#dispatch) anidados antes de que los listeners sean llamados. De todas formas, es seguro que todos los listeners registrados antes de que [`dispatch()`](#dispatch) sea ejecutado van a ser llamados con el último estado existente en ese momento.

Es una API de bajo nivel. Comunmente, en vez de usarla directamente, vas a usar conexiones para React (u otros). Si sientes que necesitas que el callback sea invocado con el estado actual, probablemente quieras [convertir el store en un Obersavle o escribir un `observeStore` personalizado](https://github.com/reactjs/redux/issues/303#issuecomment-125184409)

Para desuscribir un listener invoca la función devuetla por `subscribe`.
To unsubscribe the change listener, invoke the function returned by `subscribe`.

#### Arguments

1. `listener` (*Función*): El callback que va a ser invocado cada vez que una acción es despachada y el árbol de estado puede haber cambiado. Probablemente quieras usar [`getState()`](#getState) dentro de este callback para obtener el árbol de estado actual. 

##### Regresa

(*Funciǿn*): Una función para desuscribir el listener.

##### Ejemplo

```js
function select(state) {
  return state.some.deep.property
}

let currentValue
function handleChange() {
  let previousValue = currentValue
  currentValue = select(store.getState())
  
  if (previousValue !== currentValue) {
    console.log('Some deep nested property changed from', previousValue, 'to', currentValue)
  }
}

let unsubscribe = store.subscribe(handleChange)
handleChange()
```

---

### <a id='replaceReducer'></a>[`replaceReducer(nextReducer)`](#replaceReducer)

Reemplaza el reducer actualmente usado en el store para calcular el estado.

Es una API avanzada. Probablemente lo necesitas si tu aplicación implementa separación de código y quieres cargas algunos reducers dinamicamente. Además necesitas esto si implementas mecanismos como hot-reloading en Redux.

#### Argumentos

1. `reducer` (*Función*) El nuevo reducer a ser usado en el store.
