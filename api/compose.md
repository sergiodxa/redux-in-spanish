# `compose(...functions)`

Combina funciones de derecha a izquierda.

Esta es una utilidad de programación funcional y es incluida en Redux por conveniencia.
Probablemente la quieras usar para aplicar múltiples [potenciadores de store](../glosario.md#potenciador-de-store) en serie.

#### Argumentos

1. (*argumentos*): Las funciones a combinar. Se espera que cada función acepte un único parámetro. El valor de devuelva va a ser usado como argumento a la función que este a su izquierda, y así. La única excepción es la que esté más a la derecha la cual puede recibir múltiples argumentos.

#### Regresa

(*Función*): La función final obtenido de combinas las funciones indicadas de derecha a izquierda.

#### Ejemplo

Este ejemplo demuestra como usar `compose` para potenciar el [store](./Store.md) con [`applyMiddleware`](./apply-middleware.md) y algunas pocas herramientas de desarrollo de [redux-devtools](https://github.com/gaearon/redux-devtools).

```js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import DevTools from './containers/DevTools'
import reducer from '../reducers/index'

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunk),
    DevTools.instrument()
  )
)
```

#### Consejos

* Todo lo que `compose` hace es permitirte escribir funciones transformadoras anidades fácilmente. ¡No le des mucho crédito!
