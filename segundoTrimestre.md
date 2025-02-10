# 1: Creación del modelo y decoradores en Django

## Objetivo

Esta guía proporcionará la teoría y ejemplos necesarios para trabajar con modelos y decoradores en Django. Aprenderás a crear modelos desde `models.py` y a utilizar decoradores para mejorar la funcionalidad y seguridad de las vistas en Django.

---

## Parte 1: Modelos en Django

### ¿Qué es un modelo?

En Django, un modelo es una representación de una tabla de base de datos en forma de clase Python. Cada clase define la estructura de una tabla, incluidos los campos y las relaciones entre ellos.

Un modelo no solo define cómo se estructura la tabla en la base de datos, sino también cómo interactuamos con los datos de esa tabla en el código. Esto incluye consultas, inserciones, actualizaciones y eliminaciones.

### Crear un modelo

Un modelo se define en el archivo `models.py` de la aplicación Django.

#### **Ejemplo básico**

```python
from django.db import models

class Producto(models.Model):
    nombre = models.CharField(max_length=100)  # Campo de texto
    precio = models.DecimalField(max_digits=10, decimal_places=2)  # Campo decimal
    stock = models.IntegerField()  # Campo de enteros
    descripcion = models.TextField(blank=True, null=True)  # Campo de texto largo

    def __str__(self):
        return self.nombre
```

#### **Explicación de los campos**

- `CharField`: Almacena cadenas de texto. Se define con un límite de longitud.
- `DecimalField`: Almacena números decimales, ideal para valores monetarios.
- `IntegerField`: Almacena números enteros.
- `TextField`: Similar a `CharField`, pero sin límite de longitud.
- `choices`: Permite definir un conjunto de opciones predefinidas para un campo.
- `default`: Define el valor predeterminado para un campo si no se especifica otro valor.
- `blank=True`: Permite que el campo esté vacío en los formularios.
- `null=True`: Permite que el campo acepte valores nulos en la base de datos.

### Uso de `choices` en modelos

Django permite agregar campos con opciones predefinidas utilizando el atributo `choices`. Esto es útil para limitar los valores posibles de un campo.

**Ejemplo práctico:**

```python
class UsuarioPersonalizado(AbstractUser):
    TIPO_USUARIO = [
        ('organizador', 'Organizador'),
        ('asistente', 'Asistente')
    ]
    tipo = models.CharField(max_length=20, choices=TIPO_USUARIO, default='asistente')
```

#### **Explicación:**

- `TIPO_USUARIO`: Una lista de tuplas que define las opciones disponibles para el campo `tipo`.
- `choices=TIPO_USUARIO`: Restringe los valores del campo `tipo` a las opciones definidas en la lista `TIPO_USUARIO`.
- `default='asistente'`: Si no se proporciona un valor para el campo `tipo`, el valor predeterminado será `'asistente'`.

#### **Acceso y uso del campo con `choices`:**

1. **Creación de un usuario:**

```python
usuario = UsuarioPersonalizado.objects.create_user(username='juan', password='12345', tipo='organizador')
```

2. **Consulta del tipo de usuario:**

```python
usuario = UsuarioPersonalizado.objects.get(username='juan')
print(usuario.tipo)  # Salida: organizador
```

3. **Uso en formularios:**
En los formularios de Django, el campo `tipo` se renderizará automáticamente como un desplegable con las opciones definidas en `choices`.

---


### Migraciones

Django utiliza un sistema de migraciones para sincronizar los modelos con la base de datos.

1. **Crear la migración:** Este comando genera un archivo de migración basado en los cambios en los modelos.

   ```bash
   python manage.py makemigrations
   ```

2. **Aplicar la migración:** Este comando ejecuta las migraciones y sincroniza los cambios con la base de datos.

   ```bash
   python manage.py migrate
   ```

### Relaciones entre modelos

Los modelos en Django soportan relaciones para definir cómo interactúan las tablas entre sí.

#### **Tipos de relaciones**

- **Uno a muchos (`ForeignKey`)**: Un registro en una tabla está relacionado con muchos registros en otra.
- **Muchos a muchos (`ManyToManyField`)**: Muchos registros de una tabla están relacionados con muchos registros de otra tabla.
- **Uno a uno (`OneToOneField`)**: Un registro en una tabla está relacionado con exactamente un registro en otra.

#### **Ejemplo de relaciones**

```python
class Categoria(models.Model):
    nombre = models.CharField(max_length=100)

    def __str__(self):
        return self.nombre

class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE)

    def __str__(self):
        return self.nombre
```

#### **Explicación**

- `on_delete=models.CASCADE`: Al eliminar una categoría, también se eliminan todos los productos asociados.

### Métodos útiles en los modelos

Los modelos en Django permiten añadir métodos personalizados para extender su funcionalidad. Por ejemplo:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    precio = models.DecimalField(max_digits=10, decimal_places=2)

    def aplicar_descuento(self, porcentaje):
        return self.precio * (1 - porcentaje / 100)
