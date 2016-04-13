# Acciones

Primero, vamos a definir algunas acciones.

Las **acciones** son un bloque de información que envia datos desde tu aplicación a tu store. Son la *única* fuente de información para el store. Las envias al store usando [`store.dispatch()`](../api/Store.md#dispatch).

Aquí hay unas acciones de ejemplo que representan agergar nuevas tareas pendientes:

```js
const ADD_TODO = 'ADD_TODO'
```

```js
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

Las acciones son objetos planos de JavaScript. Una acción debe tener una propiedad `type` que indica el tipo de acción a realizar. Los tipos normalmente son definidos como strings constantes. Una vez que tu aplicación sea suficientemente grande, quizas quieras moverlos a un módulo separado.

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

>##### Nota

>No necesitas definir tus tipos de acciones constantes en un archivo separado, o incluso definirlo. Para proyectos pequeños, probablemente sea más fácil usar string directamente para los tipos de acciones. De todas formas, hay algunos beneficios explícitos en declarar constnates en grandes bloques de código. Lee [Reduciendo el Boilerplate](../recipes/reducing-boilerplate.md) para más consejos prácticos sobre como mantener tu código limpio.

Además del `type`, el resto de la estructura de los objetos de acciones depende de tí. Si estas interesado, revisa[Flux Standard Action](https://github.com/acdlite/flux-standard-action) para recomendaciones de como una acción debe armarse.

Vamos a agregar una acción más para describir un usuario marcando una tarea como completa. Nos referimos a una tarea en particular como su `index` ya que vamos a almacenarlos en un array. En una aplicación real, es más inteligente generar un ID único cada vez que creamos una nueva.

```js
{
  type: COMPLETE_TODO,
  index: 5
}
```

Es una buena idea pasar la menor cantidad de información posible. Por ejemplo, es mejor pasar el `index` que todo el objeto de tarea.

Por último, vamos a agregar una acción más para cambiar las tareas actualmente visibles.

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Creadores de acciones

Los **creadores de acciones** son exactamente eso—funciones que crean acciones. Es fácil combinar los términos "acción" con "creador de acción", así que haz lo mejor por usar los términos correctos.

En implementaciones de [Flux tradicional](http://facebook.github.io/flux), los creadores de acciones ejecutan el despacho cuando son invocadas, algo así:

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

En cambio, en Redux los creadores de acciones simplemente regresan una acción:

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

Esto las hace más portables y fáciles de probar. Para efectivamente inicial un despacho, pasa el resultado a la función `dispatch()`:

```js
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

Alternativamente, puedes crear un **creador de acciones conectados** que despache automaticamente:

```js
const boundAddTodo = (text) => dispatch(addTodo(text))
const boundCompleteTodo = (index) => dispatch(completeTodo(index))
```

Ahora puedes llamarlas directamente:

```
boundAddTodo(text)
boundCompleteTodo(index)
```

La función `dispatch()` puede ser accedida directamente desde el store como [`store.dispatch()`](../api/Store.md#dispatch), pero comunmente vas a querer usar utilidades como `connect()` de [react-redux](http://github.com/gaearon/react-redux). Puedes usar [`bindActionCreators()`](../api/bind-action-creators.md) para automaticamente conectar muchos creadores de acciones a `dispatch()`.

Los creadores de acciones pueden además ser asíncronos y tener efectos secundarios. Puedes leer más sobre las [acciones asíncronas](../avanzado/acciones-asíncronas.md) en el [tutorial avanzado](../avanzado/README.md) para aprender como manejar respuestas AJAX y combinar creadores de acciones en un flujo de control asíncrono. No salta ahora mismo hasta las acciones asíncronas hasta que completes el tutorial básico, ya que cubre otros conceptos importantes que son prerequisitos para el tutorial avanzado y las acciones asíncronas.

## Código fuente

### `actions.js`

```js
/*
 * tipos de acciones
 */

export const ADD_TODO = 'ADD_TODO'
export const COMPLETE_TODO = 'COMPLETE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * otras constantes
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * creadores de acciones
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

## Siguientes pasos

¡Ahora vamos a [definir algunos reducers](./reducers.md) para especificar como el estado se actualiza cuando despachas estas acciones!