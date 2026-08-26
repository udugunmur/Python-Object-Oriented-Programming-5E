# Parte 1: Fundamentos de la Programación Orientada a Objetos en Python
## Capítulo 3: Cuando los Objetos son Similares

En el mundo de la programación, el código duplicado se considera perjudicial. No deberíamos tener múltiples copias del mismo código, o de código similar, en diferentes lugares. Cuando corregimos un error (o añadimos una funcionalidad) en una copia y no realizamos el mismo cambio en otra, nos generamos un sinfín de problemas.

Existen muchas formas de fusionar fragmentos de código u objetos que tienen una funcionalidad similar. En este capítulo, cubriremos el principio orientado a objetos más famoso: la **herencia**. Como vimos en el Capítulo 1, la herencia nos permite crear relaciones del tipo **"es-un"** (*is-a*) entre dos o más clases, abstrayendo la lógica común en superclases y extendiendo la superclase con detalles específicos en cada subclase. En particular, cubriremos la sintaxis y los principios de Python para lo siguiente:

- La relación de herencia.
- Uso de la herencia para escribir clases en Python.
- Uso de la herencia para extender tipos integrados.
- Herencia múltiple.
- Polimorfismo y *duck typing* (tipado de pato).

Comenzaremos examinando de cerca cómo funciona la herencia para factorizar características comunes y evitar la programación basada en copiar y pegar.

---

### Sección 3.1: La relación de herencia

Por ejemplo, hay 32 piezas de ajedrez en nuestro juego, pero solo hay seis tipos diferentes de piezas (peones, torres, alfiles, caballos, rey y reina). Cada tipo se comporta de manera diferente cuando se mueve. Cada una de estas clases de piezas tiene propiedades, como el color y el juego de ajedrez del que forman parte, pero también tienen formas únicas cuando se dibujan en el tablero de ajedrez y realizan movimientos diferentes.

La Figura 3.1 muestra cómo los seis tipos de piezas pueden heredar de una clase `Piece`:

> **Figura 3.1: Cómo heredan las piezas de ajedrez de la clase `Piece`**  
> *(Pawn, Rook, Bishop, Knight, King, Queen ──▷ Piece)*

Las flechas huecas indican que las clases individuales de piezas heredan de la clase `Piece`. Todas las clases hijas tienen automáticamente los atributos `chess_set` y `color` heredados de la clase base. Cada pieza proporciona una propiedad `shape` diferente (para dibujarse en la pantalla al renderizar el tablero) y un método `move` diferente para mover la pieza a una nueva posición en el tablero en cada turno.

Para poder moverse, todas las subclases de la clase `Piece` deben tener un método `move`; de lo contrario, cuando el tablero intente mover la pieza, se producirá un error. Es posible que queramos crear una nueva versión del juego de ajedrez que tenga una pieza adicional (tal vez un mago). Nuestro diseño actual nos permitiría agregar esta pieza sin proporcionarle un método `move`. Intentar usar esta pieza sin el método requerido causará problemas en tiempo de ejecución.

Podemos solucionar esto creando un método ficticio `move()` en la clase `Piece`. Las subclases pueden luego sobrescribir este método con una implementación más específica. La implementación predeterminada podría, por ejemplo, mostrar un mensaje de error que diga: *"Esa pieza no se puede mover"*.

Sobrescribir métodos en subclases permite desarrollar sistemas orientados a objetos muy potentes. Por ejemplo, si quisiéramos implementar una clase `Player` con inteligencia artificial, podríamos proporcionar un método `calculate_move` que tome un objeto `Board` y decida qué pieza mover y hacia dónde. Una clase muy básica podría elegir aleatoriamente una pieza y una dirección y moverla en consecuencia. Luego podríamos sobrescribir este método en una subclase con la implementación de *Deep Blue*. La primera clase sería adecuada para jugar contra un principiante absoluto; la segunda desafiaría a un gran maestro. Lo importante es que otros métodos de la clase, como los que informan al tablero sobre qué movimiento se eligió, no necesitan cambiarse; esta implementación se puede compartir entre ambas clases.

