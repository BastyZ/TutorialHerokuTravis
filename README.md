# Agregar CI/CD con Heroku y Travis a un proyecto de Django existente

## Heroku

### Requisitos
El proyecto de Django debe estár en git

### Paso 1
Primero se debe instalar Heroku CLI(command line interface)
Instaladores de Windows y MacOS se encuentran en esta página https://devcenter.heroku.com/articles/getting-started-with-python#set-up

Por otro lado se puede instalar en Mac Os por medio de homebrew
``` shell
$ brew install heroku/brew/heroku
```

Para Windows con Chocolatey https://chocolatey.org/packages/heroku-cli
``` shell
choco install heroku-cli
```

Para Linux
```shell
$ sudo snap install heroku --classic
```
Por último en la consola se debe ejecutar el comando de login 
```shell
heroku login
```
Lo que abrirá el navegador con la pantalla de login

### Paso 2
Asegurar que la raíz del proyecto de Django tenga los siguientes archivos:
- runtime.txt con la versión de Python a usar
``` python
python-3.7.5
```
- requirements.txt con al menos:
```
django
Django-admin
djangorestframework
Gunicorn
psycopg2==2.7.5
Django-heroku
```
- Procfile con 
```web: gunicorn <NOMBRE_PROYECTO>.wsgi --log-file -```
- Procfile.windows con 
```web: python manage.py runserver 0.0.0.0:5000```


### Paso 3
Editar `<NOMBRE_PROYECTO>/<NOMBRE_PROYECTO>/settings.py`, agregando lo siguiente:
- Al comienzo del archivo
``` python
import django_heroku
```
- Al final del archivo 
``` python
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
django_heroku.settings(locals(), test_runner=FALSE)
```

`test_runner` queda como falso dado que dejaremos los test a Travis


### Paso 4
Luego, crear una aplicación en Heroku con el siguiente comando:
```shell
heroku create
```
Esto también creará un proceso “git remote” asociado al repositorio local

Para hacer un despliegue del código se utiliza
```shell
$ git push heroku master
```

La aplicación ha sido desplegada, se asegura que al menos una instancia de la app está corriendo mediante
``` shell
Heroku ps:scale web=1
```

Para visitar la aplicación una vez desplegada se puede utilizar el siguiente comando
```shell
heroku open
```

## Travis

### Prerequisitos
- Tener el repositorio en github
- Tener credenciales de autor del repositorio
- ruby 2.4.0 o superior

### Paso 1
Instalar Travis CLI desde https://github.com/travis-ci/travis.rb#installation


Ir a la página de [Travis](travis-ci.com) e iniciar sesión con github o bitbucket , luego aceptar la autorización de github o bitbucket

Activar el repositorio en la sección **settings** que se puede acceder al hacer click en la foto del perfil.

### Paso 2
Agregar `.travis.yml` en la raíz del proyecto con la siguiente información:
``` yaml
language: python
python:
“3.7.5”
install:
pip install -r requirements.txt
script:
python manage.py makemigrations
python manage.py migrate
python manage.py test
deploy:
    provider: heroku
        api_key:
            secure: <el resultado del comando: travis encrypt $(heroku auth:token)>
        app: <Nombre de la App en Heroku>
        on:
            repo: <Usuario de git>/<Nombre del repo>
```
### Paso 3
Desde la interfaz web de heroku se debe realizar el link con el repositorio de github, esto se hace desde la pestaña de “Deploy”, indicando conexión github, indicando el nombre del repositorio.Una vez hecho esto, en github se se observa en “settings” -> ”webhooks” que el repositorio se ha enlazado a heroku.



https://medium.com/@felipeluizsoares/automatically-deploy-with-travis-ci-and-heroku-ddba1361647f
