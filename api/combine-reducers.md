# `combineReducers(reducers)`

Mientras tu aplicación se vuelve más compleja, vas a querer separar tus [funciones reductoras](../glosario.md#reducer) en funciones separadas, cada una manejando partes independientes del [estado](../glosario.md#estado).

La función `combineReducers` devuelve un objeto cuyos valores son diferentes funciones reductoras en una única función reductora que puedes enviar a [`createStore`](createStore.md).

El reducer resultante llama cada reducer interno, y junta sus resultados en un único objeto de estado. **La forma del objeto de estado es igual a las llaves enviadas a `reducers`**.

Consecuentemente, el objeto de estado luciría así: 

```
{
  reducer1: ...
  reducer2: ...
}
```

Puedes controlar los nombres de llaves usando diferentes llaves para los reducers pasados a los objetos. Por ejemplo, podrías usar `combineReducers({ todos: myTodosReducer, counter: myCounterReducer })` para crear un estado con la forma `{ todos, counter }`.

Una convención popular es nombrar los reducers con el pedazo de estado que controlan, así puedes usar forma abreviada de definir propiedades: `combineReducers({ counter, todos })`. Esto es lo mismo que hacer `combineReducers({ counter: counter, todos: todos })`.

> ##### Una nota para usuarios de Flux

> Esta función te ayuda a organizar tus reducers para que manejen su propia parte del estado, similar a si tuvieras diferentes Stores de Flux para manejar diferentes estados. Con Redux, hay un solo store, pero `combineReducers` te ayuda a mantener la misma lógica de separación entre reducers.

#### Argumentos

1. `reducers` (*Objeto*): Un objeto cuyos valores corresponden a diferentes funciones reductoras que necesitas combinar en uno solo. Revisa las notas debajo para ver algunas reglas que cada reducer debe seguir.

> La primers documentación sugería usar la sintaxis de ES `import * as reducers` para obtener el objeto reducer. Esto fue una fuente de confusión, razón por la cual ahora recomendados exportar un único reducer obtenido usando `combineReducers()` en `reducers/index.js`. Se incluye un ejemplo debajo.

#### Regresa

(*Función*): Un reducer que invoca cada reducer dentro del objeto `reducers`, y arma el objeto de estado con la misma forma.

#### Notas

Esta función es medianamente opinionada y este hecha para evitar algunos error de principiantes. Es por eso que te fuerza a seguir ciertas reglas que no tendrías que seguir si escribieses tu reducer manualmente.

Cualquier reducer enviado a `combineReducers` debe satisfaces las siguientes reglas:

* Si no reconoce una acción, debe regresarel `state` recibido como primer argumento.

* Nunca debe devolver `undefined`. Es muy fácil hacer eso por error con un `return`, por eso `combineReducers` tira un error en vez de dejar que el error se manifieste en otra parte.

* Si el `state` proporcionado es `undefined`, debe devolver el estado inicial para ese reducer específico.  De acuerdo con las reglas anteriores, el estado inicial tampoco puede ser `undefined`. Es fácil especificarlo usando la sintaxis de parámetros opcionales de ES^, pero además debes específicamente verificar que el primer argumento no sea `undefined`.

Mientras que `combineReducers` intenta validar tus reducers conforme a algunas de estas reglas, debes recordarlas y hacer lo posible para seguirlas.

#### Ejemplo

#### `reducers/todos.js`

```js
export default function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([ action.text ])
    default:
      return state
  }
}
```

#### `reducers/counter.js`

```js
export default function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

#### `reducers/index.js`

```js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({
  todos,
  counter
})
```

#### `App.js`

```js
import { createStore } from 'redux'
import reducer from './reducers/index'

let store = createStore(reducer)
console.log(store.getState())
// {
//   counter: 0,
//   todos: []
// }

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
console.log(store.getState())
// {
//   counter: 0,
//   todos: [ 'Use Redux' ]
// }
```

#### Consejos

* 
* This helper is just a convenience! You can write your own `combineReducers` that [works differently](https://github.com/acdlite/reduce-reducers), or even assemble the state object from the child reducers manually and write a root reducing function explicitly, like you would write any other function.

* You may call `combineReducers` at any level of the reducer hierarchy. It doesn’t have to happen at the top. In fact you may use it again to split the child reducers that get too complicated into independent grandchildren, and so on.
