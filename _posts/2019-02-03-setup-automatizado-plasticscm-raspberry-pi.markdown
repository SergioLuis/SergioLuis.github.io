---
layout:     single
title:      "Setup automatizado de Plastic SCM en una Raspberry Pi"
date:       2019-02-03 19:00:00 +0100
categories:
    - meta
tags:
    - plasticscm
    - raspberrypi
excerpt: "El porqué y el cómo automaticé mi instalación de Plastic SCM en una RPi. ¡Actualizaciones y backups automáticos, y notificationes en tu móvil!"
header:
    overlay_image: assets/images/posts/2019-02-03-setup-automatizado-plasticscm-raspberry-pi/header.jpg
    overlay_filter: rgba(0, 191, 165, 0.5)
    caption: "Foto: [**Matt Wareham**](https://flic.kr/p/eSZxTp)"
    actions:
    - label: "Post original"
      url: "http://blog.plasticscm.com/2019/01/fully-automated-plastic-scm-setup-on-raspberry-pi.html"
---
Prácticamente desde que comencé mis Prácticas de Empresa del Grado en Códice Software no he usado ningún otro _software de control de versiones_ que no sea [**Plastic SCM**](http://plasticscm.com/) (salvo cuando usar Git ha sido obligatorio para compartir algo en GitHub, o con mis compañeros y profesores del Máster).

[![Enlace directo al blogpost en blog.plasticscm.com](/assets/images/posts/2019-02-03-setup-automatizado-plasticscm-raspberry-pi/miniatura.png)](http://blog.plasticscm.com/2019/01/fully-automated-plastic-scm-setup-on-raspberry-pi.html)

Y, prácticamente desde que comencé a utilizar Plastic SCM como mi opción principal para controlar no solo código, sino todo tipo de ficheros que requiera versionado, tengo un servidor instalado en una Raspberry Pi, al que puedo acceder cómodamente desde cualquier lugar gracias a un servicio de [Dynamic DNS](https://www.noip.com/) y redirección de puertos en el NAT.

Sin embargo, por cómodo que fuese el acceso, debido al alto ritmo al que se publican nuevas versiones de Plastic SCM (muchas veces más de dos a la semana) mantener todas las instalaciones al día (portátil de trabajo, portátil y ordenador sobremesa personales, Raspberry Pi...) se había convertido en una tarea tediosa. Así pues, me propuse **automatizar la instalación de Plastic SCM en la Raspberry Pi lo máximo posible**, convirtiéndola de una simple _commodity_ a un _full-blown-central-server-de-la-leche_.

Los objetivos eran los siguientes:

1. Actualización del servidor de Plastic SCM a nuevas versiones en un plazo máximo de 24 horas desde su publicación.
2. Backups comprimidos, automatizados, cifrados y en Cloud de los ficheros de _backend_ en los que Plastic SCM almacena los repositorios.
3. Que los puntos 1) y 2) no interrumpiesen mi trabajo **ni requiriesen supervisión explícita** (esto es: tener que entrar por SSH a la Raspberry y comprobar versiones, logs...)

Y el _TL;DR_ de lo que hice para conseguirlo es:

* Seis recetas de [**IFTTT**](https://ifttt.com/), con un _WebHook_ como _trigger_, para mandar notificaciones a mi teléfono móvil desde la Raspberry Pi con información sobre las acciones que esta iba ejecutando tanto en materia de actualizaciones como de copia de seguridad.
* Un [script](https://gist.github.com/SergioLuis/18df57d1098443bb6e5ba4836287bd93) que hiciese _backup_ automático del _backend_ a OneDrive, cifrado con ``GPG``, asegurando la consistencia de los datos activando y desactivando el modo ``readonly`` en el servidor de Plastic SCM cuando fuese necesario.
* Otro [script](https://gist.github.com/SergioLuis/2c6b347e0c9a909482d6ec02f0e5ee6f) que, comprobando las _release notes_ de plasticscm.com, pudiese hacer _scrapping_ de cuál es la última versión publicada para:
  * Descargar de Internet y descomprimir los binarios de la versión correcta.
  * Parar el servidor en ejecución de manera ordenada, para evitar la posible pérdida de datos (¡como si eso fuese posible! 😜)
  * Sustituir los binarios viejos por los nuevos.
  * Arrancar de nuevo el servidor en modo _daemon_.

Ahora que os he contado el _qué_, si tenéis la curiosidad suficiente, podéis consultar el _cómo_ en el blogpost que escribí (en inglés) [**en el blog oficial de Plastic SCM**](http://blog.plasticscm.com/2019/01/fully-automated-plastic-scm-setup-on-raspberry-pi.html).
