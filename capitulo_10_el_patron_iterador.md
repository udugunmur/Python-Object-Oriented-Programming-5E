# Parte 3: Patrones de Diseño y Buenas Prácticas
## Capítulo 10: El Patrón Iterador

Hemos analizado cómo muchas de las funciones integradas e modismos de Python parecen, a primera vista, contradecir los principios de la programación orientada a objetos, cuando en realidad proporcionan acceso a los objetos mediante una sintaxis funcional elegante. En este capítulo, descubriremos cómo la instrucción `for` es en realidad un envoltorio ligero sobre un conjunto de patrones de diseño orientados a objetos. También examinaremos extensiones sintácticas que crean diversos tipos de objetos automáticamente:

- Qué son los **patrones de diseño**.
- El **protocolo de iterador**: uno de los patrones de diseño más potentes y ubicuos.
- Comprensiones de listas, conjuntos y diccionarios.
- **Funciones generadoras** y expresiones generadoras (*lazy evaluation*).

---

### Sección 10.1: Patrones de diseño en resumen

Cuando los ingenieros y arquitectos deciden construir un puente o un rascacielos, siguen principios rigurosos para garantizar la integridad estructural. Existen diversos diseños estandarizados para puentes (colgantes, en voladizo, etc.); si un ingeniero prescinde de estos patrones sin un diseño alternativo fundamentado, la estructura colapsará.

Los **patrones de diseño** aplican esta misma formalización a la ingeniería de software: son soluciones generales y reutilizables a problemas comunes que los desarrolladores enfrentan en situaciones específicas.

Un patrón de diseño propone un conjunto de objetos que interactúan de una manera determinada para resolver un problema general. La tarea del desarrollador consiste en reconocer el problema y adaptar el patrón general a sus necesidades precisas.

---

### Sección 10.2: Iteradores

Conceptualmente, un iterador es un objeto con un método para obtener el siguiente elemento (`next()`) y otro para comprobar si la secuencia ha terminado (`done()`):

```python
iterator = some_collection.iterator() 
while not iterator.done(): 
    item = iterator.next() 
    # do something with the item from some_collection...
```

En Python, la iteración está profundamente integrada en la sintaxis:
- El método para obtener el siguiente elemento se denomina **`__next__()`** (accesible mediante la función integrada `next(iterator)`).
- En lugar de un método `done()`, el protocolo de iteradores de Python lanza la excepción **`StopIteration`** cuando no quedan más datos.
- La sentencia **`for item in iterator:`** abstrae por completo el bucle `while` manual y la captura de excepciones.

#### 10.2.1 El protocolo de iterador

La clase base abstracta `Iterator` en `collections.abc` define el contrato formal:

> **Figura 10.1: Abstracciones para `Iterable` e `Iterator`**  
> `Iterable` (`__iter__()` ──▷ `Iterator`) ──▷ `Iterator` (`__next__()` + `__iter__()`)

Todo objeto **Iterable** implementa `__iter__()`, devolviendo una instancia de `Iterator`. A su vez, todo `Iterator` implementa `__next__()` y `__iter__()` (devolviéndose a sí mismo).

Implementación explícita mediante clases:

```python
from typing import Iterable, Iterator 
 
class CapitalIterable(Iterable[str]): 
    def __init__(self, string: str) -> None: 
        self.string = string 
 
    def __iter__(self) -> Iterator[str]: 
        return CapitalIterator(self.string) 
 
class CapitalIterator(Iterator[str]): 
    def __init__(self, string: str) -> None: 
        self.words = [w.capitalize() for w in string.split()] 
        self.index = 0 
 
    def __next__(self) -> str: 
        if self.index == len(self.words): 
            raise StopIteration() 
 
        word = self.words[self.index] 
        self.index += 1 
        return word
```

Uso interactivo mediante bucle manual `while` + `StopIteration`:

```python
>>> iterable = CapitalIterable('the quick brown fox jumps over the lazy dog')
>>> iterator = iter(iterable)
>>> while True:
...     try:
...         print(next(iterator))
...     except StopIteration:
...         break
...
The
Quick
Brown
Fox
Jumps
Over
The
Lazy
Dog
```

