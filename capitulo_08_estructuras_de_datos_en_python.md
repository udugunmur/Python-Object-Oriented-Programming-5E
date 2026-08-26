# Parte 2: Diseño y Tipado Avanzado
## Capítulo 8: Estructuras de Datos en Python

A partir del Capítulo 2 introdujimos conceptos fundamentales en la definición de clases. Los capítulos 3 y 6 ampliaron estas definiciones para dotar de mayores capacidades a las clases.

En muchos ejemplos hemos aprovechado las estructuras de datos integradas de Python. En este capítulo analizaremos el uso de estas estructuras como base para la definición de clases, cuándo conviene utilizarlas y cómo sacarles el máximo provecho.

Las clases integradas son ejemplos de tipos genéricos. Al escribir pistas de tipado (*type hints*), los parámetros ayudan a clarificar cómo se utilizará una colección en un contexto determinado. Asimismo, las clases integradas pueden extenderse mediante herencia para incorporar métodos propios.

Opciones que exploraremos en este capítulo:
- **Tuplas y tuplas con nombre (`NamedTuple`):** Contenedores inmutables de datos ordenados con acceso por nombre e índice.
- **`@dataclass`:** Permite definir clases orientadas a datos de forma limpia y concisa, generando automáticamente métodos mágicos (`__init__`, `__repr__`, `__eq__`, comparaciones de ordenación, etc.).
- **Diccionarios y `TypedDict`:** Mapeos clave-valor dinámicos y su formalización estructural mediante `typing.TypedDict`.
- **Listas y Conjuntos:** Colecciones secuenciales y colecciones de elementos únicos sin duplicados.
- **Colas (*Queues*):** Implementación de colas FIFO mediante listas, `collections.deque` y `queue.Queue`.

---

### Sección 8.1: Tuplas y tuplas con nombre

Las **tuplas** son secuencias inmutables de longitud fija. Son idóneas para agrupar un conjunto fijo de valores relacionados (como coordenadas `(x, y)` o componentes de color). Al ser inmutables, las tuplas cuyos elementos son todos inmutables poseen un valor hash (`__hash__`), lo que permite utilizarlas como claves de diccionarios o elementos de conjuntos.

Creación básica y empaquetado/desempaquetado:

```python
>>> stock = "AAPL", 226.20, 237.49, 164.075
>>> stock2 = ("AAPL", 226.20, 237.49, 164.075)
```

Función que procesa y desempaqueta una tupla:

```python
>>> import datetime
>>> def middle(stock, date):
...     symbol, current, high, low = stock
...     return (((high + low) / 2), date)
... 
>>> middle(("AAPL", 226.20, 237.49, 164.075), datetime.date(2024, 11, 22))
(200.7825, datetime.date(2024, 11, 22))
```

Tupla de un solo elemento (requiere coma final):

```python
>>> a = 42,
>>> a
(42,)
```

Acceso mediante índices y segmentación (*slicing*):

```python
>>> stock = "AAPL", 226.20, 237.49, 164.075
>>> high = stock[2]
>>> high
237.49
>>> stock[1:3]
(226.2, 237.49)
```

El acceso mediante índices numéricos ("números mágicos") resta legibilidad al código.

Función auxiliar funcional:

```python
>>> def high(quote):
...     symbol, current, high, low = quote
...     return high
... 
>>> high(stock)
237.49
```

#### 8.1.1 Tuplas con nombre mediante `typing.NamedTuple`

`NamedTuple` permite crear tuplas inmutables con atributos nombrados y tipados, generando automáticamente `__init__()`, `__repr__()`, `__eq__()` y `__hash__()`:

```python
from decimal import Decimal
from typing import NamedTuple

class Stock(NamedTuple):
    symbol: str
    current: Decimal
    high: Decimal
    low: Decimal
```

Instanciación y acceso:

