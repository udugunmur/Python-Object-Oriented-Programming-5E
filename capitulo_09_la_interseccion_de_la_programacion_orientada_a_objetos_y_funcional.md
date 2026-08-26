# Parte 2: Diseño y Tipado Avanzado
## Capítulo 9: La Intersección de la Programación Orientada a Objetos y Funcional

Existen muchos aspectos de Python que se asemejan más a la **programación funcional** que a la programación orientada a objetos. Aunque la POO es el eje central de este libro, existen casos de uso convincentes para las técnicas de diseño funcional. En Python, la implementación subyacente es orientada a objetos; sin embargo, adoptar ciertas técnicas funcionales hace que el código sea mucho más conciso y expresivo.

En este capítulo cubriremos una selección de características no estrictamente limitadas a la POO tradicional:

- Funciones integradas (*built-ins*) que resuelven tareas comunes en una sola llamada.
- Una alternativa idiomática a la sobrecarga de métodos.
- Las funciones tratadas como **objetos de primera clase** (*first-class citizens*).
- Objetos invocables (*callables*) y administradores de estado.

---

### Sección 9.1: Funciones integradas de Python

Python proporciona numerosas funciones globales que realizan cálculos sobre diferentes tipos de objetos abstrayendo operaciones comunes sin obligar a usar la sintaxis `objeto.metodo()`. Estas funciones aprovechan el *duck typing* delegando directamente en **métodos especiales** (*dunder methods*).

#### 9.1.1 La función `len()`

La función `len()` devuelve la cantidad de elementos en contenedores como diccionarios, conjuntos, tuplas o listas:

```python
>>> len([1, 2, 3, 4])
4
```

Una expresión como `len(myobj)` delega internamente en `MyObj.__len__(myobj)`.

¿Por qué usar una función global `len(x)` en lugar de un método `x.length()`?
1. **Rendimiento:** Evita la búsqueda dinámica de atributos en el espacio de nombres de la instancia y la resolución MRO completa; Python accede directamente a la ranura de la clase subyacente.
2. **Claridad y estética matemática:** La notación funcional `len(x)` resulta más natural y uniforme en contextos algebraicos y funcionales.

Cualquier clase que implemente la ABC `collections.abc.Sized` proporciona `__len__()` y es compatible con `len()`.

#### 9.1.2 La función `reversed()`

`reversed()` acepta cualquier secuencia e itera sobre sus elementos en orden inverso. Delega en el método especial `__reversed__()` o, en su defecto, combina `__len__()` y `__getitem__()`:

```python
from collections.abc import Sequence, Iterator
from typing import Any

class CustomSequence(Sequence[Any]):
    def __init__(self, arg: Sequence[Any]) -> None:
        self._list = arg

    def __len__(self) -> int:
        # This doesn't seem right, does it?
        return 5

    def __getitem__(self, index: int | slice) -> Any:
        return f"x{index}"

class FunkyBackwards(list[Any]):
    def __reversed__(self) -> Iterator[Any]:
        return iter("BACKWARDS!")
```

Probando `reversed()` con tres secuencias diferentes:

```python
>>> generic = [1, 2, 3, 4, 5]
>>> custom = CustomSequence([6, 7, 8, 9, 10])
>>> funkadelic = FunkyBackwards([11, 12, 13, 14, 15])
>>> for sequence in generic, custom, funkadelic:
...     print(f"{sequence.__class__.__name__}: ", end="")
...     for item in reversed(sequence):
...         print(f"{item}, ", end="")
...     print()
... 
list: 5, 4, 3, 2, 1, 
CustomSequence: x4, x3, x2, x1, x0, 
FunkyBackwards: B, A, C, K, W, A, R, D, S, !, 
```

#### 9.1.3 La función `enumerate()`

`enumerate()` genera una secuencia de tuplas `(indice, elemento)` a partir de cualquier iterable, admitiendo un índice de inicio configurable (`start`):

```python
>>> from pathlib import Path
>>> with open(Path.cwd() / "data" / "sample_data.md") as source:
...     for index, line in enumerate(source, start=1):
...         print(f"{index:3d}: {line.rstrip()}")
... 
  1: # Python 3 Object-Oriented Programming
  2: 
  3: ## Chapter 9. The Intersection of Object-Oriented and Functional Programming
  4: 
  5: Some sample data to show how the 'enumerate()' function works.
```

