# You Don't Know JS: Up & Going
# Chapter 3: Hacia YDKJS

¿De qué trata esta serie? En pocas palabras, se trata de tomar en serio la tarea de aprender *todas las partes de JavaScript*, no solo un subconjunto del lenguaje que alguien llamó "las partes buenas" ("the good parts") y no solo la cantidad mínima que necesita para hacer su trabajo en el trabajo.

Los desarrolladores serios en otros lenguajes esperan esforzarse por aprender la mayoría o todos los lenguajes en los que escriben principalmente, pero los desarrolladores de JS parecen sobresalir entre la multitud en el sentido de que normalmente no aprenden mucho del lenguaje. Esto no es algo bueno, y no es algo que debamos seguir permitiendo que sea la norma.

La serie *You Don't Know JS* (* YDKJS *) está en marcado contraste con los enfoques típicos para el aprendizaje de JS, y es diferente a casi cualquier otro libro de JS que lea. Te desafía a ir más allá de tu zona de confort y a hacer preguntas más profundas sobre "por qué" para cada comportamiento que encuentres. ¿Estás preparado para ese desafío?

Voy a utilizar este capítulo final para resumir brevemente qué esperar del resto de los libros de la serie, y cómo ir más efectivamente para construir una base de aprendizaje de JS sobre *YDKJS*.

## Scope & Closures

Tal vez una de las cosas más fundamentales con las que deberá familiarizarse rápidamente es cómo funciona realmente el scope de las variables en JavaScript. No es suficiente tener creencias borrosas *anecdóticas* sobre el scope.

El título *Scope & Closures* comienza por desacreditar la idea errónea de que JS es un "lenguaje interpretado" y, por lo tanto, no está compilado. No.

El motor JS compila su código justo antes de la ejecución (y, a veces, durante). Así que utilizamos un entendimiento más profundo del enfoque del compilador de nuestro código para entender cómo encuentra y trata las declaraciones de variables y funciones. A lo largo del camino, vemos la metáfora típica de la gestión de alcance de variables de JS, "Elevación" o "Hoisting."

Esta comprensión crítica del "scope léxico" es en lo que luego basamos nuestra exploración del "closure" para el último capítulo del libro. El closure es quizás el concepto más importante de todo JS, pero si no ha entendido bien cómo funciona el scope, es probable que el closure quede fuera de su alcance.

Una aplicación importante del closure es el patrón del módulo, como lo presentamos brevemente en este libro en el capítulo 2. El patrón del módulo es quizás el patrón de organización de código más frecuente en todo JavaScript; su profunda comprensión debería ser una de tus más altas prioridades.

## this & Object Prototypes

Quizás uno de los errores más extendidos y persistentes sobre JavaScript es que la palabra clave `this` se refiere a la función en la que aparece. Terriblemente equivocada.

La palabra clave `this` está vinculada dinámicamente en función de cómo se ejecuta la función en cuestión, y resulta que hay cuatro reglas simples para comprender y determinar completamente la vinculación de` this`.

Muy relacionado con la palabra clave `this` está el mecanismo de prototipo de objeto, que es una cadena de búsqueda de propiedades, similar a cómo se encuentran las variables de scope léxico. Pero envuelto en los prototipos está el otro gran error de JS: la idea de emular clases (falsas) y la herencia (llamada "prototípica").

Desafortunadamente, el deseo de llevar el pensamiento de patrones de diseño de clase y herencia a JavaScript es casi lo peor que podrías intentar hacer, porque aunque la sintaxis puede engañarte para que creas que hay algo como clases presentes, de hecho, el mecanismo prototipo es fundamentalmente opuesto en su comportamiento.

Lo que está en discusión es si es mejor ignorar el desajuste y pretender que lo que estás implementando es "herencia", o si es más apropiado aprender y aceptar cómo funciona realmente el sistema de prototipos de objetos. Este último se denomina más apropiadamente "delegación de comportamiento".

Esto es más que una preferencia sintáctica. La delegación es un patrón de diseño completamente diferente y más poderoso, que reemplaza la necesidad de diseñar con clases y herencia. Sin embargo, estas afirmaciones van a ser absolutamente contrarias a casi todas las demás publicaciones de blogs, libros y conferencias sobre el tema durante toda la vida de JavaScript.

Las afirmaciones que hago con respecto a la delegación frente a la herencia no provienen de una aversión al lenguaje y su sintaxis, sino del deseo de ver la verdadera capacidad del lenguaje adecuadamente aprovechado y acabar de una vez por toda con la confusión y frustración generada.