```

Este método calcula el precio del producto después de aplicar un descuento.

---

### Modelos predefinidos en Django

Django incluye algunos modelos predefinidos en sus aplicaciones integradas. Estos modelos están diseñados para facilitar la implementación de funcionalidades comunes como la gestión de usuarios, sesiones y permisos. Uno de los modelos más importantes es el modelo de **Usuario**.

#### **El modelo Usuario (`django.contrib.auth.models.User`)**
Este modelo es parte de la aplicación de autenticación de Django y ofrece una estructura predeterminada para manejar usuarios.

**Campos principales del modelo Usuario:**
- `username`: Nombre de usuario único.
- `email`: Correo electrónico del usuario.
- `password`: Contraseña cifrada.
- `is_staff`: Indica si el usuario puede acceder al sitio de administración.
- `is_superuser`: Indica si el usuario tiene todos los permisos.
- `date_joined`: Fecha de creación del usuario.
- `is_active`: Determina si el usuario está activo.

### Extender el modelo Usuario con campos adicionales
Si necesitas agregar más campos al modelo Usuario estándar, puedes personalizarlo utilizando un modelo basado en `AbstractUser`. Este enfoque permite añadir nuevos campos manteniendo la funcionalidad predeterminada de Django.

#### **Configuración necesaria para usar un modelo extendido:**

1. **Definir el modelo en `models.py`:**
   ```python
   from django.contrib.auth.models import AbstractUser
   from django.db import models

   class UsuarioPersonalizado(AbstractUser):
       telefono = models.CharField(max_length=15, blank=True)
       direccion = models.TextField(blank=True)
   ```

2. **Actualizar la configuración del proyecto en `settings.py`:**
   ```python
   AUTH_USER_MODEL = 'tuapp.UsuarioPersonalizado'
   ```

3. **Crear y aplicar las migraciones:**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

#### **Crear y usar usuarios extendidos:**
1. **Crear un usuario:**
   ```python
   from django.contrib.auth import get_user_model

   Usuario = get_user_model()
   usuario = Usuario.objects.create_user(username='maria', password='12345', telefono='123456789')
   usuario.direccion = 'Calle Falsa 123'
   usuario.save()
   ```

2. **Consultar un usuario:**
   ```python
   usuario = Usuario.objects.get(username='maria')
   print(usuario.telefono)  # Salida: 123456789
   ```

### **Ventajas de extender el modelo Usuario:**
- **Escalabilidad:** Permite personalizar el modelo según los requisitos del proyecto.
- **Integración nativa:** Mantiene la compatibilidad con las herramientas y funcionalidades predeterminadas de Django.
- **Simplificación:** Facilita el manejo de datos de usuarios sin necesidad de modelos adicionales.

### Otros modelos predefinidos en Django
Django incluye algunos modelos predefinidos en sus aplicaciones integradas. Estos modelos están diseñados para facilitar la implementación de funcionalidades comunes como la gestión de usuarios, sesiones y permisos. Uno de los modelos más importantes es el modelo de **Usuario**.

#### **Lista de modelos predefinidos:**
1. **Group (`django.contrib.auth.models.Group`)**:
   Representa un grupo de usuarios, útil para gestionar permisos de forma colectiva.

2. **Permission (`django.contrib.auth.models.Permission`)**:
   Define permisos individuales que pueden ser asignados a usuarios o grupos.

3. **Session (`django.contrib.sessions.models.Session`)**:
   Almacena datos relacionados con sesiones de usuario.

4. **Site (`django.contrib.sites.models.Site`)**:
   Permite gestionar múltiples dominios en un mismo proyecto Django.

5. **ContentType (`django.contrib.contenttypes.models.ContentType`)**:
   Proporciona una forma de referirse a modelos específicos de manera genérica.

6. **LogEntry (`django.contrib.admin.models.LogEntry`)**:
   Registra acciones realizadas en el sitio de administración de Django.

Estos modelos son fundamentales para muchas aplicaciones y pueden integrarse fácilmente en proyectos cuando se habilitan las aplicaciones correspondientes en `INSTALLED_APPS` del archivo `settings.py`.

---
### Configurar el Administrador de Django

#### 1. Registrar el modelo en `admin.py`

Para que un modelo esté disponible en el panel de administración, primero deben registrarlo.

1. Abre el archivo `admin.py` de la aplicación.
2. Importa el modelo que quieres administrar.
3. Regístralo con el administrador de Django.

**Ejemplo:**
```python
from django.contrib import admin
from .models import TuModelo

admin.site.register(TuModelo)
```

#### 2. Crear un Superusuario

El administrador de Django requiere un usuario con permisos de superusuario para iniciar sesión.

1. Ejecuta este comando en el terminal:
   ```bash
   python manage.py createsuperuser
   ```
2. Proporciona los datos requeridos:
   - **Nombre de usuario:** elige un nombre (por ejemplo, "admin").
   - **Dirección de correo electrónico:** introduce un correo válido.
   - **Contraseña:** escribe y confirma la contraseña.
3. Una vez creado, verás un mensaje de confirmación.

#### 3. Ejecutar el Servidor de Desarrollo

Inicia el servidor de Django para acceder al panel de administración:
```bash
python manage.py runserver
```
Accede al administrador en la URL:
```
http://127.0.0.1:8000/admin/
```

#### 4. Acceder al Panel de Administración

1. Abre la URL en un navegador web.
2. Inicia sesión con las credenciales del superusuario que acabas de crear.

#### 5. Agregar Datos desde el Administrador

Una vez dentro del administrador:

1. **Ubica el modelo correspondiente:**
   Los modelos registrados en `admin.py` aparecerán organizados por aplicaciones. Por ejemplo, si tienes un modelo llamado Producto dentro de una aplicación llamada tienda, debería verse algo así:
   ```
   Tienda
   - Productos
   ```
2. **Haz clic en el modelo**, por ejemplo, Productos.
3. **Selecciona "Add Producto" (o equivalente):**
   - Cada campo del modelo aparecerá como un campo editable en el formulario. Ejemplo:
     - Nombre: Escribe "Laptop".
     - Descripción: Añade un detalle, como "Ordenador portátil".
     - Precio: Introduce un valor, por ejemplo, "999.99".
   - Haz clic en el botón "Save" (o "Guardar").

#### 6. Ver y Editar Datos Existentes

Una vez que se hayan agregado datos, volverás a la lista del modelo. Aquí podrás:
- **Ver** una tabla con las instancias del modelo.
- **Editar** datos existentes: haz clic en cualquier registro.
- **Eliminar** datos: selecciona uno o más registros y elige "Delete selected items" en el menú desplegable.

#### 7. Personalización Opcional del Administrador

Si deseas mejorar la visualización en el administrador:

Modifica el archivo `admin.py` para personalizar cómo se muestran los datos. Por ejemplo:
```python
from django.contrib import admin
from .models import Producto

class ProductoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'precio', 'stock')  # Muestra estas columnas en la tabla
    search_fields = ('nombre',)  # Añade barra de búsqueda por nombre
    list_filter = ('categoria',)  # Añade filtros por categoría

admin.site.register(Producto, ProductoAdmin)
```

---
## Parte 2: Decoradores en Django

### ¿Qué es un decorador?

Un decorador es una función que modifica el comportamiento de otra función o método sin alterar directamente su código fuente. En Django, los decoradores son una herramienta poderosa para manejar la autenticación, permisos, restricciones de métodos HTTP y más.

### Decoradores comunes en Django

#### **1. `@login_required`**

Restringe el acceso a una vista para usuarios autenticados. Solo los usuarios que han iniciado sesión pueden acceder a la vista decorada con este decorador.

#### **¿Cuándo usarlo?**

- Para proteger áreas sensibles o funcionalidad destinada solo a usuarios registrados, como perfiles o configuraciones personales.

#### **Ejemplo:**

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render

@login_required
def perfil_usuario(request):
    return render(request, 'perfil.html')
```

**¿Cómo funciona?**

