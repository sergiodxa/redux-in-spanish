# Migrando a Redux

Redux no es un framework monolítico, sino un conjunto de contratos y [algunas funciones que hacen que todo funcione en conjunto](../api/README.md). La mayor parte de tu "código de Redux" ni siquiera va a hacer uso de la API de Redux, ya que la mayor parte del tiempo vas a crear funciones.

Esto hace fácil migrar a o desde Redux.

¡No queremos encerrarte!

## Desde Flux
Los [reducers](../glosario.md#reducer) capturan "la esencia" de los Stores de Flux, así que es posible migrar gradualmente de un proyecto Flux existente a uno de Redux, ya sea que uses [Flummox](http://github.com/acdlite/flummox), [Alt](http://github.com/goatslacker/alt), [Flux tradicional](https://github.com/facebook/flux) o cualquier otra librería de Flux.

También es posible hacer lo contrario y migrar de Redux a cualquier de estas siguiendo los siguiente pasos:

Tu proceso debería ser algo como esto:

* Crea una función llamada `createFluxStore(reducer)` que cree un store de Flux compatible con tu aplicación actual a partir de un reducer. Internamente debería ser similar a la implementación de [`createStore`](../api/create-store.md) ([código fuente](https://github.com/rackt/redux/blob/master/src/createStore.js)) de Redux. Su función dispatch solo debería llamar al `reducer` por cada acción, guardar el siguiente estado y emitir el cambio.
* Esto te permite gradualmente reescribir cada Store de Flux de tu aplicación como un reducer, pero todavía exportar `createFluxStore(reducer)` así el resto de tu aplicación no se entera de que esto está ocurriendo con los Stores de Flux.
* Mientras reescribes tus Stores, vas a darte cuenta que deberías evitar algunos anti-patrones de Flux como peticiones a APIs dentro del Store, o ejecutar acciones desde el Store. Tu código de Flux va a ser más fácil de entender cuando lo modifiques para que funcione en base a reducers.
* Cuando hayas portado todos los Stores de Flux para que funcionen con reducers, puedes reemplazar tu librería de Flux con un único Store de Redux, y combinar estos reducers que ya tienes usando [`combineReducers(reducers)`](../api/combine-reducers.md).
* Ahora lo único que falta es portar la UI para que use [react-redux](http://redux.js.org/docs/basics/UsageWithReact.html) o alguno similar.
* Finalmente, seguramente quieras usar cosas como middlewares para simplificar tu código asíncrono.
 
## Desde Backbone
La capa de modelos de Backbone es bastante diferente de Redux, así que no recomendamos combinarlos. Si es posible, lo mejor es reescribir la capa de modelos de tu aplicación desde cero en vez de conectar Backbone a Redux. Igualmente, si no es posible reescribirlo, capaz debería usar [backbone-redux](https://github.com/redbooth/backbone-redux) para migrar gradualmente, y mantener tu Store de Redux sincronizado con los modelos y colecciones de Backbone.
