## Sistema de gestión de eventos y participación

### Introducción

El objetivo de esta práctica es desarrollar un sistema web completo utilizando Django. La aplicación permitirá gestionar eventos, participantes y comentarios, con diferentes funcionalidades según el rol de los usuarios (organizadores y participantes). El proyecto se abordará en cuatro etapas principales, cada una enfocada en un aspecto clave del desarrollo web en entorno servidor.

---

### Requisitos del Sistema

#### **1. Usuarios Personalizados**

El sistema debe gestionar usuarios que pueden tener uno de dos roles: organizadores y participantes. Cada usuario tiene información básica como su nombre, correo electrónico y contraseña. Además, pueden incluir una biografía opcional para describirse.

- Los organizadores son responsables de crear y gestionar eventos.
- Los participantes pueden inscribirse en eventos y publicar comentarios.

#### **2. Eventos**

El sistema debe permitir gestionar eventos organizados por los usuarios. Los eventos incluyen información como:

- Un título que los identifique.
- Una descripción detallada.
- Fecha y hora del evento.
- Un límite de capacidad máxima de asistentes.
- Una url con una imagen relacionada con el evento

Cada evento pertenece a un único organizador, quien tiene control total sobre su gestión.

#### **3. Reservas**

El sistema debe permitir a los participantes realizar reservas para eventos. Una reserva contiene:

- Información del usuario que realiza la reserva.
- El evento asociado.
- El número de entradas reservadas.
- El estado de la reserva, que puede ser "pendiente", "confirmada" o "cancelada".

Las reservas están limitadas por la capacidad máxima del evento. Solo los organizadores pueden cambiar el estado de una reserva.

#### **4. Comentarios**

Los usuarios pueden dejar comentarios en los eventos. Cada comentario incluye:

- Texto escrito por el usuario.
- La información del evento al que pertenece.
- El usuario que lo publicó.
- La fecha en que fue creado.

Cada comentario está asociado tanto a un usuario como a un evento.

---

### Desglose por Etapas

#### **Etapa 1: Creación de Modelos y Relaciones**

1. Diseñar los modelos a partir de los requisitos del sistema.
2. Implementar las relaciones entre los modelos usando `ForeignKey` y `ManyToManyField` donde sea necesario.
3. Crear un esquema visual de la base de datos utilizando herramientas como **dbdiagram.io** o cualquier otra.
4. Configurar el panel de administración de Django para gestionar los modelos.

#### **Etapa 2: Implementación de Endpoints Avanzados**

1. **CRUD de eventos:**
   - GET: Listar todos los eventos disponibles (filtros opcionales por título o fecha).
   - POST: Crear un evento (solo organizadores).
   - PUT/PATCH: Actualizar un evento (solo organizadores).
   - DELETE: Eliminar un evento (solo organizadores).
2. **Gestín de reservas:**
   - GET: Listar reservas de un usuario autenticado.
   - POST: Crear una nueva reserva.
   - PUT/PATCH: Actualizar el estado de una reserva (solo organizadores).
   - DELETE: Cancelar una reserva (solo participantes para sus reservas).
3. **Comentarios:**
   - GET: Listar comentarios de un evento.
   - POST: Crear un comentario asociado a un evento (solo usuarios autenticados).
4. **Optimización de consultas:**
   - Usar `select_related` y `prefetch_related` para mejorar el rendimiento de las vistas.

#### **Etapa 3: Autenticación, Seguridad y Documentación**

1. Implementar autenticación por tokens con **TokenAuthentication**.
2. Configurar permisos según roles de usuario:
   - Los organizadores pueden gestionar eventos y reservas.
   - Los participantes pueden listar eventos y crear/cancelar sus propias reservas.
3. Documentar la API usando **Swagger** (drf-yasg).

#### **Etapa 4: Plantillas y Vistas Dinámicas**

1. Crear plantillas para:
   - Página de inicio: Mostrar la lista de eventos disponibles.
   - Detalle del evento: Mostrar información completa y formulario para reservas.
   - Panel de usuario: Mostrar las reservas del usuario autenticado.
2. Implementar herencia de plantillas con una estructura base.
3. Integrar CSS para mejorar el diseño y agregar animaciones ligeras (hover y transiciones).

---

### Entregables

1. **Etapa 1:**

   - Código del proyecto en gitHub haciendo commits cuando toque y en Classroom.
   - Esquema visual de la base de datos.
   - Capturas del panel de administración con registros de prueba.

2. **Etapa 2:**

   - Código del proyecto en gitHub haciendo commits cuando toque y en Classroom.
   - Capturas de las pruebas en postman en classroom

3. **Etapa 3:**

   - Código del proyecto en gitHub haciendo commits cuando toque y en Classroom.

4. **Etapa 4:**

   - Código del proyecto en gitHub haciendo commits cuando toque y en Classroom.
   - Capturas de las plantillas funcionando con datos dinámicos en Classroom

---

### Evaluación

1. **Modelos y Relaciones:** 20%
2. **Endpoints y Consultas:** 40%
3. **Autenticación y Permisos:** 20%
4. **Diseño y Plantillas:** 20%

---

