# Implementation Plan: Serviteca ADSO

## Overview

Implementación incremental de la aplicación de escritorio **Serviteca ADSO** siguiendo la arquitectura MVC con Python 3.10+, GUI tkinter y persistencia en archivos JSON locales. Las tareas se organizan en capas de dependencia: infraestructura → modelos → persistencia → controladores → vistas → pruebas → integración final.

---

## Tasks

- [ ] 1. Configuración inicial del proyecto
  - [ ] 1.1 Crear la estructura de directorios del proyecto
    - Crear los directorios: `serviteca_adso/`, `serviteca_adso/models/`, `serviteca_adso/repositories/`, `serviteca_adso/controllers/`, `serviteca_adso/views/`, `serviteca_adso/data/`, `tests/`, `tests/unit/`, `tests/property/`
    - Crear los archivos `__init__.py` vacíos en cada paquete Python (`models/`, `repositories/`, `controllers/`, `views/`)
    - _Requisitos: 9.1, 9.2_

  - [ ] 1.2 Crear `config.py` con constantes globales
    - Definir las rutas a los archivos JSON: `RUTA_USUARIOS`, `RUTA_CLIENTES`, `RUTA_CARROS`, `RUTA_SERVICIOS` (usando `pathlib.Path`)
    - Definir las constantes del sistema: `MAX_INTENTOS_AUTH = 5`, `TIEMPO_BLOQUEO_SEG = 900`, `MAX_REGISTROS = 10_000`
    - _Requisitos: 1.7, 9.2_

  - [ ] 1.3 Crear `main.py` — punto de entrada de la aplicación
    - Instanciar `tk.Tk()`, configurar el título y tamaño mínimo de ventana
    - Instanciar `AppController(root)` y llamar a `controller.start()`
    - Llamar a `root.mainloop()`
    - _Requisitos: 2.1, 2.2_

  - [ ] 1.4 Crear `requirements.txt` con las dependencias de prueba
    - Agregar `pytest>=7.0` y `hypothesis>=6.0` con versiones fijas
    - _Requisitos: (infraestructura de pruebas)_

- [ ] 2. Capa de modelos — dataclasses y enumeraciones
  - [ ] 2.1 Implementar `models/usuario.py`
    - Definir `@dataclass class Usuario` con campos `username: str` y `password_hash: str`
    - Implementar método de clase `from_dict(d: dict) -> Usuario` y método de instancia `to_dict() -> dict`
    - _Requisitos: 1.3, 1.4_

  - [ ] 2.2 Implementar `models/cliente.py`
    - Definir `class EstadoCliente(str, Enum)` con valores `ACTIVO = "Activo"` y `SUSPENDIDO = "Suspendido"`
    - Definir `@dataclass class Cliente` con campos: `cedula`, `nombres`, `apellidos`, `celular`, `correo = ""`, `estado = EstadoCliente.ACTIVO`
    - Implementar `from_dict()` y `to_dict()` con serialización del enum
    - _Requisitos: 3.1, 3.4, 3.5_

  - [ ] 2.3 Implementar `models/carro.py`
    - Definir `class EstadoCarro(str, Enum)` con valores `ACTIVO = "Activo"` e `INACTIVO = "Inactivo"`
    - Definir `@dataclass class Carro` con campos: `placa`, `marca`, `modelo`, `estado = EstadoCarro.ACTIVO`
    - Implementar `from_dict()` y `to_dict()`
    - _Requisitos: 4.1, 4.5_

  - [ ] 2.4 Implementar `models/servicio.py`
    - Definir `class TipoServicio(str, Enum)` con los 5 valores exactos del requisito
    - Definir `@dataclass class Servicio` con campos: `id_servicio`, `placa_carro`, `cedula_cliente`, `fecha`, `tipo`, `notas = ""`
    - Implementar `from_dict()` y `to_dict()` con serialización de `date` como cadena ISO 8601
    - _Requisitos: 5.1, 5.3_

  - [ ] 2.5 Implementar `models/resultado_operacion.py`
    - Definir `@dataclass class ResultadoOperacion` con campos: `exito: bool`, `mensaje: str = ""`, `errores: list[str] = field(default_factory=list)`, `dato: object = None`
    - _Requisitos: (DTO compartido para todos los controladores)_

  - [ ]* 2.6 Escribir pruebas de propiedad para round-trip de serialización de modelos
    - **Propiedad 19: Round-trip de serialización JSON**
    - Para cada modelo (Cliente, Carro, Servicio), generar instancias con Hypothesis y verificar que `Model.from_dict(instancia.to_dict())` produce una entidad equivalente campo a campo
    - **Valida: Requisito 9.1**
    - Archivo: `tests/property/test_persistencia_properties.py`

