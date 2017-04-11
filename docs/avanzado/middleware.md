# Middleware

Has visto los middleware en acción en el ejemplo [Async Actions](../avanzado/acciones-asincronas.md). Si has utilizado librerías de tipo *server-side* como [Express](http://expressjs.com/) y [Koa](http://koajs.com/), probablemente ya estes familiarizado con el concepto de *middleware*. En estos frameworks, el middleware es un código que se ejecuta entre el framework que recibe una solicitud, y el framework que genera una respuesta. Por ejemplo, a travé de los middleware Express o Koa puede agregar encabezados *CORS*, realizar registro de eventos, compresión y más. La mejor característica de un middleware es que se puede ejecutar en cadena. Puedes utilizar múltiples middleware independientes de terceros en un solo proyecto.

El mecanismo de middleware en Redux resuelve diferentes problemas cuando lo comparamos con los middleware en Express o Koa, pero se comporta de manera conceptualmente similar. **Proporciona un punto de extensión para terceros entre el envío de una acción y el momento en que alcanza el reductor.** La gente utiliza Redux middleware para el registro de eventos, informes de fallos, para mantener las llamadas a una API asíncrona, enrutamiento y más.

Este artículo provee una introducción a fondo para ayudarle a digerir el concepto, y [algunos ejemplos prácticos](#siete-ejemplos) para al final poder mostrar el poder de los middleware. Puede resultarle útil brincar entre ellos, ya que puedes sentirte entre aburrido e inspirado.

## Entendiendo los middleware

Mientras que los middleware se pueden utilizar para una variedad de cosas, incluyendo llamadas asíncronas a un API, es muy importante que usted entienda de dónde surgen. Le guiaremos a través del proceso de pensamiento que conduce al middleware, utilizando el registro de eventos (*logging*) y el informe de fallos como ejemplos.

### Problema: Registo de eventos (*Logging*)

Uno de los beneficios de Redux es que hace que los cambios en el estado sean predecibles y transparentes. Cada vez que se envía una acción, se calcula y se guarda el nuevo estado. El estado no puede cambiar por sí solo, sólo puede cambiar como consecuencia de una acción específica.

¿No sería bueno si registramos todas las acciones que ocurren en la aplicación, junto con el estado calculado después? Cuando algo sale mal, podemos mirar hacia atrás en nuestro registro, y averiguar qué acción corrompió el estado.

<img src='http://i.imgur.com/BjGBlES.png' width='70%'>

¿Cómo atacamos esto con Redux?

### Intento #1: Registro manual

La solución más ingenua es simplemente registrar la acción y el siguiente estado manualmente cada vez que llame a [`store.dispatch(action)`](../api/Store.md#dispatch). No es realmente una solución, sino un primer paso hacia la comprensión del problema.

> ##### Nota

> Si está utilizando [react-redux](https://github.com/gaearon/react-redux) o *bindings* similares, probablemente no tendrá acceso directo a la instancia de almacenamiento en sus componentes. Para los próximos párrafos, asuma que usted pasa el store de forma explícita.

Digamos, usted invoca lo siguiente al crear un to-do:

```js
store.dispatch(addTodo('Usar Redux'))
```

Para registrar la acción y el estado, puede cambiarlo a algo como esto:

```js
let action = addTodo('Usar Redux')

console.log('dispatching', action)
store.dispatch(action)
console.log('next state', store.getState())
```

Esto produce el efecto deseado, pero no querrás hacerlo una y otra vez.

### Intento #2: *Wrapping Dispatch*

Puede extraer el registro en una función:

```js
function dispatchAndLog(store, action) {
  console.log('dispatching', action)
  store.dispatch(action)
  console.log('next state', store.getState())
}
```

Podrás utilizarlo en cualquier parte en lugar de `store.dispatch()`:

```js
dispatchAndLog(store, addTodo('Usar Redux'))
```

Podríamos terminar esto aquí, pero no es muy conveniente importar una función especial una y otra vez.

### Intento #3: Reemplazo dinámico del envío (*Monkeypatching Dispatch*)

¿Qué sucede si reemplazamos la función `dispatch` en la instancia del store? El Redux store es un simple objeto con [unos métodos](../api/store.md), y estamos escribiendo JavaScript, así que podemos simplemente reemplazar dinámicamente (*monkeypatch*) la implementación `dispatch`:

```js
let next = store.dispatch
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

Con esto estamos más cerca de lo que queremos! No importa dónde enviemos una acción, se garantiza su registro. Hacer *monkeypatching* nunca se siente bien, pero podemos vivir con esto por ahora.

### Problema: Reporte de fallos

¿Qué pasa si queremos aplicar **más de una** transformación a `dispatch`?

Se me ocurre una transformación útil y es reportar errores de JavaScript en producción. El evento global `window.onerror` no es fiable porque no proporciona información del *stack* en algunos navegadores antiguos, lo cual es crucial para entender por qué se está produciendo un error.

¿No sería útil si, en cualquier momento que se genere un error como resultado del envío de una acción, lo podamos enviar a un servicio de informes de fallos como [Sentry](https://getsentry.com/welcome/) con el *stack* completo, la acción que causó el error y el estado actual? De esta manera es mucho más fácil reproducir el error en el ambiente de desarrollo.

Sin embargo, es importante que mantengamos el registro y los informes de fallos separados. Idealmente queremos que sean módulos diferentes, potencialmente en diferentes paquetes. De lo contrario no podemos tener un ecosistema de tales servicios públicos. (Sugerencia: vamos lentamente a lo que middleware es!)

Si el registro y el informe de fallos son utilidades independientes, podrían tener el siguiente aspecto:

```js
function patchStoreToAddLogging(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}

function patchStoreToAddCrashReporting(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action)
    } catch (err) {
      console.error('Caught an exception!', err)
      Raven.captureException(err, {
        extra: {
          action,
          state: store.getState()
        }
      })
      throw err
    }
  }
}
```

Si estas funciones se publican como módulos independientes, podemos usarlas posteriormente para parchear nuestro store:

```js
patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```

Sin embargo, esto no es agradable.

### Intento #4: Ocultando el *Monkeypatching*

*Monkeypatching* es un hack. "Reemplazar cualquier método que te gusta", ¿qué tipo de API es esa? En su lugar, vamos a averiguar su importancia. Anteriormente, nuestras funciones reemplazaron `store.dispatch`. ¿Qué pasa si en vez se *devuelve* una nueva función `dispatch`?

```js
function logger(store) {
  let next = store.dispatch

  // Anteriormente:
  // store.dispatch = function dispatchAndLog(action) {

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

Podríamos proporcionar un *helper* dentro de Redux que aplicaría el actual monkeypatching como un detalle de la implementación:

```js
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  // Transforma la función de despacho con cada middleware.
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store)
  )
}
```

Podríamos utilizarlo para aplicar múltiples middleware como este:

```js
applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])
```

Sin embargo, todavía sigue siendo *monkeypatching*.
El hecho de ocultarlo dentro de la biblioteca no altera este hecho.

### Intento #5: Eliminar el *Monkeypatching*

¿Por qué incluso sobrescribimos `dispatch`? Por supuesto, para poder llamarla más tarde, pero también hay otra razón: para que cada middleware pueda acceder (y llamar) al `store.dispatch` previamente *wrapped*:

```js
function logger(store) {
  // Debe apuntar a la función devuelta por el middleware anterior:
  let next = store.dispatch

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

¡Es esencial encadenar los middleware!

Si `applyMiddlewareByMonkeypatching` no asigna `store.dispatch` inmediatamente después de procesar el primer middleware, `store.dispatch` seguirá apuntando a la función `dispatch` original. Entonces, el segundo middleware también estará enlazado a la función `dispatch` original.

Pero también hay una forma diferente de habilitar el encadenamiento. El middleware podría aceptar la función de despacho `next()` como un parámetro en vez de leerlo desde la instancia `store`.

```js
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log('dispatching', action)
      let result = next(action)
      console.log('next state', store.getState())
      return result
    }
  }
}
```

Esto es uno de esos momentos [“we need to go deeper”](http://knowyourmeme.com/memes/we-need-to-go-deeper), por lo que puede tomar un tiempo para que esto te haga sentido. La función cascada se siente intimidante. Las funciones de flecha ES6 hacen que este [*Currying*](https://en.wikipedia.org/wiki/Currying) sea mucho más fácil a la vista:

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

**Esto es exactamente el aspecto de un middleware en Redux.**

Ahora el middleware toma la función de despacho `next()` y devuelve una función de despacho, que a su vez sirve como `next()` al middleware a la izquierda, y así sucesivamente. Aún así es útil tener acceso a algunos métodos del store como `getState()`, por lo que `store` se mantiene disponible como argumento de nivel superior.

### Intento #6: Aplicando ingenuamente el middleware

En lugar de `applyMiddlewareByMonkeypatching()`, podríamos escribir `applyMiddleware()` que primero obtiene la función final de `dispatch()` totalmente envuelta y devuelve una copia del store usando:

```js
// Warning: Implementación ingenua!
// Esto *no* es un Redux API.

