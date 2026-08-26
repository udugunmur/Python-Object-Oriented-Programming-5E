# Parte 4: Calidad de Software, Pruebas y Concurrencia
## Capítulo 13: Pruebas de Programas Orientados a Objetos

Los programadores experimentados coinciden en que las pruebas son uno de los aspectos más cruciales del desarrollo de software. Todo lo aprendido hasta ahora sobre orientación a objetos y diseño nos servirá de guía fundamental para escribir pruebas automatizadas robustas.

En este capítulo exploraremos:

- La importancia de las **pruebas unitarias** y el **desarrollo guiado por pruebas** (*Test-Driven Development*, TDD).
- El módulo estándar **`unittest`**.
- El framework y ejecutor de pruebas **`pytest`**.
- El uso de **`unittest.mock`** para aislar componentes mediante dobles de prueba y objetos simulados.
- Medición y análisis de la **cobertura de código** (*code coverage*).

---

### Sección 13.1: ¿Por qué hacer pruebas?

Sin pruebas automatizadas, el código está roto por definición y nadie tiene forma de saberlo con certeza. En Python, las pruebas rara vez comprueban únicamente tipos; comprueban **valores**, estados internos, secuencias de operaciones y efectos colaterales.

> *"Las características del software que no se pueden demostrar mediante pruebas automatizadas simplemente no existen."*  
> — **Kent Beck**, *Extreme Programming Explained*

Razones principales para escribir pruebas automatizadas:
1. Garantizar que el código funciona como el desarrollador espera.
2. Asegurar que el código continúa funcionando tras refactorizaciones o adición de funciones (*prevención de regresiones*).
3. Confirmar que el desarrollador comprendió correctamente los requisitos del dominio.
4. Diseñar interfaces y APIs limpias, desacopladas y mantenibles.

#### 13.1.1 Desarrollo guiado por pruebas (TDD)

El ciclo esencial de TDD sigue la regla:
1. **Rojo (*Red*):** Escribir una prueba unitaria que falle para la funcionalidad aún no implementada.
2. **Verde (*Green*):** Escribir la cantidad mínima de código necesaria para que la prueba pase.
3. **Refactorización (*Refactor*):** Limpiar el diseño y eliminar duplicaciones manteniendo las pruebas en verde.

#### 13.1.2 Objetivos de las pruebas

- **Pruebas unitarias (*Unit Tests*):** Validan componentes aislados (clases o funciones individuales) en un entorno controlado. Ocupan la base de la Pirámide de Pruebas de Fowler.
- **Pruebas de integración (*Integration Tests*):** Validan la interacción y el flujo de comunicación entre múltiples componentes integrados (módulos, bases de datos, redes).

#### 13.1.3 Estructura de escenarios: GIVEN-WHEN-THEN (Arrange-Act-Assert)

Todo escenario de prueba sigue un patrón estándar de tres fases:

```text
SCENARIO: Caso de uso de una característica
GIVEN (Dado): Una precondición o estado inicial
WHEN (Cuando): Se ejecuta una acción o método sobre el objeto
THEN (Entonces): Ocurre un cambio de estado observable o resultado esperado
```

Ejemplo conceptual en el docstring de una función:

```python
def average(data: list[int | None]) -> float:
    """
    GIVEN a list, data = [1, 2, None, 3, 4]
    WHEN we compute m = average(data)
    THEN the result, m, is 2.5
    """
    pass
```

---

### Sección 13.2: Pruebas unitarias con `unittest`

El módulo integrado `unittest` proporciona una infraestructura orientada a objetos basada en clases derivada del patrón xUnit:

```python
import unittest

class CheckNumbers(unittest.TestCase):
    def test_int_float(self) -> None:
        self.assertEqual(1, 1.0)
```

Salida de ejecución exitosa:

```text
.
--------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

Adición de una aserción fallida:

```python
def test_str_float(self) -> None:
    self.assertEqual(1, "1")
```

Salida de error detallada:

```text
.F
============================================================
FAIL: test_str_float (__main__.CheckNumbers)
--------------------------------------------------------------
Traceback (most recent call last):
  File "first_unittest.py", line 9, in test_str_float
    self.assertEqual(1, "1")
