# `bindActionCreators(actionCreators, dispatch)`

Convierte un objeto cuyos valores son [creadores de acciones](../glosario.md#creador-de-acciones), en un objeto, con las mismas claves, pero donde cada creador de acciones está envuelto en una llamada a [`dispatch`](./Store.md#dispatch) para ser invocada inmediatamente.

Normalmente solo deberías llamar a [`dispatch`](./Store.md#dispatch) directamente desde tu instancia del [`Store`](./Store.md). Si usas Redux con React, [react-redux](https://github.com/gaearon/react-redux) te provee de la función [`dispatch`](Store.md#dispatch) para que la llames directamente.

El único caso de uso para `bindActionCreatores` es cuando quieres pasar un creador de acciones a un componente que no sabe nada de Redux y no quieres pasarle [`dispatch`](./Store.md#dispatch) o el store de Redux.

Por conveniencia, también puedes pasar una sola función como primer argumento, y obtener una función devuelta.

#### Parametros

1. `actionCreators` (*Función* u *Objeto*): Un [creador de acciones](../glosario.md#creador-de-acciones), o un objeto cuyos valores son creadores de acciones.

2. `dispatch` (*Función*):  La función [`dispatch`](./Store.md#dispatch) disponible en la instancia del [`Store`](./Store.md).

#### Regresa

(*Función* u *Objeto*): Un objeto similar al original, pero donde cada función despacha inmediatamente la acción regresada por el creador de acciones correspondiente. Si pasas una función como `actionCreators`, el valor devuelto va a ser también una única función.

#### Ejemplo

#### `TodoActionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

#### `SomeComponent.js`

```js
import { Component } from 'react'
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'

import * as TodoActionCreators from './TodoActionCreators'
console.log(TodoActionCreators)
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  componentDidMount() {
    // Inyectado por react-redux:
    let { dispatch } = this.props

    // Nota: esto no funciona
    // TodoActionCreators.addTodo('Use Redux')

    // Solamente estás llamando una función que crea una acción
    // ¡También deberías despachar la acción!

    // Esto funcionaría
    let action = TodoActionCreators.addTodo('Use Redux')
    dispatch(action)
  }

  render() {
    // Inyectado por react-redux:
    let { todos, dispatch } = this.props

    // Acá hay un buen caso de uso para bindActionCreators:
    // Quieres un componente hijo  que esté completamente desatendido de Redux

    let boundActionCreators = bindActionCreators(TodoActionCreators, dispatch)
    console.log(boundActionCreators)
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }

    return (
      <TodoList todos={todos}
                {...boundActionCreators} />
    )

    // Una alternativa a bindActionCreators es enviar
    // la función dispatch hacia abajo, pero entonces tu componente
    // hijo necesita importar los creadores de acciones y saber sobre ellos

    // return <TodoList todos={todos} dispatch={dispatch} />
  }
}

export default connect(
  state => ({ todos: state.todos })
)(TodoListContainer)
```

#### Consejos

* Capaz te preguntes: ¿por qué no unimos los creadores de acciones a la instancia del store directamente, como en Flux? El problema es que eso no funciona en aplicaciones universales que deben renderizar en el servidor. Normalmente vas a querer tener una instancia del store por cada petición así puedes usarlas con diferentes datos, pero conectar los creadores de acciones durante su definición significa que estás atascado con una única instancia por petición.

* Si usas ES5, en vez de usar la sintaxis `import * as` puedes simplemente pasar `require('./TodoActionCreators')` a `bindActionCreators` como primer argumento. Lo único que te tiene que importar es que los valores del argumento `actionCreators` sean funciones. El sistema de módulo no importa.
