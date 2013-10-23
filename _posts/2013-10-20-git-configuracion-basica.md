---
layout: post
title: "HowTo: Configuración básica de Git"
description: "Pasos básicos para la configuración de Git como sistema de control de versiones"
category: 'coding'
tags: ['git', 'ssh']
---
{% include JB/setup %}

>
> Un par de commits de casi 7 millones de líneas de código dieron inicio a Git, el 
> ''manejador de contenidos del infierno ''
>

```sh
 commit e83c5163316f89bfbde7d9ab23ca2e25604af29 
 Author: Linus Torvalds <torvalds@ppc970.osdl.org> 
 Date: Thu Apr 7 15:13:13 2005 -0700

 Initial revision of "git", the information manager from hell

 commit 1da177e4c3f41524e886b7f1b8a0c1fc7321cac2 
 Author: Linus Torvalds <torvalds@ppc970.osdl.org> 
 Date: Sat Apr 16 15:20:36 2005 -0700

 Linux-2.6.12-rc2

 Initial git repository build. I'm not bothering with the full history, even though we have 
 it. We can create a separate "historical" git archive of that later if we want to, and 
 in the meantime it's about 3.2GB when imported into git - space that would just make the 
 early git days unnecessarily complicated, when we dont have a lot of good infrastructure 
 for it.
 
 Let it rip!
```

Después de la [famosa controversia BitKeeper][30], **Linus Torvalds** decide crear su propio
sistema de control de versiones, el resultado de ese ''stretch'' es [Git][15].

Entre los aspectos más importantes en su diseño están: 

####__Gratuito__
Sip, cero pesos, cero centavos, otro ejemplo más de como el trabajo colaborativo le lleva ventaja a las 
licencias y patentes. 

####__Facilitar el desarrollo distribuido__ 
Hacer eficiente el desarrollo en paralelo, esto es, distintos equipos con repositorios independientes 
pueden trabajar en sus propias versiones sin un repositorio central. Cuando el momento sea apropiado, 
sincronizar con estos **remotos**, comparar y decidir que `commits` se quiere usar.

####__Rapidez y eficiencia__
Cortos tiempos de transferencia, operaciones mucho mas rápidas y eficientes. Contar con un repositorio 
completo en todo momento para no depender de un servidor remoto para las operaciones cotidianas.

####__Diseño__
Crear un diseño limpio y eficiente que aproveche al máximo todas las fortalezas de el sistema de 
archivos de Unix/Linux.

Como resultado de esto, [Git][15] no sólo es muy apropiado para escenarios en donde existen cientos
de desarrolladores, trabajando en sus propios __repos centrales__ o __"Hubs"__. Funciona perfectamente 
como sistema local de control de cambios, sin necesidad de sincronizar nada en absoluto con ningún 
repositorio remoto si esto no es necesario. La __base de datos__ del repositorio estará contenida 
en una sola carpeta dentro del proyecto: `.git`, usando una de las herramientas más poderosas del
sistema operativo, su sistema de archivos.

>
> Yo no utilizo sistema de control de versiones porque casi siempre trabajo solo.
> -- programador anónimo
>

Este argumento se oye mas a menudo de lo que se debería. Un sistema de control de versiones no solamente
es útil en un entorno de trabajo colaborativo, diría que [Git][15] está precisamente diseñado para 
hacer la vida de cualquier programador __"solo"__ mas sencilla. 

Comenzamos a escribir una solución sencilla para un problema. Conforme avanzamos, se nos ocurren
casos extremos o mejores formas de hacer alguna parte del algoritmo, pero queremos llegar a una 
versión inicial que funcione. En el mundo __no-repo__ se suele sacar copias de archivos o carpetas 
completas, de cierta forma estamos versionando intuitivamente pero con las herramientas de la edad de 
piedra. Si la experimentación toma un par de __"versiones"__, el resultado será casi siempre un montón 
de carpetas con una nomenclatura confusa y un desarrollador confundido perdiendo tiempo buscando 
la versión __X__ de la función __Y__. 

En el mundo de los sistemas de control de versiones centralizados, donde cada `commit` hace parte 
de un repo central público, este escenario es hasta cierto punto comprensible. Después de todo, nadie 
quiere incluir versiones __"a medias"__ de un componente. 

