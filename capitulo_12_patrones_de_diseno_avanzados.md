# Parte 3: Patrones de Diseño y Buenas Prácticas
## Capítulo 12: Patrones de Diseño Avanzados

En este capítulo analizaremos patrones de diseño más avanzados, cubriendo tanto las implementaciones canónicas como las variantes idiomáticas en Python:

- El patrón **Adapter** (Adaptador)
- El patrón **Facade** (Fachada)
- Inicialización perezosa y el patrón **Flyweight** (Peso mosca)
- Optimización de memoria con **`__slots__`**
- El patrón **Abstract Factory** (Fábrica abstracta)
- El patrón **Composite** (Objeto compuesto)
- El patrón **Template** (Método plantilla)

---

### Sección 12.1: El patrón Adapter

El patrón **Adapter** se utiliza para conectar dos objetos preexistentes cuyas interfaces no encajan de forma directa, actuando como un intermediario o convertidor de protocolos sin requerir la modificación del código fuente original.

A diferencia del patrón Decorator (cuya interfaz suele ser idéntica al objeto que envuelve), un adaptador **traduce** una interfaz hacia otra distinta.

> **Figura 12.1: Patrón Adapter**  
> `Client` ──▷ interactúa con `Adapter` (`load_data()`) ──▷ traduce y delega en `Implementation` (`read_raw_data()`, `parse_raw_data()`, `create_useful_object()`).

#### 12.1.1 Ejemplo de Adapter: Conversión de marcas temporales en logs

Dada una clase preexistente y probada que calcula intervalos de tiempo a partir de cadenas sin formato `HHMMSS.S`:

```python
class TimeSince:
    """Expects time as six digits, no punctuation."""
    def parse_time(self, time: str) -> tuple[float, float, float]:
        return (
            float(time[0:2]),
            float(time[2:4]),
            float(time[4:]),
        )

    def __init__(self, starting_time: str) -> None:
        self.hr, self.min, self.sec = self.parse_time(starting_time)
        self.start_seconds = ((self.hr * 60) + self.min) * 60 + self.sec

    def interval(self, log_time: str) -> float:
        log_hr, log_min, log_sec = self.parse_time(log_time)
        log_seconds = ((log_hr * 60) + log_min) * 60 + log_sec
        return log_seconds - self.start_seconds
```

Uso interactivo:

```python
>>> ts = TimeSince("000123")  # Log started at 00:01:23
>>> ts.interval("020304")
7301.0
>>> ts.interval("030405")
10962.0
```

Datos de prueba de registros de log:

```python
>>> data = [
...     ("000123", "INFO", "Gila Flats 1959-08-20"),
...     ("000142", "INFO", "test block 15"),
...     ("004201", "ERROR", "intrinsic field chamber door locked"),
...     ("004210.11", "INFO", "generator power active"),
...     ("004232.33", "WARNING", "extra mass detected")
... ]
```

Queremos un procesador de logs que calcule intervalos relativos a cada línea de error (`ERROR`), pero `TimeSince` mantiene un estado inicial fijo:

```python
class LogProcessor:
    def __init__(self, log_entries: list[tuple[str, str, str]]) -> None:
        self.log_entries = log_entries

    def report(self) -> None:
        first_time, first_sev, first_msg = self.log_entries[0]
        for log_time, severity, message in self.log_entries:
            if severity == "ERROR":
                first_time = log_time
            interval = ### Need to compute an interval ???
            print(f"{interval:8.2f} | {severity:7s} {message}")
```

#### Implementación del Adaptador (`IntervalAdapter`)

```python
from typing import Optional

class IntervalAdapter:
    def __init__(self) -> None:
        self.ts: Optional[TimeSince] = None

    def time_offset(self, start: str, now: str) -> float:
        if self.ts is None:
            self.ts = TimeSince(start)
        else:
            h_m_s = self.ts.parse_time(start)
            if h_m_s != (self.ts.hr, self.ts.min, self.ts.sec):
                self.ts = TimeSince(start)
        return self.ts.interval(now)
```

Integración en `LogProcessor`:

```python
class LogProcessor:
    def __init__(
        self, log_entries: list[tuple[str, str, str]]
    ) -> None:
        self.log_entries = log_entries
        self.time_convert = IntervalAdapter()

    def report(self) -> None:
        first_time, first_sev, first_msg = self.log_entries[0]
        for log_time, severity, message in self.log_entries:
            if severity == "ERROR":
                first_time = log_time
            interval = self.time_convert.time_offset(first_time, log_time)
            print(f"{interval:8.2f} | {severity:7s} {message}")
```

---

### Sección 12.2: El patrón Facade

El patrón **Facade** proporciona una interfaz unificada y simplificada para un subsistema complejo o conjunto de clases interconectadas, ocultando la complejidad operativa detrás de una API de alto nivel.

> **Figura 12.2: Patrón Facade**  
> `Client` ──▷ `Facade` (`make_all_images()`) ──▷ coordina `FindUML` y `PlantUML` dentro del subsistema.

#### 12.2.1 Ejemplo de Facade: Automatización de diagramas UML con PlantUML

Búsqueda de archivos `.uml`:

```python
import re
from pathlib import Path
from typing import Iterator

class FindUML:
    def __init__(self, base: Path) -> None:
        self.base = base
        self.start_pattern = re.compile(r"@startuml *(.*)")

    def uml_file_iter(self) -> Iterator[tuple[Path, Path]]:
        for source in self.base.glob("**/*.uml"):
            if any(n.startswith(".") for n in source.parts):
                continue
            body = source.read_text()
            for output_name in self.start_pattern.findall(body):
                if output_name:
                    target = source.parent / output_name
                else:
                    target = source.with_suffix(".png")
                yield (source.relative_to(self.base), target.relative_to(self.base))
```

Ejecutor de subprocesos Java para PlantUML:

```python
from pathlib import Path
import subprocess

class PlantUML:
    local_dir = Path.cwd()

    def __init__(
        self, plantjar: str | Path = "plantuml-1.2024.7.jar", venv_name: str = ".venv"
    ) -> None:
        def find_first(name: str | Path) -> Path:
            places = [
                self.local_dir,
                self.local_dir / venv_name,
            ]
            places += Path.cwd().parents
            for place in places:
                if (path := place / name).exists():
                    return path
            raise FileNotFoundError(f"could not find {plantjar}")

        match plantjar:
            case Path() as path if path.is_absolute():
                self.plantjar = path
            case Path() as path if not path.is_absolute():
                self.plantjar = find_first(path)
            case str() as name:
                self.plantjar = find_first(name)

    def process(self, source: Path) -> None:
        env: dict[str, str] = {
            # Rarely needed...
            # "GRAPHVIZ_DOT": str(Path("/")/"to"/"graphviz"/"dot"),
        }
        command = ["java", "-jar", str(self.plantjar), "-progress", str(source)]
        subprocess.run(command, env=env, check=True)
```

Fachada unificada (`GenerateImages`):

```python
class GenerateImages:
    def __init__(self, base: Path, verbose: int = 0) -> None:
        self.finder = FindUML(base)
        self.painter = PlantUML()
        self.verbose = verbose

    def make_all_images(self) -> None:
        for source, target in self.finder.uml_file_iter():
            if not target.exists() or source.stat().st_mtime > target.stat().st_mtime:
                print(f"Processing {source} -> {target}")
                self.painter.process(source)
            else:
                if self.verbose > 0:
                    print(f"Skipping {source} -> {target}")

def main() -> None:
    g = GenerateImages(Path.cwd())
    g.make_all_images()

if __name__ == "__main__":
    main()
```

---

### Sección 12.3: El patrón Flyweight

El patrón **Flyweight** optimiza el consumo de memoria compartiendo de forma transparente el estado común entre un gran número de objetos pequeños similares en lugar de duplicar datos.

> **Figura 12.3: Patrón Flyweight**  
> `Client` ──▷ `FlyweightFactory` ──▷ retorna instancias compartidas de `Flyweight` que operan sobre `SpecificState` externo.

#### 12.3.1 Ejemplo de Flyweight en Python: Tramas GPS y referencias débiles (`weakref`)