- [ ] 3. Capa de persistencia — JSONStorage y repositorios
  - [ ] 3.1 Implementar `repositories/json_storage.py`
    - Implementar `class JSONStorage` con `__init__(self, ruta: str)`, `cargar() -> list[dict]` y `guardar(datos: list[dict]) -> None`
    - En `guardar()`: escribir a archivo `.tmp` y luego `os.replace()` para escritura atómica
    - En `cargar()`: retornar lista vacía si el archivo no existe; lanzar `StorageError` en JSON malformado o error de I/O
    - Definir excepción personalizada `StorageError`
    - _Requisitos: 9.1, 9.3, 9.4_

  - [ ]* 3.2 Escribir pruebas de ejemplo para JSONStorage
    - Verificar escritura atómica: el archivo `.tmp` no queda tras un guardado exitoso
    - Verificar que `cargar()` retorna lista vacía si el archivo no existe
    - Verificar que `cargar()` lanza `StorageError` con JSON malformado
    - Archivo: `tests/unit/test_json_storage.py`

  - [ ] 3.3 Implementar `repositories/usuario_repository.py`
    - Implementar `class UsuarioRepository` con `obtener_por_username(username: str) -> Optional[Usuario]` y `obtener_todos() -> list[Usuario]`
    - Usar `JSONStorage` para carga y persistencia
    - _Requisitos: 1.3, 1.8_

  - [ ] 3.4 Implementar `repositories/cliente_repository.py`
    - Implementar `class ClienteRepository` con métodos: `obtener_todos()`, `obtener_por_cedula(cedula)`, `agregar(cliente)`, `actualizar(cliente)`, `eliminar(cedula)`, `existe_cedula(cedula, excluir="")`
    - _Requisitos: 3.2, 3.3, 3.8, 3.9_

  - [ ] 3.5 Implementar `repositories/carro_repository.py`
    - Implementar `class CarroRepository` con métodos: `obtener_todos()`, `obtener_por_placa(placa)`, `agregar(carro)`, `actualizar(carro)`, `eliminar(placa)`, `existe_placa(placa, excluir="")`
    - _Requisitos: 4.2, 4.4, 4.9_

  - [ ] 3.6 Implementar `repositories/servicio_repository.py`
    - Implementar `class ServicioRepository` con métodos: `obtener_todos()`, `agregar(servicio)`, `obtener_por_placa(placa) -> list[Servicio]` ordenado por fecha descendente
    - _Requisitos: 6.2, 6.6_

  - [ ]* 3.7 Escribir prueba de propiedad para round-trip de persistencia completa
    - **Propiedad 19: Round-trip de serialización JSON (persistencia a disco)**
    - Generar listas de entidades con Hypothesis, guardarlas con el repositorio y verificar que cargarlas de nuevo produce entidades equivalentes
    - **Valida: Requisito 9.1**
    - Archivo: `tests/property/test_persistencia_properties.py`

- [ ] 4. Checkpoint — verificar persistencia
  - Asegurarse de que todas las pruebas de la capa de modelos y repositorios pasan. Consultar al usuario si surge alguna duda.