function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  let dispatch = store.dispatch
  middlewares.forEach(middleware =>
    dispatch = middleware(store)(dispatch)
  )

  return Object.assign({}, store, { dispatch })
}
```

La implementación de [`applyMiddleware()`](../api/apply-middleware.md) que Redux provee es similar, pero **diferente en tres aspectos importantes**:

* Sólo expone un subconjunto de la API [store](../api/Store.md) al middleware: [`dispatch(action)`](../api/Store.md#dispatch) y [`getState()`](../api/Store.md#getState).

* Engaña un poco para asegurarse de que si llamas `store.dispatch(action)` desde tu middleware en lugar de `next(action)`, la acción recorrerá toda la cadena de middlewares de nuevo, incluyendo el middleware actual. Esto es útil para middleware asíncronos, como hemos visto [anteriormente](acciones-asincronas.md).

* Para asegurarse de que sólo se puede aplicar el middleware una vez, se trabaja en `createStore()` y no en `store`. En lugar de `(store, middlewares) => store`, su firma es `(... middlewares) => (createStore) => createStore`.

Debido a que es engorroso aplicar funciones a `createStore()` antes de usarlo, `createStore()` acepta un último argumento opcional para especificar dichas funciones.

### El enfoque final

Dado el middleware que acabamos de escribir:

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

A continuación, le indicamos cómo aplicarlo a un Redux store:

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'

let todoApp = combineReducers(reducers)
let store = createStore(
  todoApp,
  // applyMiddleware() le indica a createStore() cómo manejar el middleware
  applyMiddleware(logger, crashReporter)
)
```