Otras funciones integradas destacadas:
- `abs()`, `str()`, `repr()`, `pow()`, `divmod()`, `bytes()`, `format()`, `hash()`, `bool()`: Mapean directamente a sus respectivos métodos mágicos (`__abs__`, `__str__`, etc.).
- `all()`, `any()`: Reducciones booleanas sobre iterables.
- `zip()`: Agrupa elementos paralelos de múltiples secuencias.
- `getattr()`, `setattr()`, `hasattr()`, `delattr()`: Introspección y acceso dinámico a atributos.

---

### Sección 9.2: Una alternativa a la sobrecarga de métodos

En lenguajes estáticos tradicionales, la sobrecarga de métodos implica definir múltiples métodos con el mismo nombre y firmas diferentes.

En Python se adopta un enfoque mucho más flexible: **un único método** puede recibir parámetros obligatorios, opcionales con valor por defecto, posicionales y por palabra clave.

Parámetros posicionales obligatorios:

```python
from typing import Any

def mandatory_params(x: Any, y: Any, z: Any) -> str:
    return f"{x=}, {y=}, {z=}"
```

```python
>>> a_variable = 42
>>> mandatory_params("a string", a_variable, True)
"x='a string', y=42, z=True"
```

#### 9.2.1 Valores por defecto para parámetros

```python
def latitude_dms(
    deg: float,
    min: float,
    sec: float = 0.0,
    dir: str | None = None
) -> str:
    if dir is None:
        dir = "N"
    return f"{deg:02.0f} {min+sec/60:05.3f}{dir}"
```

Invocaciones válidas:

```python
>>> latitude_dms(36, 51, 2.9, "N")
'36 51.048N'
>>> latitude_dms(38, 58, dir="N")
'38 58.000N'
>>> latitude_dms(38, 19, dir="N", sec=7)
'38 19.117N'
```

#### Parámetros exclusivamente por palabra clave (`*`)

El asterisco en solitario `*` delimita parámetros que deben pasarse obligatoriamente con nombre:

```python
def kw_only(x: Any, y: str = "defaultkw", *, a: bool, b: str = "only") -> str:
    return f"{x=}, {y=}, {a=}, {b=}"
```

```python
>>> kw_only('x', a='a', b='b')
"x='x', y='defaultkw', a='a', b='b'"
```

#### Parámetros exclusivamente posicionales (`/`)

La barra inclinada `/` delimita parámetros que solo pueden pasarse por posición (PEP 570):

```python
>>> pos_only(2, "three")
"x=2, y='three', z=None"
```

#### Trampa de los valores por defecto mutables

> [!CAUTION]
> Los valores por defecto de las funciones en Python se evalúan **una sola vez en tiempo de definición**. NUNCA uses estructuras mutables (`list`, `dict`, `set`) como valores por defecto.

Diseño erróneo:

```python
def bad_default(tag: str, history: list[str] = []) -> list[str]:
    """A Very Bad Design (VBD)."""
    history.append(tag)
    return history
```

```python
>>> h = bad_default("tag1")
>>> h2 = bad_default("tag21")
>>> h is h2
True
```

Diseño correcto usando `None`:

```python
from typing import Optional

def good_default(tag: str, history: Optional[list[str]] = None) -> list[str]:
    history = [] if history is None else history
    history.append(tag)
    return history
```

#### 9.2.2 Listas de argumentos variables (`*args`, `**kwargs`)

Recepción de argumentos posicionales arbitrarios (`*links`):

```python
from urllib.parse import urlparse
from pathlib import Path

def get_pages(*links: str) -> None:
    for link in links:
        url = urlparse(link)
        name = "index.html" if url.path in ("", "/") else url.path
        target = Path(url.netloc.replace(".", "_")) / name
        print(f"Create {target} from {link!r}")
        # etc.
```

```python
>>> get_pages('https://www.archlinux.org', 'https://dusty.phillips.codes')
Create www_archlinux_org...index.html from 'https://www.archlinux.org'
Create dusty_phillips_codes...index.html from 'https://dusty.phillips.codes'
```

Recepción de argumentos con nombre arbitrarios (`**kwargs`):

