# You Don't Know JS: Scope & Closures
# Chapter 1: What is Scope?

Uno de los paradigmas más fundamentales de casi todos los lenguajes de programación es la capacidad de almacenar valores en variables, y luego recuperar o modificar esos valores. De hecho, la capacidad de almacenar valores y extraer valores de las variables es lo que le da a un programa el *estado*.

Sin tal concepto, un programa podría realizar algunas tareas, pero serían extremadamente limitados y no terriblemente interesantes.

Pero la inclusión de variables en nuestro programa genera las preguntas más interesantes que abordaremos ahora: ¿dónde viven esas variables *en vivo*? En otras palabras, ¿dónde se almacenan? Y, lo más importante, ¿cómo los encuentra nuestro programa cuando los necesita?

Estas preguntas hablan de la necesidad de un conjunto bien definido de reglas para almacenar variables en algún lugar y para encontrarlas más adelante. Llamaremos a ese conjunto de reglas: *Scope*.

Pero, ¿dónde y cómo se establecen estas reglas del *scope*?

## Teoría delCompilador

Puede ser evidente, o puede ser sorprendente, dependiendo de su nivel de interacción con varios lenguajes, pero a pesar del hecho de que JavaScript se encuentra dentro de la categoría general de idiomas "dinámicos" o "interpretados", de hecho es una lenguaje *compilado*. *No* se compila con mucha antelación, como sí harían la mayoría de lenguajes compilados tradicionalmente, ni los resultados de la compilación son portables entre varios sistemas distribuidos.

Pero, sin embargo, el motor de JavaScript realiza muchos de los mismos pasos, aunque de formas más sofisticadas de lo que comúnmente podemos saber de cualquier compilador de lenguaje tradicional.

En un proceso tradicional de lenguaje compilado, una parte del código fuente, su programa, se someterá generalmente a tres pasos *antes* de que se ejecute, a lo que Llamaremos toscamente "compilación":

1. **Tokenizing/Lexing:** dividir una cadena de caracteres en trozos significativos (para el lenguaje), llamados tokens. Por ejemplo, considere el programa: `var a = 2;`. Este programa probablemente se dividiría en los siguientes tokens: `var`,` a`, `=`, `2` y `;`. El espacio en blanco puede o no persistir como un token, dependiendo de si es significativo o no.

    **Nota:* La diferencia entre la tokenización y el lexing es sutil y académica, pero se centra en si estos tokens se identifican o no de una manera *sin estado* o *con estado*. En pocas palabras, si el tokenizador invocara reglas de análisis de estado para determinar si `a` debería considerarse un token distinto o solo como parte de otro token, *ese* sería **lexing**.

2. **Parsing:** Análizar gramáticamente tomando una secuencia (matriz) de tokens y convirtiéndola en un árbol de elementos anidados, que en conjunto representan la estructura gramatical del programa. Este árbol se llama "AST"  (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).

    El árbol para `var a = 2;` podría comenzar con un nodo de nivel superior llamado `VariableDeclaration`, con un nodo hijo llamado` Identifier` (cuyo valor es `a`), y otro hijo llamado `AssignmentExpression` que tiene un hijo llamado `NumericLiteral` (cuyo valor es `2`).

3. **Code-Generation:** el proceso de tomar un AST y convertirlo en código ejecutable. Esta parte varía mucho según el lenguaje, la plataforma a la que se dirige, etc.    

    Así que, en lugar de quedarnos atascados en los detalles, simplemente pasaremos la mano y diremos que hay una manera de tomar nuestro AST descrito anteriormente para `var a = 2;` y convertirlo en un conjunto de instrucciones de máquina para realmente *crear* una variable se llama `a` (incluida la reserva de memoria, etc.), y luego almacena un valor en `a`.

    **Nota:** Los detalles de cómo el motor gestiona los recursos del sistema son más profundos de lo que vamos a investigar, por lo que simplemente damos por sentado que el motor puede crear y almacenar variables según sea necesario.

El motor de JavaScript es mucho más complejo que *solo* esos tres pasos, como lo son la mayoría de los compiladores de otros lenguajes. Por ejemplo, en el proceso de análisis y generación de código, ciertamente existen pasos para optimizar el rendimiento de la ejecución, incluido el colapso de elementos redundantes, etc.

