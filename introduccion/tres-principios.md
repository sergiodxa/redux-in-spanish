# Tres Principios

Redux puede ser descrito en tres principios fundamentales:

## Única fuente de la verdad
**El [estado](glosario#estado) de toda tu aplicación esta almacenado en un árbol guardado en un único [store](glosario.md#store).**

Esto hace fácil crear aplicaciones universales, ya que el estado en tu servidor puede ser serializado y enviado al cliente sin ningún esfuerzo extra. Como un único árbol de estado también hace más fácil depurar una aplicación; te permite también mantener el estado de la aplicación en desarrollo, para un ciclo de desarrollo más veloz. Algunas funcionalidades que históricamente han sido difíciles de implementar - como Deshacer/Rehacer, por ejemplo - se vuelven triviales si todo tu estado se guarda en un solo árbol.
```javascript
console.log(store.getState())
/* Imprime
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Considerar usar Redux',
      completed: true
    },
    {
      text: 'Mantener todo el estado en un solo árbol',
      completed: false
    }
  ]
}
*/
```

## El estado es de solo lectura
**La única forma de modificar el estado es emitiendo una [acción](glosario.md#accion), un objeto describiendo que ocurrió.**

Esto te asegura que ni tu vista ni callbacks de red van a modificar el estado directamente. En vez de eso, expresan un intento de modificar el estado. Ya que todas las modificaciones están centralizadas y suceden en un orden estricto, no hay que preocuparse por una carrera entre las acciones. y como las acciones son objetos planos, pueden ser registrados, serializados y almacenados para volver a ejecutarlos por cuestiones de depuración y pruebas.
```javascript
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
});

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
});
```

## Los cambios se realizan con funciones puras
**Para especificar como el árbol de estado es transformado por las acciones, se utilizan [reducers](glosario.md#reducer) puros.**

Los reducers son funciones puras que toman el estado anterior y una acción, y devuelven un nuevo estado. Recuerda devolver un nuevo objeto de estado en vez de modificar el anterior. Puedes empezar con un único reducer, y mientras tu aplicación crece, dividirlo en varios reducers pequeños que manejan partes específicas del árbol de estado. Ya que los reducers son funciones puras, puedes controlar el orden en que se ejecutan, pasarle datos adicionales, o incluso hacer reducers reusables para tareas comunes como paginación.
```javascript
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter;
    default:
      return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false,
        },
      ];
    case 'COMPLETE_TODO':
      return [
        ...state.slice(0, action.index),
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1),
      ];
    default:
      return state;
  }
}

import { combineReducers, createStore } from 'redux';
let reducer = combineReducers({ visibilityFilter, todos });
let store = createStore(reducer);
```

¡Y eso es todo! Ahora sabes de que se trata Redux.