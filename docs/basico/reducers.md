# Reducers

Las [acciones](./acciones.md) describen que *algo pasó*, pero no especifican cómo cambió el estado de la aplicación en respuesta. Esto es trabajo de los reducers.

## Diseñando la forma del estado

En Redux, todo el estado de la aplicación es almacenado en un único objeto. Es una buena idea pensar en su forma antes de escribir código. ¿Cuál es la mínima representación del estado de la aplicación como un objeto?

Para nuestra aplicación de tareas, vamos a querer guardar dos cosas diferentes:

* El filtro de visibilidad actualmente seleccionado;
* La lista actual de tareas.

Algunas veces verás que necesitas almacenar algunos datos, así como el estado de la UI, en el árbol de estado. Esto está bien, pero trata de mantener los datos separados del estado de la UI.

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

>En aplicaciones más complejas, vas a necesitar tener diferentes entidades que se referencien una a otra. Sugerimos mantener el estado tan normalizado como sea posible, sin nada de anidación. Mantener cada entidad en un objeto con el ID como llave, y usa los IDs para referenciar otras entidades, o para listas. Piensa en el estado de la aplicación como una base de datos. Este enfoque se describe en la documentación de [normalizr](https://github.com/gaearon/normalizr) más detalladamente. Por ejemplo, manteniendo `todosById: { id ->  todo }` y `todos: array<id>` dentro del estado es mejor para una aplicación real, pero lo vamos a matener simple para el ejemplo.

## Manejando Acciones

Ahora que decidimos cómo se verá nuestro objeto de estado, estamos listos para escribir nuestro reducer. El reducer es una función pura que toma el estado anterior y una acción, y devuelve en nuevo estado.

```js
(previousState, action) => newState
```

Se llama reducer porque es el tipo de función que pasarías a [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/reduce). Es muy importante que los reducer se mantengan puros. Cosas que **nunca** deberías hacer dentron de un reducer:

* Modificar sus argumentos;
* Realizar tareas con efectos secundarios como llamas a un API o transiciones de rutas.
* Llamar una función no pura, por ejemplo `Date.now()` o `Math.random()`.

En la [guía avanzada](../avanzado/README.md) vamos a ver como realizar efectos secundarios. Por ahora, solo recuerda que los reducers deben ser puros. **Dados los mismos argumentos, debería calcular y devolver el siguiente estado. Sin sorpresas. Sin efectos secundarios. Sin llamadas a APIs. Sin mutaciones. Solo cálculos.**

Con esto dicho, vamos a empezar a escribir nuestro reducer gradualmente enseñandole como entender las [acciones](../acciones.md) que definimos antes.

Vamos a empezar por especificar el estado inicial. Redux va a llamar a nuestros reducers con `undefined` como valor del estado la primera vez. Esta es nuestra oportunidad de devolver el estado inicial de nuestra applicación.

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

  // Por ahora, no maneja ninguna acción
  // y solo devuelve el estado que recibimos.
  return state
}
```

Un estupendo truco es usar la [sintáxis de parámetros por defecto de ES6](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Funciones/Parametros_por_defecto) para hacer lo anterior de forma más compacta:

```js
function todoApp(state = initialState, action) {
  // Por ahora, no maneja ninguna acción
  // y solo devuelve el estado que recibimos.
  return state
}
```

Ahora vamos a manejar `SET_VISIBILITY_FILTER`. Todo lo que necesitamos hacer es cambiar la propiedad `visibilityFilter` en el estado. Fácil:

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

Nota que:

1. **No modificamos el `state`.** Creamos una copia con [`Object.assign()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Object/assign). `Object.assign(state, { visibilityFilter: action.filter })` también estaría mal: esto modificaría el primero argumento. **Debes** mandar un objeto vacío como primer parámetro. También puedes activar la [propuesta del operador spread](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Operadores/Spread_operator) para escribir `{ ...state, ...newState }`.

2. **Devolvemos el anterior `state` en el caso `default`**. Es importarte devolver el anterior `state` por cualquier acción desconocida.

>##### Nota sobre `Object.assign`

