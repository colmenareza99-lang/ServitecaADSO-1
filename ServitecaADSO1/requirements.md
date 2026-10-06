# Requirements Document

## Introduction

El sistema de gestión para serviteca es una aplicación de escritorio interactiva diseñada para administrar clientes, vehículos y servicios automotrices prestados en un taller. El sistema permite al personal del taller registrar y consultar información de clientes y sus vehículos, registrar los servicios realizados, y consultar el historial de mantenimiento por vehículo. El acceso al sistema está protegido mediante autenticación con usuario y contraseña.

---

## Glossary

- **Sistema**: La aplicación de escritorio de gestión para la serviteca.
- **Usuario_Autenticado**: El empleado del taller que ha iniciado sesión correctamente en el Sistema.
- **Módulo_Autenticación**: El componente que gestiona el inicio y cierre de sesión.
- **Módulo_Clientes**: El componente que gestiona el registro, edición, búsqueda y eliminación de clientes.
- **Módulo_Carros**: El componente que gestiona el registro, edición, búsqueda y eliminación de vehículos.
- **Módulo_Servicios**: El componente que gestiona el registro y la consulta histórica de servicios prestados.
- **Módulo_Ayuda**: El componente que muestra las instrucciones de uso del Sistema.
- **Cliente**: Persona natural identificada por cédula que tiene uno o más vehículos registrados en la serviteca.
- **Carro**: Vehículo identificado por su placa, vinculado obligatoriamente a un Cliente propietario mediante su Cédula. Un Cliente puede tener muchos Carros (relación uno a muchos Cliente → Carros).
- **Propietario_Carro**: El Cliente al que pertenece un Carro, identificado por su Cédula en el registro del vehículo.
- **Servicio**: Registro de un trabajo automotriz realizado a un vehículo en una fecha específica.
- **Credenciales**: Combinación de nombre de usuario y contraseña utilizados para autenticar el acceso al Sistema.
- **Placa**: Identificador alfanumérico único que identifica a un Carro en el Sistema.
- **Cédula**: Identificador numérico único que identifica a un Cliente en el Sistema.
- **Estado_Cliente**: Valor que indica si un Cliente está Activo o Suspendido.
- **Estado_Carro**: Valor que indica si un Carro está Activo o Inactivo.
- **Tipo_Servicio**: Categoría del trabajo realizado; puede ser: mantenimiento, cambio de aceite, frenos, revisión general, u otro definido por el usuario.

---

## Requirements

### Requerimiento 1: Autenticación de Usuario

**User Story:** Como empleado de la serviteca, quiero iniciar sesión con usuario y contraseña, para acceder de forma segura al sistema y que solo personal autorizado pueda operarlo.

#### Criterios de Aceptación

1. THE Módulo_Autenticación SHALL presentar un formulario con los campos "Usuario" y "Contraseña" al iniciar la aplicación.
2. THE Módulo_Autenticación SHALL ocultar los caracteres ingresados en el campo "Contraseña" mediante sustitución visual por caracteres enmascarados.
3. WHEN el Usuario hace clic en el botón "Ingresar", THE Módulo_Autenticación SHALL validar las Credenciales contra el repositorio de usuarios registrados en un plazo máximo de 3 segundos.
4. WHEN las Credenciales son válidas, THE Módulo_Autenticación SHALL redirigir al usuario a la Pantalla Principal del Sistema en un plazo máximo de 3 segundos.
5. IF las Credenciales son inválidas, THEN THE Módulo_Autenticación SHALL mostrar un mensaje de error indicando que el usuario o la contraseña son incorrectos, sin revelar cuál de los dos campos es incorrecto.
6. IF el campo "Usuario" o el campo "Contraseña" se encuentran vacíos al hacer clic en "Ingresar", THEN THE Módulo_Autenticación SHALL mostrar un mensaje de validación indicando que ambos campos son obligatorios.
7. IF el Usuario falla la autenticación 5 veces consecutivas, THEN THE Módulo_Autenticación SHALL bloquear el acceso al formulario durante 15 minutos y mostrar un mensaje indicando el tiempo de bloqueo restante.
8. IF el repositorio de usuarios no está disponible al momento de la validación, THEN THE Módulo_Autenticación SHALL mostrar un mensaje de error observable al Usuario indicando que el sistema no está disponible y SHALL denegar el acceso.

