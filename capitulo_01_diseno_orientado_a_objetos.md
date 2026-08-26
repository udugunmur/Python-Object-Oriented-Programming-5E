# Parte 1: Fundamentos de la Programación Orientada a Objetos en Python
## Capítulo 1: Diseño Orientado a Objetos

En el desarrollo de software, el diseño a menudo se considera el paso que precede a la programación. Esto no es cierto; en realidad, el análisis, la programación y el diseño tienden a solaparse, combinarse y entrelazarse. A lo largo de este libro, cubriremos una mezcla de cuestiones de diseño y programación sin intentar dividirlas en compartimentos separados. Una de las ventajas de un lenguaje como Python es su capacidad para expresar el diseño con claridad.

En este capítulo, hablaremos un poco sobre cómo podemos pasar de una buena idea a escribir software. Crearemos algunos artefactos de diseño —como diagramas— que pueden ayudar a clarificar nuestro pensamiento antes de empezar a escribir código. Cubriremos los siguientes temas:

- Qué significa **orientado a objetos**.
- La diferencia entre el **diseño orientado a objetos** y la **programación orientada a objetos**.
- Algunos principios básicos del **diseño orientado a objetos**.
- **Lenguaje Unificado de Modelado (UML)** básico y cuándo no resulta perjudicial.

---

### Sección 1.1: Requisitos técnicos

