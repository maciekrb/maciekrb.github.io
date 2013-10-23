---
layout: post
title: "PaaS y datacenters"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Estos servicios son distintos a, por ejemplo, Amazon EC3. Si bien tanto Amazon EC2 como Google App Engine y Heroku explotan los entornos virtualizados, estos dos últimos se ocupan de los aspectos de administración y optimización de las máquinas virtuales. En nuestras instancias EC2, normalmente tendremos que continuar actualizando árboles de ports o dependencias y depende en gran medida de cada SysAdmin construir herramientas para automatizar la creación de instancias automáticas y el deployment de aplicaciones en estas.

Heroku, además de eliminar una buena parte de la carga administrativa que representa mantener un OS al día, permite crear un proceso automatizado para hacer deployment de aplicaciones. Una analogía para quienes hemos mantenidos servidores por algún tiempo, es como recibir un servidor dedicado, y tener un script mágico que compila todas las dependencias, instala todas las aplicaciones, copia partes aquí y allá y arranca nuestra aplicación automáticamente. 

Esto ya es suficiente para llamar la atención de cualquier SysAdmin al que le timbre el teléfono a las 3 AM porque algún servidor está atorado por cualquier rázon que seguramente tomará una rídicula cantidad de tiempo en determinar y reestablecer, pero por la misma razón por la que todo corre en un flujo automatizado, escalar aplicaciones en Heroku es un proceso tivial.   
