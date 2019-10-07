# You Don't Know JS: Scope & Closures
# Chapter 2: Lexical Scope

En el capítulo 1, definimos "scope" como el conjunto de reglas que gobiernan cómo el *Engine* puede buscar una variable por su nombre de identificador y encontrarla, ya sea en el *Scope* actual, o en cualquiera de los *Scope anidados* en el que está contenido.

Hay dos modelos predominantes de cómo funciona el scope. El primero de ellos es, con mucho, el más común, utilizado por la gran mayoría de los lenguajes de programación. Se llama **Lexical Scope**, y lo examinaremos en profundidad. El otro modelo, que todavía se usa en algunos lenguajes (como las secuencias de comandos Bash, algunos modos en Perl, etc.) se llama **Scope dinámico**.

El scope dinámico se trata en el Apéndice A. Lo menciono aquí solo para proporcionar un contraste con el scope léxico, que es el modelo de scope que emplea JavaScript.

## Lex-time

Como vimos en el Capítulo 1, la primera fase tradicional de un compilador de lenguaje estándar se llama lexing (también conocido como tokenización). Si recuerdas, el proceso lexing examina una cadena de caracteres de código fuente y asigna un significado semántico a los tokens como resultado de un análisis de estado completo.

Es este concepto el que proporciona la base para comprender qué es el scope léxico y de dónde proviene el nombre.

Para definirlo de forma circular, el scope léxico es el scope que se define en el momento de la lectura (lexing time). En otras palabras, el scope léxico se basa en dónde ha escrito usted en su código las variables y bloques de scope, y por lo tanto está (en su mayoría) escrito en piedra en el momento que el "lexer" procese su código.

**Nota:** Veremos en breve que hay algunas formas de hacer trampa en el scope léxico, modificándolo así una vez que el lexer haya pasado, pero están mal vistos. Se considera una buena práctica tratar el scope léxico como, de hecho, solo léxico y, por lo tanto, de naturaleza enteramente de tiempo de escritura (author-time).

Consideremos este bloque de código:

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

Hay tres ámbitos anidados inherentes en este ejemplo de código. Puede ser útil pensar en estos ámbitos como burbujas unas dentro de otras.

<img src="fig2.png" width="500">

**Burbujas 1** abarca el scope global, y tiene un solo identificador: `foo`.

**Burbujas 2** abarca el alcance de `foo`, que incluye los tres identificadores: `a`, `bar` y `b`.

**Burbujas 3** abarca el alcance de `bar`, e incluye solo un identificador: `c`.

Las burbujas de alcance se definen según dónde se escriben los bloques de scope, cuál está anidado dentro del otro, etc. En el próximo capítulo, analizaremos diferentes unidades de alcance, pero por ahora, supongamos que cada función crea una nueva burbuja de alcance.

La burbuja para `bar` está completamente contenida dentro de la burbuja para `foo`, porque (y solo porque) es donde elegimos definir la función `bar`.

Observe que estas burbujas anidadas están estrictamente anidadas. No estamos hablando de diagramas de Venn donde las burbujas pueden cruzar los límites. En otras palabras, ninguna burbuja de una función puede existir simultáneamente (parcialmente) dentro de las otras dos burbujas de alcance externo, al igual que ninguna función puede estar parcialmente dentro de cada una de las dos funciones padres.

### Búsquedas

La estructura y la ubicación relativa de estas burbujas de alcance explica completamente al *Engine* todos los lugares en los que necesita buscar para encontrar un identificador.

En el fragmento de código anterior, el *Engine* ejecuta la instrucción `console.log(..)` y busca las tres variables referenciadas `a`,` b` y `c`. Primero comienza con la burbuja de alcance más interna, el alcance de la función `bar(..)`. No encontrará `a` allí, por lo que sube un nivel, hasta la siguiente burbuja de alcance más cercana, el alcance de `foo(..) `. Encuentra `a` allí, y por eso usa ese `a`. Lo mismo para `b`. Pero `c`, se encuentra dentro de `bar(..) `.

