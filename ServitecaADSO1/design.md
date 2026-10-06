# Documento de DiseÃ±o TÃ©cnico â€” Serviteca ADSO

## Resumen ejecutivo

Este documento describe la arquitectura tÃ©cnica y el diseÃ±o detallado de la aplicaciÃ³n de escritorio **Serviteca ADSO**: un sistema de gestiÃ³n para talleres de vehÃ­culos construido en Python con interfaz grÃ¡fica tkinter y persistencia en archivos JSON locales. El documento cubre la arquitectura MVC, los componentes principales, los modelos de datos, las propiedades de correcciÃ³n verificables, el manejo de errores y la estrategia de pruebas.

---

## Overview

### Contexto del sistema

La aplicaciÃ³n Serviteca ADSO es un sistema monousuario de escritorio diseÃ±ado para el personal de talleres automotrices. Permite gestionar el ciclo completo de atenciÃ³n al cliente: desde el registro de clientes y vehÃ­culos hasta el seguimiento del historial de servicios prestados.

### Objetivos tÃ©cnicos principales

- **Sin dependencias externas**: toda la interfaz usa `tkinter` (estÃ¡ndar de Python) y la persistencia usa el mÃ³dulo `json` de la librerÃ­a estÃ¡ndar.
- **Arquitectura limpia**: separaciÃ³n estricta de responsabilidades mediante el patrÃ³n MVC.
- **Robustez ante fallos**: manejo explÃ­cito de errores de I/O, validaciÃ³n defensiva de entradas y recuperaciÃ³n graceful.
- **Portabilidad**: compatible con Windows, macOS y Linux con Python 3.10+.

### Alcance

| MÃ³dulo | Funcionalidad |
|---|---|
| AutenticaciÃ³n | Login / logout con bloqueo por intentos fallidos |
| Clientes | CRUD completo, bÃºsqueda, suspensiÃ³n de cuenta |
| Carros | CRUD completo, bÃºsqueda, suspensiÃ³n de vehÃ­culo |
| Servicios | Registro de servicios, historial por placa |
| Ayuda | Pantalla de instrucciones offline |
| Persistencia | Lectura/escritura de archivos JSON locales |

### Decisiones tecnolÃ³gicas clave

| DecisiÃ³n | Alternativas consideradas | JustificaciÃ³n |
|---|---|---|
| `tkinter` para GUI | PyQt5, wxPython, customtkinter | LibrerÃ­a estÃ¡ndar, sin instalaciÃ³n, soportada en todas las plataformas Python |
| JSON para persistencia | SQLite, CSV, pickle | Legible por humanos, sin dependencias, fÃ¡cil de depurar y respaldar |
| PatrÃ³n MVC | MVP, MVVM | Claridad conceptual para el nivel ADSO, separaciÃ³n explÃ­cita de capas |
| `dataclasses` para modelos | dict puro, Pydantic | Tipado fuerte sin dependencias externas; compatible con Python 3.7+ |
| `hypothesis` para PBT | QuickCheck (Haskell), fast-check (JS) | LibrerÃ­a de property-based testing mÃ¡s madura para Python |

---

## Architecture

### PatrÃ³n MVC aplicado a tkinter

```mermaid
graph TB
    subgraph View["Vista (tkinter)"]
        LV[LoginView]
        MV[MainView]
        CLV[ClientesListView]
        CEV[ClientesFormView]
        CARV[CarrosListView]
        CAREF[CarrosFormView]
        SV[ServiciosView]
        HV[AyudaView]
    end

    subgraph Controller["Controlador"]
        AC[AppController]
        AuthC[AuthController]
        CC[ClientesController]
        CarC[CarrosController]
        SC[ServiciosController]
    end

    subgraph Model["Modelo"]
        UR[UsuarioRepository]
        CR[ClienteRepository]
        CarR[CarroRepository]
        SR[ServicioRepository]
        JS[JSONStorage]
    end

    View -->|eventos de usuario| Controller
    Controller -->|actualiza estado| Model
    Controller -->|refresca vista| View
    Model -->|lee/escribe| JS
    JS -->|archivos JSON| FS[(Sistema de archivos)]
```

### DescripciÃ³n de capas

**Capa Vista (`views/`)**
- Solo contiene cÃ³digo de presentaciÃ³n tkinter (widgets, layouts, binding de eventos).
- No tiene lÃ³gica de negocio ni acceso directo a datos.
- Expone mÃ©todos de actualizaciÃ³n que el controlador llama: `update_table()`, `show_error()`, `clear_form()`, etc.

**Capa Controlador (`controllers/`)**
- Orquesta las interacciones entre vistas y modelos.
- Contiene toda la lÃ³gica de validaciÃ³n de entradas y reglas de negocio.
- `AppController` es el punto de entrada principal: gestiona la ventana raÃ­z y el ciclo de vida de la sesiÃ³n.

**Capa Modelo (`models/`, `repositories/`)**
- `models/` define las estructuras de datos (`dataclasses`).
- `repositories/` encapsula las operaciones CRUD sobre los archivos JSON.
- `JSONStorage` es el adaptador de bajo nivel para lectura/escritura de archivos.

### Flujo de navegaciÃ³n