Sin embargo, con [Git][15] nada es realmente público hasta que no se hace un `git push`. Un 
 `commit` en [Git][15] es como una fotografía del sistema de archivos del proyecto en un momento 
particular del tiempo, pero estas fotografías tienen nombre, descripción, fecha, autor y las 
diferencias entre `commits` pueden verse fácilmente. Cuando todo está listo, es posible agregar 
todos los `commits` en uno solo y compartir con el mundo si es el caso.

Este es la primera parte de una serie de **HowTos** que muestran paso a paso como incluir 
[Git][15] en el flujo diario de trabajo.  

Configuración básica
--------------------

El primer paso para utilizar `git` consiste en configurar la información del autor, de manera que 
cada `commit` incluya la información completa de quien realiza los cambios. La configuración mínima
incluye un nombre y un e-mail

```sh
~ $ git config --global user.name "Maciek Ruckgaber"
~ $ git config --global user.emall maciek@example.com
```

Los anteriores comandos crearán el archivo `.gitconfig` en el directorio raíz del usuario, y 
serán utilizados a manera de datos predeterminados para todos los proyectos. Esto a razón de
que estamos usando el flag `--global`. De no usar el flag, esta información se guardará en el
archivo `.git/config` dentro del proyecto actual y será únicamente relevante a ese proyecto.

Nuestro archivo de configuración debe verse mas o menos así:

```ini

[user]
	name = Maciek Ruckgaber
	email = maciek@example.com
```

Otras opciones que resulta útil personalizar son las siguientes. Éstas bien pueden agregarse usando
la sintaxis utilizada para agregar el nombre y e-mail, o editando directamente el archivo.

```ini

# Core hace referencia a la configuración general de Git
# - editor: Editor utilizado al hacer commit, nano es una opción popular si no eres 
#           amigo de vim
# - excludesfile: Especifica un archivo que define patrones de carpetas u otros archivos 
#           que normalmente se quiere ignorar, como .zip, .tar, .project, etc
[core]
  editor = vim 
  excludesfile = /Users/maciekrb/.gitignore 

# Las siguientes son simplemente preferencias de color en la salida de  
# git status, git diff y git branch
[color]
  ui = true
  branch = auto
  status = auto
  diff = auto
[color "diff"]
  meta = yellow
  frag = cyan
  old = red
  new = green
[color "branch"]
  current = yellow reverse
  local = yellow
  remote = green

```
 
Agregando contenido al repo
---------------------------

Hecho lo anterior, estamos listos para inicializar tantos repos que queramos. Usaremos una 
estructura sencilla para ejemplificar los pasos siguientes.

```sh
~ $ tree
.
gitTutorial
  ├── assets
  │   ├── css
  │   │   └── style.css
  │   └── images
  │       └── logo.png
  ├── config
  │   └── config.inc
  ├── vendor
  │   ├── libABC
  │   └── zend-framework
  └── public
      └── index.php
```

Inicializar Git en un proyecto existente es muy sencillo

```sh
~ $ git init
Initialized empty Git repository in /Users/maciekrb/Devel/gitTutorial/.git/
```

Lo anterior simplemente crea una carpeta llamada `.git` dentro del directorio del proyecto. Esta carpeta
tendrá internamente la __base de datos__ necesaria para hacer seguimiento de los archivos del proyecto. 

Esta base de datos consiste básicamente de unos cuantos tipos de objetos: 
####Blobs
Contienen los datos de los archivos

####Trees
Representan las rutas de un sistema de archivos y parte de su meta-información 

####Commits
Meta-información de los cambios a los archivos, toda la información de las __"fotos"__ del sistema
de archivos del proyecto.

####Tags 
Que no son otra cosa que nombres del tipo `V2.0-RC1` para algo que de otra forma podría llamarse 
`606470d540e57844c522210ee013d0a2d382d784`

####Pack Files
Los `Blobs`, `Trees`, `Commits` y `Tags` se acumulan con el tiempo y a fin de usar eficientemente 
el espacio en disco y ancho de banda, se comprimen en `Pack Files`.

