# Parte 4: Calidad de Software, Pruebas y Concurrencia
## Capítulo 14: Concurrencia

La concurrencia es el arte de hacer que una computadora realice (o aparente realizar) múltiples tareas a la vez. Históricamente, esto implicaba alternar rápidamente entre diferentes tareas muchas veces por segundo en un único procesador. En sistemas modernos, también significa ejecutar dos o más tareas simultáneamente en núcleos de procesamiento independientes.

Aunque la concurrencia no es un concepto estrictamente orientado a objetos, las bibliotecas y herramientas de concurrencia en Python exponen interfaces orientadas a objetos.

En este capítulo exploraremos:

- Hilos de ejecución (**`threading`**)
- Multiprocesamiento (**`multiprocessing`**)
- Futuros y promesas (**`concurrent.futures`**)
- Programación asíncrona no bloqueante (**`asyncio`**)
- El benchmark clásico de los **filósofos comensales** (*Dining Philosophers*)

---

### Sección 14.1: Fundamentos del procesamiento concurrente

Podemos visualizar el procesamiento concurrente como un grupo de trabajadores que colaboran sin verse directamente, coordinándose mediante mensajes, colas, tokens o bloqueos compartidos.

El problema fundamental de la programación concurrente radica en **prevenir cambios de estado inesperados** y **condiciones de carrera** (*race conditions*) cuando múltiples entidades intentan acceder o mutar los mismos recursos compartidos al mismo tiempo.

Mecanismos de concurrencia disponibles en Python:
1. **Multiprocesamiento (`multiprocessing`):** Múltiples procesos del sistema operativo independientes, cada uno con su propio intérprete de Python, memoria aislada y su propio GIL. Ideal para tareas intensivas en CPU (*CPU-bound*).
2. **Hilos (`threading`):** Múltiples hilos dentro del mismo proceso que comparten espacio de memoria. Sujeto al bloqueo global del intérprete (*Global Interpreter Lock*, GIL). Ideal para operaciones con esperas de entrada/salida (*I/O-bound*).
3. **Futuros (`concurrent.futures`):** Abstracción de alto nivel para gestionar ejecuciones asíncronas basadas en fondos de hilos (*ThreadPoolExecutor*) o fondos de procesos (*ProcessPoolExecutor*).
4. **Asincronía (`asyncio`):** Multitarea cooperativa no apropiativa basada en corrutinas (`async`/`await`) gestionadas por un bucle de eventos (*event loop*) en un único hilo.

---

### Sección 14.2: Hilos (*Threads*)

Un hilo representa una secuencia de instrucciones de bytecode de Python que puede pausarse y reanudarse. Permite que la aplicación continúe procesando mientras espera operaciones de entrada/salida (disco, red o usuario).

#### 14.2.1 Ejemplo básico con `threading.Thread`

```python
class Chef(Thread):
    def __init__(self, name: str) -> None:
        super().__init__(name=name)
        self.total = 0

    def get_order(self) -> None:
        self.order = THE_ORDERS.pop(0)

    def prepare(self) -> None:
        """Simulate doing a lot of work with a BIG computation"""
        start = time.monotonic()
        target = start + 1 + random.random()
        for i in range(1_000_000_000):
            self.total += math.factorial(i)
            if time.monotonic() >= target:
                break
        print(f"{time.monotonic():.3f} {self.name} made {self.order}")

    def run(self) -> None:
        while True:
            try:
                self.get_order()
                self.prepare()
            except IndexError:
                break  # No more orders
```

Definición de pedidos compartidos y ejecución:

```python
import math
import random
from threading import Thread
import time

THE_ORDERS = [
    "Reuben",
    "Ham and Cheese",
    "Monte Cristo",
    "Tuna Melt",
    "Cuban",
    "Grilled Cheese",
    "French Dip",
    "BLT",
]

Mo = Chef("Michael")
Constantine = Chef("Constantine")

if __name__ == "__main__":
    random.seed(42)
    Mo.start()
    Constantine.start()
```

Salida de ejecución:

```text
1.076 Constantine made Ham and Cheese
1.676 Michael made Reuben
2.351 Constantine made Monte Cristo
2.899 Michael made Tuna Melt
4.094 Constantine made Cuban
4.576 Michael made Grilled Cheese
5.664 Michael made BLT
5.987 Constantine made French Dip
```

#### 14.2.2 Desafíos y limitaciones de los hilos

