# Glosario
Este es un glosario de los términos principales en Redux, junto a su tipo de dato. Los tipos están documentados usando la [notación Flow](http://flowtype.org/docs/quick-reference.html).

## Estado
```
type State = any
```
Estado (también llamado árbol de estado) es un termino general, pero en la API de Redux normalmente se refiere al valor de estado único que es manejado por el Store y devuelto por [`getState()`](http://redux.js.org/docs/api/Store.html#getState). Representa el estado de tu aplicación de Redux, normalmente es un objeto con muchas anidaciones.

Por convención, el estado a nivel superior es un objeto o algún tipo de colección llave-valor como un Map, pero técnicamente puede ser de cualquier tipo. Aun así, debes hacer tu mejor esfuerzo en mantener el estado serializable. No pongas nada dentro que no puedas fácilmente convertirlo a un JSON.

## Acción
```
type Action = object
```
Una acción es un objeto plano (POJO - Plan Old JavaScript Object) que representa una intención de modificar el estado. Las acciones son la única forma en que los datos llegan al store. Cualquier dato, ya sean eventos de UI, callbacks de red, u otros recursos como WebSockets eventualmente van a ser despachados como acciones.

Las acciones deben tener un campo `type` que indica el tipo de acción a realizar. Los tipos pueden ser definidos como constantes e importados desde otro módulo Es mejor usar strings como tipos en vez de [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) ya que los strings son serializables.

Aparte del `type`, la estructura de una acción depende de vos. Si estás interesado, revisa [Flux Standard Action](https://github.com/acdlite/flux-standard-action) para recomendaciones de como deberías estar estructurado una acción.

Revisa [acción asíncrona](#accion-asincrona) debajo.

## Reducer
```
type Reducer<S, A> = (state: S, action: A) => S
```
Un *reducer* (también llamado *función reductora*) es una función que acepta una acumulación y una valor y devuelve una nueva acumulación. Son usados para reducir una colección de valores a un único valor.

Los reducers no son únicos de Redux — son un concepto principal de la programación funcional. Incluso muchos lenguajes no funcionales, como JavaScript, tienen una API para reducción. En JavaScript, es [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce).

En Redux, el valor acumulado es el árbol de estado, y los valores que están siendo acumulados son acciones. Los reducers calculan el nuevo estado en base al anterior estado y la acción. Deben ser *funciones puras* — funciones que devuelven el mismo valor dados los mismos argumentos. Deben estar libres de efectos secundarios. Esto es lo que permite características increibles como hot reloading y time travel.

Los reducers son el concepto más importante en Redux.

*No hagas peticiones a APIs en los reducers.*

## Función `dispatch`
```
type BaseDispatch = (a: Action) => Action
type Dispatch = (a: Action | AsyncAction) => any
```




