---

### Requerimiento 2: Pantalla Principal y Navegación

**User Story:** Como Usuario_Autenticado, quiero ver una pantalla principal con un menú de navegación claro, para acceder rápidamente a cada módulo del sistema y saber con qué cuenta se está operando.

#### Criterios de Aceptación

1. WHEN el Usuario_Autenticado accede a la Pantalla Principal, THE Sistema SHALL mostrar el nombre del usuario que inició sesión en la barra superior o cabecera de la interfaz, con un tamaño de fuente mínimo de 12px.
2. THE Sistema SHALL presentar un menú de navegación con exactamente las siguientes secciones en el orden indicado: Clientes, Carros, Servicios, Ayuda y Cerrar Sesión.
3. WHEN el Usuario_Autenticado selecciona una sección del menú, THE Sistema SHALL cargar el módulo correspondiente en un plazo máximo de 3 segundos sin restablecer ni modificar el estado de autenticación activo.
4. IF un módulo no puede cargarse, THEN THE Sistema SHALL mostrar un mensaje de error al Usuario_Autenticado indicando que el módulo no está disponible y SHALL mantener el menú de navegación accesible.
5. THE Sistema SHALL mantener visible el menú de navegación en todas las pantallas del Sistema mientras el Usuario_Autenticado se encuentre autenticado, sin que el menú superponga ni bloquee el contenido del módulo activo.

---

### Requerimiento 3: Gestión de Clientes

**User Story:** Como Usuario_Autenticado, quiero registrar, editar, buscar y eliminar clientes, para mantener actualizada la información de las personas que utilizan los servicios de la serviteca.

#### Criterios de Aceptación

1. THE Módulo_Clientes SHALL presentar un formulario con los campos: Cédula (máximo 10 dígitos numéricos), Nombres (máximo 100 caracteres), Apellidos (máximo 100 caracteres), Correo Electrónico (formato válido de email, opcional) y Celular (máximo 15 dígitos numéricos) para el registro de un nuevo Cliente.
2. WHEN el Usuario_Autenticado envía el formulario de registro, THE Módulo_Clientes SHALL validar que todos los campos obligatorios (Cédula, Nombres, Apellidos y Celular) estén completos y con formato válido antes de guardar el registro.
3. IF se intenta registrar un Cliente con una Cédula ya existente en el Sistema, THEN THE Módulo_Clientes SHALL mostrar un mensaje de error indicando que la Cédula ya está registrada y no guardará el registro duplicado.
4. WHEN el registro de un nuevo Cliente es guardado exitosamente, THE Módulo_Clientes SHALL persistir el Cliente con Estado_Cliente "Activo" y reflejar el nuevo registro en la tabla en un plazo máximo de 2 segundos.
5. THE Módulo_Clientes SHALL mostrar una tabla con los registros de clientes existentes, incluyendo las columnas: ID/Cédula, Nombres, Apellidos, Celular y Estado_Cliente.
6. WHEN el Usuario_Autenticado ingresa texto en el campo de búsqueda, THE Módulo_Clientes SHALL filtrar en menos de 1 segundo los registros de la tabla mostrando únicamente los clientes cuyo nombre, apellido o cédula contengan el texto ingresado, sin distinción de mayúsculas y minúsculas.
7. WHEN el Usuario_Autenticado selecciona la acción "Editar" sobre un registro de Cliente, THE Módulo_Clientes SHALL cargar los datos del Cliente en el formulario para permitir su modificación.
8. WHEN el Usuario_Autenticado confirma la edición de un Cliente, THE Módulo_Clientes SHALL validar que los campos obligatorios y de formato sigan siendo válidos, actualizar el registro en el Sistema y reflejar los cambios en la tabla en un plazo máximo de 2 segundos.
9. IF durante la edición se intenta asignar una Cédula que ya pertenece a otro Cliente distinto, THEN THE Módulo_Clientes SHALL mostrar un mensaje de error indicando el conflicto y no guardará el cambio.
10. WHEN el Usuario_Autenticado selecciona la acción "Eliminar" sobre un registro de Cliente, THE Módulo_Clientes SHALL solicitar confirmación antes de proceder con la eliminación permanente del registro, y SHALL eliminar el registro y actualizar la tabla en un plazo máximo de 2 segundos tras la confirmación.
11. WHEN el Usuario_Autenticado selecciona la acción "Suspender" sobre un registro de Cliente con Estado_Cliente "Activo", THE Módulo_Clientes SHALL cambiar el Estado_Cliente a "Suspendido" y reflejar el cambio visualmente en la tabla en un plazo máximo de 2 segundos.
12. THE Módulo_Clientes SHALL aplicar un indicador visual diferenciado (por ejemplo, color o ícono) en la tabla para distinguir los registros con Estado_Cliente "Activo" de los registros con Estado_Cliente "Suspendido".
13. IF el Usuario_Autenticado intenta registrar o editar un Cliente con una Cédula que contenga caracteres no numéricos o más de 10 dígitos, THEN THE Módulo_Clientes SHALL mostrar un mensaje de validación indicando que la Cédula debe contener únicamente entre 1 y 10 dígitos numéricos.