Si hubiera habido una `c` tanto dentro de `bar(..) `como dentro de `foo (..)`, la declaración `console.log(..)` hubiera encontrado y usado la de `bar(..) `, nunca llegaría a la de `foo(..) `.

**La búsqueda de scope se detiene una vez que encuentra la primera coincidencia**. El mismo nombre de identificador se puede especificar en varias capas del ámbito anidado, lo que se denomina "sombreado" o "shadowing" (el identificador interno "hace sombra" al identificador externo). Independientemente del sombreado, la búsqueda de alcance siempre comienza en el alcance más interno que se está ejecutando en ese momento, y se abre camino hacia afuera/arriba hasta la primera coincidencia, y se detiene.

**Nota:** Las variables globales también son automáticamente propiedades del objeto global (`window` en los navegadores, etc.), por lo que *es* posible hacer referencia a una variable global no directamente por su nombre léxico, sino indirectamente como un propiedad referencia del objeto global.

```js
window.a
```

Esta técnica da acceso a una variable global que, de lo contrario, sería inaccesible debido a su sombreado. Sin embargo, no se puede acceder a las variables sombreadas no globales.

Independientemente de *dónde* se invoca una función, o incluso de *cómo* se invoca, su ámbito léxico está **solo** definido por el lugar donde se declaró la función.

El proceso de búsqueda de alcance léxico *solo* se aplica a identificadores de primera clase, como `a`,` b` y `c`. Si tuviera una referencia a `foo.bar.baz` en un fragmento de código, la búsqueda del ámbito léxico se aplicaría a la búsqueda del identificador `foo`, pero una vez que localice esa variable, las reglas de acceso a la propiedad del objeto asumirán el control para resolver las propiedades `bar` y` baz`, respectivamente.

## Trampas Léxicas

Si el ámbito léxico se define solo por el lugar donde se declara una función, que es totalmente una decisión en tiempo de escritura, ¿cómo podría haber una manera de "modificar" (o hacer trampa) el ámbito léxico en tiempo de ejecución?

JavaScript tiene dos mecanismos de este tipo. Ambos son igualmente mal vistos en la comunidad en general como malas prácticas para usar en su código. Pero los argumentos típicos en contra de ellos a menudo pierden el punto más importante: **Engañar el alcance léxico conduce a un peor rendimiento**.

Sin embargo, antes de explicar el problema de rendimiento, veamos cómo funcionan estos dos mecanismos.

### `eval`

La función `eval(..)` en JavaScript toma una cadena como un argumento, y trata el contenido de la cadena como si realmente se hubiera creado un código en ese punto del programa. En otras palabras, puede generar código dentro de su código creado mediante programación y ejecutar el código generado como si hubiera estado allí en el momento de escritura (author-time).

Al evaluar `eval(..)` (juego de palabras) desde ese punto de vista, debe quedar claro cómo `eval(..)` le permite modificar el entorno de alcance léxico haciendo trampa y simulando que ese código estuvo allí todo el tiempo.

En las siguientes líneas de código después de que se haya ejecutado un `eval (..)`, el *Engine* no "sabrá" o "cuidará" que el código anterior en cuestión se interpretó dinámicamente y, por lo tanto, modificó el entorno de alcance léxico. El *Engine* simplemente realizará sus búsquedas de alcance léxico como siempre lo hace.

Considere el siguiente código:

```js
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1 3
```

La cadena `"var b = 3;"` se trata, en el punto de la llamada `eval(..)`, como código que hubiera estado ahí todo el tiempo. Debido a que el código pasa a declarar una nueva variable `b`, modifica el alcance léxico existente de `foo(..)`. De hecho, como se mencionó anteriormente, este código crea realmente la variable `b` dentro de `foo (..)` que sombrea el `b` que se declaró en el ámbito externo (global).

Cuando se produce la llamada `console.log(..)`, encuentra tanto `a` como `b` en el alcance de `foo(..)`, y nunca encuentra el `b` externo. Por lo tanto, imprimimos "1 3" en lugar de "1 2" como normalmente hubiera sido el caso.