Para evitar duplicar buffers masivos de bytes y evitar fugas por referencias circulares, utilizamos objetos Flyweight que apuntan al mismo buffer subyacente mediante `weakref.ref`:

```python
from collections.abc import Sequence, Iterator
from typing import overload

class Buffer(Sequence[int]):
    def __init__(self, content: bytes) -> None:
        self.content = content

    def __len__(self) -> int:
        return len(self.content)

    def __iter__(self) -> Iterator[int]:
        return iter(self.content)

    @overload
    def __getitem__(self, index: int) -> int: ...
    @overload
    def __getitem__(self, index: slice) -> bytes: ...
    def __getitem__(self, index: int | slice) -> int | bytes:
        return self.content[index]
```

Clase base abstracta Flyweight para mensajes GPS:

```python
import abc
import weakref

class Message(abc.ABC):
    def __init__(self) -> None:
        self.buffer: weakref.ReferenceType[Buffer]
        self.offset: int
        self.end: int | None
        self.commas: list[int]

    def from_buffer(self, buffer: Buffer, offset: int) -> "Message":
        self.buffer = weakref.ref(buffer)
        self.offset = offset
        self.commas = [offset]
        self.end = None
        for index in range(offset, offset + 82):
            if buffer[index] == ord(b","):
                self.commas.append(index)
            elif buffer[index] == ord(b"*"):
                self.commas.append(index)
                self.end = index + 3
                break
        if self.end is None:
            raise GPSError("Incomplete")  # TODO: confirm checksum.
        return self

    def __getitem__(self, field: int) -> bytes:
        if not hasattr(self, "buffer") or (buffer := self.buffer()) is None:
            raise RuntimeError("Broken reference")
        start, end = self.commas[field] + 1, self.commas[field + 1]
        return buffer[start:end]

    def get_fix(self) -> Point:
        return Point.from_bytes(
            self.latitude(), self.lat_n_s(), self.longitude(), self.lon_e_w()
        )

    @abc.abstractmethod
    def latitude(self) -> bytes: ...
    @abc.abstractmethod
    def lat_n_s(self) -> bytes: ...
    @abc.abstractmethod
    def longitude(self) -> bytes: ...
    @abc.abstractmethod
    def lon_e_w(self) -> bytes: ...
```

Subclase Flyweight concreta:

```python
class GPGLL(Message):
    def latitude(self) -> bytes:
        return self[1]

    def lat_n_s(self) -> bytes:
        return self[2]

    def longitude(self) -> bytes:
        return self[3]

    def lon_e_w(self) -> bytes:
        return self[4]
```

Fábrica de mensajes:

```python
def message_factory(header: bytes) -> Message | None:
    if header == b"GPGGA":
        return GPGGA()
    elif header == b"GPGLL":
        return GPGLL()
    elif header == b"GPRMC":
        return GPRMC()
    else:
        return None
```

#### 12.3.2 Múltiples mensajes en un buffer compartido

```python
>>> buffer_2 = Buffer(
...     b"$GPGLL,3751.65,S,14507.36,E*77\r\n"
...     b"$GPGLL,3723.2475,N,12158.3416,W,161229.487,A,A*41\r\n"
... )
>>> start = 0
>>> flyweight = message_factory(buffer_2[start+1 : start+6])
>>> p_1 = flyweight.from_buffer(buffer_2, start).get_fix()
>>> print(p_1)
(3751.6500S, 14507.3600E)
```

#### 12.3.3 Optimización de memoria en Python mediante `__slots__`

Al definir `__slots__`, Python reemplaza el diccionario dinámico interno `__dict__` por una estructura en C de tamaño fijo, reduciendo drásticamente el espacio consumido por cada instancia:

```python
from math import radians, floor

class Point:
    __slots__ = ("latitude", "longitude")

    def __init__(self, latitude: float, longitude: float) -> None:
        self.latitude = latitude
        self.longitude = longitude

    def __repr__(self) -> str:
        return f"Point(latitude={self.latitude}, longitude={self.longitude})"

    @classmethod
    def from_bytes(
        cls,
        latitude: bytes,
        N_S: bytes,
        longitude: bytes,
        E_W: bytes,
    ) -> "Point":
        lat_deg = float(latitude[:2]) + float(latitude[2:]) / 60
        lat_sign = 1 if N_S.upper() == b"N" else -1
        lon_deg = float(longitude[:3]) + float(longitude[3:]) / 60
        lon_sign = 1 if E_W.upper() == b"E" else -1
        return Point(lat_deg * lat_sign, lon_deg * lon_sign)

    def __str__(self) -> str:
        lat = abs(self.latitude)
        lat_deg = floor(lat)
        lat_min_sec = 60 * (lat - lat_deg)
        lat_dir = "N" if self.latitude > 0 else "S"
        lon = abs(self.longitude)
        lon_deg = floor(lon)
        lon_min_sec = 60 * (lon - lon_deg)
        lon_dir = "E" if self.longitude > 0 else "W"
        return f"({lat_deg:02.0f}°{lat_min_sec:07.4f}{lat_dir}, {lon_deg:03.0f}°{lon_min_sec:07.4f}{lon_dir})"
```

> [!TIP]
> En dataclasses, podemos habilitar esta optimización automáticamente mediante `@dataclass(slots=True)`.

---

### Sección 12.4: El patrón Abstract Factory

El patrón **Abstract Factory** proporciona una interfaz para crear familias de objetos relacionados o dependientes sin especificar sus clases concretas directamente en el cliente.

> **Figura 12.6: Patrón Abstract Factory**  
> `Client` ──▷ `AbstractFactory` (`make_card()`, `make_hand()`)  
> `ConcreteFactory1` produce `Card1` y `Hand1`  
> `ConcreteFactory2` produce `Card2` y `Hand2`.

#### 12.4.1 Ejemplo de Abstract Factory: Familias de cartas para Cribbage y Poker

Modelos base de dominio:

```python
import abc
from enum import Enum
from typing import NamedTuple

class Suit(str, Enum):
    Clubs = "\N{Black Club Suit}"
    Diamonds = "\N{Black Diamond Suit}"
    Hearts = "\N{Black Heart Suit}"
    Spades = "\N{Black Spade Suit}"

class Card(NamedTuple):
    rank: int
    suit: Suit
    def __str__(self) -> str:
        return f"{self.rank}{self.suit.value}"

class Trick(int, Enum):
    pass

class Hand(list[Card]):
    def __init__(self, *cards: Card) -> None:
        super().__init__(cards)

    @abc.abstractmethod
    def scoring(self) -> list[Trick]:
        ...
```

Fábrica abstracta:

```python
class CardGameFactory(abc.ABC):
    @abc.abstractmethod
    def make_card(self, rank: int, suit: Suit) -> "Card":
        ...

    @abc.abstractmethod
    def make_hand(self, *cards: Card) -> "Hand":
        ...
```

Implementación de Poker:

```python
class PokerCard(Card):
    def __str__(self) -> str:
        if self.rank == 14:
            return f"A{self.suit}"
        return f"{self.rank}{self.suit}"

class PokerHand(Hand):
    def scoring(self) -> list[Trick]:
        ### details omitted...
        return [rank]

class PokerFactory(CardGameFactory):
    def make_card(self, rank: int, suit: Suit) -> "Card":
        if rank == 1:  # Aces above kings
            rank = 14
        return PokerCard(rank, suit)

    def make_hand(self, *cards: Card) -> "Hand":
        return PokerHand(*cards)
```

Pruebas con fábrica de Cribbage:

```python
>>> factory = CribbageFactory()
>>> cards = [
...     factory.make_card(6, Suit.Clubs),
...     factory.make_card(7, Suit.Diamonds),
...     factory.make_card(8, Suit.Hearts),
...     factory.make_card(9, Suit.Spades),
... ]
>>> starter = factory.make_card(5, Suit.Spades)
>>> hand = factory.make_hand(*cards)
>>> score = sorted(hand.upcard(starter).scoring())
>>> [t.name for t in score]
['Fifteen', 'Fifteen', 'Run_5']
```

#### 12.4.2 Fábricas abstractas idiomáticas mediante `typing.Protocol`

