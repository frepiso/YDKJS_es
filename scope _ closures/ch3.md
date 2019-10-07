# You Don't Know JS: Scope & Closures
# Chapter 3: Alcance de Función vs. Alcance del bloque

Como exploramos en el capítulo 2, el alcance consiste en una serie de "burbujas" donde cada una actúa como un contenedor, en el que se declaran los identificadores (variables, funciones). Estas burbujas se anidan ordenadamente una dentro de la otra, y esta anidación se define en el momento del autor, al escribirse el código.

¿Pero qué hace exactamente una nueva burbuja? ¿Es sólo la función? ¿Pueden otras estructuras en JavaScript crear burbujas de alcance?

##  Alcance de Funciones

La respuesta más común a estas preguntas es que JavaScript tiene un alcance basado en funciones. Es decir, cada función que declara crea una burbuja para sí misma, pero ninguna otra estructura crea sus propias burbujas de alcance. Como veremos en un momento, esto no es del todo cierto.

Pero primero, exploremos el alcance de la función y sus implicaciones.

Considere este código:

```js
function foo(a) {
	var b = 2;

	// some code

	function bar() {
		// ...
	}

	// more code

	var c = 3;
}
```

En este fragmento, la burbuja de alcance para `foo(..)` incluye los identificadores `a`, `b`, `c` y `bar`. **No importa** *donde* aparece una declaración dentro del scope, la variable o función pertenece a la burbuja de alcance que la contiene independientemente. Exploraremos cómo funciona exactamente *eso* en el siguiente capítulo.

`bar(..)` tiene su propia burbuja de alcance. Lo mismo ocurre con el scope global, que tiene un solo identificador adjunto: `foo`

Como `a`, `b`, `c` y `bar` pertenecen a la burbuja de alcance de `foo(..)`, no son accesibles desde fuera de `foo(..)`. Es decir, el siguiente código daría como resultado errores `ReferenceError`, ya que los identificadores no están disponibles para el alcance global:

```js
bar(); // fails

console.log( a, b, c ); // all 3 fail
```

Sin embargo, todos estos identificadores (`a`, `b`, `c`, `foo` y `bar`) son accesibles *dentro* de `foo(..)`, y de hecho también están disponibles dentro de `bar(..)` (asumiendo que no hay declaraciones de identificador que hagan sombra dentro de `bar(..)`).

El alcance de la función fomenta la idea de que todas las variables pertenecen a la función, y se pueden usar y reutilizar en toda la función (y, de hecho, se puede acceder incluso a los ámbitos anidados). Este enfoque de diseño puede ser bastante útil, y ciertamente puede hacer un uso completo de la naturaleza "dinámica" de las variables de JavaScript para tomar valores de diferentes tipos según sea necesario.

Por otro lado, si no toma precauciones cuidadosas, las variables que existen en la totalidad de un alcance pueden llevar a algunos escollos inesperados.

## Escondiendose en un scope simple

La forma tradicional de pensar acerca de las funciones es que usted declara una función y luego agrega un código dentro de ella. Pero el pensamiento inverso es igualmente poderoso y útil: tome cualquier sección arbitraria del código que haya escrito y envuelva una declaración de función a su alrededor, que en efecto "oculta" el código.

El resultado práctico es crear una burbuja de alcance alrededor del código en cuestión, lo que significa que cualquier declaración (variable o función) en ese código ahora se vinculará al alcance de la nueva función de ajuste, en lugar del alcance previo. En otras palabras, puede "ocultar" variables y funciones encerrándolas en el alcance de una función.

¿Por qué "ocultar" variables y funciones sería una técnica útil?

Hay una variedad de razones que motivan esta ocultación basada en el alcance. Tienden a surgir del principio de diseño de software "Principio de privilegio mínimo" [^note-leastprivilege], también a veces llamado "Autoridad mínima" o "Exposición mínima". Este principio establece que en el diseño de software, como la API para un módulo/objeto, debe exponer solo lo que es mínimamente necesario y "ocultar" todo lo demás.