Uso idiomático mediante bucle `for`:

```python
>>> for i in iterable: 
...     print(i) 
... 
The 
Quick 
Brown 
Fox 
Jumps 
Over 
The 
Lazy 
Dog
```

En Python, prácticamente cualquier contenedor o flujo de datos es iterable: cadenas, listas, tuplas, conjuntos, diccionarios, archivos de texto, rangos (`range`) y generadores.

---

### Sección 10.3: Comprensiones

Las comprensiones proporcionan una sintaxis concisa y de alto rendimiento para transformar y filtrar objetos iterables en una sola línea.

#### 10.3.1 Comprensiones de listas (*List Comprehensions*)

Transformación tradicional con bucle `for`:

```python
>>> input_strings = ["1", "5", "28", "131", "3"] 
>>> output_integers = [] 
>>> for num in input_strings: 
...    output_integers.append(int(num))
```

Transformación equivalente y optimizada mediante comprensión de lista:

```python
>>> output_integers = [int(num) for num in input_strings]
```

Filtrado mediante cláusula `if`:

```python
>>> output_integers = [int(num) for num in input_strings if len(num) < 3] 
>>> output_integers 
[1, 5, 28, 3]
```

Procesamiento de líneas de un archivo de texto con eliminación de espacios en blanco:

```python
from pathlib import Path

source_path = Path('src') / 'iterator_protocol.py' 
with source_path.open() as source: 
    examples = [line.rstrip() 
        for line in source 
        if ">>>" in line]
```

Combinación con `enumerate()` para indexar números de línea:

```python
source_path = Path('src') / 'iterator_protocol.py' 
with source_path.open() as source: 
    examples = [(number, line.rstrip()) 
        for number, line in enumerate(source, start=1) 
        if ">>>" in line]
```

#### 10.3.2 Comprensiones de conjuntos y diccionarios

Dada una colección de libros representados con `NamedTuple`:

```python
from typing import NamedTuple 
 
class Book(NamedTuple): 
    author: str 
    title: str 
    genre: str

books = [ 
    Book("Pratchett", "Nightwatch", "fantasy"), 
    Book("Pratchett", "Thief Of Time", "fantasy"), 
    Book("Le Guin", "The Dispossessed", "scifi"), 
    Book("Le Guin", "A Wizard Of Earthsea", "fantasy"), 
    Book("Jemisin", "The Broken Earth", "fantasy"), 
    Book("Turner", "The Thief", "fantasy"), 
    Book("Phillips", "Preston Diamond", "western"), 
    Book("Phillips", "Twice Upon A Time", "scifi"), 
]
```

Comprensión de conjuntos (elimina duplicados automáticamente):

```python
>>> fantasy_authors = {b.author for b in books if b.genre == "fantasy"}
>>> fantasy_authors
{'Jemisin', 'Le Guin', 'Pratchett', 'Turner'}
```

Comprensión de diccionarios (crea un índice por título):

```python
>>> fantasy_titles = {b.title: b for b in books if b.genre == "fantasy"}
```

#### 10.3.3 Expresiones generadoras (*Generator Expressions*)

Cuando procesamos volúmenes masivos de datos (como archivos de log de varios gigabytes), almacenar todos los elementos en memoria en una lista resulta ineficiente.

Las **expresiones generadoras** utilizan paréntesis `()` y operan bajo demanda (**evaluación perezosa** o *lazy evaluation*), manteniendo en memoria únicamente el elemento en curso:

```python
from pathlib import Path 

full_log_path = Path.cwd() / "data" / "sample.log" 
warning_log_path = Path.cwd() / "data" / "warnings.tab" 
 
with open(full_log_path) as source: 
    warning_lines = (line for line in source if "WARN" in line) 
    with open(warning_log_path, 'w') as target: 
        for line in warning_lines: 
            target.write(line)
```

Salida resultante en el archivo de destino:

```text
Apr 05, 2021 20:03:53 WARNING This is a warning. It could be serious. 
Apr 05, 2021 20:03:59 WARNING Another warning sent. 
Apr 05, 2021 20:04:35 WARNING Warnings should be heeded. 
Apr 05, 2021 20:04:41 WARNING Watch for warnings.
```