####Index
Finalmente, el `Indice`, corresponde a un archivo binario temporal que describe la estructura del 
repositorio en un momento particular y guarda la información del estado actual del repositorio. El
**Indice** registra los cambios hasta que estos sean incluidos en un `Commit`.

Regresando a la práctica, después de inicializar el repositorio, podemos revisar fácilmente el 
estado del `Indice` ejecutando :

```sh
~ $ git status
# On branch master
#
# Initial commit
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#   assets/
#   config/
#   vendor/
#   public/
nothing added to commit but untracked files present (use "git add" to track)

```

[Git][15] Normalmente nos dará pistas de lo que ocurre, en este caso nos indica que estamos
en el `branch master`, que este es un `commit Inicial`, que hay una lista de `Untracked files`
entre los que están las carpetas `assets`, `config`, `vendor` y `public`, y que no hemos 
agregado nada al repositorio, y para hacerlo,  que debemos usar `git add`. 

El concepto clave para entender [Git][15] y des-aprender lo que sepamos de otros sistemas
de _control de versiones_, es que [Git][15] es más un **sistema de seguimiento de contenido**, 
que un sistema de seguimiento de versiones. Esto quiere decir por ejemplo, que si dos archivos
tienen exactamente el mismo contenido (evaluado usando un [SHA1][31]), [Git][15] creará un
único objeto de este contenido en su base de datos, identificado con el [SHA1][31] correspondiente, 
el nombre del archivo en el sistema, no tiene incidencia alguna en el cálculo del hash. La estructura
de archivos de nuestro proyecto, no tiene incidencia alguna en la forma como [Git][15] almacenará
los objetos de contenido. La información del nombre de archivo, se guarda en otro tipo de objeto, 
el `Tree`. 

Adicionalmente, [Git][15] no guarda las diferencias entre un archivo y otro, guarda cada versión
del archivo, hábilmente usando [hard links][32] cuando no existen diferencias entre la versión 
de un `commit` y la versión de otro. Dicho esto, es evidente que para regresar a una versión
específica de un contenido, [Git][15] no necesita procesar cientos de [diffs][33] para obtener la
versión, simplemente debe resolver un apuntador.

Es así que contrario a otros sistemas de control de versiones, `add` es un comando que usaremos con
frecuencia, dado que nos permite **"marcar"** un archivo para ser incluido en el siguiente `commit`.

```sh
~ $ git add assets
~ $ git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#new file:   assets/css/style.css
#new file:   assets/images/logo.png
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#  config/
#  vendor/
#  public/
``` 

Al ejecutar `git add assets`, [Git][15] marcó los archivos contenidos en la carpeta `assets` para
ser incluidos en la próxima foto `commit`, esto nos permite fácilmente controlar los cambios que 
queremos incluir en el `commit` siguiente. [Git][15] nos sugiere, que en caso de querer des-marcar
algún archivo, para que este no sea incluído en el próximo `commit`, podemos usar `git rm --cached`.

Podríamos usar `git add`  carpeta por carpeta o archivo por archivo es útil, pero no suena práctico
para agregar todo un proyecto. Para agregar el contenido de todos los archivos del proyecto, podemos
usar `git add .` pero esto seguramente va a incluir archivos que no queremos en el repo. 

### .gitignore
Cada tipo de proyecto o entorno de desarrollo tiene ciertos archivos que no son bienvenidos en un
repo. Cosas como `.project`, `index.php~`, `index.php.swp`, `.venv`, archivos `.zip`, `.tar.gz`, 
son ejemplos de patrones que queremos no incluir. Este tipo de casos comunes a la mayoría de
proyectos en los que trabajaremos, son buenos candidatos para ser incluidos en el archivo 
`.gitignore` que definimos en el parámetro `excludesfile = /Users/maciekrb/.gitignore` en nuestro
`.gitconfig`, así no tendremos que agregarlos en el `.gitignore` específico del proyecto cada
vez que inicialicemos un nuevo repo. Las reglas configuradas en el `.gitignore` del proyecto, tiene
prioridad respecto a las del archivo definido por `excludesfile`. 