```python
>>> Stock("AAPL", Decimal('226.20'), Decimal('237.49'), Decimal('164.075'))
Stock(symbol='AAPL', current=Decimal('226.20'), high=Decimal('237.49'), low=Decimal('164.075'))
>>> s2 = Stock("AAPL", Decimal('226.20'), high=Decimal('237.49'), low=Decimal('164.075'))
>>> s2
Stock(symbol='AAPL', current=Decimal('226.20'), high=Decimal('237.49'), low=Decimal('164.075'))
>>> s2.high
Decimal('237.49')
>>> s2[2]
Decimal('237.49')
>>> symbol, current, high, low = s2
>>> high
Decimal('237.49')
```

Al ser inmutable, cualquier intento de reasignación genera un error:

```python
>>> s2.current = Decimal('229.87')
Traceback (most recent call last):
  ...
    s2.current = Decimal('229.87')
    ^^^^^^^^^^
AttributeError: can't set attribute
```

> [!NOTE]
> Una tupla inmutable que contiene un elemento mutable (como una lista) no puede calcular su hash:

```python
>>> t = ("Relayer", ["Gates of Delirium", "Sound Chaser"])
>>> t[1].append("To Be Over")
>>> t
('Relayer', ['Gates of Delirium', 'Sound Chaser', 'To Be Over'])
>>> hash(t)
Traceback (most recent call last):
  ...
    hash(t)
TypeError: unhashable type: 'list'
```

Podemos añadir métodos y propiedades a una `NamedTuple`:

```python
class StockM(NamedTuple):
    symbol: str
    current: Decimal
    high: Decimal
    low: Decimal

    @property
    def middle(self) -> Decimal:
        return (self.high + self.low) / 2
```

```python
>>> s_m = StockM("AAPL", Decimal('226.20'), high=Decimal('237.49'), low=Decimal('164.075'))
>>> s_m.middle
Decimal('200.7825')
```

---

### Sección 8.2: Dataclasses

Introducidas en Python 3.7, las **dataclasses** simplifican la definición de clases de datos mediante el decorador `@dataclass`:

```python
from decimal import Decimal
from dataclasses import dataclass

@dataclass
class Stock:
    symbol: str
    current: Decimal
    high: Decimal
    low: Decimal
```

Uso:

```python
>>> s2 = Stock("AAPL", Decimal('226.20'), high=Decimal('237.49'), low=Decimal('164.075'))
>>> s2
Stock(symbol='AAPL', current=Decimal('226.20'), high=Decimal('237.49'), low=Decimal('164.075'))
>>> s2.high
Decimal('237.49')
>>> s2.current = Decimal('229.87')
```

#### Valores por defecto

```python
@dataclass
class StockDefaults:
    name: str
    current: Decimal = Decimal('0.00')
    high: Decimal = Decimal('0.00')
    low: Decimal = Decimal('0.00')
```

```python
>>> StockDefaults("GOOG")
StockDefaults(name='GOOG', current=Decimal('0.00'), high=Decimal('0.00'), low=Decimal('0.00'))
```

#### Comparación y ordenación automática (`order=True`)

```python
@dataclass(order=True)
class StockOrdered:
    name: str
    current: Decimal = Decimal('0.00')
    high: Decimal = Decimal('0.00')
    low: Decimal = Decimal('0.00')
```

```python
>>> stock_ordered1 = StockOrdered("GOOG", Decimal('166.57'), Decimal('193.31'), Decimal('129.40'))
>>> stock_ordered2 = StockOrdered("GOOG")
>>> stock_ordered3 = StockOrdered("GOOG", Decimal('142.45'), high=Decimal('151.85'), low=Decimal('84.95'))
>>> stock_ordered1 < stock_ordered2
False
>>> stock_ordered1 > stock_ordered2
True
>>> from pprint import pprint
>>> pprint(sorted([stock_ordered1, stock_ordered2, stock_ordered3]))
[StockOrdered(name='GOOG', current=Decimal('0.00'), high=Decimal('0.00'), low=Decimal('0.00')),
 StockOrdered(name='GOOG', current=Decimal('142.45'), high=Decimal('151.85'), low=Decimal('84.95')),
 StockOrdered(name='GOOG', current=Decimal('166.57'), high=Decimal('193.31'), low=Decimal('129.40'))]
```