---

### Sección 10.4: Funciones generadoras (`yield`)

Cuando la lógica de transformación requiere múltiples sentencias, manejo de excepciones (`try...except`) o comprobaciones complejas, creamos una **función generadora** utilizando la palabra clave **`yield`**.

Comparemos tres enfoques para procesar un archivo de log y exportar un CSV:

#### Enfoque 1: Procedural con anidamiento profundo

```python
import csv 
import re 
from pathlib import Path 
from typing import Match, cast 
 
def extract_and_parse_1(source_log_path: Path, warning_tab_path: Path) -> None: 
    with warning_tab_path.open("w", newline="") as target: 
        writer = csv.writer(target, delimiter="\t") 
        pattern = re.compile(r"(\w\w\w \d\d, \d\d\d\d \d\d:\d\d:\d\d) (\w+) (.*)") 
        with source_log_path.open() as source: 
            for line in source: 
                if line_match := pattern.match(line): 
                    line_groups = line_match.groups() 
                    if "WARN" in line_groups[1]: 
                        writer.writerow(line_groups)
```

#### Enfoque 2: Orientado a objetos tradicional mediante clase `Iterator`

```python
import csv 
import re 
from pathlib import Path 
from typing import Match, cast, Iterator, TextIO 
 
class WarningReformat(Iterator[tuple[str, ...]]): 
    pattern = re.compile(r"(\w\w\w \d\d, \d\d\d\d \d\d:\d\d:\d\d) (\w+) (.*)") 
 
    def __init__(self, source: TextIO) -> None: 
        self.insequence = source 
 
    def __iter__(self) -> Iterator[tuple[str, ...]]: 
        return self 
 
    def __next__(self) -> tuple[str, ...]: 
        line = self.insequence.readline() 
        while line: 
            if match := self.pattern.match(line): 
                groups = match.groups() 
                if "WARN" in groups[1]: 
                    return groups 
            line = self.insequence.readline() 
        raise StopIteration

def extract_and_parse_2(full_log_path: Path, warning_log_path: Path) -> None: 
    with warning_log_path.open("w", newline="") as target: 
        writer = csv.writer(target, delimiter="\t") 
        with full_log_path.open() as source: 
            filter_reformat = WarningReformat(source) 
            for line_groups in filter_reformat: 
                writer.writerow(line_groups)
```

#### Enfoque 3: Función generadora con `yield`

```python
import csv 
import re 
from pathlib import Path 
from typing import Match, cast, Iterator, Iterable 
 
def warnings_filter(source: Iterable[str]) -> Iterator[tuple[str, ...]]: 
    pattern = re.compile(r"(\w\w\w \d\d, \d\d\d\d \d\d:\d\d:\d\d) (\w+) (.*)") 
    for line in source: 
        if "WARN" in line: 
            yield tuple(cast(Match[str], pattern.match(line)).groups()) 
 
def extract_and_parse_3(full_log_path: Path, warning_log_path: Path) -> None: 
    with warning_log_path.open("w", newline="") as target: 
        writer = csv.writer(target, delimiter="\t") 
        with full_log_path.open() as infile: 
            filter = warnings_filter(infile) 
            for line_groups in filter: 
                writer.writerow(line_groups)
```

Al incluir `yield`, Python convierte automáticamente la función en una fábrica de objetos generadores que suspenden y reanudan su ejecución preservando su estado local:

```python
>>> print(warnings_filter([])) 
<generator object warnings_filter at 0xb728c6bc>
```

Correspondencia sintáctica:

> **Figura 10.2: Comparación entre funciones generadoras y expresiones generadoras**  
> `def gen(source): for x in source: if cond(x): yield expr(x)`  
> $\Longleftrightarrow$  
> `(expr(x) for x in source if cond(x))`

#### 10.4.1 Delegación de iterables con `yield from`

`yield from` permite delegar la emisión de valores a otro iterable o subgenerador de forma transparente:

```python
import csv 
import re 
from pathlib import Path 
from typing import Match, cast, Iterator, Iterable 
 
def file_extract(path_iter: Iterable[Path]) -> Iterator[tuple[str, ...]]: 
    for path in path_iter: 
        with path.open() as infile: 
            yield from warnings_filter(infile) 
 
def extract_and_parse_d(directory: Path, warning_log_path: Path) -> None: 
    with warning_log_path.open("w", newline="") as target: 
        writer = csv.writer(target, delimiter="\t") 
        log_files = list(directory.glob("sample*.log")) 
        for line_groups in file_extract(log_files): 
            writer.writerow(line_groups)
```

#### 10.4.2 Pilas y canalizaciones de generadores (*Generator Stacks / Pipelines*)

Podemos estructurar transformaciones complejas encadenando múltiples expresiones generadoras desacopladas en una canalización (*pipeline*) perezosa:

```python
import datetime
import re

pattern = re.compile( 
    r"(?P<dt>\w\w\w \d\d, \d\d\d\d \d\d:\d\d:\d\d)" 
    r"\s+(?P<level>\w+)" 
    r"\s+(?P<msg>.*)" 
)

possible_match_iter = (pattern.match(line) for line in source) 
group_iter = (match.groupdict() for match in possible_match_iter if match) 
warnings_iter = (group for group in group_iter if "WARN" in group["level"]) 
dt_iter = ( 
    ( 
        datetime.datetime.strptime(g["dt"], "%b %d, %Y %H:%M:%S"), 
        g["level"], 
        g["msg"], 
    ) 
    for g in warnings_iter 
) 
warnings_filter = ((g[0].isoformat(), g[1], g[2]) for g in dt_iter)
```

Canalización equivalente utilizando `map()`, `filter()` y funciones `lambda`:

```python
possible_match_iter = map(pattern.match, source) 
good_match_iter = filter(None, possible_match_iter) 
group_iter = map(lambda m: m.groupdict(), good_match_iter) 
warnings_iter = filter(lambda g: "WARN" in g["level"], group_iter) 
dt_iter = map( 
    lambda g: ( 
        datetime.datetime.strptime(g["dt"], "%b %d, %Y %H:%M:%S"), 
        g["level"], 
        g["msg"], 
    ), 
    warnings_iter, 
) 
warnings_filter = map(lambda g: (g[0].isoformat(), g[1], g[2]), dt_iter)
```

---

### Sección 10.5: Repaso

Puntos clave tratados en este capítulo:

- **Patrones de diseño:** Soluciones arquitectónicas consolidadas y reutilizables ante problemas recurrentes de software.
- **Protocolo de Iterador:** Compuesto por `Iterable` (`__iter__()`) e `Iterator` (`__next__()` y `StopIteration`).
- **Comprensiones:** Sintaxis compacta y optimizada para mapear y filtrar listas (`[]`), conjuntos (`{}`) y diccionarios (`{k: v}`).
- **Expresiones y funciones generadoras (`yield`, `yield from`):** Procesamiento perezoso elemento por elemento con uso constante de memoria $\mathcal{O}(1)$.
- **Canalizaciones de generadores:** Composición de etapas discretas de mapeo y filtrado sin almacenar resultados intermedios.

---

### Sección 10.6: Ejercicios

1. **Benchmark de rendimiento con `timeit`:** Compara el tiempo de ejecución y consumo de memoria entre un bucle `for` tradicional con `.append()`, una comprensión de listas y una expresión generadora sobre un millón de elementos.
2. **Generadores infinitos:** Implementa un generador que produzca la secuencia de Fibonacci bajo demanda sin límite superior.
3. **Filtros combinados de logs:** Modela una canalización de generadores que filtre entradas de registro simultáneamente por nivel de severidad y por ventana temporal de fechas.

---

### Sección 10.7: Resumen

En este capítulo analizamos en profundidad el patrón iterador en Python, desde los fundamentos del protocolo `__iter__` y `__next__` hasta las comprensiones y las funciones generadoras basadas en `yield`.

En el próximo capítulo, exploraremos los **patrones de diseño comunes** en Python: Decorator, Observer, Strategy, Command, State y Singleton.
