# Store

En las secciones anteriores, definimos las [acciones](acciones.md) que representan los hechos sobre "lo que pasó" y los [reductores](reducers.md) son los que actualizan el estado de acuerdo a esas acciones.

El ***Store*** es el objeto que los reúne. El *store* tiene las siguientes responsabilidades:

* Contiene el estado de la aplicación;
* Permite el acceso al estado via [`getState()`](../api/Store.md#getState);
* Permite que el estado sea actualizado via [`dispatch(action)`](../api/Store.md#dispatch);
* Registra los *listeners* via [`subscribe(listener)`](../api/Store.md#subscribe);
* Maneja la anuliación del registro de los *listeners* via el retorno de la función de [`subscribe(listener)`](../api/Store.md#subscribe).

Es importante destacar que sólo tendrás un *store* en una aplicación Redux. Cuando desees dividir la lógica para el manejo de datos, usarás [composición de reductores](Reducers.md#splitting-reducers) en lugar de muchos *stores*.

Es fácil crear una *store* si tienes un reductor. En la [sección anterior](Reducers.md), usamos [`combineReducers()`](../api/combine-Reducers.md) para combinar varios reductores en uno solo. Ahora lo vamos a importar y pasarlo a [`createStore()`](../api/create-Store.md).

```js
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```

Opcionalmente puedes especificar el estado inicial a través del segundo argumento para [`createStore()`](../api/create-Store.md). Esto es útil para hidratar el estado del cliente para que coincida con el estado de una aplicación Redux que se ejecuta en el servidor.

```js
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```

## Enviar Acciones

Ahora que hemos creado un *store*, vamos a verificar que nuestro programa funciona! Incluso sin ninguna interfaz de usuario, ya podemos verificar la lógica de actualización.

```js
import { addTodo, toggleTodo, setVisibilityFilter, VisibilityFilters } from './actions'

// Mostramos el estado inicial
console.log(store.getState())

// Cada vez que el estado cambie, lo mostramos
// Tenga en cuenta que subscribe() devuelve una función para anular el registro del listener
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// Enviamos algunas acciones
store.dispatch(addTodo('Aprender sobre acciones'))
store.dispatch(addTodo('Aprender sobre reductores'))
store.dispatch(addTodo('Aprender sobre stores'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// Anulamos el monitoreo de las actualizaciones al estado
unsubscribe()
```

Puedes ver como esto hace que el estado del *store* cambie:

<img src='http://i.imgur.com/zMMtoMz.png' width='70%'>

Hemos especificado el comportamiento de nuestra aplicación incluso antes de empezar a escribir la interfaz de usuario. No lo haremos en este tutorial, pero en este momento usted puede escribir pruebas para sus reductores y creadores de acción. No necesitarás hacer *mock* de nada porque son funciones [puras](../introduccion/tres-principios.md#changes-are-made-with-pure-functions). Invóquelas, y haga afirmaciones (assertions) sobre lo que devuelvan.

## Código Fuente

#### `index.js`

```js
import { createStore } from 'redux'
import todoApp from './reducers'

let store = createStore(todoApp)
```

## Próximos Pasos

Antes de crear la interfaz de usuario para nuestra aplicación TODO, tomaremos un desvío para ver [cómo fluyen los datos en una aplicación Redux](flujo-de-datos.md).
