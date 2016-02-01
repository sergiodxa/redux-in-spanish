# Flujo asíncrono

Sin [middlewares](middleware.md), Redux solo soporta [flujos de datos síncronos](../basico/flujo-de-datos.md). Es lo que obtienes por defecto con [`createStore()`](../api/create-store.md).

Seguramente mejoraste [`createStore()`](../api/create-store.md) con [`applyMiddleware()`](../api/apply-middleware.md). No es necesario, pero te permite [usar acciones asíncrona de forma más fácil](