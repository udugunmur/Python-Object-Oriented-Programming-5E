# Parte 1: Fundamentos de la Programación Orientada a Objetos en Python
## Capítulo 4: Esperando lo Inesperado

El software es robusto; sin embargo, los sistemas informáticos pueden ser frágiles. Mientras que el software es altamente predecible, el contexto de ejecución puede proporcionar entradas y situaciones imprevistas. Los dispositivos fallan, las redes son poco fiables y la anarquía puede desatarse sobre nuestra aplicación. Necesitamos disponer de una forma de gestionar el espectro de fallos que acechan a los sistemas complejos como los ordenadores modernos.

Existen dos enfoques generales para lidiar con lo imprevisto:
1. **Retornar un valor reconocible de señalización de error desde una función:** Se puede utilizar un valor como `None`. La aplicación puede emplear otras funciones de librería para recuperar detalles de la condición de error. Una variante consiste en emparejar el valor de retorno de una petición del sistema operativo con un indicador de éxito o fracaso.
2. **Interrumpir la ejecución secuencial normal de las instrucciones y desviar el flujo hacia sentencias diseñadas para gestionar la situación excepcional:** Este segundo enfoque es el que adopta Python: elimina la necesidad de comprobar continuamente los valores de retorno en busca de errores.

En este capítulo, estudiaremos las **excepciones**, objetos especiales de error que se lanzan cuando una respuesta normal resulta imposible. En particular, cubriremos lo siguiente:

- Cómo hacer que ocurra una excepción.
- Cómo recuperarse cuando se ha producido una excepción.
- Cómo gestionar diferentes tipos de excepciones de maneras distintas.
- Tareas de limpieza cuando ha ocurrido una excepción (`finally`).
- Creación de nuevos tipos de excepciones personalizadas.
- Uso de la sintaxis de excepciones para el control de flujo.

Comenzaremos examinando el concepto de `Exception` en Python y cómo se lanzan y gestionan.

---

### Sección 4.1: Lanzamiento de excepciones

El comportamiento normal de Python consiste en ejecutar las sentencias en el orden en que se encuentran, ya sea en un archivo o de forma interactiva en el prompt `>>>`. Unas pocas instrucciones, específicamente `if`, `while` y `for`, alteran la secuencia simple de arriba a abajo. Además, una **excepción** puede romper el flujo secuencial de ejecución: cuando se lanza una excepción, se interrumpe la ejecución normal de las instrucciones.

En Python, la excepción que se lanza es también un **objeto**. Existen muchas clases de excepciones integradas disponibles y podemos definir fácilmente más de nuestra propia cosecha. Lo que todas tienen en común es que heredan de una clase integrada denominada `BaseException` (en la práctica, nos interesan mucho más las excepciones basadas en la clase `Exception`).

Cuando se lanza una excepción, todo lo que debía ocurrir a continuación queda cancelado. En su lugar, el procesamiento de excepciones reemplaza al flujo normal.

La forma más sencilla de provocar una excepción es cometer un error sintáctico o de lógica. Por ejemplo, cada vez que Python encuentra una línea que no puede entender, lanza una excepción `SyntaxError`:

```python
>>> print "hello world"
  Traceback (most recent call last):
    ...
    print "hello world"
    ^^^^^^^^^^^^^^^^^^^
SyntaxError: Missing parentheses in call to 'print'. Did you mean print(...)?
```

La función `print()` requiere que los argumentos estén encerrados entre paréntesis. Observa que hemos usado `...` para omitir algunos detalles del mensaje de traza (*traceback*) que no resultan críticos por ahora.

Además de `SyntaxError`, en el siguiente ejemplo se muestran otras excepciones comunes:

```python
>>> x = 5 / 0
Traceback (most recent call last):
  ...
    x = 5 / 0
      ~~^~~
ZeroDivisionError: division by zero

>>> lst = [1,2,3]
>>> print(lst[3])
Traceback (most recent call last):
  ...
    print(lst[3])
          ~~~^^^
IndexError: list index out of range

>>> lst + 2
Traceback (most recent call last):
  ...
    lst + 2
    ~~~~^~~
TypeError: can only concatenate list (not "int") to list

>>> lst.add
Traceback (most recent call last):
  ...
AttributeError: 'list' object has no attribute 'add'

>>> d = {'a': 'hello'}
>>> d['b']
Traceback (most recent call last):
  ...
    d['b']
    ~^^^^^
KeyError: 'b'

>>> print(this_is_not_a_var)
Traceback (most recent call last):
  ...
    print(this_is_not_a_var)
          ^^^^^^^^^^^^^^^^^
NameError: name 'this_is_not_a_var' is not defined
```