Por lo tanto, estoy pintando sólo a grandes rasgos aquí. Pero creo que pronto verán por qué *estos* detalles que *cubrimos*, incluso en un nivel alto, son relevantes.

Por un lado, los motores de JavaScript no pueden darse el lujo (como otros compiladores) de disponer de mucho tiempo para optimizar, porque la compilación de JavaScript no se produce en un paso previo de construcción, como ocurre con otros idiomas.

Para JavaScript, la compilación que ocurre sucede, en muchos casos, en meros microsegundos (¡o menos!) Antes de que se ejecute el código. Para garantizar el rendimiento más rápido, los motores JS utilizan todo tipo de trucos (como los JIT, que compilan de forma perezosa e incluso se vuelven a compilar en caliente, etc.) que están mucho más allá del "alcance" de nuestra discusión aquí.

Digamos, para simplificar, que cualquier fragmento de JavaScript debe compilarse antes (generalmente  ¡*justo* antes!). Se ejecuta. Entonces, el compilador JS tomará el programa `var a = 2;` y lo compilará *primero*, y luego estará listo para ejecutarlo, generalmente de inmediato.

## Entendiendo el Scope

La forma en que abordaremos el aprendizaje sobre el scope es pensar el proceso en términos de una conversación. Pero, ¿*quién* está teniendo la conversación?

### El Reparto / The Cast

Conozcamos al elenco de personajes que interactúan para procesar el programa `var a = 2;`, de forma que entendamos sus conversaciones que escucharemos en breve:

1. *Engine*: responsable de la compilación y ejecución de principio a fin de nuestro programa JavaScript.

2. *Compiler*: uno de los amigos de *Engine*; maneja todo el trabajo sucio de análisis y generación de código (ver la sección anterior).

3. *Scope*: otro amigo de *Engine*; recopila y mantiene una lista de búsqueda de todos los identificadores (variables) declarados, y aplica un conjunto estricto de reglas en cuanto a cómo estos son accesibles al código que se está ejecutando actualmente.

Para que *entiendas completamente* cómo funciona JavaScript, debes comenzar a *pensar* como *Engine* (y amigos), hacer las preguntas que hacen y responder esas preguntas de la misma manera.

### Atrás & Alante / Back & Forth

Cuando vea el programa `var a = 2;`, lo más probable es que piense en eso como una declaración. Pero no es así como lo ve nuestro nuevo amigo *Engine*. De hecho, *Engine* ve dos declaraciones distintas, una que *Compiler* manejará durante la compilación, y otra que *Engine* manejará durante la ejecución.

Entonces, analicemos cómo *Engine* y sus amigos abordarán el programa `var a = 2;`.

Lo primero que *Compiler* hará con este programa es realizar lexing para descomponerlo en tokens, que luego se analizará en un árbol. Pero cuando *Compiler* llega a la generación de código, tratará este programa de una manera algo diferente a la que se supone.

Una suposición razonable sería que *Compiler* producirá un código que podría resumirse con este pseudo-código: "Asignar memoria a una variable, etiquetarla como `a`, luego pegar el valor `2` en esa variable". Desafortunadamente, eso no es del todo exacto.

*Compiler* procederá en su lugar así:

1. Al encontrar `var a`, *Compiler* pregunta a *Scope* para ver si ya existe una variable `a` para esa colección de alcance en particular. Si es así, *Compilador* ignora esta declaración y continúa. De lo contrario, *Compiler* le pide a *Scope* que declare una nueva variable llamada `a` para esa colección de alcance.

2. Luego *Compiler* produce el código para manejar la asignación `a = 2` para que lo ejecute más tarde *Engine*. Para ejecutar ese código, *Engine* lo primero qué hará es preguntar a *Scope* si hay una variable llamada `a` accesible en la colección de scope actual. Si es así, *Engine* usa esa variable. Si no es así, *Engine* se ve *en otro lugar*  (vea la sección *scope anidado* o *nested scope*).

Si *Engine* finalmente encuentra una variable, le asigna el valor `2`. Si no, ¡*Engine* levantará su mano y gritará un error!

Para resumir: se realizan dos acciones distintas para una asignación de variable: Primero, *Compiler* declara una variable (si no se declaró anteriormente en el scope actual), y segundo, cuando se ejecuta, *Engine* busca la variable en *Scope* y se la asigna, si la se encuentra.