```python
from typing import Protocol

class CardGameFactoryProtocol(Protocol):
    def make_card(self, rank: int, suit: Suit) -> "Card":
        ...
    def make_hand(self, *cards: Card) -> "Hand":
        ...
```

---

### Sección 12.5: El patrón Composite

El patrón **Composite** organiza objetos en estructuras de árbol para representar jerarquías de parte-todo, permitiendo que los clientes traten de forma homogénea tanto a objetos individuales (*Leaf*) como a composiciones de objetos (*Composite*).

> **Figura 12.9: Patrón Composite**  
> `Component` (`move()`, `copy()`, `remove()`)  
> ├── `Leaf (File)`  
> └── `Composite (Folder)` (contiene colección de `Component`).

#### 12.5.1 Ejemplo de Composite: Jerarquía del sistema de archivos

Clase abstracta `Node`:

```python
import abc

class Node(abc.ABC):
    def __init__(
        self, name: str,
    ) -> None:
        self.name = name
        self.parent: "Folder | None" = None

    def move(self, new_place: "Folder") -> None:
        previous = self.parent
        new_place.add_child(self)
        if previous:
            del previous.children[self.name]

    @abc.abstractmethod
    def copy(self, new_folder: "Folder") -> None:
        ...

    @abc.abstractmethod
    def remove(self) -> None:
        ...
```

Nodo contenedor compuesto (`Folder`):

```python
from typing import cast

class Folder(Node):
    def __init__(self, name: str, children: dict[str, "Node"] | None = None) -> None:
        super().__init__(name)
        self.children = children or {}

    def __repr__(self) -> str:
        return f"Folder({self.name!r}, {self.children!r})"

    def add_child(self, node: "Node") -> "Node":
        node.parent = self
        return self.children.setdefault(node.name, node)

    def copy(self, new_folder: "Folder") -> None:
        target = cast(Folder, new_folder.add_child(Folder(self.name)))
        for c in self.children:
            self.children[c].copy(target)

    def remove(self) -> None:
        names = list(self.children)
        for c in names:
            self.children[c].remove()
        if self.parent:
            del self.parent.children[self.name]
```

Nodo hoja (`File`):

```python
class File(Node):
    def __repr__(self) -> str:
        return f"File({self.name!r})"

    def copy(self, new_folder: "Folder") -> None:
        new_folder.add_child(File(self.name))

    def remove(self) -> None:
        if self.parent:
            del self.parent.children[self.name]
```

Uso y reubicación en el árbol compuesto:

```python
>>> tree = Folder("Tree")
>>> tree.add_child(Folder("src"))
Folder('src', {})
>>> tree.children["src"].add_child(File("ex1.py"))
File('ex1.py')
>>> tree.children["src"].add_child(File("test1.py"))
File('test1.py')
>>> test1 = tree.children["src"].children["test1.py"]
>>> tree.add_child(Folder("tests"))
Folder('tests', {})
>>> test1.move(tree.children["tests"])
```

Estructura resultante:
```text
+-- Tree
    +-- src
    |   +-- ex1.py
    +-- tests
        +-- test1.py
```

---

### Sección 12.6: El patrón Template (Método Plantilla)

El patrón **Template** define el esqueleto invariante de un algoritmo en un método de la superclase, delegando la implementación de ciertos pasos específicos a las subclases sin alterar la estructura general del proceso.

> **Figura 12.11: Patrón Template**  
> `QueryTemplate` (`process_format()` define el orden de ejecución: `connect()` $\to$ `construct_query()` $\to$ `do_query()` $\to$ `output_results()`).

#### 12.6.1 Ejemplo de Template: Reportes de base de datos SQLite

Preparación de la base de datos de ventas:

```python
import sqlite3

def db_preparation(db_name: str = "sales.db") -> sqlite3.Connection:
    conn = sqlite3.connect(db_name)
    conn.execute(
        """
        CREATE TABLE IF NOT EXISTS Sales (
            salesperson text,
            amt currency,
            year integer,
            model text,
            new boolean
        )
        """
    )
    conn.execute(
        """
        DELETE FROM Sales
        """
    )
    values = [
        ('Tim', 16000, 2010, 'Honda Fit', 'true'),
        ('Tim', 9000, 2006, 'Ford Focus', 'false'),
        ('Hannah', 8000, 2004, 'Dodge Neon', 'false'),
        ('Hannah', 28000, 2009, 'Ford Mustang', 'true'),
        ('Hannah', 50000, 2010, 'Lincoln Navigator', 'true'),
        ('Jason', 20000, 2008, 'Toyota Prius', 'false')
    ]
    conn.executemany(
        """
        INSERT INTO Sales VALUES (?,?,?,?,?)
        """, values)
    conn.commit()
    return conn
```

Clase base plantilla (`QueryTemplate`):

```python
import contextlib
import csv
import sys
from typing import ContextManager, TextIO, cast

class QueryTemplate:
    def __init__(self, db_name: str = "sales.db") -> None:
        self.db_name = db_name
        self.conn: sqlite3.Connection
        self.results: list[tuple[str, ...]]
        self.query: str
        self.header: list[str]

    def connect(self) -> None:
        self.conn = sqlite3.connect(self.db_name)

    def construct_query(self) -> None:
        raise NotImplementedError("construct_query not implemented")

    def do_query(self) -> None:
        results = self.conn.execute(self.query)
        self.results = results.fetchall()

    def output_context(self) -> ContextManager[TextIO]:
        self.target_file = sys.stdout
        return cast(ContextManager[TextIO], contextlib.nullcontext())

    def output_results(self) -> None:
        writer = csv.writer(self.target_file)
        writer.writerow(self.header)
        writer.writerows(self.results)

    def process_format(self) -> None:
        self.connect()
        self.construct_query()
        self.do_query()
        with self.output_context():
            self.output_results()
```

Subclases concretas con pasos personalizados:

```python
import datetime
from pathlib import Path

class NewVehiclesQuery(QueryTemplate):
    def construct_query(self) -> None:
        self.query = """
            SELECT * FROM Sales WHERE new='true'
        """
        self.header = ["salesperson", "amt", "year", "model", "new"]

class SalesGrossQuery(QueryTemplate):
    def construct_query(self) -> None:
        self.query = """
            SELECT salesperson, sum(amt) FROM Sales GROUP BY salesperson
        """
        self.header = ["salesperson", "total sales"]

    def output_context(self) -> ContextManager[TextIO]:
        today = datetime.date.today()
        filepath = Path(f"gross_sales_{today:%Y%m%d}.csv")
        self.target_file = filepath.open("w")
        return self.target_file
```

---

### Sección 12.7: Repaso

Puntos clave tratados en este capítulo:

- **Adapter:** Convierte la interfaz de una clase existente en la interfaz esperada por el cliente sin modificar el código original.
- **Facade:** Simplifica el acceso a subsistemas complejos agrupando operaciones en una interfaz clara.
- **Flyweight & `__slots__`:** Minimizan el uso de memoria compartiendo estados o eliminando el diccionario interno `__dict__`.
- **Abstract Factory:** Crea familias de objetos relacionados garantizando su interoperabilidad.
- **Composite:** Trata de forma idéntica a elementos individuales y a composiciones recursivas en árbol.
- **Template Method:** Define el flujo maestro de un algoritmo permitiendo a las subclases redefinir pasos individuales.

---

### Sección 12.8: Ejercicios

1. **Persistencia real en Composite:** Extiende `Folder` y `File` para que las operaciones `move()`, `copy()` y `remove()` reflejen los cambios reales en el disco usando `pathlib` y `shutil`.
2. **Fachada de múltiples formatos:** Modifica `GenerateImages` para admitir la exportación a formatos rasterizados (PNG) y vectoriales (SVG/LaTeX).
3. **Optimización con `__slots__` en Dataclasses:** Compara el uso de memoria y rendimiento en una colección de un millón de registros utilizando `@dataclass` con y sin `slots=True`.

---

### Sección 12.9: Resumen

En este capítulo exploramos patrones avanzados de diseño estructurales y de comportamiento en Python.

En el próximo capítulo, abordaremos un pilar fundamental para garantizar la calidad del software orientado a objetos: las **pruebas unitarias y suites de testing** con `unittest`, `pytest` y `doctest`.