```sh
# Archivo /Users/maciekrb/.gitignore

.DS_Store
.env
.venv
.ropeproject
*.svn
*.pyc
*.swp
*.swo
*.sqlite
*.sql3
*.mo
*.php~
*.zip
```

```sh
# Archivo /Users/maciekrb/Devel/gitTutorial/.gitignore
vendor/
```

El archivo `.gitignore` del proyecto, puede tener cosas específicas a este, tales como por
ejemplo `vendor/` en este caso. Normalmente no queremos incluir el código fuente de las librerías
dentro de nuestro código. Una mucho mejor estrategia consiste en usar [composer][34] para `PHP`, 
[pip][35] para `Python` o [npm][36] para `node.js` o [Bower][37] para `Javascript`, e incluir
el archivo correspondiente de configuración de dependencias en el repo, e ignorar el directorio.

Por ejemplo, con [composer][34], creamos un archivo composer.json, que agregamos al repo.

```  json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.zendframework.com/"
    }
  ],
  "require": {
    "zendframework/zend-config": "2.0.*",
    "zendframework/zend-http": "2.0.*"
  }
}
```

Una vez ejecutado `composer.phar install` dentro del proyecto, [composer][34] creará un archivo 
adicional `composer.lock`, este archivo contiene las versiones exactas de las dependencias del 
proyecto, permitiendo fácilmente instalar todas las dependencias sin tenerlas dentro del repo. 
El mismo efecto se logra guardando en el repo el resultante de ejecutar `pip freeze > dependencies.txt`

Hecho esto, podemos terminar de agregar los archivos restantes

```sh
~ $ git add .
~ $ git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
# new file:   .gitignore
# new file:   assets/css/style.css
# new file:   assets/images/logo.png
# new file:   composer.json
# new file:   composer.lock
# new file:   config/config.inc
# new file:   public/index.php
#

~ $ git commit
[master (root-commit) b249ede] Linea con descripción corta, hasta 50 chars
7 files changed, 150 insertions(+)
create mode 100644 .gitignore
create mode 100644 assets/css/style.css
create mode 100644 assets/images/logo.png
create mode 100644 composer.json
create mode 100644 composer.lock
create mode 100644 config/config.inc
create mode 100644 public/index.php
```

Al ejecutar el `commit` se cargará el editor configurado en nuestro archivo `.gitconfig`, incluyendo
algunos detalles de lo que incluye el `commit` que estamos haciendo. Simplemente es necesario escribir 
los cambios que estamos incluyendo. La siguiente convención sirve bastante bien:

  - _primera linea_: descripción de cambios como si se tratase del asunto de un e-mail (max 50 
     caracteres)
  - _bloque a continuación_: Después de un salto de linea que ayuda a la legibilidad, se incluye una 
    descripción detallada que puede tomar las lineas necesarias. Se obtiene mejores resultados
    usando una longitud de linea de máximo 72 columnas.

```sh
Linea con descripción corta, hasta 50 chars

Esta es una descripción mas larga y detallada. Ayuda limitar el número
de columnas a 72. Los mensajes deben escribirse en presente, "Resuelve
Issue XYZ", en lugar de "Resuelto Issue XYZ" o "Issue XYS resuelto",
esto coincide con el formato de mensaje de git merge y git revert.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
# new file:   .gitignore
# new file:   assets/css/style.css
# new file:   assets/images/logo.png
# new file:   composer.json
# new file:   composer.lock
# new file:   config/config.inc
# new file:   public/index.php
#

```

Al guardar y salir del editor set **"tomará la foto"** del `Indice`, y los cambios quedarán guardados 
en el repositorio. Esto corresponde a un **conjunto de cambios atómico** para convertir una versión 
anterior en la actual, en este caso puntual, la inclusión inicial de los archivos al proyecto. Podemos 
entonces ver los detalles del `commit` que acabamos de hacer.

```sh
~ $ git log
commit b249edeb237663aa183a854e074a320cb5e64afe
Author: Maciek Ruckgaber <maciekrb@gmail.com>
Date:   Wed Oct 23 15:45:46 2013 -0500

    Linea con descripción corta, hasta 50 chars
    
    Esta es una descripción mas larga y detallada. Ayuda limitar el número
    de columnas a 72. Los mensajes deben escribirse en presente, "Resuelve
    Issue XYZ", en lugar de "Resuelto Issue XYZ" o "Issue XYS resuelto",
    esto coincide con el formato de mensaje de git merge y git revert.
```