### El Compilador Habla

Necesitamos un poco más de terminología del compilador para continuar con la comprensión.

Cuando *Engine* ejecuta el código que *Compiler* produjo para el paso (2), tiene que buscar la variable `a` para ver si se ha declarado, y esta búsqueda está consultando a *Scope*. Pero el tipo de búsqueda que realiza *Engine* afecta el resultado de la búsqueda.

En nuestro caso, se dice que *Engine* estaría realizando una búsqueda "LHS" para la variable `a`. El otro tipo de búsqueda se llama "RHS".

Apuesto a que puedes adivinar qué significan las letras "L" y "R". Estos términos representan "Lado izquierdo" y "Lado derecho" (Left y Right).

Lado ... de qué? **De una operación de asignación**.

En otras palabras, se realiza una búsqueda de LHS cuando aparece una variable en el lado izquierdo de una operación de asignación, y se realiza una búsqueda de RHS cuando aparece una variable en el lado derecho de una operación de asignación.

En realidad, seamos un poco más precisos. Una búsqueda de RHS es indistinguible, para nuestros propósitos, de una simple búsqueda del *valor de alguna variable*, mientras que la búsqueda de LHS está tratando de encontrar el *contenedor de la variable* en sí, para que pueda asignar. De esta manera, RHS no significa *realmente* "el lado derecho de una tarea" per se, sino que, más precisamente, significa "no el lado izquierdo".

Al ser ligeramente simplista por un momento, también podría pensar que "RHS" significa "recuperar su fuente (valor)", lo que implica que RHS significa "obtener el valor de ...".

Vamos a profundizar en eso más profundo.

Cuando yo digo:

```js
console.log( a );
```

La referencia a `a` es una referencia de RHS, porque aquí no se asigna nada a `a`. En su lugar, estamos buscando para recuperar el valor de `a`, de modo que el valor se pueda pasar a `console.log(..)`.

Por el contrario:

```js
a = 2;
```

La referencia a `a` aquí es una referencia de LHS, porque en realidad no nos importa cuál es el valor actual, simplemente queremos encontrar la variable como objetivo para la operación de asignación `=2`.

**Nota:** LHS y RHS que significan "lado izquierdo/derecho de una asignación" no necesariamente significan literalmente "lado izquierdo/derecho del operador de asignación `=`". Hay varias otras formas en que se realizan las asignaciones, por lo que es mejor pensar conceptualmente como "quién es el objetivo de la asignación (LHS)" y "quién es la fuente de la asignación (RHS)".

Considere este programa, que tiene referencias tanto de LHS como de RHS:

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

La última línea que invoca `foo(..)`, como una llamada a una función, requiere una referencia RHS a `foo`, que significa “ve a buscar el valor de `foo`, y entrégamelo”. Además, `(..)` significa que el valor de `foo` debe ejecutarse, por lo que es mejor que sea una función.

Hay una tarea sutil pero importante aquí. **¿Lo viste?**

Es posible que haya omitido el `a = 2` implícito en este fragmento de código. Ocurre cuando el valor `2` se pasa como un argumento a la función `foo(..)`, en cuyo caso el valor `2` se **asigna** al parámetro `a`. Para (implícitamente) asignar al parámetro `a`, se realiza una búsqueda de LHS.

También hay una referencia RHS para el valor de `a`, y el valor resultante se pasa a `console.log (..)`. `console.log(..)` necesita una referencia para ejecutarse. Es una búsqueda de RHS para el objeto `console`, luego se produce una resolución de propiedad para ver si tiene un método llamado` log`.

Finalmente, podemos conceptualizar que hay un intercambio de LHS/RHS de pasar el valor `2` (mediante la búsqueda de RHS de la variable `a`) a `log(..)`. Dentro de la implementación nativa de `log (..)`, podemos asumir que tiene parámetros, el primero de los cuales (quizás llamado `arg1`) tiene una búsqueda de referencia de LHS, antes de asignarle `2`.

**Nota:** Podría estar tentado a conceptualizar la declaración de función `function foo(a) {...` como una declaración y asignación de variable normal, como `var foo` y `foo = function(a) {. ..`. Al hacerlo, sería tentador pensar que esta declaración de función implica una búsqueda de LHS.

