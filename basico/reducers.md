# Reducers

Las [acciones](./acciones.md) describen que *algo paso*, pero no especifican como el estado de la aplicación cambio en respuesta. Esto es trabajo de los reducers.

## Diseñando la forma del estado

En Redux, todo el estado de la aplicación es almacenado en un único objeto. Es una buena idea pensar en su forma antes de escribir código. ¿Cuál es la mínima representación del estado de la aplicación como un objeto?

Para nuestra aplicación de tareas, vamos a querer guardar dos cosas diferentes:

* El filtro de visibilidad actualmente seleccionado;
* La lista actual de tareas.

Algunas veces vas a ver que necesitas almacenar algunos datos, así como el estado de la UI, en el árbol de estado. Esto esta bien, pero trata de mantener los datos separado del estado de la UI.

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

>##### Nota sobre relaciones

>En aplicaciones más complejas, vas a necesitar tener diferentes entidades que se referencien una a otra. Sugerimos mantener el estado tan normalizado como sea posible, sin nada de anidación. Mantener cada entidad en un objeto con el ID como llave, y usa los IDs para referenciar otras entidades, o para listas. Piensa en el estado de la aplicación como una base de datos. Este enfoque es descripto en la documentación de [normalizr's](https://github.com/gaearon/normalizr) más detalladamente. Por ejemplo, manteniendo `todosById: { id ->  todo }` y `todos: array<id>` dentro del estado es mejor para una aplicación real, pero lo vamos a matener simple para el ejemplo.

## Manejando Acciones

Ahora que decidimos como se va a ver nuestro objeto de estado, estamos listos para escribir nuestro reducer. El reducer es una función pura que toma el estado anterior y una acción, y devuelve en nuevo estado.

```js
(previousState, action) => newState
```

Se llama reducer porque es el tipo de función que pasarías a [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/reduce). Es muy importante que los reducer se mantengan puros. Cosas que **nunca** deberías hacer dentron de un reducer:

* Modificar sus argumentos;
* Realizar tareas con efectos secundarios como llamas a un API o transiciones de rutas.
* Llamar una función no pura, por ejemplo `Date.now()` o `Math.random()`.

We’ll explore how to perform side effects in the [advanced walkthrough](../advanced/README.md). For now, just remember that the reducer must be pure. **Given the same arguments, it should calculate the next state and return it. No surprises. No side effects. No API calls. No mutations. Just a calculation.**

With this out of the way, let’s start writing our reducer by gradually teaching it to understand the [actions](Actions.md) we defined earlier.

We’ll start by specifying the initial state. Redux will call our reducer with an `undefined` state for the first time. This is our chance to return the initial state of our app:

```js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
}

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // For now, don’t handle any actions
  // and just return the state given to us.
  return state
}
```

One neat trick is to use the [ES6 default arguments syntax](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters) to write this in a more compact way:

```js
function todoApp(state = initialState, action) {
  // For now, don’t handle any actions
  // and just return the state given to us.
  return state
}
```

Now let’s handle `SET_VISIBILITY_FILTER`. All it needs to do is to change `visibilityFilter` on the state. Easy:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

Note that:

1. **We don’t mutate the `state`.** We create a copy with [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign). `Object.assign(state, { visibilityFilter: action.filter })` is also wrong: it will mutate the first argument. You **must** supply an empty object as the first parameter. You can also enable the [object spread operator proposal](../recipes/UsingObjectSpreadOperator.md) to write `{ ...state, ...newState }` instead.

2. **We return the previous `state` in the `default` case.** It’s important to return the previous `state` for any unknown action.

>##### Note on `Object.assign`

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) is a part of ES6, but is not implemented by most browsers yet. You’ll need to either use a polyfill, a [Babel plugin](https://www.npmjs.com/package/babel-plugin-object-assign), or a helper from another library like [`_.assign()`](https://lodash.com/docs#assign).

>##### Note on `switch` and Boilerplate

>The `switch` statement is *not* the real boilerplate. The real boilerplate of Flux is conceptual: the need to emit an update, the need to register the Store with a Dispatcher, the need for the Store to be an object (and the complications that arise when you want a universal app). Redux solves these problems by using pure reducers instead of event emitters.

>It’s unfortunate that many still choose a framework based on whether it uses `switch` statements in the documentation. If you don’t like `switch`, you can use a custom `createReducer` function that accepts a handler map, as shown in [“reducing boilerplate”](../recipes/ReducingBoilerplate.md#reducers).

## Handling More Actions

We have two more actions to handle! Let’s extend our reducer to handle `ADD_TODO`.

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })    
    default:
      return state
  }
}
```

Just like before, we never write directly to `state` or its fields, and instead we return new objects. The new `todos` is equal to the old `todos` concatenated with a single new item at the end. The fresh todo was constructed using the data from the action.

Finally, the implementation of the `COMPLETE_TODO` handler shouldn’t come as a complete surprise:

```js
case COMPLETE_TODO:
  return Object.assign({}, state, {
    todos: state.todos.map((todo, index) => {
      if (index === action.index) {
        return Object.assign({}, todo, {
          completed: true
        })
      }
      return todo
    })
  })