El código de este capítulo se encuentra en el repositorio de PacktPublishing: [https://github.com/PacktPublishing/Python-Object-Oriented-Programming-5E](https://github.com/PacktPublishing/Python-Object-Oriented-Programming-5E). Dentro de los archivos de ese repositorio, nos centraremos en el directorio `ch_01`.

Todos los ejemplos fueron probados con Python 3.12 y 3.13. Se puede utilizar la herramienta `uv` para probar el código:

```bash
uvx tox
```

---

### Sección 1.2: Qué significa orientado a objetos

Fuera del mundo del software, un objeto es una cosa tangible que podemos percibir, sentir y manipular. Los primeros objetos con los que interactuamos suelen ser juguetes de bebé. Los bloques de madera, las formas de plástico y las piezas de rompecabezas de gran tamaño son objetos iniciales habituales. Los bebés aprenden rápidamente que ciertos objetos hacen ciertas cosas: las campanas suenan, los botones se presionan y las palancas se accionan.

La definición de un objeto en el desarrollo de software no es muy diferente. Los objetos de software pueden no ser cosas tangibles que puedas recoger, percibir o sentir, pero son modelos de algo que tiene un estado interno, puede hacer ciertas cosas y responde cuando se realizan acciones sobre él. Formalmente, un **objeto** es una colección de datos y comportamientos asociados.

La **programación orientada a objetos** significa escribir código dirigido a modelar objetos. Esta es una de las muchas técnicas utilizadas para describir las acciones de sistemas complejos. El comportamiento general surge de la colaboración entre objetos. Un estado interno complicado se descompone en los estados de objetos individuales separados.

Para realizar una buena programación orientada a objetos existen disciplinas adicionales. Estas incluyen el **análisis orientado a objetos**, el **diseño orientado a objetos**, e incluso el proceso combinado y optimizado de **análisis y diseño orientado a objetos**. Todas estas disciplinas utilizan el concepto fundamental de los objetos de software y sus interacciones para analizar un problema y diseñar una solución. Las interacciones entre objetos incluyen la creación de objetos, la modificación de sus valores y su asociación con otros objetos.

El análisis, el diseño y la programación son etapas del desarrollo de software. Calificarlas como "orientadas a objetos" aclara qué tipo de técnicas de modelado se emplearán.

El **análisis orientado a objetos (OOA)** es el proceso de examinar un problema e identificar los objetos y las interacciones entre ellos. La etapa de análisis se centra en describir *qué* se debe hacer.

El resultado de la etapa de análisis es una descripción del problema y una solución al mismo, a menudo en forma de requisitos. Si completáramos la etapa de análisis en un solo paso, habríamos transformado una tarea del tipo *"Soy botánico y necesito un sitio web que ayude a los usuarios a clasificar plantas para que yo pueda colaborar en su correcta identificación"* en un conjunto de características requeridas. Como ejemplo, aquí hay algunos requisitos sobre lo que un visitante del sitio web podría necesitar hacer. Cada elemento es una acción vinculada a un objeto (*cursiva* para las acciones y **negrita** para los objetos):

- *Explorar* **cargas previas** (*Browse previous uploads*)
- *Subir* **nuevos ejemplos conocidos** (*Upload new known examples*)
- *Comprobar* la **calidad** (*Test for quality*)
- *Explorar* **productos** (*Browse products*)
- *Ver* **recomendaciones** (*See recommendations*)

En cierto modo, el término "análisis" puede ser equívoco. Un bebé que interactúa con objetos no analiza los bloques ni las piezas del rompecabezas. En cambio, explora su entorno, manipula formas y observa dónde encajan. Una mejor expresión podría ser **exploración orientada a objetos**. En el desarrollo de software, las etapas iniciales del análisis incluyen la exploración: entrevistar a los clientes, estudiar sus procesos y descartar posibilidades que no resuelven el problema.

El **diseño orientado a objetos (OOD)** es el proceso de convertir dichos requisitos en una especificación de implementación. El diseñador debe nombrar los objetos, definir los comportamientos y especificar formalmente qué objetos pueden activar comportamientos específicos en otros objetos. La etapa de diseño consiste en transformar el *qué se debe hacer* en *cómo se debe hacer*.

El resultado de la etapa de diseño es una especificación de implementación. Si completáramos la etapa de diseño en un solo paso, habríamos convertido los requisitos en un conjunto de especificaciones para las clases de objetos. La especificación podría incluir diagramas de transición de estados, diagramas de colaboración, diagramas de actividades y otros detalles útiles para describir el estado y el comportamiento. Estos quedan listos para implementarse en (idealmente) un lenguaje de programación orientado a objetos.

La **programación orientada a objetos (OOP)** es el proceso de convertir un diseño en un programa funcional. Este programa cumple lo que el propietario del producto solicitó originalmente durante la fase de análisis.

Sería ideal que el mundo funcionara de manera tan lineal. Si fuera así, podríamos seguir estas etapas una por una, en estricto orden, como un método bien definido para producir software. Como de costumbre, el mundo real es mucho más complejo. No importa cuánto intentemos separar estas etapas: siempre encontraremos aspectos que requieren un análisis más profundo mientras diseñamos. Y cuando estamos programando, descubrimos características que necesitan aclaraciones de diseño adicionales.

La mayoría de las prácticas modernas de desarrollo de software reconocen que una cascada (*waterfall*) de etapas no funciona bien. Lo que suele funcionar mejor es un **modelo de desarrollo iterativo**. En el desarrollo iterativo, se analiza, diseña y programa una pequeña parte de la tarea. Se puede decir que los desarrolladores forman un *scrum* (o melé, como en el rugby), donde todos los miembros del equipo empujan juntos en una misma dirección. El incremento de producto resultante se revisa. El desarrollo iterativo utiliza ciclos continuos para mejorar las características existentes y añadir nuevas funcionalidades. La metodología Scrum enfatiza este reajuste periódico del equipo seguido de la búsqueda enfocada de un objetivo a corto plazo. En algún momento, el producto es utilizable. El desarrollo en realidad nunca termina por completo, pero llega un punto en que resulta evidente que el costo de realizar mejoras adicionales supera los beneficios.

El resto de este libro trata sobre la OOP. En este capítulo, cubriremos los principios básicos orientados a objetos en el contexto del diseño. Esto nos permite comprender los conceptos sin tener que lidiar simultáneamente con las características sintácticas del lenguaje Python.

---

### Sección 1.3: Objetos y clases

Un objeto es una colección de datos que define un estado interno con comportamientos asociados. ¿Cómo diferenciamos entre categorías de objetos? Las manzanas y las naranjas son ambas objetos, pero existe el dicho común de que no se pueden comparar. Las manzanas y las naranjas no se modelan con mucha frecuencia en la programación informática, pero imaginemos que estamos creando una aplicación de inventario para una granja de frutas.

Una de las primeras cosas que aprendemos en nuestro análisis es que las manzanas van en barriles y las naranjas van en cestas. El dominio del problema que hemos descubierto hasta ahora tiene cuatro tipos, categorías o variedades de objetos: manzanas, naranjas, cestas y barriles. En el modelado orientado a objetos, el término preferido para un tipo de objeto es **clase** (*class*). Parecemos tener cuatro clases de objetos.

Es fundamental comprender la diferencia entre un objeto y una clase. La idea es que los objetos se pueden clasificar en función de estados o comportamientos comunes. Las clases describen objetos relacionados. La definición de una clase es como un plano (*blueprint*) para crear objetos individuales. Puedes tener tres naranjas sobre la mesa frente a ti. Cada naranja es un objeto distinto, pero las tres tienen los atributos y comportamientos asociados a una misma clase: la clase general de las naranjas.

La relación entre las cuatro clases de objetos en nuestro sistema de inventario se puede describir utilizando el Lenguaje Unificado de Modelado (invariablemente denominado **UML**, porque los acrónimos de tres letras nunca pasan de moda). Específicamente, utilizando un diagrama de clases UML. La Figura 1.1 representa nuestro primer diagrama de clases:

> **Figura 1.1: Diagrama de clases**  
> *(Orange — Basket | Apple — Barrel)*

Este diagrama de clases muestra que las instancias de la clase `Orange` (habitualmente llamadas "naranjas") están asociadas de algún modo con una `Basket`. También muestra cómo las instancias de la clase `Apple` ("manzanas") están asociadas de algún modo con un `Barrel`. La **asociación** es la forma más básica en la que los objetos (instancias de clases) pueden relacionarse. Al limitarnos a asociaciones simples, evitamos hacer suposiciones innecesarias. A menudo, una asociación requerirá una aclaración adicional. Podríamos, por ejemplo, sobrecargar el modelo con atributos adicionales —color, tamaño o cualquier otro aspecto—. El diagrama ayuda a centrar la discusión en las categorías de fruta y sus contenedores.

La sintaxis de un diagrama UML suele ser bastante intuitiva; no es necesario leer un tutorial extenso para entender (en su mayor parte) lo que ocurre al ver uno. UML también es bastante fácil de dibujar. Al fin y al cabo, muchas personas, al describir clases y sus relaciones, dibujan de forma natural rectángulos unidos por líneas. Disponer de un estándar basado en estos diagramas intuitivos facilita que los programadores se comuniquen con diseñadores, propietarios de producto y entre sí.

Ten en cuenta que el diagrama UML generalmente describe las definiciones de clase; a menudo necesitamos describir los atributos de los objetos. Los diagramas UML pueden mostrar una clase con atributos y métodos; también nos permiten omitir detalles innecesarios. El diagrama muestra la clase `Apple` y la clase `Barrel`, indicando que cualquier manzana dada está en algún barril específico. Aunque podemos usar UML para representar objetos individuales, rara vez es necesario. Mostrar las relaciones entre clases en un diagrama nos brinda detalles importantes sobre los objetos que son miembros de cada clase. No nos lo dice todo: el modelo sirve para resaltar detalles seleccionados del dominio general del problema.

Algunos programadores desprecian UML considerándolo una pérdida de tiempo. Basándose en el desarrollo iterativo, argumentan que las especificaciones formales elaboradas en elegantes diagramas UML quedarán obsoletas antes de implementarse. Además, se quejan de que mantener estos diagramas formales solo hace perder tiempo sin beneficiar a nadie.

Sin embargo, todo equipo de programación compuesto por más de una persona tendrá que sentarse ocasionalmente a discutir los detalles de los componentes que se están construyendo. Cuanto mayor sea el rendimiento del equipo en su conjunto, con mayor frecuencia se compartirá este tipo de información. UML es sumamente útil para garantizar una comunicación rápida, sencilla y consistente. Incluso aquellas organizaciones que desestiman los diagramas de clases formales suelen utilizar alguna versión informal de UML en sus reuniones de diseño o debates técnicos.

Además, la persona más importante con la que tendrás que comunicarte es **tu yo del futuro**. Todos creemos recordar las decisiones de diseño que tomamos, pero siempre habrá momentos de *"¿Por qué hice esto así?"* acechando en el futuro. Si conservamos los bocetos y notas sobre los que trazamos nuestros primeros diagramas, descubriremos con el tiempo que son una referencia invaluable.

Este capítulo, no obstante, no pretende ser un tutorial completo sobre UML. Existen muchos disponibles en internet y numerosos libros sobre el tema. UML abarca mucho más que diagramas de clases y objetos: incluye sintaxis para casos de uso, despliegue, cambios de estado y actividades. En este análisis de diseño orientado a objetos nos ocuparemos de la sintaxis más común de diagramas de clases. Podrás asimilar la estructura mediante ejemplos y, de forma casi intuitiva, utilizar esta sintaxis inspirada en UML en tus notas de diseño personales o de equipo.

Nuestro diagrama inicial, aunque correcto, no nos recuerda que las manzanas van en barriles ni cuántos barriles puede ocupar una sola manzana. Solo indica que las manzanas están asociadas de algún modo con los barriles. A veces la asociación entre clases es evidente y no requiere más explicaciones, pero a menudo debemos añadir mayor claridad.

La ventaja de UML radica en que la mayoría de los elementos son opcionales. Solo necesitamos especificar en el diagrama la cantidad de información que tenga sentido para la situación actual. En una sesión rápida frente a la pizarra, bastará con trazar líneas simples entre cajas. En un documento más formal y permanente, podremos profundizar en los detalles.

En el caso de las manzanas y los barriles, podemos estar seguros de que la asociación es: *muchas manzanas van en un barril*. Para asegurarnos de que nadie confunda la asociación con *una manzana estropea un barril*, podemos enriquecer el diagrama.

La Figura 1.2 muestra mayor nivel de detalle:

> **Figura 1.2: Diagrama de clases con mayor detalle**  
> *(Orange 1..* ── 1 Basket | Apple 1..* ── 1 Barrel)*

Este diagrama nos indica que las naranjas van en cestas, con una pequeña flecha indicando qué va dentro de qué. Leyéndolo en sentido inverso, una cesta contiene naranjas; esta es la relación fundamental **"tiene-un"** (*has-a*) que vemos prácticamente en todas partes. Existen muchos otros tipos de relaciones, pero nos centraremos en esta porque es común y fácil de visualizar. El diagrama también nos muestra la cantidad de objetos que pueden participar en la asociación en ambos lados de la relación. Un objeto `Basket` puede contener muchos objetos `Orange`, lo cual se representa anotando la línea con un `*`. Cualquier `Orange` individual puede estar exactamente en una `Basket`. Este número se denomina **multiplicidad** de la asociación, la cual especifica cuántas instancias de una clase pueden vincularse a otra. Aunque la multiplicidad a veces se confunde con la cardinalidad, esta última define un recuento exacto de elementos. En el contexto de UML, la multiplicidad es la forma en que definimos el rango permitido de valores de cardinalidad concretos. La propiedad de "más de una instancia" en una relación se describe mejor como multiplicidad.

A veces podemos olvidar en qué extremo de la línea debe colocarse cada anotación de multiplicidad. La anotación de multiplicidad más cercana a una clase representa cuántos objetos de esa clase pueden asociarse con un solo objeto en el otro extremo. Para la asociación de la manzana en el barril, leyendo de izquierda a derecha, muchas instancias de la clase `Apple` (es decir, muchos objetos `Apple`) pueden ir en un solo objeto `Barrel`. Leyendo de derecha a izquierda, exactamente un `Barrel` puede asociarse con cualquier `Apple`.

Hemos visto los fundamentos de las clases y cómo pueden especificar relaciones entre objetos. Ahora, debemos hablar sobre los atributos que definen el estado de un objeto y los comportamientos de un objeto que pueden implicar cambios de estado o interacción con otros objetos.

Los lectores con experiencia en diseño orientado a objetos notarán que no hemos descrito características comunes entre las clases `Barrel` o `Basket`. Estamos evitando intencionadamente la búsqueda de rasgos comunes por el momento. Un salto prematuro hacia la búsqueda de elementos compartidos puede a veces oscurecer distinciones más sutiles entre las clases de objetos.

---

### Sección 1.4: Especificación de atributos y comportamientos

Ahora ya dominamos parte de la terminología básica orientada a objetos. Los objetos son instancias de clases que pueden asociarse entre sí. Una naranja concreta sobre la mesa frente a nosotros es una instancia de la clase general de las naranjas. Como veremos en las siguientes secciones, los comportamientos se implementan como **métodos** de la clase, introduciendo un término más al conjunto.

La naranja tiene un estado (por ejemplo, madura o verde); implementamos el estado de un objeto mediante los valores de atributos específicos. Una naranja también tiene comportamientos. Por sí mismas, las naranjas suelen ser pasivas: se les imponen cambios de estado (imaginemos lo alarmante que sería una naranja activa). Profundicemos en el significado de estas dos palabras: **estado** y **comportamientos**.

#### 1.4.1 Los datos describen el estado del objeto

Comencemos con los datos. Los datos representan las características individuales de un objeto determinado: su estado actual. Una clase define las características de todos los objetos que son sus miembros. Cualquier objeto específico puede tener diferentes valores de datos para una característica dada. Por ejemplo, las tres naranjas en nuestra mesa (si no nos hemos comido ninguna) podrían pesar una cantidad diferente. La clase naranja podría tener un atributo `weight` para representar ese dato. Todas las instancias de la clase naranja tienen un atributo `weight`, pero cada naranja tiene un valor diferente para este atributo. Sin embargo, los atributos no tienen por qué ser únicos; dos naranjas cualesquiera pueden pesar exactamente lo mismo.

Los atributos se denominan frecuentemente **miembros** (*members*) o **propiedades** (*properties*). Algunos autores sugieren que estos términos tienen significados distintos: habitualmente, que los atributos son modificables (*settable*), mientras que las propiedades son de solo lectura. En Python, una propiedad puede definirse como de solo lectura, pero su valor se basará en valores de atributos que son —en última instancia— modificables, lo que hace que la distinción estricta sea irrelevante; a lo largo de este libro, utilizaremos ambos términos indistintamente. Además, como analizaremos en el Capítulo 5, la palabra clave `property` tiene un significado especial en Python para un tipo particular de atributo.

En Python, a un atributo también se le denomina **variable de instancia** (*instance variable*). Esto ayuda a clarificar cómo funcionan los atributos: son variables con valores únicos para cada instancia de una clase. Python dispone de otros tipos de atributos, pero nos centraremos en el más común para empezar.

En nuestra aplicación de inventario de frutas, el agricultor querrá saber de qué huerto proviene la naranja, cuándo fue recolectada y cuánto pesa. También querrá llevar un registro de dónde se almacena cada `Basket`. Las manzanas podrían tener un atributo `color`, y los barriles podrían tener diferentes tamaños.

Con frecuencia notaremos que varias clases comparten las mismas propiedades; por ejemplo, también queremos saber cuándo se recolectan las manzanas. Para este primer ejemplo, agregaremos algunos atributos a nuestro diagrama de clases. La Figura 1.3 ilustra algunos de ellos:

> **Figura 1.3: Diagrama de clases con atributos**  
> *(Orange: weight, orchard, date_picked | Basket: location | Apple: color, orchard, date_picked | Barrel: size)*

Dependiendo del nivel de detalle que requiera nuestro diseño, también podemos especificar el tipo para el valor de cada atributo. En UML, los tipos de atributos suelen expresarse con nombres genéricos comunes a muchos lenguajes de programación, tales como `integer`, `floating-point number`, `string`, `byte` o `Boolean`. No obstante, también pueden representar colecciones genéricas como listas, árboles o grafos o, más notablemente, otras clases específicas de la aplicación. Esta es una de las áreas donde la etapa de diseño se solapa con la de programación: las primitivas y colecciones integradas en un lenguaje pueden ser distintas a las de otro.

La Figura 1.4 muestra atributos con pistas de tipo (*type hints*) enfocadas (en su mayoría) en Python:

> **Figura 1.4: Diagrama de clases con atributos y sus tipos**  
> *(Orange: weight: float, orchard: str, date_picked: date, basket: Basket)*

Por lo general, no necesitamos preocuparnos en exceso por los tipos de datos en la fase de diseño, ya que los detalles específicos de implementación se eligen durante la programación. Los nombres genéricos son normalmente suficientes para el diseño; por eso incluimos `date` como marcador de posición para un tipo de Python como `datetime.datetime`. Si nuestro diseño requiere un tipo de contenedor de lista, los programadores de Java pueden optar por utilizar un `LinkedList` o un `ArrayList` al implementarlo, mientras que los programadores de Python (¡nosotros!) podemos especificar `list[Apple]` como pista de tipo y utilizar el tipo `list` para la implementación.

En nuestro ejemplo de inventario de frutas, los atributos mostrados hasta ahora son tipos primitivos integrados. Sin embargo, existen atributos implícitos que podemos hacer explícitos; estos implementan las asociaciones. Para una naranja determinada, tenemos un atributo que hace referencia a la cesta que la contiene: el atributo `basket`, con la pista de tipo `Basket`.

#### 1.4.2 Los comportamientos son acciones

Ahora que sabemos cómo los datos definen el estado de un objeto, el último concepto que debemos revisar es el **comportamiento**. Los comportamientos son acciones que pueden ocurrir sobre un objeto. Los comportamientos que se pueden ejecutar sobre los miembros de una clase se expresan como los **métodos** de la clase. A nivel de programación, los métodos son esencialmente funciones que tienen acceso a los atributos del objeto (en Python, las variables de instancia). Al igual que las funciones, los métodos pueden aceptar parámetros y devolver valores.

Los parámetros de un método definen los objetos que deben pasarse a dicho método. Las instancias de objetos reales que se pasan durante una invocación específica se denominan **argumentos**. Estos objetos se vinculan a los nombres de las variables de los parámetros en el cuerpo del método. El método los utiliza para realizar cualquier comportamiento o tarea prevista. Los valores devueltos son los resultados de dicha tarea. Las modificaciones en el estado interno son un posible efecto secundario de la evaluación de un método (algunas personas se refieren a esto como "llamar" o "ejecutar" un método; son términos sinónimos).

Hemos ampliado nuestro ejemplo de comparar manzanas y naranjas para convertirlo en una aplicación de inventario básica. Llevémoslo un paso más allá para ver hasta dónde llega. La idea es capturar suficientes detalles del dominio del problema; el software final que escribamos puede no necesitar implementar cada detalle del modelo inicial. Una acción que se puede asociar con las naranjas es la acción conceptual `pick` (recolectar). Al pensar en los detalles de implementación de esta clase, un método `pick` podría necesitar realizar dos cosas:

1. Colocar la naranja en una cesta actualizando el atributo `basket` de la naranja.
2. Añadir la naranja a la lista de naranjas en la `Basket` correspondiente.

Por lo tanto, este método `pick` necesita saber con qué cesta está interactuando. Hacemos esto proporcionando al método un parámetro de tipo `Basket`. Dado que nuestro agricultor también vende zumo, podemos añadir un método `squeeze` (exprimir) a la clase `Orange`. Al ejecutarse, el método `squeeze` podría devolver la cantidad de zumo obtenida, al mismo tiempo que elimina la `Orange` de la `Basket` en la que se encontraba.

La clase `Basket` puede tener una acción `sell` (vender). Cuando se vende una cesta, nuestro sistema de inventario podría actualizar datos en objetos aún no especificados para cálculos contables y de beneficios. Alternativamente, nuestra cesta de naranjas podría estropearse antes de que podamos venderla, por lo que también podríamos necesitar un método `discard` (descartar).

La Figura 1.5 añade métodos a nuestro diagrama:

> **Figura 1.5: Diagrama de clases con atributos y métodos**  
> *(Orange: +pick(basket: Basket), +squeeze(): float | Basket: +sell(), +discard())*

¿Realmente necesitamos todos estos métodos y asociaciones? Como es de esperar, la respuesta suele ser "no": el software de la aplicación no necesita modelar todos estos comportamientos. Por ahora, queremos lanzar ideas al modelo para explorar qué es posible. Más adelante, recortaremos esto hasta quedarnos con lo estrictamente necesario.

Añadir atributos y métodos a objetos individuales nos permite crear un **sistema de objetos interactivos**. Cada objeto del sistema es miembro de una clase determinada. Estas clases especifican qué tipos de datos puede contener el objeto y qué métodos pueden invocarse sobre él. Los datos de cada objeto pueden estar en un estado diferente al de otras instancias de la misma clase; cada objeto puede reaccionar a las llamadas a métodos de manera distinta debido a esas diferencias de estado.

El análisis y diseño orientados a objetos consisten en descubrir cuáles son esos objetos y cómo deben interactuar. Cada clase tiene responsabilidades y colaboraciones. La siguiente sección describe los principios que se pueden aplicar para hacer que esas interacciones sean lo más intuitivas posible.

Observa que vender una cesta no es incondicionalmente una característica de la clase `Basket`. Es posible que alguna otra clase (no mostrada) se encargue de supervisar las cestas y su ubicación. A menudo establecemos límites alrededor de nuestro diseño. También nos surgirán dudas sobre las responsabilidades asignadas a las distintas clases. El problema de asignación de responsabilidades no siempre tiene una solución técnica única, lo que nos obliga a dibujar (y redibujar) nuestros diagramas UML más de una vez para examinar diseños alternativos.

En muchos contextos, el proceso de análisis también puede resultar esclarecedor para los propietarios de producto y los usuarios. Lo que al principio parecía un problema de negocio insuperable que requería gran cantidad de software costoso, puede resultar ser simplemente una falta de colaboración entre dos departamentos. El proceso de creación de un modelo analítico orientado a objetos puede revelar detalles que no se resuelven necesariamente con más software. No es raro que un proyecto avance a través de un extenso modelado orientado a objetos y concluya con un resultado exitoso para los usuarios finales sin que apenas se haya escrito código. Los modelos (y la comprensión resultante) fueron suficientes para entender lo que realmente ocurría.

---

### Sección 1.5: Ocultación de detalles y creación de la interfaz pública

El propósito principal de modelar un objeto en el diseño orientado a objetos es determinar cuál será la **interfaz pública** de ese objeto. La interfaz es el conjunto de atributos y métodos a los que otros objetos pueden acceder para interactuar con él. Los objetos no necesitan —y en algunos lenguajes no se les permite— acceder a los mecanismos internos de los objetos de otra clase.

Un ejemplo cotidiano muy común es la televisión (o cualquier electrodoméstico). Nuestra interfaz con la televisión es el mando a distancia. Cada botón del mando a distancia representa un método que se puede invocar en el objeto televisión. Cuando nosotros, como objeto invocador, accedemos a estos métodos, no sabemos ni nos importa si la televisión recibe la señal a través de cable, antena parabólica o un dispositivo conectado a internet. La televisión es un conglomerado complejo de componentes. No nos importa qué señales electrónicas se envían para ajustar el volumen, ni si el sonido se dirige a altavoces internos o auriculares. Si abrimos la televisión para acceder a su circuito interno (por ejemplo, para desviar la señal de salida tanto a altavoces externos como a auriculares), podríamos anular la garantía.

Este proceso de ocultar los detalles de implementación de un objeto se denomina **ocultación de información** (*information hiding*). También se le conoce a veces como **encapsulamiento** (*encapsulation*). Los datos encapsulados no están necesariamente ocultos. El encapsulamiento es, literalmente, crear una cápsula (o envoltorio) alrededor de los atributos. La carcasa exterior de la televisión encapsula el estado y el comportamiento del televisor: tenemos acceso a la pantalla, los altavoces y el mando, pero no tenemos acceso directo al cableado de los amplificadores o receptores internos.

Cuando compramos un sistema de entretenimiento por componentes modulares, modificamos el nivel de encapsulamiento, exponiendo más interfaces entre componentes. Si somos aficionados a la electrónica o al *Internet de las Cosas*, podemos descomponerlo aún más, abriendo las carcasas y rompiendo la ocultación de información diseñada por el fabricante.

La distinción entre encapsulamiento y ocultación de información es sutil en el diseño. Muchas referencias prácticas utilizan estos términos indistintamente. Como programadores de Python, en realidad no disponemos ni necesitamos una ocultación de información mediante variables completamente privadas e inaccesibles (analizaremos las razones de esto en el Capítulo 2), por lo que la definición más amplia de encapsulamiento resulta perfectamente adecuada.

La interfaz pública de una clase es sumamente importante. Debe diseñarse con sumo cuidado, ya que puede ser difícil de modificar una vez que el software ha sido escrito y otras clases dependen de ella. Podemos cambiar los componentes internos todo lo que queramos (por ejemplo, para hacerlo más eficiente o acceder a datos a través de la red en lugar de localmente), y los objetos cliente podrán seguir comunicándose con él, sin modificaciones, utilizando la interfaz pública. Por el contrario, si alteramos la interfaz cambiando nombres de atributos de acceso público o el orden y tipo de los argumentos que acepta un método, todas las clases cliente también deberán modificarse. Al diseñar interfaces públicas, **mantenlo simple**. Diseña siempre la interfaz de un objeto en función de lo fácil que resulte de utilizar, no de lo difícil que sea de programar (este consejo se aplica igualmente a las interfaces de usuario). Por esta razón, a menudo verás variables en Python con un guion bajo inicial (`_`) en su nombre como advertencia de que no forman parte de la interfaz pública.

Recuerda: los objetos del programa pueden representar objetos reales, pero eso no los convierte en objetos reales. Son **modelos**. Uno de los mayores beneficios del modelado es la capacidad de ignorar detalles irrelevantes. La maqueta de coche que uno de los autores construyó de niño parecía un Thunderbird de 1956 real por fuera, pero evidentemente no funcionaba. Cuando era demasiado joven para conducir, esos detalles mecánicos eran excesivamente complejos e irrelevantes. El modelo es una **abstracción** de un concepto real.

La **abstracción** es otro término orientado a objetos estrechamente relacionado con el encapsulamiento y la ocultación de información. La abstracción significa trabajar con el nivel de detalle más apropiado para una tarea determinada. Es el proceso de extraer una interfaz pública a partir de los detalles internos. El conductor de un automóvil interactúa con el volante, el acelerador y los frenos. El funcionamiento interno del motor, la transmisión y el sistema de frenos no le incumben al conductor. Un mecánico, por otro lado, trabaja en un nivel de abstracción diferente: afinando el motor y purgando los frenos.

La Figura 1.6 muestra dos niveles de abstracción para un automóvil:

> **Figura 1.6: Niveles de abstracción de un automóvil**  
> *(Nivel Conductor: Volante, Pedales | Nivel Mecánico: Motor, Circuito de frenado, Transmisión)*

Ahora contamos con varios términos que hacen referencia a conceptos similares. Resumamos toda esta terminología en un par de frases: la **abstracción** es el proceso de encapsular información con una interfaz pública diferenciada. Cualquier elemento privado puede estar sujeto a la **ocultación de información**. En los diagramas UML, podemos utilizar un signo `-` en lugar de un `+` inicial para indicar que un elemento no forma parte de la interfaz pública.

La lección fundamental de todas estas definiciones es hacer que nuestros modelos sean comprensibles para otros objetos que deban interactuar con ellos. Esto implica prestar atención a los pequeños detalles:

- Asegúrate de que los métodos y propiedades tengan **nombres claros y coherentes**. Al analizar un sistema, los objetos representan normalmente los **sustantivos** del problema original, mientras que los métodos son los **verbos**. Los atributos suelen aparecer como **adjetivos** u otros sustantivos. Nombra tus clases, atributos y métodos en consecuencia.
- Al diseñar la interfaz, imagínate que eres el objeto: deseas definiciones claras de tus responsabilidades y tienes una marcada preferencia por la privacidad para poder cumplir con ellas. No permitas que otros objetos accedan a tus datos a menos que sea estrictamente necesario. No ofrezcas a otras clases una interfaz que te obligue a realizar una tarea a menos que estés seguro de que es tu responsabilidad llevarla a cabo.

---

### Sección 1.6: Principios de diseño

El diseño orientado a objetos no es sencillo. Ningún diseño lo es. Existen una serie de principios rectores que ayudan a tomar decisiones. Uno de los conjuntos de principios más conocidos es **SOLID**. Este es un acrónimo práctico para cinco ideas que pueden transformar un diseño desordenado en una estructura sólida, coherente y fácil de mantener.

Utilizaremos estos principios a lo largo de todo el libro. Esta es solo una introducción inicial. Los cinco principios son:

1. **S**ingle Responsibility Principle (*Principio de Responsabilidad Única*)
2. **O**pen/Closed Principle (*Principio de Abierto/Cerrado*)
3. **L**iskov Substitution Principle (*Principio de Sustitución de Liskov*)
4. **I**nterface Segregation Principle (*Principio de Segregación de Interfaces*)
5. **D**ependency Inversion Principle (*Principio de Inversión de Dependencias*)

Estos principios se aplican ampliamente en el diseño orientado a objetos. Cabe señalar que el principio de sustitución de Liskov se centra en la herencia y en la relación "es-un" (*is-a*), algo que hemos evitado en los ejemplos anteriores.

El orden del acrónimo SOLID es útil para memorizarlos, pero no es la forma más práctica de explicarlos. Los abordaremos en una secuencia más intuitiva.

#### 1.6.1 Principio de Segregación de Interfaces (Interface Segregation Principle)

Tratamos este principio en primer lugar porque es esencial para entender los límites que rodean la definición de una clase. Al preguntarnos qué encapsular, resulta fundamental mantener la interfaz lo más reducida posible. Cuando un objeto es demasiado complejo, la interfaz puede crecer reflejando esa complejidad, obligando a las clases colaboradoras a depender de métodos y atributos que en realidad no necesitan.

El objetivo es mantener la interfaz reducida. Esto minimiza la carga cognitiva para entender la clase y garantiza que otras clases puedan evolucionar y cambiar sin provocar problemas derivados de dependencias no deseadas o imprevistas.

#### 1.6.2 Principio de Abierto/Cerrado (Open/Closed Principle)

Uno de los ingredientes clave en un buen diseño es contar con definiciones de clase que estén **abiertas a la extensión pero cerradas a la modificación**. Queremos poder añadir funcionalidades a una clase mediante técnicas como la herencia y la composición (las cuales analizaremos en detalle en el Capítulo 3), sin tener que retocar ni alterar el código fuente ya implementado.

Un aspecto de este principio consiste en recurrir a la composición de múltiples objetos para generar comportamientos complejos. Esto encaja con el Principio de Segregación de Interfaces al separar funcionalidades en clases diferenciadas. De este modo, podemos extender una de esas clases sin riesgo de romper el resto de las clases de la aplicación o librería.

El otro aspecto consiste en crear clases que hereden características de una clase base. A través de la herencia, la subclase se puede utilizar en cualquier lugar donde se espere la clase base, pero realizando una tarea más especializada, útil o adecuada que la que ofrecía la clase base. Hay más detalles sobre esta idea de herencia reflejados en el Principio de Sustitución de Liskov.

#### 1.6.3 Principio de Sustitución de Liskov (Liskov Substitution Principle)

Este principio —nombrado en honor a Barbara Liskov, creadora de uno de los primeros lenguajes orientados a objetos, CLU— ofrece directrices para delimitar el uso de la herencia. Dejaremos los detalles específicos para más adelante y nos centraremos en el objetivo general de un diseño bien estructurado.

Si disponemos de una clase base como `Container`, queremos que todas las subclases (`Barrel`, `Basket` y cualquier otra que necesitemos inventar) mantengan la misma interfaz que la clase base. Todas son contenedores, cada una con detalles de implementación únicos. Al compartir la misma interfaz, cualquiera de las subclases puede utilizarse en sustitución de la clase base.

Cuando utilizamos herramientas como `mypy` o `pyright` para comprobar nuestras anotaciones de tipo, estas herramientas nos advertirán sobre violaciones de la Sustitución de Liskov. Los errores señalarán los puntos exactos donde la interfaz de una subclase no cumple la promesa establecida por la clase base. Analizaremos esto en detalle en el Capítulo 7.

#### 1.6.4 Principio de Inversión de Dependencias (Dependency Inversion Principle)

El nombre de este principio puede resultar confuso: ¿invertido respecto a qué? Si no sabemos cuál es la posición "normal", ¿cómo juzgamos si la dependencia está "al revés"?

La dependencia más obvia y directa consiste en que una clase mencione explícitamente la clase de objetos con la que colabora. Esto es sencillo y práctico para ejemplos de tutoriales o clases introductorias de programación. A largo plazo, sin embargo, cuando una clase depende directamente de otra clase concreta, surgen dificultades al realizar cambios.

Imaginemos el código Python de la clase `Apple` haciendo referencia directa a la clase `Barrel`. Esto se convierte en un problema del Principio de Abierto/Cerrado. Cuando necesitemos comenzar a transportar manzanas en grandes cajas de embalaje, la nueva clase `PackingCrate` será una extensión de la clase `Barrel`, que a su vez era una extensión de una clase base abstracta `Container`. No queremos tener que editar grandes bloques de código solo para incorporar la nueva clase `PackingCrate`.

La idea de la **inversión de dependencias** establece que la clase `Apple` solo debe hacer referencia a la clase base `Container`. De este modo, cualquier tipo de contenedor en el árbol genealógico podrá asociarse con una instancia de `Apple`. La relación concreta entre la clase `Apple` y la clase `Barrel` debe ser algo configurado en tiempo de ejecución; no debe estar acoplada en la estructura fundamental del código.

La idea de utilizar la clase base refuerza el Principio de Sustitución de Liskov y ayuda a implementar el Principio de Abierto/Cerrado. Como veremos más adelante, este principio actúa como un detalle de implementación que asegura el cumplimiento de los demás principios.

#### 1.6.5 Principio de Responsabilidad Única (Single Responsibility Principle)

Este principio, aunque presentado habitualmente en primer lugar, actúa en realidad como un resumen de los anteriores: **una clase debe tener una única responsabilidad**.

Para alcanzar este ideal, debemos comenzar segregando las interfaces. Una vez que hemos descompuesto una clase en partes más sencillas para simplificar las interfaces, debemos asegurarnos de mantener el diseño abierto a la extensión pero cerrado a la modificación.

Tras estos pasos iniciales, debemos revisar los detalles para asegurarnos de que la Sustitución de Liskov se cumpla. Y, por supuesto, debemos evitar dependencias rígidas mediante la inversión de dependencias, haciendo que el código dependa únicamente de clases base o abstracciones.

Una vez que hemos reflexionado sobre nuestro diseño aplicando estos principios, descubriremos que nuestras clases tienen una responsabilidad única y fácil de resumir. A partir de ahí, podemos trabajar en cómo colaboran los objetos para crear las funcionalidades deseadas del software.

Esto es, por supuesto, solo un esquema introductorio. Cada principio conlleva consecuencias y consideraciones que desarrollaremos a lo largo de este libro.

---

### Sección 1.7: Colaboración entre objetos

Hemos abordado dos componentes fundamentales de la programación orientada a objetos: la **clase** y la **responsabilidad**. Clasificamos los objetos en función de su estado interno y su comportamiento, y procuramos definir una responsabilidad clara y enfocada para cada clase.

El componente restante es la **colaboración**. Una vez que descomponemos un problema en clases separadas, el comportamiento de la aplicación final surgirá de la colaboración entre objetos de esas clases. La aplicación final —al igual que una buena salsa— es una mezcla equilibrada de ingredientes que se complementan mutuamente.

Existen diversas maneras de enfocar la colaboración entre clases. Dejaremos los detalles específicos sobre los distintos tipos de colaboración para el Capítulo 3. Aquí introduciremos dos conceptos clave:

- **Composición** (*Composition*)
- **Herencia** (*Inheritance*)

Cuando aplicamos el Principio de Segregación de Interfaces, solemos descomponer algo complejo en un conjunto de elementos más simples. En muchos casos, podemos diseñar un modelo del elemento complejo original como una composición de esos elementos más simples. Ya vimos cómo funciona la composición al hablar de los automóviles: un vehículo de combustible fósil se compone de motor, transmisión, motor de arranque, faros y parabrisas, entre muchas otras piezas. Cada una de ellas puede descomponerse a su vez en componentes activos con estado. El sistema de faros, por ejemplo, puede estar apagado, encendido o con luces largas; un control modifica el estado de este sistema, y puede existir un sensor automatizado para encender las luces de noche y atenuarlas ante tráfico en sentido contrario.

La descomposición plantea un problema potencial: ¿qué ocurre si tenemos varios elementos con aspectos comunes? Tenemos faros, luces interiores y un sistema de sonido, y todos ellos utilizan fusibles. ¿Queremos repetir la propiedad *"depende de un fusible"* en toda la jerarquía de clases? Eso implicaría duplicar código y causaría problemas de mantenimiento cuando se produzcan cambios en la implementación.

Para evitar la repetición, podemos utilizar la **herencia**. Resulta útil concebir la herencia como una relación **"es-un"** (*is-a*). Podemos extraer una característica común, como *"protegido por un fusible"*. Cada componente eléctrico del automóvil es un componente protegido por un fusible: el sistema de faros es un componente protegido por un fusible; el sistema de sonido es un componente protegido por un fusible.

Hacemos esto definiendo una clase base: `Fused`. A continuación, definimos subclases que extienden esta clase base. Muchos autores utilizan el término **superclase** en lugar de clase base. En los diagramas UML, la clase base suele representarse en la parte superior; sin embargo, conceptualmente es fundamental, y las demás clases se construyen sobre ella.

El Principio de Sustitución de Liskov (junto con el Principio de Abierto/Cerrado) establece las pautas para la herencia: debemos asegurarnos de que cada subclase mantenga la misma interfaz que la clase base, de modo que cualquier subclase pueda sustituir a la superclase.

Al pensar en que tanto los faros como el equipo de sonido tienen fusibles, esto puede parecer curioso. No encendemos la radio para iluminar la carretera de noche. Las subclases no realizan todas la misma tarea; simplemente comparten una interfaz coherente. Tanto los faros como el equipo de sonido tienen una interfaz (un par de cables) hacia el fusible y hacia la toma de tierra. Esta interfaz cumple con el Principio de Sustitución de Liskov.

Cuando añadimos luces diurnas al sistema de faros, respetamos la Sustitución de Liskov porque las luces diurnas utilizan el mismo fusible. El uso de cables largos y flexibles deja los sistemas del vehículo abiertos a la extensión. El uso de un conjunto de faros sellado hace que las luces estén cerradas a la modificación: si queremos cambiarlas, reemplazamos el conjunto completo en lugar de manipular la bombilla individualmente.

Ahora que hemos comenzado a explorar el diseño orientado a objetos, examinaremos un fragmento de software heredado que no siguió estos principios de diseño, para comenzar a pensar en cómo refactorizarlo desde un bloque desordenado de instrucciones hacia definiciones de clases más comprensibles.

---

### Sección 1.8: Un posible desorden

Es común que quienes conocen algo de Python pero no han utilizado a fondo las características de la POO se pregunten si todo este esfuerzo mental realmente se traduce en un mejor software. Abordaremos primero algunas dudas frecuentes (en realidad, objeciones planteadas como preguntas) y luego analizaremos un ejemplo concreto de cómo transformar un script tradicional en objetos.

Una pregunta habitual es: *“¿No es una clase simplemente un grupo de funciones con datos compartidos?”* La respuesta corta es **sí**. La característica orientada a objetos esencial aquí es agrupar los datos y las funciones relacionadas en un único espacio de nombres (*namespace*), denominado definición de clase. Cuando escribimos un lote de funciones estrechamente vinculadas, solemos darles nombres similares para evidenciar su relación. Este es el propósito de una clase: proporcionar un contenedor común para funciones relacionadas.

Además, una definición de clase nos permite crear múltiples instancias de los datos compartidos. Esto ayuda a encapsular el procesamiento para múltiples objetos con comportamiento similar pero estados distintos.

Otra pregunta frecuente es: *“¿Por qué una colección de definiciones de clase es más fácil de entender que una sola función extensa?”* La respuesta corta es la **fragmentación en bloques** (*chunking*). Para retener ideas complejas en nuestra mente, dividimos la información en fragmentos más pequeños. Por ejemplo, no leemos un número telefónico largo como una secuencia desordenada de dígitos; lo descomponemos en bloques. Por eso introducimos signos de puntuación en los números de teléfono (en Norteamérica se escribe `(111)222-3333` para dividir un número de 10 dígitos en tres bloques manejables). Al hablar del "interior" o del "motor" de un automóvil, estamos descomponiendo un todo complejo en partes intelectualmente más accesibles.

Un script o una función muy extensa suele ser difícil de entender. El programador a menudo divide el script largo en secciones mediante comentarios, a veces con grandes encabezados que anuncian pasos importantes del procesamiento. Cada una de esas secciones podría haber sido una función más pequeña. Y funciones más pequeñas estrechamente relacionadas a menudo gestionan el estado de un único objeto: estos son los métodos de una clase.

#### 1.8.1 Lectura de un script grande

Imaginemos un script largo en Python que resume información de varios archivos en formato JSON. Abre archivos, analiza el contenido JSON, localiza detalles y genera un resumen consolidado. Realiza muchas operaciones y refleja una complejidad mal gestionada. Este es un esquema del código:

```python
import json
from pathlib import Path
import shlex

def main():

    optional = {"type"}

    result_dir = Path.cwd() / "data"
    for file in result_dir.glob("*.json"):
        # 1. Load file
        result = json.loads(file.read_text())
        # 2. Set Outcome
        app_name = file.stem
        env_outcome = None
        # 3. Examine environments
        for env_name, env in result['testenvs'].items():
            # 2a. Skip special names
            if env_name.startswith("."):
                continue
            # 2b. Accumulate outcomes
            if env:
                if env['result']['success']:
                    if env_outcome is None:
                        env_outcome = "ok"
                else:
                    for step in env['test']:
                        if step['retcode'] != 0:
                            command = Path(step['command'][0]).stem
                            args = shlex.join(step['command'][1:])
                            message = f"{env_name} failed {command} {args}"
                            if env_outcome is None or env_outcome == "ok":
                                env_outcome = message
                            else:
                                env_outcome = f"{env_outcome}, {message}"
            else:
                if env_outcome is None:
                    env_outcome = f"{env_name} did not run"
                elif env_outcome == "ok" and env_name in optional:
                    env_outcome = f"ok (except {env_name})"
                else:
                    env_outcome = f"{env_outcome}, {env_name} did not run"
        # 4. Write summary
        print(f"{app_name:20s} {env_outcome}")

if __name__ == "__main__":
    main()
```

Este script tiene casi 50 líneas de código. Dentro de esta función se producen continuos cambios de enfoque: primero las rutas de archivos, luego el documento JSON en cada ruta, luego los entornos probados y finalmente los comandos ejecutados. En última instancia, existen reglas complejas que definen el estado final que se imprime. Estos cambios de contexto pueden tener sentido para el autor original, pero resultan difíciles de asimilar para cualquier otra persona.

Además, por supuesto, esta complejidad resulta muy difícil de probar mediante pruebas unitarias.

Enterradas en medio del desorden de detalles de procesamiento se encuentran algunas ideas esenciales. Este código procesa un conjunto de instancias de aplicaciones donde cada aplicación se prueba con la herramienta `tox`. Dicha herramienta genera los archivos en formato JSON con los resultados de las pruebas de cada aplicación (esta herramienta es una de las muchas disponibles en PyPI que se añaden comúnmente a los proyectos para automatizar pruebas). La herramienta ejecuta cada aplicación en varios entornos. Cada entorno contiene una serie de comandos que utilizan herramientas como `pytest`, `pyright` y `ruff`. Un entorno puede tener un resultado donde el atributo `success` sea verdadero (`true`), lo que significa que todos los comandos funcionaron correctamente; de lo contrario, al menos un comando falló.

Observa cómo empezamos a resaltar los conceptos clave que podrían implementarse como clases de objetos: una **aplicación**, varias instancias de **entornos** y varias instancias de **comandos**. Un comando, por ejemplo, tiene una representación JSON como una lista de cadenas. Un entorno tiene una representación JSON como una simple cadena `"3.13"`, con un diccionario de detalles complementarios.

El script se sumerge en detalles de bajo nivel sobre archivos, entornos y comandos. Es raro que un script de este tipo proporcione una visión general clara de los tres posibles resultados para cada aplicación probada:

1. Todos los comandos en todos los entornos fueron exitosos. La aplicación está lista para despliegue.
2. Un comando falló en un entorno. La aplicación requiere depuración.
3. Ocurrió algún otro problema y no existe ningún archivo JSON. Esto también indica que la aplicación no está lista para despliegue, o bien sugiere que algo falló en el entorno de pruebas en su totalidad.

Identificamos tres clases de objetos en el dominio del problema:

- Una **aplicación** (`Application`), asociada con uno o más entornos. La aplicación también se asocia con un resumen que consolida los detalles de entornos y comandos en una decisión final.
- Un **entorno** (`Environment`), asociado con uno o más comandos. El objeto entorno contendrá un resumen de los comandos en forma de estado de éxito o fracaso.
- Un **comando** (`Command`), que contiene los detalles de cada paso ejecutado. Estos resultan de especial interés cuando registran un fallo.

Al revisar el script, observamos una gran cantidad de navegación a través de estructuras de datos JSON. Aunque este es un detalle de implementación importante, tiende a oscurecer el objetivo principal de comprender aplicaciones y entornos.

Conviene notar que a menudo pasamos por alto otras categorías de objetos que forman parte de los detalles de implementación:

- El objeto `Path` con su método `glob()` para localizar todos los archivos.
- Los objetos `dict` creados por el módulo `json`.
- Los objetos `list` que contienen los comandos dentro de un entorno.

Al programar en Python, resulta útil reconocer que estas clases de implementación —`pathlib.Path`, `dict` y `list`— son la esencia misma de la programación orientada a objetos. Son clases que nosotros no escribimos, pero que utilizamos para construir nuestras aplicaciones. Resulta que partes de cualquier script en Python ya son orientadas a objetos, incluso si el script en su conjunto no presenta un diseño formal robusto.

Revisar el script nos permite enfatizar el procesamiento de aplicaciones, entornos y comandos. Conviene reflexionar sobre cómo transformar este código hacia un modelo orientado a objetos. No entraremos en esos detalles ahora; en su lugar, exploraremos las ideas detrás de la refactorización a lo largo del libro. Terminamos el capítulo habiendo expuesto el problema y trazado un camino hacia la solución. Hay dos conceptos interrelacionados aquí:

- **Python ya es orientado a objetos:** los tipos integrados se basan en definiciones de clases.
- **Un buen diseño orientado a objetos desplaza el foco:** pasa de los detalles de implementación a los conceptos esenciales del problema que se resuelve.

---

### Sección 1.9: Repaso

Los siguientes son algunos de los puntos clave tratados en este capítulo:

- Análisis de requisitos de problemas en un contexto orientado a objetos.
- Cómo trazar diagramas UML para comunicar el funcionamiento del sistema.
- Discusión de sistemas orientados a objetos utilizando la terminología y jerga técnica correcta.
- Comprensión de la distinción entre clase, objeto, atributo y comportamiento.

---

### Sección 1.10: Ejercicios

Este es un libro práctico. Por lo tanto, no asignaremos problemas ficticios de análisis orientado a objetos para que crees diseños artificiales. En su lugar, queremos darte ideas que puedas aplicar directamente a tus propios proyectos. Si tienes experiencia previa con la orientación a objetos, no necesitarás dedicar demasiado esfuerzo a este capítulo; no obstante, son ejercicios mentales valiosos si has estado usando Python durante un tiempo pero nunca has profundizado en el diseño con clases.

1. **Analiza un proyecto reciente:** Piensa en un proyecto de programación reciente que hayas completado. Identifica el objeto más prominente en el diseño. Intenta pensar en la mayor cantidad de atributos posible para este objeto. ¿Tenía color, peso, tamaño, beneficio, coste, nombre, número de ID, precio o estilo?
2. **Evalúa tipos y métodos:** Reflexiona sobre los tipos de los atributos. ¿Eran tipos primitivos o clases? ¿Eran algunos de esos atributos en realidad comportamientos disfrazados? A veces, lo que parece un dato se calcula a partir de otros datos del objeto, y puedes utilizar un método para realizar esos cálculos. ¿Qué otros métodos o comportamientos tenía el objeto? ¿Qué objetos llamaban a esos métodos? ¿Qué tipo de relaciones tenían con este objeto?
3. **Analiza un proyecto futuro:** Piensa en un proyecto próximo (puede ser un proyecto personal de tiempo libre o un contrato corporativo importante; no tiene que ser una aplicación completa, puede ser simplemente un subsistema). Realiza un análisis orientado a objetos básico. Identifica los requisitos y los objetos que interactúan. Esboza un diagrama de clases con el nivel más alto de abstracción de ese sistema. Identifica los objetos principales de interacción y los objetos secundarios de soporte. Detalla los atributos y métodos de los más interesantes.
   - El objetivo no es diseñar un sistema completo ahora mismo, sino reflexionar sobre el diseño orientado a objetos desde la perspectiva de la clase, la responsabilidad y la colaboración. Enfocarte en proyectos reales en los que hayas trabajado o vayas a trabajar hace que el ejercicio sea práctico y aplicable.
4. **Explora recursos de UML:** Visita tu motor de búsqueda preferido y consulta tutoriales sobre UML. Existen multitud de ellos; encuentra el que mejor se adapte a tu estilo de estudio. Dibuja algunos diagramas de clases o un diagrama de secuencia para los objetos identificados anteriormente. No te obsesiones con memorizar la sintaxis (siempre puedes consultarla cuando la necesites); simplemente familiarízate con el lenguaje visual. Te resultará mucho más sencillo comunicarte en tus próximas discusiones sobre POO si puedes esbozar rápidamente un diagrama.
   - Los diagramas UML de este libro se elaboraron con la herramienta PlantUML. La documentación de esta herramienta incluye numerosos ejemplos de diagramas que ilustran cómo describir objetos y sus relaciones. Muchas personas también utilizan [Mermaid](https://mermaid.live/) para crear diagramas UML.

---

### Sección 1.11: Resumen

En este capítulo, realizamos un recorrido completo por la terminología del paradigma orientado a objetos, centrándonos en el diseño orientado a objetos. Podemos clasificar diferentes objetos en una taxonomía de clases distintas y describir los atributos y comportamientos de dichos objetos mediante la interfaz de la clase.

El encapsulamiento y la ocultación de información son conceptos estrechamente vinculados. Los objetos pueden clasificarse y tienen responsabilidades claramente asignadas. El comportamiento de una aplicación en su conjunto surge de la colaboración entre objetos. La sintaxis de UML constituye un método de comunicación tanto útil como práctico.

En el próximo capítulo, exploraremos cómo implementar clases y métodos en Python.
