# Acciones asíncronas

En la [guía básica](../basico/README.md), hemos creado una simple aplicación de tareas pendientes (To-do). Dicho ejemplo era totalmente sincrónico. Cada vez que se envía una acción, el estado se actualiza inmediatamente.

En esta guía, crearemos una aplicación asíncrona diferente. Utilizaremos la API de Reddit para mostrar los titulares actuales de un subreddit seleccionado. ¿Cómo encaja la asincronicidad en el flujo de Redux?

## Acciones 

Cuando se hace una llama asíncrona a una API, hay dos momentos cruciales en la ejecución: el momento en el que inicias la llamada y el momento en que recibes una respuesta (o un tiempo de espera).

Cada uno de estos dos momentos suele requerir un cambio en el estado de la aplicación; Para ello, es necesario enviar acciones normales que serán procesadas de forma síncrona por los reductores. Por lo general, para cualquier solicitud de un API, desearás enviar (*dispatch*) al menos tres tipos diferentes de acciones:

* **Una acción informando a los reductores que la solicitud comenzó.**

  Los reductores pueden manejar esta acción alternando un indicador `isFetching` en el estado. De esta manera la interfaz de usuario sabe que es el momento de mostrar un indicador de cargando (*spinner*).

* **Una acción que informa a los reductores de que la solicitud finalizó correctamente.**

  Los reductores pueden manejar esta acción fusionando los nuevos datos en el estado que administran y restableciendo `isFetching`. La interfaz de usuario ocultaría el indicador de cargando y mostraría los datos obtenidos.

* **Una acción que informa a los reductores que la solicitud falló.**

  Los reductores pueden manejar esta acción restableciendo `isFetching`. Además, algunos reductores pueden querer almacenar el mensaje de error para que la interfaz de usuario pueda mostrarlo.

Puedes utilizar un campo dedicado de tipo `status` en tus acciones:

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

O puedes definir diference tipos para cada uno:

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

