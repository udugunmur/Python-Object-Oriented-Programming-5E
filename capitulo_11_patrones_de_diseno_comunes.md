# Parte 3: Patrones de Diseño y Buenas Prácticas
## Capítulo 11: Patrones de Diseño Comunes

En el capítulo anterior introdujimos el concepto de patrón de diseño y analizamos el patrón iterador, una estructura tan útil y común que se encuentra integrada en el núcleo del propio lenguaje. En este capítulo revisaremos otros patrones comunes y cómo se implementan en Python.

Al igual que con la iteración, Python suele ofrecer una sintaxis alternativa que simplifica la resolución de estos problemas. Nos centraremos en las implementaciones idiomáticas de Python para los siguientes patrones:

- El patrón **Decorator** (Decorador)
- El patrón **Observer** (Observador)
- El patrón **Strategy** (Estrategia)
- El patrón **Command** (Comando)
- El patrón **State** (Estado)
- El patrón **Singleton** (Instancia única)

---

### Sección 11.1: El patrón Decorator

El patrón **Decorator** permite envolver un objeto que proporciona una funcionalidad central con otros objetos que alteran o extienden dicha funcionalidad. Cualquier objeto que interactúe con el componente decorado lo hará exactamente de la misma manera que si no estuviera decorado, ya que la interfaz del decorador es idéntica a la del objeto núcleo.

Usos principales:
1. Mejorar o transformar la respuesta de un componente antes de enviarla a otro.
2. Soportar múltiples comportamientos opcionales mediante **composición** en lugar de herencia múltiple.

> **Figura 11.1: Patrón Decorator en UML**  
> `Interface` ◁─── `Core`  
> `Interface` ◁─── `Decorator` (contiene una referencia por composición a `Interface`) ──▷ delega en `Core` o en otro `Decorator`.

#### 11.1.1 Ejemplo de Decorator: Servidor de sockets y procesamiento de datos

Implementemos un servidor TCP que responde a tiradas de dados mediante sockets:

`socket_server.py`:

```python
import contextlib
import socket

def main_1() -> None:
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(("localhost", 2401))
    server.listen(1)
    with contextlib.closing(server):
        while True:
            client, addr = server.accept()
            dice_response(client)
```

Función de respuesta del servidor:

```python
def dice_response(client: socket.socket) -> None:
    request = client.recv(1024)
    try:
        # Future: response = dice.dice_roller(request)
        response = dice_roller_ex(request)
    except (ValueError, KeyError) as ex:
        response = repr(ex).encode("utf-8")
    client.send(response)
```

Lógica básica de simulación de dados:

```python
import random

def dice_roller_ex(request: bytes) -> bytes:
    request_text = request.decode("utf-8")
    numbers = [random.randint(1, 6) for _ in range(6)]
    response = f"{request_text} = {numbers}"
    return response.encode("utf-8")
```

Cliente interactivo `socket_client.py`:

```python
import socket

def main() -> None:
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.connect(("localhost", 2401))
    count = input("How many rolls: ") or "1"
    pattern = input("Dice pattern nd6[dk+-]a: ") or "d6"
    command = f"Dice {count} {pattern}"
    server.send(command.encode("utf8"))
    response = server.recv(1024)
    print(response.decode("utf-8"))
    server.close()
```

#### Envoltorio decorador para el socket (`LogSocket`)

```python
class LogSocket:
    def __init__(self, socket: socket.socket) -> None:
        self.socket = socket

    def recv(self, count: int = 0) -> bytes:
        data = self.socket.recv(count)
        print(f"Receiving {data!r} from {self.socket.getpeername()[0]}")
        return data

    def send(self, data: bytes) -> None:
        print(f"Sending {data!r} to {self.socket.getpeername()[0]}")
        self.socket.send(data)

    def close(self) -> None:
        self.socket.close()
```

Uso del socket decorado en el bucle del servidor:

```python
from typing import cast

def main_2() -> None:
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(("localhost", 2401))
    server.listen(1)
    with contextlib.closing(server):
        while True:
            client, addr = server.accept()
            logging_socket = cast(socket.socket, LogSocket(client))  # New
            dice_response(logging_socket)
            client.close()
```

#### Decoración a nivel de función de transformación de bytes

Decorador para registro (*logging*):

```python
from collections.abc import Callable

type Address = tuple[str, int]

class LogRoller:
    def __init__(self, dice: Callable[[bytes], bytes], remote_addr: Address) -> None:
        self.dice_roller = dice
        self.remote_addr = remote_addr

    def __call__(self, request: bytes) -> bytes:
        print(f"Receiving {request!r} from {self.remote_addr}")
        dice_roller = self.dice_roller
        response = dice_roller(request)
        print(f"Sending {response!r} to {self.remote_addr}")
        return response
```

Decorador para compresión GZip:

```python
import gzip
import io

class ZipRoller:
    def __init__(self, dice: Callable[[bytes], bytes]) -> None:
        self.dice_roller = dice

    def __call__(self, request: bytes) -> bytes:
        dice_roller = self.dice_roller
        response = dice_roller(request)
        buffer = io.BytesIO()
        with gzip.GzipFile(fileobj=buffer, mode="w") as zipfile:
            zipfile.write(response)
        return buffer.getvalue()
```

Composición encadenada de múltiples decoradores:

```python
def dice_response(client: socket.socket) -> None:
    request = client.recv(1024)
    try:
        remote_addr = client.getpeername()
        roller_1 = ZipRoller(dice.dice_roller)
        roller_2 = LogRoller(roller_1, remote_addr=remote_addr)
        response = roller_2(request)
    except (ValueError, KeyError) as ex:
        response = repr(ex).encode("utf-8")
    client.send(response)
```

#### 11.1.2 Decoradores nativos de Python (`@syntax`)

Python incorpora soporte sintáctico de primera clase para decorar funciones y clases mediante `@`:

```python
from functools import wraps
from typing import Any, Callable

def log_args(function: Callable[..., Any]) -> Callable[..., Any]:
    @wraps(function)
    def wrapped_function(*args: Any, **kwargs: Any) -> Any:
        print(f"Calling {function.__name__}(*{args}, **{kwargs})")
        result = function(*args, **kwargs)
        return result
    return wrapped_function
```

Aplicación con sintaxis `@`:

```python
@log_args
def test1(a: int, b: int, c: int) -> float:
    return sum(range(a, b + 1)) / c
```

```python
>>> test1(1, 9, 2)
Calling test1(*(1, 9, 2), **{})
22.5
```

Uso de decoradores con parámetros de la biblioteca estándar (`@lru_cache`):

```python
from math import factorial
from functools import lru_cache

@lru_cache(64)
def binom(n: int, k: int) -> int:
    return factorial(n) // (factorial(k) * factorial(n - k))
```

Decorador configurable basado en clases:

```python
import logging
import time

class NamedLogger:
    def __init__(self, logger_name: str) -> None:
        self.logger = logging.getLogger(logger_name)

    def __call__(self, function: Callable[..., Any]) -> Callable[..., Any]:
        @wraps(function)
        def wrapped_function(*args: Any, **kwargs: Any) -> Any:
            start = time.perf_counter()
            try:
                result = function(*args, **kwargs)
                s = (time.perf_counter() - start) * 1_000_000
                self.logger.info(f"{function.__name__}, {s:.1f}s")
                return result
            except Exception as ex:
                s = (time.perf_counter() - start) * 1_000_000
                self.logger.error(f"{ex}, {function.__name__}, {s:.1f}s")
                raise
        return wrapped_function
```

---

### Sección 11.2: El patrón Observer

El patrón **Observer** permite que un objeto central (*Observable* o *Subject*) notifique automáticamente cualquier cambio de estado a una lista dinámica de objetos observadores (*Observers*), desacoplando completamente el modelo del evento de las acciones derivadas.