>[`Object.assign()`](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Object/assign) es parte de ES6, pero no esta implementado en la mayoría de los navegadores todavía. Vas a necesitar usar ya sea un polyfill,  el [plugin de Babel](https://www.npmjs.com/package/babel-plugin-object-assign), o alguna otra función como [`_.assign()`](https://lodash.com/docs#assign).

>##### Nota sobre `switch` y Boilerplate

>La sentencia `switch` no es verdadero boilerplate. El verdadero boilerplate de Flux es conceptual: la necesidad de emitir una actualización, la necesidad de registrar el Store con el Dispatcher, la necesidad de que el Store sea un objeto (y las complicaciones que existen para hacer aplicaciones universales). Redux resuelve estos problemas usando reducers puros en vez de emisores de eventos.

>Desafortunadamente muchos todavía eligen un framework basados en si usan `switch` en su documentación. Si no te gusta `switch`, puedes usar alguna función `createReducer` personalizada que acepte un mapa, como se ve en ["Reduciendo el Boilerplate"](../recetas/reduciendo-el-boilerplate.md#reducers).

## Manejando más acciones

¡Todavía tenemos dos acciones más que manejar! Vamos a extender nuestro reducer para manejar `ADD_TODO`.

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

Tal cual como antes, nunca modificamos directamente `state` o sus propiedades, en cambio devolvemos un nuevo objeto. El nuevo `todos` es igual al viejo `todos` agregándole un único objeto nuevo al final. La tarea más nueva es creada usando los datos de la acción.

Finalmente, la implementación de `COMPLETE_TODO` no va a venir con ninguna una sorpresa:

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

Debido a que queremos actualizar un objeto específico del array sin recurrir a modificaciones, necesitamos crear un nuevo array con los mismo objetos menos el objeto en la posición. Si te encuentras realizando mucho estas operaciones, es una buena idea usar utilidades como [react-addons-update](https://facebook.github.io/react/docs/update.html), [updeep](https://github.com/substantial/updeep), o incluso una librería como [Immutable](http://facebook.github.io/immutable-js/) que tienen soporte nativo a actualizaciones profundas. Solo recuerda nunca asignar nada a algo dentro de `state` antes de clonarlo primero.

## Separando Reducers

Este es nuestro código hasta ahora. Es algo verboso:

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

¿Hay alguna forma de hacerlo más fácil de entender? Parece que `todos` y `visibilityFilter` se actualizan de forma completamente separadas. Algunas veces campos del estado dependen uno de otro y hay que tener en cuenta más cosas, pero en nuestro caso podemos facilmente actualizar `todos` en una función separada:

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

Fijate que `todos` acepta `state`—¡Pero es un array! Ahora `todoApp` solo le manda una parte del estado para que la maneje, y `todos` sabe como actualizar esa parte. **Esto es llamado *composición de reducers*, y es un patrón fundamental al construir aplicaciones de Redux.**

Vamos a explorar la composición de reducers un poco más. ¿Podemos extraer a otro reducer el control de `visibilityFilter`? Podemos:

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

Ahora podemos reescribir el reducer principal como una función que llama a los reducers que controlan distintas partes del estado, y los combina en un solo objeto. Ni siquiera necesita saber el estado inicial. Es suficiente con que los reducers hijos devuelvan su estado inicial cuando reciben `undefined` la primera vez.

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

**Nota que cada uno de estos reducers esta manejando su propia parte del estado global. El parámetro `state` es diferente por cada reducer, y corresponde con la parte del estado que controla.**

¡Esto ya se está viendo mejor! Cuando una aplicación es muy grande, podemos dividir nuestros reducers en archivos separados y mantenerlos completamente independientes y controlando datos específicos.

Por último, Redux viene con una utilidad llamada [`combineReducers()`](../api/combine-reducers.md) que realiza la misma lógica que usa `todoApp` arriba. Con su ayuda, podemos reescribir `todoApp` de esta forma.

```js
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

Fíjate que esto es exactamente lo mismo que:

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

Además puedes darles diferentes nombres, o llamar funciones diferentes. Estas dos formas de combinar reducers son exactamente lo mismo:

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

Todo lo que [`combineReducers()`](../api/combine-reducers.md) hace es generar una función que llama a tus reducers **con la parte del estado seleccionada de acuerdo a su propiedad**, y combinar sus resultados en un único objeto. [It’s not magic.](https://github.com/reactjs/redux/issues/428#issuecomment-129223274).

>##### Nota para expertos de ES6

>Ya que `combineReducers` espera un objeto, podemos poner todos nuestros reducers en un archivo separado, `export` cada función reductora, y usar `import * as reducers` para obtenerlos todos juntos como objetos con sus nombres como propiedades.

>```javascript
>import { combineReducers } from 'redux'
>import * as reducers from './reducers'
>
>const todoApp = combineReducers(reducers)
```

>Ya que `import *` es todavía una sintaxis nueva, no la usamos más en la documentación para evitar [confusiones](https://github.com/reactjs/redux/issues/428#issuecomment-129223274), pero probalemente te encuentro con algunos ejemplos en la comunidad.

## Código fuente

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

## Siguientes pasos

A continuación, vamos a ver como [crear un store de Redux](./store.md) que contenga todo el estado y se encargue de llamar a nuestro reducer cuando se despache una acción.