Elegir si deseas utilizar un solo tipo de acción o varios tipos de acciones, está a tu discreción. Es una convención que usted necesita discutir con su equipo de trabajo. Múltiples tipos dejan menos espacio para un error, pero esto no es un problema si se generan creadores de acciones y reductores con una librería auxiliar como [redux-actions](https://github.com/acdlite/redux-actions).

Cualquiera que sea la convención que elijas, mantente con ella durante toda la aplicación.

Utilizaremos tipos separados en este tutorial.

## Creadores de acciones síncronas

Comencemos por definir los varios tipos de acción síncrona y los creadores de acción que necesitamos en nuestra aplicación de ejemplo. Aquí, el usuario puede seleccionar un subreddit para ser mostrado:

#### `actions.js`

```js
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}
```

También pueden presionar un botón de "actualizar" para actualizarlo:

```js
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}
```

Estas son las acciones gobernadas por la interacción del usuario. También tendremos otro tipo de acción, regida por las solicitudes de red. Eventualmente veremos cómo enviarlas, pero por ahora, solo queremos definirlas.

Cuando llegue el momento de buscar las publicaciones de algún subreddit, enviaremos una acción `REQUEST_POSTS`:

```js
export const REQUEST_POSTS = 'REQUEST_POSTS'

function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```

Es importante que esté separado de `SELECT_SUBREDDIT` o `INVALIDATE_SUBREDDIT`. Aunque pueden ocurrir una tras otra, a medida que la aplicación se vuelve más compleja, es posible que desees obtener algunos datos independientemente de la acción del usuario (por ejemplo, para buscar de forma más rápida los subreddits más populares o actualizar los datos antiguos de vez en cuando). También es posible que desees buscar en respuesta a un cambio de ruta, por lo que no es aconsejable acoplar la busqueda a algún evento de interfaz de usuario desde el principio.

Finalmente, cuando llegue la solicitud de red, enviaremos `RECEIVE_POSTS`:

```js
export const RECEIVE_POSTS = 'RECEIVE_POSTS'

function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}
```

Esto es todo lo que necesitamos saber por ahora. El mecanismo particular para enviar estas acciones junto con las solicitudes de red se discutirá más adelante.

> ##### Nota sobre el manejo de errores

> En una aplicación real, también se desea enviar una acción que responda a una petición de error. No implementaremos el manejo de errores en este tutorial, pero en la sección [de ejemplo en el mundo real](../introduccion/ejemplos.md#real-world) se muestra uno de los posibles enfoques.

## Diseñando la apariencia del estado (*State Shape*)

Al igual que en el tutorial básico, necesitarás [diseñar la forma del estado de tu aplicación](../basico/reducers.md#designing-the-state-shape) antes de entrar en la implementación. Con el código asíncrono, hay más estados para velar, por lo que tenemos que pensarlo detalladamente.

Esta parte suele ser confusa para los principiantes, ya que no se tiene claro inmediatamente qué información describe el estado de una aplicación asíncrona y cómo organizarla en un solo árbol.

Comenzaremos con el caso de uso más común: listas. Las aplicaciones Web suelen mostrar listas de cosas. Por ejemplo, una lista de publicaciones o una lista de amigos. Tendrás que averiguar qué tipo de listas puede mostrar tu aplicación. Se desea almacenarlos por separado en el estado, porque de esta manera se pueden almacenar en caché y sólo se puede recuperar si es necesario.

La apariencia de estado para nuestra aplicación "Titulares de Reddit" podría verse así:

```js
{
  selectedSubreddit: 'frontend',
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [
        {
          id: 42,
          title: 'Confusion about Flux and Relay'
        },
        {
          id: 500,
          title: 'Creating a Simple Application Using React JS and Flux Architecture'
        }
      ]
    }
  }
}
```

Algunos detalles importantes aquí:

* Almacenamos la información de cada subreddit por separado para poder almacenar en caché cada subreddit. Cuando el usuario cambie entre ellos la segunda vez, la actualización será instantánea, y no necesitaremos refetch a menos que lo deseemos. No te preocupes por el hecho de que todos estos elementos residan en la memoria: a menos que estés tratando con decenas de miles de elementos, y su usuario rara vez cierre la pestaña, no necesitarás ningún tipo de limpieza.

* Para cada lista de elementos, debes almacenar `isFetching` para mostrar un spinner,`didInvalidate` para que puedas cambiarlo más tarde cuando los datos estén expirados, `lastUpdated` para que sepas cuándo fue buscado la última vez, y los `items` como tal. En una aplicación real, también desearás almacenar el estado de paginación como `fetchedPageCount` y `nextPageUrl`.

> ##### Nota sobre entidades anidadas

> En este ejemplo, almacenamos los elementos recibidos junto con la información de paginación. Sin embargo, este enfoque no funcionará bien si tienes entidades anidadas que se hacen referenica entre sí o si deja que el usuario edite elementos. Imagine que el usuario desea editar una publicación recuperada, pero esta publicación se duplica en varios lugares del árbol de estado. Esto sería muy difícil de implementar.

> Si tiene entidades anidadas o si deja que los usuarios editen entidades recibidas, debe mantenerlas separadas en el estado como si se tratara de una base de datos. En la información de paginación, sólo se hará referencia a ellos por sus ID. Esto le permite mantenerlos siempre actualizados. El [ejemplo del mundo real](../introduccion/ejemplos.md#real-world) muestra este enfoque, junto con [normalizr](https://github.com/paularmstrong/normalizr) para normalizar las respuestas anidadas de la API. Con este enfoque, su estado podría verse así:

>```js
> {
>   selectedSubreddit: 'frontend',
>   entities: {
>     users: {
>       2: {
>         id: 2,
>         name: 'Andrew'
>       }
>     },
>     posts: {
>       42: {
>         id: 42,
>         title: 'Confusion about Flux and Relay',
>         author: 2
>       },
>       100: {
>         id: 100,
>         title: 'Creating a Simple Application Using React JS and Flux Architecture',
>         author: 2
>       }
>     }
>   },
>   postsBySubreddit: {
>     frontend: {
>       isFetching: true,
>       didInvalidate: false,
>       items: []
>     },
>     reactjs: {
>       isFetching: false,
>       didInvalidate: false,
>       lastUpdated: 1439478405547,
>       items: [ 42, 100 ]
>     }
>   }
> }
>```

> En esta guía, no normalizaremos las entidades, pero es algo que usted debe considerar para una aplicación más dinámica.

## Manejando acciones

Antes de entrar en los detalles de las acciones de despacho junto con las solicitudes de red, escribiremos los reductores para las acciones que definimos anteriormente.

> ##### Nota sobre la composición del reductor

> Aquí, vamos a suponer que usted entiende la composición del reductor usando [`combineReducers ()`](../api/combine-reducers.md), tal como se describe en la sección [Separando Reductores](../basico/reducers.md#separando-reducers) de la [guía básica](../basico/README.md). Si no lo conoce, por favor [léalo primero](../basico/Reducers.md#separando-reducers).

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  SELECT_SUBREDDIT, INVALIDATE_SUBREDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions'

function selectedSubreddit(state = 'reactjs', action) {
  switch (action.type) {
    case SELECT_SUBREDDIT:
      return action.subreddit
    default:
      return state
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
      return Object.assign({}, state, {
        didInvalidate: true
      })
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true,
        didInvalidate: false
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        didInvalidate: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}

function postsBySubreddit(state = {}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
    case RECEIVE_POSTS:
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        [action.subreddit]: posts(state[action.subreddit], action)
      })
    default:
      return state
  }
}

const rootReducer = combineReducers({
  postsBySubreddit,
  selectedSubreddit
})

export default rootReducer
```

En este código, hay dos partes interesantes:

* Utilizamos la sintaxis de la propiedad computada ES6 para poder actualizar `state [action.subreddit]` con `Object.assign()` de una forma concisa de la siguiente manera:

  ```js
  return Object.assign({}, state, {
    [action.subreddit]: posts(state[action.subreddit], action)
  })
  ```
  lo cual es equivalente a:

  ```js
  let nextState = {}
  nextState[action.subreddit] = posts(state[action.subreddit], action)
  return Object.assign({}, state, nextState)
  ```

* Extraímos `posts(state, action)` que gestiona el estado de una lista de articíulos específicos. Esto es sólo [composición del reductor](../basico/reducers.md#separando-reducers)! Es nuestra elección cómo dividir el reductor en reductores más pequeños, y en este caso, estamos delegando elementos de actualización dentro de un objeto a un reductor `posts`. El [ejemplo del mundo real](../introduccion/ejemplos.md#real-world) va aún más allá, mostrando cómo crear una factoría de reductores para reductores de paginación parametrizados.

Recuerde que los reductores son sólo funciones, por lo que puede utilizar la composición funcional y funciones de orden superior de acuerdo a su conveniencia.

## Creadores de acciones asíncronas

Finalmente, ¿cómo usamos los creadores de acciones síncronas que [definimos anteriormente](#-Creadores-de-acciones-síncronas) junto con las solicitudes de red? La forma estándar de hacerlo con Redux es usando el [middleware Redux Thunk](https://github.com/gaearon/redux-thunk). Viene en un paquete separado llamado `redux-thunk`. Explicaremos cómo funcionan los middlewares en general [más adelante](middleware.md); Por ahora, sólo hay una cosa importante que debes saber: al usar este middleware, un creador de acciones puede devolver una función en lugar de un objeto de acción. De esta manera, el creador de acción se convierte en un [thunk](https://en.wikipedia.org/wiki/Thunk).

Cuando un creador de acciones devuelve una función, esa función será ejecutada por el middleware Redux Thunk. Esta función no necesita ser pura; Por lo tanto se permite tener efectos secundarios, incluyendo la ejecución de llamadas API asíncronas. La función también puede enviar acciones, como las acciones síncronas que definimos anteriormente.

Todavía podemos que definir estos creadores de acción de *thunk* especiales dentro de nuestro archivo `actions.js`:

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

// He aqui nuestro primer creador thunk action!
// Aunque internamente son diferentes, lo usarás como cualquier otro creador de acción:
// store.dispatch(fetchPosts('reactjs'))

export function fetchPosts(subreddit) {

  // Thunk middleware sabe como manejar funciones.
  // Pasa el método de despacho como un argumento a la función,
  // haciéndolo así capaz de despachar las acciones por sí mismo.

  return function (dispatch) {

    // Primer envío: se actualiza el estado de la aplicación para informar
    // que la llamada al API está iniciando.

    dispatch(requestPosts(subreddit))

    // La función llamada por el middleware thunk puede devolver un valor,
    // que se transmite como el valor de retorno del método de envío.

    // En este caso, devolvemos una promesa para esperar por la respuesta.
    // Esto no es requerido por middleware thunk, pero es conveniente para nosotros.

    return fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json =>

        // ¡Podemos despachas muchas veces!
        // Aquí, actualizamos el estado de la aplicación con los resultados de la llamada a la API.

        dispatch(receivePosts(subreddit, json))
      )

        // En una aplicación del mundo real, también
        // capturamos cualquier error en la llamada de red.
  }
}
```

> ##### Nota sobre `fetch`

> Utilizamos la API de `fetch`(https://developer.mozilla.org/en/docs/Web/API/Fetch_API) en los ejemplos. Es una nueva API para realizar peticiones de red que reemplaza a `XMLHttpRequest` para las necesidades más comunes. Debido a que la mayoría de los navegadores aún no lo admiten de forma nativa, le sugerimos que utilice la librearía [`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch):

>```js
>// Hacer esto en todos los archivos en los que utilice `fetch`
>import fetch from 'isomorphic-fetch'
>```

> Internamente, se utiliza [`whatwg-fetch` polyfill](https://github.com/github/fetch) en el cliente y [`node-fetch`](https://github.com/bitinn/node-fetch) en el servidor, por lo que no necesitará cambiar las llamadas de la API si cambia su aplicación a tipo [universal](https://medium.com/@mjackson/universal-javascript-4761051b7ae9).

> Tenga en cuenta que cualquier polyfill `fetch` asume que un polyfill [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) ya está presente. La forma más sencilla de asegurarse de que tiene un *polyfill Promise* es habilitar el polyfill ES6 de Babel en su punto de entrada antes de que se ejecute cualquier otro código:

>```js
>// Haga esto una vez antes de cualquier otro código en su aplicación
>import 'babel-polyfill'
>```

¿Cómo incluimos el middleware Redux Thunk en el mecanismo de envío? Utilizamos el *store enhancer* [`applyMiddleware()`](../api/applyMiddleware.md) de Redux, como se muestra a continuación:

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk'
import createLogger from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // nos permite despachar funciones
    loggerMiddleware // buen middleware que registra las acciones
  )
)

store.dispatch(selectSubreddit('reactjs'))
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
)
```

Lo bueno de los thunks es que pueden enviar resultados uno del otro:

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))
    return fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {

  // Observe que la función también recibe getState()
  // que le permite elegir qué enviar luego.

  // Esto es útil para evitar una solicitud de red si
  // un valor en caché ya está disponible.

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // Dispatch a thunk from thunk!
      return dispatch(fetchPosts(subreddit))
    } else {
      // Deje que el código que invoca la llamada
      // sepa que no hay nada que esperar.
      return Promise.resolve()
    }
  }
}
```

Esto nos permite escribir un flujo de control asíncrono más sofisticado, mientras que el código de consumo puede permanecer prácticamente igual:

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState())
)
```

> ##### Nota sobre la renderización en el servidor (*SSR*)

> Los creadores de acción asíncrona son especialmente convenientes para la renderización en el servidor. Puede crear un *store*, enviar un creador de acción asíncrona único que distribuya a otros creadores de acción asíncrona para obtener datos de toda una sección de su aplicación y que sólo se procesen una vez que se haya completado la Promesa que devuelve. Entonces su *store* ya estará hidratado con el estado que necesita antes del rendering.

[Thunk middleware](https://github.com/gaearon/redux-thunk) no es la única manera de orquestar acciones asíncronas en Redux:
- Puedes utilizar [redux-promise](https://github.com/acdlite/redux-promise) o [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) para enviar Promesas en lugar de funciones.
- Puedes utilizar [redux-observable](https://github.com/redux-observable/redux-observable) para enviar *Observables*.
- Puedes utilizar el middleware [redux-saga](https://github.com/yelouafi/redux-saga/) para construir acciones asíncronas más complejas.
- Puedes utilizar el middleware [redux-pack](https://github.com/lelandrichardson/redux-pack) para enviar acciones asíncronas basadas en Promesas.
- Incluso puede escribir un middleware personalizado para describir las llamadas a su API, como se hace en el [ejemplo del mundo real](../introduccion/ejemplos.md#real-world).

Depende de ti probar las diferentes opciones, elegir una convención que te guste y seguirla, con o sin el middleware.

## Conexión a la interfaz de usuario

La distribución de acciones asíncronas no es diferente de enviar acciones síncronas, por lo que no discutiremos esto en detalle. Vea [Uso con React](../basico/uso-con-react.md) para una introducción al uso de Redux desde los componentes de React. Consulte [Ejemplo: Reddit API](ejemplo-api-reddit.md) para el código fuente completo discutido en este ejemplo.

## Próximos pasos

Lea [Flujo Asíncrono](flujo-asincrono.md) para recapitular cómo las acciones asíncronas encajan en el flujo Redux.