- **Memoria compartida y sincronización:** El acceso concurrente no coordinado a variables compartidas provoca estados corruptos. Se deben usar cerrojos (`threading.Lock`) o colas sincronizadas (`queue.Queue`).
- **El Bloqueo Global del Intérprete (GIL):** En CPython, el GIL asegura que un solo hilo ejecute bytecode a la vez, impidiendo el paralelismo real de tareas intensivas en CPU sobre múltiples núcleos.
- **Sobrecarga de gestión de hilos (*Thread Overhead*):** Cada hilo consume memoria del kernel y ciclos de cambio de contexto (*context switching*).

---

### Sección 14.3: Multiprocesamiento (*Multiprocessing*)

El módulo `multiprocessing` elude el GIL creando procesos independientes a nivel de sistema operativo, permitiendo paralelismo real en arquitecturas multinúcleo.

#### 14.3.1 Ejemplo básico con `multiprocessing.Process`

```python
from multiprocessing import Process, cpu_count
from threading import Thread
import time
import os

class MuchCPU(Process):
    def run(self) -> None:
        print(f"OS PID {os.getpid()}")
        _ = sum(2 * i + 1 for i in range(100_000_000))

if __name__ == "__main__":
    workers = [MuchCPU() for f in range(cpu_count())]

    t = time.perf_counter()
    for p in workers:
        p.start()
    for p in workers:
        p.join()
    print(f"work took {time.perf_counter() - t:.3f} seconds")
```

> [!IMPORTANT]
> El bloque `if __name__ == "__main__":` es obligatorio al trabajar con `multiprocessing` para evitar la creación recursiva infinita de procesos al importar el módulo en los procesos hijos.

Comparación de rendimiento en CPU multinúcleo:
- Con `multiprocessing.Process`: ~20.7 segundos (8 núcleos al 100% en paralelo).
- Con `threading.Thread`: ~69.3 segundos (ejecución serializada por el GIL).

#### 14.3.2 Fondos de procesos (*Multiprocessing Pools*)

Cálculo paralelo de factores primos:

```python
from math import sqrt, ceil
import random
from multiprocessing.pool import Pool

def prime_factors(value: int) -> list[int]:
    if value in {2, 3}:
        return [value]
    factors: list[int] = []
    for divisor in range(2, ceil(sqrt(value)) + 1):
        quotient, remainder = divmod(value, divisor)
        if not remainder:
            factors.extend(prime_factors(divisor))
            factors.extend(prime_factors(quotient))
            break
    else:
        factors = [value]
    return factors

if __name__ == "__main__":
    to_factor = [random.randint(100_000_000, 1_000_000_000) for i in range(40_960)]
    with Pool() as pool:
        results = pool.map(prime_factors, to_factor)
    primes = [
        value for value, factor_list in zip(to_factor, results) if len(factor_list) == 1
    ]
    print(f"9-digit primes {primes}")
```

#### 14.3.3 Comunicación interprocesos mediante colas (`multiprocessing.Queue`)

Buscador concurrente de código fuente en disco:

```python
from collections.abc import Iterator
from pathlib import Path
from multiprocessing import Queue
import os

type Query_Q = Queue[str | None]
type Result_Q = Queue[list[str]]

def search(paths: list[Path], query_q: Query_Q, results_q: Result_Q) -> None:
    print(f"PID: {os.getpid()}, paths {len(paths)}")
    lines: list[str] = []
    for path in paths:
        lines.extend(
            line.rstrip() for line in path.read_text().splitlines()
        )
    while True:
        if (query_text := query_q.get()) is None:
            break
        results = [line for line in lines if query_text in line]
        results_q.put(results)
```

Fachada `DirectorySearch` para coordinar colas y procesos hijos:

```python
from fnmatch import fnmatch
import os
from multiprocessing import Process, cpu_count

class DirectorySearch:
    def __init__(self) -> None:
        self.query_queues: list[Query_Q]
        self.results_queue: Result_Q
        self.search_workers: list[Process]

    def setup_search(self, paths: list[Path], cpus: int | None = None) -> None:
        if cpus is None:
            cpus = cpu_count()
        worker_paths = [paths[i::cpus] for i in range(cpus)]
        self.query_queues = [Queue() for p in range(cpus)]
        self.results_queue = Queue()
        self.search_workers = [
            Process(target=search, args=(paths, q, self.results_queue))
            for paths, q in zip(worker_paths, self.query_queues)
        ]
        for proc in self.search_workers:
            proc.start()

    def teardown_search(self) -> None:
        # Signal process termination
        for q in self.query_queues:
            q.put(None)
        for proc in self.search_workers:
            proc.join()

    def search(self, target: str) -> Iterator[str]:
        for q in self.query_queues:
            q.put(target)
        for i in range(len(self.query_queues)):
            for match in self.results_queue.get():
                yield match
```