---

### Requerimiento 4: Gestión de Carros

**User Story:** Como Usuario_Autenticado, quiero registrar, editar, buscar y eliminar vehículos vinculados a un cliente propietario, para mantener un inventario actualizado de los carros con su respectivo dueño y poder consultar todos los vehículos de un cliente específico.

#### Criterios de Aceptación

1. THE Módulo_Carros SHALL presentar un formulario con los campos: Placa (entre 5 y 7 caracteres alfanuméricos), Marca (máximo 50 caracteres), Modelo (máximo 50 caracteres) y Cédula del Propietario (cédula de un Cliente registrado en el Sistema), con Estado_Carro establecido en "Activo" por defecto, para el registro de un nuevo Carro.
2. WHEN el Usuario_Autenticado envía el formulario de registro de un Carro, THE Módulo_Carros SHALL validar que los campos Placa, Marca, Modelo y Cédula del Propietario estén completos y que la Placa tenga entre 5 y 7 caracteres alfanuméricos antes de guardar el registro.
3. IF los campos Placa, Marca, Modelo o Cédula del Propietario están vacíos, o la Placa tiene un formato inválido al enviar el formulario de registro, THEN THE Módulo_Carros SHALL mostrar un mensaje de validación especificando qué campo requiere corrección y no guardará el registro.
4. IF se intenta registrar un Carro con una Placa ya existente en el Sistema, THEN THE Módulo_Carros SHALL mostrar un mensaje de error indicando que la Placa ya está registrada y no guardará el registro duplicado.
5. IF la Cédula del Propietario ingresada en el formulario de registro de un Carro no corresponde a un Cliente registrado en el Sistema, THEN THE Módulo_Carros SHALL mostrar un mensaje de error indicando que la cédula no pertenece a ningún cliente registrado y no guardará el registro.
6. THE Módulo_Carros SHALL permitir que un mismo Cliente sea propietario de múltiples Carros (relación uno a muchos: un Cliente puede tener muchos Carros, pero cada Carro tiene exactamente un Propietario).
7. THE Módulo_Carros SHALL mostrar una tabla con los registros de vehículos existentes, incluyendo las columnas: Placa, Marca, Modelo, Propietario (nombre del cliente vinculado) y Estado_Carro.
8. WHEN el Usuario_Autenticado ingresa texto en el campo de búsqueda, THE Módulo_Carros SHALL filtrar en un plazo máximo de 500 ms los registros de la tabla mostrando únicamente los vehículos cuya Placa, Marca, Modelo o nombre del Propietario contengan el texto ingresado, sin distinción de mayúsculas y minúsculas.
9. WHEN el Usuario_Autenticado selecciona la acción "Editar" sobre un registro de Carro, THE Módulo_Carros SHALL cargar los datos del Carro en el formulario para permitir su modificación, incluyendo la Cédula del Propietario.
10. WHEN el Usuario_Autenticado confirma la edición de un Carro, THE Módulo_Carros SHALL validar que los campos Placa, Marca, Modelo y Cédula del Propietario sigan siendo válidos, actualizar el registro en el Sistema y reflejar los cambios en la tabla en un plazo máximo de 2 segundos.
11. IF durante la edición se intenta asignar una Placa que ya pertenece a otro Carro distinto, THEN THE Módulo_Carros SHALL mostrar un mensaje de error indicando el conflicto y no guardará el cambio.
12. IF durante el registro o edición se intenta asignar una Cédula del Propietario que no existe en el repositorio de Clientes, THEN THE Módulo_Carros SHALL mostrar un mensaje de error indicando que el cliente no existe y no guardará el cambio.
13. WHEN el Usuario_Autenticado selecciona la acción "Eliminar" sobre un registro de Carro y confirma la eliminación, THE Módulo_Carros SHALL eliminar el registro permanentemente y actualizar la tabla en un plazo máximo de 2 segundos.
14. WHEN el Usuario_Autenticado selecciona la acción "Eliminar" sobre un registro de Carro, THE Módulo_Carros SHALL solicitar confirmación antes de proceder con la eliminación permanente del registro.
15. WHEN el Usuario_Autenticado selecciona la acción "Suspender" sobre un Carro con Estado_Carro "Activo", THE Módulo_Carros SHALL cambiar el Estado_Carro a "Inactivo" y reflejar el cambio visualmente en la tabla en un plazo máximo de 2 segundos.
16. THE Módulo_Carros SHALL aplicar un indicador visual diferenciado en la tabla para distinguir los registros con Estado_Carro "Activo" de los registros con Estado_Carro "Inactivo".
17. THE Módulo_Clientes SHALL ofrecer, en la vista de detalle de un Cliente, una sección que muestre todos los Carros vinculados a dicho Cliente, permitiendo al Usuario_Autenticado consultar los vehículos de un propietario específico sin necesidad de buscarlos individualmente en el Módulo_Carros.