```python
from typing import Any

class Options(dict[str, Any]):
    default_options: dict[str, Any] = {
        "port": 21,
        "host": "localhost",
        "username": None,
        "password": None,
        "debug": False,
    }

    def __init__(self, **kwargs: Any) -> None:
        super().__init__({**self.default_options, **kwargs})
```

```python
>>> options = Options(username="dusty", password="Hunter2", debug=True)
>>> options['debug']
True
>>> options['port']
21
```

Ejemplo combinando todos los modos de parámetros:

```python
import contextlib
from typing import TextIO, Any
from pathlib import Path

def doctest_everything(
    output: TextIO,
    *directories: Path,
    verbose: bool = False,
    **stems: str,
) -> None:
    if verbose:
        log = print
    else:
        def log(*args: Any, **kwars: Any) -> None:
            pass

    with contextlib.redirect_stdout(output):
        for directory in directories:
            log(f"Searching {directory}")
            for dirpath, dirnames, filenames in directory.walk():
                remove_excluded(dirnames)
                for name in filenames:
                    if not name.endswith('.py'):
                        continue
                    path = (dirpath / name).relative_to(Path.cwd())
                    log(
                        f"File {path}, "
                        f"{path.stem=}"
                    )
                    options = stems.get(path.stem, "")
                    if options.upper() == "SKIP":
                        log("Skipped")
                        continue
                    doctest_opts = (options.upper().split(",") if options else [])
                    r = run_test(path, doctest_opts)
                    if r.returncode:
                        log(r.stderr)
```

#### 9.2.3 Desempaquetado de argumentos (`*` y `**`)

```python
def show_args(arg1: Any, arg2: Any, arg3: Any="THREE") -> str:
    return f"{arg1=}, {arg2=}, {arg3=}"
```

Desempaquetando tuplas/listas (`*`) y diccionarios (`**`):

```python
>>> some_args = range(3)
>>> show_args(*some_args)
'arg1=0, arg2=1, arg3=2'

>>> more_args = {"arg1": "ONE", "arg2": "TWO"}
>>> show_args(**more_args)
"arg1='ONE', arg2='TWO', arg3='THREE'"
```

Fusión de diccionarios con `{**x, **y}`:

```python
>>> x = {'a': 1, 'b': 2}
>>> y = {'b': 11, 'c': 3}
>>> z = {**x, **y}
>>> z
{'a': 1, 'b': 11, 'c': 3}
```

---

### Sección 9.3: Las funciones también son objetos

En Python, las funciones son **objetos de primera clase**: pueden pasarse como argumentos, asignarse a variables, almacenarse en colecciones y devolverse desde otras funciones (*funciones de orden superior*).

```python
from typing import Callable

def fizz(x: int) -> bool:
    return x % 3 == 0

def buzz(x: int) -> bool:
    return x % 5 == 0

def name_or_number(
    number: int,
    *tests: Callable[[int], bool]
) -> str:
    for t in tests:
        if t(number):
            return t.__name__
    return str(number)
```

```python
>>> for i in range(1, 11):
...     print(name_or_number(i, fizz, buzz))
... 
1
2
fizz
4
buzz
fizz
7
8
fizz
buzz
```

#### 9.3.1 Objetos función y callbacks (Programación dirigida por eventos)

Implementación de un temporizador/planificador (*scheduler*) basado en cola de prioridad con `heapq`:

```python
from collections.abc import Callable
from dataclasses import dataclass, field
import heapq
import time

type Callback = Callable[[int], None]

@dataclass(frozen=True, order=True)
class Task:
    scheduled: int
    callback: Callback = field(compare=False)
    delay: int = field(default=0, compare=False)
    limit: int = field(default=1, compare=False)

    def repeat(self, current_time: int) -> "Task | None":
        if self.delay > 0 and self.limit > 2:
            return Task(
                current_time + self.delay,
                self.callback,
                self.delay,
                self.limit - 1,
            )
        elif self.delay > 0 and self.limit == 2:
            return Task(
                current_time + self.delay,
                self.callback,
            )
        else:
            return None

class Scheduler:
    def __init__(self) -> None:
        self.tasks: list[Task] = []

    def enter(
        self,
        after: int,
        task: Callback,
        delay: int = 0,
        limit: int = 1,
    ) -> None:
        new_task = Task(after, task, delay, limit)
        heapq.heappush(self.tasks, new_task)

    def run(self) -> None:
        current_time = 0
        while self.tasks:
            next_task = heapq.heappop(self.tasks)
            if (delay := next_task.scheduled - current_time) > 0:
                time.sleep(delay)
            current_time = next_task.scheduled
            next_task.callback(current_time)
            if again := next_task.repeat(current_time):
                heapq.heappush(self.tasks, again)
```

