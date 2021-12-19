# DJANGO

## Requisitos 📋

Ubuntu 20.04 (Vagrant ubuntu/focal64)

## Configuración 🔧

Antes de clonar el repositorio tenemos que instalar git
```
sudo apt update
sudo apt install git
```

Lo primero que debe hacer es clonar el repositorio:
```
git clone https://github.com/oscarabril/oscarabril
cd oscarabril
```
Cree un entorno virtual para instalar dependencias, actívelo e instalar django:
```
sudo apt install python3.8-venv
python3 -m venv djenv
source djenv/bin/activate
(djenv) $ pip install django
```

Añadimos la libreria  django-environ
```
(env) $ pip install django-environ 
```

Crea el archivo .env con las variables de entorno, en este caso SECRET_KEY y también DEBUG.(ejemplo env.example)
El archivo debe de quedar asi:
```
SECRET_KEY=key
DEBUG=False
```
### MYSQL
Django utiliza la BD SQLite, pero necesitamos una BD apta para producción como MySQL, para eso instalaremos los siguientes paquetes:
```
sudo apt install mysql-server
sudo apt install libmysqlclient-dev gcc python3-dev python3-mysqldb
(djenv) $ pip install mysqlclient
```
Una vez instalados los paquetes crearemos un usario y una base de datos para django
```
sudo mysql
create user admin@localhost identified by "admin123";
create database polls;
grant all privileges on *.* to 'admin'@'localhost';
exit;
```
Ajusta la variable de la BD de esta forma en el archivo creado anteriormente .env:
```
DATABASE_URL='mysql://admin:admin123@localhost:3306/polls'
```
### Servidor de aplicaciones uWSGI
En realidad su instalación y puesta en marcha es muy sencilla:
```
(djenv) $ pip install uwsgi
```
Recopilar los archivos estáticos con la instrucción:
```
(env) $ ./manage.py collectstatic
```
Instalación de nginx para uWSHI
```
sudo apt install nginx
```
Una vez configurado configurare el nginx 
```
cd /etc/nginx/sites-available/
sudo nano mysite_nginx.conf
```
Dentro del archivo mysite_nginx.conf copiaremos lo siguiente, pero modificando las rutas de las "locations" a la carpeta donde tenemos el proyecto descargado (en mi caso en /home/vagrant/proyectodjango/oscarabril)
Tenemos que modificar todas las rutas menos la ultima de uwsgi_params que la tenemos en /etc/ngix/uwsgi_params
```
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /home/vagrant/proyectodjango/oscarabril/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /home/vagrant/proyectodjango/oscarabril/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /etc/nginx/uwsgi_params; # the uwsgi_params file you installed
    }
}
```
Crear un enlace simbólico a este archivo desde /etc/nginx/sites-enabled para que nginx pueda verlo:
```
sudo ln -s /etc/nginx/sites-available/mysite_nginx.conf /etc/nginx/sites-enabled/
```
Una vez hecho esto reiniciaremos el servicio nginx 


Volveremos al directorio donde tenemos el proyecto y ejecutaremos uwsgi
```
(djenv) $ uwsgi --socket :8001 --module mysite.wsgi
```
### POSIBLES ERRORES
Si al entrar al http://localhost:8000/admin sale un error al iniciar session
usuario admin:admin

Primero migraremos **python manage.py migrate**,  despues volveremos a importar polls 
**python manage.py makemigrations polls** y despues crearemos un susperusuario *python manage.py createsuperuser*
