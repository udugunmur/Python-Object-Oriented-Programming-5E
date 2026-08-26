# Parte 1: Fundamentos de la Programación Orientada a Objetos en Python
## Capítulo 6: Clases Base Abstractas y Sobrecarga de Operadores

A menudo necesitamos distinguir entre **clases concretas**, que poseen un conjunto completo de atributos y métodos, y una **clase abstracta**, a la que le faltan ciertos detalles de implementación. Esto es paralelo a la idea filosófica de la abstracción como una forma de sintetizar complejidades. Podríamos decir que un velero y un avión comparten la relación abstracta común de ser vehículos, pero los detalles sobre cómo se desplazan son completamente distintos.

En Python, disponemos de dos enfoques principales para definir entidades similares:

1. **Tipado de pato (*Duck typing*):** Cuando dos definiciones de clase tienen los mismos atributos y métodos, las instancias de ambas clases comparten el mismo protocolo y pueden usarse indistintamente (*"Si camina como un pato, nada como un pato y grazna como un pato, llamo a esa ave un pato"*).
2. **Herencia:** Cuando dos definiciones de clase comparten aspectos comunes, una subclase puede reutilizar características de una superclase. Los detalles de implementación pueden variar, pero las clases resultan intercambiables respecto a las características comunes definidas por la superclase.

Podemos llevar la herencia un paso más allá mediante **superclases abstractas**: clases que no pueden ser instanciadas directamente por sí mismas, pero que sirven como base a través de la herencia para crear clases concretas.

> **Figura 6.1: Clase base abstracta**  
> *(Subclass1 ──▷ BaseClass [Abstract] ◁── Subclass2)*

En este capítulo, cubriremos los siguientes temas:

- Creación de clases base abstractas (ABCs).
- ABCs y comprobación estática de tipos (*type hints*).
- El módulo `collections.abc`.
- Creación de nuestras propias clases base abstractas.
- Desmitificando la magia: qué ocurre internamente en una ABC.
- Sobrecarga de operadores (*operator overloading*).
- Extensión de tipos integrados.
- Metaclases.

---

### Sección 6.1: Creación de una clase base abstracta

Imagina que estamos creando un reproductor multimedia capaz de interactuar con plugins de terceros. Es muy recomendable crear una **clase base abstracta (ABC)** para documentar formalmente la API que los plugins deben implementar. Establecer claramente el contrato entre el reproductor y el plugin es uno de los casos de uso más sólidos para las ABCs.

El módulo `abc` proporciona las herramientas necesarias. A continuación se define una clase abstracta que exige a cualquier subclase proporcionar un método concreto `play()` y un atributo `ext`:

```python
import abc

class MediaLoader(abc.ABC):
    ext: str

    @abc.abstractmethod
    def play(self) -> None:
        ...
```

La clase `abc.ABC` utiliza una **metaclase** (una clase utilizada para construir clases). La metaclase por defecto de Python es `type`, la cual no comprueba métodos abstractos al instanciar objetos. La clase `abc.ABC` extiende esta metaclase para impedir la instanciación de clases que no hayan implementado todos sus métodos abstractos.

El decorador `@abc.abstractmethod` marca los métodos abstractos, cuyo cuerpo suele escribirse literalmente como `...` (*Ellipsis*).

Al declarar métodos abstractos, la clase genera un atributo especial llamado `__abstractmethods__`:

```python
>>> MediaLoader.__abstractmethods__
frozenset({'play'})
```

Si intentamos instanciar una subclase incompleta:

```python
>>> class Wav(MediaLoader):
...     pass
... 
>>> x = Wav()
Traceback (most recent call last):
  ...
    x = Wav()
TypeError: Can't instantiate abstract class Wav without an implementation for abstract method 'play'
```

En cambio, una subclase concreta que satisface el contrato se instancia sin problemas:

```python
>>> class Ogg(MediaLoader):
...     ext = '.ogg'
...     def play(self) -> None:
...         pass
... 
>>> o = Ogg()
```

#### 6.1.1 Las ABCs del módulo `collections`

Un ejemplo exhaustivo de ABCs en la biblioteca estándar de Python se encuentra en el módulo `collections.abc`. La jerarquía comienza en la clase abstracta fundamental `Container`:

```python
>>> from collections.abc import Container
>>> Container.__abstractmethods__
frozenset({'__contains__'})
```

```python
>>> help(Container.__contains__)
Help on function __contains__ in module collections.abc:

__contains__(self, x)
```

El método especial `__contains__()` implementa el operador `in` de Python. Cualquier clase que defina `__contains__()` cumple el protocolo de `Container`:

```python
class OddIntegers:
    def __contains__(self, x: int) -> bool:
        return x % 2 != 0
```