```mermaid
stateDiagram-v2
    [*] --> Login: inicio aplicaciÃ³n
    Login --> PantallaP: credenciales vÃ¡lidas
    Login --> Login: credenciales invÃ¡lidas (< 5 intentos)
    Login --> Bloqueado: 5 intentos fallidos
    Bloqueado --> Login: transcurridos 15 minutos
    PantallaP --> Clientes: menÃº â†’ Clientes
    PantallaP --> Carros: menÃº â†’ Carros
    PantallaP --> Servicios: menÃº â†’ Servicios
    PantallaP --> Ayuda: menÃº â†’ Ayuda
    PantallaP --> ConfirmLogout: menÃº â†’ Cerrar SesiÃ³n
    ConfirmLogout --> PantallaP: cancelar
    ConfirmLogout --> Login: confirmar
    Clientes --> PantallaP: navegar
    Carros --> PantallaP: navegar
    Servicios --> PantallaP: navegar
    Ayuda --> PantallaP: navegar
```

### Estructura de directorios del proyecto

```
serviteca_adso/
â”œâ”€â”€ main.py                    # Punto de entrada
â”œâ”€â”€ config.py                  # Constantes globales (rutas JSON, timeouts)
â”œâ”€â”€ models/
â”‚   â”œâ”€â”€ __init__.py
â”‚   â”œâ”€â”€ usuario.py             # dataclass Usuario
â”‚   â”œâ”€â”€ cliente.py             # dataclass Cliente + EstadoCliente enum
â”‚   â”œâ”€â”€ carro.py               # dataclass Carro + EstadoCarro enum
â”‚   â””â”€â”€ servicio.py            # dataclass Servicio + TipoServicio enum
â”œâ”€â”€ repositories/
â”‚   â”œâ”€â”€ __init__.py
â”‚   â”œâ”€â”€ json_storage.py        # Adaptador JSON (load/save atÃ³mico)
â”‚   â”œâ”€â”€ usuario_repository.py
â”‚   â”œâ”€â”€ cliente_repository.py
â”‚   â”œâ”€â”€ carro_repository.py
â”‚   â””â”€â”€ servicio_repository.py
â”œâ”€â”€ controllers/
â”‚   â”œâ”€â”€ __init__.py
â”‚   â”œâ”€â”€ app_controller.py      # Orquestador principal
â”‚   â”œâ”€â”€ auth_controller.py     # LÃ³gica de autenticaciÃ³n y bloqueo
â”‚   â”œâ”€â”€ clientes_controller.py
â”‚   â”œâ”€â”€ carros_controller.py
â”‚   â””â”€â”€ servicios_controller.py
â”œâ”€â”€ views/
â”‚   â”œâ”€â”€ __init__.py
â”‚   â”œâ”€â”€ base_view.py           # Frame base con helpers comunes
â”‚   â”œâ”€â”€ login_view.py
â”‚   â”œâ”€â”€ main_view.py           # Frame principal con menÃº lateral
â”‚   â”œâ”€â”€ clientes_view.py
â”‚   â”œâ”€â”€ carros_view.py
â”‚   â”œâ”€â”€ servicios_view.py
â”‚   â””â”€â”€ ayuda_view.py
â”œâ”€â”€ data/                      # Archivos JSON de persistencia (generados en runtime)
â”‚   â”œâ”€â”€ usuarios.json
â”‚   â”œâ”€â”€ clientes.json
â”‚   â”œâ”€â”€ carros.json
â”‚   â””â”€â”€ servicios.json
â””â”€â”€ tests/
    â”œâ”€â”€ unit/
    â”‚   â”œâ”€â”€ test_auth_controller.py
    â”‚   â”œâ”€â”€ test_clientes_controller.py
    â”‚   â”œâ”€â”€ test_carros_controller.py
    â”‚   â””â”€â”€ test_servicios_controller.py
    â””â”€â”€ property/
        â”œâ”€â”€ test_auth_properties.py
        â”œâ”€â”€ test_clientes_properties.py
        â”œâ”€â”€ test_carros_properties.py
        â”œâ”€â”€ test_servicios_properties.py
        â””â”€â”€ test_persistencia_properties.py
```

---

## Components and Interfaces

### AppController

Responsabilidad: gestionar el ciclo de vida de la ventana raÃ­z de tkinter, el estado de sesiÃ³n global y la navegaciÃ³n entre vistas.

```python
class AppController:
    def __init__(self, root: tk.Tk) -> None: ...
    def start(self) -> None:
        """Inicializa repositorios, lanza LoginView y entra al mainloop."""
    def on_login_success(self, usuario: Usuario) -> None:
        """Recibe el usuario autenticado, guarda la sesiÃ³n y muestra MainView."""
    def on_logout(self) -> None:
        """Invalida la sesiÃ³n activa y regresa a LoginView."""
    def navigate_to(self, modulo: str) -> None:
        """Carga el frame del mÃ³dulo indicado en el Ã¡rea de contenido."""
    def require_auth(self) -> bool:
        """Devuelve True si hay sesiÃ³n activa; redirige a login si no."""
```

### AuthController

Responsabilidad: validar credenciales, gestionar el contador de intentos fallidos y el bloqueo temporal.