- Si un usuario no está autenticado, será redirigido automáticamente a la página de inicio de sesión definida en la configuración del proyecto (`LOGIN_URL`).

---

#### **2. `@csrf_exempt`**

Excluye una vista de la protección contra ataques CSRF (Cross-Site Request Forgery). Esto puede ser útil para integraciones con servicios externos que no envían el token CSRF.

### **¿Qué es un ataque CSRF?**

Un ataque CSRF ocurre cuando un atacante engaña a un usuario autenticado para que realice acciones no deseadas en una aplicación en la que el usuario está autenticado. Por ejemplo:

1. El usuario inicia sesión en su cuenta de banca en línea.
2. Mientras navega, visita un sitio web malicioso que envía una solicitud POST a la banca en línea para transferir dinero a la cuenta del atacante.
3. Debido a que la sesión del usuario está activa, el servidor confía en la solicitud y realiza la transferencia.

### **Prevención de ataques CSRF en Django**

Django incluye protección automática contra CSRF mediante un middleware que se encuentra habilitado por defecto. Este middleware valida que las solicitudes POST, PUT, DELETE o PATCH incluyan un token CSRF válido.

#### **Cómo funciona el middleware CSRF**

1. **Configuración en el servidor:**
   Asegúrate de que el middleware CSRF está habilitado en el archivo `settings.py`:
   ```python
   MIDDLEWARE = [
       ...
       'django.middleware.csrf.CsrfViewMiddleware',
       ...
   ]
   ```

2. **Uso en formularios HTML:**
   Al crear formularios, utiliza la etiqueta `{% csrf_token %}` para incluir automáticamente el token CSRF:
   ```html
   <form method="post">
       {% csrf_token %}
       <input type="text" name="nombre" placeholder="Nombre">
       <button type="submit">Enviar</button>
   </form>
   ```

3. **Respuestas del servidor:**
   Si el token CSRF no está presente o no es válido, el middleware devuelve un error `403 Forbidden`. Esto protege la aplicación de solicitudes maliciosas.

4. **Excepciones para APIs públicas:**
   Si necesitas deshabilitar la protección CSRF en una vista específica (como en APIs REST), usa el decorador `@csrf_exempt`:
   ```python
   from django.views.decorators.csrf import csrf_exempt
   from django.http import JsonResponse

   @csrf_exempt
   def api_publica(request):
       return JsonResponse({"mensaje": "Acceso sin CSRF"})
   ```

**Precaución:** Deshabilitar la protección CSRF debe hacerse con cuidado, ya que puede exponer tu aplicación a vulnerabilidades.

---

#### **3. `@require_http_methods`**

Restringe los métodos HTTP que una vista puede manejar. Esto mejora la seguridad y evita que las vistas manejen solicitudes no deseadas.

#### **¿Cuándo usarlo?**

- Para limitar vistas a métodos específicos, como solo permitir `GET` y `POST` en formularios.

#### **Ejemplo:**

```python
from django.views.decorators.http import require_http_methods
from django.http import JsonResponse

@require_http_methods(["GET", "POST"])
def gestion_productos(request):
    if request.method == "GET":
        return JsonResponse({"mensaje": "Consulta de productos"})
    elif request.method == "POST":
        return JsonResponse({"mensaje": "Creación de producto"})
```

**Beneficio:** Ayuda a evitar errores y comportamientos inesperados al garantizar que solo los métodos esperados sean manejados.

---

#### **4. `@transaction.atomic`**

Garantiza que todas las operaciones dentro de una vista se ejecuten como una sola transacción. Si ocurre un error en cualquier operación, se revierte toda la transacción.

#### **¿Cuándo usarlo?**

- Para operaciones complejas que involucran múltiples escrituras en la base de datos y requieren consistencia, como procesos de compra o actualizaciones masivas.

#### **Ejemplo:**

```python
from django.db import transaction

@transaction.atomic
def realizar_compra(request):
    producto.stock -= 1
    producto.save()
    pedido = Pedido(producto=producto, cliente=request.user)
    pedido.save()
```

**Ventaja:** Asegura la integridad de los datos en operaciones complejas.

---

# 2: Funciones del ORM de Django y endpoints avanzados

## Objetivo

Esta guía proporcionará la teoría y ejemplos necesarios para trabajar con las funciones avanzadas del ORM (Object-Relational Mapping) de Django y la implementación de endpoints que manejen diferentes métodos HTTP, incluyendo el uso de parámetros en URLs y consultas avanzadas. Al finalizar, los estudiantes estarán preparados para implementar una API REST funcional y eficiente.

---

## Parte 1: funciones del ORM de Django

### ¿Qué es el ORM de Django?

El ORM de Django es una herramienta que permite interactuar con bases de datos utilizando código Python en lugar de consultas SQL. Esto simplifica las operaciones de creación, lectura, actualización y eliminación (CRUD) y permite trabajar con diferentes sistemas de bases de datos sin cambiar el código.

### Consultas básicas

#### Crear registros

Para añadir un registro a la base de datos:

```python
from app.models import Producto

producto = Producto(nombre="Laptop", precio=1200.00, stock=10)
producto.save()
```

#### Leer registros

##### Obtener todos los registros:

```python
productos = Producto.objects.all()
```

##### Filtrar registros:

```python
productos_caros = Producto.objects.filter(precio__gte=1000)  # Productos con precio mayor o igual a 1000
```

##### Obtener un único registro:

```python
producto = Producto.objects.get(id=1)  # Producto con ID 1
```

**Nota:** `get` lanza un error si no encuentra ningún registro o si encuentra más de uno.

#### Actualizar registros

```python
producto = Producto.objects.get(id=1)
producto.stock = 5
producto.save()
```

#### Eliminar registros

```python
producto = Producto.objects.get(id=1)
producto.delete()
```

### Consultas avanzadas

#### Uso de Lookups

Los **lookups** son operadores que permiten realizar consultas más complejas. Algunos de los más comunes son:

- `exact`: Busca valores exactos.
  ```python
  productos = Producto.objects.filter(nombre__exact="Laptop")
  ```
- `iexact`: Similar a `exact`, pero no distingue entre mayúsculas y minúsculas.
  ```python
  productos = Producto.objects.filter(nombre__iexact="laptop")
  ```
- `contains`: Busca valores que contengan una subcadena.
  ```python
  productos = Producto.objects.filter(nombre__contains="top")
  ```