```

Because we want to update a specific item in the array without resorting to mutations, we have to create a new array with the same items except the item at the index. If you find yourself often writing such operations, it’s a good idea to use a helper like [react-addons-update](https://facebook.github.io/react/docs/update.html), [updeep](https://github.com/substantial/updeep), or even a library like [Immutable](http://facebook.github.io/immutable-js/) that has native support for deep updates. Just remember to never assign to anything inside the `state` unless you clone it first.

## Splitting Reducers

Here is our code so far. It is rather verbose:

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case COMPLETE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if(index === action.index) {
            return Object.assign({}, todo, {
              completed: true
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

Is there a way to make it easier to comprehend? It seems like `todos` and `visibilityFilter` are updated completely independently. Sometimes state fields depend on one another and more consideration is required, but in our case we can easily split updating `todos` into a separate function:

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case COMPLETE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
    case COMPLETE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

Note that `todos` also accepts `state`—but it’s an array! Now `todoApp` just gives it the slice of the state to manage, and `todos` knows how to update just that slice. **This is called *reducer composition*, and it’s the fundamental pattern of building Redux apps.**

Let’s explore reducer composition more. Can we also extract a reducer managing just `visibilityFilter`? We can:

```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter
  default:
    return state
  }
}
```

Now we can rewrite the main reducer as a function that calls the reducers managing parts of the state, and combines them into a single object. It also doesn’t need to know the complete initial state anymore. It’s enough that the child reducers return their initial state when given `undefined` at first.

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case COMPLETE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

**Note that each of these reducers is managing its own part of the global state. The `state` parameter is different for every reducer, and corresponds to the part of the state it manages.**

This is already looking good! When the app is larger, we can split the reducers into separate files and keep them completely independent and managing different data domains.

Finally, Redux provides a utility called [`combineReducers()`](../api/combineReducers.md) that does the same boilerplate logic that the `todoApp` above currently does. With its help, we can rewrite `todoApp` like this:

```js
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

Note that this is completely equivalent to:

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

You could also give them different keys, or call functions differently. These two ways to write a combined reducer are completely equivalent:

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})
```

```js
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

All [`combineReducers()`](../api/combineReducers.md) does is generate a function that calls your reducers **with the slices of state selected according to their keys**, and combining their results into a single object again. [It’s not magic.](https://github.com/reactjs/redux/issues/428#issuecomment-129223274)

>##### Note for ES6 Savvy Users

>Because `combineReducers` expects an object, we can put all top-level reducers into a separate file, `export` each reducer function, and use `import * as reducers` to get them as an object with their names as the keys:

>```javascript
>import { combineReducers } from 'redux'
>import * as reducers from './reducers'
>
>const todoApp = combineReducers(reducers)
```

>Because `import *` is still new syntax, we don’t use it anymore in the documentation to avoid [confusion](https://github.com/reactjs/redux/issues/428#issuecomment-129223274), but you may encounter it in some community examples.

## Source Code

#### `reducers.js`

```javascript
import { combineReducers } from 'redux'
import { ADD_TODO, COMPLETE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case COMPLETE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

## Next Steps

Next, we'll explore how to [create a Redux store](Store.md) that holds the state and takes care of calling your reducer when you dispatch an action.