- [ ] 5. Controladores — lógica de negocio y validaciones
  - [ ] 5.1 Implementar `controllers/auth_controller.py`
    - Implementar `class AuthController` con `autenticar(usuario, contraseña) -> ResultadoAuth`, `registrar_intento_fallido()`, `esta_bloqueado() -> bool`, `segundos_restantes_bloqueo() -> int`, `reiniciar_intentos()`
    - Aplicar SHA-256 con `hashlib` para comparación de contraseñas; nunca comparar texto plano
    - Bloquear tras 5 intentos fallidos consecutivos durante 900 segundos
    - _Requisitos: 1.3, 1.4, 1.5, 1.6, 1.7, 1.8_

  - [ ]* 5.2 Escribir prueba de propiedad — autenticación correcta para cualquier credencial
    - **Propiedad 1: Autenticación correcta para cualquier credencial**
    - Generar pares (usuario, contraseña) válidos e inválidos con Hypothesis y verificar que el resultado es éxito si y solo si coincide con el repositorio
    - **Valida: Requisitos 1.4, 1.5**
    - Archivo: `tests/property/test_auth_properties.py`

  - [ ]* 5.3 Escribir prueba de propiedad — rechazo de campos vacíos en autenticación
    - **Propiedad 2: Rechazo de credenciales con campos vacíos**
    - Generar combinaciones con usuario y/o contraseña vacíos; verificar rechazo sin consultar repositorio
    - **Valida: Requisito 1.6**
    - Archivo: `tests/property/test_auth_properties.py`

  - [ ]* 5.4 Escribir prueba de propiedad — bloqueo por intentos fallidos consecutivos
    - **Propiedad 3: Bloqueo por intentos fallidos consecutivos**
    - Simular N ≥ 5 intentos fallidos consecutivos con Hypothesis y verificar que `esta_bloqueado()` retorna `True`
    - **Valida: Requisito 1.7**
    - Archivo: `tests/property/test_auth_properties.py`

  - [ ] 5.5 Implementar `controllers/clientes_controller.py`
    - Implementar `class ClientesController` con métodos: `crear_cliente(datos)`, `editar_cliente(cedula_original, datos)`, `eliminar_cliente(cedula)`, `suspender_cliente(cedula)`, `buscar_clientes(texto)`, `obtener_todos()` y `_validar_datos_cliente(datos, cedula_excluir="")`
    - Validar: cédula solo dígitos 1–10 chars, nombres/apellidos no vacíos ≤ 100, celular solo dígitos 1–15, correo formato válido si no vacío
    - Retornar siempre `ResultadoOperacion`; no lanzar excepciones hacia las vistas
    - _Requisitos: 3.2, 3.3, 3.4, 3.6, 3.7, 3.8, 3.9, 3.10, 3.11, 3.13_

  - [ ]* 5.6 Escribir prueba de propiedad — validación de campos obligatorios de Cliente
    - **Propiedad 4: Validación de campos obligatorios de Cliente**
    - Generar datos con campos obligatorios vacíos o con formato inválido; verificar que `_validar_datos_cliente()` retorna errores y no persiste
    - **Valida: Requisitos 3.2, 3.8, 3.13**
    - Archivo: `tests/property/test_clientes_properties.py`

  - [ ]* 5.7 Escribir prueba de propiedad — unicidad de cédula en el sistema
    - **Propiedad 5: Unicidad de cédula en el sistema**
    - Insertar un cliente con cédula X; intentar crear/editar otro con la misma cédula y verificar rechazo
    - **Valida: Requisitos 3.3, 3.9**
    - Archivo: `tests/property/test_clientes_properties.py`

  - [ ]* 5.8 Escribir prueba de propiedad — estado inicial "Activo" al crear Cliente
    - **Propiedad 6: Estado inicial "Activo" al crear cliente**
    - Generar datos de cliente válidos con Hypothesis y verificar que el cliente creado tiene `estado = EstadoCliente.ACTIVO`
    - **Valida: Requisito 3.4**
    - Archivo: `tests/property/test_clientes_properties.py`

  - [ ]* 5.9 Escribir prueba de propiedad — filtrado de clientes case-insensitive y completo
    - **Propiedad 7: Filtrado de clientes es case-insensitive y completo**
    - Generar listas de clientes y textos de búsqueda; verificar que todos los resultados contienen el texto y ningún coincidente es excluido
    - **Valida: Requisito 3.6**
    - Archivo: `tests/property/test_clientes_properties.py`

  - [ ]* 5.10 Escribir prueba de propiedad — round-trip de carga de Cliente para edición
    - **Propiedad 8: Round-trip de carga de cliente para edición**
    - Generar clientes, guardarlos y verificar que `obtener_todos()` retorna exactamente los mismos campos sin alteración
    - **Valida: Requisito 3.7**
    - Archivo: `tests/property/test_clientes_properties.py`

  - [ ]* 5.11 Escribir prueba de propiedad — suspensión de cliente activo
    - **Propiedad 9: Suspensión de cliente activo**
    - Para cualquier cliente con estado `ACTIVO`, verificar que `suspender_cliente()` produce estado `SUSPENDIDO` y persiste el cambio
    - **Valida: Requisito 3.11**
    - Archivo: `tests/property/test_clientes_properties.py`

  - [ ] 5.12 Implementar `controllers/carros_controller.py`
    - Implementar `class CarrosController` con métodos: `crear_carro(datos)`, `editar_carro(placa_original, datos)`, `eliminar_carro(placa)`, `suspender_carro(placa)`, `buscar_carros(texto)`, `obtener_todos()` y `_validar_datos_carro(datos, placa_excluir="")`
    - Validar: placa solo alfanuméricos 5–7 chars, marca/modelo no vacíos ≤ 50
    - _Requisitos: 4.2, 4.3, 4.4, 4.6, 4.7, 4.8, 4.9, 4.10, 4.11, 4.12_

  - [ ]* 5.13 Escribir prueba de propiedad — validación de formato de placa de Carro
    - **Propiedad 10: Validación de formato de placa de Carro**
    - Generar placas válidas (5–7 alfanuméricos) e inválidas con Hypothesis; verificar aceptación/rechazo correcto
    - **Valida: Requisitos 4.2, 4.3**
    - Archivo: `tests/property/test_carros_properties.py`

  - [ ]* 5.14 Escribir prueba de propiedad — unicidad de placa en el sistema
    - **Propiedad 11: Unicidad de placa en el sistema**
    - Insertar un carro con placa X; intentar crear/editar otro con la misma placa y verificar rechazo
    - **Valida: Requisitos 4.4, 4.9**
    - Archivo: `tests/property/test_carros_properties.py`

  - [ ]* 5.15 Escribir prueba de propiedad — filtrado de carros case-insensitive y completo
    - **Propiedad 12: Filtrado de carros es case-insensitive y completo**
    - Generar listas de carros y textos de búsqueda; verificar completitud e insensibilidad a mayúsculas
    - **Valida: Requisito 4.6**
    - Archivo: `tests/property/test_carros_properties.py`

  - [ ]* 5.16 Escribir prueba de propiedad — suspensión de carro activo
    - **Propiedad 13: Suspensión de carro activo**
    - Para cualquier carro con estado `ACTIVO`, verificar que `suspender_carro()` produce estado `INACTIVO` y persiste el cambio
    - **Valida: Requisito 4.12**
    - Archivo: `tests/property/test_carros_properties.py`

  - [ ] 5.17 Implementar `controllers/servicios_controller.py`
    - Implementar `class ServiciosController` con métodos: `registrar_servicio(datos)`, `obtener_historial_por_placa(placa) -> list[Servicio]` y `_validar_datos_servicio(datos)`
    - Validar: placa y cédula existen en sus repositorios respectivos; fecha ≤ `date.today()`; tipo es valor válido de `TipoServicio`; notas ≤ 500 chars
    - Generar `id_servicio` con `uuid.uuid4()`
    - _Requisitos: 5.2, 5.4, 5.5, 5.6, 5.7, 5.8, 6.2, 6.6_

  - [ ]* 5.18 Escribir prueba de propiedad — rechazo de fechas futuras en Servicios
    - **Propiedad 14: Rechazo de fechas futuras en Servicios**
    - Generar fechas futuras con Hypothesis (`st.dates(min_value=date.today() + timedelta(days=1))`) y verificar rechazo
    - **Valida: Requisito 5.2**
    - Archivo: `tests/property/test_servicios_properties.py`

  - [ ]* 5.19 Escribir prueba de propiedad — integridad referencial al registrar un Servicio
    - **Propiedad 15: Integridad referencial al registrar un Servicio**
    - Generar datos de servicio con placa o cédula inexistentes; verificar rechazo. Con referencias válidas, verificar guardado exitoso
    - **Valida: Requisitos 5.4, 5.5, 5.6, 5.7**
    - Archivo: `tests/property/test_servicios_properties.py`

  - [ ]* 5.20 Escribir prueba de propiedad — completitud del historial de servicios por placa
    - **Propiedad 16: Completitud del historial de servicios por placa**
    - Insertar N servicios para una placa y verificar que `obtener_historial_por_placa()` retorna exactamente N sin omisiones ni duplicados
    - **Valida: Requisito 6.2**
    - Archivo: `tests/property/test_servicios_properties.py`

  - [ ]* 5.21 Escribir prueba de propiedad — orden descendente del historial de servicios
    - **Propiedad 17: Orden descendente del historial de servicios**
    - Generar servicios con fechas aleatorias; verificar que el historial retornado está ordenado de forma descendente por fecha
    - **Valida: Requisito 6.6**
    - Archivo: `tests/property/test_servicios_properties.py`

  - [ ] 5.22 Implementar `controllers/app_controller.py`
    - Implementar `class AppController` con métodos: `start()`, `on_login_success(usuario)`, `on_logout()`, `navigate_to(modulo)`, `require_auth() -> bool`
    - Gestionar el ciclo de vida de la sesión: guardar usuario autenticado, invalidar sesión en logout
    - Redirigir a `LoginView` si `require_auth()` retorna `False`
    - _Requisitos: 2.3, 2.4, 8.2, 8.3, 8.4, 8.5_

  - [ ]* 5.23 Escribir prueba de propiedad — sesión cerrada impide acceso a módulos protegidos
    - **Propiedad 18: Sesión cerrada impide acceso a módulos protegidos**
    - Verificar que `require_auth()` retorna `False` tras `on_logout()` y que cualquier intento de navegación redirige al login
    - **Valida: Requisitos 8.4, 8.5**
    - Archivo: `tests/unit/test_auth_controller.py`