Ahora podemos comenzar a hacer modificar nuevamente los archivos, y simplemente repetimos el
ciclo `git add`, `git commit`, tantas veces sea conveniente. 

### Remotos

Ya tenemos un repositorio local configurado y andando, sin embargo, el hecho de tener una única copia 
del código fuente es normalmente un riesgo. Si el disco de la maquina falla, o está sufre un cambio
inesperado de propietario, se perdería todo nuestro trabajo, personalmente no encuentro nada mas
tedioso que escribir nuevamente, código que ya escribí una vez.  

La manera más sencilla de compartir código con [Git][15], es hacerlo a través de un servicio que permita
alojar código usando [Git][15]. Entre los servicios más populares se encuentra [Github][25] y 
[BitBucket][26].  Estos servicios harán las veces de **_remotos_** o **_remotes_**, a los que podemos 
empujar el código a fin de sincronizarlo desde otras máquinas o compartirlo con otras personas. 

Los **remotes** o **remotos** no son otra cosa que **_alias_**, o nombres resumidos para direcciones
que sería muy tedioso digitar cada vez en la linea de comando. Para agregar un remoto simplemente
se debe ejecutar `git remote add nombre_remoto git@github.com/usuarioABC/repositorioBCD`. Cuando 
solamente existe un único remoto, normalmente se usa la convención de llamarlo **origin**.  Una vez
creado, las dos sentencias siguientes son equivalentes:

```sh
~ $ git pull nombre_remoto master
~ $ git pull git@github.com/usuarioABC/repositorioBCD master
```

**nombre_remoto** no es otra cosa que un **alias** a la dirección `git@github.com/usuarioABC/repositorioBCD`. Git ni siquiera verifica que esta dirección exista al momento de hacer `git remote add`. Una vez configurado el remoto es posible ejecutar `git push nombre_remoto master` o `git pull nombre_remoto master` para empujar o halar **commits** respectivamente.

Para verificar cuales remotos tenemos configurados podemos usar `git remote -v`. 

Git y SSH
---------

Los remotos alojados en servicios como [Github][25] o [BitBucket][26] se comunican usando [SSH][27], 
un protocolo de red que permite la transmisión segura de datos. [SSH][27] utiliza 
[criptografía asimétrica][28] para autenticar una persona o maquina remota, y así se controla el 
acceso de quienes tienen privilegios de escritura en dado repositorio de [Github][25] o [BitBucket][26]. La lectura de dichos repositorios varía, un repositorio puede ser públicamente accesible (cualquier persona puede hacer clone, fetch) o privada, en cuyo caso la llave criptrográfica es el mecanismo para reconocer si alguien puede o no hacer clone o fetch. 

El proceso para crear una llave privada y su contraparte pública (la que será registrada en el servicio
de alojamiento de código) es muy sencillo: `ssh-keygen -t rsa -C "maciek@example.com"`. El comando
anterior crea una llave de tipo [RSA][29] con un comentario que contiene un e-mail. Al crear la llave
se solicitará una contraseña, ésta contraseña es una medida de seguridad que únicamente permite usar
la llave a quien que conozca la contraseña con la que fue creada. El resultado del proceso resulta
normalmente en dos llaves `id_rsa` y `id_rsa.pub` en el folder `.ssh` el **home** del usuario, 
salvo que se haya definido una ruta diferente al momento de crearlas. 

Una vez creadas, es necesario registrar el contenido de la llave id_rsa.pub en el servicio de alojamiento 
de código, normalmente en opciones de cuenta, en una seccion llamada **SSH Keys**. Es importante copiar
el contenido del archivo `id_rsa.pub` sin incluir espacios o saltos de linea adicionales. Finalmente 
se puede realizar una prueba : 