En el caso de las piezas de ajedrez, en realidad no tiene sentido proporcionar una implementación predeterminada del método `move`. Todo lo que necesitamos hacer es especificar que el método `move` es obligatorio en cualquier subclase. Esto se puede lograr haciendo que `Piece` sea una **clase abstracta** con el método `move` declarado como abstracto. Los métodos abstractos declaran esencialmente lo siguiente:

> *"Exigimos que este método exista en cualquier subclase no abstracta, pero declinamos especificar una implementación en esta clase base."*

De hecho, es posible crear una abstracción que no implemente ningún método en absoluto. Dicha clase simplemente nos indicará qué debe hacer la clase, sin proporcionar ninguna ayuda sobre cómo hacerlo. En Python, solemos utilizar el módulo `abc` para definir estas clases base abstractas. Marcamos los métodos abstractos con el decorador `@abstractmethod` como recordatorio de que se requiere una implementación concreta.

```python
import abc

class Piece(abc.ABC):
    def __init__(self, set: ChessSet, color: Color, shape: Graphic) -> None:
        self.chess_set = set
        self.color = color
        self.shape = shape

    @abc.abstractmethod
    def move(self, board: Board, to: Position) -> None:
        ...
```

Hacemos que una clase base abstracta sea una subclase de `abc.ABC`. Hacer esto garantiza que se generará una excepción si la aplicación intenta crear una instancia de una subclase que no proporciona implementaciones concretas para todos los métodos abstractos.

El decorador `@abc.abstractmethod` marca una definición como una especificación que requiere ser sobrescrita. Y sí, el cuerpo del método es literalmente `...` (puntos suspensivos o el objeto `Ellipsis`); esto es Python válido y generará un error en tiempo de ejecución si, por alguna razón, este método llegara a ejecutarse. Se generará una excepción `TypeError` al intentar instanciar una clase que tenga métodos abstractos pendientes de implementación. Crear una instancia de una subclase que proporciona una implementación concreta para los métodos abstractos funcionará según lo previsto.

---

### Sección 3.2: Uso de la herencia

La idea central detrás de la herencia es proporcionar una forma de evitar repetir código en múltiples clases.

Técnicamente, cada clase que creamos utiliza herencia. Todas las clases de Python son subclases de la clase integrada denominada `object`. Esta clase proporciona metadatos básicos y comportamientos integrados para que Python pueda tratar todos los objetos de manera consistente.