- `icontains`: Similar a `contains`, pero sin distinguir mayúsculas y minúsculas.
  ```python
  productos = Producto.objects.filter(nombre__icontains="top")
  ```
- `gte`/`lte`: Mayor o igual (`gte`), menor o igual (`lte`).
  ```python
  productos = Producto.objects.filter(precio__gte=1000)  # Precio mayor o igual a 1000
  ```
- `startswith`/`endswith`: Busca cadenas que empiecen o terminen con una subcadena específica.
  ```python
  productos = Producto.objects.filter(nombre__startswith="L")
  ```

#### Uso de `Q` y `F`

##### `Q`: Combinar condiciones dinámicas

```python
from django.db.models import Q

productos = Producto.objects.filter(Q(precio__gte=1000) | Q(stock__lt=5))
```

##### `F`: Comparar campos

```python
from django.db.models import F

productos = Producto.objects.filter(precio__gt=F('stock'))  # Productos donde el precio es mayor que el stock
```

#### Consultas agregadas

Django permite realizar cálculos como sumas, promedios, máximos y mínimos.

```python
from django.db.models import Avg, Sum, Max, Min

promedio_precio = Producto.objects.aggregate(Avg('precio'))
suma_stock = Producto.objects.aggregate(Sum('stock'))
```

#### Consultas relacionadas

Optimiza el acceso a datos relacionados:

- `select_related`: Para relaciones `ForeignKey`.
- `prefetch_related`: Para relaciones `ManyToMany` o consultas más complejas.

#### Uso y ejemplos detallados

**`select_related`:**

Este método realiza un JOIN en la base de datos para traer datos relacionados en una única consulta, optimizando el rendimiento.

```python
productos = Producto.objects.select_related('categoria').all()
for producto in productos:
    print(producto.categoria.nombre)  # Evita consultas adicionales por cada producto
```

**`prefetch_related`:**

Este método es ideal para relaciones `ManyToMany` o cuando se necesitan consultas separadas pero optimizadas para evitar el `N+1 problem`.

```python
categorias = Categoria.objects.prefetch_related('producto_set').all()
for categoria in categorias:
    productos = categoria.producto_set.all()
    for producto in productos:
        print(producto.nombre)
```

---

## Parte 2: endpoints avanzados

### ¿Qué es un endpoint?

Un endpoint es una URL específica que una aplicación expone para que los clientes puedan interactuar con el servidor a través de solicitudes HTTP (GET, POST, PUT, PATCH, DELETE).

### Métodos HTTP

#### **GET**

Se utiliza para recuperar datos.

```python
from django.http import JsonResponse
from app.models import Producto

def listar_productos(request):
    productos = Producto.objects.all()
    data = [{"id": p.id, "nombre": p.nombre, "precio": p.precio} for p in productos]
    return JsonResponse(data, safe=False)
```

#### **POST**

Se utiliza para crear nuevos recursos.

```python
from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse
from app.models import Producto
import json

@csrf_exempt
def crear_producto(request):
    if request.method == "POST":
        data = json.loads(request.body)
        producto = Producto.objects.create(
            nombre=data["nombre"],
            precio=data["precio"],
            stock=data["stock"]
        )
        return JsonResponse({"id": producto.id, "mensaje": "Producto creado exitosamente"})
```

#### **PUT y PATCH**

Se utilizan para actualizar recursos.

- **PUT**: Actualización completa.
- **PATCH**: Actualización parcial.

```python
@csrf_exempt
def actualizar_producto(request, id):
    if request.method in ["PUT", "PATCH"]:
        data = json.loads(request.body)
        producto = Producto.objects.get(id=id)
        producto.nombre = data.get("nombre", producto.nombre)
        producto.precio = data.get("precio", producto.precio)
        producto.stock = data.get("stock", producto.stock)
        producto.save()
        return JsonResponse({"mensaje": "Producto actualizado"})
```

#### **DELETE**

Se utiliza para eliminar recursos.

```python
@csrf_exempt
def eliminar_producto(request, id):
    if request.method == "DELETE":
        producto = Producto.objects.get(id=id)
        producto.delete()
        return JsonResponse({"mensaje": "Producto eliminado"})
```

### Parámetros en URLs y consultas

#### Parámetros en URLs

Django permite capturar parámetros desde las URLs.

##### Definición en `urls.py`:

```python
from django.urls import path
from app import views

urlpatterns = [
    path('productos/<int:id>/', views.obtener_producto),
]
```

##### Uso en vistas:

```python
def obtener_producto(request, id):
    producto = Producto.objects.get(id=id)
    data = {"id": producto.id, "nombre": producto.nombre, "precio": producto.precio}
    return JsonResponse(data)
```

#### Parámetros de query

Los parámetros de query se envían a través de la URL usando `?key=value`.

##### Parámetros comunes:

- **Filtrar por campos específicos:**

  ```python
  def buscar_productos(request):
      nombre = request.GET.get("nombre", "")
      productos = Producto.objects.filter(nombre__icontains=nombre)
      data = [{"id": p.id, "nombre": p.nombre, "precio": p.precio} for p in productos]
      return JsonResponse(data, safe=False)
  ```

- **Ordenar resultados:**

El parámetro `orden` se pasa como un string con el nombre del campo. Para ordenar de manera descendente, se añade un guion antes del campo.

```python
def listar_productos_ordenados(request):
    orden = request.GET.get("orden", "nombre")  # Ejemplo: ?orden=-precio
    productos = Producto.objects.all().order_by(orden)
    data = [{"id": p.id, "nombre": p.nombre, "precio": p.precio} for p in productos]
    return JsonResponse(data, safe=False)
```

Ejemplos de URLs:
- `?orden=nombre`: Ordena por nombre en orden ascendente.
- `?orden=-precio`: Ordena por precio en orden descendente.

- **Limitar resultados:**

```python
def listar_productos_limitados(request):
    limite = int(request.GET.get("limite", 10))  # Ejemplo: ?limite=5
    productos = Producto.objects.all()[:limite]
    data = [{"id": p.id, "nombre": p.nombre, "precio": p.precio} for p in productos]
    return JsonResponse(data, safe=False)
```

- **Combinar filtros, orden y límite con paginación:**