```python
class AuthController:
    MAX_INTENTOS: int = 5
    TIEMPO_BLOQUEO_SEGUNDOS: int = 900  # 15 minutos

    def autenticar(self, usuario: str, contraseÃ±a: str) -> ResultadoAuth:
        """
        Valida las credenciales. Retorna ResultadoAuth con:
        - exito: bool
        - usuario: Optional[Usuario]
        - mensaje_error: Optional[str]
        """
    def registrar_intento_fallido(self) -> None: ...
    def esta_bloqueado(self) -> bool: ...
    def segundos_restantes_bloqueo(self) -> int: ...
    def reiniciar_intentos(self) -> None: ...
```

### ClientesController

Responsabilidad: operaciones CRUD de clientes, validaciones de negocio y sincronizaciÃ³n con la vista.

```python
class ClientesController:
    def crear_cliente(self, datos: dict) -> ResultadoOperacion: ...
    def editar_cliente(self, cedula_original: str, datos: dict) -> ResultadoOperacion: ...
    def eliminar_cliente(self, cedula: str) -> ResultadoOperacion: ...
    def suspender_cliente(self, cedula: str) -> ResultadoOperacion: ...
    def buscar_clientes(self, texto: str) -> list[Cliente]: ...
    def obtener_todos(self) -> list[Cliente]: ...
    def _validar_datos_cliente(self, datos: dict, cedula_excluir: str = "") -> list[str]:
        """Retorna lista de mensajes de error; vacÃ­a si es vÃ¡lido."""
```

### CarrosController

Responsabilidad: operaciones CRUD de carros, validaciones de negocio y sincronizaciÃ³n con la vista.

```python
class CarrosController:
    def crear_carro(self, datos: dict) -> ResultadoOperacion: ...
    def editar_carro(self, placa_original: str, datos: dict) -> ResultadoOperacion: ...
    def eliminar_carro(self, placa: str) -> ResultadoOperacion: ...
    def suspender_carro(self, placa: str) -> ResultadoOperacion: ...
    def buscar_carros(self, texto: str) -> list[Carro]: ...
    def obtener_todos(self) -> list[Carro]: ...
    def obtener_carros_por_cliente(self, cedula: str) -> list[Carro]:
        """Retorna todos los Carros cuyo cedula_propietario coincide con la cedula dada."""
    def _validar_datos_carro(self, datos: dict, placa_excluir: str = "") -> list[str]:
        """
        Valida: placa alfanumerica 5-7 chars, marca/modelo no vacios <= 50,
        cedula_propietario existe en ClienteRepository.
        Retorna lista de mensajes de error; vacia si es valido.
        """
```

### ServiciosController

Responsabilidad: registro de servicios y consulta del historial por placa.

```python
class ServiciosController:
    def registrar_servicio(self, datos: dict) -> ResultadoOperacion: ...
    def obtener_historial_por_placa(self, placa: str) -> list[Servicio]:
        """Retorna servicios ordenados por fecha descendente."""
    def _validar_datos_servicio(self, datos: dict) -> list[str]: ...
```

### JSONStorage

Responsabilidad: lectura y escritura atÃ³mica de archivos JSON.

```python
class JSONStorage:
    def __init__(self, ruta: str) -> None: ...
    def cargar(self) -> list[dict]:
        """Lee el archivo JSON. Retorna lista vacÃ­a si no existe."""
    def guardar(self, datos: list[dict]) -> None:
        """Escribe de forma atÃ³mica usando archivo temporal + rename."""
    def existe(self) -> bool: ...
```

> **DecisiÃ³n de diseÃ±o â€” escritura atÃ³mica**: para evitar corrupciÃ³n de datos ante cierres inesperados, `JSONStorage.guardar()` escribe primero a un archivo temporal (`.tmp`) y luego hace `os.replace()` al archivo definitivo. Esta operaciÃ³n es atÃ³mica en sistemas de archivos POSIX y en Windows NTFS (Python 3.3+).

### Interfaces de vista (contratos)

Cada vista implementa los siguientes mÃ©todos mÃ­nimos para ser controlada desde el controlador:

```python
class BaseView(tk.Frame):
    def mostrar(self) -> None: ...
    def ocultar(self) -> None: ...
    def show_error(self, mensaje: str) -> None: ...
    def show_info(self, mensaje: str) -> None: ...
```

---

## Data Models

### Usuario

```python
from dataclasses import dataclass

@dataclass
class Usuario:
    username: str          # identificador Ãºnico, case-insensitive en validaciÃ³n
    password_hash: str     # SHA-256 del password; nunca se almacena el plaintext
```

**Archivo JSON**: `data/usuarios.json`
```json
[
  {
    "username": "admin",
    "password_hash": "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
  }
]
```

> **DecisiÃ³n de seguridad**: las contraseÃ±as se almacenan como hash SHA-256 (mÃ³dulo `hashlib` estÃ¡ndar de Python). En una aplicaciÃ³n de producciÃ³n se recomendarÃ­a bcrypt o PBKDF2, pero dado el contexto de escritorio sin servidor, SHA-256 es suficiente y no requiere dependencias externas.

---

### Cliente