**Nota:** En este ejemplo, por razones de simplicidad, la cadena de "código" que pasamos era un literal fijo. Pero podría haberse creado fácilmente mediante programación agregando caracteres en función de la lógica de su programa. `eval(..)` se usa generalmente para ejecutar código creado dinámicamente, ya que la evaluación dinámica de un código esencialmente estático a partir de un literal de cadena no proporcionaría un beneficio real al solo crear el código directamente.

De forma predeterminada, si una cadena de código que `eval(..)` ejecuta contiene una o más declaraciones (variables o funciones), esta acción modifica el alcance léxico existente en el que reside `eval(..)`. Técnicamente, `eval (..)` puede invocarse "indirectamente", a través de varios trucos (más allá de nuestra discusión aquí), lo que hace que se ejecute en el contexto del ámbito global, modificándolo así. Pero en cualquier caso, `eval(..)` puede en tiempo de ejecución modificar un alcance léxico en tiempo de autor.

**Nota:** `eval(..)` cuando se usa en un programa en strict-mode opera en su propio ámbito léxico, lo que significa que las declaraciones realizadas dentro del `eval()` no modifican realmente el ámbito adjunto.

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

Hay otras utilidades en JavaScript que equivalen a un efecto muy similar a `eval(..)`. `setTimeout(..)` y `setInterval(..)` *pueden* tomar una cadena para su respectivo primer argumento, cuyo contenido se evalúa como el código de una función generada dinámicamente. Este es un comportamiento antiguo, heredado y desde hace mucho tiempo en desuso. ¡No lo haga!

El constructor de la función `new Function(..)` toma de manera similar una cadena de código en su **último** argumento para convertirse en una función generada dinámicamente (los primeros argumentos, si los hay, son los parámetros nombrados para el nueva función). Esta sintaxis de función-constructor es ligeramente más segura que `eval(..)`, pero aún debe evitarse en su código.

Los casos de uso para generar código dinámicamente dentro de su programa son increíblemente raros, ya que las degradaciones de rendimiento casi nunca valen la pena.

### `with`

La otra característica mal vista (y ahora obsoleta -- deprecated) en JavaScript que hace trampa al ámbito léxico es la palabra clave `con`. Existen múltiples formas válidas de explicar `with`, pero para explicarlo aquí elegiré la perspectiva de cómo interactúa y afecta el alcance léxico.

`with` se explica normalmente como un atajo para hacer múltiples referencias de propiedades de un objeto *sin* repetir la referencia del objeto en sí misma cada vez.

Por ejemplo:

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