```python
from django.core.paginator import Paginator
from django.http import JsonResponse
from .models import Producto

def buscar_y_ordenar_productos_paginados(request):
    nombre = request.GET.get("nombre", "")  # Filtrar por nombre
    orden = request.GET.get("orden", "nombre")  # Ordenar por el campo especificado
    limite = int(request.GET.get("limite", 10))  # Resultados por página
    pagina = int(request.GET.get("pagina", 1))  # Página actual

    # Filtrar y ordenar productos
    productos = Producto.objects.filter(nombre__icontains=nombre).order_by(orden)

    # Paginación
    paginator = Paginator(productos, limite)  # Dividir productos en páginas de tamaño `limite`

    try:
        productos_pagina = paginator.page(pagina)  # Obtener los productos de la página actual
    except Exception as e:
        return JsonResponse({"error": str(e)}, status=400)  # Manejar errores de paginación

    # Crear respuesta con datos paginados
    data = {
        "count": paginator.count,  # Número total de productos
        "total_pages": paginator.num_pages,  # Número total de páginas
        "current_page": pagina,  # Página actual
        "next": pagina + 1 if productos_pagina.has_next() else None,  # Página siguiente
        "previous": pagina - 1 if productos_pagina.has_previous() else None,  # Página anterior
        "results": [
            {"id": p.id, "nombre": p.nombre, "precio": p.precio} for p in productos_pagina
        ]  # Resultados actuales
    }

    return JsonResponse(data, safe=False)

```
### Parámetros esperados en la solicitud
1. **`nombre`**: Filtra productos cuyo nombre contenga el texto especificado.
2. **`orden`**: Ordena los resultados por un campo del modelo (por defecto, `nombre`).
3. **`limite`**: Especifica el número de resultados por página (por defecto, 10).
4. **`pagina`**: Especifica el número de la página que se desea consultar (por defecto, 1).

### Ejemplo de solicitud:
Supongamos que tienes el siguiente modelo de productos:

```python
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    precio = models.DecimalField(max_digits=10, decimal_places=2)
```

Haz una llamada como esta para obtener resultados paginados:

```http
GET /api/productos/?nombre=camisa&orden=precio&limite=5&pagina=2
```

### Ejemplo de respuesta:
```json
{
    "count": 23,
    "total_pages": 5,
    "current_page": 2,
    "next": 3,
    "previous": 1,
    "results": [
        {"id": 6, "nombre": "Camisa Roja", "precio": 15.99},
        {"id": 7, "nombre": "Camisa Azul", "precio": 18.99},
        {"id": 8, "nombre": "Camisa Verde", "precio": 19.99},
        {"id": 9, "nombre": "Camisa Negra", "precio": 20.99},
        {"id": 10, "nombre": "Camisa Blanca", "precio": 21.99}
    ]
}
```
---

### Parte 3: Guía para configurar y usar Postman

Postman es una herramienta útil para probar APIs REST. A continuación, se muestra cómo configurarlo y usarlo para probar los distintos endpoints creados.