- [ ] 6. Checkpoint — verificar controladores y pruebas de propiedades
  - Ejecutar `python -m pytest tests/ -v` y confirmar que todas las pruebas pasan. Consultar al usuario si surge alguna duda.

- [ ] 7. Capa de vistas — interfaces tkinter
  - [ ] 7.1 Implementar `views/base_view.py`
    - Definir `class BaseView(tk.Frame)` con métodos abstractos: `mostrar()`, `ocultar()`, `show_error(mensaje)`, `show_info(mensaje)`
    - Incluir helpers de layout: método para crear campos de formulario etiquetados y método para crear botones estandarizados
    - _Requisitos: 2.5_

  - [ ] 7.2 Implementar `views/login_view.py`
    - Crear formulario con campos "Usuario" y "Contraseña" (con `show="*"`)
    - Botón "Ingresar" que llama al controlador de autenticación
    - Mostrar mensajes de error y estado de bloqueo con tiempo restante
    - _Requisitos: 1.1, 1.2, 1.5, 1.6, 1.7_

  - [ ] 7.3 Implementar `views/main_view.py`
    - Crear frame principal con menú lateral (Clientes, Carros, Servicios, Ayuda, Cerrar Sesión)
    - Mostrar nombre del usuario autenticado en la cabecera con fuente mínima de 12px
    - Mantener el menú visible en todas las pantallas sin superponer el contenido
    - _Requisitos: 2.1, 2.2, 2.5_

  - [ ] 7.4 Implementar `views/clientes_view.py`
    - Crear tabla (`ttk.Treeview`) con columnas: Cédula, Nombres, Apellidos, Celular, Estado
    - Aplicar indicador visual diferenciado (color de fila o tag) para registros Activo/Suspendido
    - Crear formulario de registro/edición con los 5 campos del requisito
    - Crear campo de búsqueda con filtrado en tiempo real al escribir
    - Botones de acción por fila: Editar, Eliminar, Suspender
    - Confirmar antes de eliminar con `messagebox.askyesno()`
    - _Requisitos: 3.1, 3.5, 3.6, 3.7, 3.10, 3.11, 3.12_

  - [ ] 7.5 Implementar `views/carros_view.py`
    - Crear tabla con columnas: Placa, Marca, Modelo, Estado
    - Aplicar indicador visual diferenciado para Activo/Inactivo
    - Crear formulario de registro/edición con los 3 campos del requisito
    - Crear campo de búsqueda con filtrado en tiempo real
    - Botones de acción por fila: Editar, Eliminar, Suspender
    - _Requisitos: 4.1, 4.5, 4.6, 4.7, 4.10, 4.11, 4.12, 4.13_

  - [ ] 7.6 Implementar `views/servicios_view.py`
    - Crear formulario de registro con: campo Placa, campo Cédula, selector de fecha (`DateEntry` con tkinter o `ttk.Combobox` de partes de fecha), lista desplegable `TipoServicio` con las 5 opciones exactas, campo Notas (máx. 500 chars)
    - Restablecer todos los campos tras guardado exitoso
    - Crear sección de consulta histórica: campo Placa + botón "Buscar" + tabla de resultados (ID, Fecha, Tipo, Notas)
    - Mostrar resultados ordenados por fecha descendente
    - _Requisitos: 5.1, 5.2, 5.3, 5.8, 6.1, 6.2, 6.3, 6.4, 6.5, 6.6_

  - [ ] 7.7 Implementar `views/ayuda_view.py`
    - Crear pantalla con contenido estático dividido en secciones: Clientes, Carros, Servicios
    - Cada sección visualmente separada con el nombre del módulo como encabezado
    - Incluir mínimo una instrucción operativa por módulo (registrar, editar, eliminar)
    - Máximo 3 niveles de jerarquía por sección; usar `tk.Text` o `ttk.Label` con scroll
    - _Requisitos: 7.1, 7.2, 7.3, 7.4_

