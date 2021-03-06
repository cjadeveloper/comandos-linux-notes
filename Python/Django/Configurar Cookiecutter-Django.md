# Configurar Plantilla Cookiecutter-Django

[Cookiecutter Django](https://github.com/pydanny/cookiecutter-django) es un marco para poner en marcha rápidamente los proyectos de Django listos para producción.

## Instalar [cookiecutter](https://github.com/audreyr/cookiecutter) globalmente.

```console
pip install "cookiecutter>=1.4.0"
```

## Instalar cookiecutter-django y correrlo localmente para desarrollo.

1. Crearse un entorno virtual con python 3.6
2. Con el entorno activo, ejecutamos cookiecutter contra el repositorio de cookiecutter-django y completamos las opciones.
3. Vinculamos el venv actual con el projecto e instalamos los requerimientos para desarrollo local.
4. Creamos y configuramos la base de datos en PostgreSQL para usarla con Django.
5. Corregimos error de Celery (si especificamos en las opciones que use Celery).
6. Aplicamos las migraciones.
7. Set/Unset variables de entorno de Django con virtualenvwrapper.

### 1. Crearse un entorno virtual con python 3.6

Con [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/projects.html) podemos aprovechar su potencia y crear ya un proyecto que nos cree un entorno virtual en el directorio WORKON_HOME que tengamos configurado y una carpeta de projecto en el directorio PROJECT_HOME que tengamos configurado en nuestra variable de entorno, lo active y nos quede listo ya para trabajar.

*Si bien podemos hacer esto de un solo paso con `mkproject`, al momento de ejecutar `cookiecutter` contra el repo de `cookiecutter-django` nos va dar error porque la carpeta del projecto ya existe. Lo que vamos a hacer, entonces es crear el entorno virtual primero, situarnos en nuestra carpeta de projecto, PROJECT_HOME o en la que queramos guardar nuestro projecto, correr `cookiecutter` alli y luego vincular el entorno virtual con la carpeta del projecto ya creado con el comando `setvirtualenvproject`.*

Creamos un nuevo entorno virtual en el `WORKON_HOME` definido el archivo `~/.bashrc` haciendo

```console
mkvirtualenv [-a project_path] [-i package] [-r requirements_file] [virtualenv options] ENVNAME
```

Por ejemplo, para crear un nuevo entorno virtual llamado `my_cookie` en el directorio de mis entornos virtuales `~/.virtualenvs/` utilizando mi Python 3.6.4 escribo:

```console
mkvirtualenv -p python3.6 my_cookie
```

### 2. Con el entorno activo, ejecutamos cookiecutter contra el repositorio de cookiecutter-django.

Nos dirigimos al directorio donde queremos crear el projecto, en mi caso sera dentro del PROJECT_HOME, `~/project/`, y ejecutamos

```console
cookiecutter https://github.com/pydanny/cookiecutter-django
```

Contestaremos a las preguntas (es bueno ir anotando las respuestas, asi nos acordamos las opciones que hemos definido, ya que son muchas). Por ejemplo, en nuestro caso:

```console
(my_cookie) user@laptop:~/djprojects$ cookiecutter https://github.com/pydanny/cookiecutter-django
project_name [My Awesome Project]: My Cookiecutter Project
project_slug [my_cookiecutter_project]: my_cookie           
description [Behold My Awesome Project!]: Cookiecutter Django basico de prueba
author_name [Daniel Roy Greenfeld]: Cristian Azulay
domain_name [example.com]: mycookie.com
email [cristian-javier-azulay@example.com]: cris@azulay.com
version [0.1.0]: 
Select open_source_license:
1 - MIT
2 - BSD
3 - GPLv3
4 - Apache Software License 2.0
5 - Not open source
Choose from 1, 2, 3, 4, 5 [1]: 1
timezone [UTC]: America/Argentina/Buenos_Aires
windows [n]: n
use_pycharm [n]: n
use_docker [n]: n
Select postgresql_version:
1 - 10.4
2 - 10.3
3 - 10.2
4 - 10.1
5 - 9.6
6 - 9.5
7 - 9.4
8 - 9.3
Choose from 1, 2, 3, 4, 5, 6, 7, 8 [1]: 5
Select js_task_runner:
1 - None
2 - Gulp
Choose from 1, 2 [1]: 1
custom_bootstrap_compilation [n]: n
use_compressor [n]: n
use_celery [n]: n
use_mailhog [n]: n
use_sentry [n]: n
use_whitenoise [n]: n
use_heroku [n]: s
use_travisci [n]: n
keep_local_envs_in_vcs [y]: y
debug [n]: n
 [SUCCESS]: Project initialized, keep up the good work!
```

### 3. Vinculamos el venv actual con el projecto e instalamos los requerimientos para desarrollo local

Luego que finalizamos, exitosamente, la parte de las preguntas y con el projecto de cookiecutter-django ya creado, vamos a vincular el entorno virtual con el directorio de trabajo con virtualenvwrapper, cosa que cuando iniciemos el venv en un futuro, nos dirija directamente a esa carpeta automaticamente, entre otros beneficios.

En este punto vamos a estar con el entorno virtual activo, si nos paramos dentro de la carpeta del proyecto y ejecutamos el comando `setvirtualenvproject` sin argumentos, se supone el directorio actual y el venv activo, asi que podemos hacer directamente 

```console
setvirtualenvproject
```

Luego, siempre dentro de nuestro directorio de trabajo y con nuestro  activo, instalaremos los requerimientos para el desarrollo local haciendo

```console
pip install -r requirements/local.txt
```

### 4. Creamos y configuramos la base de datos en PostgreSQL para usarla con Django.

Nos conectamos a psql con usuario `postgres` o el superusuario que tengamos definido y creamos la nueva base de datos

```sql
-- Crea la base de datos que has ingresado como project_slug en la etapa de configuración.
-- La babosa (slug) de su proyecto sin guiones ni espacios. Se usa para nombrar su repositorio y
-- en otros lugares donde se necesita una versión importable de Python del nombre de su proyecto.
-- Si le indicamos my_project, entonces la base de datos la indicamos como my_project
CREATE DATABASE my_cookie;

-- Crear usuario para la base de datos
CREATE ROLE my_cookie_user 
WITH LOGIN ENCRYPTED PASSWORD 'password' 
CREATEDB;
-- Edit September 25/2015 : 
-- For security Only set CREATEDB permission
-- Which is required for the Django tests

-- Modificamos algunos parámetros de conexión para el usuario que acabamos de crear. 
-- Esto acelerará las operaciones de la base de datos, por lo que no es necesario consultar
-- los valores correctos y configurarlos cada vez que se establece una conexión.

-- Configuramos la codificación predeterminada para UTF-8, que Django espera. También configuramos
-- el esquema de aislamiento de transacción predeterminado como "read committed", que bloquea las 
-- lecturas de transacciones no confirmadas.
-- Por último, configuramos la zona horaria. Por defecto, nuestros proyectos de Django se configurarán
-- para usar UTC:
ALTER ROLE my_user SET client_encoding TO 'utf8';
ALTER ROLE my_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE my_user SET timezone TO 'localtime';

-- Otorgar todos los provilegios al usuario para la base de datos
GRANT ALL PRIVILEGES ON DATABASE my_cookie TO my_user;
```
En este punto, deberíamos tener una base de datos vacía trabajando con el usuario especificado.

Ahora tenemos que decirle a Django la información de mi base de datos. Para hacerlo, debe definir una variable de entorno DATABASE_URL utilizando la convención de Django:

Si estás en un entorno Unix. El siguiente comando hace el trabajo, dentro del entorno virtual:

```console
export DATABASE_URL=postgres://my_cookie_user:password@localhost:5432/my_cookie
```

### 5. Corregimos error de Celery (si especificamos en las opciones que use Celery).

Al momento de ejecutar este tutorial, si en las especificaciones del projecto, especificamos `use_celery [n]: n`, pero en casos que especifiquemos que si, cuando queramos migrar nos va a salir un [error](https://github.com/pydanny/cookiecutter-django/issues/1741) que tenemos solucionar, cambiando la configuración en 'settings/base.py':

```python
CELERY_BROKER_URL = env('CELERY_BROKER_URL') 
```

Por esto

```python
CELERY_BROKER_URL = env('CELERY_BROKER_URL', default="redis://localhost:6379")
```

Guardamos y eso es todo.

### 6. Aplicamos las migraciones.

Después de esto, debería poder ejecutar las migraciones de Django sin ningún problema:

```console
python manage.py migrate
```

Si todo sale bien, deberiamos ver aplicarse las migraciones. Solo queda ejecutar

```console
python manage.py runserver
```

Y... voila!!!

### 7. Activar/Desactivar las variables de entorno de Django con virtualenvwrapper

Puede pasar que, al salir del entorno y volverlo a activar, la variable de  la base de datos que exportamos en el paso 4 no quede almacenada en la variable de entorno del venv. No daremos cuenta si al ejecutar `python manage.py runserver` nos da un error de que falta el password y blablabla.

¿Cómo lo solucionamos? Activando las variables de entorno para nuestro projecto en el script _**bin/postactivate**_ dentro del entorno virtual

```console
### ~/.virtualenvs/mycookie/bin/postactivate
##
#!/bin/bash
# This hook is sourced after this virtualenv is activated.

# Seteo la variable DATABASE_URL de postgresql para desarrollo local
export DATABASE_URL=postgres://user:password@localhost:5432/db_name
```

Luego, en _**bin/postdeactivate**_ desactivo las variables de entorno

```console
#!/bin/bash
# This hook is sourced after this virtualenv is deactivated.

unset DATABASE_URL
```