```sh
~ $ ssh git@github.com
Hi usuarioABC! You've sucessfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

Dado que el servicio ofrece un shell restringido que únicamente permite ejecutar comandos de git, 
podemos entender esto como un mensaje exitoso de conexión usando la llave remotea para usuarioABC. 

Errores comunes
---------------
Uno de los errores mas comunes en la configuración de [Git][15], es crear una llave con un nombre 
que SSH no reconoce no reconoce con su configuración predeterminada. Salvo que se haya definido un 
valor distinto en el archivo .ssh/config, el cliente [SSH][27] **siempre** buscará una llave privada 
llamada `id_rsa` al conectarse con un servicio que realiza autenticación utilizando llaves 
criptográficas. Si la llave tiene un nombre distinto (mi_llave), el servicio remoto probablemente 
retornará un error como `Permission denied (publickey)`. 

Es muy frecuente que la gente pierda sus llaves públicas. Estas llaves, como su nombre lo indica, 
son en efecto públicas y revelarlas no tiene mayor inconveniente, contrario a las llaves privadas. Las
llaves privadas contienen la información para generar la llave pública. En caso de perder una llave 
pública, solo es necesario correr `ssh-keygen -y -f ~/.ssh/id_rsa > .ssh/id_rsa.pub` para obtener la 
llave pública correspondiente. 

Otro asunto frecuente, es confundir llaves públicas. Para identificar una llave pública, se puede obtener
su **fingerprint** con el siguiente comando: 

```sh
~ $ ssh-keygen -l -f ~/.ssh/id_rsa.pub 
2048 2a:d9:96:ef:ba:2a:ba:05:22:6d:26:b1:b2:68:29:b1  maciek@example.com (RSA)
```

Algunos servicios de alojamiento de código siempre muestran el fingerprint de cada llave registrada. Las 
llaves privadas, no tienen fingerprint, dado que de ellas es posible obtener la llave pública.

No sobra decir, que vale la pena guardar la llave privada en alguna parte adicional a la maquina 
(una memoria USB, Dropbox, Google Drive son algunas opciones), de manera que si algo ocurre con
la máquina, podemos recuperar nuestro trabajo en unos pocos minutos.

[1]: http://www.drupal.org
[2]: https://drupal.org/project/features
[3]: https://drupal.org/project/storage_api
[4]: http://drush.org
[5]: http://aws.amazon.com/s3/
[6]: https://drupal.org/node/21947
[7]: https://drupal.org/project/views
[8]: http://pear.drush.org/
[9]: https://drupal.org/node/1022020
[10]: https://github.com/drush-ops/drush
[11]: https://drupal.org/node/670460
[12]: http://dev.mysql.com/doc/refman/5.6/en/binary-log.html
[13]: https://docs.google.com/a/rtvc.gov.co/file/d/0ByO1zwYiBv1ybkNTY0RSeHRmb28/edit?usp=sharing 
[14]: http://en.wikipedia.org/wiki/Scalability
[15]: http://git-scm.com
[16]: http://nvie.com/posts/a-successful-git-branching-model
[17]: http://www.youtube.com/watch?v=LJ31O2Gk6HE
[18]: http://www.youtube.com/watch?v=X_5V9hPbTns
[19]: http://www.youtube.com/watch?v=NPThk0hLyEg
[20]: http://www.youtube.com/watch?v=ETDR1b15Ks0
[21]: http://www.youtube.com/watch?v=P8V9v0C0K0o
[22]: https://github.com/scottjehl/Respond
[23]: https://code.google.com/p/css3-mediaqueries-js/
[24]: http://en.wikipedia.org/wiki/Symbolic_link
[25]: http://github.com
[26]: http://bitbucket.org
[27]: http://en.wikipedia.org/wiki/Secure_Shell
[28]: http://en.wikipedia.org/wiki/Public-key_cryptography
[29]: http://en.wikipedia.org/wiki/RSA_(algorithm)
[30]: http://www.serverwatch.com/eur/article.php/3501351/Enterprise-Unix-Roundup--The-Bitkeeper-Controversy.htm
[31]: http://en.wikipedia.org/wiki/SHA-1
[32]: http://en.wikipedia.org/wiki/Hard_link
[33]: http://en.wikipedia.org/wiki/Diff
[34]: http://getcomposer.org/
[35]: http://www.pip-installer.org/en/latest/index.html
[36]: https://npmjs.org/
[37]: http://bower.io