La herencia requiere una cantidad mínima de sintaxis adicional respecto a una definición de clase básica: incluir el nombre de la clase padre entre paréntesis después del nombre de la clase (y antes de los dos puntos). Esto es todo lo que tenemos que hacer para indicarle a Python que la nueva clase debe derivar de la superclase especificada. Para obtener más información sobre la sintaxis, consulta la sección 9.5 ([https://docs.python.org/3/tutorial/classes.html#inheritance](https://docs.python.org/3/tutorial/classes.html#inheritance)) del Tutorial de Python.

¿Cómo aplicamos la herencia en la práctica? Un uso común de la herencia es añadir funcionalidad a una clase existente. Comencemos con un gestor de contactos que realiza un seguimiento de los nombres y direcciones de correo electrónico de varias personas. Una clase `Contact` puede ser responsable de mantener una lista global de todos los contactos vistos en una variable de clase, y de inicializar el nombre y la dirección de cada contacto individual:

```python
from __future__ import annotations
from typing import ClassVar

class Contact:
    all_contacts: ClassVar[list[Contact]] = []

    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email
        Contact.all_contacts.append(self)

    def __repr__(self) -> str:
        return (
            f"{self.__class__.__name__}("
            f"{self.name!r}, {self.email!r}"
            f")"
        )
```

Este ejemplo introduce dos conceptos: **variables de clase** y el método `__repr__()`. La lista `all_contacts`, al formar parte de la definición de la clase, es compartida por todas las instancias de esta clase. Esto significa que solo existe una única lista `Contact.all_contacts`. Hay dos modos de acceso para esto: lectura y escritura.

- Podemos obtener el valor `self.all_contacts` dentro de cualquier método en una instancia de la clase `Contact`. Esto funciona porque cualquier atributo que no se encuentre en el objeto (a través de `self`) se buscará en la clase.
- Solo podemos escribir o asignar valores utilizando explícitamente el nombre `Contact.all_contacts` (si alguna vez intentas asignar una variable de clase usando `self.all_contacts = ...`, en realidad estarás creando una nueva variable de instancia asociada solo a ese objeto).

El método `__repr__()` es utilizado por la función integrada `repr()`. Esta función se emplea a menudo al imprimir un objeto en el entorno interactivo REPL de Python. Solemos diseñar esta función para producir una línea de código Python que recree el objeto. Hemos utilizado los nombres especiales `__class__` y `__name__` para extraer información interna de cada objeto, exponiendo el nombre de la clase del objeto.

Podemos ver cómo la clase realiza el seguimiento de los datos con el siguiente ejemplo:

```python
>>> c_1 = Contact("Dusty", "dusty@example.com")
>>> c_2 = Contact("Steve", "steve@itmaybeahack.com")
>>> Contact.all_contacts
[Contact('Dusty', 'dusty@example.com'), Contact('Steve', 'steve@itmaybeahack.com')]
```

Creamos dos instancias de la clase `Contact` y las asignamos a las variables `c_1` y `c_2`. Al inspeccionar la variable de clase `Contact.all_contacts`, observamos que la lista se ha actualizado para registrar ambos objetos.

Esta clase nos permite rastrear información sobre cada contacto. Pero, ¿qué ocurre si algunos de nuestros contactos son también proveedores a los que necesitamos realizar pedidos? Podríamos añadir un método `order` a la clase `Contact`, pero eso permitiría pedir suministros accidentalmente a contactos que son clientes o familiares. En su lugar, creemos una nueva clase `Supplier` que actúe como nuestra clase `Contact`, pero que disponga de un método adicional `order` que acepte un objeto `Order`:

```python
class Supplier(Contact):
    def order(self, order: "Order") -> None:
        print(
            "If this were a real system we would send "
            f"'{order}' order to '{self.name}'"
        )
```

Ahora, si probamos esta clase en el intérprete, vemos que todos los contactos, incluidos los proveedores, aceptan un nombre y una dirección de correo electrónico en su método `__init__()`. Pero también vemos que solo las instancias de `Supplier` tienen un método `order()`:

```python
>>> c = Contact("Some Body", "somebody@example.net")
>>> s = Supplier("Sue Plier", "supplier@example.net")
>>> print(c.name, c.email, s.name, s.email)
Some Body somebody@example.net Sue Plier supplier@example.net
>>> from pprint import pprint
>>> pprint(c.all_contacts)
[Contact('Dusty', 'dusty@example.com'),
 Contact('Steve', 'steve@itmaybeahack.com'),
 Contact('Some Body', 'somebody@example.net'),
 Supplier('Sue Plier', 'supplier@example.net')]
>>> c.order("I need pliers")
Traceback (most recent call last):
  ...
AttributeError: 'Contact' object has no attribute 'order'
>>> s.order("I need pliers")
If this were a real system we would send 'I need pliers' order to 'Sue Plier'
```

Nuestra clase `Supplier` puede hacer todo lo que hace la clase `Contact` (incluido añadirse a la lista `Contact.all_contacts`) y además realiza las tareas específicas necesarias para un proveedor. Esta reutilización de código es la gran ventaja de la herencia.

Dado que utilizamos los atributos especiales `__class__` y `__name__` del objeto, tanto la clase `Contact` como la subclase `Supplier` reportarán su nombre de clase correcto. Asimismo, observa que `Contact.all_contacts` ha recopilado todas las instancias tanto de la clase `Contact` como de la subclase `Supplier`.

#### 3.2.1 Extensión de tipos integrados

Un uso interesante de la herencia es añadir funcionalidad a clases integradas (*built-ins*). En la clase `Contact` anterior, estamos añadiendo contactos a una lista de todos los contactos. ¿Qué pasaría si también quisiéramos buscar en esa lista por nombre? Podríamos añadir un método en la clase `Contact` para buscar en ella, pero tiene más sentido que este método pertenezca a la lista misma.

El siguiente ejemplo muestra cómo podemos hacer esto mediante la herencia de un tipo integrado, extendiendo en este caso el tipo `list`:

```python
from __future__ import annotations

class ContactList(list["Contact"]):
    def search(self, name: str) -> list[Contact]:
        """All Contacts with name that contains the name parameter's value."""
        matching_contacts: list[Contact] = []
        for contact in self:
            if name in contact.name:
                matching_contacts.append(contact)
        return matching_contacts

class Contact:
    all_contacts = ContactList()

    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email
        Contact.all_contacts.append(self)

    def __repr__(self) -> str:
        return (
            f"{self.__class__.__name__}("
            f"{self.name!r}, {self.email!r}"
            f")"
        )
```

Ten en cuenta que el tipo que estamos creando se escribe como `list["Contact"]` con comillas alrededor del tipo `Contact` aún no definido. Este tipo de referencia hacia adelante (*forward reference*) requiere cadenas cuando la clase aún no ha sido evaluada. Alternativamente, podríamos cambiar el orden de las definiciones.

En lugar de instanciar la clase genérica `list` para crear nuestra variable de clase `all_contacts`, creamos una nueva instancia de `ContactList`; esto extiende el tipo de datos integrado `list`, estableciendo una especificación más precisa sobre qué tipos de objetos poblarán esta lista. Podemos probar la nueva funcionalidad de búsqueda de la siguiente manera:

```python
>>> c1 = Contact("John A", "johna@example.net")
>>> c2 = Contact("John B", "johnb@sloop.net")
>>> c3 = Contact("Jenna C", "cutty@sark.io")
>>> Contact.all_contacts.search('John')
[Contact('John A', 'johna@example.net'), Contact('John B', 'johnb@sloop.net')]
```

Recuerda que Python tiene dos formas de crear objetos de lista genéricos. Crear una lista con `[]` es en realidad un atajo sintáctico (*azúcar sintáctico*) para crear una lista usando `list()`; ambas sintaxis se comportan de manera idéntica:

```python
>>> [] == list()
True
```

Herramientas como `mypy` pueden verificar el cuerpo del método `ContactList.search()` para asegurarse de que realmente generará una instancia de lista poblada con objetos `Contact`.

Como segundo ejemplo, podemos extender la clase `dict`, que es una colección de claves y sus valores asociados. Aquí hay un diccionario extendido que realiza un seguimiento de la clave más larga que ha registrado:

```python
class LongNameDict(dict[str, int]):
    def longest_key(self) -> str | None:
        """In effect, max(self, key=len), but less obscure"""
        longest = None
        for key in self:
            if longest is None or len(key) > len(longest):
                longest = key
        return longest
```

La pista de tipo para la clase acota el `dict` genérico a un `dict[str, int]` más específico. Esto significa que las claves son de tipo `str` y los valores son de tipo `int`. Dado que las claves son objetos de tipo `str`, la instrucción `for key in self:` iterará sobre objetos `str`. El resultado será un `str`, o posiblemente `None`. Por eso el tipo de retorno se describe como `str | None`.

Podemos probar esto en el intérprete interactivo:

```python
>>> articles_read = LongNameDict()
>>> articles_read['lucy'] = 42
>>> articles_read['c_c_phillips'] = 6
>>> articles_read['steve'] = 7
>>> articles_read.longest_key()
'c_c_phillips'
```

Si quisiéramos un diccionario más genérico, por ejemplo con cadenas o enteros como valores, usaríamos una unión de tipos como `dict[str, str | int]`. Analizaremos más sobre uniones de tipos en el Capítulo 7.

#### 3.2.2 Sobrescritura y `super()`

La herencia es excelente para agregar nuevos comportamientos a clases existentes, pero ¿qué ocurre cuando necesitamos **modificar un comportamiento existente**? Nuestra clase `Contact` solo permite un nombre y una dirección de correo electrónico. Esto puede ser suficiente para la mayoría de los contactos, pero ¿qué pasa si queremos agregar un número de teléfono para nuestros amigos cercanos?

Podemos hacer esto sobrescribiendo el método `__init__()`. **Sobrescribir** (*overriding*) significa alterar o reemplazar un método de la superclase con un nuevo método (con el mismo nombre) en la subclase. No se necesita ninguna sintaxis especial para hacer esto; el método recién creado de la subclase se invoca automáticamente en lugar del método de la superclase:

```python
class Friend(Contact):
    def __init__(self, name: str, email: str, phone: str) -> None:
        self.name = name
        self.email = email
        self.phone = phone
```

Cualquier método puede ser sobrescrito, no solo `__init__()`. Sin embargo, en este ejemplo las clases `Contact` y `Friend` tienen código duplicado para configurar las propiedades `name` y `email`. Además, nuestra clase `Friend` está omitiendo agregarse a la lista `all_contacts` creada en la clase `Contact`.

Lo que necesitamos es una forma de ejecutar el método `__init__()` original de la clase `Contact` desde dentro de nuestra nueva clase. Esto es exactamente lo que hace la función integrada `super()`: devuelve el objeto delegado a la clase padre, permitiéndonos invocar el método padre directamente:

```python
class Friend_S(Contact):
    def __init__(self, name: str, email: str, phone: str) -> None:
        super().__init__(name, email)
        self.phone = phone
```

Este ejemplo vincula primero la instancia a la clase padre usando `super()` y llama a `__init__()` en ese objeto, pasando los argumentos esperados. Luego, la clase `Friend_S` realiza su propia inicialización específica: establecer el atributo `phone`.

La clase `Contact` proporcionaba una definición para el método `__repr__()` para producir una representación en cadena. Nuestra subclase no sobrescribió el método `__repr__()` heredado de la superclase. Esta es la consecuencia:

```python
>>> f = Friend("Dusty", "Dusty@private.com", "555-1212")
>>> f
Friend('Dusty', 'Dusty@private.com')
```

Los detalles mostrados para una instancia de `Friend_S` no incluyen el nuevo atributo `phone`. Esto requiere sobrescribir también el método `__repr__()`:

```python
    def __repr__(self) -> str:
        return (
            f"{self.__class__.__name__}("
            f"{self.name!r}, {self.email!r}, {self.phone!r}"
            f")"
        )
```

Se puede realizar una llamada a `super()` dentro de cualquier método. Por lo tanto, todos los métodos pueden extenderse mediante sobrescritura y uso de llamadas a `super()`. La llamada a `super()` también se puede realizar en cualquier punto del método; no tiene que ser necesariamente la primera línea si necesitamos validar o transformar los parámetros antes de enviarlos a la superclase.

---

### Sección 3.3: Composición como alternativa a la herencia

Al examinar el diseño de las piezas de ajedrez en la sección 3.1, utilizamos la herencia para aislar características comunes de distintas variedades de piezas. Esta no es la única herramienta disponible: podríamos, con la misma facilidad, definir una serie de clases que tengan los diferentes tipos de métodos `move()`.

Cada pieza sería entonces una **composición** de métodos de `Piece` que muestran la posición y el icono de visualización, junto con una instancia de una de las clases de movimiento (`Move`).

La pregunta habitual es: *“¿Cuál es mejor?”*. Esta es una decisión de diseño fundamental. Generalmente, caracterizamos una relación de herencia con la expresión **"es-un"** (*is-a*), mientras que caracterizamos la composición y la agregación como **"tiene-un"** (*has-a*). En el caso del ajedrez, *"un peón es una pieza de ajedrez"* resulta claro y representativo.

Imaginemos una aplicación de comercio electrónico donde una dirección de correo electrónico se modela como una subclase de URLs porque comparten ciertos métodos de validación de cadenas. Defender la afirmación *"una dirección de correo electrónico es una URL"* ante un colega resulta inconsistente con la realidad. Un correo electrónico no es una URL, aunque comparta ciertas características de formato. En este caso, utilizar una composición de funcionalidades resulta mucho más adecuado que la herencia.

Ambas técnicas de diseño son complementarias: logran los mismos objetivos con costes conceptuales y sobrecargas de ejecución similares. Los principios de diseño SOLID proporcionan una guía valiosa para evaluar qué enfoque utilizar en cada escenario.

---

### Sección 3.4: Herencia múltiple

La **herencia múltiple** es un tema de debate recurrente. En principio, es simple: una subclase que hereda de más de una clase padre puede acceder a la funcionalidad de todas ellas. En la práctica, requiere cuidado para comprender con precisión el orden en que se resuelven las sobrescrituras de métodos.

La forma más sencilla y útil de herencia múltiple sigue el patrón de diseño conocido como **mixin**. Una clase *mixin* no está diseñada para existir por sí misma, sino para ser heredada por otra clase con el fin de proporcionarle funcionalidad adicional. Por ejemplo, supongamos que queremos añadir funcionalidad a nuestra clase `Contact` para permitir el envío de correos electrónicos a `self.email`.

Enviar correos es una tarea común que podríamos desear en muchas otras clases. Por lo tanto, podemos escribir una clase mixin para añadir esta capacidad:

```python
from typing import Protocol

class Emailable(Protocol):
    email: str

class MailSender(Emailable):
    def send_mail(self, message: str) -> None:
        print(f"Sending mail to {self.email=}")
        # Add e-mail logic here
```

Es una práctica común utilizar un adjetivo (como `Emailable`) para el nombre del protocolo o mixin. El mixin `MailSender` define el comportamiento `send_mail()`, confiando en que la clase que lo incorpore proporcionará el atributo `email`.

Podemos utilizar el mixin `MailSender` con cualquier clase que tenga definido un atributo `email`, creando una nueva clase mediante herencia múltiple:

```python
class EmailableContact(Contact, MailSender):
    pass
```

La sintaxis para la herencia múltiple incluye múltiples clases base separadas por comas dentro de los paréntesis. Podemos probar este nuevo híbrido para ver el mixin en acción:

```python
>>> e = EmailableContact("John B", "johnb@sloop.net")
>>> Contact.all_contacts
[EmailableContact('John B', 'johnb@sloop.net')]
>>> e.send_mail("Hello, test e-mail here")
Sending mail to self.email='johnb@sloop.net'
```

El inicializador de `Contact` continúa añadiendo el nuevo contacto a la lista `all_contacts`, y el mixin envía el correo a `self.email` exitosamente.

Exploremos qué ocurre cuando necesitamos añadir una dirección postal a nuestra clase `Friend` mediante un mixin `AddressHolder`:

```python
class AddressHolder:
    def __init__(
        self, street: str, city: str, state: str, code: str
    ) -> None:
        self.street = street
        self.city = city
        self.state = state
        self.code = code
```

#### 3.4.1 El problema del diamante (The Diamond Problem)

Al combinar `Contact` y `AddressHolder` en una clase `Friend`, nos encontramos con que ambas clases tienen métodos `__init__()` con diferentes conjuntos de parámetros.

Un enfoque ingenuo consistiría en invocar directamente los métodos `__init__()` de cada superclase:

```python
class Friend_A(Contact, AddressHolder):
    def __init__(
        self,
        name: str,
        email: str,
        phone: str,
        street: str,
        city: str,
        state: str,
        code: str,
    ) -> None:
        Contact.__init__(self, name, email)
        AddressHolder.__init__(self, street, city, state, code)
        self.phone = phone
```

Este enfoque presenta un riesgo crítico en jerarquías complejas: **el problema del diamante** (*Diamond Problem*).

La Figura 3.2 ilustra este problema de herencia:

> **Figura 3.2: Diagrama de herencia para la clase Friend**  
> *(Friend ──▷ Contact ──▷ object | Friend ──▷ AddressHolder ──▷ object)*

Dado que tanto `Contact` como `AddressHolder` heredan de `object`, una llamada manual directa provoca que la superclase `object` se inicialice dos veces.

Veamos un ejemplo con métodos explícitos para observar cómo se duplican las llamadas en una estructura de diamante:

> **Figura 3.3: Herencia en diamante**  
> *(Subclass ──▷ LeftSubclass ──▷ BaseClass | Subclass ──▷ RightSubclass ──▷ BaseClass)*

```python
class BaseClass:
    num_base_calls = 0

    def call_me(self) -> None:
        print("Calling method on BaseClass")
        self.num_base_calls += 1

class LeftSubclass(BaseClass):
    num_left_calls = 0

    def call_me(self) -> None:
        BaseClass.call_me(self)
        print("Calling method on LeftSubclass")
        self.num_left_calls += 1

class RightSubclass(BaseClass):
    num_right_calls = 0

    def call_me(self) -> None:
        BaseClass.call_me(self)
        print("Calling method on RightSubclass")
        self.num_right_calls += 1

class Subclass(LeftSubclass, RightSubclass):
    num_sub_calls = 0

    def call_me(self) -> None:
        LeftSubclass.call_me(self)
        RightSubclass.call_me(self)
        print("Calling method on Subclass")
        self.num_sub_calls += 1
```

Al ejecutar `Subclass.call_me()`:

```python
>>> s = Subclass()
>>> s.call_me()
Calling method on BaseClass
Calling method on LeftSubclass
Calling method on BaseClass
Calling method on RightSubclass
Calling method on Subclass
>>> print(
...     s.num_sub_calls,
...     s.num_left_calls,
...     s.num_right_calls,
...     s.num_base_calls
... )
1 1 1 2
```

El método `call_me()` de `BaseClass` se ejecutó **dos veces**. Si esta operación realizara una transacción bancaria o una conexión de red, causaría un fallo grave.

Python resuelve esto linealizando la jerarquía mediante el algoritmo de **Orden de Resolución de Métodos (MRO - *Method Resolution Order*)**, accesible a través del atributo `__mro__`.

Al reescribir el código utilizando `super()`:

```python
class LeftSubclass_S(BaseClass):
    num_left_calls = 0

    def call_me(self) -> None:
        super().call_me()
        print("Calling method on LeftSubclass_S")
        self.num_left_calls += 1

class RightSubclass_S(BaseClass):
    num_right_calls = 0

    def call_me(self) -> None:
        super().call_me()
        print("Calling method on RightSubclass_S")
        self.num_right_calls += 1

class Subclass_S(LeftSubclass_S, RightSubclass_S):
    num_sub_calls = 0

    def call_me(self) -> None:
        super().call_me()
        print("Calling method on Subclass_S")
        self.num_sub_calls += 1
```

Resultado de la ejecución con `super()`:

```python
>>> ss = Subclass_S()
>>> ss.call_me()
Calling method on BaseClass
Calling method on RightSubclass_S
Calling method on LeftSubclass_S
Calling method on Subclass_S
>>> print(
...     ss.num_sub_calls,
...     ss.num_left_calls,
...     ss.num_right_calls,
...     ss.num_base_calls
... )
1 1 1 1
```

Inspeccionando el MRO:

```python
>>> from pprint import pprint
>>> pprint(Subclass_S.__mro__)
(<class 'diamond.Subclass_S'>,
 <class 'diamond.LeftSubclass_S'>,
 <class 'diamond.RightSubclass_S'>,
 <class 'diamond.BaseClass'>,
 <class 'object'>)
```

`super()` no invoca necesariamente al padre directo en el árbol sintáctico, sino al **siguiente elemento en la secuencia del MRO**, asegurando que cada método en la jerarquía se ejecute exactamente una sola vez.

#### 3.4.2 Diferentes conjuntos de argumentos en llamadas cooperativas

Para propagar argumentos distintos a través de la cadena del MRO al usar `super()`, utilizamos argumentos por palabra clave mediante `**kwargs`:

```python
class Contact:
    all_contacts = ContactList()

    def __init__(self, /, name: str = "", email: str = "", **kwargs: Any) -> None:
        super().__init__(**kwargs)  # type: ignore [call-arg]
        self.name = name
        self.email = email
        self.all_contacts.append(self)

    def __repr__(self) -> str:
        return f"Contact({self.name!r}, {self.email!r})"
```

```python
class AddressHolder:
    def __init__(
        self,
        /,
        street: str = "",
        city: str = "",
        state: str = "",
        code: str = "",
        **kwargs: Any,
    ) -> None:
        super().__init__(**kwargs)  # type: ignore [call-arg]
        self.street = street
        self.city = city
        self.state = state
        self.code = code
```

```python
class Friend(Contact, AddressHolder):
    def __init__(self, /, phone: str = "", **kwargs: Any) -> None:
        super().__init__(**kwargs)
        self.phone = phone
```

Cada clase consume los parámetros que le corresponden y propaga el resto (`**kwargs`) a la siguiente clase en el MRO mediante `super().__init__(**kwargs)`.

---

### Sección 3.5: Polimorfismo

El **polimorfismo** describe un concepto fundamental: diferentes comportamientos se ejecutan según la subclase utilizada, sin necesidad de que el código consumidor conozca explícitamente qué subclase específica es. Esto se formaliza mediante el **Principio de Sustitución de Liskov**.

Consideremos un reproductor de archivos de audio que procesa distintos formatos (`.mp3`, `.wav`, `.ogg`):

```python
from pathlib import Path
import abc

class AudioFile(abc.ABC):
    ext: str

    def __init__(self, filepath: Path) -> None:
        if not filepath.suffix == self.ext:
            raise ValueError(f"invalid file format {filepath.suffix!r}")
        self.filepath = filepath

    @abc.abstractmethod
    def play(self) -> None:
        ...
```

Subclases concretas:

```python
class MP3File(AudioFile):
    ext = ".mp3"

    def play(self) -> None:
        print(f"playing {self.filepath} as mp3")

class WavFile(AudioFile):
    ext = ".wav"

    def play(self) -> None:
        print(f"playing {self.filepath} as wav")

class OggFile(AudioFile):
    ext = ".ogg"

    def play(self) -> None:
        print(f"playing {self.filepath} as ogg")
```

Uso polimórfico:

```python
>>> p_1 = MP3File(Path("Heart of the Sunrise.mp3"))
>>> p_1.play()
playing Heart of the Sunrise.mp3 as mp3
>>> p_2 = WavFile(Path("Roundabout.wav"))
>>> p_2.play()
playing Roundabout.wav as wav
>>> p_3 = OggFile(Path("Heart of the Sunrise.ogg"))
>>> p_3.play()
playing Heart of the Sunrise.ogg as ogg
>>> error = MP3File(Path("The Fish.mov"))
Traceback (most recent call last):
  ...
ValueError: invalid file format '.mov'
```

#### Tipado de pato (*Duck Typing*)

Python facilita aún más el polimorfismo gracias al **duck typing** (*"si camina como un pato y grazna como un pato, es un pato"*). No es obligatorio heredar de una clase base común: cualquier clase que implemente los métodos esperados puede utilizarse indistintamente:

```python
class FlacFile:
    def __init__(self, filepath: Path) -> None:
        if not filepath.suffix == ".flac":
            raise ValueError("Not a .flac file")
        self.filepath = filepath

    def play(self) -> None:
        print(f"playing {self.filepath} as flac")
```

Podemos formalizar este contrato utilizando `typing.Protocol`:

```python
from typing import Protocol

class Playable(Protocol):
    def play(self) -> None:
        ...
```

O definiendo una unión de tipos:

```python
type Playable = AudioFile | FlacFile
```

---

### Sección 3.6: Repaso

Puntos clave tratados en este capítulo:

- **Herencia:** Permite reutilizar lógica común en una superclase y especializarla en subclases, respetando la relación "es-un".
- **Herencia múltiple y Mixins:** Python permite heredar de múltiples clases. Los mixins encapsulan comportamientos reutilizables y el MRO gestiona el orden de ejecución mediante `super()`.
- **Polimorfismo y Duck Typing:** Permite interactuar con objetos a través de una interfaz común sin depender de su clase concreta.

---

### Sección 3.7: Ejercicios

1. **Taxonomías del mundo real:** Modela una jerarquía de herencia para objetos de tu entorno de trabajo. Identifica qué atributos y métodos compartirían en una superclase y cuáles requerirían sobrescritura polimórfica.
2. **Prototipo con Mixins:** Diseña un sistema donde utilices clases mixin para proporcionar capacidades auxiliares (por ejemplo, serialización a JSON, registro en logs o validaciones).
3. **Refactorización del Capítulo 1:** Analiza el script de procesamiento de pruebas JSON del Capítulo 1. ¿Cómo puede la herencia o los protocolos estructurar los diferentes tipos de comandos y resultados de ejecución?

---

### Sección 3.8: Resumen

Hemos explorado desde la herencia simple hasta la herencia múltiple, la resolución del problema del diamante mediante `super()` y el MRO, y el diseño polimórfico asistido por *duck typing* y protocolos.

En el próximo capítulo, abordaremos el manejo de situaciones excepcionales y el diseño de excepciones personalizadas.