Este principio se extiende a la elección de qué ámbito contener variables y funciones. Si todas las variables y funciones estuvieran en el ámbito global, por supuesto serían accesibles a cualquier ámbito anidado. Pero esto violaría el principio de "... mínimo" en el sentido de que (probablemente) está exponiendo muchas variables o funciones que de otra manera debería mantener en privado, ya que el uso adecuado del código desalentaría el acceso a esas variables/funciones.

Por ejemplo:

```js
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```

En este fragmento, la variable `b` y la función `doSomethingElse(..)` son probablemente detalles "privados" de cómo `doSomething(..)` hace su trabajo. Dar acceso de alcance a `b` y `doSomethingElse(..)` no solo es innecesario sino también posiblemente "peligroso", ya que se pueden usarse de manera inesperada, intencionalmente o no, y esto puede violar lo que esperamos de `doSomething (..)`.

Un diseño más "apropiado" ocultaría estos detalles privados dentro del alcance de `doSomething(..)`, como:

```js
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

Ahora, `b` y `doSomethingElse(..)` no son accesibles a ninguna influencia externa, en su lugar están controlados solo por `doSomething (..)`. La funcionalidad y el resultado final no se han visto afectados, pero el diseño mantiene privados los detalles privados, lo que generalmente se considera un mejor software.

### Prevención de Colisiones

Otro beneficio de "ocultar" las variables y las funciones dentro de un alcance es evitar la colisión involuntaria entre dos identificadores diferentes con el mismo nombre pero diferentes usos previstos. La colisión resulta a menudo en sobrescritura inesperada de valores.

Por ejemplo:

```js
function foo() {
	function bar(a) {
		i = 3; // changing the `i` in the enclosing scope's for-loop
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, infinite loop ahead!
	}
}

foo();
```

La asignación `i = 3` dentro de `bar(..)` sobrescribe, inesperadamente, el `i` que se declaró en `foo(..)` en el bucle for. En este caso, dará como resultado un bucle infinito, porque `i` se establece en un valor fijo de `3` y permanecerá para siempre `< 10`.

La asignación dentro de `bar(..)` necesita declarar una variable local para usar, independientemente del nombre del identificador que se elija. `var i = 3;` solucionaría el problema (y crearía la declaración de "variable sombreada" mencionada anteriormente para `i`). Una opción *adicional*, no alternativa, es elegir otro nombre de identificador por completo, como `var j = 3;`. Pero, naturalmente, el diseño de su software puede requerir el mismo nombre de identificador, por lo que utilizar el ámbito para "ocultar" su declaración interna es su mejor/única opción en ese caso.

#### "Espacios de nombres" globales / Global "Namespaces"

Un ejemplo particularmente fuerte de (probable) colisión de variables ocurre en el ámbito global. Múltiples librerias cargadas en su programa pueden chocar fácilmente entre sí si no ocultan adecuadamente sus funciones y variables internas/privadas.

Estas librerias normalmente crearán una declaración de variable única, a menudo un objeto, con un nombre suficientemente exclusivo, en el scope global. Este objeto se usa luego como un "namespace"  para esa libreria, donde todas las exposiciones específicas de la funcionalidad se hacen como propiedades de ese objeto (namespace), en lugar de como identificadores de nivel superior de ámbito léxico.

Por ejemplo:

```js
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

#### Gestión de Módulos

Otra opción para evitar colisiones es el enfoque más moderno del "módulo", que utiliza cualquiera de los varios administradores de dependencias. Al usar estas herramientas, ninguna biblioteca agrega ningún identificador al alcance global, sino que se requiere que sus identificadores se importen explícitamente a otro alcance específico mediante el uso de los diversos mecanismos del administrador de dependencias.

Debe observarse que estas herramientas no poseen una funcionalidad "mágica" que está exenta de las reglas de alcance léxicas. Simplemente usan las reglas de alcance como se explica aquí para hacer cumplir que no se inyectan identificadores en ningún alcance compartido y, en cambio, se mantienen en ámbitos privados, no susceptibles de colisión, lo que evita colisiones de alcance accidentales.