AssertionError: 1 != '1'
--------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (failures=1)
```

> [!IMPORTANT]
> Cada método de prueba debe ser completamente **independiente y aislado**. El resultado o estado de una prueba nunca debe influir en las demás.

---

### Sección 13.3: Pruebas unitarias con `pytest`

`pytest` es el framework de pruebas más popular y potente del ecosistema Python. Permite escribir pruebas directamente como funciones simples aprovechando la instrucción nativa `assert`:

```python
def test_int_float() -> None:
    assert 1 == 1.0
```

También admite pruebas agrupadas en clases sin herencia obligatoria:

```python
class TestNumbers:
    def test_int_float(self) -> None:
        assert 1 == 1.0

    def test_int_str(self) -> None:
        assert 1 == "1"  # type: ignore [comparison-overlap]
```

Ejecución con `pytest`:

```bash
% python -m pytest tests/test_with_pytest.py
```

#### 13.3.1 Funciones de configuración y limpieza (*Setup* y *Teardown*)

```python
from collections.abc import Callable
from typing import Any

def setup_module(module: Any) -> None:
    print(f"setting up MODULE {module.__name__}")

def teardown_module(module: Any) -> None:
    print(f"tearing down MODULE {module.__name__}")

def test_a_function() -> None:
    print("RUNNING TEST FUNCTION")

class BaseTest:
    @classmethod
    def setup_class(cls: type["BaseTest"]) -> None:
        print(f"setting up CLASS {cls.__name__}")

    @classmethod
    def teardown_class(cls: type["BaseTest"]) -> None:
        print(f"tearing down CLASS {cls.__name__}\n")

    def setup_method(self, method: Callable[[], None]) -> None:
        print(f"setting up METHOD {method.__name__}")

    def teardown_method(self, method: Callable[[], None]) -> None:
        print(f"tearing down METHOD {method.__name__}")

class TestClass1(BaseTest):
    def test_method_1(self) -> None:
        print("RUNNING METHOD 1-1")

    def test_method_2(self) -> None:
        print("RUNNING METHOD 1-2")

class TestClass2(BaseTest):
    def test_method_1(self) -> None:
        print("RUNNING METHOD 2-1")

    def test_method_2(self) -> None:
        print("RUNNING METHOD 2-2")
```

#### 13.3.2 Accesorios (*Fixtures*) en `pytest`

Las **fixtures** son fábricas reutilizables decoradas con `@pytest.fixture` que establecen la fase GIVEN de las pruebas mediante inyección de dependencias:

Clase de dominio bajo prueba (`StatsList`):

```python
from collections import defaultdict

