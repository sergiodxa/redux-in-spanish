# Uso con React

Desde el principio, necesitamos enfatizar que Redux no tiene relación con React. Puedes escribir aplicaciones Redux con React, Angular, Ember, jQuery o vanilla JavaScript.

Dicho esto, Redux funciona especialmente bien con librerías como [React](http://facebook.github.io/react/) y [Deku](https://github.com/dekujs/deku) porque te permiten describir la interfaz de usuario como una función de estado, y Redux emite actualizaciones de estado en respuesta a acciones.

Usaremos React para crear nuestra una aplicación sencilla de asuntos pendites (todo).

## Instalando React Redux

[React Redux](https://github.com/reactjs/react-redux) no se incluye en Redux de forma predeterminada. Debe instalarlo explícitamente:

```
npm install --save react-redux
```

Si no usas npm, puedes obtener la distribución UMD (Universal Module Definition) más reciente desde unpkg (ya sea la distribución de [desarrollo](https://unpkg.com/react-redux@latest/dist/react-redux.js) o la de [producción](https://unpkg.com/react-redux@latest/dist/react-redux.min.js)). La distribución UMD exporta una variable global llamada `window.ReactRedux` por si la añades a tu pagina a través de la etiqueta `<script>`.

## Componentes de Presentación y Contenedor

Para asociar React con Redux se recurre a la idea de **separacion presentational y componentes contenedores**. Si no está familiarizado con estos términos, [lea sobre ellos primero](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), y luego vuelva. ¡Son importantes, así que vamos lo vamos a esperar!

¿Ha terminado de leer el artículo? Repasemos sus diferencias:

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
          <td>Leer datos de los <em>props</em></td>
          <td>Suscribirse al estado en Redux</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Para manipular datos</th>
          <td>Invocar llamadaa de retorno desde los <em>props</em></td>
          <td>Enviar acciones a Redux</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Son escritas</th>
          <td>Manualmente</td>
          <td>Usualmente generados por React Redux</td>
        </tr>
    </tbody>
</table>

La mayoría de los componentes que escribiremos serán presentacionales, pero necesitaremos generar algunos componentes contenedores para conectarlos al *store* que maneja Redux. Esto y el resumen de diseño que mencionaremos a continuación no implica que los componentes contenedores deban estar cerca o en la parte superior del árbol de componentes. Si un componente contenedor se vuelve demasiado complejo (es decir, tiene componentes presentionales fuertemente anidados con innumerables devoluciones de llamadas que se pasan hacia abajo), introduzca otro contenedor dentro del árbol de componentes como se indica en el [FAQ](../faq/ReactRedux.md#react-multiple-componentes).

Técnicamente usted podría escribir los componentes contenedores manualmente usando [`store.subscribe()`](../api/Store.md#subscribe). No le aconsejamos que haga esto porque React Redux hace muchas optimizaciones de rendimiento que son difíciles de hacer a mano. Por esta razón, en lugar de escribir los componentes contenedores, los generaremos utilizando el comando [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-Mapdispatchtoprops-mergeprops-options) función proporcionada por React Redux, como verá a continuación.

## Diseño de la jerarquía de componentes

Recuerda cómo [diseñamos y dimos forma al objecto del estado raíz](Reducers.md)? Es hora de diseñar la jerarquía de la interfaz de usuario para que coincida con este objeto del estado. Esto no es una tarea específica de Redux. [*Thinking in React*](https://facebook.github.io/react/docs/thinking-in-react.html) es un excelente tutorial que explica el proceso.

Nuestro breve resumen del diseño es simple. Queremos mostrar una lista de asuntos pendientes. Al hacer clic, un elemento de la lista se tachará como completado. Queremos mostrar un campo en el que el usuario puede agregar una tarea nueva. En el pie de página, queremos mostrar un *toggle* para mostrar todas las taras, sólo las completadas, o sólo las activas.

### Diseño de componentes de presentación

Podemos ver los siguientes componentes de presentación y sus *props* surgir a través de esta breve descripción:

* **`TodoList`** es una lista que muestras para tareas pendientes.
  - `todos: Array` es un arreglo de tareas pendientes que contiene la siguiente descripción `{ id, text, completed }`.
  - `onTodoClick(id: number)` es un *callback* para invocar cuando un asunto pendientes es presionado.
* **`Todo`** es un asunto pendiente.
  - `text: string` es el texto a mostrar.
  - `completed: boolean` indica si la tarea debe aparecer tachada.
  - `onClick()` es un *callback* para invocar cuando la tarea es presionada.
* **`Link`** es el enlace con su *callback*.
  - `onClick()` es un *callback* para invocar cuando el enlace es presionado.
* **`Footer`** es donde dejamos que el usuario cambie las tareas pendientes visibles actualmente.
* **`App`** es el componente raíz que representa todo lo demás.

Cada articulo describe la *apariencia* pero no conoce *de donde* los datos vienen, o *cómo* cambiarlos. Sólo muestran lo que se les da. Si migras de Redux a otra cosa, podrás mantener todos estos componentes exactamente iguales. No dependen de Redux en absoluto.

### Diseño de componentes contenedores

También necesitaremos algunos componentes contenedores para conectar los componentes de presentación a Redux. Por ejemplo, el componente presentational `TodoList` necesita un contenedor como` VisibleTodoList` que se suscribe al *store* de Redux y debe saber cómo aplicar el filtro de visibilidad. Para cambiar el filtro de visibilidad, proporcionaremos un componente contenedor `FilterLink` que renderiza un `Link` que distribuye la debida acción al hacer clic:

* **`VisibleTodoList`** filtra los asuntos de acuerdo a la visibilidad actual y renderiza el `TodoList`.
* **`FilterLink`** obtiene la el filtro de visibilidad actual y renderiza un `Link`.
  - `filter: string` es la representacion del filtro de visibilidad.

  ### Diseño de otros componentes

A veces es difícil saber si un componente debe ser componente de presentación o contenedor. Por ejemplo, a veces la forma y la función están realmente juntas, como en el caso de este pequeño componente:

* **`AddTodo`** es un campo de entrada con un botón "Añadir"

Técnicamente podríamos dividirlo en dos componentes, pero podría ser demasiado pronto en esta etapa. Está bien mezclar presentación y lógica en un componente que sea muy pequeño. A medida que crece, será más obvio cómo dividirlo, así que lo dejaremos en uno solo.  

## Implementación de componentes

Vamos a escribir los componentes! Comenzaremos con los componentes de presentación por lo que no es necesario pensar en la relación con Redux todavía.

### Implementación de componentes de presentación

Todos estos son componentes normales de React, por lo que no los examinaremos en detalle. Escribiremos componentes funcionales sin-estado a menos que necesitemos usar el estado local o los métodos del ciclo de duración. Esto no significa que los componentes de presentación *tengan que ser* funciones - es solo que es más fácil definirlos de esta manera. Si y cuando necesites agregar un estado local, métodos de ciclo de duración u optimizaciones de rendimiento, puede convertirlos a clases.

#### `components/Todo.js`

```js
import React, { PropTypes } from 'react'

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
import React, { PropTypes } from 'react'
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
import React, { PropTypes } from 'react'

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

Ahora es el momento de conectar los componentes de presentación a Redux mediante la creación de algunos contenedores. Técnicamente, un componente contenedor es sólo un componente de React que utiliza [`store.subscribe ()`](../api/Store.md#subscribe) para leer una parte del árbol de estado en Redux y suministrar los *props* a un componente de presentación que renderiza. Puedes escribir un componente contenedor manualmente, pero sugerimos generar los componentes contenedores con la función [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) de la librería React Redux, ya que proporciona muchas optimizaciones útiles para evitar *re-renders* innecesarios. (Un resultado de esta utilización es que usted no tiene que preocuparse por la implementación del método `shouldComponentUpdate` [recomendado por React para mejor rendimiento](https://facebook.github.io/react/docs/advanced-performance.html).)

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