```python
from dataclasses import dataclass, field
from enum import Enum

class EstadoCliente(str, Enum):
    ACTIVO = "Activo"
    SUSPENDIDO = "Suspendido"

@dataclass
class Cliente:
    cedula: str              # 1â€“10 dÃ­gitos numÃ©ricos; clave primaria
    nombres: str             # mÃ¡x 100 caracteres
    apellidos: str           # mÃ¡x 100 caracteres
    celular: str             # mÃ¡x 15 dÃ­gitos numÃ©ricos
    correo: str = ""         # formato email vÃ¡lido o cadena vacÃ­a; opcional
    estado: EstadoCliente = EstadoCliente.ACTIVO
```

**Archivo JSON**: `data/clientes.json`
```json
[
  {
    "cedula": "1234567890",
    "nombres": "Juan",
    "apellidos": "PÃ©rez",
    "celular": "3001234567",
    "correo": "juan@ejemplo.com",
    "estado": "Activo"
  }
]
```

**Reglas de validaciÃ³n**:
- `cedula`: solo dÃ­gitos, longitud 1â€“10. Ãšnico en el repositorio.
- `nombres`, `apellidos`: no vacÃ­os, longitud â‰¤ 100.
- `celular`: solo dÃ­gitos, longitud 1â€“15.
- `correo`: si no estÃ¡ vacÃ­o, debe coincidir con el patrÃ³n `^[^@\s]+@[^@\s]+\.[^@\s]+$`.

---

### Carro

```python
class EstadoCarro(str, Enum):
    ACTIVO = "Activo"
    INACTIVO = "Inactivo"

@dataclass
class Carro:
    placa: str              # 5â€“7 caracteres alfanumÃ©ricos; clave primaria
    marca: str              # mÃ¡x 50 caracteres
    modelo: str             # mÃ¡x 50 caracteres
    estado: EstadoCarro = EstadoCarro.ACTIVO
```

**Archivo JSON**: `data/carros.json`
```json
[
  {
    "placa": "ABC123",
    "marca": "Toyota",
    "modelo": "Corolla",
    "cedula_propietario": "1234567890",
    "estado": "Activo"
  }
]
```

**Reglas de validaciÃ³n**:
- `placa`: solo caracteres alfanumÃ©ricos (`[A-Za-z0-9]`), longitud 5â€“7. Ãšnico en el repositorio.
- `marca`, `modelo`: no vacíos, longitud <= 50.
- `cedula_propietario`: obligatorio; debe existir en el repositorio de Clientes (integridad referencial). Un Cliente puede tener múltiples Carros; cada Carro tiene exactamente un propietario (relación **uno a muchos** `Cliente -> Carro`).

---

### Servicio

```python
from datetime import date

class TipoServicio(str, Enum):
    MANTENIMIENTO = "Mantenimiento"
    CAMBIO_ACEITE = "Cambio de Aceite"
    FRENOS = "Frenos"
    REVISION_GENERAL = "RevisiÃ³n General"
    OTRO = "Otro"

@dataclass
class Servicio:
    id_servicio: str        # UUID generado en creaciÃ³n
    placa_carro: str        # FK a Carro.placa
    cedula_cliente: str     # FK a Cliente.cedula
    fecha: date             # â‰¤ fecha actual; almacenada como "YYYY-MM-DD"
    tipo: TipoServicio
    notas: str = ""         # mÃ¡x 500 caracteres; opcional
```

**Archivo JSON**: `data/servicios.json`
```json
[
  {
    "id_servicio": "a1b2c3d4-...",
    "placa_carro": "ABC123",
    "cedula_cliente": "1234567890",
    "fecha": "2024-11-15",
    "tipo": "Mantenimiento",
    "notas": "Cambio filtros y revisiÃ³n completa"
  }
]
```

**Reglas de validaciÃ³n**:
- `placa_carro`: debe existir en el repositorio de Carros.
- `cedula_cliente`: debe existir en el repositorio de Clientes.
- `fecha`: formato ISO 8601, no puede ser posterior a `date.today()`.
- `tipo`: debe ser uno de los valores definidos en `TipoServicio`.
- `notas`: longitud â‰¤ 500 si se provee.

---

### ResultadoOperacion (DTO compartido)

```python
@dataclass
class ResultadoOperacion:
    exito: bool
    mensaje: str = ""
    errores: list[str] = field(default_factory=list)
    dato: object = None  # entidad resultante cuando exito=True
```

---

### Diagrama de relaciones entre entidades

```mermaid
erDiagram
    USUARIO {
        string username PK
        string password_hash
    }
    CLIENTE {
        string cedula PK
        string nombres
        string apellidos
        string celular
        string correo
        string estado
    }
    CARRO {
        string placa PK
        string marca
        string modelo
        string cedula_propietario FK
        string estado
    }
    SERVICIO {
        string id_servicio PK
        string placa_carro FK
        string cedula_cliente FK
        string fecha
        string tipo
        string notas
    }

    CLIENTE ||--o{ CARRO : "posee (1:N)"
    CARRO ||--o{ SERVICIO : "tiene"
    CLIENTE ||--o{ SERVICIO : "genera"
```