La opción `frozen=True` genera clases inmutables equivalentes a tuplas con nombre pero con mayor flexibilidad (como el método de inicialización posterior `__post_init__()`).

---

### Sección 8.3: Diccionarios y diccionarios tipados

Los diccionarios (`dict`) asocian claves con valores mediante tablas hash:

```python
>>> stocks = {
...     "GOOG": Stock("GOOG", Decimal('166.57'), high=Decimal('193.31'), low=Decimal('129.40')),
...     "MSFT": Stock("MSFT", Decimal('110.41'), high=Decimal('110.45'), low=Decimal('109.84')),
... }
```

#### Agrupación de datos con `setdefault`, `defaultdict` y `Counter`

Dada una estructura de cotizaciones diarias:

```python
import datetime
from decimal import Decimal
from typing import NamedTuple

class DailyQuote(NamedTuple):
    symbol: str
    date: datetime.date
    price: Decimal
```

Agrupación manual con `setdefault`:

```python
>>> summary: dict[str, list[DailyQuote]] = {}
>>> for dq in some_source_of_daily_quotes:
...     summary.setdefault(dq.symbol, list())
...     summary[dq.symbol].append(dq)
```

Agrupación idiomática con `defaultdict`:

```python
>>> from collections import defaultdict
>>> summary: defaultdict[str, list[DailyQuote]] = defaultdict(list)
>>> for dq in some_source_of_daily_quotes:
...     summary[dq.symbol].append(dq)
```

Conteo de frecuencias con `Counter`:

```python
>>> from collections import Counter
>>> symbols = (dq.symbol for dq in some_source_of_daily_quotes)
>>> frequency = Counter(symbols)
```

#### Extensión de diccionarios con `__missing__()`

```python
class StockQuoteSummary(dict[str, list[DailyQuote]]):
    def __missing__(self, symbol: str) -> list[DailyQuote]:
        self[symbol] = list()
        return self[symbol]

    def by_date(self, symbol: str) -> list[DailyQuote]:
        return sorted(self[symbol], key=lambda dq: dq.date)
```

```python
>>> summary = StockQuoteSummary()
>>> for dq in some_source_of_daily_quotes:
...     summary[dq.symbol].append(dq)
>>> for symbol in summary:
...     print(summary.by_date(symbol))
```

#### 8.3.1 Diccionarios tipados (`typing.TypedDict`)

Permiten tipar esquemas JSON y diccionarios heterogéneos:

```python
from decimal import Decimal
from typing import TypedDict

class Range(TypedDict):
    low: Decimal
    high: Decimal

class Stock(TypedDict):
    symbol: str
    current: Decimal
    range: Range
```

```python
>>> s_td = Stock(
...     symbol='GOOG',
...     current=Decimal('166.57'),
...     range=Range(low=Decimal('129.40'), high=Decimal('193.31'))
... )
```

Atributos opcionales con `NotRequired`:

```python
import datetime
from typing import TypedDict, NotRequired

class StockN(TypedDict):
    symbol: str
    name: NotRequired[str]
    current: Decimal
    range: Range
    date: NotRequired[datetime.date]
```

#### 8.3.2 Criterios de diseño entre Diccionarios, Tuplas y Dataclasses

1. **`@dataclass`:** Primera opción para modelar entidades de dominio con estado, métodos y validaciones.
2. **`NamedTuple` / `frozen=True`:** Óptimo para objetos de datos inmutables y ligeros.
3. **`TypedDict`:** Ideal para interactuar con APIs REST/JSON y esquemas dinámicos.

#### 8.3.3 Claves de diccionario y colisiones de Hash

Para ser clave de diccionario, un objeto debe implementar `__hash__()` y `__eq__()`. Objetos iguales deben producir hashes idénticos:

```python
>>> x = 2020
>>> y = 2305843009213695971
>>> hash(x) == hash(y)
True
>>> x == y
False
```

---

### Sección 8.4: Listas

Las listas (`list[T]`) almacenan secuencias mutables y ordenadas de elementos del mismo tipo conceptual.

Ejemplo ineficiente usando listas en lugar de diccionarios:

```python
import string

CHARACTERS = list(string.ascii_letters) + [" "]

def letter_frequency(sentence: str) -> list[tuple[str, int]]:
    frequencies = [(c, 0) for c in CHARACTERS]
    for letter in sentence:
        index = CHARACTERS.index(letter)
        frequencies[index] = (letter, frequencies[index][1] + 1)
    non_zero = [(letter, count) for letter, count in frequencies if count > 0]
    return non_zero
```

#### 8.4.1 Ordenación de listas

Ordenación personalizada de subtipos heterogéneos (*tagged unions*):

| Data Source | Timestamp | Creation Date | Name, Owner, etc. |
| :--- | :--- | :--- | :--- |
| Local | 1607280522.68012 | | "Some File", etc. |
| Remote | | "2020-12-06T13:47:52.849153" | "Another File", etc. |
| Local | 1579373292.452993 | | "This File", etc. |
| Remote | | "2020-01-18T13:48:12.452993" | "That File", etc. |

*Tabla 8.1: Datos de muestra*

Implementación con `__lt__()`:

```python
from typing import cast, Any
from dataclasses import dataclass
import datetime
from datetime import timezone

@dataclass(frozen=True)
class MultiItem:
    data_source: str
    timestamp: float | None
    creation_date: str | None
    name: str
    owner_etc: str

    def __lt__(self, other: Any) -> bool:
        if self.data_source == "Local":
            self_datetime = datetime.datetime.fromtimestamp(
                cast(float, self.timestamp), tz=timezone.utc
            )
        else:
            self_datetime = datetime.datetime.fromisoformat(
                cast(str, self.creation_date)
            ).replace(tzinfo=timezone.utc)

        if other.data_source == "Local":
            other_datetime = datetime.datetime.fromtimestamp(
                cast(float, other.timestamp), tz=timezone.utc
            )
        else:
            other_datetime = datetime.datetime.fromisoformat(
                cast(str, other.creation_date)
            ).replace(tzinfo=timezone.utc)

        return self_datetime < other_datetime
```

Uso:

```python
>>> mi_0 = MultiItem("Local", 1607262522.000000, None, "Some File", "etc. 0")
>>> mi_1 = MultiItem("Remote", None, "2020-12-06T13:47:52.000001", "Another File", "etc. 1")
>>> mi_2 = MultiItem("Local", 1579355292.000002, None, "This File", "etc. 2")
>>> mi_3 = MultiItem("Remote", None, "2020-01-18T13:48:12.000003", "That File", "etc. 3")
>>> file_list = [mi_0, mi_1, mi_2, mi_3]
>>> file_list.sort()
```

#### Ordenación mediante función clave (`key=`) con `attrgetter` y `lambda`

Es más limpio transformar el valor al vuelo con una propiedad unificada:

```python
from functools import total_ordering

@total_ordering
@dataclass(frozen=True)
class MultiItemTO:
    data_source: str
    timestamp: float | None
    creation_date: str | None
    name: str
    owner_etc: str

    @property
    def datetime(self) -> datetime.datetime:
        if self.data_source == "Local":
            return datetime.datetime.fromtimestamp(
                cast(float, self.timestamp), tz=timezone.utc
            )
        else:
            return datetime.datetime.fromisoformat(
                cast(str, self.creation_date)
            ).replace(tzinfo=timezone.utc)

    def __eq__(self, other: object) -> bool:
        return self.datetime == cast(MultiItemTO, other).datetime

    def __lt__(self, other: object) -> bool:
        return self.datetime < cast(MultiItemTO, other).datetime
```

Ordenación declarativa con `attrgetter` o `lambda`:

```python
>>> from operator import attrgetter
>>> file_list.sort(key=attrgetter('datetime'))
>>> file_list.sort(key=lambda item: item.name)
```

---

### Sección 8.5: Conjuntos (`set`)

Los **conjuntos** almacenan elementos únicos no ordenados y ofrecen comprobaciones de pertenencia (`in`) en tiempo constante $\mathcal{O}(1)$:

```python
>>> song_library = [
...     ("Phantom Of The Opera", "Sarah Brightman"),
...     ("Knocking On Heaven's Door", "Guns N' Roses"),
...     ("Captain Nemo", "Sarah Brightman"),
...     ("Patterns In The Ivy", "Opeth"),
...     ("November Rain", "Guns N' Roses"),
...     ("Beautiful", "Sarah Brightman"),
...     ("Mal's Song", "Vixy and Tony"),
... ]
>>> artists = set()
>>> for song, artist in song_library:
...     artists.add(artist)
>>> artists
{'Opeth', "Guns N' Roses", 'Vixy and Tony', 'Sarah Brightman'}
```

Comprensión de conjuntos:

```python
>>> artists = set(artist for _, artist in song_library)
```

#### Operaciones de álgebra de conjuntos: Unión (`|`), Intersección (`&`), Diferencia Simétrica (`^`)

```python
>>> dusty_artists = {
...     "Sarah Brightman",
...     "Guns N' Roses",
...     "Opeth",
...     "Vixy and Tony",
... }
>>> steve_artists = {"Yes", "Guns N' Roses", "Genesis"}
>>> all = dusty_artists | steve_artists
>>> both = dusty_artists.intersection(steve_artists)
>>> not_both = dusty_artists ^ steve_artists
```

---

### Sección 8.6: Tres tipos de colas

Una cola sigue el principio **FIFO** (*First In, First Out*):

> **Figura 8.1: Concepto de cola**  
> `append()` / `put()` ──▷ `[ Elemento 3 | Elemento 2 | Elemento 1 ]` ──▷ `pop(0)` / `popleft()` / `get()`

#### 1. Cola basada en `list`

```python
from pathlib import Path

class ListQueue(list[Path]):
    def put(self, item: Path) -> None:
        self.append(item)

    def get(self) -> Path:
        return self.pop(0)

    def empty(self) -> bool:
        return len(self) == 0
```

#### 2. Cola basada en `collections.deque` (Óptima para un solo hilo)

```python
from pathlib import Path
from typing import Deque

class DeQueue(Deque[Path]):
    def put(self, item: Path) -> None:
        self.append(item)

    def get(self) -> Path:
        return self.popleft()

    def empty(self) -> bool:
        return len(self) == 0
```

#### 3. Cola protegida para concurrencia (`queue.Queue`)

```python
from pathlib import Path
import queue

class ThreadQueue(queue.Queue[Path]):
    pass
```

Tipo unificado mediante unión de tipos:

```python
PathQueue = ListQueue | DeQueue | ThreadQueue
```

---

### Sección 8.7: Repaso

Puntos clave tratados en este capítulo:

- **`NamedTuple`:** Inmutabilidad y eficiencia para registros estáticos con acceso por nombre.
- **`@dataclass`:** Flexibilidad total para objetos de dominio mutables o inmutables con métodos autogenerados.
- **`dict`, `defaultdict`, `Counter` y `TypedDict`:** Mapeos clave-valor optimizados para agregaciones y contratos JSON.
- **`list` y `set`:** Secuencias ordenadas indexables frente a conjuntos de elementos únicos con pertenencia $\mathcal{O}(1)$.
- **Colas FIFO:** Implementaciones especializadas con `deque` y `queue.Queue`.

---

### Sección 8.8: Ejercicios

1. **Refactorización a Dataclasses:** Revisa clases tradicionales con métodos `__init__` repetitivos y transfórmalas en `@dataclass(frozen=True, order=True)`.
2. **Estructuras jerárquicas con TypedDict:** Modela esquemas complejos de respuestas de APIs externas usando `TypedDict` anidados con campos `Required` y `NotRequired`.
3. **Procesador de archivos en cola:** Implementa un recorrido de árbol de directorios utilizando `DeQueue` para indexar archivos de forma iterativa sin recursión.

---

### Sección 8.9: Resumen

En este capítulo exploramos el amplio catálogo de estructuras de datos que ofrece Python y cómo seleccionarlas, extenderlas y combinarlas en nuestros diseños orientados a objetos.

En el próximo capítulo, estudiaremos la **intersección entre la programación orientada a objetos y la programación funcional** en Python.