- [ ] 8. Pruebas unitarias de controladores
  - [ ] 8.1 Escribir pruebas unitarias para `AuthController`
    - Verificar hash SHA-256 correcto al crear usuario
    - Verificar comportamiento cuando el repositorio no está disponible (acceso denegado)
    - Verificar que el tiempo de validación es ≤ 3 segundos
    - Archivo: `tests/unit/test_auth_controller.py`

  - [ ] 8.2 Escribir pruebas unitarias para `ClientesController`
    - Verificar eliminación con confirmación: cliente eliminado ya no aparece en `obtener_todos()`
    - Verificar indicador visual: campo `estado` diferencia correctamente Activo/Suspendido
    - Archivo: `tests/unit/test_clientes_controller.py`

  - [ ] 8.3 Escribir pruebas unitarias para `CarrosController`
    - Verificar round-trip de carga para edición (análogo a Prop 8)
    - Verificar que el estado inicial al crear un carro es `ACTIVO`
    - Archivo: `tests/unit/test_carros_controller.py`

  - [ ] 8.4 Escribir pruebas unitarias para `ServiciosController`
    - Verificar formulario con las 5 opciones exactas de `TipoServicio`
    - Verificar reset del formulario tras guardado exitoso (todos los campos vacíos o por defecto)
    - Archivo: `tests/unit/test_servicios_controller.py`