> **Figura 11.3: Patrón Observer en UML**  
> `Observable` (`attach()`, `detach()`, `_notify_observers()`) ──▷ contiene lista de `Observer` (`__call__()`).

#### 11.2.1 Ejemplo de Observer: Auditoría de jugadas en el juego Zonk

Definición de protocolos base:

```python
from typing import Protocol

class Observer(Protocol):
    def __call__(self) -> None:
        ...

class Observable:
    def __init__(self) -> None:
        self._observers: list[Observer] = []

    def attach(self, observer: Observer) -> None:
        self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    def _notify_observers(self) -> None:
        for observer in self._observers:
            observer()
```

Sujeto observable del dominio:

```python
type Hand = list[int]

class ZonkHandHistory(Observable):
    def __init__(self, player: str, dice_set: Dice) -> None:
        super().__init__()
        self.player = player
        self.dice_set = dice_set
        self.rolls: list[Hand]

    def start(self) -> Hand:
        self.dice_set.roll()
        self.rolls = [self.dice_set.dice]
        self._notify_observers()  # Notificación de cambio de estado
        return self.dice_set.dice

    def roll(self) -> Hand:
        self.dice_set.roll()
        self.rolls.append(self.dice_set.dice)
        self._notify_observers()  # Notificación de cambio de estado
        return self.dice_set.dice
```

Observador para serialización y persistencia:

```python
import json
import time

class SaveZonkHand(Observer):
    def __init__(self, hand: ZonkHandHistory) -> None:
        self.hand = hand
        self.count = 0

    def __call__(self) -> None:
        self.count += 1
        message = {
            "player": self.hand.player,
            "sequence": self.count,
            "hands": json.dumps(self.hand.rolls),
            "time": time.time(),
        }
        print(f"SaveZonkHand {message}")
```

Observador para reglas de juego específicas (tres parejas):

```python
class ThreePairZonkHand:
    """Observer of ZonkHandHistory"""
    def __init__(self, hand: ZonkHandHistory) -> None:
        self.hand = hand
        self.zonked = False

    def __call__(self) -> None:
        last_roll = self.hand.rolls[-1]
        distinct_values = set(last_roll)
        self.zonked = len(distinct_values) == 3 and all(
            last_roll.count(v) == 2 for v in distinct_values
        )
        if self.zonked:
            print("3 Pair Zonk!")
```

---

### Sección 11.3: El patrón Strategy

El patrón **Strategy** desacopla un algoritmo de su contexto de ejecución encapsulándolo en un objeto independiente. Permite intercambiar estrategias de cómputo en tiempo de ejecución sin modificar el cliente.

> **Figura 11.4: Patrón Strategy en UML**  
> `Context / Core` (contiene referencia a `Strategy`) ──▷ `Strategy Interface` ◁─── `ConcreteStrategyA`, `ConcreteStrategyB`.

#### 11.3.1 Ejemplo de Strategy: Redimensionamiento y ajuste de fondos de pantalla

Definición de la abstracción base:

```python
import abc
from pathlib import Path
import PIL.Image as image_module
from PIL.Image import Image

type Size = tuple[int, int]

class FillAlgorithm(abc.ABC):
    @abc.abstractmethod
    def make_background(self, img_file: Path, desktop_size: Size) -> Image:
        pass
```

Estrategias concretas:

```python
class TiledStrategy(FillAlgorithm):
    def make_background(self, img_file: Path, desktop_size: Size) -> Image:
        in_img = image_module.open(img_file)
        out_img = image_module.new("RGB", desktop_size)
        num_tiles = [o // i + 1 for o, i in zip(out_img.size, in_img.size)]
        for x in range(num_tiles[0]):
            for y in range(num_tiles[1]):
                out_img.paste(
                    in_img,
                    (
                        in_img.size[0] * x,
                        in_img.size[1] * y,
                        in_img.size[0] * (x + 1),
                        in_img.size[1] * (y + 1),
                    ),
                )
        return out_img

class CenteredStrategy(FillAlgorithm):
    def make_background(self, img_file: Path, desktop_size: Size) -> Image:
        in_img = image_module.open(img_file)
        out_img = image_module.new("RGB", desktop_size)
        left = (out_img.size[0] - in_img.size[0]) // 2
        top = (out_img.size[1] - in_img.size[1]) // 2
        out_img.paste(
            in_img,
            (left, top, left + in_img.size[0], top + in_img.size[1]),
        )
        return out_img

class ScaledStrategy(FillAlgorithm):
    def make_background(self, img_file: Path, desktop_size: Size) -> Image:
        in_img = image_module.open(img_file)
        out_img = in_img.resize(desktop_size)
        return out_img
```

Contexto cliente (`Resizer`):

```python
class Resizer:
    def __init__(self, algorithm: FillAlgorithm) -> None:
        self.algorithm = algorithm

    def resize(self, image_file: Path, size: Size) -> Image:
        result = self.algorithm.make_background(image_file, size)
        return result
```

#### 11.3.2 Strategy funcional en Python

Dado que en Python las funciones son objetos de primera clase, podemos reemplazar las clases de estrategia por funciones puras con firmas homogéneas `Callable[[Path, Size], Image]`:

```python
type FillAlgorithm_T = TiledStrategy | CenteredStrategy | ScaledStrategy
```

---

### Sección 11.4: El patrón Command

El patrón **Command** encapsula una solicitud o acción como un objeto independiente, permitiendo parametrizar clientes con diferentes operaciones, encolar o registrar solicitudes y soportar operaciones reversibles (*undo/redo*).

> **Figura 11.5: Patrón Command en UML**  
> `Invoker / Core` ──▷ lista de `Command` (`apply()` / `execute()`) ──▷ interactúa con `Receiver`.

#### 11.4.1 Ejemplo de Command: Intérprete y evaluador de tiradas de dados complejas

Expresión regular con grupos nombrados para analizar sintaxis de dados (`3d6k2+5`):

```python
import re

dice_pattern = re.compile(r"(?P<n>\d*)d(?P<d>\d+)(?P<a>(?:[dk+-]\d+)*)")
```

> **Figura 11.6: Diagrama de estados para el analizador sintáctico de dados**  
> `(?P<n>\d*)` ──▷ `"d"` ──▷ `(?P<d>\d+)` ──▷ `(?P<a>(?:[dk+-]\d+)*)`

Jerarquía de comandos de ajuste:

```python
import abc
import random

class Adjustment(abc.ABC):
    def __init__(self, amount: int) -> None:
        self.amount = amount

    @abc.abstractmethod
    def apply(self, dice: "Dice") -> None:
        ...

class Roll(Adjustment):
    def __init__(self, n: int, d: int) -> None:
        self.n = n
        self.d = d

    def apply(self, dice: "Dice") -> None:
        dice.dice = sorted(random.randint(1, self.d) for _ in range(self.n))
        dice.modifier = 0

class Drop(Adjustment):
    def apply(self, dice: "Dice") -> None:
        dice.dice = dice.dice[self.amount :]

class Keep(Adjustment):
    def apply(self, dice: "Dice") -> None:
        dice.dice = dice.dice[: self.amount]

class Plus(Adjustment):
    def apply(self, dice: "Dice") -> None:
        dice.modifier += self.amount

class Minus(Adjustment):
    def apply(self, dice: "Dice") -> None:
        dice.modifier -= self.amount
```

Clase contenedora y ejecutora de comandos (`Dice`):