Función auxiliar para localizar archivos y script principal:

```python
def all_source(path: Path, pattern: str) -> Iterator[Path]:
    for root, dirs, files in os.walk(path):
        for skip in {".tox", ".mypy_cache", "__pycache__", ".idea", ".venv"}:
            if skip in dirs:
                dirs.remove(skip)
        yield from (Path(root) / f for f in files if fnmatch(f, pattern))
```

```python
from multiprocessing import Process, Queue, cpu_count
import time

if __name__ == "__main__":
    ds = DirectorySearch()
    base = Path.cwd().parent
    all_paths = list(all_source(base, "*.py"))
    ds.setup_search(all_paths)
    for target in ("import", "class", "def"):
        start = time.perf_counter()
        count = 0
        for line in ds.search(target):
            count += 1
        milliseconds = 1000 * (time.perf_counter() - start)
        print(
            f"Found {count} {target!r} in {len(all_paths)} files "
            f"in {milliseconds:.3f}ms"
        )
    ds.teardown_search()
```

Salida:

```text
PID: 29387, paths 19
PID: 29389, paths 19
PID: 29388, paths 19
PID: 29390, paths 19
PID: 29391, paths 19
PID: 29392, paths 19
PID: 29393, paths 19
PID: 29394, paths 18
Found 611 'import' in 151 files in 190.105ms
Found 464 'class' in 151 files in 1.293ms
Found 1036 'def' in 151 files in 1.330ms
```

---

### Sección 14.4: Futuros (*Concurrent Futures*)

El módulo `concurrent.futures` proporciona una API uniforme y elegante basada en objetos `Future`, permitiendo intercambiar fácilmente entre hilos y procesos.

#### 14.4.1 Análisis de AST mediante `ThreadPoolExecutor`

Análisis sintáctico de declaraciones `import`:

```python
import ast
from pathlib import Path
from typing import NamedTuple

class ImportResult(NamedTuple):
    path: Path
    imports: set[str]
    @property
    def focus(self) -> bool:
        return "typing" in self.imports

class ImportVisitor(ast.NodeVisitor):
    def __init__(self) -> None:
        self.imports: set[str] = set()

    def visit_Import(self, node: ast.Import) -> None:
        for alias in node.names:
            self.imports.add(alias.name)

    def visit_ImportFrom(self, node: ast.ImportFrom) -> None:
        if node.module:
            self.imports.add(node.module)

def find_imports(path: Path) -> ImportResult:
    tree = ast.parse(path.read_text())
    iv = ImportVisitor()
    iv.visit(tree)
    return ImportResult(path, iv.imports)
```

Distribución del trabajo concurrente con `concurrent.futures.as_completed`:

```python
from concurrent import futures
import time

def main(base: Path = Path.cwd()) -> None:
    print(f"\n{base}")
    start = time.perf_counter()
    with futures.ThreadPoolExecutor(24) as pool:
        analyzers = [
            pool.submit(find_imports, path) for path in all_source(base, "*.py")
        ]
        analyzed = (worker.result() for worker in futures.as_completed(analyzers))
        for example in sorted(analyzed):
            print(
                f"{'->' if example.focus else '':2s} "
                f"{example.path.relative_to(base)} {example.imports}"
            )
    end = time.perf_counter()
    rate = 1000 * (end - start) / len(analyzers)
    print(f"Searched {len(analyzers)} files in {base} at {rate:.3f}ms/file")
```

---

### Sección 14.5: AsyncIO (Asincronía y Corrutinas)

`asyncio` implementa **multitarea cooperativa** en un único hilo mediante un bucle de eventos (*Event Loop*) y corrutinas declaradas con `async def` y pausadas con `await`.

#### 14.5.1 Ejemplo básico de tareas concurrentes