- [ ] 9. Integración y datos iniciales
  - [ ] 9.1 Crear el archivo `data/usuarios.json` con el usuario administrador inicial
    - Generar el hash SHA-256 de la contraseña `"admin"` usando `hashlib.sha256(b"admin").hexdigest()`
    - Crear el archivo con el usuario `admin` y el hash calculado
    - _Requisitos: 1.3, 1.4_

  - [ ] 9.2 Crear archivos JSON vacíos de datos iniciales
    - Crear `data/clientes.json`, `data/carros.json` y `data/servicios.json` con contenido `[]`
    - _Requisitos: 9.2_

  - [ ] 9.3 Conectar vistas con controladores en `AppController`
    - En `navigate_to(modulo)`: instanciar la vista correspondiente si no existe, destruir la vista anterior y mostrar la nueva en el área de contenido
    - Pasar los controladores de dominio a cada vista en su constructor
    - Verificar `require_auth()` antes de cargar cualquier módulo protegido
    - _Requisitos: 2.3, 2.4, 8.4, 8.5_

  - [ ] 9.4 Implementar el diálogo de confirmación de cierre de sesión
    - En `MainView`: al pulsar "Cerrar Sesión", mostrar `messagebox.askyesno()` con pregunta explícita
    - Si confirma → llamar `app_controller.on_logout()`; si cancela → cerrar diálogo y mantener sesión
    - _Requisitos: 8.1, 8.2, 8.3_