Callbacks y prueba de ejecución:

```python
import datetime

def format_time(message: str) -> None:
    now = datetime.datetime.now()
    print(f"{now:%I:%M:%S}: {message}")

def one(timer: float) -> None:
    format_time("Called One")

def two(timer: float) -> None:
    format_time("Called Two")

def three(timer: float) -> None:
    format_time("Called Three")

class Repeater:
    def __init__(self) -> None:
        self.count = 0

    def four(self, timer: float) -> None:
        self.count += 1
        format_time(f"Called Four: {self.count}")

if __name__ == "__main__":
    s = Scheduler()
    s.enter(1, one)
    s.enter(2, one)
    s.enter(2, two)
    s.enter(4, two)
    s.enter(3, three)
    s.enter(6, three)
    repeater = Repeater()
    s.enter(5, repeater.four, delay=1, limit=5)
```

#### 9.3.2 Uso de funciones para parchear clases (*Monkey Patching*)

Podemos reemplazar métodos dinámicamente en tiempo de ejecución (habitual para dobles de pruebas o mocks en tests unitarios):

```python
class A:
    def show_something(self) -> None:
        print("My class is A")

def patched_show_something() -> None:
    print("My class is NOT A")
```

```python
>>> a_object = A()
>>> a_object.show_something = patched_show_something
>>> a_object.show_something()
My class is NOT A
```

> [!WARNING]
> El *monkey patching* altera el contrato del código en tiempo de ejecución. Fuera del ámbito controlado de las pruebas unitarias, suele considerarse un antipatrón.

#### 9.3.3 Objetos invocables (`__call__`)

Cualquier objeto cuya clase defina el método especial `__call__()` puede invocarse como una función estándar con estado encapsulado:

```python
class Repeater_2:
    def __init__(self) -> None:
        self.count = 0

    def __call__(self, timer: float) -> None:
        self.count += 1
        format_time(f"Called Four: {self.count}")
```

```python
s2 = Scheduler()
s2.enter(5, Repeater_2(), delay=1, limit=5)
s2.run()
```

---

### Sección 9.4: Repaso

Puntos clave tratados en este capítulo:

- Las **funciones integradas** (`len()`, `reversed()`, `enumerate()`, `zip()`) delegan en métodos especiales para proporcionar APIs uniformes.
- Python resuelve la sobrecarga de métodos mediante **valores por defecto**, parámetros posicionales (`/`), palabras clave obligatorias (`*`) y argumentos variables (`*args`, `**kwargs`).
- Los valores por defecto mutables son compartidos; deben inicializarse con `None`.
- Las **funciones son objetos**: pueden pasarse como callbacks, guardarse en colas y manipularse en tiempo de ejecución.
- Implementar `__call__()` convierte cualquier instancia de clase en un objeto invocable con estado.

---

### Sección 9.5: Ejercicios

1. **Corrección de FizzBuzz:** Modifica la función `name_or_number()` para que recopile y concatene todas las coincidencias válidas (p. ej., "fizzbuzz" para 15 y "fizzbuzzbazz" para 105).
2. **Validación de opciones con `**kwargs`:** Crea una subclase de `dict` que lance `ValueError` si se intenta añadir una clave no existente en las opciones por defecto.
3. **Objetos invocables con caché:** Implementa una clase invocable con `__call__()` que calcule términos de la sucesión de Fibonacci memorizando resultados previos en un diccionario interno.

---

### Sección 9.6: Resumen

En este capítulo exploramos cómo convergen la programación funcional y la orientada a objetos en Python a través de funciones integradas, argumentos flexibles, callbacks y objetos invocables con `__call__()`.

En el próximo capítulo, estudiaremos en profundidad uno de los patrones más esenciales de Python: el **patrón iterador** y las **funciones generadoras**.