Como tal, puede codificar a la defensiva y lograr los mismos resultados que los administradores de dependencias sin necesidad de usarlos, si así lo desea. Consulte el capítulo 5 para obtener más información sobre el patrón del módulo.

## Funciones como alcances

Hemos visto que podemos tomar cualquier fragmento de código y envolver una función a su alrededor, y que efectivamente "oculta" cualquier variable incluida o declaración de función del ámbito externo dentro del ámbito interno de esa función.

Por ejemplo:

```js
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

Si bien esta técnica "funciona", no es necesariamente muy ideal. Hay algunos problemas que introduce. La primera es que tenemos que declarar una función nombrada `foo ()`, lo que significa que el nombre del identificador `foo` en sí mismo "contamina" el ámbito que lo contiene (global, en este caso). También tenemos que llamar explícitamente a la función por su nombre (`foo()`) para que el código envuelto realmente se ejecute.

Sería más ideal si la función no necesitara un nombre (o, más bien, el nombre no contaminara el ámbito de aplicación), y si la función pudiera ejecutarse automáticamente.

Afortunadamente, JavaScript ofrece una solución a ambos problemas.

```js
var a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

Vamos a desglosar lo que está pasando aquí.

Primero, observe que la declaración de la función comienza envuelta con `(función...` en lugar de solo `función...`. Si bien esto puede parecer un detalle menor, en realidad es un cambio importante. En lugar de tratar la función como un *declaración estándar*, la función se trata como una *función-expresión*.

**Nota:** La forma más fácil de distinguir declaración vs. expresión es la posición de la palabra "función" en la declaración (no solo una línea, sino una declaración distinta). Si "función" es lo primero en la declaración, entonces es una declaración de función. De lo contrario, es una expresión de función.

La diferencia clave que podemos observar aquí entre una declaración de función y una expresión de función está relacionado a dónde está enlazado su nombre como un identificador.

Compara los dos fragmentos anteriores. En el primer fragmento de código, el nombre `foo` está enlazado en el scope que lo contiene, y lo llamamos directamente con `foo()`. En el segundo fragmento, el nombre `foo` no está enlazado en el scope que lo incluye, sino que está enlazado solo dentro de su propia función.

En otras palabras, `(function foo(){ .. })` como expresión significa que el identificador `foo` se encuentra *solo* en el scope donde indica `..`, no en el scope externo. Ocultar el nombre `foo` dentro de sí mismo significa que no contamina el ámbito de acceso innecesariamente.

### Anonimas vs. Nombradas

Probablemente esté más familiarizado con las expresiones de funciones como parámetros de funciones de llamada (callbacks), como:

```js
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

Esto se denomina "expresión de función anónima", porque `function() ...` no tiene ningún identificador de nombre. Las expresiones de funciones pueden ser anónimas, pero las declaraciones de funciones no pueden omitir el nombre, lo que sería una gramática JS ilegal.

Las expresiones de funciones anónimas son rápidas y fáciles de escribir, y muchas bibliotecas y herramientas tienden a fomentar este estilo idiomático de código. Sin embargo, tienen varios inconvenientes a considerar:

1. Las funciones anónimas no tienen un nombre útil para mostrar en las trazas de la pila (stack traces), lo que puede dificultar la depuración.

2. Sin un nombre, si la función necesita referirse a sí misma, para recursión, etc., la referencia `arguments.callee` (**deprecated**) es desafortunadamente necesaria. Otro ejemplo de la necesidad de auto referenciarse es cuando una función de controlador de eventos desea desenlazarse después de que se dispara.

3. Las funciones anónimas omiten un nombre que a menudo es útil para proporcionar un código más legible/comprensible. Un nombre descriptivo ayuda a auto-documentar el código en cuestión.

**Las expresiones de función en línea** son potentes y útiles -- la cuestión de anónimo contra nombrado no resta valor a eso. Proporcionar un nombre para su expresión de función aborda de manera bastante efectiva todos estos inconvenientes, pero no tiene desventajas tangibles. La mejor práctica es nombrar siempre las expresiones de sus funciones:

```js
setTimeout( function timeoutHandler(){ // <-- Look, I have a name!
	console.log( "I waited 1 second!" );
}, 1000 );
```

### Invocar expresiones de función inmediatamente

```js
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