Gracias al tipado de pato estructural de las ABCs:

```python
>>> from collections.abc import Container
>>> odd = OddIntegers()
>>> isinstance(odd, Container)
True
>>> issubclass(OddIntegers, Container)
True
```

El operador `in` delega directamente en `__contains__()`:

```python
>>> odd = OddIntegers()
>>> 1 in odd
True
>>> 2 in odd
False
>>> 3 in odd
True
```

#### 6.1.2 Clases base abstractas y protocolos

Un protocolo (`typing.Protocol`) formaliza el *duck typing*: cuando dos clases exponen el mismo conjunto de métodos, ambas satisfacen el protocolo sin requerir herencia explícita. Las ABCs, por su parte, permiten comprobaciones explícitas tanto estáticas (con `mypy`) como en tiempo de ejecución.

#### 6.1.3 El módulo `collections.abc`

El módulo `collections.abc` define la jerarquía que sustenta las colecciones estándar:

> **Figura 6.2: Abstracciones de mapeo**  
> `Sized` (`__len__`) + `Iterable` (`__iter__`) + `Container` (`__contains__`) ──▷ `Collection` ──▷ `Mapping` (`__getitem__`, ...) ──▷ `MutableMapping` ──▷ `dict`

Vamos a implementar una estructura de mapeo inmutable y ordenada llamada `Lookup` que hereda de `Mapping`:

```python
import bisect
from collections.abc import Iterator, Iterable, Mapping
from typing import Protocol, Any, overload, cast

class Comparable(Protocol):
    def __eq__(self, other: Any) -> bool: ...
    def __ne__(self, other: Any) -> bool: ...
    def __le__(self, other: Any) -> bool: ...
    def __lt__(self, other: Any) -> bool: ...
    def __ge__(self, other: Any) -> bool: ...
    def __gt__(self, other: Any) -> bool: ...

class Lookup(Mapping[Comparable, Any]):
    @overload
    def __init__(self, source: Iterable[tuple[Comparable, Any]]) -> None: ...
    @overload
    def __init__(self, source: "Mapping[Comparable, Any]") -> None: ...
    def __init__(
        self,
        source: Any = None,
    ) -> None:
        sorted_pairs: list[tuple[Comparable, Any]]
        match source:
            case Iterable() as an_iter:
                # Assume it's pairs.
                sorted_pairs = sorted(
                    cast(Iterable[tuple[Comparable, Any]], an_iter)
                )
            case Mapping() as a_map:
                sorted_pairs = sorted(a_map.items())
            case _:
                sorted_pairs = []
        self.key_list: list[Comparable] = [p[0] for p in sorted_pairs]
        self.value_list: list[Any] = [p[1] for p in sorted_pairs]

    def __len__(self) -> int:
        return len(self.key_list)

    def __iter__(self) -> Iterator[Comparable]:
        return iter(self.key_list)

    def __contains__(self, key: object) -> bool:
        index = bisect.bisect_left(
            self.key_list, cast(Comparable, key)
        )
        return key == self.key_list[index]

    def __getitem__(self, key: Comparable) -> Any:
        index = bisect.bisect_left(self.key_list, key)
        if key == self.key_list[index]:
            return self.value_list[index]
        raise KeyError(key)
```

Probando la clase `Lookup`:

```python
>>> x = Lookup(
...     [
...         ("z", "Zillah"),
...         ("a", "Amy"),
...         ("c", "Clara"),
...         ("b", "Basil"),
...     ]
... )
>>> x["c"]
'Clara'
```

Al ser inmutable, los intentos de asignación fallan de forma esperada:

```python
>>> x["m"] = "Maud"
Traceback (most recent call last):
  ...
TypeError: 'Lookup' object does not support item assignment
```

#### 6.1.4 Creación de nuestras propias clases base abstractas

Modelemos una simulación de dados poliédricos (D4, D6, D8, D12, D20):

```python
import abc
import random

class Die(abc.ABC):
    def __init__(self) -> None:
        self.face: int
        self.roll()

    @abc.abstractmethod
    def roll(self) -> None:
        ...

    def __repr__(self) -> str:
        return f"{self.face}"
```

Subclases concretas:

```python
class D4(Die):
    def roll(self) -> None:
        self.face = random.choice((1, 2, 3, 4))

class D6(Die):
    def roll(self) -> None:
        self.face = random.randint(1, 6)
```

Creemos ahora una clase abstracta `Dice` para representar conjuntos de dados con diferentes reglas de tirada:

```python
class Dice(abc.ABC):
    def __init__(self, n: int, die_class: type[Die]) -> None:
        self.dice = [die_class() for _ in range(n)]

    @abc.abstractmethod
    def roll(self) -> None:
        ...

    @property
    def total(self) -> int:
        return sum(d.face for d in self.dice)
```

Implementación simple que tira todos los dados:

```python
class SimpleDice(Dice):
    def roll(self) -> None:
        for d in self.dice:
            d.roll()
```

Implementación especializada para juegos con relanzamiento selectivo (como el *Yacht*):

```python
from collections.abc import Iterable

class YachtDice(Dice):
    def __init__(self) -> None:
        super().__init__(5, D6)
        self.saved: set[int] = set()

    def saving(self, positions: Iterable[int]) -> "YachtDice":
        if not all(0 <= n < 6 for n in positions):
            raise ValueError("Invalid position")
        self.saved = set(positions)
        return self

    def roll(self) -> None:
        for n, d in enumerate(self.dice):
            if n not in self.saved:
                d.roll()
        self.saved = set()
```

Uso:

```python
>>> sd = YachtDice()
>>> sd.roll()
>>> sd.dice
[2, 2, 2, 6, 1]
>>> sd.saving([0, 1, 2]).roll()
>>> sd.dice
[2, 2, 2, 6, 6]
```

#### 6.1.5 Desmitificando la magia de las ABCs

Inspección interna:

```python
>>> Die.__abstractmethods__
frozenset({'roll'})
>>> Die.roll.__isabstractmethod__
True
```

El decorador `@abc.abstractmethod` establece `__isabstractmethod__ = True`. Al crear la clase, la metaclase `abc.ABCMeta` (que extiende a `type`) recopila todos estos métodos en el conjunto `__abstractmethods__`. La clase sólo podrá instanciarse cuando este conjunto quede vacío en una subclase.

Podemos asociar la metaclase explícitamente mediante el argumento `metaclass`:

```python
class DieM(metaclass=abc.ABCMeta):
    def __init__(self) -> None:
        self.face: int
        self.roll()

    @abc.abstractmethod
    def roll(self) -> None:
        ...
```

---

### Sección 6.2: Sobrecarga de operadores

Los operadores de Python (`+`, `-`, `*`, `/`, etc.) se implementan mediante **métodos especiales**.

Por ejemplo, el operador `/` en `pathlib.Path`:

```python
>>> from pathlib import Path
>>> home = Path.home()
>>> home / ".cargo" / "bin" / "uv"
PosixPath('/Users/slott/.cargo/bin/uv')
```

Para una expresión `A op B`:
1. Si `B` es subclase de `A`, Python intenta `B.__rop__(A)`.
2. Python intenta `A.__op__(B)`. Si retorna `NotImplemented`, continúa.
3. Python intenta `B.__rop__(A)`. Si retorna `NotImplemented`, lanza `TypeError`.

Extendamos nuestra simulación de dados para soportar operaciones aritméticas (`+`, `*`, `+=`):

```python
class DDice:
    def __init__(self, *die_class: type[Die]) -> None:
        self.classes = die_class
        self.dice = [dc() for dc in self.classes]
        self.adjust: int = 0

    def plus(self, adjust: int = 0) -> "DDice":
        self.adjust = adjust
        return self

    def roll(self) -> None:
        for d in self.dice:
            d.roll()

    @property
    def total(self) -> int:
        return sum(d.face for d in self.dice) + self.adjust

    def __add__(self, die_class: Any) -> "DDice":
        match die_class:
            case type() if issubclass(die_class, Die):
                new_classes = self.classes + (die_class,)
                new = DDice(*new_classes).plus(self.adjust)
                return new
            case int() as adj:
                new = DDice(*self.classes).plus(self.adjust + adj)
                return new
            case _:
                return NotImplemented

    def __radd__(self, die_class: Any) -> "DDice":
        match die_class:
            case type() if issubclass(die_class, Die):
                new_classes = (die_class,) + self.classes
                new = DDice(*new_classes).plus(self.adjust)
                return new
            case int() as adj:
                new = DDice(*self.classes).plus(self.adjust + adj)
                return new
            case _:
                return NotImplemented

    def __mul__(self, n: Any) -> "DDice":
        match n:
            case int():
                new_classes = self.classes * n
                return DDice(*new_classes).plus(self.adjust)
            case _:
                return NotImplemented

    __rmul__ = __mul__

    def __iadd__(self, die_class: Any) -> "DDice":
        match die_class:
            case type() if issubclass(die_class, Die):
                self.classes += (die_class,)
                self.dice = [dc() for dc in self.classes]
                return self
            case int() as adj:
                self.adjust += adj
                return self
            case _:
                return NotImplemented
```

Construcción y tirada de dados utilizando operadores sobrecargados:

```python
>>> y = DDice(D6, D6)
>>> y += D6
>>> y += 2
>>> y.roll()
>>> y.dice
[5, 6, 2]
```

---

### Sección 6.3: Extensión de tipos integrados

Podemos crear estructuras especializadas heredando directamente de tipos integrados como `dict`, interceptando la inserción de claves duplicadas:

```python
from collections.abc import Iterable, Hashable, Mapping
from typing import cast, Any

type DictInit = (
    Iterable[tuple[Hashable, Any]] | Mapping[Hashable, Any] | None
)

class NoDupdict(dict[Hashable, Any]):
    def __setitem__(self, key: Hashable, value: Any) -> None:
        if key in self:
            raise ValueError(f"duplicate {key!r}")
        super().__setitem__(key, value)

    def __init__(self, init: DictInit = None, **kwargs: Any) -> None:
        match init:
            case Mapping():
                super().__init__(init, **kwargs)
            case Iterable():
                for k, v in cast(Iterable[tuple[Hashable, Any]], init):
                    self[k] = v
            case None:
                super().__init__(**kwargs)
            case _:
                super().__init__(init, **kwargs)
```

---

### Sección 6.4: Metaclases

Las **metaclases** son clases cuyas instancias son clases. La metaclase por defecto es `type`.

> **Figura 6.3: Creación de clases con `type`**  
> `class statement` ──▷ `metaclass.__prepare__()` ──▷ `namespace execution` ──▷ `metaclass.__new__()`

Podemos crear metaclases personalizadas heredando de `type` o `abc.ABCMeta` para inyectar comportamientos automáticos (como registro en logs) en los métodos de las clases derivadas:

```python
import logging
from functools import wraps
from typing import Any, cast
import abc

class DieMeta(abc.ABCMeta):
    def __new__(
        cls: type,
        name: str,
        bases: tuple[type, ...],
        namespace: dict[str, Any],
        **kwargs: Any,
    ) -> "DieMeta":
        if "roll" in namespace and not getattr(
            namespace["roll"], "__isabstractmethod__", False
        ):
            namespace.setdefault("logger", logging.getLogger(name))
            original_method = namespace["roll"]

            @wraps(original_method)
            def logged_roll(self: "DieLog") -> None:
                original_method(self)
                self.logger.info(f"Rolled {self.face}")

            namespace["roll"] = logged_roll

        new_object = cast(
            "DieMeta", abc.ABCMeta.__new__(cls, name, bases, namespace)
        )
        return new_object

class DieLog(metaclass=DieMeta):
    logger: logging.Logger

    def __init__(self) -> None:
        self.face: int
        self.roll()

    @abc.abstractmethod
    def roll(self) -> None:
        ...

class D6L(DieLog):
    def roll(self) -> None:
        """Some documentation on D6L"""
        self.face = random.randrange(1, 7)
```

Uso:

```python
>>> random.seed(42)
>>> d = D6L()
>>> d2 = D6L()
>>> d2.face
6
```

---

### Sección 6.5: Repaso

Puntos clave tratados en este capítulo:

- **Clases Base Abstractas (ABCs):** Definen contratos e interfaces formales mediante `abc.ABC` y `@abc.abstractmethod`.
- **`collections.abc`:** Proporciona las abstracciones fundamentales para contenedores, secuencias, conjuntos y mapeos.
- **Sobrecarga de operadores:** Permite vincular operadores sintácticos (`+`, `*`, `/`, `in`, `len()`) a métodos mágicos (`__add__`, `__mul__`, `__truediv__`, `__contains__`, `__len__`).
- **Metaclases:** Permiten personalizar la construcción y registro de clases intermediando en el ciclo de vida de `type.__new__()`.

---

### Sección 6.6: Ejercicios

1. **Jerarquías de lectura de archivos:** Diseña una ABC `DataExtractor` que declare métodos abstractos `read()`, `validate()` y `transform()`, e implementa subclases para formatos CSV, JSON y Parquet.
2. **Operadores de combinación:** Modela una clase `QueryFilter` que sobrecargue los operadores `&` (`__and__`) y `|` (`__or__`) para componer filtros de búsqueda.
3. **Metaclase de registro de plugins:** Diseña una metaclase que mantenga un registro global automático de todas las subclases concretas que se definan en una aplicación.

---

### Sección 6.7: Resumen

En este capítulo exploramos las clases base abstractas, la integración con las abstracciones de colecciones estándar, la sobrecarga de operadores para construir APIs naturales y la personalización de clases mediante metaclases.

En los próximos capítulos, profundizaremos en el sistema de tipos genéricos y las estructuras de datos avanzadas en Python.

