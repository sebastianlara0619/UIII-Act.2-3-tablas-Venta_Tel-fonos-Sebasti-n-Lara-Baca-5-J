Perfecto ✅
Aquí tienes **TODOS los archivos completos** listos para copiar y pegar en tu proyecto **Venta_Teléfonos** (solo Proveedor totalmente funcional).
Incluyo estructura de carpetas, contenido de cada archivo y las instrucciones finales para correr el servidor.
Después de pegar todo, solo corres `python manage.py runserver 8017` y tendrás tu CRUD funcionando.

---

## 🧱 ESTRUCTURA COMPLETA DE CARPETAS

Crea esta estructura dentro de tu carpeta principal `UIII_Venta_Telefonos_0619`:

```
UIII_Venta_Telefonos_0619/
├─ .venv/
├─ backend_Venta_Telefonos/
│  ├─ __init__.py
│  ├─ asgi.py
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
├─ app_Venta_Telefonos/
│  ├─ __init__.py
│  ├─ admin.py
│  ├─ apps.py
│  ├─ migrations/
│  │  └─ __init__.py
│  ├─ models.py
│  ├─ urls.py
│  ├─ views.py
│  └─ templates/
│     ├─ base.html
│     ├─ navbar.html
│     ├─ footer.html
│     ├─ inicio.html
│     └─ proveedor/
│        ├─ agregar_proveedor.html
│        ├─ ver_proveedores.html
│        ├─ actualizar_proveedor.html
│        └─ borrar_proveedor.html
└─ manage.py
```

---

## 🧩 1. `models.py`

```python
from django.db import models

class Proveedor(models.Model):
    nombre = models.CharField(max_length=100)
    empresa = models.CharField(max_length=100)
    telefono = models.CharField(max_length=20)
    email = models.EmailField()
    direccion = models.CharField(max_length=200)
    ciudad = models.CharField(max_length=100)
    fecha_registro = models.DateField(auto_now_add=True)

    def __str__(self):
        return f"{self.nombre} ({self.empresa})"


class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    marca = models.CharField(max_length=50)
    modelo = models.CharField(max_length=50)
    precio_venta = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField()
    descripcion = models.TextField(blank=True, null=True)
    fecha_ingreso = models.DateField(auto_now_add=True)

    proveedores = models.ManyToManyField(Proveedor, through='Producto_Proveedor', related_name='productos')

    def __str__(self):
        return f"{self.nombre} - {self.marca} ({self.modelo})"


class Producto_Proveedor(models.Model):
    producto = models.ForeignKey(Producto, on_delete=models.CASCADE, related_name='producto_proveedores')
    proveedor = models.ForeignKey(Proveedor, on_delete=models.CASCADE, related_name='productos_proveedor')

    precio_compra = models.DecimalField(max_digits=10, decimal_places=2)
    cantidad_disponible = models.PositiveIntegerField()
    garantia_meses = models.PositiveIntegerField(default=12)
    fecha_entrega = models.DateField()
    lote = models.CharField(max_length=50, blank=True, null=True)

    def __str__(self):
        return f"{self.producto.marca} {self.producto.modelo} - {self.proveedor.empresa}"
```

---

## ⚙️ 2. `admin.py`

```python
from django.contrib import admin
from .models import Proveedor, Producto, Producto_Proveedor

@admin.register(Proveedor)
class ProveedorAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'empresa', 'telefono', 'email', 'ciudad', 'fecha_registro')
    search_fields = ('nombre', 'empresa', 'ciudad')

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'marca', 'modelo', 'precio_venta', 'stock', 'fecha_ingreso')
    search_fields = ('nombre', 'marca', 'modelo')

@admin.register(Producto_Proveedor)
class ProductoProveedorAdmin(admin.ModelAdmin):
    list_display = ('producto', 'proveedor', 'precio_compra', 'cantidad_disponible', 'fecha_entrega')
    list_select_related = ('producto', 'proveedor')
```

---

## 🌐 3. `urls.py` (de la app)

```python
from django.urls import path
from . import views

app_name = 'app_Venta_Telefonos'

urlpatterns = [
    path('', views.inicio_ventas, name='inicio'),

    # CRUD Proveedores
    path('proveedor/agregar/', views.agregar_proveedor, name='agregar_proveedor'),
    path('proveedor/ver/', views.ver_proveedores, name='ver_proveedores'),
    path('proveedor/actualizar/<int:proveedor_id>/', views.actualizar_proveedor, name='actualizar_proveedor'),
    path('proveedor/realizar_actualizacion/<int:proveedor_id>/', views.realizar_actualizacion_proveedor, name='realizar_actualizacion_proveedor'),
    path('proveedor/borrar/<int:proveedor_id>/', views.borrar_proveedor, name='borrar_proveedor'),
]
```