#### Instalación
1. Descarga e instala Postman desde [https://www.postman.com/downloads/](https://www.postman.com/downloads/).
2. Regístrate o inicia sesión con tu cuenta.

#### Crear una colección
1. Haz clic en **Collections** y luego en **New Collection**.
2. Nombra la colección, por ejemplo: "API Productos".
3. Añade tus endpoints como solicitudes dentro de esta colección.

#### Probar métodos HTTP

##### **GET**
1. Haz clic en **New Request**.
2. Selecciona el método `GET`.
3. Introduce la URL del endpoint, por ejemplo: `http://127.0.0.1:8000/productos/`.
4. Haz clic en **Send**.
5. Verifica los datos en la pestaña **Body** de la respuesta.

##### **POST**
1. Selecciona el método `POST`.
2. Introduce la URL del endpoint: `http://127.0.0.1:8000/productos/crear/`.
3. Ve a la pestaña **Body** y selecciona **raw**. Cambia el tipo a `JSON`.
4. Introduce el cuerpo de la solicitud, por ejemplo:

```json
{
    "nombre": "Tablet",
    "precio": 300,
    "stock": 15
}
```

5. Haz clic en **Send**.
6. Verifica la respuesta en la pestaña **Body**.

##### **PUT/PATCH**
1. Selecciona `PUT` o `PATCH`.
2. Introduce la URL del endpoint, por ejemplo: `http://127.0.0.1:8000/productos/1/`.
3. En **Body**, selecciona **raw** y `JSON`. Introduce los datos a actualizar:

```json
{
    "precio": 350
}
```

4. Haz clic en **Send**.

##### **DELETE**
1. Selecciona el método `DELETE`.
2. Introduce la URL del endpoint, por ejemplo: `http://127.0.0.1:8000/productos/1/`.
3. Haz clic en **Send**.
4. Verifica que el producto se haya eliminado correctamente.

#### Parámetros de Query
1. Añade parámetros a la URL desde la pestaña **Params**.
2. Introduce las claves y valores según los filtros, orden o límites que desees probar (por ejemplo, `nombre`, `orden`, `limite`).
3. Haz clic en **Send** para ver los resultados filtrados o modificados.

# 3: Autenticación, seguridad y uso avanzado de Postman y Swagger

## Objetivo
En este apartado aprenderás a implementar autenticación robusta y seguridad en APIs REST con Django. También explorarás cómo usar Postman para probar las funcionalidades avanzadas de la API y cómo documentarla utilizando Swagger, haciéndola accesible para otros desarrolladores.

---

## Parte 1: Autenticación y seguridad en Django

### Introducción a la autenticación
La autenticación es el proceso de verificar la identidad de un usuario. En Django, el sistema de autenticación permite proteger los recursos mediante distintos métodos.

### Autenticación basada en tokens
Usar tokens permite a los clientes autenticar peticiones enviando un identificador único en los encabezados. Django REST Framework (DRF) es un paquete poderoso y extensible para crear APIs RESTful en Django. Proporciona herramientas listas para usar, como serialización de datos, clases de vistas genéricas, manejo de autenticación y permisos, que simplifican el desarrollo de APIs escalables y robustas. Usar DRF permite ahorrar tiempo y mantener las mejores prácticas al desarrollar APIs modernas. ofrece soporte nativo para autenticación basada en tokens.

#### Configuración de autenticación por tokens
1. **Instalación del paquete**:
   Asegúrate de tener instalado Django REST Framework si no lo está.
   ```bash
   pip install djangorestframework
   ```
2. **Configuración en `settings.py`**:
   ```python
   INSTALLED_APPS = [
       'rest_framework',
       'rest_framework.authtoken',
   ]

   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework.authentication.TokenAuthentication',
       ],
   }
   ```

3. **Migración**:
   Genera las tablas necesarias para almacenar los tokens. Antes de ejecutar esta migración, asegúrate de que las tablas necesarias estén definidas correctamente en `models.py` para garantizar que los datos necesarios sean estructurados adecuadamente.

   **Tabla para tokens**:
   Django REST Framework crea automáticamente una tabla llamada `authtoken_token` al ejecutar la migración. Esta tabla no es un campo adicional en el modelo de usuario, sino que almacena los tokens como registros independientes. Cada token está asociado a un usuario a través de una relación `ForeignKey`.

   Esquema de ejemplo:
   ```plaintext
   authtoken_token
   ┌──────────────┬────────────┬─────────────┐
   │ id           │ key        │ user_id     │
   ├──────────────┼────────────┼─────────────┤
   │ 1            │ abc123...  │ 42          │
   │ 2            │ xyz789...  │ 43          │
   └──────────────┴────────────┴─────────────┘
   ```

   Este diseño permite gestionar los tokens de forma eficiente y reutilizable para múltiples usuarios en el sistema.
   ```bash
   python manage.py migrate
   ```

4. **Creación de tokens para usuarios**:
   Agrega un endpoint para generar tokens usando `ObtainAuthToken`.
   ```python
   from rest_framework.authtoken.views import ObtainAuthToken
   from django.urls import path

   urlpatterns = [
       path('api-token-auth/', ObtainAuthToken.as_view(), name='api_token_auth'),
   ]
   ```
   Los usuarios podrán autenticarse enviando sus credenciales (usuario y contraseña) y recibir un token como respuesta.

#### Proteger vistas con autenticación
Para proteger las vistas, usa la clase `IsAuthenticated`, proveniente de Django REST Framework. Esta clase es utilizada para garantizar que solo los usuarios autenticados puedan acceder a las vistas protegidas. Es particularmente útil en APIs REST, donde las operaciones deben restringirse a clientes que han iniciado sesión correctamente. Además, `IsAuthenticated` se combina frecuentemente con otros permisos como `AllowAny` (que permite acceso sin restricciones) o `IsAdminUser` (que restringe el acceso a administradores), proporcionando flexibilidad en la gestión de accesos.
```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class VistaProtegida(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"mensaje": "Acceso autorizado"})
```

---


**Transición de Vistas Normales a APIView en Django REST Framework**

**Introducción a APIView**
Django REST Framework (DRF) introduce `APIView`, una clase base que permite manejar peticiones HTTP de manera estructurada para construir endpoints RESTful. A diferencia de las vistas normales, `APIView` proporciona soporte integrado para manejo de peticiones, serialización y control de excepciones, entre otras funcionalidades.

**Configurar APIView en DRF**
1. **Requisitos previos:**
   - Asegúrate de tener instalado DRF:
     ```
     pip install djangorestframework
     ```
   - Incluye `rest_framework` en `INSTALLED_APPS` de `settings.py`.

2. **Migrar una vista normal a APIView:**
   A continuación, transformamos una vista normal que devuelve un JSON en una basada en APIView.

   **Vista Normal:**
   ```python
   from django.http import JsonResponse

   def ejemplo_vista(request):
       if request.method == 'GET':
           return JsonResponse({"mensaje": "Hola desde una vista normal"})
   ```

   **Con APIView:**
   ```python
   from rest_framework.views import APIView
   from rest_framework.response import Response

   class EjemploAPIView(APIView):
       def get(self, request):
           return Response({"mensaje": "Hola desde una APIView"})
   ```

**Tipos de Peticiones y Manejo de Datos**

1. **GET: Consultar Información**
   ```python
   class ObtenerDatosAPIView(APIView):
       def get(self, request):
           datos = ["Python", "Django", "DRF"]
           return Response({"datos": datos})
   ```

2. **POST: Crear Recursos**
   ```python
   from rest_framework import status

   class CrearRecursoAPIView(APIView):
       def post(self, request):
           datos = request.data  # Extraer datos del cuerpo de la petición
           return Response({"mensaje": "Recurso creado", "datos": datos}, status=status.HTTP_201_CREATED)
   ```

3. **PUT: Actualizar Recursos**
   ```python
   class ActualizarRecursoAPIView(APIView):
       def put(self, request, recurso_id):
           datos = request.data
           return Response({"mensaje": f"Recurso {recurso_id} actualizado", "datos": datos})
   ```

4. **DELETE: Eliminar Recursos**
   ```python
   class EliminarRecursoAPIView(APIView):
       def delete(self, request, recurso_id):
           return Response({"mensaje": f"Recurso {recurso_id} eliminado"}, status=204)
   ```

**Parámetros en Endpoints**

1. **Query Params:**
   Utilizados para filtrar datos.
   ```python
   class FiltrarDatosAPIView(APIView):
       def get(self, request):
           filtro = request.query_params.get('tipo', 'general')
           return Response({"mensaje": f"Filtrado por: {filtro}"})
   ```

2. **Path Params:**
   Utilizados para identificar recursos específicos en la URL.
   ```python
   class RecursoDetalleAPIView(APIView):
       def get(self, request, recurso_id):
           return Response({"mensaje": f"Detalles del recurso {recurso_id}"})
   ```

3. **Body:**
   Los datos del cuerpo de la petición se extraen con `request.data`.
   ```python
   class CrearUsuarioAPIView(APIView):
       def post(self, request):
           nombre = request.data.get('nombre')
           return Response({"mensaje": f"Usuario {nombre} creado"})
   ```

**Configuración de URLs para APIView**
Para utilizar `APIView` en las rutas, debes configurar el archivo `urls.py`. A continuación se muestra un ejemplo de cómo hacerlo para los diferentes tipos de vistas mencionadas:

```python
from django.urls import path
from .views import (
    EjemploAPIView, ObtenerDatosAPIView, CrearRecursoAPIView,
    ActualizarRecursoAPIView, EliminarRecursoAPIView,
    FiltrarDatosAPIView, RecursoDetalleAPIView, CrearUsuarioAPIView
)

urlpatterns = [
    path('ejemplo/', EjemploAPIView.as_view(), name='ejemplo'),
    path('datos/', ObtenerDatosAPIView.as_view(), name='obtener_datos'),
    path('crear/', CrearRecursoAPIView.as_view(), name='crear_recurso'),
    path('actualizar/<int:recurso_id>/', ActualizarRecursoAPIView.as_view(), name='actualizar_recurso'),
    path('eliminar/<int:recurso_id>/', EliminarRecursoAPIView.as_view(), name='eliminar_recurso'),
    path('filtrar/', FiltrarDatosAPIView.as_view(), name='filtrar_datos'),
    path('detalle/<int:recurso_id>/', RecursoDetalleAPIView.as_view(), name='detalle_recurso'),
    path('usuario/crear/', CrearUsuarioAPIView.as_view(), name='crear_usuario'),
]
```
Cada ruta apunta al método `as_view()` de la clase `APIView`. Esto permite que la vista maneje las peticiones HTTP según lo configurado en sus métodos.





### Seguridad en la API

#### Protección contra ataques comunes
1. **CSRF**:
   Aunque las APIs REST suelen desactivar CSRF, usa tokens o encabezados personalizados para seguridad adicional.

2. **CORS**:
   Permite o restringe el acceso a la API desde orígenes externos mediante el paquete `django-cors-headers`.
   ```bash
   pip install django-cors-headers
   ```
   Configuración en `settings.py`:
   ```python
   INSTALLED_APPS += ['corsheaders']

   MIDDLEWARE = [
       'corsheaders.middleware.CorsMiddleware',
   ]

   CORS_ALLOWED_ORIGINS = [
       'http://localhost:3000',
   ]
   ```

3. **Rate Limiting**:
   Usa paquetes como `django-ratelimit` para limitar la cantidad de solicitudes que un cliente puede realizar en un periodo de tiempo.

#### Uso de permisos personalizados
Puedes crear permisos para garantizar acceso basado en roles o condiciones específicas.
```python
from rest_framework.permissions import BasePermission

class EsAdmin(BasePermission):
    def has_permission(self, request, view):
        return request.user and request.user.is_staff
```
Aplica el permiso en vistas:
```python
class VistaAdmin(APIView):
    permission_classes = [EsAdmin]

    def get(self, request):
        return Response({"mensaje": "Solo administradores"})
```

---

## Parte 2: Uso avanzado de Postman

### Instalación y configuración
Descarga e instala Postman desde [https://www.postman.com/downloads/](https://www.postman.com/downloads/).

### Probar autenticación basada en tokens
1. **Generar token**:
   - Configura una petición POST al endpoint `/api-token-auth/`.
   - En el cuerpo de la petición (JSON):
     ```json
     {
         "username": "usuario",
         "password": "contraseña"
     }
     ```
   - Guarda el token de la respuesta.

2. **Enviar token en solicitudes protegidas**:
   - En las peticiones subsiguientes, agrega un encabezado `Authorization`:
     ```
     Authorization: Token <token_aquí>
     ```

### Colecciones en Postman
Organiza tus peticiones en colecciones:
1. Crea una nueva colección para la API.
2. Agrega carpetas como `Autenticación`, `Productos`, etc.
3. Guarda las configuraciones para compartir con el equipo.


---

## Parte 3: Documentación y uso de Swagger

### ¿Qué es Swagger?
Swagger (ahora OpenAPI) es un estándar para documentar y probar APIs. Django REST Framework tiene soporte para generar documentación automáticamente mediante paquetes como `drf-yasg`.

### Configuración de Swagger con `drf-yasg`
1. **Instalar paquete**:
   ```bash
   pip install drf-yasg
   ```
2. **Añadir a `INSTALLED_APPS` en `settings.py`**
    ```python
    INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'drf_yasg',
    ]
    ```

   
2. **Configurar en `urls.py`**:
   ```python
   from drf_yasg.views import get_schema_view
   from drf_yasg import openapi
   from rest_framework.permissions import AllowAny

   schema_view = get_schema_view(
       openapi.Info(
           title="API Documentación",
           default_version="v1",
           description="Documentación de la API",
       ),
       public=True,
       permission_classes=[AllowAny],
   )

   urlpatterns += [
       path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
       path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
   ]
   ```

3. **Acceder a Swagger**:
   - Visita `/swagger/` para ver la interfaz de Swagger.
   - Visita `/redoc/` para una versión alternativa de la documentación.

### Añadir descripciones personalizadas
Usa docstrings en las vistas para enriquecer la documentación:
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from drf_yasg.utils import swagger_auto_schema
from drf_yasg import openapi

class ProductoAPI(APIView):
    @swagger_auto_schema(
        operation_description="Obtener lista de productos",
        responses={200: openapi.Response("Lista de productos")},
    )
    def get(self, request):
        return Response([])
```
#### Añadir parámetros
```python
@swagger_auto_schema(
 operation_description="Lista eventos filtrados por título o fecha.",
        manual_parameters=[
            openapi.Parameter('titulo', openapi.IN_QUERY, description="Filtrar por título", type=openapi.TYPE_STRING),
            openapi.Parameter('fecha', openapi.IN_QUERY, description="Filtrar por fecha (YYYY-MM-DD)", type=openapi.TYPE_STRING),
            openapi.Parameter('pagina', openapi.IN_QUERY, description="Número de página", type=openapi.TYPE_INTEGER),
        ],
        responses={200: openapi.Response(description="Lista de eventos")}
)
```
#### Añadir cuerpo a la petición
```python
@swagger_auto_schema(
        operation_description="Crea un nuevo evento.",
        request_body=openapi.Schema(
            type=openapi.TYPE_OBJECT,
            properties={
                'titulo': openapi.Schema(type=openapi.TYPE_STRING, description='Título del evento'),
                'descripcion': openapi.Schema(type=openapi.TYPE_STRING, description='Descripción del evento'),
                'fecha_hora': openapi.Schema(type=openapi.TYPE_STRING, format="date-time",
                                             description='Fecha y hora del evento'),
                'capacidad': openapi.Schema(type=openapi.TYPE_INTEGER, description='Capacidad máxima de asistentes'),
            },
            required=['titulo', 'descripcion', 'fecha_hora', 'capacidad']
        ),
        responses={201: openapi.Response(description="Evento creado"),403: openapi.Response(description="No tienes permisos para crear eventos.")}
    )
```
---

# 4: Uso de plantillas en Django y manejo dinámico de contenido

## Objetivo
En este apartado aprenderás a trabajar con el sistema de plantillas de Django para generar contenido HTML dinámico y a optimizar la interacción entre vistas y plantillas. Se incluyen ejemplos prácticos y explicaciones detalladas para que puedas aplicar este conocimiento en tus proyectos.

---

## Parte 1: Introducción al sistema de plantillas en Django

### ¿Qué es el sistema de plantillas de Django?
El sistema de plantillas de Django permite generar contenido HTML dinámico mediante el uso de archivos de plantilla. Combina código HTML estático con variables y etiquetas que se procesan en el servidor para crear páginas personalizadas.

### Características principales
1. **Separación de lógica y presentación**: El código Python y HTML están separados.
2. **Compatibilidad con contenido dinámico**: Puedes mostrar datos provenientes de la base de datos o variables de contexto.
3. **Uso de etiquetas y filtros**: Herramientas simples para manipular datos dentro de las plantillas.

---

## Parte 2: Configuración del sistema de plantillas

### ¿Qué son las plantillas y cómo se crean?
Las plantillas en Django son archivos que contienen código HTML mezclado con variables, etiquetas y filtros que se procesan en el servidor antes de enviarse al cliente. Se usan para generar contenido dinámico a partir de datos proporcionados por vistas.

Para crear plantillas:
1. Crea una carpeta llamada `templates` dentro del directorio raíz del proyecto o en una aplicación específica.
2. Dentro de la carpeta `templates`, organiza tus archivos HTML según las necesidades del proyecto. Puedes crear subcarpetas para separar plantillas por módulos.

Ejemplo de estructura de carpetas:
```
proyecto/
├── app/
│   ├── templates/
│   │   ├── app/
│   │   │   ├── inicio.html
│   │   │   ├── contacto.html
│   │   │   ├── base.html
```
3. Crea archivos HTML con el contenido estático y dinámico necesario.

### Configuración inicial
Asegúrate de que tu proyecto esté configurado para usar plantillas.

1. **Directorio de plantillas**:
   En `settings.py`, define el directorio donde estarán las plantillas:
   ```python
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
           'DIRS': [BASE_DIR / 'templates'],
           'APP_DIRS': True,
           'OPTIONS': {
               'context_processors': [
                   'django.template.context_processors.debug',
                   'django.template.context_processors.request',
                   'django.contrib.auth.context_processors.auth',
                   'django.contrib.messages.context_processors.messages',
               ],
           },
       },
   ]
   ```
   Asegúrate de crear la carpeta `templates` en el directorio base del proyecto.

2. **Estructura básica**:
   Crea un archivo HTML en `templates/`:
   ```html
   <!-- templates/base.html -->
   <!DOCTYPE html>
   <html>
   <head>
       <title>{% block title %}Mi sitio{% endblock %}</title>
   </head>
   <body>
       {% block content %}{% endblock %}
   </body>
   </html>
   ```

---

## Parte 3: Uso de plantillas dinámicas

### Renderizado de plantillas desde vistas
En Django, las vistas envían datos a las plantillas mediante un contexto.

#### Ejemplo básico
1. **Vista en `views.py`**:
   ```python
   from django.shortcuts import render

   def inicio(request):
       contexto = {
           'nombre': 'Django',
           'version': 4.0,
       }
       return render(request, 'inicio.html', contexto)
   ```
2. **Definición en `urls.py`**
```python
from django.urls import path
from .views import inicio