Podemos clasificar estas excepciones a grandes rasgos en tres categorías:

1. **Indicadores de un error evidente en la forma en que escribimos el programa:** Excepciones como `SyntaxError` y `NameError` indican que debemos buscar el número de línea señalado y corregir el problema. Un `ImportError`, por ejemplo, suele ser un nombre mal escrito o una librería que no se ha instalado en el entorno virtual.
2. **Indicadores de un fallo en el entorno de ejecución de Python:** Como ejemplo, existe una excepción genérica `SystemError` que puede lanzarse ante problemas internos. En ocasiones, reiniciar el equipo resuelve el problema; otras veces, indica que es momento de actualizar la versión de Python.
3. **Problemas de diseño:** Ocurren cuando omitimos contemplar un caso límite, como calcular el promedio de una lista vacía, lo que resulta en un `ZeroDivisionError`. Cuando encontramos estas excepciones, debemos retroceder desde el punto de fallo para identificar qué causó que un objeto terminara en un estado imprevisto.

La mayor parte de estas excepciones surgen cerca de las interfaces del programa: cualquier entrada de usuario o petición al sistema operativo (incluidas las operaciones con archivos) puede encontrar problemas con recursos externos, lo que se traduce en excepciones del tipo `OSError` o `IOError`.

Habrás notado que la mayoría de las excepciones integradas en Python terminan con la palabra `Error`. En Python, los términos "error" y "excepción" se utilizan casi indistintamente.

Todas las clases de error del ejemplo anterior tienen a `Exception` como su clase base. La excepción `SystemExit` es un ejemplo de las pocas excepciones basadas directamente en `BaseException`, y se utiliza internamente para detener la ejecución de Python.

#### 4.1.1 Lanzar una excepción (`raise`)

¿Qué debemos hacer si escribimos un programa que necesita informar al usuario o a una función invocadora sobre una situación excepcional? Usamos la instrucción `raise`.

A continuación se muestra una clase simple que añade elementos a una lista únicamente si son números enteros pares:

```python
class EvenOnly(list[int]):
    def append(self, value: int) -> None:
        match value:
            case int():
                if value % 2 != 0:
                    raise ValueError("Only even numbers can be added")
            case _:
                raise TypeError("Only integers can be added")
        super().append(value)
```

La palabra clave `raise` va seguida del objeto que se lanza como excepción. En el ejemplo anterior, se construyen objetos a partir de las clases integradas `TypeError` y `ValueError`. El objeto lanzado podría ser igualmente una instancia de una clase de excepción personalizada creada por nosotros.

Probemos esta clase en el intérprete de Python:

```python
>>> e = EvenOnly()
>>> e.append("a string")
Traceback (most recent call last):
  ...
TypeError: Only integers can be added
>>> e.append(3)
Traceback (most recent call last):
  ...
ValueError: Only even numbers can be added
>>> e.append(2)
```

#### 4.1.2 Los efectos de una excepción

Cuando se lanza una excepción, la ejecución del programa se detiene inmediatamente en ese punto. Las líneas posteriores a la excepción no se ejecutan y, a menos que la excepción sea capturada por una cláusula `except`, el programa finalizará mostrando un mensaje de error y la traza de llamadas (*traceback*).

Considera esta función básica:

```python
from typing import NoReturn

def never_returns() -> NoReturn:
    print("I am about to raise an exception")
    raise Exception("This is always raised")
    print("This line will never execute")
    return "I won't be returned"
```

Si ejecutamos esta función:

```python
>>> never_returns()
I am about to raise an exception
Traceback (most recent call last):
  ...
    never_returns()
  ...
    raise Exception("This is always raised")
Exception: This is always raised
```

Si una función llama a otra función que lanza una excepción, nada se ejecuta en la función llamadora después del punto donde ocurrió la excepción:

```python
def call_exceptor() -> None:
    print("call_exceptor starts here...")
    never_returns()
    print("an exception was raised...")
    print("...so these lines don't run")
```