---

### Requerimiento 5: Registro de Servicios

**User Story:** Como Usuario_Autenticado, quiero registrar los servicios prestados a cada vehículo asociando la placa y la cédula del cliente, para llevar un control detallado de los trabajos realizados.

#### Criterios de Aceptación

1. THE Módulo_Servicios SHALL presentar un formulario con los campos: Placa del Carro, Cédula del Cliente, Fecha del Servicio, Tipo_Servicio y Notas (máximo 500 caracteres) para el registro de un nuevo Servicio.
2. THE Módulo_Servicios SHALL ofrecer un selector de fecha para el campo "Fecha del Servicio" que permita únicamente fechas de calendario reales iguales o anteriores a la fecha actual, rechazando fechas futuras y formatos no reconocibles.
3. THE Módulo_Servicios SHALL ofrecer una lista desplegable para el campo Tipo_Servicio con exactamente las opciones: Mantenimiento, Cambio de Aceite, Frenos, Revisión General y Otro.
4. WHEN el Usuario_Autenticado envía el formulario de registro de un Servicio con todos los campos obligatorios completos y válidos, THE Módulo_Servicios SHALL guardar el registro de Servicio en el Sistema.
5. IF los campos Placa, Cédula, Fecha o Tipo_Servicio están vacíos o tienen formato inválido al enviar el formulario, THEN THE Módulo_Servicios SHALL mostrar un mensaje de validación especificando qué campos requieren corrección y no guardará el registro.
6. IF la Placa ingresada en el formulario de Servicio no corresponde a un Carro registrado en el Sistema, THEN THE Módulo_Servicios SHALL mostrar un mensaje de error indicando que la Placa no existe y no guardará el registro.
7. IF la Cédula ingresada en el formulario de Servicio no corresponde a un Cliente registrado en el Sistema, THEN THE Módulo_Servicios SHALL mostrar un mensaje de error indicando que la Cédula no existe y no guardará el registro.
8. WHEN el registro de Servicio es guardado exitosamente, THE Módulo_Servicios SHALL mostrar un mensaje de confirmación al Usuario_Autenticado y restablecer todos los campos del formulario a su estado vacío o por defecto para permitir un nuevo ingreso.