```python
import asyncio
import random

async def random_sleep(counter: int) -> None:
    delay = random.random() * 5
    print(f"{counter} sleeps for {delay:.2f} seconds")
    await asyncio.sleep(delay)
    print(f"{counter} awakens, refreshed")

async def sleepers(how_many: int = 5) -> None:
    print(f"Creating {how_many} tasks")
    tasks = [asyncio.create_task(random_sleep(i)) for i in range(how_many)]
    print(f"Waiting for {how_many} tasks")
    await asyncio.gather(*tasks)

if __name__ == "__main__":
    asyncio.run(sleepers(5))
    print("Done with the sleepers")
```

Salida de intercalación asíncrona:

```text
Creating 5 tasks
Waiting for 5 tasks
0 sleeps for 4.69 seconds
1 sleeps for 1.59 seconds
2 sleeps for 4.57 seconds
3 sleeps for 3.45 seconds
4 sleeps for 0.77 seconds
4 awakens, refreshed
1 awakens, refreshed
3 awakens, refreshed
2 awakens, refreshed
0 awakens, refreshed
Done with the sleepers
```

#### 14.5.2 Servidor de red asíncrono de alto rendimiento

Recepción asíncrona de tramas y delegación de escritura bloqueante a hilos con `asyncio.to_thread`:

```python
import asyncio
import asyncio.exceptions
import json
from pathlib import Path
import pickle
import struct
from typing import TextIO

SIZE_FORMAT = ">L"
SIZE_BYTES = struct.calcsize(SIZE_FORMAT)

TARGET: TextIO
LINE_COUNT = 0

def serialize(bytes_payload: bytes) -> str:
    object_payload = pickle.loads(bytes_payload)
    text_message = json.dumps(object_payload)
    TARGET.write(text_message)
    TARGET.write("\n")
    return text_message

async def log_writer(bytes_payload: bytes) -> None:
    global LINE_COUNT
    LINE_COUNT += 1
    await asyncio.to_thread(serialize, bytes_payload)

async def log_catcher(
    reader: asyncio.StreamReader, writer: asyncio.StreamWriter
) -> None:
    count = 0
    client_socket = writer.get_extra_info("socket")
    size_header = await reader.read(SIZE_BYTES)
    while size_header:
        payload_size = struct.unpack(SIZE_FORMAT, size_header)
        bytes_payload = await reader.read(payload_size[0])
        await log_writer(bytes_payload)
        count += 1
        size_header = await reader.read(SIZE_BYTES)
    print(f"From {client_socket.getpeername()}: {count} lines")
```

Inicialización del servidor con `asyncio.start_server`:

```python
server: asyncio.AbstractServer

async def main(host: str, port: int) -> None:
    global server
    server = await asyncio.start_server(
        log_catcher,
        host=host,
        port=port,
    )
    if server.sockets:
        addr = server.sockets[0].getsockname()
        print(f"Serving on {addr}")
    else:
        raise ValueError("Failed to create server")
    async with server:
        await server.serve_forever()
    server.close_clients()

if __name__ == "__main__":
    HOST, PORT = "localhost", 18842
    with Path("one.log").open("w") as TARGET:
        try:
            asyncio.run(main(HOST, PORT))
        except (asyncio.exceptions.CancelledError, KeyboardInterrupt):
            ending = {"lines_collected": LINE_COUNT}
            print(ending)
            TARGET.write(json.dumps(ending) + "\n")
```

#### 14.5.3 Clientes asíncronos con `httpx`

Consulta concurrente de múltiples previsiones meteorológicas:

```python
import asyncio
import re
import time
from typing import NamedTuple
import httpx

class Zone(NamedTuple):
    zone_name: str
    zone_code: str
    same_code: str

    @property
    def forecast_url(self) -> str:
        return (
            f"https://tgftp.nws.noaa.gov/data/forecasts"
            f"/marine/coastal/an/{self.zone_code.lower()}.txt"
        )

ZONES = [
    Zone("Chesapeake Bay from Pooles Island to Sandy Point, MD", "ANZ531", "073531"),
    Zone("Chesapeake Bay from Sandy Point to North Beach, MD", "ANZ532", "073532"),
    Zone("Chesapeake Bay from North Beach to Drum Point, MD", "ANZ533", "073533"),
    Zone(
        "Tangier Sound and the Inland Waters surrounding Bloodsworth Island",
        "ANZ543",
        "073543",
    ),
]

class MarineWX:
    advisory_pat = re.compile(r"\n\.\.\.(.*?)\.\.\.\n", re.M | re.S)

    def __init__(self, zone: Zone) -> None:
        super().__init__()
        self.zone = zone
        self.doc = ""

    async def run(self) -> None:
        async with httpx.AsyncClient() as client:
            response = await client.get(self.zone.forecast_url)
            self.doc = response.text

    @property
    def advisory(self) -> str:
        if match := self.advisory_pat.search(self.doc):
            return match.group(1).replace("\n", " ")
        return ""

    def __repr__(self) -> str:
        return f"{self.zone.zone_name} {self.advisory}"

async def task_main() -> None:
    start = time.perf_counter()
    forecasts = [MarineWX(z) for z in ZONES]
    await asyncio.gather(
        *(f.run() for f in forecasts)
    )
    for f in forecasts:
        print(f)
    print(
        f"Got {len(forecasts)} forecasts "
        f"in {time.perf_counter() - start:.3f} seconds"
    )

if __name__ == "__main__":
    asyncio.run(task_main())
```