Al invocarla:

```python
>>> call_exceptor()
call_exceptor starts here...
I am about to raise an exception
Traceback (most recent call last):
  ...
    call_exceptor()
  ...
    never_returns()
  ...
    raise Exception("This is always raised")
Exception: This is always raised
```

Una excepción no capturada se propaga hacia arriba a través de la pila de llamadas (*call stack*) hasta que es manejada o fuerza la salida del intérprete.

---

### Sección 4.2: Manejo de excepciones

Gestionamos las excepciones envolviendo el código susceptible de fallar dentro de un bloque `try...except`:

```python
def handler() -> None:
    try:
        never_returns()
        print("Never executed")
    except Exception as ex:
        print(f"I caught an exception: {ex!r}")
    print("Executed after the exception")
```

Ejecutando la función:

```python
>>> handler()
I am about to raise an exception
I caught an exception: Exception('This is always raised')
Executed after the exception
```

El bloque `except` captura la excepción, permitiendo limpiar recursos o reportar la incidencia, y la ejecución del programa continúa normalmente tras el bloque `try...except`.

#### Captura de excepciones específicas

En lugar de capturar la clase genérica `Exception`, es una mejor práctica capturar excepciones específicas:

```python
def funny_division(divisor: float) -> str | float:
    try:
        return 100 / divisor
    except ZeroDivisionError:
        return "Zero is not a good idea!"
```

Probando diferentes entradas:

```python
>>> print(funny_division(0))
Zero is not a good idea!
>>> print(funny_division(50.0))
2.0
>>> print(funny_division("hello"))
Traceback (most recent call last):
  ...
TypeError: unsupported operand type(s) for /: 'int' and 'str'
```

> [!WARNING]
> A partir de Python 3.14, el uso de la cláusula `except:` sin especificar ninguna clase ("*bare except*") genera advertencias y quedará obsoleta (PEP 760). Equivale a capturar `BaseException`, lo que puede interceptar señales críticas del sistema como `KeyboardInterrupt` (Ctrl + C) o `SystemExit`, impidiendo detener el programa cuando es necesario.

#### Manejo de múltiples tipos de excepción

Podemos agrupar múltiples excepciones en una sola cláusula `except` mediante una tupla:

```python
def funnier_division(divisor: int) -> str | float:
    try:
        if divisor == 13:
            raise ValueError("13 is an unlucky number")
        return 100 / divisor
    except (ZeroDivisionError, TypeError):
        return "Enter a number other than zero"
```

Prueba con diversos valores:

```python
>>> for val in (0, "hello", 50.0):
...     print(f"Testing {val!r}:", end=" ")
...     print(funnier_division(val))
... 
Testing 0: Enter a number other than zero
Testing 'hello': Enter a number other than zero
Testing 50.0: 2.0
>>> val = 13
>>> print(funnier_division(val))
Traceback (most recent call last):
  ...
ValueError: 13 is an unlucky number
```

#### Re-lanzamiento de excepciones (`re-raise`) y múltiples bloques `except`

Podemos encadenar varios bloques `except` y relanzar una excepción con la palabra clave `raise` en solitario:

```python
def funniest_division(divisor: int) -> str | float:
    try:
        if divisor == 13:
            raise ValueError("13 is an unlucky number")
        return 100 / divisor
    except ZeroDivisionError:
        return "Enter a number other than zero"
    except TypeError:
        return "Enter a numerical value"
    except ValueError:
        print("No, No, not 13!")
        raise
```

> [!TIP]
> Los bloques `except` deben ordenarse **de la subclase más específica a la superclase más genérica**. Si colocas `except Exception:` al principio, capturará todas las excepciones e impedirá que se ejecuten los bloques más específicos situados debajo.

#### Acceso a los argumentos de la excepción (`as e`)

Podemos capturar el objeto de la excepción asignándolo a una variable con la cláusula `as`:

```python
>>> try:
...     raise ValueError("This is an argument")
... except ValueError as e:
...     print(f"The exception arguments were {e.args}")
... 
The exception arguments were ('This is an argument',)
```

#### Cláusulas `else` y `finally`

Python proporciona dos cláusulas adicionales para complementar `try...except`:

- **`else`:** Se ejecuta únicamente si **no** se produjo ninguna excepción en el bloque `try`.
- **`finally`:** Se ejecuta **siempre**, independientemente de si se produjo una excepción, si fue capturada o si se ejecutó una sentencia `return`.

```python
some_exceptions = [ValueError, TypeError, IndexError, None]

for choice in some_exceptions:
    try:
        print(f"\nRaising {choice}")
        if choice:
            raise choice("An error")
        else:
            print("no exception raised")
    except ValueError:
        print("Caught a ValueError")
    except TypeError:
        print("Caught a TypeError")
    except Exception as e:
        print(f"Caught some other error: {e.__class__.__name__}")
    else:
        print("This code called if there is no exception")
    finally:
        print("This cleanup code is always called")
```

Salida resultante:

```text
Raising <class 'ValueError'>
Caught a ValueError
This cleanup code is always called

Raising <class 'TypeError'>
Caught a TypeError
This cleanup code is always called

Raising <class 'IndexError'>
Caught some other error: IndexError
This cleanup code is always called

Raising None
no exception raised
This code called if there is no exception
This cleanup code is always called
```

El bloque `finally` se utiliza tradicionalmente para tareas de limpieza (como cerrar conexiones a bases de datos o archivos), aunque hoy en día es más habitual encapsular estas operaciones mediante **administradores de contexto** (*context managers* con la sentencia `with`).

#### 4.2.1 La jerarquía de excepciones

La jerarquía de excepciones estándar de Python se estructura de la siguiente manera:

> **Figura 4.1: Jerarquía de excepciones**  
> `BaseException`  
> ├── `KeyboardInterrupt`  
> ├── `SystemExit`  
> ├── `GeneratorExit`  
> ├── `BaseExceptionGroup`  
> └── `Exception`  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── `ArithmeticError` (`ZeroDivisionError`, ...)  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── `LookupError` (`IndexError`, `KeyError`, ...)  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── `ValueError`  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├── `TypeError`  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└── `OSError` (`FileNotFoundError`, `PermissionError`, ...)  

---

### Sección 4.3: Definición de nuestras propias excepciones

Cuando ninguna de las excepciones integradas refleja con precisión la semántica de nuestro dominio, podemos crear excepciones personalizadas heredando de `Exception` (o de una subclase adecuada):

```python
class InvalidWithdrawal_1(ValueError):
    pass
```

```python
>>> raise InvalidWithdrawal_1("You don't have $50 in your account")
Traceback (most recent call last):
  ...
banking.InvalidWithdrawal_1: You don't have $50 in your account
```

Podemos enriquecer la excepción personalizada con atributos y métodos propios:

```python
from decimal import Decimal

class InvalidWithdrawal(ValueError):
    def __init__(self, balance: Decimal, amount: Decimal) -> None:
        super().__init__(f"account doesn't have ${amount}")
        self.amount = amount
        self.balance = balance

    def overage(self) -> Decimal:
        return self.amount - self.balance
```

Lanzamiento y manejo del error:

```python
>>> raise InvalidWithdrawal(Decimal('28.63'), Decimal('42.00'))
Traceback (most recent call last):
  ...
banking.InvalidWithdrawal: account doesn't have $42.00
```

```python
>>> balance = Decimal('28.63')
>>> transfer = Decimal('42.00')
>>> try:
...     new_balance = do_transfer(balance, transfer)
... except InvalidWithdrawal as ex:
...     print("I'm sorry, but your withdrawal is "
...           "more than your balance by "
...           f"${ex.overage()}")
... 
I'm sorry, but your withdrawal is more than your balance by $13.37
```

#### 4.3.1 Las excepciones no son tan excepcionales (EAFP vs. LBYL)

En Python predomina el principio **EAFP** (*Easier to Ask for Forgiveness than Permission* — "Es más fácil pedir perdón que pedir permiso"), frente al enfoque **LBYL** (*Look Before You Leap* — "Mira antes de saltar"):

```python
def divide_with_exception(dividend: int, divisor: int) -> None:
    try:
        print(f"{dividend / divisor=}")
    except ZeroDivisionError:
        print("You can't divide by zero")

def divide_with_if(dividend: int, divisor: int) -> None:
    if divisor == 0:
        print("You can't divide by zero")
    else:
        print(f"{dividend / divisor=}")
```