Sin embargo, la diferencia sutil pero importante es que *Compiler* maneja tanto la declaración como la definición de valor durante la *generación de código*, de manera que cuando *Engine* lo ejecuta, no es necesario procesar "asignar" un valor de función a `foo`. Por lo tanto, no es realmente apropiado pensar en una declaración de función como una tarea de búsqueda de LHS en la forma en que las discutimos aquí.

### Conversación Engine/Scope

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Imaginemos el intercambio anterior (que procesa este fragmento de código) como una conversación. La conversación iría algo así:

> ***Engine***: Ey! *Scope*, tengo una referencia de RHS para `foo`. ¿Has oído hablar de él?

> ***Scope***: Sí, lo tengo. *Compiler* lo declaró hace apenas un segundo. Es una función. Aquí tienes.

> ***Engine***: Genial, gracias! OK, estoy ejecutando `foo`.

> ***Engine***: Ey!, *Scope*, Tengo una referencia de LHS para `a`, ¿Has oído hablar de él?

> ***Scope***: Sí, lo tengo. *Compiler* lo declaró como un parámetro formal de `foo` recientemente. Aquí tienes.

> ***Engine***: Útil como siempre, *Scope*. Gracis de nuevo. Ahora, hora de asignar `2` a `a`.

> ***Engine***: Ey!, *Scope*, iento molestarte de nuevo. Necesito una búsqueda RHS para `console`. ¿Has oído hablar de él?

> ***Scope***: Sin problemas, *Engine*, esto es lo que hago todo el día. Sí, tengo `console`. Está integrado. Aquí tienes.

> ***Engine***: Perfecto. Buscando `log(..)`. OK, genial, es una función.

> ***Engine***: Ey!, *Scope*. Puedes ayudarme con una referencia RHS a `a`. Creo que lo recuerdo, pero quiero volver a comprobar.

> ***Scope***: Tienes razón, *Engine*. El mismo tipo, no ha cambiado. Aquí tienes.

> ***Engine***: Guay. Pasando el valor de `a`, que es`2`, a `log(..)`.

> ...

### Cuestionario

Compruebe su comprensión hasta ahora. Asegúrate de jugar la parte de *Engine* y tener una "conversación" con *Scope*:

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identifique todas las búsquedas de LHS (hay 3!).

2. Identifique todas las búsquedas de RHS (hay 4!).

**Nota:** ¡Vea el resumen del capítulo para las respuestas al cuestionario!

## Alcance Anidado / Nested Scope

Dijimos que *Scope* es un conjunto de reglas para buscar variables por su nombre de identificador. Sin embargo, generalmente hay más de un *Scope* a considerar.

Al igual que un bloque o función se anida dentro de otro bloque o función, los ámbitos se anidan dentro de otros ámbitos. Por lo tanto, si no se puede encontrar una variable en el ámbito inmediato, *Engine* consulta el siguiente ámbito de contenido externo, continuando hasta que se encuentre o hasta que se alcance el alcance más externo (también conocido como global).

Considerando:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

La referencia RHS para `b` no se puede resolver dentro de la función` foo`, pero se puede resolver en el *Scope* que la rodea (en este caso, el global).

Entonces, revisando las conversaciones entre *Engine* y *Scope*, escucharíamos:

> ***Engine***: "Ey!, *Scope* de `foo`, ¿Has oído hablar de `b`? Tengo una referencia de RHS para él."

> ***Scope***: "No, nunca he oído hablar de él. Ve a buscar."

> ***Engine***: "Hey, *Scope* fuera de `foo`, ¡oh! eres el *Scope* global, que guay. ¿Has oído hablar de `b`?  Tengo una referencia de RHS para él."

> ***Scope***: "Sí, claro. Aquí tienes.""

Las reglas simples para atravesar el *Scope anidado*: *Engine* comienza en el *Scope* actualmente en ejecución, busca la variable allí, luego, si no se encuentra, sigue subiendo un nivel, y así sucesivamente. Si se alcanza el alcance global más externo, la búsqueda se detiene, ya sea que encuentre la variable o no.

### Construir sobre metáforas

Para visualizar el proceso de resolución del *Scope* anidado, quiero que piensen en este edificio alto.

<img src="fig1.png" width="250">

El edificio representa el conjunto de reglas *Scope* anidado de nuestro programa. El primer piso del edificio representa tu *Scope* actualmente en ejecución, dondequiera que estés. El nivel superior del edificio es el *Scope* global.

