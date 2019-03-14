# Uso con React

Comencemos enfatizando que Redux no tiene relación alguna con React. Puedes escribir aplicaciones Redux con React, Angular, Ember, jQuery o vanilla JavaScript.

Dicho esto, Redux funciona especialmente bien con librerías como [React](http://facebook.github.io/react/) y [Deku](https://github.com/dekujs/deku) porque te permiten describir la interfaz de usuario como una función de estado, y Redux emite actualizaciones de estado en respuesta a acciones.

Usaremos React para crear nuestra aplicación sencilla de asuntos pendientes (To-do).

## Instalando React Redux

[React Redux](https://github.com/reactjs/react-redux) no está incluido en Redux de manera predeterminada. Debe instalarlo explícitamente:

```
npm install --save react-redux
```

Si no usas npm, puedes obtener la distribución UMD (Universal Module Definition) más reciente desde unpkg (ya sea la distribución de [desarrollo](https://unpkg.com/react-redux@latest/dist/react-redux.js) o la de [producción](https://unpkg.com/react-redux@latest/dist/react-redux.min.js)). La distribución UMD exporta una variable global llamada `window.ReactRedux` por si la añades a tu página a través de la etiqueta `<script>`.

## Componentes de Presentación y Contenedores

Para asociar React con Redux se recurre a la idea de **separación de presentación y componentes contenedores**. Si no estás familiarizado con estos términos, [lee sobre ellos primero](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), y luego vuelve. ¡Son importantes, así que vamos a esperarte!

¿Has terminado de leer el artículo? Repasemos sus diferencias:

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">Componentes de Presentación</th>
            <th scope="col" style="text-align:left">Componentes Contenedores</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">Propósito
</th>
          <td>Como se ven las cosas (<em>markup</em>, estilos)</td>
          <td>Como funcionan las cosas (búsqueda de datos, actualizaciones de estado)</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Pertinente a Redux</th>
          <td>No</th>
          <td>Yes</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Para leer datos</th>
          <td>Lee datos de los <em>props</em></td>
          <td>Se suscribe al estado en Redux</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Para manipular datos</th>
          <td>Invoca llamada de retorno (callback) desde los <em>props</em></td>
          <td>Envía acciones a Redux</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Son escritas</th>
          <td>Manualmente</td>
          <td>Usualmente generados por React Redux</td>
        </tr>
    </tbody>
</table>

La mayoría de los componentes que escribiremos serán de presentación, pero necesitaremos generar algunos componentes contenedores para conectarlos al *store* que maneja Redux. Con esto y el resumen de diseño que mencionaremos a continuación no implica que los componentes contenedores deban estar cerca o en la parte superior del árbol de componentes. Si un componente contenedor se vuelve demasiado complejo (es decir, tiene componentes de presentación fuertemente anidados con innumerables devoluciones de llamadas que se pasan hacia abajo), introduzca otro contenedor dentro del árbol de componentes como se indica en las [FAQ](../faq/ReactRedux.md#react-multiple-componentes).

Técnicamente podrías escribir los componentes contenedores manualmente usando [`store.subscribe()`](../api/Store.md#subscribe). No le aconsejamos que haga esto porque React Redux hace muchas optimizaciones de rendimiento que son difíciles de hacer a mano. Por esta razón, en lugar de escribir los componentes contenedores, los generaremos utilizando el comando [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-Mapdispatchtoprops-mergeprops-options), función proporcionada por React Redux, como verá a continuación.

## Diseño de la jerarquía de componentes

Recuerda cómo [diseñamos y dimos forma al objecto del estado raíz](Reducers.md)? Es hora de diseñar la jerarquía de la interfaz de usuario para que coincida con este objeto del estado. Esto no es una tarea específica de Redux. [*Thinking in React*](https://facebook.github.io/react/docs/thinking-in-react.html) es un excelente tutorial que explica el proceso.

Nuestro breve resumen del diseño es simple. Queremos mostrar una lista de asuntos pendientes. Al hacer clic, un elemento de la lista se tachará como completado. Queremos mostrar un campo en el que el usuario puede agregar una tarea nueva. En el pie de página, queremos mostrar un *toggle* para mostrar todas las tareas, sólo las completadas, o sólo las activas.

### Diseño de componentes de presentación

Podemos ver los siguientes componentes de presentación y sus *props* surgidos a través de esta breve descripción:

* **`TodoList`** es una lista que mostrará las tareas pendientes disponibles.
  - `todos: Array` es un arreglo de tareas pendientes que contiene la siguiente descripción `{ id, text, completed }`.
  - `onTodoClick(id: number)` es un *callback* para invocar cuando un asunto pendiente es presionado.
* **`Todo`** es un asunto pendiente.
  - `text: string` es el texto a mostrar.
  - `completed: boolean` indica si la tarea debe aparecer tachada.
  - `onClick()` es un *callback* para invocar cuando la tarea es presionada.
* **`Link`** es el enlace con su *callback*.
  - `onClick()` es un *callback* para invocar cuando el enlace es presionado.
* **`Footer`** es donde dejamos que el usuario cambie las tareas pendientes visibles actualmente.
* **`App`** es el componente raíz que representa todo lo demás.

Cada artículo describe la *apariencia* pero no conoce *de donde* vienen los datos, o *cómo* cambiarlos. Sólo muestran lo que se les da. Si migras de Redux a otra cosa, podrás mantener todos estos componentes exactamente iguales. No dependen de Redux en absoluto.

### Diseño de componentes contenedores

También necesitaremos algunos componentes contenedores para conectar los componentes de presentación a Redux. Por ejemplo, el componente de presentación `TodoList` necesita un contenedor como` VisibleTodoList` que se suscribe al *store* de Redux y debe saber cómo aplicar el filtro de visibilidad. Para cambiar el filtro de visibilidad, proporcionaremos un componente contenedor `FilterLink` que renderiza un `Link` que distribuye la debida acción al hacer clic:

* **`VisibleTodoList`** filtra los asuntos de acuerdo a la visibilidad actual y renderiza el `TodoList`.
* **`FilterLink`** obtiene el filtro de visibilidad actual y renderiza un `Link`.
  - `filter: string` es el tipo del filtro de visibilidad.

### Diseño de otros componentes

A veces es difícil saber si un componente debe ser componente de presentación o contenedor. Por ejemplo, a veces la forma y la función están realmente entrelazadas, como en el caso de este pequeño componente:

* **`AddTodo`** es un campo de entrada con un botón "Añadir tarea"

Técnicamente podríamos dividirlo en dos componentes, pero podría ser demasiado pronto en esta etapa. Está bien mezclar presentación y lógica en un componente que sea muy pequeño. A medida que crece, será más obvio cómo dividirlo, así que lo dejaremos en uno solo.  

## Implementación de componentes

¡Vamos a escribir los componentes! Comenzaremos con los componentes de presentación por lo que no es necesario pensar en la relación con Redux todavía.

### Implementación de componentes de presentación

Todos estos son componentes normales de React, por lo que no los examinaremos en detalle. Escribiremos componentes funcionales sin-estado a menos que necesitemos usar el estado local o los métodos del ciclo de duración. Esto no significa que los componentes de presentación *tengan que ser* funciones - es solo que es más fácil definirlos de esta manera. Si, y cuando necesites agregar un estado local, métodos de ciclo de duración u optimizaciones de rendimiento, puedes convertirlos a clases.

#### `components/Todo.js`

```js
import React from 'react'
import PropTypes from 'prop-types';

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```

#### `components/TodoList.js`

```js
import React from 'react';
import PropTypes from 'prop-types';
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```

#### `components/Link.js`

```js
import Reac from 'react';
import PropTypes from 'prop-types';

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

#### `components/Footer.js`

```js
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      Todos
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Activo
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completado
    </FilterLink>
  </p>
)

export default Footer
```

#### `components/App.js`

```js
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
)

export default App
```

Ahora es el momento de conectar los componentes de presentación a Redux mediante la creación de algunos contenedores. Técnicamente, un componente contenedor es sólo un componente de React que utiliza [`store.subscribe ()`](../api/Store.md#subscribe) para leer una parte del árbol de estado en Redux y suministrar los *props* a un componente de presentación que renderiza. Puedes escribir un componente contenedor manualmente, pero sugerimos generar los componentes contenedores con la función [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) de la librería React Redux, ya que proporciona muchas optimizaciones útiles para evitar *re-renders* innecesarios. (Un beneficio de utilizar esta librería es que no tienes que preocuparte por la implementación del método `shouldComponentUpdate` [recomendado por React para un mejor rendimiento](https://facebook.github.io/react/docs/advanced-performance.html).)

Para usar `connect()`, es necesario definir una función especial llamada `mapStateToProps` que indiqua cómo transformar el estado actual del *store* Redux en los *props* que desea pasar a un componente de presentación. Por ejemplo, `VisibleTodoList` necesita calcular `todos` para pasar a `TodoList`, así que definimos una función que filtra el `state.todos` de acuerdo con el `state.visibilityFilter`, y lo usamos en su `mapStateToProps`:

```js
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

Además de leer el estado, los componentes contenedores pueden enviar acciones. De manera similar, puedes definir una función llamada `mapDispatchToProps()` que recibe el método [`dispatch()`](../api/Store.md#dispatch) y devuelve los *callback props* que deseas inyectar en el componente de presentación. Por ejemplo, queremos que `VisibleTodoList` inyecte un *prop* llamado `onTodoClick` en el componente `TodoList`, y queremos que `onTodoClick` envíe una acción `TOGGLE_TODO`:

```js
const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}
```

Finalmente, creamos `VisibleTodoList` llamando `connect()` y le pasamos estas dos funciones:

```js
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

Estos son los conceptos básicos de la API de React Redux, pero hay algunos atajos y opciones avanzadas por lo que vamos a revisar [su documentación](https://github.com/reactjs/react-redux) en detalle. En caso de que te preocupe el hecho que `mapStateToProps` esté creando objetos nuevos con demasiada frecuencia, quizás desees aprender acerca de [computar datos derivados](../recipes/ComputingDerivedData.md) con [*reselect*](https://github.com/reactjs/Reselect).

El resto de los componentes contenedores están definidos a continuación:

#### `containers/FilterLink.js`

```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

#### `containers/VisibleTodoList.js`

```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

### Implementación de otros componentes

#### `containers/AddTodo.js`

```js
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <form onSubmit={e => {
        e.preventDefault()
        if (!input.value.trim()) {
          return
        }
        dispatch(addTodo(input.value))
        input.value = ''
      }}>
        <input ref={node => {
          input = node
        }} />
        <button type="submit">
          Añadir tarea
        </button>
      </form>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```

## Transferir al *store*

Todos los componentes contenedores necesitan acceso al *store Redux* para que puedan suscribirse a ella. Una opción sería pasarlo como un *prop* a cada componente contenedor. Sin embargo, se vuelve tedioso, ya que hay que enlazar `store` incluso a través del componente de presentación ya que puede suceder que tenga que renderizar un contenedor allá en lo profundo del árbol de componentes.

La opción que recomendamos es usar un componente *React Redux* especial llamado [`<Proveedor>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) para [mágicamente](https://facebook.github.io/react/docs/context.html) hacer que el *store* esté disponible para todos los componentes del contenedor en la aplicación sin pasarlo explícitamente. Sólo es necesario utilizarlo una vez al renderizar el componente raíz:

#### `index.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

## Siguientes pasos

Lee el [código fuente completo de este tutorial](ExampleTodoList.md) para internalizar mejor el conocimiento que ha adquirido. Luego, dirígete directamente al [tutorial avanzado](../advanced/README.md) para aprender a manejar los *network requests* y el *routing*!