> **Decisión de diseño — relación uno a muchos Cliente → Carro**: cada `Carro` almacena la `cedula_propietario` como clave foránea hacia `Cliente.cedula`. Esta FK se valida en `CarrosController._validar_datos_carro()` antes de persistir. No se implementa eliminación en cascada: si se elimina un Cliente, sus Carros quedan huérfanos y deben eliminarse o reasignarse manualmente. Esta decisión es deliberada para el contexto de escritorio monousuario: conservar el historial de servicios asociado a la placa tiene más valor operativo que forzar la cascada. En una versión futura se puede añadir una validación de integridad en `ClienteRepository.eliminar()`.

---

## Correctness Properties

*Una propiedad es una caracterÃ­stica o comportamiento que debe ser verdadero en todas las ejecuciones vÃ¡lidas de un sistema â€” esencialmente, una afirmaciÃ³n formal sobre lo que el sistema debe hacer. Las propiedades sirven como puente entre especificaciones legibles por humanos y garantÃ­as de correcciÃ³n verificables por mÃ¡quinas.*

---

### Property 1: AutenticaciÃ³n correcta para cualquier credencial

*Para cualquier* par (usuario, contraseÃ±a), la funciÃ³n de autenticaciÃ³n debe retornar Ã©xito si y solo si el par coincide con un registro vÃ¡lido del repositorio. Para pares invÃ¡lidos, debe retornar un error genÃ©rico que no revele cuÃ¡l campo es incorrecto.

**Validates: Requirements 1.4, 1.5**

---

### Property 2: Rechazo de credenciales con campos vacÃ­os

*Para cualquier* combinaciÃ³n donde el campo usuario o el campo contraseÃ±a (o ambos) sean cadenas vacÃ­as, la validaciÃ³n previa a la autenticaciÃ³n debe rechazar el intento sin consultar el repositorio.

**Validates: Requirements 1.6**

---

### Property 3: Bloqueo por intentos fallidos consecutivos

*Para cualquier* secuencia de N intentos de autenticaciÃ³n fallidos donde N â‰¥ 5, el sistema debe reportar estado bloqueado al finalizar la secuencia, independientemente del orden o contenido de las credenciales invÃ¡lidas.

**Validates: Requirements 1.7**

---

### Property 4: ValidaciÃ³n de campos obligatorios de Cliente

*Para cualquier* conjunto de datos de cliente donde uno o mÃ¡s campos obligatorios (cÃ©dula, nombres, apellidos, celular) estÃ©n vacÃ­os o tengan formato invÃ¡lido, la funciÃ³n de validaciÃ³n debe retornar al menos un mensaje de error y no persistir el registro.

**Validates: Requirements 3.2, 3.8, 3.13**

---

### Property 5: Unicidad de cÃ©dula en el sistema

*Para cualquier* cÃ©dula ya registrada en el repositorio de clientes, cualquier intento de crear o editar un cliente con esa misma cÃ©dula (perteneciente a un registro distinto) debe ser rechazado con un mensaje de conflicto.

**Validates: Requirements 3.3, 3.9**

---

### Property 6: Estado inicial "Activo" al crear cliente

*Para cualquier* cliente creado exitosamente a travÃ©s del formulario de registro, su Estado_Cliente inicial debe ser "Activo", independientemente de los datos ingresados.

**Validates: Requirements 3.4**

---

### Property 7: Filtrado de clientes es case-insensitive y completo

*Para cualquier* lista de clientes y cualquier texto de bÃºsqueda no vacÃ­o, todos los registros retornados por la funciÃ³n de filtrado deben contener el texto de bÃºsqueda (ignorando mayÃºsculas y minÃºsculas) en al menos uno de los campos: nombres, apellidos o cÃ©dula; y ningÃºn registro que contenga el texto debe ser excluido.

**Validates: Requirements 3.6**

---

### Property 8: Round-trip de carga de cliente para ediciÃ³n

*Para cualquier* cliente registrado en el repositorio, cargar sus datos en el formulario de ediciÃ³n debe producir exactamente el mismo conjunto de campos que el registro almacenado â€” sin pÃ©rdida, alteraciÃ³n ni adiciÃ³n de valores.

**Validates: Requirements 3.7**

---

### Property 9: SuspensiÃ³n de cliente activo

*Para cualquier* cliente con Estado_Cliente "Activo", aplicar la acciÃ³n "Suspender" debe producir Estado_Cliente "Suspendido" y persistir ese cambio inmediatamente en el repositorio.

**Validates: Requirements 3.11**

---

### Property 10: ValidaciÃ³n de formato de placa de Carro

*Para cualquier* cadena de texto que no cumpla exactamente con el patrÃ³n alfanumÃ©rico de 5 a 7 caracteres, la validaciÃ³n de placa debe rechazarla. Para cualquier cadena que sÃ­ cumpla el patrÃ³n, la validaciÃ³n debe aceptarla.

**Validates: Requirements 4.2, 4.3**

---

### Property 11: Unicidad de placa en el sistema

*Para cualquier* placa ya registrada en el repositorio de carros, cualquier intento de crear o editar un carro con esa misma placa (perteneciente a un registro distinto) debe ser rechazado con un mensaje de conflicto.

**Validates: Requirements 4.4, 4.9**

---

### Property 12: Filtrado de carros es case-insensitive y completo

