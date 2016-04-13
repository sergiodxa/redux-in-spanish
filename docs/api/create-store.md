# `createStore(reducer, [initialState], [enhancer])`
Crea un [store](../glosario.md#Store) de Redux que mantiene el árbol de estado de tu aplicación. Solo debe haber un único store en tu aplicación.

#### Argumentos
1. `reducer` *(Función)*: Una [función reductora](../glosario.md#Reducer) que devuelve el siguiente [árbol de estado](../glosario.md#Estado), dado el árbol de estado actual y el una [acción](../glosario.md#Acción).

2. [`initialState`] *(cualquier cosa)*: El estado inicial. Puedes opcionalmente especificarlo para hidratar la aplicación con el estado del servidor en aplicaciones universales, o restaurar una sesión anterior serializada. Si crear el `reducer` con [`combineReducers`](combine-reducers.md), este debe ser un objeto plano con la misma forma usada ahí. De otra forma, eres libre de pasar cualquier cosa que el `reducer` pueda entender.

3. [`enhancer`] *(Función)*: El potenciador del store. Puedes opcionalmente especificarlo para mejorar el store con funcionalidad de terceros como los middlewares, time travel, persistencia, etc. El único potenciador de store que viene con Redux es [`applyMiddleware()`](./apply-middleware.md).

#### Regresa

([*`Store`*](./Store.md)): Un objeto que contiene el estado actual de la aplicación. La única forma de cambiar su estado es [despachando acciones](./Store.md#dispatch). Además deberías [suscribirte](./Store.md#subscribe) a los cambios de estado para actualizar la UI.

#### Ejemplo

```js
import { createStore } from 'redux'

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([ action.text ])
    default:
      return state
  }
}

let store = createStore(todos, [ 'Use Redux' ])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
// [ 'Use Redux', 'Read the docs' ]
```

#### Consejos

* ¡No crees más de un Store en la aplicación! En cambio, usa [`combineReducers`](./combine-reducers.md) un solo reducer principal a partir de varios..

* Es tu decisión la forma del estado. Puedes usar objetos planos o algo como [Immutable](http://facebook.github.io/immutable-js/). Si no estas seguro, usa objetos planos.

* Si tu estado es un objeto plano, ¡Asegúrate de nunca modificarlo! Por ejemplo, en vez de regresar algo como `Object.assign(state, newData)` en tus reducers, regresa `Object.assign({}, state, newData)`. De esta forma no vas a sobreescribir el `state` anterior. Además puedes hacer `return { ...state, ...newData }` si habilitas la [propuesta del operador spread](../recipes/using-object-spread-operator.md).

* En aplicaciones universales que corren en el servidor, crea una instancia del store en cada petición de forma que estén aisladas. Despacha unas pocas acciones para obtener datos a la instancia del store y espera que se completen antes de renderizar la aplicación en el servidor.

* Cuando un store es creado, Redux despacha una acción falsa para que tu reducer llene el store con su estado inicial. No se supone que manejes esa acción directamente. solo recuerda que tu reducer debe devolver alguna clase de estado inicial si el estado provisto como primer argumento es `undefined`, y ya esta listo.

* Para aplicar multiples potenciadores de stores, probablemente quieras usar [`compose()`](./compose.md).

