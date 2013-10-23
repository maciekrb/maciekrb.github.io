---
layout: post
title: "HowTo: Wordpress en Heroku"
description: "Flujo de trabajo con Git, Wordpres y Heroku"
category: 'coding'
tags: ['wordpress', 'heroku', 'git', 'PHP']
---
{% include JB/setup %}

[Heroku](http://www.heroku.com) es un servicio *PaaS* (Platform as a Service) que oficialmente soporta aplicaciones construidas en `Ruby`, `Python`, `node.js`, `Java`, `Clojure` y `Scala`. Pero prácticamente es posible correr cualquier cosa, incluyendo desde luego `PHP`. La filosofía detrás de plataformas como *Heroku* es muy similar a *Google App Engine*: olvidarse de los servidores y sus quehaceres para concentrarse en el desarrollo de aplicaciones. [Mas sobre PaaS]() 

Crear una aplicación en `PHP` es bastante sencillo, con solamente hacer `git push` de un `index.php` con cualquier código ejecutable de PHP y listo. Un `phpinfo()` indica la siguiente configuración: 

## Modulos de Apache
core mod_authn_file mod_authn_default mod_authz_host mod_authz_groupfile mod_authz_user mod_authz_default mod_auth_basic mod_include mod_filter mod_log_config mod_env mod_setenvif mod_version prefork http_core mod_mime mod_status mod_autoindex mod_asis mod_cgi mod_negotiation mod_dir mod_actions mod_userdir mod_alias mod_rewrite mod_so mod_php5

En resumen, basta con crear un script sencillo 
El proceso de deployment es sencillo para quienes tienen alguna experiencia usando `asta con crear una instancia o `Dyno`, inicializar un repo de `Git` y hacer push, pero hay aspectos que no resultan obvios al principio. Tantas tecnologías juntas, unas interactuando con las otras le nublan la mente a cualquiera y se termina con un desorden atroz antes de comenzar a escribir cualquier cosa que medio funcione. 

Lo que tal vez no es trivial, es comprender su modelo de funcionamiento. En resumen, es como correr un web server en un proceso de unix:  es un proceso con recuros limitados, puede ser replicado, corre en un entorno específico previamente configurado y es efímero por naturaleza. Casi como si hubiera si de antemano se hubiera pensado en poner todas las buenas prácticas juntas, la forma en que se hace deploy de las aplicaciones, es mediante un repo de `Git`.



Wordpress, Drupal y Joomla, son plataformas extremadamente populares para el desarrollo de sitios web de todo tipo, El objetivo de este post es compartir mi experiencia configurando y corriendo Wordpress sobre Heroku y ayudar a entender algunos flujos  convenientes en la administración del desarrollo de aplicaciones. El proceso debería ser relativamente fácil de replicar con Drupal o Joomla. Con el fin de exprimir al máximo las instancias (dynos de Heroku) que ejecutarían estas apps, usaré NginX en lugar de Apache web server.

Para correr esta configuración, es necesario crear el entorno correcto de ejecución apropiado, en términos de Heroku un *buildpack*. El *buildpack* debe incluír los binarios de `PHP` y Apache o NginX compilados para Heroku (normalmente instancias de un Ubuntu en los huesos). 


El proceso para realizar deploy en heroku con esta estructura es siguiente: 


### 1. Instalar el Heroku Toolbelt

Heroku Toolbelt es una serie de herramientas de linea de comando (CLI) que permiten administrar aplicaciones que corren sobre la infraestructura Heroku, por ejemplo, la siguiente linea de comando crea una instancia de de una aplicación Heroku llamada MiSuperApp con un entorno de ejecución proporcionado por https://github.com/rtvc/heroku-buildpack-wordpress.git

```sh
$ heroku create \ 
    --stack cedar \ 
    --buildack https://github.com/rtvc/heroku-buildpack-wordpress.git \ 
    MiSuperApp
```


La siguiente linea permite visualizar los procesos activos en una instancia de heroku:

```sh
$ heroku ps
```


Esta, escalar rápidamente a 3 instancias para absorber un pico de tráfico:

```sh
$ heroku ps:scale web=3
```


Esta, visualizar los registros (logs) de la aplicación (stream continuo con --tail):

```sh
$ heroku logs --tail
```

El toolbelt de heroku puede descargarse para cada plataforma en [el sitio oficial](https://toolbelt.heroku.com)


Una vez instalado el toolbelt, se puede verificar que la instalación fue exitosa ejecutando:

```sh
$ heroku --version
heroku-toolbelt/2.37.2 (x86_64-darwin10.8.0) ruby/1.9.3
```

---


### 2. Fork de este repositorio y creación de ramas (branches)


El paso siguiente consiste en realizar el fork de este repositorio. Este fork contendrá las partes del proyecto Wordpres que se subirán a la instancia de Heroku.

Git es suficientemente flexible para permitir distintos flujos de trabajo, bien sea un único desarrollador o un grupo de colaboradores. Una buena práctica en el manejo de releases, consiste en mantener una rama (branch) `production` dedicada a contener los commits de las versiones estables, en contraposición a la gran cantidad de commits que pueden existir en el branch `master`. Para esto basta hacer:

```sh
$ git checkout -B production
```

Con los cambios separados, es posible mantener una historia limpia en la rama de producción y hacer el respectivo deploy:

```sh
$ git push heroku production:master
```

#### Quiere esto decir que no se sube cambios directamente desde master a heroku ?
Si. La idea es mantener la historia del branch de producción tan limpia como sea posible y así sacar el máximo provecho del sistema de releases de heroku:

```sh
$ heroku releases
=== MiSuperApp Releases
v38  Deploy e4d5a22                   me@example.com  2013/04/17 17:35:28 
v37  Deploy ec0f441                   me@example.com  2013/03/17 17:11:45  
v36  Deploy a3a37ee                   me@example.com  2013/02/17 17:06:52  
v35  Deploy d1f9d42                   me@example.com  2013/01/17 16:48:56 
v34  Deploy b40d883                   me@example.com  2012/12/17 16:14:39 
v33  Deploy 9a77795                   me@example.com  2012/11/17 13:42:45 

$ heroku releases:rollback ec0f441 # Volver a la anterior version estable de la app 
```

#### Y como hago pruebas ?
Este esquema está pensado precisamente en resolver el problema de replicación de los entornos de pruebas. La idea es que cada desarrollador del proyecto cree su propia instancia de la aplicación en heroku usando el mismo buildpack que se utiliza en producción, y allí puede hacer tantos push del branch master como sea necesario. Estas instancias de prueba son públicas, de forma que también funcionan a manera de pasarela (staging server). 

---

## 3. Instalación Wordpress local 
Dado que ninguna instancia virtual es aún tan efectiva como un entorno local de desarrollo, haremos una instalación local de Wordpres y así sea posible trabajar en la maquina local. Para esto es necesario entender la estructura siguiente de carpetas:

```
└──.gitignore             # Lista de archivos ignorados por git
└── config                # Archivos de configuración
    ├── public            # Document Root (público) de nuestra aplicación
    │   └── wp-content    # Themes & plugins
    │       ├── plugins
    │       └── themes
    └── vendor            # Archivos de configuración de las vendored apps
        ├── nginx
        │   └── conf      # nginx.conf + wordpress.conf.erb
        └── php           # php.ini
            └── etc       # php-fpm.conf
```

Dado que el buildpack se encarga de instalar Wordpress en el entrorno virtual que correrá en Heroku,  este repositorio *NO hará tracking* de los archivos base de la instalación Wordpress. En su lugar, solamente contendrá los archivos que es necesario sobreescribir sobre una instalación normal de Wordpress. La ventajas mas sobresalientes de este esquema son las siguientes: 

  - Favorecer el desarrollo de componentes compatibles (plugins, temas), en lugar de modificar el código base de Wordpress.   
  - Permitir pruebas en una versión diferente de Wordpress con un esfuerzo mínimo. Basta con descomprimir la distribución de Wordpress (sobreescribiendo la anterior) sobre una versión limpia del branch master y haciendo: 

```sh
# Reinicia los archivos contenidos en el repo a la versión del último commit (eliminando cambios)
$ git reset --hard 
```

El objetivo de esto último es devolver al estado del último commit los archivos que hayan sido sobre escritos al descomprimir la versión de Wordpress (normalmente wp-config.php).

---

## 4. Instalación de plugins, temas y desarrollo 
Para implementar algún tipo de funcionalidad, basta con agregar los temas y plugins respectivamente en las carpetas `themes/` y `plugins/`  dentro de `config/public`. 