---

### Requerimiento 6: Consulta Histórica de Servicios por Vehículo

**User Story:** Como Usuario_Autenticado, quiero buscar todos los servicios realizados a un vehículo específico por su placa, para consultar su historial de mantenimiento de forma rápida.

#### Criterios de Aceptación

1. THE Módulo_Servicios SHALL presentar una sección de consulta histórica con un campo de texto para ingresar la Placa y un botón "Buscar".
2. WHEN el Usuario_Autenticado hace clic en "Buscar" con una Placa registrada que tiene al menos un Servicio asociado, THE Módulo_Servicios SHALL mostrar en una tabla todos los Servicios asociados a esa Placa, incluyendo las columnas: ID Servicio, Fecha del Servicio, Tipo_Servicio y Notas.
3. WHEN el Usuario_Autenticado hace clic en "Buscar" con una Placa registrada que no tiene Servicios registrados, THE Módulo_Servicios SHALL mostrar un mensaje indicando que no se encontraron servicios para el vehículo consultado.
4. IF el campo de Placa en la sección de consulta está vacío al hacer clic en "Buscar", THEN THE Módulo_Servicios SHALL mostrar un mensaje de validación indicando que debe ingresar una Placa para realizar la búsqueda.
5. IF el campo de Placa contiene un valor que no corresponde a ningún Carro registrado en el Sistema al hacer clic en "Buscar", THEN THE Módulo_Servicios SHALL mostrar un mensaje de error indicando que la Placa no existe en el Sistema.
6. THE Módulo_Servicios SHALL ordenar los resultados de la tabla de historial por Fecha del Servicio en orden descendente, mostrando el servicio más reciente primero.

---

### Requerimiento 7: Módulo de Ayuda

**User Story:** Como Usuario_Autenticado, quiero acceder a una pantalla de ayuda con instrucciones de uso del sistema, para poder resolver dudas operativas sin necesidad de asistencia externa.

#### Criterios de Aceptación

1. WHEN el Usuario_Autenticado selecciona "Ayuda" en el menú de navegación, THE Módulo_Ayuda SHALL mostrar una pantalla que contenga al menos una sección de instrucciones por cada módulo disponible: Clientes, Carros y Servicios.
2. THE Módulo_Ayuda SHALL incluir, en cada sección, un mínimo de una instrucción operativa que describa cómo realizar la operación principal del módulo correspondiente (registrar, editar o eliminar un registro).
3. THE Módulo_Ayuda SHALL organizar el contenido en secciones visualmente separadas, cada una identificada con el nombre del módulo al que pertenece, con un máximo de 3 niveles de jerarquía de contenido por sección.
4. IF el Usuario_Autenticado no tiene conexión a recursos externos al momento de acceder a "Ayuda", THE Módulo_Ayuda SHALL mostrar el contenido de ayuda disponible localmente sin degradación de contenido.

---

### Requerimiento 8: Cierre de Sesión

**User Story:** Como Usuario_Autenticado, quiero cerrar sesión de forma segura con confirmación, para evitar que otras personas accedan al sistema después de terminar mi turno.

#### Criterios de Aceptación

