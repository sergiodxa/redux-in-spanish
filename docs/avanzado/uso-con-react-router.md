# Uso con React Router

Así que deseas hacer enrutamiento en tu aplicación Redux. Puedes usar [React Router](https://github.com/reactjs/react-router). Redux será la fuente de verdad para tus datos y React Router será la fuente de verdad para tu URL. En la mayoría de los casos, **está bien** tenerlos separados a menos que necesites *viajar en el tiempo* y dar marcha atrás a acciones que causen cambios en la URL.

## Instalación de React Router
`React-router` está disponible en npm. Esta guía asume que se está utilizando `react-router@^2.7.0`.

`npm install --save react-router`

## Configuración de la URL de recuperación (*fallback*)

Antes de integrar React Router, necesitamos configurar nuestro servidor de desarrollo. De hecho, nuestro servidor de desarrollo puede no ser consciente de las rutas declaradas en la configuración de React Router. Por ejemplo, si accedes a `/todos` y se refresca el navegador, tu servidor de desarrollo debe ser configurado para servir `index.html` porque es una aplicación de una sola página (SPA). A continuación, te indicamos cómo habilitarlo con los servidores de desarrollo más populares.

>### Nota sobre *Create React App*

> Si estás utilizando *Create React App*, no necesitarás configurar una URL de recuperación ya que se realizará automáticamente.

### Configurando Express
Si estás sirviendo el archivo `index.html` desde Express:
``` js
  app.get('/*', (req,res) => {
    res.sendFile(path.join(__dirname, 'index.html'))
  })
```

### Configurando WebpackDevServer
Si estás sirviendo el archivo `index.html` desde WebpackDevServer:
Añade a tu archivo de configuración `webpack.config.dev.js` lo siguiente:
```js
  devServer: {
    historyApiFallback: true,
  }
```

## Conectar React Router con Redux

A lo largo de este capítulo, utilizaremos el ejemplo de listas [*To-dos*](https://github.com/reactjs/redux/tree/master/examples/todos). Te recomendamos que lo clones mientras lees este capítulo.

Primero tendremos que importar `<Router />` y `<Route />` de React Router. A continuación, te indicamos cómo hacerlo:

```js
import { Router, Route, browserHistory } from 'react-router';
```

En una aplicación React, usualmente agruparás `<Route />` en `<Router />` para que cuando cambie la URL, `<Router />` coincida con el recurso de sus rutas y procese los componentes configurados. `<Route />` se utiliza para asignar mapas de manera declarativa a la jerarquía de componentes de tu aplicación. Declararás en `path` la ruta utilizada en la URL y en `component` el único componente que se renderizará cuando la ruta coincida con la URL.

```js
const Root = () => (
  <Router>
    <Route path="/" component={App} />
  </Router>  
);
```

Sin embargo, en nuestra aplicación Redux todavía necesitaremos `<Provider />`. `<Provider />` es el componente de orden superior proporcionado por React Redux que te permite asociar Redux a React (ver [Uso con React](../basico/uso-con-react.md)).

Importaremos el componente `<Provider />` de React Redux.

```js
import { Provider } from 'react-redux';
```

Agruparemos `<Router />` en `<Provider />` para que los manejadores de rutas puedan [accesar el `store`](../basico/uso-con-react.md#transferir-al-store).

```js
const Root = ({ store }) => (
  <Provider store={store}>
    <Router>
      <Route path="/" component={App} />
    </Router>
  </Provider>
);
```

Ahora el componente `<App />` se renderizará si la URL coincide con '/'. Además, agregaremos el parámetro opcional `(: filter)` a `/`, porque lo vamos a necesitar más adelante cuando tratemos de leer el parámetro `(: filter)` de la URL.

```js
<Route path="/(:filter)" component={App} />
```

Probablemente desees eliminar el hash (#) de la URL (por ejemplo: `http://localhost:3000/#/?_k=4sbb0i`). Para hacer esto, necesitarás también importar `browserHistory` de React Router:

```js
import { Router, Route, browserHistory } from 'react-router';
```

Y pasarlo al `<Router />` para eliminar el hash de la URL:

```js
<Router history={browserHistory}>
  <Route path="/(:filter)" component={App} />
</Router>
```

A menos que sea requisito apoyar navegadores antiguos como IE9, siempre puedes usar `browserHistory`.

#### `components/Root.js`
``` js
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { Router, Route, browserHistory } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/(:filter)" component={App} />
    </Router>
  </Provider>
);

Root.propTypes = {
  store: PropTypes.object.isRequired,
};

export default Root;
```

También necesitaremos refactorizar `index.js` para renderizar el componente `<Root />` al DOM.

#### `index.js`
```js
import React from 'react';
import { render } from 'react-dom'
import { createStore } from 'redux'
import todoApp from './reducers'
import Root from './components/Root'

let store = createStore(todoApp)

render(
  <Root store={store} />,
  document.getElementById('root')
)
```

## Navegación con React Router

React Router viene con un componente [`<Link />`](https://github.com/ReactTraining/react-router/blob/v3/docs/API.md#link) que te permite navegar entre tu aplicación. En nuestro ejemplo, podemos agrupar `<Link />` con un nuevo componente contenedor `<FilterLink />` para cambiar dinámicamente la URL. La propiedad `activeStyle = {}` nos permite aplicar un estilo al estado activo.

#### `containers/FilterLink.js`
```js
import React from 'react';
import { Link } from 'react-router';

const FilterLink = ({ filter, children }) => (
  <Link
    to={filter === 'SHOW_ALL' ? '/' : filter}
    activeStyle={{
      textDecoration: 'none',
      color: 'black'
    }}
  >
    {children}
  </Link>
);

export default FilterLink;
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
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
);

export default Footer
```

Ahora, si haces clic en `<FilterLink />` verás que su URL cambiará entre `'/SHOW_COMPLETED'`, `'/ SHOW_ACTIVE'` y `'/'`. Incluso si retrocedes usando el navegador, se utilizará el historial de tu navegador y efectivamente irá al URL anterior.

## Leyendo desde la URL

Actualmente, la lista de tareas no filtra, incluso después de que se haya cambiado la URL. Esto se debe a que estamos filtrando desde `<VisibleTodoList />`, `mapStateToProps()` sigue vinculado al `state` y no a la URL. `MapStateToProps` tiene un segundo argumento opcional, `ownProps`, que es un objeto con todos los *props* pasados a `<VisibleTodoList />`

#### `containers/VisibleTodoList.js`
```js
const mapStateToProps = (state, ownProps) => {
  return {
    todos: getVisibleTodos(state.todos, ownProps.filter) // anteriormente era 
    //getVisibleTodos(state.todos, state.visibilityFilter)
  };
};
```

En este momento no estamos pasando nada a `<App />` así que `ownProps` es un objeto vacío. Para filtrar nuestros To-dos según la URL, queremos pasar los parámetros de URL a `<VisibleTodoList />`.

Cuando previamente escribimos: `<Route path="/(:filter)" component={App} />`, esto puso a nuestra disposición dentro de `App` una propiedad, `params`.

La propiedad `params` es un objeto con cada parámetro especificado en la url. *Por ejemplo: `params` será igual a `{filter: 'SHOW_COMPLETED'}` si estamos navegando a `localhost:3000/SHOW_COMPLETED`. Ahora podemos leer la URL desde `<App>`.*

Nota que estamos usando [*ES6 destructuring*](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) en las propiedades para mover `params` a `<VisibleTodoList />`.

#### `components/App.js`
```js
const App = ({ params }) => {
  return (
    <div>
      <AddTodo />
      <VisibleTodoList
        filter={params.filter || 'SHOW_ALL'}
      />
      <Footer />
    </div>
  );
};
```

## Próximos pasos

Ahora que sabes cómo hacer el enrutamiento básico, puedes obtener más información sobre el [API React Router](https://github.com/reactjs/react-router/tree/v3/docs/)

>##### Nota sobre otras bibliotecas de enrutamiento

>*Redux Router* es una biblioteca experimental, que te permite mantener completamente el estado de tu URL dentro de su store Redux. Tiene la misma API del API React Router pero tiene un apoyo de la comunidad más reducido que react-router.

>*React Router Redux* crea un vínculo entre tu aplicación redux y react-router y los mantiene sincronizados. Sin este vínculo, no podrás retroceder las acciones usando *Time Travel*. A menos que lo necesites, React Router y Redux pueden funcionar completamente por separados.
