# Referencia API
La API de Redux es diminuta. Redux define una serie de contratos que debes implementar (como los [reducers](../glosario.md#reducer) y provee una pocas funciones de ayuda para unir todo.

Esta sección documenta completamente la API de Redux. Recuerda que Redux solo se preocupa con mantener el estado de tu aplicación. En una aplicación de verdad, seguramente quieras usar conexiones como [react-redux](https://github.com/gaearon/react-redux).

### Exportaciones de nivel superior
- [`createStore(reducer, [initialState])`]()
- [`combineReducers(reducers)`]()
- [`applyMiddleware(...middlewares)`]()
- [`bindActionCreatores(actionCreators, dispatch)`]()
- [`compose(...functions)`]()

### API del Store
- [Store]()
    - [`getState()`]()
    - [`dispatch(action)`]()
    - [`subscribe(listener)`]()
    - [`replaceReducer(nextReducer)`]()

### Importando
Cada función descrita arriba ex una exportación de nivel superior. Puedes importar cualquiera de estas de esta forma:

#### **ES6**
```javascript
import { createStore } from 'redux';
```
#### **ES5 (CommonJS)**
```javascript
var createStore = require('redux').createStore;
```
#### **ES5 (UMD build)**
```javascript
var createStore = Redux.createStore;
```