# Herencia

Redux tiene una herencia mixta. Es similar a ciertos patrones y tecnologías, pero también es diferente en varias formas importantes. Vamos a ver las similitudes y diferencias debajo.

### Flux

Puede Redux ser considerado una implementación de [Flux](https://facebook.github.io/flux/)?
[Sí](https://twitter.com/fisherwebdev/status/616278911886884864) y [no](https://twitter.com/andrestaltz/status/616270755605708800).

(No te preocupes, [los creadores de Flux](https://twitter.com/jingc/status/616608251463909376) [lo aprueba](https://twitter.com/fisherwebdev/status/616286955693682688), si es lo que necesitas saber.)

Redux se inspiro en muchas de las cualidades importantes de Flux. Como Flux, Redux te hace concentrar la lógica de actualización de modelos en una capa específica de tu aplicación ("stores" en Flux, "reducers" en Redux). En vez de dejar al código de la aplicación modificar los datos directamente, ambos te hacen describir cada mutación como objetos planos llamados "acciones".

A diferencia de Flux, **en Redux no existe el concepto de Dispatcher**. Esto es porque se basa en funciones puras en vez de emisores de ventos, y las funciones puras son fáciles de componer y no necesitan entidades adicionales para controlarlas. Dependiendo de como veas Flux, puedes ver esto tanto como una desviación o un detalle de implementación. Flux siempre fue [descripto como `(state, action) => state`](https://speakerdeck.com/jmorrell/jsconf-uy-flux-those-who-forget-the-past-dot-dot-dot-1). En ese sentido, Redux es una verdadera arquitectura Flux, pero más simple gracias a las funciones puras.

Otra diferencia importante con Flux es que **Redux asume que nunca modificas los datos**. Puede usar objetos planos o arrays para tu estado y esta perfecto, pero modificarlos dentro de los reducers no es recomendable. Siempre deberías devolver un nuevo objeto, lo cual es simple con la [propuesta del operador spread](../recetas/usando-el-operador-spread.md), o con librerías como [Immutable](https://facebook.github.io/immutable-js).

Mientras que es tecnicamente *posible* escribir [reducers impuros](https://github.com/reactjs/redux/issues/328#issuecomment-125035516) que modifican los datos en algunos por razones de rendimiento, desalentamos activamente hacer eso. Características de desarrollo como time travel, registrar/rehacer, o hot reloading pueden romperse. Además características como inmutabilidad no parecen causar problemas de rendimiento en aplicaciones reales, ya que, como [Om](https://github.com/omcljs/om) demostró, incluso si pierdes la posibilidad de usar asignación en objetos, todavía ganas por evitar re-renders y recalculos caros, debido a que sabes exactamente que cambio gracias a los reducers puros.

### Elm

[Elm](http://elm-lang.org/) es un lenguaje de programación funcional inspirado por Haskell y creado por [Evan Czaplicki](https://twitter.com/czaplic). Utiliza [una arquitectura 'modelo vista actualizador"](https://github.com/evancz/elm-architecture-tutorial/), donde el actualizador tiene el siguiente formato: `(action, state) => state`. Los "actualizadores" de Elm sirven para el mismo propósito que los reducers en Redux.

A diferencia de Redux, Elm es un lenguaje, así que puede beneficiarse de de muchas cosas como pureza forzada, tipado estático, inmutabilidad, y coincidencia de patrones (usando la expresión `case`). Incluso si no planeas usar Elm, deberías leer sobre la arquitectura de Elm, y jugar con eso. Hay un interesante [playground de librerías de JavaScript que implementa ideas similares](https://github.com/paldepind/noname-functional-frontend-framework). ¡Debemos mirar ahí por inspiración en Redux! Una forma de acercarnos al tipado estático de Elm es [usando una solución de tipado gradual como Flow](https://github.com/reactjs/redux/issues/290).

### Immutable

[Immutable](https://facebook.github.io/immutable-js) es una librería de JavaScript que implementa estructuras de datos persistentes. Es una API JavaScript idiomática y performante.

Immutable y la mayoría de las librerías similares son ortogonales a Redux. ¡Sientete libre de usarlos juntos!

**Redux no se preocupa por si guardas el estado en objetos planos, objetos de Immutable o cualquier otra cosa.** Probablemente quieras un mecanismo de (de)serialización para crear aplicaciones universales y rehidratar su estado desde el servidor, pero aparte de eso, puedes usar cualquier librería de almacenamiento de datos *mientras soporte inmutabilidad*. Por ejemplo, no tiene sentido usar Backbone para tu estado de Redux, ya que estos son mutables.

Ten en cuenta que que incluso si tu librería inmutable soporta punteros, no deberías usarlos en una aplicación de Redux. Todo el árbol de estado debería ser considerado de solo lectura, y deberías usar Redux para actualizar el estado, y suscribirse a los cambios. Por lo tanto, actualizar mediante recuerdos no tiene sentido en Redux. **Si tu único caso de uso para los punteros es desacoplar el árbol de estado del el árbolde UI y gradualmente refinar tus punteros, deberías revisar los selectores mejor.** Los selectores son funciones getter combinables. Mira [reselect](http://github.com/faassen/reselect) para una muy buena y concisa implementación de selectores combinables.

### Baobab

[Baobab](https://github.com/Yomguithereal/baobab) es otra popular librería para implementar inmutabilidad para actualizar objetos planos en JavaScript. Aunque puedes usarlo con Redux, hay muy pocos beneficios de usarlos juntos.

La mayorías de las funcionalidades que provee Baobab están relacionados con actualizar los datos con punteros, pero Redux impone que la única forma de actualizar los datos es despachando acciones. Por lo tanto, resuelven el mismo problema de forma diferente, y no se complementar uno con otro.

A diferencia de Immutable, Baobab todavía no implemente ninguna estructura de datos especialmente eficiente, así que no ganas nada realmente por usarlo junto a Redux. Es más fácil simplemente usar objetos planos en su lugar.

### Rx

[Reactive Extensions](https://github.com/Reactive-Extensions/RxJS) (y su [reescritura moderna](https://github.com/ReactiveX/RxJS) en proceso) es una formas magnífica de manejar la complejidad de aplicaciones asíncronas. De hecho [hay un esfuerzo de crear una librería para controlar la interacción entre humano y computadora como observables independientes](http://cycle.js.org).

¿Tiene sentido usar Redux junto con Rx? ¡Seguro! Funcionan genial juntos. Por ejemplo, es fácil exponer el store de Redux como un observable:

```javascript
function toObservable(store) {
  return {
    subscribe({ onNext }) {
      let dispose = store.subscribe(() => onNext(store.getState()));
      onNext(store.getState());
      return { dispose };
    },
  };
}
```

De forma similar, puedes componer diferentes streams asíncronos para convertirlos en acciones antes de enviarlos al `store.dispatch()`.

La pregunta es: de verdad necesitas REdux si ya usar Rx? Probablemente no. No es dificil [reimplementar Redux en Rx](https://github.com/jas-chen/rx-redux). Algunos dicen, que son solo 2 líneas usando el método `.scan()` de Rx. ¡Y probablemente lo sea!

Si tienes dudas, revisa el código fuente de Redux (no hay mucho ahí), así como su ecosistema (por ejemplo, [las herramientas de desarrolladores](https://github.com/gaearon/redux-devtools)). Si no te interesa tanto eso y quieres que los datos reactivos simplemente fluyan, probablemente quieras usar algo como [Cycle](http://cycle.js.org) en su lugar, o incluso combinarlos con Redux. ¡Déjanos saber como resulta eso!