- [ ] 10. Checkpoint final — verificación completa
  - Ejecutar `python -m pytest tests/ -v` y confirmar que todas las pruebas pasan.
  - Ejecutar `python serviteca_adso/main.py` y verificar el flujo completo: login → pantalla principal → módulos → cierre de sesión.
  - Consultar al usuario si surge alguna duda antes de declarar la implementación completa.

---

## Notes

- Las tareas marcadas con `*` son opcionales y pueden omitirse para un MVP más rápido; representan pruebas unitarias y de propiedades.
- Las tareas de prueba de propiedades usan `@settings(max_examples=100)` de Hypothesis y deben incluir el comentario `# Feature: serviteca-gestion, Property N: <nombre>` al inicio de cada función.
- Los controladores nunca lanzan excepciones hacia las vistas; siempre retornan `ResultadoOperacion`.
- La escritura atómica en `JSONStorage` usa `os.replace()` para garantizar que no haya corrupción ante cierres inesperados.
- Los componentes tkinter (`LoginView`, `MainView`, etc.) se prueban mediante repositorios mock en memoria; no requieren display real para las pruebas de controladores.
- El bloqueo de autenticación no persiste entre reinicios de la aplicación (decisión de diseño para uso monousuario de escritorio).

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2", "1.4"] },
    { "id": 1, "tasks": ["1.3", "2.1", "2.2", "2.3", "2.4", "2.5"] },
    { "id": 2, "tasks": ["2.6", "3.1"] },
    { "id": 3, "tasks": ["3.2", "3.3", "3.4", "3.5", "3.6"] },
    { "id": 4, "tasks": ["3.7", "5.1", "5.5", "5.12", "5.17"] },
    { "id": 5, "tasks": ["5.2", "5.3", "5.4", "5.6", "5.7", "5.8", "5.9", "5.10", "5.11", "5.13", "5.14", "5.15", "5.16", "5.18", "5.19", "5.20", "5.21"] },
    { "id": 6, "tasks": ["5.22"] },
    { "id": 7, "tasks": ["5.23", "7.1"] },
    { "id": 8, "tasks": ["7.2", "7.3"] },
    { "id": 9, "tasks": ["7.4", "7.5", "7.6", "7.7"] },
    { "id": 10, "tasks": ["8.1", "8.2", "8.3", "8.4"] },
    { "id": 11, "tasks": ["9.1", "9.2"] },
    { "id": 12, "tasks": ["9.3", "9.4"] }
  ]
}
```