Ahora que tenemos una función como expresión gracias a envolverla en un par `()`, podemos ejecutar esa función agregando otro `()` al final, como `(function foo(){..})()`. El primer par envoltorio `()` convierte la función en una expresión, y el segundo `()` ejecuta la función.

Este patrón es tan común que hace unos años, la comunidad acordó un término: **IIFE**, que significa **I**mmediately **I**nvoked **F**unction **E**xpression.

Por supuesto, IIFE no necesita nombres, necesariamente -- la forma más común de IIFE es usar una expresión de función anónima. Aunque ciertamente menos común, nombrar un IIFE tiene todos los beneficios antes mencionados sobre las expresiones de funciones anónimas, por lo que es una buena práctica a adoptar.

```js
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

Hay una ligera variación en la forma tradicional de IIFE, que algunos prefieren: `(function(){ .. }())`. Mire de cerca para ver la diferencia. En la primera forma, la expresión de la función está envuelta en `()`, y luego el par invocador `()` está en el exterior justo después de ella. En la segunda forma, el par invocador `()` se mueve al interior del par envolvente externo `()`.

Estas dos formas son idénticas en funcionalidad. **Es puramente una elección estilística que prefieres.**

Otra variación de los IIFE que es bastante común es usar el hecho de que, de hecho, son solo llamadas a funciones y pasar argumentos.

Por ejemplo:

```js
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

Pasamos en la referencia de objeto `window`, pero nombramos el parámetro` global`, de modo que tengamos una delineación estilística clara para las referencias globales frente a las no globales. Por supuesto, puede pasar cualquier cosa del ámbito que desee y puede asignar a los parámetros el nombre que más le convenga. Esto es principalmente una elección estilística.

Otra aplicación de este patrón aborda la inquietud de que el identificador predeterminado `undefined` podría tener su valor sobrescrito incorrectamente, lo que provocaría resultados inesperados. Al nombrar un parámetro `undefined`, pero sin pasar ningún valor por ese argumento, podemos garantizar que el identificador` undefined` es de hecho el valor indefinido en un bloque de código:

```js
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

	var a;
	if (a === undefined) {
		console.log( "Undefined is safe here!" );
	}

})();
```

Otra variación más del IIFE invierte el orden de las cosas, donde la función a ejecutar se da en segundo lugar, *después* de la invocación y los parámetros para pasar a ella. Este patrón se utiliza en el proyecto UMD (Universal Module Definition). Algunas personas consideran que es un poco más limpio de entender, aunque es un poco más verboso.

```js
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

La expresión de la función `def` se define en la segunda mitad del fragmento, y luego se pasa como un parámetro (también llamado `def`) a la función `IIFE` definida en la primera mitad del fragmento. Finalmente, se invoca el parámetro `def` (la función), pasando` window` como el parámetro `global`.

## Bloques como Scopes

Si bien las funciones son la unidad de alcance más común y ciertamente la más amplia de los enfoques de diseño en la mayoría de JS en circulación, otras unidades de alcance son posibles, y el uso de estas otras unidades de alcance puede llevar incluso a una manera  más limpia y mejor de mantener el código.

Muchos lenguajes distintos de JavaScript son compatibles con Block Scope, por lo que los desarrolladores de esos lenguajes están acostumbrados a la mentalidad, mientras que aquellos que solo han trabajado principalmente en JavaScript pueden encontrar el concepto un poco extraño.

Pero incluso si nunca ha escrito una sola línea de código en forma de bloque, probablemente aún esté familiarizado con este lenguaje extremadamente común en JavaScript:

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

Declaramos la variable `i` directamente dentro de la cabecera del bucle for, lo más probable es que nuestra *intención* es usar `i` solo dentro del contexto de ese bucle for, y esencialmente ignoramos el hecho de que la variable realmente se encamina al scope que lo engloba (función o global).

De eso se trata el bloqueo de alcance. Declarar las variables lo más cerca posible, lo más local posible, a donde se utilizarán. Otro ejemplo:

```js
var foo = true;