*Para cualquier* lista de carros y cualquier texto de bÃºsqueda no vacÃ­o, todos los registros retornados deben contener el texto (ignorando mayÃºsculas y minÃºsculas) en al menos uno de los campos: placa, marca o modelo; y ningÃºn registro que contenga el texto debe ser excluido.

**Validates: Requirements 4.6**

---

### Property 13: SuspensiÃ³n de carro activo

*Para cualquier* carro con Estado_Carro "Activo", aplicar la acciÃ³n "Suspender" debe producir Estado_Carro "Inactivo" y persistir ese cambio en el repositorio.

**Validates: Requirements 4.12**

---

### Property 14: Rechazo de fechas futuras en Servicios

*Para cualquier* fecha posterior a la fecha actual del sistema, el selector de fecha y la validaciÃ³n del formulario de servicios deben rechazar esa fecha como invÃ¡lida.

**Validates: Requirements 5.2**

---

### Property 15: Integridad referencial al registrar un Servicio

*Para cualquier* intento de registrar un servicio, si la placa del carro o la cÃ©dula del cliente no existen en sus respectivos repositorios, el registro debe ser rechazado con el mensaje de error correspondiente; si ambas referencias existen y el resto de campos son vÃ¡lidos, el servicio debe guardarse exitosamente.

**Validates: Requirements 5.4, 5.5, 5.6, 5.7**

---

### Property 16: Completitud del historial de servicios por placa

*Para cualquier* placa registrada a la que se le hayan asociado N servicios, la consulta del historial debe retornar exactamente esos N servicios, sin omisiones ni duplicados.

**Validates: Requirements 6.2**

---

### Property 17: Orden descendente del historial de servicios

*Para cualquier* lista de servicios retornada por la consulta histÃ³rica de una placa, cada servicio en la posiciÃ³n i debe tener una fecha mayor o igual a la del servicio en la posiciÃ³n i+1 (orden descendente por fecha).

**Validates: Requirements 6.6**

---

### Property 18: SesiÃ³n cerrada impide acceso a mÃ³dulos protegidos

*Para cualquier* mÃ³dulo protegido del sistema (Clientes, Carros, Servicios, Ayuda), intentar acceder con la sesiÃ³n invalidada debe resultar en redirecciÃ³n a la pantalla de login, sin mostrar datos del mÃ³dulo solicitado.

**Validates: Requirements 8.4, 8.5**

---

### Property 19: Round-trip de serializaciÃ³n JSON

*Para cualquier* entidad del sistema (Cliente, Carro, Servicio), serializar la entidad a JSON y luego deserializar el resultado debe producir una entidad equivalente campo a campo, sin pÃ©rdida de datos ni alteraciÃ³n de valores.

**Validates: Requirements 9.1**

---

## Error Handling

### Principios generales

1. **Fallo observable**: todo error que afecte al usuario debe mostrarse mediante un diÃ¡logo o etiqueta de error visible, nunca silenciarse.
2. **Fallo aislado**: un error en un mÃ³dulo no debe propagar excepciones no manejadas que cierren la aplicaciÃ³n.
3. **PreservaciÃ³n de datos**: ante errores de escritura, los datos existentes no se modifican (gracias a la escritura atÃ³mica).
4. **Recuperabilidad**: ante errores de carga de datos al inicio, el sistema arranca con tablas vacÃ­as y permite operaciÃ³n normal.

### ClasificaciÃ³n de errores

| CategorÃ­a | Ejemplo | Tratamiento |
|---|---|---|
| ValidaciÃ³n de entrada | CÃ©dula con letras, placa muy corta | Mensaje inline debajo del campo afectado |
| Conflicto de negocio | CÃ©dula duplicada, placa ya registrada | `messagebox.showerror()` con descripciÃ³n del conflicto |
| Error de I/O al guardar | Disco lleno, permisos denegados | `messagebox.showerror()` indicando que no se pudo guardar; operaciÃ³n cancelada |
| Error de I/O al cargar | JSON malformado, archivo corrupto | `messagebox.showwarning()` al inicio; tablas se muestran vacÃ­as |
| Repositorio de usuarios no disponible | `usuarios.json` ausente o corrupto | Denegar acceso; mostrar `messagebox.showerror()` |
| Referencia invÃ¡lida en Servicio | Placa o cÃ©dula no registrada | Mensaje de error especÃ­fico antes de intentar guardar |

### Estrategia de manejo en repositorios

```python
# PatrÃ³n en JSONStorage.guardar()
def guardar(self, datos: list[dict]) -> None:
    ruta_tmp = self.ruta + ".tmp"
    try:
        with open(ruta_tmp, "w", encoding="utf-8") as f:
            json.dump(datos, f, ensure_ascii=False, indent=2)
        os.replace(ruta_tmp, self.ruta)
    except OSError as e:
        # Limpiar archivo temporal si existe
        if os.path.exists(ruta_tmp):
            os.remove(ruta_tmp)
        raise StorageError(f"No se pudo guardar en {self.ruta}: {e}") from e
```

```python
# PatrÃ³n en JSONStorage.cargar()
def cargar(self) -> list[dict]:
    if not os.path.exists(self.ruta):
        return []
    try:
        with open(self.ruta, "r", encoding="utf-8") as f:
            return json.load(f)
    except (json.JSONDecodeError, OSError) as e:
        raise StorageError(f"No se pudo leer {self.ruta}: {e}") from e
```