1. WHEN el Usuario_Autenticado selecciona "Cerrar Sesión" en el menú de navegación, THE Módulo_Autenticación SHALL mostrar un diálogo de confirmación que contenga una pregunta explícita de confirmación y dos opciones: confirmar y cancelar.
2. WHEN el Usuario_Autenticado confirma el cierre de sesión en el diálogo, THE Módulo_Autenticación SHALL invalidar el estado de sesión activo en un plazo máximo de 2 segundos y redirigir al usuario a la pantalla de inicio de sesión.
3. WHEN el Usuario_Autenticado cancela el cierre de sesión en el diálogo, THE Módulo_Autenticación SHALL cerrar el diálogo y mantener la sesión activa sin cambios en el estado de autenticación ni en los datos en pantalla.
4. WHILE la sesión está cerrada, THE Sistema SHALL impedir el acceso a cualquier módulo distinto al Módulo_Autenticación y redirigir cualquier intento de navegación a la pantalla de inicio de sesión.
5. IF el Usuario_Autenticado intenta navegar a un módulo protegido con la sesión cerrada, THEN THE Sistema SHALL redirigir al Usuario_Autenticado a la pantalla de inicio de sesión sin mostrar datos del módulo solicitado.

---

### Requerimiento 9: Persistencia de Datos

**User Story:** Como Usuario_Autenticado, quiero que los datos registrados en el sistema persistan entre sesiones, para que la información no se pierda al cerrar la aplicación.

#### Criterios de Aceptación

1. THE Sistema SHALL persistir todos los registros de Clientes, Carros y Servicios en el mecanismo de almacenamiento local inmediatamente después de cada operación de creación, edición o eliminación, de modo que los datos sean recuperables tras un cierre normal o inesperado de la aplicación.
2. WHEN el Sistema inicia, THE Sistema SHALL cargar la totalidad de los registros almacenados y presentarlos en las tablas correspondientes de cada módulo en un plazo máximo de 5 segundos, independientemente del número de registros almacenados hasta un máximo de 10.000 registros por entidad.
3. IF ocurre un error al intentar guardar un registro en el mecanismo de almacenamiento, THEN THE Sistema SHALL mostrar un mensaje de error al Usuario_Autenticado indicando que el registro no pudo ser guardado y SHALL descartar únicamente el registro fallido, preservando el resto de los datos almacenados sin modificación.
4. IF ocurre un error al cargar los datos almacenados durante el inicio del Sistema, THEN THE Sistema SHALL mostrar un mensaje de error al Usuario_Autenticado indicando que los datos no pudieron ser cargados y SHALL permitir el acceso al Sistema mostrando las tablas vacías.

---

### Requerimiento 10: Modo de Visualizacion (Claro / Oscuro) y Paleta de Color

**User Story:** Como usuario del sistema, quiero poder cambiar entre modo oscuro y modo claro, para adaptar la interfaz a mis preferencias visuales o condiciones de iluminacion, con una paleta de colores activa y llamativa.

#### Criterios de Aceptacion

1. THE Sistema SHALL proveer un boton de cambio de tema visible en la pantalla de inicio (Landing), en la pantalla de inicio de sesion (Login) y en la barra superior de la aplicacion (topbar), que permita alternar entre modo oscuro y modo claro.
2. WHEN el Usuario_Autenticado activa el modo claro, THE Sistema SHALL cambiar inmediatamente la apariencia de todos los elementos de la interfaz al esquema de color claro sin recargar la pagina.
3. WHEN el Usuario_Autenticado activa el modo oscuro, THE Sistema SHALL cambiar inmediatamente la apariencia de todos los elementos de la interfaz al esquema de color oscuro sin recargar la pagina.
4. THE Sistema SHALL persistir la preferencia de tema seleccionada entre sesiones, de modo que al recargar la aplicacion se muestre el ultimo tema elegido por el usuario.
5. THE Sistema SHALL usar la paleta de colores definida: fondo oscuro #121820, color de accion principal #FF6B00 (naranja), y texto principal #FFFFFF en modo oscuro / #121820 en modo claro.
6. IF el usuario no ha seleccionado ningun tema previamente, THE Sistema SHALL mostrar el modo oscuro por defecto.
7. THE Sistema SHALL garantizar que el contraste entre el color de texto principal y el fondo de cada tema cumpla con el nivel AA de las WCAG 2.1 (relacion de contraste minima 4.5:1 para texto de cuerpo).
