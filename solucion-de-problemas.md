# Solución de problemas

Este un un lugar para compartir soluciones a problemas comunes.
Los ejemplos usan React, pero deberías encontrarlos útiles incluso si usas otra cosa.

## No ocurre nada el despachar una acción
Algunas veces, tratas de despachar una acción, pero la vista no se actualiza. ¿Por qué ocurre esto? Hay varias razons.

### Nunca modifiques los argumentos del reducer
Si estas tratando de modificar el `state` o `action` que te da Redux. ¡No lo hagas!

Redux asume que nunca modificas el estado que te da en los reducers. **Siempre debes devolver un nuevo objeto con el estado**. Incluso si no usas librerías como [https://facebook.github.io/immutable-js/](Immutable), necesitas evitar completamente las modificaciones.

La inmutabilidad es lo que permite a [react-redux](https://github.com/gaearon/react-redux) suscribirse eficientemente a actualizaciones muy específicas de tu estado. También permite un gran experiencia de desarrollo gracias a características como time travel con [redux-devtools](http://github.com/gaearon/redux-devtools).

Por ejemplo, un reducer como este está mal porque modifica el estado:
```javascript
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      // Mal! Esto modifica el estado
      state.push({
        text: action.text,
        completed: false
      })
      return state
    case 'COMPLETE_TODO':
      // Mal! Esto modifica state[action.index]
      state[action.index].completed = true
      return state
    default:
      return state
  }
}
```
Debe ser re programado de esta forma:
```javascript
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      // Devuelve un nuevo array
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      // Devuelve un nuevo array
      return [
        ...state.slice(0, action.index),
        // Copia el objeto antes de modificarlo
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ]
    default:
      return state
  }
}
```
Es más código, pero es lo que permite a Redux ser predecible y eficiente. si quieres tener menos código puedes usar herramientas como [`React.addons.update`](https://facebook.github.io/react/docs/update.html) para escribir transformaciones inmutables con una sintaxis pequeña:
```javascript
// Antes:
return [
  ...state.slice(0, action.index),
  Object.assign({}, state[action.index], {
    completed: true,
  }),
  ...state.slice(action.index + 1),
];

// Después:
return update(state, {
  [action.index]: {
    completed: {
      $set: true,
    },
  },
});
```
Por último, para actualizar objeto, vas a necesitar algo como `_.extend` de Lodash o mejor, un polyfill de [`Object.assign](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign).

Asegurate de que usas `Object.assign` correctamente. Por ejemplo, en vez de devolver algo como `Object.assign(state, newData)` en tus reducers, devuelve `Object.assign({}, state, newData)`. De estas forma no vas a modificar el `state` anterior.

También puedes habilitar [ES7 object spread proposal](https://github.com/sebmarkbage/ecmascript-rest-spread) con [Babel stage 1](http://babeljs.io/docs/usage/experimental/):
```javascript
// Antes:
return [
  ...state.slice(0, action.index),
  Object.assign({}, state[action.index], {
    completed: true,
  }),
  ...state.slice(action.index + 1),
];

// Después:
return [
  ...state.slice(0, action.index),
  { ...state[action.index], completed: true },
  ...state.slice(action.index + 1),
];
```
Ten en cuenta que las funciones experimentales estás sujetas a cambios, y no es bueno depender de ellas en grandes cantidades de código.

### No olvides llamar [`dispatch(action)`](http://redux.js.org/docs/api/Store.html#dispatch)
Sí defines un creador de acciones, ejecutándolo *no* va a despachar tus acciones automáticamente. Por ejemplo, este código no hace nada:

#### **`TodoActions.js`**
```javascript
export function addTodo(text) {
  return { type: 'ADD_TODO', text };
};
```

#### **`AddTodo.js`**
```javascript
import React, { Component } from 'react';
import { addTodo } from './TodoActions';

class AddTodo extends Component {
  handleClick() {
    // ¡No funciona!
    addTodo('Fix the issue');
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}>
        Add
      </button>
    );
  }
}
```
No funciona ya que el creador de acciones es solo una función que *devuelve* una acción. Pero depende de vos despacharla.No podemos conectar tu creador de acciones a una instancia específica del Store durante su definición ya que las aplicaciones que renderizas en el servidor necesitas un store de Redux para cada petición.

Esto se arreglar ejecutando el método [`dispatch()`](http://redux.js.org/docs/api/Store.html#dispatch) de la instancia del [store](http://redux.js.org/docs/api/Store.html):
```javascript
handleClick() {
  // ¡Funciona! (pero debes obtener el store de alguna forma)
  store.dispatch(addTodo('Fix the issue'));
}
```
Si estas en algún punto profundo en la jerarquía de componentes, es incómodo pasar el store manualmente. Es por eso que [react-redux](https://github.com/gaearon/react-redux) te permite usar el [componente de nivel superior](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) `connect` que va a además de suscribirse al store de Redux, inyectar `dispatch` en los props de tus componentes.

El código arreglado quedaría así:

#### **`AddTodo.js`**
```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { addTodo } from './TodoActions';

class AddTodo extends Component {
  handleClick() {
    // ¡Funciona!
    this.props.dispatch(addTodo('Fix the issue'));
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}>
        Add
      </button>
    )
  }
}

// Además del state, `connect` agregar `dispatch` en nuestros props.
export default connect()(AddTodo);
```
Puedes pasar `dispatch` a otros componentes manualmente, si quieres.

## Algo más no funciona
Pregunta en el canal #redux con [Reactiflux](http://reactiflux.com/), o [crea un issue](https://github.com/rackt/redux/issues).
Si lo descubre, [modifica este documento](https://github.com/rackt/redux/edit/master/docs/Troubleshooting.md) como cortesía a la siguiente persona con el mismo problema.