urlpatterns = [
    path('', inicio, name='inicio'),
]
```
De esta forma, cuando accedas a la URL raíz (`/`), Django cargará la plantilla `inicio.html`.

3. **Plantilla en `templates/inicio.html`**:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Inicio</title>
   </head>
   <body>
       <h1>Bienvenido a {{ nombre }}</h1>
       <p>Estás usando la versión {{ version }}</p>
   </body>
   </html>
   ```

Resultado: Al visitar esta vista, el navegador mostrará:
```
Bienvenido a Django
Estás usando la versión 4.0
```

---

## Parte 4: Herencia de plantillas

Django permite reutilizar estructuras HTML mediante la herencia de plantillas.

1. **Plantilla base**:
   Define la estructura general del sitio:
   ```html
   <!-- templates/base.html -->
   <!DOCTYPE html>
   <html>
   <head>
       <title>{% block title %}Mi sitio{% endblock %}</title>
   </head>
   <body>
       <header>
           <h1>Mi sitio web</h1>
       </header>
       <main>
           {% block content %}{% endblock %}
       </main>
       <footer>
           <p>&copy; 2025 Mi sitio</p>
       </footer>
   </body>
   </html>
   ```

2. **Plantilla hija**:
   Extiende la plantilla base:
   ```html
   <!-- templates/inicio.html -->
   {% extends 'base.html' %}

   {% block title %}Inicio{% endblock %}

   {% block content %}
   <h2>Bienvenido</h2>
   <p>Esta es la página de inicio.</p>
   {% endblock %}
   ```

