# Parte 2: Diseño y Tipado Avanzado
## Capítulo 7: Pistas de Tipado en Python

A lo largo de este libro, prácticamente todos los ejemplos han incluido **pistas de tipado** (*type hints*). Python añadió sintaxis para "anotaciones" en Python 3.0 (ver PEP 3107: [https://peps.python.org/pep-3107/](https://peps.python.org/pep-3107/)). Esta sintaxis se aplicó al problema formal del tipado a partir de Python 3.6 (ver PEP 526: [https://peps.python.org/pep-0526/](https://peps.python.org/pep-0526/)). Desde entonces, el sistema de tipos formalizado de Python ha evolucionado y crecido de manera continua.

Actualmente, la especificación detallada se encuentra en [https://typing.readthedocs.io/en/latest/spec/](https://typing.readthedocs.io/en/latest/spec/).

Las pistas de tipado son opcionales. Sin embargo, aportan una clarificación indispensable y describen de forma explícita la intención del desarrollador. Además, las pistas de tipo pueden ser validadas por herramientas estáticas, proporcionando retroalimentación temprana sobre posibles discrepancias lógicas. Módulos como `dataclasses` o librerías como Pydantic hacen un uso intensivo de estas anotaciones para construir clases automáticamente y validar datos en tiempo de ejecución.

En este capítulo cubriremos los siguientes temas:

- Pistas de tipado y su uso en la programación orientada a objetos.
- Comprobación estática de tipos y herramientas de *linting*.
- Validación de valores en tiempo de ejecución con librerías como Pydantic.

---

### Sección 7.1: Pistas de tipo y programación orientada a objetos

En los diagramas de clases UML, podemos capturar relaciones estructurales directamente mediante anotaciones de tipos en Python (como `points: list[Point]`), lo que facilita la transición directa desde la pizarra al código fuente.

> **Figura 7.1: Diseño típico de clases**  
> *(Point, Rectangle, Square, Field ──▷ Polygon)*

Dos aspectos esenciales al pasar de UML a Python:
1. Anotar atributos, parámetros de métodos y tipos de retorno con nombres de tipo (por ejemplo, `def contains(self, point: Point) -> bool:`).
2. Aprovechar los tipos integrados genéricos como `list`, `dict`, `set` y `tuple`.

Herramientas como Mypy y Pyright analizan automáticamente aquellas funciones y métodos que disponen de pistas de tipo completas.

#### 7.1.1 Opcionalidad y Uniones

El objeto `None` se utiliza habitualmente como marcador de posición para valores ausentes.

Para definir parámetros opcionales o valores que pueden adoptar diferentes tipos, utilizamos la sintaxis de **unión** (`|`):

```python
from random import randint

def roll_dice(sides: list[int] | None = None) -> list[int]:
    dice_mix = sides if sides is not None else [6, 6]
    return [randint(1, s) for s in dice_mix]
```

Uso:

```python
>>> roll_dice([6, 6, 6])
[6, 1, 1]
>>> roll_dice()
[6, 3]
```

Un ejemplo más complejo con múltiples tipos posibles procesados mediante coincidencia de patrones (*pattern matching* con `match`):

```python
from random import randint

def roll_ndice(
    dice: int | list[int] | None = None,
    faces: int | None = None
) -> list[int]:
    match dice:
        case int() as n:
            dice_mix = n * [faces or 6]
        case None:
            if faces is None:
                dice_mix = [6, 6]
            else:
                dice_mix = [faces]
        case list() as dice_mix:
            assert faces is None, "faces must be None when dice is a list"
        case _:
            raise TypeError(f"can't parse {dice=!r}")
    return [randint(1, s) for s in dice_mix]
```

> [!TIP]
> Si una unión acumula más de 3 o 4 tipos distintos, suele ser síntoma de que conviene rediseñar la solución mediante una jerarquía de clases polimórfica en lugar de un `match-case` monolítico.

#### 7.1.2 Métodos sobrecargados (`@overload`)

Cuando una función tiene diferentes combinaciones válidas de parámetros que no pueden expresarse de forma precisa con una sola unión, utilizamos el decorador `@overload` del módulo `typing`:

```python
from typing import overload

@overload
def roll_ndice2(
    dice: list[int] | None
) -> list[int]: ...

@overload
def roll_ndice2(
    dice: int | None = None,
    faces: int | None = None
) -> list[int]: ...

def roll_ndice2(
    dice: int | list[int] | None = None,
    faces: int | None = None
) -> list[int]:
    # The body goes here...
```

Las definiciones decoradas con `@overload` contienen solo la firma y `...` (*Ellipsis*); la función final sin decorador contiene la implementación real que resuelve los argumentos.

#### 7.1.3 Tipos genéricos

Las colecciones integradas de Python son genéricas respecto a los tipos que almacenan. Podemos parametrizarlas formalmente:

- `list[int]`: Lista exclusiva de enteros.
- `dict[str, str]`: Diccionario con claves y valores de cadena.
- `set[tuple[float, float]]`: Conjunto de pares de números decimales.
- Tipos de `collections.abc` y `collections` (`defaultdict`, `deque`, `Counter`).
- `typing.NamedTuple` y `typing.TypedDict`.

#### 7.1.4 Protocolos y Duck Typing

Un **protocolo** (`typing.Protocol`) define formalmente las capacidades exigidas a un objeto sin obligar a heredar de una clase base concreta.

Por ejemplo, si una función solo necesita iterar sobre números enteros, exigir `Iterable[int]` es mucho más flexible y preciso que restringirlo artificialmente a `list[int]`:

```python
return [randint(1, s) for s in dice_mix]
```

Cualquier generador, tupla, conjunto o secuencia personalizada que implemente `__iter__()` satisfará el protocolo `Iterable[int]`.

---

### Sección 7.2: Comprobación estática y linting

El análisis estático verifica que las pistas de tipo y el código sean coherentes antes de la ejecución.

#### 7.2.1 Instalación de herramientas

Podemos instalar herramientas de análisis estático en nuestro entorno virtual:

Con `pip`:

```bash
python -m pip install ruff mypy pyright
```

Con `uv`:

```bash
uv add --dev ruff mypy pyright
```

#### 7.2.2 Comprobación de tipos con Mypy

Consideremos el archivo `bad_hints.py`:

```python
def odd(n: int) -> bool:
    return n % 2 != 0

def main() -> None:
    print(odd("Hello, world!"))

if __name__ == "__main__":
    main()
```

Al ejecutar `mypy`:

```bash
% mypy --strict src/bad_hints.py
```

Mypy detecta el error antes de la ejecución:

```text
ch_07/src/bad_hints.py:15: error: Argument 1 to "odd" has incompatible type "str"; expected "int" [arg-type]
```

#### 7.2.3 Comparación de herramientas

Herramientas como **Pyright** y **Mypy** ofrecen diagnósticos complementarios y soportan las últimas especificaciones de tipos (como la sintaxis `type Alias = ...` introducida en PEP 695).

#### 7.2.4 Verificación de calidad de código (*Linting*) con Ruff

El análisis con **Ruff** identifica malas prácticas, código muerto y posibles errores sutiles (*bugs*):

```bash
% ruff check src
```

---

### Sección 7.3: Comprobación de valores en tiempo de ejecución y el paquete Pydantic

Aunque las anotaciones estándar de Python no validan valores en tiempo de ejecución por defecto, librerías como **Pydantic** aprovechan el sistema de tipos para realizar validación estricta y transformación de datos.

Instalación con `uv`:

```bash
% uv add pydantic
```

#### Validación con validadores de campo (`@field_validator`)

```python
from pydantic import field_validator, Field
from pydantic.dataclasses import dataclass

@dataclass
class Result:
    success: bool
    exit_code: int
    duration: float

    @field_validator('exit_code')
    @classmethod
    def must_be_non_negative(cls, v: int) -> int:
        if v < 0:
            raise ValueError('must be non-negative')
        return v
```

#### Validación declarativa mediante `Annotated` y `Field`

```python
from typing import Annotated
from pydantic import Field
from pydantic.dataclasses import dataclass

@dataclass
class Result2:
    success: bool
    exit_code: Annotated[int, Field(ge=0)]
    duration: float
```

En este caso, la restricción `Field(ge=0)` garantiza en tiempo de ejecución que `exit_code` sea un entero mayor o igual a cero.

---

### Sección 7.4: Repaso

Puntos clave tratados en este capítulo:

- Las pistas de tipo documentan de forma verificable las interfaces y relaciones entre objetos.
- `|` expresa uniones y tipos opcionales (`| None`).
- `@overload` describe múltiples variantes válidas de invocación para funciones polimórficas complejas.
- Los tipos genéricos (`list[T]`, `dict[K, V]`) acotan con precisión el contenido de estructuras de datos.
- Los protocolos (`Protocol`) formalizan el tipado de pato sin imponer herencia rígida.
- Herramientas estáticas (`mypy`, `pyright`, `ruff`) detectan errores antes del despliegue.
- Librerías como Pydantic extienden el sistema de tipos para validar datos en tiempo de ejecución.

---

### Sección 7.5: Ejercicios

1. **Incorporación gradual de tipos:** Añade pistas de tipo completas y modo estricto de `mypy` a módulos o scripts previos.
2. **Modelado con protocolos:** Define un protocolo `Serializable` con métodos `to_json()` y `from_json()` y valida que distintas clases desacopladas lo cumplan.
3. **Validación de esquemas JSON con Pydantic:** Revisa el script de procesamiento de resultados de pruebas del Capítulo 1 y modela las estructuras JSON mediante dataclasses de Pydantic.

---

### Sección 7.6: Resumen

En este capítulo exploramos el sistema de pistas de tipado en Python, su integración con la programación orientada a objetos, el análisis estático mediante herramientas especializadas y la validación en tiempo de ejecución con Pydantic.

En el próximo capítulo, analizaremos en profundidad las **estructuras de datos en Python**, desde tuplas y dataclasses hasta diccionarios tipados, conjuntos y colas.
