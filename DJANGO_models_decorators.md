# Semana 1: creación del modelo y decoradores en django

## Objetivo
Esta guía proporcionará la teoría y ejemplos necesarios para trabajar con modelos y decoradores en django. Aprenderás a crear modelos desde `models.py` y a utilizar decoradores para mejorar la funcionalidad y seguridad de las vistas en django.

---

## Parte 1: modelos en django
### ¿Qué es un modelo?
En django, un modelo es una representación de una tabla de base de datos en forma de clase python. Cada clase define la estructura de una tabla, incluidos los campos y las relaciones entre ellos.

Un modelo no solo define cómo se estructura la tabla en la base de datos, sino también cómo interactuamos con los datos de esa tabla en el código. Esto incluye consultas, inserciones, actualizaciones y eliminaciones.

### Crear un modelo
Un modelo se define en el archivo `models.py` de la aplicación django.

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
- `CharField`: almacena cadenas de texto. Se define con un límite de longitud.
- `DecimalField`: almacena números decimales, ideal para valores monetarios.
- `IntegerField`: almacena números enteros.
- `TextField`: similar a `CharField`, pero sin límite de longitud.
- `blank=True`: permite que el campo esté vacío en los formularios.
- `null=True`: permite que el campo acepte valores nulos en la base de datos.

#### **Migraciones**
django utiliza un sistema de migraciones para sincronizar los modelos con la base de datos.

1. **Crear la migración:** Este comando genera un archivo de migración basado en los cambios en los modelos.
   ```bash
   python manage.py makemigrations
   ```

2. **Aplicar la migración:** Este comando ejecuta las migraciones y sincroniza los cambios con la base de datos.
   ```bash
   python manage.py migrate
   ```

### Relaciones entre modelos
Los modelos en django soportan relaciones para definir cómo interactúan las tablas entre sí.

#### Tipos de relaciones
- **Uno a muchos (`ForeignKey`)**: Un registro en una tabla está relacionado con muchos registros en otra.
- **Muchos a muchos (`ManyToManyField`)**: Muchos registros de una tabla están relacionados con muchos registros de otra tabla.

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
Los modelos en django permiten añadir métodos personalizados para extender su funcionalidad. Por ejemplo:
```python
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    precio = models.DecimalField(max_digits=10, decimal_places=2)

    def aplicar_descuento(self, porcentaje):
        return self.precio * (1 - porcentaje / 100)
```
Este método calcula el precio del producto después de aplicar un descuento.

---

## Parte 2: decoradores en django
### ¿Qué es un decorador?
Un decorador es una función que modifica el comportamiento de otra función o método sin alterar directamente su código fuente. En django, los decoradores son una herramienta poderosa para manejar la autenticación, permisos, restricciones de métodos HTTP y más.

Los decoradores en django permiten:
- Restringir el acceso a ciertas vistas.
- Manejar métodos HTTP permitidos en las vistas.
- Proteger contra vulnerabilidades de seguridad como ataques CSRF.
- Asegurar la consistencia de las operaciones en la base de datos.

### Decoradores comunes en django
#### 1. **`@login_required`**
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

#### 2. **`@csrf_exempt`**
Excluye una vista de la protección contra ataques CSRF (Cross-Site Request Forgery). Esto puede ser útil para integraciones con servicios externos que no envían el token CSRF.

### ¿Qué es un ataque CSRF?
Un ataque CSRF ocurre cuando un atacante engaña a un usuario autenticado para que realice acciones no deseadas en una aplicación en la que el usuario está autenticado. Por ejemplo:

1. El usuario inicia sesión en su cuenta de banca en línea.
2. Mientras navega, visita un sitio web malicioso que envía una solicitud POST a la banca en línea para transferir dinero a la cuenta del atacante.
3. Debido a que la sesión del usuario está activa, el servidor confía en la solicitud y realiza la transferencia.

### **Prevención de ataques CSRF en django**
django incluye protección automática contra CSRF al habilitar el middleware correspondiente y al usar el token CSRF en los formularios.

#### **Ejemplo de protección automática:**
```html
<form method="post">
  {% csrf_token %}
  <input type="text" name="nombre" placeholder="Nombre">
  <button type="submit">Enviar</button>
</form>
```

El token `csrf_token` garantiza que solo las solicitudes legítimas del usuario sean aceptadas.

#### **Ejemplo de vista sin protección:**
```python
from django.views.decorators.csrf import csrf_exempt
from django.http import JsonResponse

@csrf_exempt
def api_sin_csrf(request):
    return JsonResponse({'mensaje': 'Acceso sin CSRF'})
```
**Precaución:** Deshabilitar la protección CSRF puede exponer tu aplicación a riesgos de seguridad.

---

#### 3. **`@require_http_methods`**
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

#### 4. **`@transaction.atomic`**
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

### Actividad práctica
#### Crear una aplicación de tienda virtual
1. **Definir los modelos:**
   - Crea los modelos `Categoria` y `Producto` con relaciones y campos adecuados.
2. **Protección de vistas:**
   - Implementa una vista protegida con `@login_required` para listar los productos.
3. **Gestión de métodos HTTP:**
   - Crea una vista con `@require_http_methods` para manejar la creación de productos vía `POST` y su listado vía `GET`.
4. **Prueba de funcionalidad:**
   - Realiza migraciones y verifica las vistas en el navegador.

Con esto, tus alumnos tendrán los conocimientos necesarios para crear modelos y utilizar decoradores en django de manera efectiva. Si deseas ampliar algún punto o agregar ejemplos adicionales, no dudes en indicarlo.