---

## 🌍 4. `urls.py` (del proyecto `backend_Venta_Telefonos`)

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Venta_Telefonos.urls', namespace='app_Venta_Telefonos')),
]
```

---

## 🧠 5. `views.py`

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Proveedor
from django.utils import timezone

def inicio_ventas(request):
    return render(request, 'inicio.html', {'current_year': timezone.now().year})

def agregar_proveedor(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre', '').strip()
        empresa = request.POST.get('empresa', '').strip()
        telefono = request.POST.get('telefono', '').strip()
        email = request.POST.get('email', '').strip()
        direccion = request.POST.get('direccion', '').strip()
        ciudad = request.POST.get('ciudad', '').strip()

        Proveedor.objects.create(
            nombre=nombre,
            empresa=empresa,
            telefono=telefono,
            email=email,
            direccion=direccion,
            ciudad=ciudad,
        )
        return redirect('app_Venta_Telefonos:ver_proveedores')

    return render(request, 'proveedor/agregar_proveedor.html')

def ver_proveedores(request):
    proveedores = Proveedor.objects.all().order_by('-fecha_registro')
    return render(request, 'proveedor/ver_proveedores.html', {'proveedores': proveedores})

def actualizar_proveedor(request, proveedor_id):
    proveedor = get_object_or_404(Proveedor, id=proveedor_id)
    return render(request, 'proveedor/actualizar_proveedor.html', {'proveedor': proveedor})

def realizar_actualizacion_proveedor(request, proveedor_id):
    proveedor = get_object_or_404(Proveedor, id=proveedor_id)
    if request.method == 'POST':
        proveedor.nombre = request.POST.get('nombre', proveedor.nombre)
        proveedor.empresa = request.POST.get('empresa', proveedor.empresa)
        proveedor.telefono = request.POST.get('telefono', proveedor.telefono)
        proveedor.email = request.POST.get('email', proveedor.email)
        proveedor.direccion = request.POST.get('direccion', proveedor.direccion)
        proveedor.ciudad = request.POST.get('ciudad', proveedor.ciudad)
        proveedor.save()
        return redirect('app_Venta_Telefonos:ver_proveedores')
    return redirect('app_Venta_Telefonos:ver_proveedores')

def borrar_proveedor(request, proveedor_id):
    proveedor = get_object_or_404(Proveedor, id=proveedor_id)
    if request.method == 'POST':
        proveedor.delete()
        return redirect('app_Venta_Telefonos:ver_proveedores')
    return render(request, 'proveedor/borrar_proveedor.html', {'proveedor': proveedor})
```

---

## 🎨 6. `base.html`

```html
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block title %}Venta Teléfonos{% endblock %}</title>

  <!-- Bootstrap -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

  <style>
    body { background-color: #f7f9fb; color: #333; }
    .navbar-brand { font-weight: 600; }
    .footer { position: fixed; bottom: 0; width: 100%; background: #fff; padding: 10px 0; box-shadow: 0 -2px 4px rgba(0,0,0,0.05); }
    .content { padding-bottom: 70px; }
  </style>
</head>
<body>
  {% include 'navbar.html' %}

  <main class="container content py-4">
    {% block content %}{% endblock %}
  </main>

  {% include 'footer.html' %}

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## 🧭 7. `navbar.html`

```html
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
  <div class="container">
    <a class="navbar-brand" href="{% url 'app_Venta_Telefonos:inicio' %}">
      <img src="https://img.icons8.com/ios-filled/24/000000/smartphone.png" alt=""> Venta de Teléfonos
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#mainNav">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="mainNav">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'app_Venta_Telefonos:inicio' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Proveedores</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'app_Venta_Telefonos:agregar_proveedor' %}">Agregar</a></li>
            <li><a class="dropdown-item" href="{% url 'app_Venta_Telefonos:ver_proveedores' %}">Ver Todos</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Productos</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar</a></li>
            <li><a class="dropdown-item" href="#">Ver Todos</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Producto-Proveedor</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar</a></li>
            <li><a class="dropdown-item" href="#">Ver Todos</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

---

## 🦶 8. `footer.html`

```html
<footer class="footer text-center">
  <div class="container">
    <small>&copy; {{ current_year }} Derechos reservados. Creado por Ing. Sebastián Lara, Cbtis 128</small>
  </div>
</footer>
```

---

## 🏠 9. `inicio.html`

```html
{% extends "base.html" %}
{% block title %}Inicio - Venta Teléfonos{% endblock %}

{% block content %}
<div class="row">
  <div class="col-md-8">
    <h1>Bienvenido al Sistema de Administración Venta de Teléfonos</h1>
    <p>Gestiona proveedores, productos y relaciones fácilmente.</p>
  </div>
  <div class="col-md-4">
    <img src="https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?q=80&w=1000&auto=format&fit=crop" class="img-fluid rounded shadow-sm">
  </div>
</div>
{% endblock %}
```