Resultado: La página generada combinará la estructura de `base.html` con el contenido de `inicio.html`.

---

## Parte 5: Recogida y uso de token en las plantillas


Para obtener y almacenar el token de autenticación después de iniciar sesión, puedes usar JavaScript dentro de de tu página de login:

```html
<script>
    document.getElementById('login-form').addEventListener('submit', async function(event) {
        event.preventDefault();
        const formData = new FormData(this);
        const response = await fetch('http://127.0.0.1:8000/tuLogin', {
            method: 'POST',
            body: JSON.stringify({
                username: formData.get('username'),
                password: formData.get('password')
            }),
            headers: {
                'Content-Type': 'application/json'
            }
        });
        const data = await response.json();
        localStorage.setItem('token', data.token);
    });
</script>
```

Para usar el token en una solicitud protegida:

```javascript
fetch('http://127.0.0.1:8000/tu_endpoint', {
    method: 'GET',// o el método correspondiente
    headers: {
        'Authorization': 'Token ' + localStorage.getItem('token')
    }
})
.then(response => response.json())
.then(data => console.log(data));
```

---

## Parte 4: Uso de etiquetas y filtros

### Etiquetas comunes
1. **For loops**:
   ```html
   <ul>
       {% for producto in productos %}
       <li>{{ producto.nombre }}: ${{ producto.precio }}</li>
       {% endfor %}
   </ul>
   ```

2. **If conditions**:
   ```html
   {% if usuario.is_authenticated %}
       <p>Hola, {{ usuario.username }}</p>
   {% else %}
       <p>Inicia sesión para continuar.</p>
   {% endif %}
   ```

### Filtros útiles
1. **Capitalizar texto**:
   ```html
   <p>{{ texto|capfirst }}</p>
   ```

2. **Formato de fechas**:
   ```html
   <p>Fecha: {{ fecha|date:"d M Y" }}</p>
   ```

3. **Truncar texto**:
   ```html
   <p>{{ descripcion|truncatechars:50 }}</p>
   ```

---