```python
class Dice:
    def __init__(self, n: int, d: int, *adj: Adjustment) -> None:
        self.adjustments = [Roll(n, d)] + list(adj)
        self.dice: list[int]
        self.modifier: int

    def roll(self) -> int:
        for a in self.adjustments:
            a.apply(self)
        return sum(self.dice) + self.modifier

    @classmethod
    def from_text(cls, dice_text: str) -> "Dice":
        dice_pattern = re.compile(r"(?P<n>\d*)d(?P<d>\d+)(?P<a>(?:[dk+-]\d+)*)")
        adjustment_pattern = re.compile(r"([dk+-])(\d+)")
        adj_class: dict[str, type[Adjustment]] = {
            "d": Drop,
            "k": Keep,
            "+": Plus,
            "-": Minus,
        }
        if (dice_match := dice_pattern.match(dice_text)) is None:
            raise ValueError(f"Error in {dice_text!r}")
        n = int(dice_match.group("n")) if dice_match.group("n") else 1
        d = int(dice_match.group("d"))
        adjustment_matches = adjustment_pattern.finditer(dice_match.group("a") or "")
        adjustments = [
            adj_class[a.group(1)](int(a.group(2))) for a in adjustment_matches
        ]
        return cls(n, d, *adjustments)
```

---

### Sección 11.5: El patrón State

El patrón **State** permite que un objeto altere su comportamiento cuando su estado interno cambia, aparentando cambiar de clase. Es idóneo para modelar autómatas finitos y analizadores sintácticos paso a paso.

> **Figura 11.7: Patrón State en UML**  
> `Context` (`feed_byte()`) ──▷ delega en `CurrentState` ──▷ retorna nuevo `NextState`.

#### 11.5.1 Ejemplo de State: Analizador sintáctico de sentencias GPS NMEA 0183

Estructura de una trama GPS NMEA: `$GPGLL,3723.2475,N,12158.3416,W,161229.487,A,A*41`

| Campo | Descripción |
| :--- | :--- |
| `$` | Inicio de sentencia |
| `GPGLL` | Identificador del emisor (`GP`) y tipo de mensaje (`GLL`) |
| `3723.2475` | Latitud (37°23.2475) |
| `N` | Hemisferio Norte |
| `12158.3416` | Longitud (121°58.3416) |
| `W` | Hemisferio Oeste |
| `161229.487` | Marca temporal UTC (16:12:29.487) |
| `A` | Estado de validez (`A` = válido, `V` = no válido) |
| `A` | Modo de operación (`A` = autónomo) |
| `*` | Fin de cuerpo e inicio de suma de verificación |
| `41` | Suma de verificación hexadecimal (XOR bit a bit) |

*Tabla 11.1: Estructura de sentencia NMEA*

> **Figura 11.8: Transiciones de estado para procesar tramas NMEA**  
> `Waiting` $\xrightarrow{\$}$ `Header` $\xrightarrow{\text{5 bytes}}$ `Body` $\xrightarrow{*}$ `Checksum` $\xrightarrow{\text{2 bytes}}$ `End`

Contenedor de mensaje acumulativo:

```python
class Message:
    def __init__(self) -> None:
        self.body = bytearray(80)
        self.checksum_source = bytearray(2)
        self.body_len = 0
        self.checksum_len = 0
        self.checksum_computed = 0

    def reset(self) -> None:
        self.body_len = 0
        self.checksum_len = 0
        self.checksum_computed = 0

    def body_append(self, input: int) -> int:
        self.body[self.body_len] = input
        self.body_len += 1
        self.checksum_computed ^= input
        return self.body_len

    def checksum_append(self, input: int) -> int:
        self.checksum_source[self.checksum_len] = input
        self.checksum_len += 1
        return self.checksum_len

    @property
    def valid(self) -> bool:
        return (
            self.checksum_len == 2
            and int(self.checksum_source, 16) == self.checksum_computed
        )
```

Estados discretos:

```python
class NMEA_State:
    def __init__(self, message: "Message") -> None:
        self.message = message

    def feed_byte(self, input: int) -> "NMEA_State":
        return self

    def valid(self) -> bool:
        return False

    def __repr__(self) -> str:
        return f"{self.__class__.__name__}({self.message})"

class Waiting(NMEA_State):
    def feed_byte(self, input: int) -> NMEA_State:
        if input == ord(b"$"):
            return Header(self.message)
        return self

class Header(NMEA_State):
    def __init__(self, message: "Message") -> None:
        self.message = message
        self.message.reset()

    def feed_byte(self, input: int) -> NMEA_State:
        if input == ord(b"$"):
            return Header(self.message)
        size = self.message.body_append(input)
        if size == 5:
            return Body(self.message)
        return self

class Body(NMEA_State):
    def feed_byte(self, input: int) -> NMEA_State:
        if input == ord(b"$"):
            return Header(self.message)
        if input == ord(b"*"):
            return Checksum(self.message)
        self.message.body_append(input)
        return self

class Checksum(NMEA_State):
    def feed_byte(self, input: int) -> NMEA_State:
        if input == ord(b"$"):
            return Header(self.message)
        if input in {ord(b"\n"), ord(b"\r")}:
            return End(self.message)
        size = self.message.checksum_append(input)
        if size == 2:
            return End(self.message)
        return self

class End(NMEA_State):
    def feed_byte(self, input: int) -> NMEA_State:
        if input == ord(b"$"):
            return Header(self.message)
        elif input not in {ord(b"\n"), ord(b"\r")}:
            return Waiting(self.message)
        return self

    def valid(self) -> bool:
        return self.message.valid
```

Lector de flujo con transición de estados:

```python
from typing import Iterable, Iterator, cast

class Reader:
    def __init__(self) -> None:
        self.buffer = Message()
        self.state: NMEA_State = Waiting(self.buffer)

    def read(self, source: Iterable[bytes]) -> Iterator[Message]:
        for byte in source:
            self.state = self.state.feed_byte(cast(int, byte))
            if self.buffer.valid:
                yield self.buffer
                self.buffer = Message()
                self.state = Waiting(self.buffer)
```

#### 11.5.2 Comparativa: State frente a Strategy

| Característica | Patrón Strategy | Patrón State |
| :--- | :--- | :--- |
| **Propósito** | Elegir un algoritmo intercambiable de forma estática o al inicio. | Representar transiciones dinámicas entre fases a lo largo del tiempo. |
| **Conocimiento entre variantes** | Las estrategias son independientes y no se conocen entre sí. | Los estados conocen a otros estados a los que pueden transicionar. |
| **Frecuencia de cambio** | Baja (se suele inyectar una sola vez al instanciar). | Alta (cambia continuamente con cada evento de entrada). |

---

### Sección 11.6: El patrón Singleton

El patrón **Singleton** restringe la instanciación de una clase a una única instancia global en todo el sistema.

> [!WARNING]
> En Python, implementar Singletons manuales con constructores forzados suele considerarse un **antipatrón**. Las alternativas idiomáticas son los **módulos** (que se cargan e importan exactamente una vez de forma *thread-safe*) y las variables a nivel de módulo.

Implementación tradicional mediante `__new__()`:

```python
from typing import Any

class OneOnly:
    _singleton = None

    def __new__(cls, *args: Any, **kwargs: Any) -> "OneOnly":
        if not cls._singleton:
            cls._singleton = super().__new__(cls, *args, **kwargs)
        return cls._singleton
```

```python
>>> o1 = OneOnly()
>>> o2 = OneOnly()
>>> o1 is o2
True
```

#### Refactorización de Estados NMEA como Singletons sin asignación de memoria

Para entornos embebidos restringidos (IoT), podemos evitar instanciar nuevos objetos de estado en cada transición utilizando métodos estáticos `@staticmethod` y referencias a las propias clases:

```python
import abc

class NMEA_State(abc.ABC):
    @staticmethod
    def enter(message: "Message") -> None:
        pass

    @staticmethod
    @abc.abstractmethod
    def feed_byte(message: "Message", input: int) -> "type[NMEA_State]":
        ...

    @staticmethod
    def valid(message: "Message") -> bool:
        return False
```

```python
class Waiting(NMEA_State):
    @staticmethod
    def feed_byte(message: "Message", input: int) -> type[NMEA_State]:
        if input == ord(b"$"):
            return Header
        return Waiting

class Header(NMEA_State):
    @staticmethod
    def enter(message: "Message") -> None:
        message.reset()

    @staticmethod
    def feed_byte(message: "Message", input: int) -> type[NMEA_State]:
        if input == ord(b"$"):
            return Header
        size = message.body_append(input)
        if size == 5:
            return Body
        return Header

class Body(NMEA_State):
    @staticmethod
    def feed_byte(message: "Message", input: int) -> type[NMEA_State]:
        if input == ord(b"$"):
            return Header
        if input == ord(b"*"):
            return Checksum
        message.body_append(input)
        return Body

class Checksum(NMEA_State):
    @staticmethod
    def feed_byte(message: "Message", input: int) -> type[NMEA_State]:
        if input == ord(b"$"):
            return Header
        if input in {ord(b"\n"), ord(b"\r")}:
            return End
        size = message.checksum_append(input)
        if size == 2:
            return End
        return Checksum

class End(NMEA_State):
    @staticmethod
    def feed_byte(message: "Message", input: int) -> type[NMEA_State]:
        if input == ord(b"$"):
            return Header
        elif input not in {ord(b"\n"), ord(b"\r")}:
            return Waiting
        return End

    @staticmethod
    def valid(message: "Message") -> bool:
        return message.valid
```

Lector optimizado reutilizando clases como estados Singleton:

```python
from typing import Iterable, Iterator, cast

class Reader:
    def __init__(self) -> None:
        self.buffer = Message()
        self.state: type[NMEA_State] = Waiting

    def read(self, source: Iterable[bytes]) -> Iterator[Message]:
        for byte in source:
            new_state = self.state.feed_byte(self.buffer, cast(int, byte))
            if self.buffer.valid:
                yield self.buffer
                self.buffer = Message()
                new_state = Waiting
            if new_state != self.state:
                new_state.enter(self.buffer)
            self.state = new_state
```

---

### Sección 11.7: Repaso

Puntos clave tratados en este capítulo:

- **Decorator:** Envoltorios de composición que extienden funcionalidades de forma transparente; en Python se expresan elegantemente mediante `@decorador`.
- **Observer:** Desacopla emisores de eventos y múltiples receptores registrados (`attach`/`detach`).
- **Strategy:** Encapsula algoritmos alternativos intercambiables en tiempo de ejecución.
- **Command:** Encapsula solicitudes de acción como objetos con métodos `apply()` o `execute()`.
- **State:** Modela máquinas de estados finitos donde el comportamiento muta conforme cambian las fases.
- **Singleton:** Garantiza una única instancia; en Python se prefieren módulos y variables de módulo a nivel global.

---

### Sección 11.8: Ejercicios

1. **Ampliación del parser de comandos de dados:** Añade soporte para operadores de explosión de dados (`!`) y repetición (`r`) según la especificación de Roll20.
2. **Observadores asíncronos:** Implementa un observador que envíe los registros de partidas a un servicio web simulado sin bloquear el hilo principal de ejecución.
3. **Métricas de rendimiento en State:** Compara el consumo de memoria y la velocidad de procesamiento entre el analizador de estados con instancias dinámicas y el analizador con clases Singleton.

---

### Sección 11.9: Resumen

En este capítulo exploramos las implementaciones idiomáticas en Python de los patrones Decorator, Observer, Strategy, Command, State y Singleton.

En el próximo capítulo, completaremos nuestro estudio de patrones con **patrones de diseño avanzados** como Adapter, Facade, Flyweight y Abstract Factory.