Sin embargo, hay mucho más en juego aquí que solo un atajo acceder a las propiedades de los objetos. Considerando:

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```

En este ejemplo de código, se crean dos objetos `o1` y` o2`. Uno tiene una propiedad `a`, y el otro no. La función `foo(..)` toma como argumento una referencia de objeto `obj`, y llama a `with (obj) {..}` en la referencia. Dentro del bloque `with`, hacemos lo que parece ser una referencia léxica normal a una variable `a`, una referencia LHS de hecho (ver capítulo 1), para asignarle el valor de `2`.

Cuando pasamos `o1`, la asignación `a = 2` encuentra la propiedad `o1.a` y le asigna el valor `2`, como se refleja en la siguiente instrucción `console.log(o1.a)`. Sin embargo, cuando pasamos `o2`, ya que no tiene una propiedad `a`, no se crea tal propiedad, y `o2.a` permanece` undefined`.

Pero luego notamos un efecto secundario peculiar, el hecho de que una variable global `a` fue creada por la asignación `a = 2`. ¿Cómo puede ser esto?

La declaración `with` toma un objeto, uno que tiene cero o más propiedades, y **trata ese objeto como si fuera un scope léxico totalmente separado**, y por lo tanto las propiedades del objeto se tratan como identificadores definidos léxicamente en ese "scope".

**Nota:** Aunque un bloque `with` trata a un objeto como un scope léxico, una declaración `var` normal dentro de ese bloque `with`, no se aplicará a ese bloque `with`, sino al scope de la función que lo contiene.

Mientras que la función `eval(..)` puede modificar el alcance léxico existente si toma una cadena de código con una o más declaraciones, la declaración `with` crea realmente un **alcance léxico completamente nuevo** de la nada, desde el objeto que le pases.

Entendido de esta manera, el "scope" declarado por la declaración `with` cuando le pasamos `o1` era `o1`, y ese "scope" tenía un "identificador" que se corresponde a la propiedad `o1.a`. Pero cuando usamos `o2` el "scope" no tenía un "identificador" `a` en él, y procederá con las reglas normales de búsqueda de identificador LHS (ver capítulo 1).

Ni el "scope" de `o2`, ni el scope de `foo(..)`, ni el alcance global, tienen un identificador `a`, por lo que cuando se ejecuta `a = 2`, se produce la creación automática-global (ya que estamos en modo no estricto).

Es una fumada ver que `with` convierte, en tiempo de ejecución, un objeto y sus propiedades en un "scope" *con* "identificadores". Pero esa es la explicación más clara que puedo dar para los resultados que vemos.

**Nota:** Además de ser una mala idea de usar, tanto `eval(..)` como `with` están afectados (restringidos) por el Strict Mode. `with` está totalmente deshabilitado, mientras que varias formas de `eval(..)` indirectas o inseguras están deshabilitadas si bien conservan la funcionalidad principal.

### Rendimiento / Performance

Tanto `eval(..)` como `with` engañan el ámbito léxico definido de otro modo por el autor, modificando o creando un nuevo ámbito léxico en tiempo de ejecución.

Entonces, ¿cuál es el problema, te preguntarás? Si ofrecen una funcionalidad más sofisticada y flexibilidad de codificación, ¿no son *buenas* estas características? **No.**

El *Engine* de JavaScript tiene una serie de optimizaciones de rendimiento que realiza durante la fase de compilación. Algunos de estos se reducen a ser capaces de analizar estáticamente el código a medida que se lexicaliza, y predeterminar dónde están todas las declaraciones de variables y funciones, por lo que se requiere menos esfuerzo para resolver los identificadores durante la ejecución.

Pero si el *Engine* encuentra un `eval(..)` o `with` en el código, esencialmente tiene que *asumir* que todo su conocimiento de la ubicación del identificador puede no ser válido, porque en tiempo de lexing no puede saber exactamente qué código se va a pasar a `eval(..)` para modificar el alcance léxico, o el contenido del objeto que puede pasar a `with` para crear un nuevo alcance léxico al que consultar.

En otras palabras, en sentido pesimista, la mayoría de esas optimizaciones que *haría* no tienen sentido si `eval(..)` o `con` están presentes, por lo que simplemente no realiza las optimizaciones *en absoluto*.

Es muy probable que su código se ejecute más lento simplemente por el hecho de que incluya un `eval(..)` o `with` en cualquier lugar del código. No importa cuán inteligente sea el *Engine* para tratar de limitar los efectos secundarios de estas suposiciones pesimistas, **no hay forma de evitar el hecho de que sin las optimizaciones, el código se ejecuta más lentamente**.

## Repaso (TL;DR)

El alcance léxico significa que el scope se define mediante decisiones de tiempo de autor de dónde se declaran las funciones. La fase lexing de la compilación es esencialmente capaz de saber dónde y cómo se declaran todos los identificadores, y así predecir cómo se buscarán durante la ejecución.

Dos mecanismos en JavaScript pueden "engañar" al ámbito léxico: `eval(..)` y `with`. El primero puede modificar el alcance léxico existente (en tiempo de ejecución) mediante la evaluación de una cadena de "código" que contiene una o más declaraciones. Este último esencialmente crea un ámbito léxico completamente nuevo (de nuevo, en tiempo de ejecución) al tratar una referencia de objeto *como* un "scope" y las propiedades de ese objeto como identificadores de su ámbito.

La desventaja de estos mecanismos es que anula la capacidad del *Engine* para realizar optimizaciones en tiempo de compilación con respecto a la búsqueda de alcance, porque el *Engine* debe asumir de forma pesimista que tales optimizaciones no serán válidas. El código se ejecutará más lentamente como resultado del uso de cualquiera de las funciones. **No las use**.