if (foo) {
	var bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}
```

Estamos usando una variable `bar` solo en el contexto de la sentencia if, por lo que tiene sentido que lo declaremos dentro del bloque if. Sin embargo, cuando declaramos las variables no es relevante cuando usamos `var`, porque siempre pertenecerán al scope que lo engloba. Este fragmento de código es esencialmente un "falso" scope de bloque, por razones estilísticas, y se basa en el auto-cumplimiento para no usar accidentalmente `bar` en otro lugar en ese ámbito.

El scope de bloque es una herramienta para extender el "Principle of Least ~~Privilege~~ Exposure" [^note-leastprivilege] de ocultar la información en funciones a ocultar información en bloques de nuestro código.

Considere el ejemplo del bucle for de nuevo:

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

¿Por qué contaminar todo el alcance de una función con la variable `i` que solo se va a usar (o que solo *debería ser*, al menos) utilizada para el bucle for?

Pero lo que es más importante, es posible que los desarrolladores prefieran *verificar* ellos mismos contra el la (re)utilización accidental de variables fuera de su propósito previsto, como el hecho de que se emita un error sobre una variable desconocida si intenta usarla en el lugar equivocado. El alcance del bloque (si fuera posible) para la variable `i` haría que `i` esté disponible solo para el bucle for, causando un error si se accede a `i` en otra parte de la función. Esto ayuda a garantizar que las variables no se reutilicen de manera confusa o difícil de mantener.

Pero, la triste realidad es que, en la superficie, JavaScript no tiene utilidades para scope de bloque.

Es decir, hasta que caves un poco más.

### `with`

Aprendimos sobre `with` en el capítulo 2. Si bien es una construcción mal vista, *es* un ejemplo de (una forma de) alcance de bloque, en que el alcance que se crea a partir del objeto solo existe durante el tiempo de vida de esa declaración `with`, y no en el ámbito de aplicación.

We learned about `with` in Chapter 2. While it is a frowned upon construct, it *is* an example of (a form of) block scope, in that the scope that is created from the object only exists for the lifetime of that `with` statement, and not in the enclosing scope.

### `try/catch`

It's a *very* little known fact that JavaScript in ES3 specified the variable declaration in the `catch` clause of a `try/catch` to be block-scoped to the `catch` block.

For instance:

```js
try {
	undefined(); // illegal operation to force an exception!
}
catch (err) {
	console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

As you can see, `err` exists only in the `catch` clause, and throws an error when you try to reference it elsewhere.

**Note:** While this behavior has been specified and true of practically all standard JS environments (except perhaps old IE), many linters seem to still complain if you have two or more `catch` clauses in the same scope which each declare their error variable with the same identifier name. This is not actually a re-definition, since the variables are safely block-scoped, but the linters still seem to, annoyingly, complain about this fact.

To avoid these unnecessary warnings, some devs will name their `catch` variables `err1`, `err2`, etc. Other devs will simply turn off the linting check for duplicate variable names.

The block-scoping nature of `catch` may seem like a useless academic fact, but see Appendix B for more information on just how useful it might be.

### `let`

Thus far, we've seen that JavaScript only has some strange niche behaviors which expose block scope functionality. If that were all we had, and *it was* for many, many years, then block scoping would not be terribly useful to the JavaScript developer.

Fortunately, ES6 changes that, and introduces a new keyword `let` which sits alongside `var` as another way to declare variables.

The `let` keyword attaches the variable declaration to the scope of whatever block (commonly a `{ .. }` pair) it's contained in. In other words, `let` implicitly hijacks any block's scope for its variable declaration.

```js
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```

Using `let` to attach a variable to an existing block is somewhat implicit. It can confuse you if you're not paying close attention to which blocks have variables scoped to them, and are in the habit of moving blocks around, wrapping them in other blocks, etc., as you develop and evolve code.

Creating explicit blocks for block-scoping can address some of these concerns, making it more obvious where variables are attached and not. Usually, explicit code is preferable over implicit or subtle code. This explicit block-scoping style is easy to achieve, and fits more naturally with how block-scoping works in other languages:

```js
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

We can create an arbitrary block for `let` to bind to by simply including a `{ .. }` pair anywhere a statement is valid grammar. In this case, we've made an explicit block *inside* the if-statement, which may be easier as a whole block to move around later in refactoring, without affecting the position and semantics of the enclosing if-statement.

**Note:** For another way to express explicit block scopes, see Appendix B.

In Chapter 4, we will address hoisting, which talks about declarations being taken as existing for the entire scope in which they occur.

However, declarations made with `let` will *not* hoist to the entire scope of the block they appear in. Such declarations will not observably "exist" in the block until the declaration statement.

```js
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

#### Garbage Collection

Another reason block-scoping is useful relates to closures and garbage collection to reclaim memory. We'll briefly illustrate here, but the closure mechanism is explained in detail in Chapter 5.

Consider:

```js
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

The `click` function click handler callback doesn't *need* the `someReallyBigData` variable at all. That means, theoretically, after `process(..)` runs, the big memory-heavy data structure could be garbage collected. However, it's quite likely (though implementation dependent) that the JS engine will still have to keep the structure around, since the `click` function has a closure over the entire scope.

Block-scoping can address this concern, making it clearer to the engine that it does not need to keep `someReallyBigData` around:

```js
function process(data) {
	// do something interesting
}

// anything declared inside this block can go away after!
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox.

#### `let` Loops

A particular case where `let` shines is in the for-loop case as we discussed previously.

```js
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
```

Not only does `let` in the for-loop header bind the `i` to the for-loop body, but in fact, it **re-binds it** to each *iteration* of the loop, making sure to re-assign it the value from the end of the previous loop iteration.

Here's another way of illustrating the per-iteration binding behavior that occurs:

```js
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}
```

The reason why this per-iteration binding is interesting will become clear in Chapter 5 when we discuss closures.

Because `let` declarations attach to arbitrary blocks rather than to the enclosing function's scope (or global), there can be gotchas where existing code has a hidden reliance on function-scoped `var` declarations, and replacing the `var` with `let` may require additional care when refactoring code.

Consider:

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	if (baz > bar) {
		console.log( baz );
	}

	// ...
}
```

This code is fairly easily re-factored as:

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	// ...
}

if (baz > bar) {
	console.log( baz );
}
```

