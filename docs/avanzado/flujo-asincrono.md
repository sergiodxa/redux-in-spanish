# Flujo asíncrono

Sin [middlewares](middleware.md), Redux solo soporta [flujos de datos síncronos](../basico/flujo-de-datos.md). Es lo que obtienes por defecto con [`createStore()`](../api/create-store.md).

Seguramente mejoraste [`createStore()`](../api/create-store.md) con [`applyMiddleware()`](../api/apply-middleware.md). No es necesario, pero te permite [usar acciones asíncrona de forma más fácil](./acciones-asincronas.md).

Middlewares asíncronos como [redux-thunk](https://github.com/gaearon/redux-thunk) o [redux-promise](https://github.com/acdlite/redux-promise) envuelven el método [`dispatch()`](../api/dispatch.md) y te permiten despachar otras cosas además de acciones, por ejemplo funciones o promesas. Cualquier middleware que uses entonces interpretan cualquier cosa que despaches, y entonces, pueden pasar acciones al siguiente middleware de la cadena. Por ejemplo, un middleware de promesas puede interceptar cualquier promesa y despachar acciones asíncronas en respuesta a estas.

Cuando el último middleware de la cadena despache una acción, tiene que ser un objeto plano. Acá es cono el [flujo de datos síncrono de Redux](http://redux.js.org/docs/basics/DataFlow.html) toma lugar.

Revisa [el código fuente del ejemplo asíncrono](http://redux.js.org/docs/advanced/ExampleRedditAPI.html).

# Siguientes pasos
Ahora que viste un ejemplo de que puede hacer un middleware en Redux, es hora de aprender como funcionan, y como puedes crear uno propio. Ve a la siguiente sección detallada sobre [Middlewares](./middleware.md).
