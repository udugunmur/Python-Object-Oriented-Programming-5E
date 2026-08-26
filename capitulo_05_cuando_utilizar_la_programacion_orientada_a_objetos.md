# Parte 1: Fundamentos de la Programación Orientada a Objetos en Python
## Capítulo 5: Cuándo Utilizar la Programación Orientada a Objetos

En los capítulos anteriores, hemos cubierto muchas de las características definitorias de la programación orientada a objetos. Ahora conocemos varios principios y paradigmas del diseño orientado a objetos y hemos revisado la sintaxis de la POO en Python.

Sin embargo, aún no sabemos exactamente cómo y, sobre todo, cuándo utilizar estos principios y sintaxis en la práctica. En este capítulo, analizaremos aplicaciones útiles del conocimiento adquirido, explorando nuevos conceptos en el camino:

- Cómo reconocer objetos.
- Datos y comportamientos, una vez más.
- Encapsulación de comportamientos sobre datos utilizando **propiedades** (*properties*).
- El principio **DRY** (*Don't Repeat Yourself* - No te repitas) y cómo evitar la duplicación.

Comenzaremos este capítulo con un examen detallado de la naturaleza de los objetos y su estado interno. Existen casos en los que no hay cambio de estado y definir una clase no resulta conveniente.

---

### Sección 5.1: Tratar a los objetos como objetos

Esto puede parecer obvio: por lo general, se debe asignar una clase específica a objetos diferenciados en el dominio del problema. Hemos visto ejemplos de esto en capítulos previos: primero identificamos los objetos en el problema y luego modelamos sus datos y comportamientos.

Identificar objetos es una tarea fundamental en el análisis y programación orientados a objetos. Sin embargo, no siempre es tan sencillo como contar los sustantivos en párrafos breves preparados explícitamente para ese fin. Recuerda: **los objetos son entidades que poseen tanto datos como comportamiento**. Si solo estamos trabajando con datos, a menudo es mejor almacenarlos en una lista, conjunto, diccionario u otra estructura de datos de Python (las cuales cubriremos a fondo en el Capítulo 8). Por otro lado, si solo trabajamos con comportamiento, pero sin almacenar estado, una función simple resulta mucho más adecuada.

Un objeto, en cambio, tiene tanto datos como comportamiento. Los programadores competentes de Python utilizan estructuras de datos integradas a menos que (o hasta que) exista una necesidad evidente de definir una clase. No hay razón para añadir una capa adicional de complejidad si no ayuda a organizar nuestro código. Por otro lado, la necesidad no siempre es evidente a primera vista.

A menudo podemos comenzar nuestros programas en Python almacenando datos en unas pocas variables. A medida que el programa crece, descubrimos que pasamos el mismo conjunto de variables relacionadas a un grupo de funciones. Ese es el momento de considerar agrupar tanto las variables como las funciones en una clase.

Por ejemplo, si estamos diseñando un programa para modelar polígonos en un espacio bidimensional, podríamos empezar representando cada polígono como una lista de puntos. Los puntos se modelarían como tuplas de dos elementos `(x, y)` que describen la ubicación del punto. Todo esto son datos almacenados en estructuras anidadas (específicamente, una lista de tuplas). A menudo comenzamos experimentando en la consola interactiva:

```python
>>> square = [(1,1), (1,2), (2,2), (2,1)]
```

Ahora, si queremos calcular el perímetro del polígono, debemos sumar las distancias entre cada par de puntos consecutivos. Para hacer esto, necesitamos una función que calcule la distancia entre dos puntos:

```python
>>> from math import hypot
>>> def distance(p_1, p_2):
...     return hypot(p_1[0] - p_2[0], p_1[1] - p_2[1])
... 
>>> def perimeter(polygon):
...     pairs = zip(polygon, polygon[1:] + polygon[:1])
...     return sum(
...         distance(p1, p2) for p1, p2 in pairs
...     )
... 
```

Podemos ejecutar las funciones para verificar nuestro trabajo:

```python
>>> perimeter(square)
4.0
```

Esto es un buen comienzo, pero no describe completamente el dominio del problema. Leyendo el código, requerimos cierto esfuerzo para deducir qué tipo de dato debe tener `polygon`.

Podemos agregar pistas de tipo (*type hints*) para clarificar la intención:

```python
from math import hypot

type Point = tuple[float, float]

def distance(p_1: Point, p_2: Point) -> float:
    return hypot(p_1[0] - p_2[0], p_1[1] - p_2[1])

type Polygon = list[Point]

def perimeter(polygon: Polygon) -> float:
    pairs = zip(polygon, polygon[1:] + polygon[:1])
    return sum(distance(p1, p2) for p1, p2 in pairs)
```

Hemos añadido dos definiciones de tipo, `Point` y `Polygon`. La definición de `Point` muestra cómo utilizamos la tupla integrada para contener dos valores flotantes. La definición de `Polygon` muestra cómo la lista integrada se basa en `Point`.

Como programadores orientados a objetos, reconocemos claramente que una clase polígono podría encapsular la lista de puntos (datos) y la función de perímetro (comportamiento). Además, una clase `Point` podría encapsular las coordenadas `x` e `y` y el método de distancia. La pregunta es: **¿vale la pena hacerlo?**

Comparemos la versión orientada a objetos:

```python
from math import hypot

class Point:
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def distance(self, other: "Point") -> float:
        return hypot(self.x - other.x, self.y - other.y)

class Polygon:
    def __init__(self) -> None:
        self.vertices: list[Point] = []

    def add_point(self, point: Point) -> None:
        self.vertices.append(point)

    def perimeter(self) -> float:
        pairs = zip(self.vertices, self.vertices[1:] + self.vertices[:1])
        return sum(p1.distance(p2) for p1, p2 in pairs)
```

Hay casi el doble de código aquí que en la versión funcional. Comparemos ambas APIs en uso:

```python
>>> square = Polygon()
>>> square.add_point(Point(1,1))
>>> square.add_point(Point(1,2))
>>> square.add_point(Point(2,2))
>>> square.add_point(Point(2,1))
>>> square.perimeter()
4.0
```

Frente al enfoque basado en funciones:

```python
>>> square = [(1,1), (1,2), (2,2), (2,1)]
>>> perimeter(square)
4.0
```

La longitud del código no es un indicador directo de su complejidad. Algunos programadores se obsesionan con escribir soluciones complejas en una sola línea (*code golf*), pero el resultado suele ser difícil de mantener.

Podemos hacer que la API orientada a objetos de `Polygon` sea tan cómoda y concisa como la funcional permitiendo que acepte una secuencia de puntos en su inicializador:

```python
from collections.abc import Iterable

class Polygon_2:
    def __init__(self, vertices: Iterable[Point] | None = None) -> None:
        self.vertices = list(vertices) if vertices else []

    def perimeter(self) -> float:
        pairs = zip(self.vertices, self.vertices[1:] + self.vertices[:1])
        return sum(p1.distance(p2) for p1, p2 in pairs)
```

Uso:

```python
>>> square = Polygon_2(
...     [Point(1,1), Point(1,2), Point(2,2), Point(2,1)]
... )
>>> square.perimeter()
4.0
```

Podemos ir un paso más allá y permitir que acepte tanto tuplas como instancias de `Point` utilizando coincidencia de patrones estructurales (*structural pattern matching* con `match`):

```python
type Pair = tuple[float, float]
type Point_or_Tuple = Point | Pair

class Polygon_3:
    def __init__(
        self, vertices: Iterable[Point_or_Tuple] | None = None
    ) -> None:
        self.vertices: list[Point] = []
        for pt_tup in vertices or []:
            self.vertices.append(
                self.make_point(pt_tup)
            )

    @staticmethod
    def make_point(item: Point_or_Tuple) -> Point:
        match item:
            case Point() as pt:
                return pt
            case (float() | int(), float() | int()) as tup:
                return Point(*tup)
            case _:
                raise TypeError(
                    f"unexpected {type(item)}: {item!r}"
                )
```

Cuanto más relevante es un conjunto de datos, más probable es que requiera múltiples funciones asociadas (`area()`, `point_in_polygon()`, transformaciones de escala, rotación, color, textura), y más sentido tiene encapsularlos en una clase.

---

### Sección 5.2: Añadir comportamientos a los datos de la clase con propiedades

En algunos lenguajes se aconseja no acceder nunca a los atributos directamente, recurriendo obligatoriamente a métodos *getter* y *setter*:

```python
class Color:
    def __init__(self, rgb_value: int, name: str) -> None:
        self._rgb_value = rgb_value
        self._name = name

    def set_name(self, name: str) -> None:
        self._name = name

    def get_name(self) -> str:
        return self._name

    def set_rgb_value(self, rgb_value: int) -> None:
        self._rgb_value = rgb_value

    def get_rgb_value(self) -> int:
        return self._rgb_value
```

Este estilo resulta engorroso y poco pythónico:

```python
>>> c = Color(0xff0000, "bright red")
>>> c.get_name()
'bright red'
>>> c.set_name("red")
>>> c.get_name()
'red'
```

En Python preferimos el acceso directo a los atributos:

```python
class Color_Py:
    def __init__(self, rgb_value: int, name: str) -> None:
        self.rgb_value = rgb_value
        self.name = name
```

```python
>>> c = Color_Py(0xff0000, "bright red")
>>> c.name
'bright red'
>>> c.name = "red"
>>> c.name
'red'
```

¿Qué ocurre si más adelante necesitamos añadir validación al asignar un valor?

```python
class Color_V:
    def __init__(self, rgb_value: int, name: str) -> None:
        self._rgb_value = rgb_value
        if not name:
            raise ValueError(f"Invalid name {name!r}")
        self._name = name

    def set_name(self, name: str) -> None:
        if not name:
            raise ValueError(f"Invalid name {name!r}")
        self._name = name
    # etc.
```

Si hubiésemos forzado a los usuarios a cambiar de `c.name = "red"` a `c.set_name("red")`, romperíamos el código existente.

Python resuelve esto elegantemente con la función **`property()`**, permitiendo que métodos actúen sintácticamente como atributos:

```python
class Color_VP:
    def __init__(self, rgb_value: int, name: str) -> None:
        self._rgb_value = rgb_value
        if not name:
            raise ValueError(f"Invalid name {name!r}")
        self._name = name

    def _set_name(self, name: str) -> None:
        if not name:
            raise ValueError(f"Invalid name {name!r}")
        self._name = name

    def _get_name(self) -> str:
        return self._name

    name = property(_get_name, _set_name)
```

Uso idéntico pero con validación en tiempo de ejecución:

```python
>>> c = Color_VP(0x0000ff, "bright red")
>>> c.name
'bright red'
>>> c.name = "red"
>>> c.name
'red'
>>> c.name = ""
Traceback (most recent call last):
  ...
  File "src/colors.py", line 85, in _set_name
    raise ValueError(f"Invalid name {name!r}")
ValueError: Invalid name ''
```

#### 5.2.1 Propiedades en detalle

La función `property(fget=None, fset=None, fdel=None, doc=None)` acepta opcionalmente métodos de borrado (`del`) y cadenas de documentación:

```python
class NorwegianBlue:
    def __init__(self, name: str) -> None:
        self._name = name
        self._state: str

    def _get_state(self) -> str:
        print(f"Getting {self._name}'s State")
        return self._state

    def _set_state(self, state: str) -> None:
        print(f"Setting {self._name}'s State to {state!r}")
        self._state = state

    def _del_state(self) -> None:
        print(f"{self._name} is pushing up daisies!")
        del self._state

    silly = property(_get_state, _set_state, _del_state, "This is a silly property")
```

Demostración interactiva:

```python
>>> p = NorwegianBlue("Polly")
>>> p.silly = "Pining for the fjords"
Setting Polly's State to 'Pining for the fjords'
>>> p.silly
Getting Polly's State
'Pining for the fjords'
>>> del p.silly
Polly is pushing up daisies!
```

Inspección de documentación (`help(NorwegianBlue)`):

```text
class NorwegianBlue(builtins.object)
 |  NorwegianBlue(name: str) -> None
 |  
 |  Methods defined here:
 |  
 |  __init__(self, name: str) -> None
 |      Initialize self.  See help(type(self)) for accurate signature.
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables
 |  
 |  __weakref__
 |      list of weak references to the object
 |  
 |  silly
 |      This is a silly property
```

#### 5.2.2 Decoradores: otra forma de crear propiedades

La forma más habitual y legible de definir propiedades en Python es mediante la sintaxis de **decoradores**:

```python
class NorwegianBlue_P:
    def __init__(self, name: str) -> None:
        self._name = name
        self._state: str

    @property
    def silly(self) -> str:
        """This is a silly property"""
        print(f"Getting {self._name}'s State")
        return self._state

    @silly.setter
    def silly(self, state: str) -> None:
        print(f"Setting {self._name}'s State to {state!r}")
        self._state = state

    @silly.deleter
    def silly(self) -> None:
        print(f"{self._name} is pushing up daisies!")
        del self._state
```

#### 5.2.3 Cuándo utilizar propiedades

Pautas recomendadas para elegir entre atributos, métodos y propiedades:

- **Métodos:** Representan **acciones** (verbos) que realizan tareas o transforman el estado.
- **Atributos estándar:** Representan **estado** directo, inicializados en `__init__()`.
- **Propiedades (`@property`):** Se utilizan cuando acceder o modificar un dato requiere computación auxiliar:
  - Validación de datos entrantes.
  - Valores derivados o calculados al vuelo a partir de otros atributos.
  - Registro en auditorías o logs.
  - Evaluación perezosa (*lazy evaluation*) y almacenamiento en caché (*caching*).

##### Ejemplo práctico: Almacenamiento en caché (*caching*) de una página web

```python
from urllib.request import urlopen

class WebPage:
    def __init__(self, url: str) -> None:
        self.url = url
        self._content: bytes | None = None

    @property
    def content(self) -> bytes:
        if self._content is None:
            print("Retrieving New Page...")
            with urlopen(self.url) as response:
                self._content = response.read()
        return self._content or b''
```

Medición del rendimiento:

```python
import time

webpage = WebPage("http://ccphillips.net/")

now = time.perf_counter()
content1 = webpage.content
first_fetch = time.perf_counter() - now

now = time.perf_counter()
content2 = webpage.content
second_fetch = time.perf_counter() - now

assert content2 == content1, "Problem: Pages were different"
print(f"Initial Request {first_fetch:.6f}")
print(f"Subsequent Requests {second_fetch:.6f}")
```

Salida:

```text
% python src/colors.py
Retrieving New Page...
Initial Request 0.506753
Subsequent Requests 0.000004
```

##### Propiedades para valores calculados

```python
class AverageList(list[int]):
    @property
    def average(self) -> float:
        return sum(self) / len(self)
```

```python
>>> a = AverageList([10, 8, 13, 9, 11, 14, 6, 4, 12, 7, 5])
>>> a.average
9.0
```

---

### Sección 5.3: De scripts a funciones, y de funciones a clases

El proceso de refactorización desde un script procedural extenso hacia un diseño orientado a objetos sigue tres etapas progresivas:

1. **De script lineal a funciones iniciales:** Encapsular la lógica en funciones (con una función `main()`) para permitir pruebas automatizadas y desacoplar la ejecución inmediata durante las importaciones.
2. **Descomposición funcional por elementos de datos:** Separar funciones extensas en funciones más pequeñas y cohesivas organizadas alrededor de estructuras de datos compartidas.
3. **Agrupación en definiciones de clase:** Unificar datos y comportamientos en clases coherentes con responsabilidades bien delimitadas.

Esta evolución permite aislar las fuentes de cambio:
- Variaciones en formatos de entrada o fuentes de datos.
- Cambios en las reglas de procesamiento y cálculo de negocio.
- Nuevos requisitos de salida, presentación o persistencia.
- Sustitución o actualización de librerías y frameworks externos.

---

### Sección 5.4: Repaso

Puntos clave tratados en este capítulo:

- La orientación a objetos resulta óptima cuando existen **datos y comportamientos interdependientes**. Para datos puros o funciones sin estado, las estructuras integradas y funciones simples son más apropiadas.
- En Python se favorece el **acceso directo a atributos** en lugar de *getters* y *setters* tradicionales.
- Las **propiedades (`@property`)** permiten añadir validación, cálculo dinámico y almacenamiento en caché de forma transparente sin alterar la interfaz pública.
- La refactorización incremental de scripts hacia funciones y clases facilita el mantenimiento, las pruebas unitarias y la extensibilidad del software.

---

### Sección 5.5: Ejercicios

1. **Identificación de propiedades:** Revisa código previo y busca métodos que devuelvan valores calculados sin aceptar parámetros ni alterar el estado. Refactorízalos como `@property`.
2. **Aplicación del principio DRY:** Localiza código duplicado en proyectos existentes y unifícalo mediante composición o herencia.
3. **Refactorización del script de pruebas del Capítulo 1:** Modela clases para representar los entornos y comandos, utilizando propiedades para calcular el estado global de ejecución de la suite de pruebas.

---

### Sección 5.6: Resumen

En este capítulo analizamos cómo identificar objetos significativos en el dominio del problema, cómo utilizar las propiedades para unificar el acceso a datos y comportamientos, y cómo transformar scripts procedurales en código modular y mantenible.

En el próximo capítulo, exploraremos las **clases base abstractas** y la **sobrecarga de operadores** en Python.
