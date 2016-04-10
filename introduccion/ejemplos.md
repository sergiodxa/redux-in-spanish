# Ejemplos

Redux es distribuido con algunos ejemplos en su [código fuente](https://github.com/reactjs/redux/tree/master/examples).

## Counter Vanilla

Ejecuta el ejemplo de [Counter Vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla)

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/counter-vanilla
open index.html
```

No requiere un sistema de build o un framework de vista y existe para mostrar la API pura de Redux usando ES5.

## Counter

Ejecuta el ejemplo de [Counter](https://github.com/reactjs/redux/tree/master/examples/counter):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/counter
npm install
npm start

open http://localhost:3000/
```

Este es el más básico ejemplo usando Redux junto a React. Por simpleza, vuelve a renderizar el componente de React manualmente cuando el store cambia. En un proyecto real, deberías usar algo con mejor rendimiento como [React Redux](https://github.com/reactjs/react-redux).

Este ejemplo incluye pruebas.

## Todos

Ejecuta el ejemplo de [Todos](https://github.com/reactjs/redux/tree/master/examples/todos):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/todos
npm install
npm start

open http://localhost:3000/
```

Este es el mejor ejemplo para obtener un conocimiento más profundo de como las actualizaciones de estado funcionan en Redux. Muestra como los reducers pueden delegar el manejo de acciones a otros reducers, y como usar [React Redux](https://github.com/reactjs/react-redux) para generar componentes contenedores para tus componentes presentacionales.

Este ejemplo incluye pruebas.

## Todos with Undo

Ejecuta el ejemplo de [Todos with Retroceder](https://github.com/reactjs/redux/tree/master/examples/todos-with-undo):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/todos-with-undo
npm install
npm start

open http://localhost:3000/
```

Este es una variación del ejemplo anterior. Es practicamente identico, pero además muestra como envolver tus reducers con [Redux Undo](https://github.com/omnidan/redux-undo) te permite agregar la funcionalidad de Deshacer/Rehacer a tu aplicaciones con una pocas líneas.

## TodoMVC

Ejecuta el ejemplo de [TodoMVC](https://github.com/reactjs/redux/tree/master/examples/todomvc):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/todomvc
npm install
npm start

open http://localhost:3000/
```

Este es el clásico ejemplo de [TodoMVC](http://todomvc.com/). Esta aquí por razones de comparación, pero cubre los mismos puntos que el ejemplo de Todos.

Este ejemplo incluye pruebas.

## Shopping Cart

Ejecuta el ejemplo de [Shopping Cart](https://github.com/reactjs/redux/tree/master/examples/shopping-cart):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/shopping-cart
npm install
npm start

open http://localhost:3000/
```

Este ejemplo muestra importantes patrones de Redux que se vuelven más importantes mientras tu aplicación crece. En particular, muestra como guardar entidades de una forma normalizada usando sus IDs, como componer reducer en varios niveles, y como definir selectores junto a los reducers así el conocimiento de la forma del estado está encapsulado. Además muestra como registrar cambios con [Redux Logger](https://github.com/fcomb/redux-logger) y despachar acciones con el middleware [Redux Thunk](https://github.com/gaearon/redux-thunk).

## Tree View

Ejecuta el ejemplo de [Tree View](https://github.com/reactjs/redux/tree/master/examples/tree-view):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/tree-view
npm install
npm start

open http://localhost:3000/
```

Este ejemplo muestra como renderizas una vista de un árbol profundamente anidado y representar su estado de forma normalizada así es fácil actualizarlo mediante reducers. Un buen rendimiento al renderizar al hacer que los componentes contenedores se suscriban solo a los nodos del árbol que renderizan.

Este ejemplo incluye pruebas.

## Async

Ejecuta el ejemplo de [Async](https://github.com/reactjs/redux/tree/master/examples/async):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/async
npm install
npm start

open http://localhost:3000/
```

Este ejemplo incluye leer desde un API asínrono, obtener datos en respuestas a la acciones del usuario, mostrar indicadores de cargando, cachear respuestas e invalidar la cache. Usa el middleware It uses [Redux Thunk](https://github.com/gaearon/redux-thunk) para encapsular efectos secundarios asíncronos.

## Universal

Ejecuta el ejemplo de [Universal](https://github.com/reactjs/redux/tree/master/examples/universal):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/universal
npm install
npm start

open http://localhost:3000/
```

Esta es las más básica demostración de [renderizado en el servidor](../receras/render-en-el-servidor.md) con Redux y React. Muestra como preparar el estado inicial en el servidor y pasarlo al cliente así se inicia desde el estado existente.

## Real World

Ejecuta el ejemplo de [Real World](https://github.com/reactjs/redux/tree/master/examples/real-world):

```
git clone https://github.com/reactjs/redux.git

cd redux/examples/real-world
npm install
npm start

open http://localhost:3000/
```

Este es el ejemplo más avanzado. Es grande por diseño. Cubre como mantener entidades en una cache normalizada, implementando un middleware personalizado para llamadas a API, renderizando páginas parcialmente cargadas, paginación, cacheo de respeustas, mostrar mensajes de error y ruteo. Adicionalmente, incluye las Redux DevTools.

## Más Ejemplos

Puedes encontrar más ejemplos en [Awesome Redux](https://github.com/xgrommx/awesome-redux).