¡Eso es todo! Ahora, cualquier acción enviada a la instancia del store pasará a través de `logger` y `crashReporter`:

```js
// Navegamos a través del middleware logger y crashReporter!
store.dispatch(addTodo('Usar Redux'))
```

## Siete ejemplos

Si su cabeza esta a reventar por la lectura de la sección anterior, imagine lo que fue escribirlo. Esta sección está destinada a ser una de relajación para usted y para mí, y ayudará a que sus engranajes giren.

Cada función a continuación es un middleware Redux válido. No son igualmente útiles, pero al menos son igualmente divertidos.

```js
/**
 * Registra todas las acciones y estados antes de ser enviados.
 */
const logger = store => next => action => {
  console.group(action.type)
  console.info('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  console.groupEnd(action.type)
  return result
}

/**
 * Envia reportes de errores tan pronto el estado es actualizado y los listeners
 * sean notificados.
 */
const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}

/**
 * Programa las acciones con { meta: { delay: N } } para que sean retrasadas por N 
 * milliseconds.
 * Hace que `dispatch` devuelva una función para cancelar el contador en este caso.
 */
const timeoutScheduler = store => next => action => {
  if (!action.meta || !action.meta.delay) {
    return next(action)
  }

  let timeoutId = setTimeout(
    () => next(action),
    action.meta.delay
  )

  return function cancel() {
    clearTimeout(timeoutId)
  }
}

/**
 * Programa las acciones con { meta: { raf: true } } para que sean enviadas
 * dentro del ciclo rAF.
 * Hace que `dispatch` devuelva una función para remover la acción de la fila 
 * en este caso.
 */
const rafScheduler = store => next => {
  let queuedActions = []
  let frame = null

  function loop() {
    frame = null
    try {
      if (queuedActions.length) {
        next(queuedActions.shift())
      }
    } finally {
      maybeRaf()
    }
  }

  function maybeRaf() {
    if (queuedActions.length && !frame) {
      frame = requestAnimationFrame(loop)
    }
  }

  return action => {
    if (!action.meta || !action.meta.raf) {
      return next(action)
    }

    queuedActions.push(action)
    maybeRaf()

    return function cancel() {
      queuedActions = queuedActions.filter(a => a !== action)
    }
  }
}

/**
 * Le permite enviar promesas, además de las acciones.
 * Si la promesa se resuelve, su resultado será enviado como una acción.
 * La promesa se devuelve desde `dispatch` para que quien haga la llama 
 * pueda manejar el rechazo.
 */
const vanillaPromise = store => next => action => {
  if (typeof action.then !== 'function') {
    return next(action)
  }

  return Promise.resolve(action).then(store.dispatch)
}

/**
 * Le permite enviar acciones especiales con un campo {promise}.
 *
 * Este middleware los convertirá en una sola acción al principio,
 * y una sola acción de éxito (o fracaso) cuando se resuelve `promise`.
 *
 * Por conveniencia, `dispatch` devolverá la promesa para que quien 
 * haga la llama pueda esperar.
 */
const readyStatePromise = store => next => action => {
  if (!action.promise) {
    return next(action)
  }

  function makeAction(ready, data) {
    let newAction = Object.assign({}, action, { ready }, data)
    delete newAction.promise
    return newAction
  }

  next(makeAction(false))
  return action.promise.then(
    result => next(makeAction(true, { result })),
    error => next(makeAction(true, { error }))
  )
}

/**
 * Le permite enviar una función en lugar de una acción.
 * Esta función recibirá `dispatch` y `getState` como argumentos.
 *
 * Útil para las salidas tempranas (condiciones de `getState()`), así como
 * para el flujo de control asíncrono (puede ser `dispatch()` algo más).
 *
 * `Dispatch` devolverá el valor devuelto de la función despachada.
 */
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action)


// You can use all of them! (It doesn't mean you should.)
let todoApp = combineReducers(reducers)
let store = createStore(
  todoApp,
  applyMiddleware(
    rafScheduler,
    timeoutScheduler,
    thunk,
    vanillaPromise,
    readyStatePromise,
    logger,
    crashReporter
  )
)
```