Pero el caso que hago con respecto a los prototipos y la delegación es mucho más complicado de lo que voy a realizar aquí. Si está listo para reconsiderar todo lo que cree que sabe sobre "clases" y "herencia" de JavaScript, le ofrezco la oportunidad de "tomar la píldora roja" (*Matrix* 1999) y consultar los capítulos 4-6 del titulo *this & Object Prototypes* de esta serie.

## Tipos y gramática / Types & Grammar

El tercer título de esta serie se centra principalmente en abordar otro tema altamente controvertido: la coercion de tipo. Quizás ningún tema cause más frustración con los desarrolladores de JS que cuando se habla de las confusiones que rodean la coerción implícita.

Con mucho, la opinión convencional es que la coerción implícita es una "parte mala" del lenguaje y debe evitarse a toda costa. De hecho, algunos han llegado al extremo de llamarlo un "defecto" en el diseño del lenguaje. De hecho, hay herramientas cuyo trabajo completo es no hacer nada más que escanear su código y quejarse si está haciendo algo remotamente parecido a la coercion.

Pero, ¿es la coercion realmente tan confusa, tan mala, tan traicionera, que su código está condenado desde el principio si la usa?

Yo digo que no. Después de haber comprendido cómo funcionan realmente los tipos y valores en los capítulos 1-3, el capítulo 4 aborda este debate y explica completamente cómo funciona la coerción, en todos sus rincones y grietas. Vemos qué partes de la coerción son realmente sorprendentes y qué partes realmente tienen sentido completo si se les da tiempo para aprenderse.

Pero no estoy simplemente sugiriendo que la coerción es sensata y fácil de aprender, estoy afirmando que la coerción es una herramienta increíblemente útil y totalmente subestimada que *debería estar usando en su código*. Estoy diciendo que la coerción, cuando se usa correctamente, no solo funciona, sino que mejora su código. Todos los detractores y los que dudan seguramente se burlarán de esa posición, pero creo que es una de las principales claves para mejorar tu juego JS.

¿Desea simplemente seguir lo que dice la multitud o está dispuesto a dejar de lado todas las suposiciones y mirar la coerción con una perspectiva nueva? El título *Types & Grammar* de esta serie forzará su pensamiento.

## Rendimiento y asincronía / Async & Performance

Los tres primeros títulos de esta serie se centran en la mecánica básica del lenguaje, pero el cuarto título se bifurca ligeramente para cubrir patrones en la parte superior de la mecánica del lenguaje para administrar la programación asíncrona. La asincronía no solo es crítica para el rendimiento de nuestras aplicaciones, sino que se está convirtiendo cada vez más en *el* factor crítico en la capacidad de escritura y el mantenimiento.

El libro comienza primero al aclarar una gran cantidad de terminología y confusión de conceptos en torno a cosas como "asíncrono", "paralelo" y "concurrente", y explica en profundidad cómo se aplican y no se aplican a JS.

Luego pasamos a examinar los *callbacks* (devoluciones de llamada) como el método principal para habilitar la asincronía. Pero es aquí donde vemos rápidamente que la devolución de llamada por sí sola es irremediablemente insuficiente para las demandas modernas de la programación asíncrona. Identificamos dos deficiencias importantes de la codificación de solo devoluciones de llamada (callbacks-only coding): *Inversión de control* (IoC), confianza en la pérdida y falta de capacidad de razón lineal.

Para abordar estas dos deficiencias principales, ES6 introduce dos nuevos mecanismos (y, de hecho, patrones): promises y generators (promesas y generadores).

Las promesas son un wrapper (envoltorio) independiente del tiempo en torno a un "valor futuro", que le permite razonar y redactarlas independientemente de si el valor está listo o no. Además, resuelven de manera efectiva los problemas de confianza de IoC enrutando las devoluciones de llamadas a través de un mecanismo de promesa confiable y componible.

Los generadores introducen un nuevo modo de ejecución para las funciones JS, mediante el cual el generador puede pausarse en los puntos `yield` "de rendimiento" y reanudarse de forma asíncrona más tarde. La capacidad de pausa y reanudación permite que el código de búsqueda secuencial y síncrona en el generador se procese de forma asíncrona entre bastidores. Al hacerlo, abordamos las confusiones no lineales, de salto no local de devoluciones de llamada y, por lo tanto, hacemos que nuestro código asíncrono parezca sincronizado para que sea más razonable.

Pero es la combinación de promesas y generadores que "produce" ("yields") nuestro patrón de codificación asíncrono más efectivo hasta la fecha en JavaScript. De hecho, gran parte de la sofisticación futura de la asincronía que viene en ES7 y más adelante seguramente se construirá sobre esta base. Para tomar en serio la programación efectiva en un mundo asíncrono, necesitarás sentirte realmente cómodo combinando promesas y generadores.