class StatsList(list[float | None]):
    """Stats with None objects rejected"""
    def mean(self) -> float:
        clean = list(filter(None, self))
        return sum(clean) / len(clean)

    def median(self) -> float:
        clean = list(filter(None, self))
        if len(clean) % 2:
            return clean[len(clean) // 2]
        else:
            idx = len(clean) // 2
            return (clean[idx] + clean[idx - 1]) / 2

    def mode(self) -> list[float]:
        freqs: defaultdict[float, int] = defaultdict(int)
        for item in filter(None, self):
            freqs[item] += 1
        mode_freq = max(freqs.values())
        modes = [item for item, value in freqs.items() if value == mode_freq]
        return modes
```

Pruebas unitarias consumiendo la fixture `valid_stats`:

```python
import pytest
from stats import StatsList

@pytest.fixture
def valid_stats() -> StatsList:
    return StatsList([1, 2, 2, 3, 3, 4])

def test_mean(valid_stats: StatsList) -> None:
    assert valid_stats.mean() == 2.5

def test_median(valid_stats: StatsList) -> None:
    assert valid_stats.median() == 2.5
    valid_stats.append(4)
    assert valid_stats.median() == 3

def test_mode(valid_stats: StatsList) -> None:
    assert valid_stats.mode() == [2, 3]
    valid_stats.remove(2)
    assert valid_stats.mode() == [3]
```

#### Fixtures generadoras con limpieza (`yield`)

Función a probar:

```python
from pathlib import Path
import hashlib

def checksum(source: Path, checksum_path: Path) -> None:
    if checksum_path.exists():
        backup = checksum_path.with_stem(f"(old) {checksum_path.stem}")
        backup.write_text(checksum_path.read_text())
    checksum = hashlib.sha256(source.read_bytes())
    checksum_path.write_text(f"{source.name} {checksum.hexdigest()}\n")
```

Fixture con preparación antes de `yield` y limpieza posterior:

```python
from collections.abc import Iterator
from pathlib import Path
import sys
import pytest
import checksum_writer

@pytest.fixture
def working_directory(tmp_path: Path) -> Iterator[tuple[Path, Path]]:
    working = tmp_path / "some_directory"
    working.mkdir()
    source = working / "data.txt"
    source.write_bytes(b"Hello, world!\n")
    checksum = working / "checksum.txt"
    checksum.write_text("data.txt Old_Checksum")
    yield source, checksum
    checksum.unlink()
    source.unlink()
```

Prueba con fixture y marcador condicional `@pytest.mark.skipif`:

```python
@pytest.mark.skipif(sys.version_info < (3, 9), reason="requires python3.9 feature")
def test_checksum(working_directory: tuple[Path, Path]) -> None:
    source_path, old_checksum_path = working_directory
    checksum_writer.checksum(source_path, old_checksum_path)
    backup = old_checksum_path.with_stem(f"(old) {old_checksum_path.stem}")
    assert backup.exists()
    assert old_checksum_path.exists()
    name, checksum = old_checksum_path.read_text().rstrip().split()
    assert name == source_path.name
    assert (
        checksum == "d9014c4624844aa5bac314773d6b689a"
        "d467fa4e1d1a50a1b8a99d5a95f72ff5"
    )
```

#### 13.3.3 Fixtures de integración con alcance de sesión (`scope="session"`)

Servidor de logs para pruebas de integración:

```python
import json
from pathlib import Path
import pickle
import socketserver
import struct
import sys
from typing import TextIO

class LogDataCatcher(socketserver.BaseRequestHandler):
    log_file: TextIO
    count: int = 0
    size_format = ">L"
    size_bytes = struct.calcsize(size_format)

    def handle(self) -> None:
        size_header_bytes = self.request.recv(LogDataCatcher.size_bytes)
        while size_header_bytes:
            payload_size = struct.unpack(LogDataCatcher.size_format, size_header_bytes)
            print(f"{size_header_bytes=} {payload_size=}", file=sys.stderr)
            payload_bytes = self.request.recv(payload_size[0])
            print(f"{len(payload_bytes)=}", file=sys.stderr)
            payload = pickle.loads(payload_bytes)
            LogDataCatcher.count += 1
            print(f"{self.client_address[0]} {LogDataCatcher.count} {payload!r}")
            self.log_file.write(json.dumps(payload) + "\n")
            try:
                size_header_bytes = self.request.recv(LogDataCatcher.size_bytes)
            except (ConnectionResetError, BrokenPipeError):
                break

def main(host: str, port: int, target: Path) -> None:
    with target.open("w") as unified_log:
        LogDataCatcher.log_file = unified_log
        with socketserver.TCPServer((host, port), LogDataCatcher) as server:
            server.serve_forever()

if __name__ == "__main__":
    HOST, PORT = "localhost", 18842
    main(HOST, PORT, Path("one.log"))
```

Aplicación cliente emisora de registros:

```python
import logging
import logging.handlers
import sys
from math import factorial

logger = logging.getLogger("app")

def work(i: int) -> int:
    logger.info("Factorial %d", i)
    f = factorial(i)
    logger.info("Factorial(%d) = %d", i, f)
    return f

if __name__ == "__main__":
    HOST, PORT = "localhost", 18842
    socket_handler = logging.handlers.SocketHandler(HOST, PORT)
    stream_handler = logging.StreamHandler(sys.stderr)
    logging.basicConfig(handlers=[socket_handler, stream_handler], level=logging.INFO)
    for i in range(10):
        work(i)
    logging.shutdown()
```

Prueba de integración con proceso en segundo plano:

```python
from collections.abc import Iterator
import logging
from pathlib import Path
import signal
import subprocess
import sys
import time
import pytest
import remote_logging_app

@pytest.fixture(scope="session")
def log_catcher() -> Iterator[None]:
    server_path = Path("src") / "log_catcher.py"
    print(f"Starting server {server_path}")
    p = subprocess.Popen(
        [sys.executable, str(server_path)],
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
    )
    time.sleep(0.25)
    yield
    p.terminate()
    p.wait()
    if p.stdout:
        print(p.stdout.read())
    assert (
        p.returncode == 1 if sys.platform == "win32" else -signal.SIGTERM.value
    ), f"Error in watcher, returncode={p.returncode}"

@pytest.fixture
def logging_config() -> Iterator[None]:
    HOST, PORT = "localhost", 18842
    socket_handler = logging.handlers.SocketHandler(HOST, PORT)
    remote_logging_app.logger.addHandler(socket_handler)
    yield
    socket_handler.close()
    remote_logging_app.logger.removeHandler(socket_handler)

def test_1(log_catcher: None, logging_config: None) -> None:
    for i in range(10):
        remote_logging_app.work(i)

def test_2(log_catcher: None, logging_config: None) -> None:
    for i in range(1, 10):
        remote_logging_app.work(52 * i)
```

#### 13.3.4 Omisión de pruebas (*Skipping*) y fallos esperados (*Xfail*)

```python
import sys
import pytest

def test_simple_skip() -> None:
    if sys.platform != "ios":
        pytest.skip("Test works only on Pythonista for ios")
    import location  # type: ignore [import]
    img = location.render_map_snapshot(36.8508, -76.2859)
    assert img is not None

@pytest.mark.skipif(
    sys.version_info < (3, 9),
    reason="requires 3.9, Path.removeprefix()"
)
def test_feature_python39() -> None:
    file_name = "(old) myfile.dat"
    assert file_name.removeprefix("(old) ") == "myfile.dat"
```

---

### Sección 13.4: Imitación de objetos mediante mocks

Los objetos simulados (**Mocks**) sustituyen dependencias complejas, externas o costosas (bases de datos, servicios de red, el reloj del sistema) para aislar completamente la unidad bajo prueba.

Clase cliente con dependencia en Redis:

```python
import datetime
from enum import Enum
from typing import cast
import redis

class Status(str, Enum):
    CANCELLED = "CANCELLED"
    DELAYED = "DELAYED"
    ON_TIME = "ON TIME"

class FlightStatusTracker:
    def __init__(self) -> None:
        self.redis = redis.Redis(host="127.0.0.1", port=6379, db=0)

    def change_status(self, flight: str, status: Status) -> None:
        if not isinstance(status, Status):
            raise ValueError(f"{status!r} is not a valid Status")
        key = f"flightno:{flight}"
        now = datetime.datetime.now(tz=datetime.timezone.utc)
        value = f"{now.isoformat()}|{status.value}"
        self.redis.set(key, value)

    def get_status(
        self, flight: str
    ) -> tuple[datetime.datetime | None, Status | None]:
        key = f"flightno:{flight}"
        value = cast(str, self.redis.get(key))
        if value:
            text_timestamp, text_status = value.split("|")
            timestamp = datetime.datetime.fromisoformat(text_timestamp)
            status = Status(text_status)
            return timestamp, status
        return None, None
```

Prueba unitaria aislando Redis con `Mock` y `monkeypatch`:

```python
import datetime
from unittest.mock import Mock, patch, call
import pytest
import flight_status_redis

@pytest.fixture
def mock_redis() -> Mock:
    mock_redis_instance = Mock(set=Mock(return_value=True))
    return mock_redis_instance

@pytest.fixture
def tracker(
    monkeypatch: pytest.MonkeyPatch,
    mock_redis: Mock
) -> flight_status_redis.FlightStatusTracker:
    fst = flight_status_redis.FlightStatusTracker()
    monkeypatch.setattr(fst, "redis", mock_redis)
    return fst

def test_monkeypatch_class(
    tracker: flight_status_redis.FlightStatusTracker,
    mock_redis: Mock
) -> None:
    with pytest.raises(ValueError) as ex:
        tracker.change_status("AC101", "lost")  # type: ignore [arg-type]
    assert ex.value.args[0] == "'lost' is not a valid Status"
    assert mock_redis.set.call_count == 0
```

#### 13.4.1 Técnicas adicionales de parcheo con `unittest.mock.patch`

Parcheo determinista del reloj del sistema (`datetime.datetime.now`):

```python
def test_patch_class(
    tracker: flight_status_redis.FlightStatusTracker,
    mock_redis: Mock
) -> None:
    fake_now = datetime.datetime(2020, 10, 26, 23, 24, 25)
    utc = datetime.timezone.utc
    with patch("flight_status_redis.datetime") as mock_datetime:
        mock_datetime.datetime = Mock(now=Mock(return_value=fake_now))
        mock_datetime.timezone = Mock(utc=utc)
        tracker.change_status("AC101", flight_status_redis.Status.ON_TIME)
        mock_datetime.datetime.now.assert_called_once_with(tz=utc)
        expected = "2020-10-26T23:24:25|ON TIME"
        mock_redis.set.assert_called_once_with("flightno:AC101", expected)
```

Diseño desacoplado mediante **inyección de dependencias**:

```python
class FlightStatusTracker_Alt:
    def __init__(self, redis_instance: redis.Connection | None = None) -> None:
        self.redis = (
            redis_instance if redis_instance else redis.Redis(host="127.0.0.1", port=6379, db=0)
        )
```

#### 13.4.2 El objeto centinela (`sentinel`)

`unittest.mock.sentinel` genera objetos opacos únicos ideales para verificar que los datos se transmiten o almacenan sin alteraciones:

```python
class FileChecksum:
    def __init__(self, source: Path) -> None:
        self.source = source
        self.checksum = hashlib.sha256(source.read_bytes())
```

```python
from unittest.mock import Mock, sentinel
from typing import Any

@pytest.fixture
def mock_hashlib(monkeypatch: Any) -> Mock:
    mocked_hashlib = Mock(sha256=Mock(return_value=sentinel.checksum))
    monkeypatch.setattr(checksum_writer, "hashlib", mocked_hashlib)
    return mocked_hashlib

def test_file_checksum(mock_hashlib: Mock, tmp_path: Any) -> None:
    source_file = tmp_path / "some_file"
    source_file.write_text("")
    cw = checksum_writer.FileChecksum(source_file)
    assert cw.source == source_file
    assert cw.checksum == sentinel.checksum
```

---

### Sección 13.5: ¿Cuántas pruebas son suficientes?

La **cobertura de código** (*code coverage*) mide el porcentaje de líneas y ramas de decisión ejecutadas durante las pruebas:

Instalación y ejecución con `coverage.py`:

```bash
% export PYTHONPATH="$(pwd)/src:$PYTHONPATH"
% coverage run -m pytest tests/test_coverage.py
% coverage report -m
```

Salida del informe de cobertura:

```text
Name                    Stmts   Miss  Cover   Missing
------------------------------------------------------
src/stats.py               19     11    42%   18-23, 26-31
tests/test_coverage.py      7      0   100%
------------------------------------------------------
TOTAL                      26     11    58%
```

> [!TIP]
> Una cobertura del 100% de líneas no garantiza la ausencia de fallos. Siempre se debe aplicar **análisis de valores límite** (*Boundary Value Analysis*) y partición de equivalencia sobre datos numéricos, colecciones vacías y valores nulos (`None`).

---

### Sección 13.6: Pruebas y desarrollo

Flujo de trabajo para corrección de anomalías guiado por pruebas:

1. Reproducir el error escribiendo una prueba unitaria específica que **falle**.
2. Modificar el código de la aplicación hasta que todas las pruebas pasen.
3. Refactorizar con total seguridad de no introducir nuevas regresiones.

---

### Sección 13.7: Repaso

Puntos clave tratados en este capítulo:

- **TDD:** Escribir primero la prueba que falla garantiza un diseño modular y enfocado a interfaces limpias.
- **`unittest`:** Módulo estándar de pruebas basado en clases y aserciones explícitas (`assertEqual`, `assertRaises`).
- **`pytest`:** Framework moderno basado en funciones, aserciones nativas `assert` y un potente sistema de fixtures con inyección automática.
- **Mocks y Parches:** Aislamiento de unidades mediante `unittest.mock.Mock`, `patch` y `sentinel`.
- **Cobertura de código:** Métrica objetiva con `coverage.py` para detectar código no ejercitado en la suite de pruebas.

---

### Sección 13.8: Ejercicios

1. **Suite completa de StatsList:** Escribe pruebas unitarias con `pytest` para cubrir al 100% las ramas de `mean()`, `median()` y `mode()` en `StatsList`, incluyendo listas vacías y elementos `None`.
2. **Mocking de APIs HTTP:** Implementa una clase que consulte una API REST externa y escribe sus pruebas unitarias simulando las respuestas con `unittest.mock.patch`.
3. **Reporte de cobertura en HTML:** Genera un informe visual interactivo ejecutando `coverage html` e inspecciona el reporte en el navegador.

---

### Sección 13.9: Resumen

En este capítulo analizamos cómo estructurar pruebas unitarias e integradas en Python utilizando `unittest`, `pytest`, `mock` y herramientas de cobertura.

En el próximo y último capítulo, abordaremos la **concurrencia en Python**: hilos (*threads*), multiprocesamiento (*multiprocessing*), futuros y programación asíncrona con **`asyncio`**.