---

## 🧾 10. Templates del CRUD de Proveedores

### ➕ `agregar_proveedor.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Agregar Proveedor</h2>
<form method="post">
  {% csrf_token %}
  <div class="row">
    <div class="col-md-6 mb-3">
      <label>Nombre</label>
      <input class="form-control" name="nombre" required>
    </div>
    <div class="col-md-6 mb-3">
      <label>Empresa</label>
      <input class="form-control" name="empresa" required>
    </div>
    <div class="col-md-6 mb-3">
      <label>Teléfono</label>
      <input class="form-control" name="telefono">
    </div>
    <div class="col-md-6 mb-3">
      <label>Email</label>
      <input class="form-control" type="email" name="email">
    </div>
    <div class="col-md-6 mb-3">
      <label>Dirección</label>
      <input class="form-control" name="direccion">
    </div>
    <div class="col-md-6 mb-3">
      <label>Ciudad</label>
      <input class="form-control" name="ciudad">
    </div>
  </div>
  <button class="btn btn-primary">Guardar</button>
</form>
{% endblock %}
```

---

### 📋 `ver_proveedores.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Proveedores</h2>
<table class="table table-striped align-middle">
  <thead class="table-light">
    <tr>
      <th>Empresa</th>
      <th>Nombre</th>
      <th>Teléfono</th>
      <th>Email</th>
      <th>Ciudad</th>
      <th>Registro</th>
      <th>Acciones</th>
    </tr>
  </thead>
  <tbody>
    {% for p in proveedores %}
    <tr>
      <td>{{ p.empresa }}</td>
      <td>{{ p.nombre }}</td>
      <td>{{ p.telefono }}</td>
      <td>{{ p.email }}</td>
      <td>{{ p.ciudad }}</td>
      <td>{{ p.fecha_registro }}</td>
      <td>
        <a href="{% url 'app_Venta_Telefonos:actualizar_proveedor' p.id %}" class="btn btn-sm btn-info">Editar</a>
        <a href="{% url 'app_Venta_Telefonos:borrar_proveedor' p.id %}" class="btn btn-sm btn-danger">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="7">No hay proveedores registrados.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

---

### ✏️ `actualizar_proveedor.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Actualizar Proveedor</h2>
<form method="post" action="{% url 'app_Venta_Telefonos:realizar_actualizacion_proveedor' proveedor.id %}">
  {% csrf_token %}
  <div class="mb-3"><label>Nombre</label><input class="form-control" name="nombre" value="{{ proveedor.nombre }}"></div>
  <div class="mb-3"><label>Empresa</label><input class="form-control" name="empresa" value="{{ proveedor.empresa }}"></div>
  <div class="mb-3"><label>Teléfono</label><input class="form-control" name="telefono" value="{{ proveedor.telefono }}"></div>
  <div class="mb-3"><label>Email</label><input class="form-control" name="email" value="{{ proveedor.email }}"></div>
  <div class="mb-3"><label>Dirección</label><input class="form-control" name="direccion" value="{{ proveedor.direccion }}"></div>
  <div class="mb-3"><label>Ciudad</label><input class="form-control" name="ciudad" value="{{ proveedor.ciudad }}"></div>
  <button class="btn btn-success">Guardar cambios</button>
  <a href="{% url 'app_Venta_Telefonos:ver_proveedores' %}" class="btn btn-secondary">Cancelar</a>
</form>
{% endblock %}
```

---

### ❌ `borrar_proveedor.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Confirmar eliminación</h2>
<p>¿Desea borrar al proveedor <strong>{{ proveedor.empresa }} - {{ proveedor.nombre }}</strong>?</p>
<form method="post">
  {% csrf_token %}
  <button type="submit" class="btn btn-danger">Sí, borrar</button>
  <a href="{% url 'app_Venta_Telefonos:ver_proveedores' %}" class="btn btn-secondary">Cancelar</a>
</form>
{% endblock %}
```

---

## 🚀 11. Pasos finales

```bash
# dentro del entorno virtual
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser  # opcional
python manage.py runserver 8017
```

Abre en navegador:
👉 **[http://127.0.0.1:8017/](http://127.0.0.1:8017/)**
Ya tendrás el **CRUD completo de Proveedores** con Bootstrap y estructura lista para seguir con Productos y Producto_Proveedor.

---

¿Deseas que te genere **ahora los CRUD completos** de `Producto` y `Producto_Proveedor` también (idéntico estilo, sin forms.py, con plantillas y vistas)?
Puedo entregártelos listos para pegar en la misma estructura.