Si las promesas y los generadores tratan de expresar patrones que permiten que nuestros programas se ejecuten de manera más concurrente y, por lo tanto, logren un mayor procesamiento en un período más corto, JS tiene muchas otras facetas de optimización de rendimiento que vale la pena explorar.

El capítulo 5 profundiza en temas como el paralelismo de programas con Web Workers y el paralelismo de datos con SIMD, así como técnicas de optimización de bajo nivel como ASM.js. El capítulo 6 analiza la optimización del rendimiento desde la perspectiva de las técnicas benchmarking (de evaluación comparativa) adecuadas, incluidos los tipos de rendimiento de los que preocuparse y qué ignorar.

Escribir JavaScript de manera efectiva significa escribir código que puede romper las barreras de restricción de ser ejecutado dinámicamente en una amplia gama de navegadores y otros entornos. Requiere una gran cantidad de planificación detallada y un esfuerzo por nuestras partes para llevar un programa de "funciona" a "funciona bien".

El título *Async & Performance* está diseñado para brindarle todas las herramientas y habilidades que necesita para escribir un código de JavaScript razonable y eficaz.

## ES6 & Más Allá

No importa cuánto sientas que has dominado JavaScript hasta este momento, la verdad es que JavaScript nunca dejará de evolucionar y, además, la tasa de evolución está aumentando rápidamente. Este hecho es casi una metáfora del espíritu de esta serie, al aceptar que nunca *conoceremos* completamente todas las partes de JS, porque tan pronto como lo domine todo, habrá nuevas cosas en el futuro que necesitará aprender

Este título está dedicado tanto a las visiones a corto como a mediano plazo de hacia dónde se dirige el lenguaje, no solo a las cosas *conocidas* como ES6 sino a las cosas *probables* más allá.

Si bien todos los títulos de esta serie abarcan el estado de JavaScript en el momento de escribir este artículo, que se encuentra a medio camino de la adopción de ES6, el enfoque principal de la serie ha sido más sobre ES5. Ahora, queremos dirigir nuestra atención a ES6, ES7 y ...

Dado que ES6 está casi completo en el momento de escribir este artículo, *ES6 & Beyond* comienza dividiendo las cosas concretas del entorno ES6 en varias categorías clave, que incluyen nueva sintaxis, nuevas estructuras de datos (colecciones) y nuevas capacidades de procesamiento y API. . Cubrimos cada una de estas nuevas características de ES6, en diferentes niveles de detalle, incluida la revisión de detalles que se mencionan en otros libros de esta serie.

Algunas cosas interesantes de ES6 sobre las cuales podemos leer: la desestructuración (destructuring), los valores de parámetros por defecto, los símbolos, los métodos concisos, las propiedades computadas, las arrow functions (funciones de flecha), el scope de bloque, las promesas, los generadores, los iteradores, los módulos, los proxies, los mapas débiles (weakmaps) y mucho, mucho más. ¡Uf, el ES6 tiene un buen golpe!

La primera parte del libro es una hoja de ruta para todas las cosas que necesita aprender para prepararse para el nuevo y mejorado JavaScript que estará escribiendo y explorando durante los próximos dos años.

La última parte del libro llama la atención para echar un vistazo breve a las cosas que probablemente podamos esperar ver en el futuro cercano de JavaScript. La realización más importante aquí es que después de ES6, es probable que JS evolucione función por función en lugar de versión por versión, lo que significa que podemos esperar que estas cosas del futuro próximo se produzcan mucho antes de lo que pueda imaginar.

El futuro para JavaScript es brillante. ¿No es hora de que empecemos a aprenderlo?

## Repaso

La serie *YDKJS* está dedicada a la propuesta de que todos los desarrolladores de JS pueden y deben aprender todas las partes de este gran lenguaje. La opinión de ninguna persona, las suposiciones de ningún marco y la fecha límite de ningún proyecto deben ser la excusa de por qué nunca se aprende y se entiende a fondo JavaScript.

Tomamos cada área importante de enfoque en el lenguaje y dedicamos un libro corto pero muy denso para explorar por completo todas las partes del mismo que quizás pensó que sabía aunque que probablemente no lo hiciera del todo.

"You Don't Know JS", No sabes JS" no es una crítica o un insulto. Es un descubrimiento que todos nosotros, incluido yo mismo, debemos aceptar. Aprender JavaScript no es un objetivo final sino un proceso. Todavía no sabemos JavaScript. ¡Pero lo haremos!