Resuelva las referencias de LHS y RHS al buscar en su piso actual, y si no lo encuentra, tome el ascensor hasta el siguiente piso, mirando hacia allí, luego al siguiente, y así sucesivamente. Una vez que llega al último piso (el *Scope* global), encontrará lo que está buscando o no. Pero en cualquier caso tiene que parar.

## Errores

¿Por qué importa si lo llamamos LHS o RHS?

Porque estos dos tipos de búsquedas se comportan de manera diferente cuando la variable aún no se ha declarado (no se encuentra en ningún *Scope* consultado).

Considerando:

```js
function foo(a) {
	console.log(a + b);
	b = a;
}

foo(2);
```

Cuando la búsqueda de RHS se produce para `b` la primera vez, no se encontrará. Se dice que esto es una variable "no declarada", porque no se encuentra en el scope.

Si una búsqueda de RHS no puede encontrar una variable, en cualquier parte del *Scope* anidado, esto da como resultado que el *Engine* arroje un `ReferenceError`. Es importante tener en cuenta que el error es del tipo `ReferenceError`.

Por el contrario, si *Engine* está realizando una búsqueda LHS y llega al piso superior (*Scope* global) sin encontrarlo, y si el programa no se está ejecutando en "Modo estricto" [^ note-strictmode], entonces el *Scope* global creará una nueva variable de ese nombre **en el scope global**, y se la devolverá a *Engine*.

*"No, no había ninguno antes, pero te ayudé y creé uno para ti".*

El "Strict Mode" [^ note-strictmode], que se agregó en ES5, tiene varios comportamientos diferentes del modo normal/relajado/perezoso. Uno de estos comportamientos es que no permite la creación de variables globales automática/implícita. En ese caso, no habría una variable en el *Scope* global para devolver desde una búsqueda de LHS, y *Engine* lanzaría un `ReferenceError` similar al caso de RHS.

Ahora, si se encuentra una variable para una búsqueda de RHS, pero intenta hacer algo que es imposible con su valor, como intentar ejecutar-como-una-función un valor que no es una funcion, o hacer referencia a una propiedad en un valor `null `o` undefined`, entonces *Engine* lanza un tipo diferente de error, llamado `TypeError`.

`ReferenceError` está relacionado con un error en la resolución del *Scope*, mientras que` TypeError` implica que la resolución *Scope* fue exitosa, pero que se intentó una acción ilegal/imposible con ese resultado.

## Repaso (TL;DR)

Scope es el conjunto de reglas que determina dónde y cómo se puede buscar una variable (identificador). Esta búsqueda puede ser con el propósito de asignar a la variable, que es una referencia de LHS (lado izquierdo), o puede ser con el propósito de recuperar su valor, que es un RHS (lado derecho) ) referencia.

Las referencias de LHS son el resultado de las operaciones de asignación. Las asignaciones relacionadas con el *Scope* pueden ocurrir con el operador `=` o al pasar argumentos para (asignar a) parámetros de una función.

El *Engine* de JavaScript primero compila el código antes de que se ejecute, y al hacerlo, divide las declaraciones como `var a = 2;` en dos pasos separados:

1. Primero, `var a` para declararlo en ese *Scope*. Esto se realiza al principio, antes de la ejecución del código.

2. Más tarde, `a = 2` para buscar la variable (referencia LHS) y asignarla si la encuentra.

Las búsquedas de referencia de LHS y RHS comienzan en el *Scope* que se está ejecutando actualmente, y si es necesario (es decir, no encuentran lo que están buscando allí), se abren camino hasta el *Scope* anidado, un scope (piso) por vez, buscando el identificador, hasta que lleguen al global (piso superior) y se detengan, habiéndolo encontrándo o no.

Las referencias de RHS no cumplidas dan como resultado el lanzamiento de `ReferenceError`s. Las referencias de LHS no cumplidas dan como resultado la creación implícita automática de ese nombre en el scope global (si no está en "Modo estricto" [^note-strictmode]), o un `ReferenceError` (si está en "Strict Mode" [^note-strictmode]).

### Respuestas al cuestionario

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identifique todas las búsquedas de LHS (hay 3!).

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identifique todas las búsquedas de RHS (hay 4!).

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)