---

### Sección 14.6: El benchmark de los filósofos comensales

El problema clásico de los **filósofos comensales** ilustra cómo evitar situaciones de interbloqueo (*deadlock*) y contención de recursos compartidos:

> **Figura 14.2: Los filósofos comensales**  
> 5 filósofos sentados en mesa redonda, 5 tenedores. Cada comensal necesita dos tenedores adyacentes para comer.  
> *Solución:* Un mayordomo (`asyncio.BoundedSemaphore`) que limita a 4 el número máximo de comensales sentados a la vez.

```python
import asyncio
import random

FORKS: list[asyncio.Lock]

async def philosopher(id: int, footman: asyncio.Semaphore) -> tuple[int, float, float]:
    async with footman:
        async with FORKS[id], FORKS[(id + 1) % len(FORKS)]:
            eat_time = 1 + random.random()
            print(f"{id} eating")
            await asyncio.sleep(eat_time)
            think_time = 1 + random.random()
            print(f"{id} philosophizing")
            await asyncio.sleep(think_time)
            return id, eat_time, think_time

async def main(faculty: int = 5, servings: int = 5) -> None:
    global FORKS
    FORKS = [asyncio.Lock() for i in range(faculty)]
    footman = asyncio.BoundedSemaphore(faculty - 1)
    for serving in range(servings):
        department = (philosopher(p, footman) for p in range(faculty))
        results = await asyncio.gather(*department)
        print(results)

if __name__ == "__main__":
    asyncio.run(main())
```

---

### Sección 14.7: Repaso

Puntos clave tratados en este capítulo:

- **`threading`:** Rápido y sencillo para I/O dentro del mismo proceso; limitado por el GIL para cómputo intensivo.
- **`multiprocessing`:** Ejecución verdaderamente paralela en múltiples núcleos; requiere serialización de datos (*pickling*) entre procesos.
- **`concurrent.futures`:** Interfaz de alto nivel unificada (`Executor`, `Future`) para alternar fácilmente entre hilos y procesos.
- **`asyncio`:** Excelente escalabilidad para aplicaciones de red y servicios I/O no bloqueantes en un solo hilo mediante corrutinas cooperativas.
- **Sincronización:** Uso de `Lock`, `Semaphore` y `Queue` para prevenir condiciones de carrera e interbloqueos (*deadlocks*).

---

### Sección 14.8: Ejercicios

1. **Refactorización de filósofos sin variables globales:** Encapsula la gestión de tenedores y filósofos en una clase `DiningRoom` eliminando el uso de `global FORKS`.
2. **Paralelización de doctest:** Reutiliza el ejecutor de subprocesos del Capítulo 9 para ejecutar validaciones de `doctest` en paralelo usando `futures.ProcessPoolExecutor`.
3. **Servidor HTTP con `asyncio`:** Construye un servidor web elemental que responda a peticiones HTTP GET utilizando `asyncio.start_server`.

---

### Sección 14.9: Resumen

Con este capítulo concluimos nuestro recorrido por la **Programación Orientada a Objetos en Python (5ª Edición)**. 

A lo largo de este libro hemos transitado desde los fundamentos del diseño orientado a objetos, el sistema de clases, herencia y excepciones, pasando por la verificación formal de tipos con type hints, las estructuras de datos nativas y funcionales, el catálogo completo de patrones de diseño, las suites de pruebas automatizadas con TDD y las arquitecturas de procesamiento concurrente.

¡Feliz programación orientada a objetos en Python!
