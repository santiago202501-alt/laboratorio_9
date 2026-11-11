# ClassroomHub

Portal de ejemplo construido con Django. En el Laboratorio 7 el catálogo de
recursos pasa a guardarse en MySQL o MariaDB mediante el ORM de Django. En el
Laboratorio 8 se añaden registro, login/logout, un panel privado con sesión y
una vista de revisión restringida por grupo y permiso. En el Laboratorio 9 se
añade un CRUD de recursos (detalle público; crear, editar y eliminar con
permisos) mediante `ModelForm` y vistas genéricas.

## Requisitos

- Python 3.12, 3.13, 3.14 o 3.15
- Django 6.1.3
- MySQL 8.4+ o MariaDB 10.11+ (con Django 5.2 LTS: MySQL 8.0.11+ o MariaDB 10.5+)
- Permiso para crear una base de datos y una cuenta SQL local

Si usas Python 3.10 u 3.11, cambia la dependencia por `Django==5.2.19` y usa la
documentación de Django 5.2.

## Base de datos

Desde una cuenta administrativa del servidor SQL:

```sql
CREATE DATABASE IF NOT EXISTS classroom_hub
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'classroomhub_user'@'127.0.0.1'
    IDENTIFIED BY 'replace_with_a_private_password';

GRANT ALL PRIVILEGES ON classroom_hub.*      TO 'classroomhub_user'@'127.0.0.1';
GRANT ALL PRIVILEGES ON test_classroom_hub.* TO 'classroomhub_user'@'127.0.0.1';
```

`test_classroom_hub` es la base descartable que Django crea y elimina al
ejecutar las pruebas. No uses `root` ni `GRANT ... ON *.*`.

## Instalación

```bash
python -m venv .venv
# Windows PowerShell:
.\.venv\Scripts\Activate.ps1
# Linux/macOS:
source .venv/bin/activate

python -m pip install -r requirements.txt
```

`mysqlclient` puede requerir encabezados del cliente y un compilador en algunas
plataformas; consulta su documentación oficial si la instalación falla.

## Configuración

Copia la plantilla y escribe tu contraseña real (el archivo `.env` no se versiona):

```bash
cp .env.example .env        # Windows: copy .env.example .env
```

Variables: `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`.

`MARIADB_COMPAT_104=1` activa un parche **no soportado** para servidores
MariaDB 10.4. Déjalo en `0` si tu servidor cumple los mínimos de arriba.

## Migraciones, administrador y ejecución

```bash
python manage.py check
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

- Sitio público: http://127.0.0.1:8000/
- Administrador: http://127.0.0.1:8000/admin/ (crea categorías y recursos).
  Entra con el superusuario que creaste con `createsuperuser`; nunca escribas
  credenciales reales en este archivo ni las subas a Git.
  USER: admin
  password: admin12345

## Pruebas

```bash
python manage.py test pages
```

Las 61 pruebas (10 del Laboratorio 7, 18 del Laboratorio 8, 3 del reto
adicional del panel y 30 del CRUD del Laboratorio 9) usan la base separada `test_classroom_hub`; los datos de
desarrollo no se tocan.

## Rutas

| Ruta | Acceso | Descripción |
|---|---|---|
| `/` | Público | Página principal |
| `/resources/` | Público | Catálogo leído de la base de datos |
| `/about/` | Público | Propósito del proyecto |
| `/accounts/signup/` | Público | Registro de cuentas normales (sin privilegios) |
| `/accounts/login/` | Público | Inicio de sesión (`LoginView`) |
| `/accounts/logout/` | Solo POST | Cierre de sesión (`LogoutView`); GET devuelve 405 |
| `/dashboard/` | Autenticado | Panel privado con contador en sesión |
| `/dashboard/summary/` | Autenticado | Resumen de solo lectura del catálogo (reto adicional) |
| `/resources/<pk>/` | Público | Detalle de un recurso (404 si no existe) |
| `/resources/new/` | `pages.add_resource` | GET muestra el formulario, POST crea |
| `/resources/<pk>/edit/` | `pages.change_resource` | GET muestra el formulario, POST actualiza |
| `/resources/<pk>/delete/` | `pages.delete_resource` | GET solo confirma; POST elimina |
| `/resources/review/` | Permiso `pages.view_resource` | Revisión del catálogo; anónimo → login, sin permiso → 403 |

## Grupo de revisores

Estar autenticado no basta para entrar en `/resources/review/`. Para autorizar
a una cuenta:

1. Inicia sesión en `/admin/` con el superusuario.
2. En **Groups** crea el grupo `Catalog reviewers` y asígnale únicamente el
   permiso `pages | resource | Can view resource`.
3. En **Users**, añade la cuenta al grupo. No marques `Staff status`: un revisor
   no necesita acceder al administrador.

## Grupo de editores (CRUD)

`Catalog editors` es independiente de `Catalog reviewers`. Créalo en
`/admin/` → **Groups** con los permisos `Can view resource`,
`Can add resource`, `Can change resource` y `Can delete resource`, y asígnalo
solo a una cuenta de prueba desde **Users**. No marques `Staff status`. El
formulario de registro público nunca asigna grupos ni permisos.

Las categorías siguen gestionándose desde `/admin/`: crea algunas antes de
usar el formulario de recursos, porque `category` es una selección obligatoria.

Alternativa por shell para el grupo de revisores (`python manage.py shell`): consulta la práctica 8.5 del
laboratorio. Las cuentas se crean desde `/accounts/signup/`; ninguna recibe
privilegios por registrarse.

## Estructura

- `config/`: configuración general y rutas del proyecto.
- `pages/`: modelos (`Category`, `Resource`), migraciones, admin, vistas, rutas,
  plantillas y pruebas.

El CRUD cubre solo recursos (no categorías, archivos, API ni paginación). No
incluye recuperación de contraseña por correo, OAuth, MFA ni despliegue a
producción; que las pruebas pasen no equivale a una auditoría de seguridad.