### Manejo de bloqueo de autenticaciÃ³n

El `AuthController` mantiene en memoria el contador de intentos y el timestamp del primer intento fallido de la serie actual. Al iniciar la aplicaciÃ³n, el bloqueo no persiste entre reinicios (decisiÃ³n de diseÃ±o: para uso monousuario de escritorio, reiniciar la app equivale a reiniciar el perÃ­odo de bloqueo â€” aceptable dado el contexto).

### PropagaciÃ³n de errores hacia la vista

Los controladores nunca lanzan excepciones hacia las vistas. En su lugar retornan `ResultadoOperacion(exito=False, mensaje=..., errores=[...])`. Las vistas inspeccionan este objeto y muestran el feedback correspondiente.

---

## Testing Strategy

### Enfoque dual: pruebas de ejemplo + pruebas de propiedades

La estrategia combina dos tipos complementarios de prueba:

- **Pruebas de ejemplo (unittest)**: verifican comportamientos concretos, casos lÃ­mite y flujos de integraciÃ³n.
- **Pruebas de propiedades (hypothesis)**: verifican propiedades universales sobre el espacio de entradas completo con generaciÃ³n aleatoria de datos.

### Biblioteca de property-based testing

Se utiliza **[Hypothesis](https://hypothesis.readthedocs.io/)** (Python), la librerÃ­a de PBT mÃ¡s madura del ecosistema Python.

InstalaciÃ³n:
```
pip install hypothesis
```

Cada prueba de propiedad se configura con un mÃ­nimo de 100 iteraciones:
```python
from hypothesis import given, settings
from hypothesis import strategies as st

@settings(max_examples=100)
@given(...)
def test_propiedad_N(...):
    ...
```

Cada prueba referencia su propiedad de diseÃ±o con un comentario:
```python
# Feature: serviteca-gestion, Property 5: Unicidad de cÃ©dula en el sistema
```

### Cobertura por mÃ³dulo

#### MÃ³dulo de AutenticaciÃ³n (`test_auth_properties.py`, `test_auth_controller.py`)

| Tipo | QuÃ© verifica |
|---|---|
| Propiedad | Prop 1 â€” autenticaciÃ³n correcta para cualquier credencial |
| Propiedad | Prop 2 â€” rechazo de campos vacÃ­os |
| Propiedad | Prop 3 â€” bloqueo tras N â‰¥ 5 intentos fallidos |
| Ejemplo | Hash SHA-256 correcto al crear usuario |
| Ejemplo | Repositorio no disponible â†’ acceso denegado |
| Ejemplo | Tiempo de validaciÃ³n â‰¤ 3 segundos |

#### MÃ³dulo de Clientes (`test_clientes_properties.py`, `test_clientes_controller.py`)

| Tipo | QuÃ© verifica |
|---|---|
| Propiedad | Prop 4 â€” validaciÃ³n de campos obligatorios |
| Propiedad | Prop 5 â€” unicidad de cÃ©dula |
| Propiedad | Prop 6 â€” estado inicial "Activo" |
| Propiedad | Prop 7 â€” filtrado case-insensitive |
| Propiedad | Prop 8 â€” round-trip de carga para ediciÃ³n |
| Propiedad | Prop 9 â€” suspensiÃ³n de cliente activo |
| Ejemplo | EliminaciÃ³n con confirmaciÃ³n |
| Ejemplo | Indicador visual diferenciado Activo/Suspendido |

#### MÃ³dulo de Carros (`test_carros_properties.py`, `test_carros_controller.py`)

| Tipo | QuÃ© verifica |
|---|---|
| Propiedad | Prop 10 â€” validaciÃ³n de formato de placa |
| Propiedad | Prop 11 â€” unicidad de placa |
| Propiedad | Prop 12 â€” filtrado case-insensitive |
| Propiedad | Prop 13 â€” suspensiÃ³n de carro activo |
| Ejemplo | Round-trip de carga para ediciÃ³n (anÃ¡logo a Prop 8) |
| Ejemplo | Estado inicial "Activo" al crear carro |

#### MÃ³dulo de Servicios (`test_servicios_properties.py`, `test_servicios_controller.py`)

| Tipo | QuÃ© verifica |
|---|---|
| Propiedad | Prop 14 â€” rechazo de fechas futuras |
| Propiedad | Prop 15 â€” integridad referencial |
| Propiedad | Prop 16 â€” completitud del historial |
| Propiedad | Prop 17 â€” orden descendente del historial |
| Ejemplo | Formulario con las 5 opciones exactas de Tipo_Servicio |
| Ejemplo | Reset del formulario tras guardado exitoso |

#### Persistencia (`test_persistencia_properties.py`)

| Tipo | QuÃ© verifica |
|---|---|
| Propiedad | Prop 19 â€” round-trip de serializaciÃ³n JSON |
| Ejemplo | Escritura atÃ³mica: no corrupciÃ³n ante interrupciÃ³n simulada |
| Ejemplo | Carga de datos al inicio con archivo ausente â†’ lista vacÃ­a |
| Ejemplo | Carga con JSON malformado â†’ StorageError controlado |

### ConfiguraciÃ³n de ejecuciÃ³n

Ejecutar todas las pruebas:
```bash
python -m pytest tests/ -v
```

Ejecutar solo pruebas de propiedades:
```bash
python -m pytest tests/property/ -v
```

Ejecutar solo pruebas de ejemplo:
```bash
python -m pytest tests/unit/ -v
```

### Generadores Hypothesis para este proyecto

```python
from hypothesis import strategies as st
import string

# Generador de cÃ©dulas vÃ¡lidas (1-10 dÃ­gitos)
cedula_valida = st.text(alphabet=string.digits, min_size=1, max_size=10).filter(lambda s: s.lstrip("0") != "" or s == "0")

# Generador de placas vÃ¡lidas (5-7 alfanumÃ©ricos)
placa_valida = st.text(alphabet=string.ascii_letters + string.digits, min_size=5, max_size=7)

# Generador de placas invÃ¡lidas (fuera del rango o con caracteres especiales)
placa_invalida = st.one_of(
    st.text(alphabet=string.ascii_letters + string.digits, max_size=4),   # muy corta
    st.text(alphabet=string.ascii_letters + string.digits, min_size=8),   # muy larga
    st.text(alphabet=string.printable, min_size=5, max_size=7).filter(   # con especiales
        lambda s: any(c not in string.ascii_letters + string.digits for c in s)
    )
)

# Generador de fechas pasadas o presentes
from datetime import date, timedelta
fecha_valida = st.dates(max_value=date.today())
fecha_futura = st.dates(min_value=date.today() + timedelta(days=1))
```

### Nota sobre pruebas de UI (tkinter)

Los componentes tkinter no se prueban de forma directa con Hypothesis dado que requieren un entorno de display. La estrategia es:

1. **LÃ³gica de negocio y validaciÃ³n**: 100% testeable de forma unitaria y con propiedades (sin tkinter).
2. **Controladores**: testeables de forma unitaria usando repositorios mock en memoria.
3. **Vistas**: verificables mediante pruebas de ejemplo con `tk.Tk()` instanciado en modo headless (usando variable de entorno `DISPLAY=:0` en Linux o mocking de `messagebox` en tests).

Las propiedades definidas en este documento se implementan sobre la capa de controladores y repositorios, que son completamente independientes de tkinter.

---

## Interfaz de Usuario — Paleta de Color y Modo Claro/Oscuro

### Paleta de colores

La interfaz usa una gama activa y llamativa basada en tres colores base:

| Token CSS | Modo Oscuro | Modo Claro | Uso |
|---|---|---|---|
| `--clr-bg` | `#121820` | `#f4f6fa` | Fondo general de la aplicacion |
| `--clr-surface` | `#1a2230` | `#ffffff` | Superficies secundarias (topbar, sidebar) |
| `--clr-card` | `#1f2d3d` | `#ffffff` | Tarjetas, tablas, modales |
| `--clr-primary` | `#FF6B00` | `#FF6B00` | Accion principal, botones, enlaces, acento naranja |
| `--clr-text` | `#FFFFFF` | `#121820` | Texto principal |
| `--clr-muted` | `#8a9bb0` | `#5a6a7a` | Texto secundario, etiquetas, placeholder |
| `--clr-danger` | `#ff4d4d` | `#e03030` | Errores, eliminacion |
| `--clr-border` | `#2a3d52` | `#d8e0ea` | Bordes de elementos |

> **Decision de diseno**: el naranja `#FF6B00` se usa como unico color de acento en ambos temas para mantener coherencia visual. El fondo oscuro `#121820` provee contraste alto con el texto blanco `#FFFFFF`, cumpliendo WCAG AA para texto de cuerpo.

### Modo claro / modo oscuro

- El tema se almacena en `localStorage` bajo la clave `srv_theme` (`"dark"` o `"light"`).
- Al cargar la pagina, se lee el tema guardado y se aplica inmediatamente via `document.documentElement.setAttribute('data-theme', theme)`.
- Un boton "theme-toggle" aparece en tres pantallas: Landing, Login y la barra superior de la app (topbar).
- El cambio de tema es instantaneo, sin recarga de pagina, gracias a las variables CSS definidas en `:root/[data-theme="dark"]` y `[data-theme="light"]`.

```
applyTheme(theme: "dark" | "light"):
  1. setAttribute("data-theme", theme) en <html>
  2. localStorage.setItem("srv_theme", theme)
  3. Actualiza el label de todos los botones toggle
```

### Relacion uno a muchos: Cliente -> Carros (UI)

- **Modal de carros**: incluye campo "Cedula del propietario" con `<datalist>` autocompletado con cedulas y nombres de clientes registrados.
- **Preview en tiempo real**: al escribir la cedula, aparece el nombre del propietario encontrado (o advertencia si no existe).
- **Validacion**: `guardarCarro()` verifica que `cedula_propietario` exista en el repositorio de clientes antes de persistir.
- **Panel de vehiculos del cliente**: en la tabla de Clientes, el boton carro abre un panel inline que muestra todos los autos vinculados al cliente seleccionado (`verCarrosCliente(cedula)`).
- **Busqueda cruzada**: el filtro de la tabla de Carros permite buscar por nombre del propietario ademas de placa, marca y modelo.