En escenarios complejos, verificar previamente si una operación tendrá éxito (*LBYL*) requiere ejecutar comprobaciones costosas por duplicado. Con *EAFP*, se ejecuta directamente la operación y se gestionan las anomalías si ocurren.

#### Uso de excepciones para el control de flujo en un inventario

Las excepciones son herramientas de comunicación excelentes entre diferentes capas de una aplicación:

```python
class OutOfStock(Exception):
    pass

class InvalidItemType(Exception):
    pass

class ItemType:
    def __init__(self, name: str) -> None:
        self.name = name
        self.on_hand = 0

class Inventory:
    def __init__(self, stock: list[ItemType]) -> None:
        pass

    def lock(self, item_type: ItemType) -> None:
        """Context Entry. Lock the item type so nobody else can manipulate the inventory while we're working."""
        pass

    def unlock(self, item_type: ItemType) -> None:
        """Context Exit. Unlock the item type."""
        pass

    def purchase(self, item_type: ItemType) -> int:
        """If the item is not locked, raise an ValueError because someting went wrong.
        If the item_type does not exist, raise InvalidItemType.
        If the item is currently out of stock, raise OutOfStock.
        If the item is available, subtract one item; return the number of items left.
        """
        # Mocked results.
        if item_type.name == "Widget":
            raise OutOfStock(item_type)
        elif item_type.name == "Gadget":
            return 42
        else:
            raise InvalidItemType(item_type)
```

Sesión de uso interactivo:

```python
>>> widget = ItemType("Widget")
>>> gadget = ItemType("Gadget")
>>> inv = Inventory([widget, gadget])
>>> item_to_buy = widget
>>> inv.lock(item_to_buy)
>>> try:
...     num_left = inv.purchase(item_to_buy)
... except InvalidItemType:
...     print(f"Sorry, we don't sell {item_to_buy.name}")
... except OutOfStock:
...     print("Sorry, that item is out of stock.")
... else:
...     print(f"Purchase complete. There are {num_left} {item_to_buy.name}s left")
... finally:
...     inv.unlock(item_to_buy)
... 
Sorry, that item is out of stock.
```

---

### Sección 4.4: Repaso

Puntos clave tratados en este capítulo:

- **Lanzamiento:** Las excepciones se lanzan con `raise` cuando se produce una situación anómala.
- **Efectos:** Interrumpen la ejecución secuencial y propagan el error por la pila de llamadas hasta encontrar un bloque de captura.
- **Manejo:** Se utiliza `try...except...else...finally` para capturar errores, realizar limpiezas y gestionar el flujo de ejecución.
- **Jerarquía:** La mayoría de las excepciones derivan de `Exception`. Las excepciones de sistema como `SystemExit` y `KeyboardInterrupt` heredan directamente de `BaseException`.
- **Excepciones personalizadas:** Permiten enriquecer los errores con datos específicos del dominio y crear contratos claros en APIs y librerías.

---

### Sección 4.5: Ejercicios

1. **Auditoría de código existente:** Revisa proyectos previos de Python e identifica lugares donde deberías manejar excepciones en lugar de comprobaciones condicionales `if` defensivas (p. ej., acceso a archivos, operaciones matemáticas, claves de diccionarios).
2. **Implementación de `else` y `finally`:** Diseña un flujo donde sea crítico asegurar que un recurso se libere siempre en la cláusula `finally`.
3. **Diseño de excepciones de API:** Modela un conjunto de excepciones personalizadas para una librería o servicio (p. ej., autenticación, validación de transacciones o procesamiento de pedidos).
4. **Refactorización del script del Capítulo 1:** Analiza qué excepciones deberían capturarse al leer y parsear los archivos JSON de resultados de pruebas y cuáles deberían detener el procesamiento.

---

### Sección 4.6: Resumen

En este capítulo analizamos en profundidad el lanzamiento, captura, propagación y personalización de excepciones en Python. Las excepciones proporcionan un mecanismo robusto y expresivo para gestionar condiciones inusuales y estructurar el control de flujo sin recurrir a códigos de error manuales.

En el próximo capítulo, integraremos todos los conceptos estudiados hasta ahora para analizar cuándo y cómo conviene aplicar el diseño orientado a objetos en aplicaciones reales en Python.
