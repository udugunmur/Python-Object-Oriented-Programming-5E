# Parte 1: Fundamentos de la Programación Orientada a Objetos en Python
## Capítulo 2: Objetos en Python

¡Tenemos un diseño en nuestras manos y estamos listos para convertirlo en un programa funcional! Por supuesto, esto no suele ocurrir de forma tan lineal. Veremos ejemplos y sugerencias para un buen diseño de software a lo largo de todo el libro, pero nuestro enfoque principal es la programación orientada a objetos. Por lo tanto, echemos un vistazo a la sintaxis de Python que nos permite crear software orientado a objetos.

Tras completar este capítulo, comprenderemos lo siguiente:

- Las **pistas de tipo** (*type hints*) de Python.
- Creación de clases e instanciación de objetos en Python.
- Uso de técnicas de **composición** para crear objetos más complejos.
- Organización de clases en **paquetes** y **módulos**.
- Acceso adecuado a los miembros de la clase, incluyendo formas de sugerir que los objetos colaboradores no alteren el estado interno de un objeto.
- Trabajo con paquetes de terceros disponibles en el Índice de Paquetes de Python (**PyPI**).
- Gestión de **entornos virtuales**.

---

### Sección 2.1: Requisitos técnicos

El código de este capítulo se encuentra en el repositorio de PacktPublishing: [https://github.com/PacktPublishing/Python-Object-Oriented-Programming-5E](https://github.com/PacktPublishing/Python-Object-Oriented-Programming-5E). Dentro de los archivos de ese repositorio, nos centraremos en el directorio `ch_02`.

Este capítulo utilizará la herramienta `mypy`, que se instala por separado. Comandos como `python -m pip install mypy` la instalarán. Si estás utilizando `uv` para gestionar tu entorno, `uvx tool install mypy` añadirá `mypy`.

Todos los ejemplos fueron probados con Python 3.12 y 3.13. Se puede utilizar la herramienta `uv` para probar el código:

```bash
uvx tox
```

---

### Sección 2.2: Introducción a tipos y clases

Antes de que podamos examinar en detalle la creación de clases, debemos hablar un poco sobre qué es una clase y cómo asegurarnos de que la estamos utilizando correctamente. Una idea central es: **todo en Python es un objeto**.

Cuando escribimos valores literales como `"Hello, world!"` o `42`, en realidad estamos creando objetos que son instancias de clases integradas (algunos lenguajes tienen "tipos primitivos" que no son objetos; Python no tiene esta complicación). Podemos iniciar una sesión interactiva de Python y utilizar la función integrada `type()` sobre la clase que define las propiedades de estos objetos:

```python
>>> type("Hello, world!")
<class 'str'>
>>> type(42)
<class 'int'>
```

El propósito de la programación orientada a objetos es resolver un problema mediante la colaboración de objetos. Cuando escribimos `6 * 7`, la multiplicación de los dos objetos es manejada por un método de la clase integrada `int`. Para comportamientos más complejos, a menudo necesitaremos escribir clases nuevas y exclusivas.

Aquí están las dos primeras reglas fundamentales sobre cómo funcionan los objetos en Python:

1. **Todo en Python es un objeto.**
2. **Cada objeto se define por ser una instancia de al menos una clase.**

Estas reglas tienen muchas consecuencias interesantes. Una definición de clase que escribimos utilizando la instrucción `class` crea un nuevo objeto de tipo clase (`type`). Cuando creamos una instancia de una clase, el objeto de clase resultante se utilizará primero para crear y luego inicializar el objeto instancia que se está construyendo.

¿Cuál es la distinción entre clase y tipo? La instrucción `class` nos permite definir nuevos tipos (sí, así es como funciona). Dado que la instrucción `class` es la que utilizamos, las llamaremos **clases** a lo largo del texto. Consulta *Python objects, types, classes, and instances — a glossary* de Eli Bendersky ([https://eli.thegreenplace.net/2012/03/30/python-objects-types-classes-and-instances-a-glossary](https://eli.thegreenplace.net/2012/03/30/python-objects-types-classes-and-instances-a-glossary)) para esta útil cita:

> *"Los términos 'clase' y 'tipo' son un ejemplo de dos nombres que hacen referencia al mismo concepto."*

Para las pistas de tipo (*type hints*), existe un principio de uso común similar, aunque no tan claramente articulado. Los términos *pistas* (*hints*) y *anotaciones* (*annotations*) representan esencialmente el mismo concepto. Seguiremos el uso habitual y llamaremos a las anotaciones **pistas de tipo**.

Existe otra regla importante:

> **Una variable es una referencia a un objeto.** Piensa en una nota adhesiva amarilla, con un nombre garabateado en ella, pegada sobre una cosa.

Esto no parece demasiado sorprendente, pero en realidad es muy potente. Significa que la información de tipo —lo que es un objeto— está definida por la clase (o clases) asociada con el objeto. Esta información de tipo no está vinculada a la variable de ninguna manera. Esto provoca que un código como el siguiente sea a la vez válido y confuso en Python:

```python
>>> a_string_variable = "Hello, world!"
>>> type(a_string_variable)
<class 'str'>
>>> a_string_variable = 42
>>> type(a_string_variable)
<class 'int'>
```

Creamos un objeto utilizando una clase integrada, `str`. Asignamos un nombre largo, `a_string_variable`, al objeto. Luego, creamos un objeto utilizando una clase integrada diferente, `int`. Asignamos este nuevo objeto al nombre original.

La Figura 2.1 muestra dos pasos, uno al lado del otro, para ilustrar cómo la variable se mueve de un objeto a otro:

> **Figura 2.1: Nombres de variables y objetos**  
> *(La etiqueta `a_string_variable` apunta primero al objeto `str: "Hello, world!"` y luego se reasigna al objeto `int: 42`)*

Los distintos valores de las propiedades forman parte del objeto, no de la variable. Cuando comprobamos el tipo de una variable con `type()`, vemos el tipo del objeto al que la variable hace referencia actualmente. Una variable no tiene un tipo propio: no es más que un nombre. Esto significa que una función no se define por los tipos de sus parámetros o su tipo de retorno; es simplemente un nombre vinculado a un objeto "invocable" (*callable*). Los objetos invocables incluyen funciones y métodos de clases, junto con otros elementos que abordaremos en los Capítulos 8 y 11. De manera similar, consultar el `id()` de una variable muestra el identificador del objeto al que se refiere la variable. Claramente, el nombre `a_string_variable` resulta muy engañoso si le asignamos un objeto entero.

Además, herramientas como `mypy` y `pyright` tendrán problemas para determinar si la variable se está utilizando adecuadamente. La coherencia es una parte fundamental de la claridad.

Mostraremos pistas de tipo en la mayoría de los ejemplos. Dejaremos los detalles sobre las pistas y cómo verificarlas para el Capítulo 7.

Si nunca las has visto antes, así es como se ven las pistas de tipo cuando las escribimos en una función:

```python
def odd(n: int) -> bool:
    return n % 2 != 0
```

Las pistas sugieren que el valor del argumento para el parámetro `n` debe ser un entero. También sugieren que el resultado será uno de los dos valores del tipo `bool`.

La sintaxis de las anotaciones es relativamente fácil de entender: podemos seguir el nombre de una variable con dos puntos, `:`, y un tipo. Podemos hacer esto en los parámetros de funciones (y métodos), y también en las sentencias de asignación. Además, al definir funciones, podemos añadir `-> tipo` a la definición para explicar el tipo de retorno esperado.

Estas anotaciones **no tienen ningún impacto en tiempo de ejecución**. Dado que Python ignora educadamente las anotaciones, son opcionales. Sin embargo, las personas que lean tu código estarán encantadas de verlas: son una forma excelente de informar al lector sobre tu intención. Puedes omitirlas mientras estás aprendiendo, pero las agradecerás cuando vuelvas a ampliar algo que escribiste anteriormente.

Las herramientas de comprobación de pistas como `pyright` y `mypy` pueden analizar las pistas de tipo para localizar lugares donde no se utilizan correctamente. Esta es una parte esencial del ciclo de desarrollo, tan importante como escribir casos de prueba. Estas herramientas de verificación de tipos no vienen integradas en Python y requieren una descarga e instalación independiente. Los entornos de desarrollo integrado (IDE) más avanzados incluyen herramientas integradas de comprobación de tipos. Hablaremos sobre entornos virtuales e instalación de herramientas en la sección de Librerías de Terceros.

Por ahora, resulta útil familiarizarse con la sintaxis de las anotaciones. La mayoría de los ejemplos de este libro tendrán pistas de tipo porque ayudan a dejar clara la intención detrás del ejemplo. Ahora que hemos hablado sobre cómo se describen los parámetros y atributos con pistas de tipo, construyamos algunas clases reales.

---

### Sección 2.3: Creación de clases en Python

No hace falta escribir mucho código en Python para darse cuenta de que es un lenguaje muy limpio. Cuando queremos hacer algo, simplemente lo hacemos, sin necesidad de configurar una gran cantidad de código previo. El omnipresente *"Hola mundo"* en Python, como probablemente hayas visto, consta de una sola línea.

De forma similar, la clase más sencilla en Python 3 se ve así:

```python
class MyFirstClass:
    pass
```

¡Ahí está nuestro primer programa orientado a objetos! Para obtener más información sobre la sintaxis, consulta la sección 9.3.1 ([https://docs.python.org/3/tutorial/classes.html#class-definition-syntax](https://docs.python.org/3/tutorial/classes.html#class-definition-syntax)) del Tutorial de Python. El nombre de la clase debe seguir las reglas estándar de nomenclatura de variables en Python: debe comenzar con una letra o un guion bajo, y solo puede contener letras, guiones bajos o números. Además, la guía de estilo de Python **PEP 8** ([https://peps.python.org/pep-0008/](https://peps.python.org/pep-0008/)) recomienda que las clases se nombren utilizando la notación `CapWords` (o `CamelCase`): comenzar con una letra mayúscula y que las palabras posteriores comiencen también con mayúscula. Asimismo, de acuerdo con la guía de estilo, utiliza cuatro espacios para la indentación a menos que tengas una razón de peso para no hacerlo.

Dado que nuestra primera clase no añade datos ni comportamientos, utilizamos la instrucción `pass` en la segunda línea. Este es un marcador de posición para cumplir con el requisito de tener un cuerpo de clase; indica que no se debe realizar ninguna acción adicional.

Podríamos pensar que no hay mucho que hacer con esta clase tan básica, pero nos permite instanciar objetos de esa clase. Podemos cargar la clase en el intérprete interactivo de Python 3 para experimentar con ella. Para hacer esto, guarda la definición de clase anterior en un archivo con un nombre como `src/first_class.py` y luego ejecuta el siguiente comando:

```bash
% python -i src/first_class.py
```

El argumento `-i` le indica a Python que ejecute el código y luego pase al intérprete interactivo. La siguiente sesión del intérprete demuestra una interacción básica con esta clase:

```python
>>> a = MyFirstClass()
>>> b = MyFirstClass()
>>> print(a)
<first_class.MyFirstClass object at ...>
>>> print(b)
<first_class.MyFirstClass object at ...>
```

Este código instancia dos objetos a partir de la nueva clase, asignando los nombres de variable `a` y `b`. Crear una instancia de una clase consiste en escribir el nombre de la clase seguido de un par de paréntesis. Se parece mucho a una llamada a función: llamar a una clase creará un nuevo objeto. Al imprimirse, los dos objetos nos indican a qué clase pertenecen y en qué dirección de memoria se encuentran (hemos reemplazado las direcciones de memoria por `...` porque siempre son diferentes; habitualmente son números hexadecimales como `0xb7b7fbac`). Las direcciones de memoria no se utilizan con frecuencia en el código Python, pero aquí demuestran que hay dos objetos distintos involucrados en dos ubicaciones de memoria independientes.

Podemos comprobar que son objetos distintos utilizando el operador `is`:

```python
>>> a is b
False
```

Esto ayuda a disipar confusiones cuando hemos creado un conjunto de objetos y les hemos asignado diferentes nombres de variable.

#### 2.3.1 Añadir atributos

Ahora tenemos una clase básica, pero es bastante inútil: no contiene datos ni realiza ninguna acción. ¿Qué tenemos que hacer para asignar un atributo a un objeto determinado?

De hecho, no tenemos que hacer nada especial en la definición de la clase para poder añadir atributos. Podemos establecer atributos arbitrarios en un objeto instanciado utilizando la **notación de punto**. Aquí hay un ejemplo:

```python
class Point:
    pass

p1 = Point()
p2 = Point()

p1.x = 5
p1.y = 4

p2.x = 3
p2.y = 6

print(p1.x, p1.y)
print(p2.x, p2.y)
```

Si ejecutamos este código, las dos líneas de impresión al final nos muestran los nuevos valores de los atributos en los dos objetos:

```text
5 4
3 6
```

Este código creó una clase `Point` vacía sin datos ni comportamientos. Luego, creó dos instancias de esa clase y asignó a cada una de ellas coordenadas `x` e `y` para identificar un punto en dos dimensiones. Todo lo que necesitamos para asignar un valor a un atributo en un objeto es usar la sintaxis `<objeto>.<atributo> = <valor>`. Esto se conoce como notación de punto. El valor puede ser cualquier cosa: un tipo primitivo de Python, un tipo de datos integrado u otro objeto. ¡Incluso puede ser una función u otra clase!

Crear atributos de esta manera puede resultar confuso para quienes intentan leer tu código y también confunde a las herramientas utilizadas para inspeccionar el código. Existe un enfoque mucho mejor para los atributos (y sus pistas de tipo) que examinaremos en *Inicialización del objeto* más adelante en este capítulo. Antes, sin embargo, añadiremos comportamientos a nuestra definición de clase.

#### 2.3.2 Haciendo que haga algo

Tener objetos con atributos es un gran comienzo. La programación orientada a objetos trata sobre la interacción entre objetos: nos interesa invocar acciones que hagan que ocurran cosas con esos atributos. Ya tenemos datos; ahora es el momento de añadir **comportamientos** a nuestras clases.

Modelemos un par de acciones en nuestra clase `Point`. Podemos comenzar con un método llamado `reset`, que mueve el punto al origen (el origen es el lugar donde `x` e `y` son ambos cero). Esta es una buena acción introductoria porque no requiere ningún parámetro:

```python
class Point:
    def reset(self) -> None:
        self.x = 0
        self.y = 0

p = Point()
p.reset()
print(p.x, p.y)
```

Esta instrucción `print` nos muestra los dos valores cero de los atributos:

```text
0 0
```

En Python, un método tiene un formato idéntico al de una función. Para más información sobre la sintaxis, consulta la sección 9.3.4 ([https://docs.python.org/3/tutorial/classes.html#method-objects](https://docs.python.org/3/tutorial/classes.html#method-objects)) del Tutorial de Python.

El parámetro `self` es esencial. Analizaremos este parámetro `self` (a veces llamado la variable de instancia) a continuación.

Una función que no devuelve un valor explícitamente devolverá implícitamente el objeto `None`. Formalizamos esta característica de Python proporcionando `-> None` como parte de las anotaciones de tipo para el método.

##### 2.3.2.1 Hablando de uno mismo (`self`)

La única diferencia sintáctica entre los métodos de las clases y las funciones fuera de las clases es que los métodos tienen un argumento obligatorio. Por convención, este argumento siempre se denomina `self`. La convención es muy poderosa en la comunidad de Python: aunque técnicamente nada te impide llamarlo `this` o incluso `Martha`, es mejor seguir la guía comunitaria de PEP 8 y mantener siempre `self`.

El argumento `self` en un método es una **referencia al objeto sobre el cual se está invocando el método**. El objeto es una instancia de una clase, y por eso a esto se le llama a menudo la variable de instancia.

Podemos acceder a los atributos y métodos de ese objeto a través de esta variable. Esto es exactamente lo que hacemos dentro del método `reset` cuando establecemos los atributos `x` e `y` del objeto `self`.

Presta atención a la diferencia entre una clase y un objeto en esta discusión. Podemos pensar en el método como una función adjunta a una clase. El parámetro `self` se refiere a una instancia específica de la clase. Cuando llamas al método en dos objetos diferentes, estás pasando dos objetos diferentes como parámetro `self`.

Observa que cuando llamamos al método `p.reset()`, no pasamos explícitamente el argumento `self`. Python se encarga automáticamente de esta parte: sabe que estamos llamando a un método en el objeto `p`, por lo que pasa automáticamente ese objeto `p` al método de la clase `Point`.

A algunos les ayuda pensar en un método como una función que simplemente forma parte de una clase. En lugar de llamar al método en el objeto, podríamos invocar la función tal como está definida en la clase, pasando explícitamente nuestro objeto como argumento `self`:

```python
>>> p = Point()
>>> Point.reset(p)  # Works, but...
>>> print(p.x, p.y)
0 0
```

La salida es idéntica a la del ejemplo anterior porque internamente ha ocurrido exactamente el mismo proceso. Aunque esto funciona, no es la mejor práctica recomendada; lo enfatizamos aquí porque ayuda a consolidar la comprensión del argumento `self`.

¿Qué sucede si olvidamos incluir el argumento `self` en la definición de nuestra clase? Python fallará con un mensaje de error:

```python
>>> class Point:
...     def reset():
...         pass
... 
>>> p = Point()
>>> p.reset()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Point.reset() takes 0 positional arguments but 1 was given
```

El mensaje de error indica que se esperaba 0 argumentos posicionales pero se proporcionó 1. Recuerda que cuando veas un mensaje de error que indique argumentos no coincidentes en métodos, lo primero que debes verificar es si olvidaste el parámetro `self` en la definición del método.

##### 2.3.2.2 Más argumentos

¿Cómo pasamos múltiples argumentos a un método? Añadamos un nuevo método que nos permita mover un punto a una posición arbitraria, no solo al origen. También podemos incluir un método que acepte otro objeto `Point` como entrada y devuelva la distancia entre ellos:

```python
from __future__ import annotations
import math

class Point:
    def move(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def reset(self) -> None:
        self.move(0.0, 0.0)

    def calculate_distance(self, other: Point) -> float:
        return math.hypot(self.x - other.x, self.y - other.y)
```

Hemos definido una clase con dos atributos (`x` e `y`) y tres métodos independientes: `move()`, `reset()` y `calculate_distance()`.

El método `move()` acepta dos argumentos, `x` e `y`, y establece los atributos correspondientes del objeto `self`. El método `reset()` llama al método `move()`, ya que un reinicio es simplemente un movimiento a una ubicación conocida específica (`0.0, 0.0`).

El método `calculate_distance()` calcula la distancia euclidiana entre dos puntos utilizando la función `math.hypot()`.

El nombre de tipo `Point` para el parámetro `other` es una referencia a la definición de clase de la cual este método forma parte. La clase aún no está completamente definida dentro de su propio cuerpo. Antes de **PEP 749**, se podía usar una cadena `"Point"` para referirse a una clase no definida aún. Con la importación `from __future__ import annotations`, podemos incluir una referencia a la clase `Point` directamente. A partir de Python 3.14, la importación de `__future__` ya no será necesaria para que este tipo de referencias funcionen como se espera.

A continuación, se muestra un ejemplo del uso de esta definición de clase:

```python
>>> point1 = Point()
>>> point2 = Point()
>>> point1.reset()
>>> point2.move(5, 0)
>>> print(point2.calculate_distance(point1))
5.0
>>> assert point2.calculate_distance(point1) == point1.calculate_distance(
...     point2
... )
>>> point1.move(3, 4)
>>> print(point1.calculate_distance(point2))
4.47213595499958
>>> print(point1.calculate_distance(point1))
0.0
```

La instrucción `assert` es una maravillosa herramienta de prueba: el programa se detendrá si la expresión después de `assert` se evalúa como `False` (o cero, vacío o `None`). En este caso, la usamos para asegurar que la distancia sea la misma independientemente de qué punto llame al método `calculate_distance()` del otro punto. Veremos mucho más sobre `assert` en el Capítulo 13.

#### 2.3.3 Inicialización del objeto

Si no establecemos explícitamente las posiciones `x` e `y` en nuestro objeto `Point`, ya sea usando `move` o accediendo a ellos directamente, tendremos un objeto `Point` defectuoso sin una posición real. ¿Qué ocurrirá cuando intentemos acceder a él?

Probémoslo en el intérprete interactivo:

```python
>>> point = Point()
>>> point.x = 5
>>> print(point.x)
5
>>> print(point.y)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Point' object has no attribute 'y'
```

Al menos lanzó una excepción útil. Cubriremos las excepciones en detalle en el Capítulo 4.

El resultado nos indica que se produjo un error `AttributeError` con un mensaje descriptivo. Podemos capturar y recuperarnos de este error, pero en este caso resulta evidente que deberíamos haber especificado algún tipo de valor predeterminado o haber obligado al usuario a suministrar las coordenadas al instanciar el objeto.

Curiosamente, las herramientas de comprobación de anotaciones como `mypy` no pueden determinar automáticamente si `y` debía ser un atributo de un objeto `Point`, ya que los atributos en Python son dinámicos. Sin embargo, Python cuenta con convenciones ampliamente seguidas para estructurarlos.

La mayoría de los lenguajes de programación orientados a objetos tienen el concepto de **constructor**, un método especial que crea e inicializa el objeto cuando se instancia. Python tiene un **inicializador**. El método constructor `__new__()` rara vez se utiliza a menos que se esté realizando algo muy especializado (veremos elementos de este diseño de bajo nivel en el Capítulo 6). Nos centraremos en el método de inicialización habitual: `__init__()`. A menudo se le llama constructor de la instancia porque construye el estado inicial del objeto.

El método de inicialización de Python es igual a cualquier otro método, excepto que tiene un nombre especial: `__init__`. Los guiones bajos dobles iniciales y finales (*dunder*) significan que este es un método especial que el intérprete de Python tratará de forma particular.

> [!WARNING]
> **Nunca nombres un método propio con guiones bajos dobles al inicio y al final.** Puede no significar nada para Python hoy, pero siempre existe la posibilidad de que los diseñadores de Python agreguen una función especial con ese nombre en el futuro, rompiendo tu código.

Añadamos una función de inicialización en nuestra clase `Point`:

```python
from __future__ import annotations
import math

class Point:
    def __init__(self, x: float, y: float) -> None:
        self.move(x, y)

    def move(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def reset(self) -> None:
        self.move(0, 0)

    def calculate_distance(self, other: Point) -> float:
        return math.hypot(self.x - other.x, self.y - other.y)
```

La instanciación de un `Point` ahora se realiza así:

```python
>>> point = Point(3, 5)
>>> print(point.x, point.y)
3 5
```

¡Ahora nuestro objeto `Point` nunca carecerá de coordenadas `x` e `y`! Si intentamos construir una instancia de `Point` sin incluir los parámetros de inicialización adecuados, fallará con una excepción `TypeError`:

```python
>>> point = Point()
Traceback (most recent call last):
  ...
TypeError: Point.__init__() missing 2 required positional arguments: 'x' and 'y'
```

La mayor parte del tiempo colocamos nuestras sentencias de inicialización en la función `__init__()`. Es muy importante asegurarse de que todos los atributos se inicialicen dentro del método `__init__()`. Hacer esto ayuda a las herramientas de verificación como `mypy` al proporcionar todos los atributos en un lugar centralizado, y facilita la lectura del código a otros desarrolladores.

#### 2.3.4 Pistas de tipo y valores predeterminados

Si no queremos que estos dos argumentos sean estrictamente obligatorios, utilizamos la sintaxis estándar de Python para proporcionar **valores de argumento predeterminados**:

```python
class Point:
    def __init__(self, x: float = 0.0, y: float = 0.0) -> None:
        self.move(x, y)
```

Cuando hay más de dos o tres parámetros, las líneas de código pueden volverse muy largas. Podemos formatear la firma en múltiples líneas físicas aprovechando los paréntesis:

```python
class Point:
    def __init__(
        self, x: float = 0.0, y: float = 0.0
    ) -> None:
        self.move(x, y)
```

Este estilo mantiene las líneas más cortas y fáciles de leer.

#### 2.3.5 Explicándose con docstrings

Python es un lenguaje muy legible; algunos dirían que es autodocumentado. Sin embargo, al programar orientado a objetos, es importante redactar documentación de API que resuma claramente lo que hace cada objeto y método.

Python soporta esto mediante el uso de **docstrings**. Cada cabecera de clase, función o método puede tener una cadena de texto de Python como la primera línea del bloque indentado.

A menudo, las docstrings son extensas y abarcan múltiples líneas (la guía de estilo sugiere que la longitud de la línea no supere los 80 caracteres), lo que hace recomendable el uso de comillas triples (`"""` o `'''`).

Una docstring debe resumir el propósito de la clase o método que describe, explicar cualquier parámetro cuyo uso no sea evidente e incluir ejemplos breves de uso. Una de las mejores cosas que se pueden incluir en una docstring es un ejemplo concreto: herramientas como `doctest` pueden verificar automáticamente que estos ejemplos sean correctos. Todos los ejemplos de este libro se validan con la herramienta `doctest`.

A continuación se muestra nuestra clase `Point` completamente documentada dividida en dos partes:

```python
class Point:
    """
    Represents a point in two-dimensional geometric coordinates
    >>> p_0 = Point()
    >>> p_1 = Point(3, 4)
    >>> p_0.calculate_distance(p_1)
    5.0
    """

    def __init__(self, x: float = 0.0, y: float = 0.0) -> None:
        """
        Initialize the position of a new point. The x and y
        coordinates can be specified. If they are not, the
        point defaults to the origin.

        :param x: float x-coordinate
        :param y: float x-coordinate
        """
        self.move(x, y)
```

Y aquí está el resto de la definición:

```python
    def move(self, x: float, y: float) -> None:
        """
        Move the point to a new location in 2D space.

        :param x: float x-coordinate
        :param y: float x-coordinate
        """
        self.x = x
        self.y = y

    def reset(self) -> None:
        """
        Reset the point back to the geometric origin: 0, 0
        """
        self.move(0.0, 0.0)

    def calculate_distance(self, other: Point) -> float:
        """
        Calculate the Euclidean distance from this point
        to a second point passed as a parameter.

        :param other: Point instance
        :return: float distance
        """
        return math.hypot(self.x - other.x, self.y - other.y)
```

Si cargamos este archivo en el intérprete (`python -i src/point_4.py`) e introducimos `help(Point)`, veremos la documentación formateada:

```text
>>> help(Point)
Help on class Point in module point_4:

class Point(builtins.object)
 |  Point(x: 'float' = 0.0, y: 'float' = 0.0) -> 'None'
 |  
 |  Represents a point in two-dimensional geometric coordinates
 |  
 |  >>> p_0 = Point()
 |  >>> p_1 = Point(3, 4)
 |  >>> p_0.calculate_distance(p_1)
 |  5.0
 |  
 |  Methods defined here:
 |  
 |  __init__(self, x: 'float' = 0.0, y: 'float' = 0.0) -> 'None'
 |      Initialize the position of a new point. The x and y
 |      coordinates can be specified. If they are not, the
 |      point defaults to the origin.
 |      
 |      :param x: float x-coordinate
 |      :param y: float x-coordinate
 |  
 |  calculate_distance(self, other: 'Point') -> 'float'
 |      Calculate the Euclidean distance from this point
 |      to a second point passed as a parameter.
 |      
 |      :param other: Point instance
 |      :return: float distance
 |  
 |  move(self, x: 'float', y: 'float') -> 'None'
 |      Move the point to a new location in 2D space.
 |      
 |      :param x: float x-coordinate
 |      :param y: float x-coordinate
 |  
 |  reset(self) -> 'None'
 |      Reset the point back to the geometric origin: 0, 0
 |  
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables
 |  
 |  __weakref__
 |      list of weak references to the object
```

Podemos ejecutar el siguiente comando para verificar el ejemplo mostrado en la docstring con `doctest`:

```bash
% python -m doctest src/point_4.py
```

Si todo funciona, no habrá salida (por defecto, `doctest` solo produce salida cuando una prueba falla; para ver una salida detallada, añade la opción `-v`).

Además, podemos ejecutar `mypy` para comprobar las pistas de tipo:

```bash
% mypy src
```

*(Nota: `%` representa el prompt de la terminal del sistema operativo, no del REPL de Python).*

Si utilizas `uv`, puedes ejecutar `uvx tool run mypy src`. Cuando no hay errores, la salida es concisa:

```text
% mypy src
Success: no issues found in 22 source files
```

---

### Sección 2.4: Composición y descomposición

Para ver la **composición** en acción, examinaremos algunos elementos aislados del diseño de una partida de ajedrez.

Una partida de ajedrez se juega entre dos jugadores sobre un tablero que contiene 64 posiciones en una cuadrícula. El tablero cuenta con dos conjuntos de 16 piezas que los jugadores mueven en turnos alternos. Cada pieza puede capturar otras piezas. El tablero debe dibujarse en la pantalla tras cada turno.

El juego de ajedrez está compuesto por un tablero y 32 piezas. El tablero comprende además 64 posiciones, comúnmente identificadas por columna (*file*, a-h) y fila (*rank*, 1-8).

La Figura 2.2 muestra un diagrama de clases donde la clase `Board` es una composición de instancias de `Position`:

> **Figura 2.2: Diagrama de clases para un tablero de ajedrez**  
> *(Board ◆── 64 Position)*

Las definiciones en Python para estas clases pueden comenzar así:

```python
class Position:
    def __init__(self, file: str, rank: str) -> None:
        self.file = file
        self.rank = rank

class Board:
    def __init__(self) -> None:
        self.positions: dict[tuple[str, str], Position] = {}
        for file in ('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'):
            for rank in ('1', '2', '3', '4', '5', '6', '7', '8'):
                self.positions[file, rank] = Position(file, rank)
```

El método `__init__()` crea los 64 objetos `Position` y los asigna al objeto `Board`.

Observamos dos niveles de detalle:
1. **Detalles de implementación:** involucran `dict`, `tuple` y `str`.
2. **Dominio del problema:** que es un `Board` compuesto por instancias de `Position`.

¿Qué sucede cuando el objeto `Board` ya no se utiliza? Todos los 64 objetos `Position` asociados dejan de ser necesarios. Esta es la regla de *"un compuesto controla la composición"*.

Existe una alternativa a la composición denominada **agregación** (*aggregation*).

La Figura 2.3 ilustra una versión ampliada que incluye las piezas:

> **Figura 2.3: Diagrama de clases para tablero y piezas**  
> *(Board ◆── 64 Position ◇── 0..1 Piece)*

El rombo abierto indica una **agregación**: `Piece` y `Position` pueden existir independientemente el uno del otro. Si retiramos una pieza del juego (al ser capturada), la posición continúa existiendo en el tablero.

En la práctica, en Python tanto la composición como la agregación se implementan de forma similar, ya que la recolección de basura y eliminación de objetos se realiza automáticamente.

---

### Sección 2.5: ¿Quién puede acceder a mis datos?

Los lenguajes orientados a objetos suelen tener conceptos estrictos de control de acceso (`private`, `protected`, `public`, `final`).

Python no funciona así: mantiene las cosas simples bajo la premisa de **"aquí todos somos adultos"** (*we're all adults here*). Todos los métodos y atributos de una clase son públicamente accesibles.

- **Atributos no públicos (`_nombre`):** Por convención, prefijamos un atributo o método con un único guion bajo `_` para indicar que es un detalle de implementación interno y que los desarrolladores deben pensarlo tres veces antes de acceder a él directamente. Las herramientas de análisis estático (*linters*) advertirán si se accede a ellos desde fuera.
- **Name Mangling (`__nombre`):** Si prefijamos un identificador con dos guiones bajos `__`, Python aplicará una transformación de nombres (*name mangling*), renombrándolo internamente como `_<NombreClase>__nombre`. Esto no garantiza privacidad absoluta, solo añade dificultad al acceso externo y rara vez se recomienda su uso en código habitual.

> [!TIP]
> - No utilices nombres con doble guion bajo (`__`) para simular privacidad estricta.
> - Utiliza nombres con un único guion bajo (`_`) para marcar detalles de implementación sujetos a cambios.

---

### Sección 2.6: Módulos y paquetes

Los **módulos** en Python son simplemente archivos `.py`. El nombre del módulo es el nombre del archivo sin la extensión `.py`.

La instrucción `import` permite importar módulos completos o clases y funciones específicas:

```python
import random

def dice1() -> tuple[int, int]:
    return (
        random.randint(1, 6),
        random.randint(1, 6)
    )
```

O importar solo la función requerida:

```python
from random import randint

def dice2() -> tuple[int, int]:
    return (
        randint(1, 6),
        randint(1, 6)
    )
```

O renombrar el objeto importado:

```python
from random import randint as rng

def dice3() -> tuple[int, int]:
    return (
        rng(1, 6),
        rng(1, 6)
    )
```

Importar múltiples elementos:

```python
from random import seed, Random
```

> [!CAUTION]
> **Evita el uso de `from random import *`:**
> - Oscurece de dónde proviene cada función en archivos grandes.
> - Puede sobrescribir nombres silenciosamente si existen colisiones entre módulos.
> - Afecta negativamente al autocompletado y análisis de los editores de código.
> - Como dice el *Zen de Python* (`import this`): *"Explicit is better than implicit"*.

#### 2.6.1 Organización de módulos en paquetes

Un **paquete** es un directorio que contiene módulos. Para que Python reconozca un directorio como paquete, debe contener un archivo `__init__.py` (generalmente vacío).

Ejemplo de estructura en `src/`:

```text
src/
+-- main.py
+-- ecommerce/
    +-- __init__.py
    +-- database.py
    +-- products.py
    +-- vendors.py
    +-- payments/
    |   +-- __init__.py
    |   +-- common.py
    |   +-- square.py
    |   +-- stripe.py
    +-- contact/
        +-- __init__.py
        +-- email.py
```

##### 2.6.1.1 Importaciones absolutas

Especifican la ruta completa desde la raíz del paquete:

```python
import ecommerce.products
product_1 = ecommerce.products.Product("fore")
```

```python
from ecommerce.products import Product
product_2 = Product("main")
```

```python
from ecommerce import products
product_3 = products.Product("mizzen")
```

##### 2.6.1.2 Importaciones relativas

Utilizan puntos para referenciar módulos relativos a la ubicación actual:

```python
from .database import Database
```

Subir un nivel en la jerarquía:

```python
from ..database import Database
```

Navegar hacia otro subpaquete:

```python
from ..contact.email import send_mail
```

##### 2.6.1.3 Paquetes como un todo

Podemos exponer variables o clases directamente desde el paquete definiéndolas en `__init__.py`:

En `ecommerce/__init__.py`:
```python
from .database import db
```

En `main.py`:
```python
from ecommerce import db
```

#### 2.6.2 Organización del código en módulos

Podemos definir variables globales a nivel de módulo o utilizar funciones de inicialización:

```python
class Database:
    """The Database Implementation"""
    def __init__(self, connection: str | None = None) -> None:
        """Create a connection to a database."""
        pass

database = Database("file:/path/to/database")
```

Para retrasar la inicialización hasta que sea necesaria:

```python
db: Database | None = None

def initialize_database(connection: str | None = None) -> None:
    global db
    if not db:
        db = Database(connection)
```

O usando una función de acceso:

```python
def get_database(connection: str | None = None) -> Database:
    global db
    if not db:
        db = Database(connection)
    return db
```

##### Patrón de ejecución de scripts con `main()`

Para evitar que el código se ejecute cuando el archivo es importado por otro módulo, utilizamos la guarda `if __name__ == "__main__":`:

```python
def main() -> None:
    """
    Does the useful work.
    >>> main()
    p1.calculate_distance(p2)=5.0
    """
    p1 = Point()
    p2 = Point(3, 4)
    print(f"{p1.calculate_distance(p2)=}")

if __name__ == "__main__":
    main()
```

##### Clases y funciones anidadas

Las clases y funciones pueden definirse localmente dentro de otras funciones:

```python
class Formatter:
    def format(self, string: str) -> str:
        return string

def format_string(string: str, formatter: Formatter | None = None) -> str:
    """
    Format a string using the formatter object, which is expected to have
    a format() method that accepts a string.
    """
    class DefaultFormatter(Formatter):
        """Format a string in title case."""
        def format(self, string: str) -> str:
            return str(string).title()

    if not formatter:
        formatter = DefaultFormatter()
    return formatter.format(string)
```

Ejecución interactiva:

```python
>>> hello_string = "hello world, how are you today?"
>>> print(f" input: {hello_string}")
 input: hello world, how are you today?
>>> print(f"output: {format_string(hello_string)}")
output: Hello World, How Are You Today?
```

---

### Sección 2.7: Librerías de terceros y entornos virtuales

Python incluye una amplia librería estándar ("*batteries included*"). Para instalar paquetes de terceros desde **PyPI** ([https://pypi.python.org/](https://pypi.python.org/)), utilizamos `pip` o gestores modernos como `uv`.

#### Creación de un entorno virtual con `venv`:

En Linux / macOS:
```bash
% cd project_directory
% python -m venv env
% source env/bin/activate
```

En Windows:
```cmd
> cd project_directory
> python -m venv env
> env\Scripts\activate
```

Una vez activado el entorno, la instalación de paquetes con `python -m pip install <paquete>` se aísla exclusivamente a dicho proyecto.

---

### Sección 2.8: Gestión de entornos virtuales

Existen herramientas avanzadas para la gestión de proyectos y dependencias:

- **uv:** Herramienta ultrarrápida para gestión de paquetes y entornos ([https://docs.astral.sh/uv/](https://docs.astral.sh/uv/)).
- **Poetry:** Gestor integral de empaquetado y dependencias ([https://python-poetry.org](https://python-poetry.org/)).
- **Conda:** Utilizado en ciencia de datos para paquetes científicos complejos ([https://docs.conda.io/](https://docs.conda.io/)).

Con `uv`:
```bash
% uv add pydantic
% uv sync
```

---

### Sección 2.9: Repaso

Puntos clave tratados en este capítulo:

- Las pistas de tipo (*type hints*) describen parámetros y tipos de retorno para documentación y análisis estático.
- Las clases se definen con `class` y sus atributos se inicializan en `__init__()`.
- Los módulos y paquetes organizan el código a mayor escala.
- En Python no existen datos privados estrictos; se utilizan convenciones como el guion bajo inicial (`_`).
- La composición permite construir clases complejas a partir de componentes más simples.
- Los entornos virtuales aíslan las dependencias de terceros de cada proyecto.

---

### Sección 2.10: Ejercicios

1. **Refactoriza un proyecto existente:** Toma un script o proyecto previo y organiza sus datos y funciones en clases con métodos claros y atributos inicializados en `__init__()`. Divide el código en módulos y paquetes.
2. **Diseño Top-Down:** Modela una aplicación para gestionar una lista de tareas pendientes (*To-Do List*). Define clases como `Task` e implementa estados (`incomplete`, `started`, `completed`).
3. **Refactoriza el script del Capítulo 1:** Diseña clases para modelar las aplicaciones, entornos y comandos que procesaba el script de análisis JSON.
4. **Modela una baraja de cartas:** Crea clases para cartas (`Card`), manos de cartas (`Hand`) y juegos de mesa (por ejemplo, *Cribbage* o *Blackjack*). Experimenta con importaciones relativas y absolutas entre módulos.

---

### Sección 2.11: Resumen

En este capítulo aprendimos a crear clases y asignarles propiedades y métodos en Python. Comprendimos la diferencia entre constructor e inicializador, el enfoque flexible de control de acceso, los niveles de alcance con módulos y paquetes, y cómo gestionar librerías de terceros con entornos virtuales.

En el próximo capítulo, exploraremos cómo compartir implementaciones entre clases mediante la **herencia**.
