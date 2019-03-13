# `applyMiddleware(...middlewares)`

Los middleware son la forma sugerida de extender Redux con funcionalidades personalizadas. Los middlewares te dejan envolver el método [`dispatch`](./Store.md#dispatch) del Store. La característica principal de los middlewares es que son combinables. Múltiples middlewares se pueden combinar juntos, donde ninguno necesita saber cual vino antes o viene después.

El uso más común de los middlewares es soportar acciones asíncronas sin demasiado código o dependiendo de librerías como [Rx](https://github.com/Reactive-Extensions/RxJS). Logra eso gracias a que te permite despachar [acciones asíncronas](../glosario.md#acción-asíncrona) además de las normales.

Por ejemplo, [redux-thunk](https://github.com/gaearon/redux-thunk) permite a los creadores de acciones invertír el control despachando funciones. Van a recibir [`dispatch`](./Store.md#dispatch) como argumento y capaz llamarlo asíncronamente. Estas funciones son llamadas *thunks*. Otro ejemplo de middleware es [redux-promise](https://github.com/acdlite/redux-promise). Este te deja despachar una [Promesa](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Promesa) como una acción asíncrona, y despachar una acción normal cuando la Promesa se resuelve.

Los middlewares no vienen incluidos en [`createStore`](./create-store.md) y no son una parte fundamental de la arquitectura de Redux, pero los consideramos suficientemente útiles para soportarlos directamente. De esta forma, hay una única forma estandarizada de extender [`dispatch`](./Store.md#dispatch) y diferentes middlewares probablemente compitan en expresividad y utilidad.

#### Argumentos

* `...middlewares` (*argumentos*): Funciones que se ajustan a la *API de middlewares* de Redux. Cada middleware recibe [`dispatch`](./Store.md#dispatch) y el [`getState`](./Store.md#getState) del [`Store`](./Store.md) como argumentos, y regresa una función. Esa función va a recibir el método para despachar el siguiente middleware, y se espera que devuelva una función que recibe `action` y llame `next(action)`. El último middleware de la cadena va a recibir el verdadero método [`dispatch`](./Store.md#dispatch) del store como parámetro `next`, terminando la cadena. Así, la forma de un middleware sería `({ getState, dispatch }) => next => action`.

#### Regresa

(*Función*) Un potenciador de store que aplican los middlewares. El potenciador de store tiene el siguiente formato `createStore => createStore`, pero es más fácil de aplicar si lo envías a [`createStore()`](./create-store.md) como el último argumento `enhancer`.

#### Ejemplo: Middleware de Logging Personalizado

```js
import { createStore, applyMiddleware } from 'redux'
import todos from './reducers'

function logger({ getState }) {
  return (next) => (action) => {
    console.log('will dispatch', action)

    // Llama al siguiente método dispatch en la cadena de middlewares
    let returnValue = next(action)

    console.log('state after dispatch', getState())

    // Este seguramente será la acción, excepto
    // que un middleware anterior la haya modificado.
    return returnValue
  }
}

let store = createStore(
  todos,
  [ 'Use Redux' ],
  applyMiddleware(logger)
)

store.dispatch({
  type: 'ADD_TODO',
  text: 'Understand the middleware'
})
// (Estas líneas son registradas por el middleware:)
// will dispatch: { type: 'ADD_TODO', text: 'Understand the middleware' }
// state after dispatch: [ 'Use Redux', 'Understand the middleware' ]
```

#### Ejemplo: Usando el Middleware Thunk para Acciones Asíncronas

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import * as reducers from './reducers'

let reducer = combineReducers(reducers)
// applyMiddleware sobrecarga createStore con middlewares:
let store = createStore(reducer, applyMiddleware(thunk))

function fetchSecretSauce() {
  return fetch('https://www.google.com/search?q=secret+sauce')
}

// Estos son los creadores normales que haz visto hasta ahora.
// Las acciones que devuelven pueden ser despachadas sin middlewares.
// De todas formas, ellos expresan "hechos" y no "flujos asíncronos".

function makeASandwich(forPerson, secretSauce) {
  return {
    type: 'MAKE_SANDWICH',
    forPerson,
    secretSauce
  }
}

function apologize(fromPerson, toPerson, error) {
  return {
    type: 'APOLOGIZE',
    fromPerson,
    toPerson,
    error
  }
}

function withdrawMoney(amount) {
  return {
    type: 'WITHDRAW',
    amount
  }
}

// Incluso sin middlewares, puedes despachar una acción:
store.dispatch(withdrawMoney(100))

// ¿Pero qué haces cuando quieres iniciar una acción asíncrona,
// como un llamado a una API, o una transición de rutas?

// Conoce a thunks.
// Un thunk es una función que devuelve una función.
// Esto es un thnk

function makeASandwichWithSecretSauce(forPerson) {

  // ¡Control invertido!
  // Devuelve una función que acepta `dispatch` así podemos despacharlas luego.
  // El middleware thunk sabe como convertír acciones asíncronas con thunks en acciones.

  return function (dispatch) {
    return fetchSecretSauce().then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize('The Sandwich Shop', forPerson, error))
    )
  }
}

// ¡El middleware thunk te deja despachar acciones asíncronas
// como si fuesen acciones!

store.dispatch(
  makeASandwichWithSecretSauce('Me')
)

// Incluso se asegura de regresar el valor que devuelve el thunk
// en el despacho, así puedo anidar promesas.

store.dispatch(
  makeASandwichWithSecretSauce('My wife')
).then(() => {
  console.log('Done!')
})

// De hecho, podría crear creadores de acciones que despachan
// acciones y acciones asíncronas desde otros creadores de acciones,
// y así crear mi propio flujo de control con promesas.

function makeSandwichesForEverybody() {
  return function (dispatch, getState) {
    if (!getState().sandwiches.isShopOpen) {

      // No necesitas devolver promesas, pero es una convención común
      // así siempre se puede llamar `.then()` como resultado.

      return Promise.resolve()
    }

    // Podemos despachar acciones tanto objetos planos como thunks,
    // lo que nos deja componer las acciones asíncronas en un único flujo.

    return dispatch(
      makeASandwichWithSecretSauce('My Grandma')
    ).then(() =>
      Promise.all([
        dispatch(makeASandwichWithSecretSauce('Me')),
        dispatch(makeASandwichWithSecretSauce('My wife'))
      ])
    ).then(() =>
      dispatch(makeASandwichWithSecretSauce('Our kids'))
    ).then(() =>
      dispatch(getState().myMoney > 42 ?
        withdrawMoney(42) :
        apologize('Me', 'The Sandwich Shop')
      )
    )
  }
}

// Esto es muy útil para renderizado en el servidor, ya que puedo esperar
// hasta que los datos estén disponibles, entonces síncronamente renderizar la app.

import { renderToString } from 'react-dom/server'

store.dispatch(
  makeSandwichesForEverybody()
).then(() =>
  response.send(renderToString(<MyApp store={store} />))
)

// También puedes despachar una acción asíncrona desde un componente
// cada vez que los props cambian para cargar los datos faltantes.

import { connect } from 'react-redux'
import { Component } from 'react'

class SandwichShop extends Component {
  componentDidMount() {
    this.props.dispatch(
      makeASandwichWithSecretSauce(this.props.forPerson)
    )
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.forPerson !== this.props.forPerson) {
      this.props.dispatch(
        makeASandwichWithSecretSauce(nextProps.forPerson)
      )
    }
  }

  render() {
    return <p>{this.props.sandwiches.join('mustard')}</p>
  }
}

export default connect(
  state => ({
    sandwiches: state.sandwiches
  })
)(SandwichShop)
```

#### Consejos

* Los middlewares solo envuelven la función [`dispatch`](./Store.md#dispatch). Técnicamente, cualquier cosa que podrías hacer un middleware, puede hacerse envolviendo manualmente la llamada a `dispatch`, pero es más fácil manejar esto en un solo lugar y definir las transformaciones de acciones a una escala de todo el proyecto.

* Si usas otros potenciadores de store además de `applyMiddleware`, asegúrate de poner `applyMiddleware` antes de ellos, porque los middlewares son potencialmente asíncronos. Por ejemplo, si debería ir antes de [redux-devtools](https://github.com/gaearon/redux-devtools) porque de otra forma las DevTools no van a ver las acciones puras emitidas por un middleware de promesas.

* Si necesitas aplicar un middleware condicionalmente, asegurate de solo importarlo cuando sea necesario:

  ```js
  let middleware = [ a, b ]
  if (process.env.NODE_ENV !== 'production') {
    let c = require('some-debug-middleware');
    let d = require('another-debug-middleware');
    middleware = [ ...middleware, c, d ];
  }

  const store = createStore(
    reducer,
    initialState,
    applyMiddleware(...middleware)
  )
  ```

  Esto hace más fácil para herramientas de empaquetado cortar los módulos no necesarios y reducir el tamaño de los paquetes.

* ¿Algunas vez te preguntantes qué es `applyMiddleware`? Debería ser un mecanismo de extensión más poderoso que los mismos middleware. En efecto, `applyMiddleware` es un ejemplo de el mecanismo más poderoso para extender Redux llamado [potenciadores de store](../glosario.md#potenciador-de-store). Es muy poco probable que alguna vez quieras escribir tu propio potenciador de store. Otro ejemplo de un potenciador de store son las [redux-devtools](https://github.com/gaearon/redux-devtools). Los middlewares son menos poderosos que un potenciador de store, pero son más fáciles de escribir.

* Los middlewares suenan mucho más complicados de lo que en realidad son. La única forma de entenderlos de verdad es ver como un middleware existente funciona, y tratar de escribir uno propio. La anidación de funciones parece intimidante, pero la mayoría de los middlewares que puedas encontrar son, de hecho, menos de 10 líneas y la anidación y combinación es lo que hace al sistema de middlewares poderosos.

* Para aplicar múltiples potenciadores de store, probablemente quieras usar [`compose()`](./compose.md).