But, be careful of such changes when using block-scoped variables:

```js
var foo = true, baz = 10;

if (foo) {
	let bar = 3;

	if (baz > bar) { // <-- don't forget `bar` when moving!
		console.log( baz );
	}
}
```

See Appendix B for an alternate (more explicit) style of block-scoping which may provide easier to maintain/refactor code that's more robust to these scenarios.

### `const`

In addition to `let`, ES6 introduces `const`, which also creates a block-scoped variable, but whose value is fixed (constant). Any attempt to change that value at a later time results in an error.

```js
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // block-scoped to the containing `if`

	a = 3; // just fine!
	b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## Review (TL;DR)

Functions are the most common unit of scope in JavaScript. Variables and functions that are declared inside another function are essentially "hidden" from any of the enclosing "scopes", which is an intentional design principle of good software.

But functions are by no means the only unit of scope. Block-scope refers to the idea that variables and functions can belong to an arbitrary block (generally, any `{ .. }` pair) of code, rather than only to the enclosing function.

Starting with ES3, the `try/catch` structure has block-scope in the `catch` clause.

In ES6, the `let` keyword (a cousin to the `var` keyword) is introduced to allow declarations of variables in any arbitrary block of code. `if (..) { let a = 2; }` will declare a variable `a` that essentially hijacks the scope of the `if`'s `{ .. }` block and attaches itself there.

Though some seem to believe so, block scope should not be taken as an outright replacement of `var` function scope. Both functionalities co-exist, and developers can and should use both function-scope and block-scope techniques where respectively appropriate to produce better, more readable/maintainable code.

[^note-leastprivilege]: [Principle of Least Privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege)
