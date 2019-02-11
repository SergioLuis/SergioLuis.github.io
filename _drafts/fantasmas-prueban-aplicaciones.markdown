---
layout: single
title: "Fantasmas que prueban aplicaciones"
date: 2019-02-11 09:00:00 +0100
categories:
    - meta
    - programming
tags:
    - plasticscm
    - dotnet
excerpt: "A principios de año he estado documentando cómo hay fantasmas en Códice dedicadas a testear la GUI de las aplicaciones. O al menos... ¡lo parece!"
header:
    overlay_image: assets/images/posts/003-fantasmas-prueban-aplicaciones/header.jpg
    overlay_filter: rgba(41, 98, 255, 0.6)
    caption: "Foto: [**Derek Bruff**](https://flic.kr/p/ieiWeh) [(CC BY-NC 2.0)](https://creativecommons.org/licenses/by-nc/2.0/)"
    actions:
    - label: "Post original"
      url: "http://blog.plasticscm.com/2019/01/guitestsharp-multiplatform-gui-testing-dotnet.html"
---
Seguro que, como desarrollador de software, estás acostumbrado a escribir concienzudos tests unitarios de los componentes de tus aplicaciones. Quizá incluso haces tests de integración, pruebas tus APIs de extremo a extremo, y sometes a tu backend a millares de peticiones simultáneas para ver cómo resisten la carga.

[![Enlace directo al blogpost en blog.plasticscm.com](/assets/images/posts/003-fantasmas-prueban-aplicaciones/miniatura.png)](http://blog.plasticscm.com/2019/01/guitestsharp-multiplatform-gui-testing-dotnet.html)

Pero, ¿y la interfaz gráfica de usuario? ¿Tienes también implementados tests automatizados que te alerten cuando algo que la GUI funcione mal, o tienes extensos departamentos de QA que pasan horas metiendo textos en cajitas y pulsando botoncitos, comprobando colorines y abriendo y cerrando diálogos concienzudamente, para asegurar que la aplicación no salga con errores?

> A QA engineer walks into a bar. Orders a beer. Orders 0 beers. Orders 99999999999 beers. Orders a lizard. Orders -1 beers. Orders a ueicbksjdhd.  
> First real customer walks in and asks where the bathroom is. The bar bursts into flames, killing everyone.  
>  
> _Brenan Keller_ ([@brenankeller](https://twitter.com/brenankeller)) - [November 30, 2018](https://twitter.com/brenankeller/status/1068615953989087232)

No hay muchos frameworks de GUI testing que hagan dicha tarea menos tediosa (¡por no decir tan siquiera agradable!), y la mayoría (al menos, que conozca) están centrados en el flujo de trabajo de _record and replay_. Tú comienzas a grabar, realizas una serie de acciones, y la herramienta graba en qué elementos haces clic y qué textos introduces, para reproducirlo a posteriori. Esto da lugar a tests complejos y frágiles: unas veces fallarán por tu culpa (porque has modificado algo en la GUI y no has vuelto a grabar el test), y otras muchas, por su propio diseño (porque no puedan encontrar elementos en la GUI por problemas de sincronización, por ejemplo).

Al menos si de lo que hablamos es de aplicaciones móviles, debido a que, al final, la mayoría se construyen utilizando las mismas tecnologías, tenemos soluciones más o menos estandarizadas, e incluso [apoyadas](https://developer.android.com/training/testing/ui-testing/) por el propio desarrollador del Sistema Operativo en cuestión.

Pero si de lo que hablamos es de aplicaciones de escritorio, nos introducimos en territorio comanche. A pesar de la longevidad del paradigma (o quizá debido a eso mismo), la diversidad de frameworks GUI es tal que es imposible hacer tu desarrollo de manera nativa en varios Sistemas Operativos y a la vez implementar tests de GUI sin invertir un esfuerzo tal que se vaya a ver recompensado (aumentando para colmo de males el _time to market_). Como se dice en España, _no se puede estar en misa y repicando_, no al menos si no recurrimos a tecnologías Web como Electron (en cuyo caso, cuentas con mi más absoluta animadversión 💔).

La realidad es que las aplicaciones de escritorio tradicional todavía existen, y si te dedicas al desarrollo de software, serán imprescindibles para ti al menos durante los próximos _``[número-razonable]``_ años.

Personalmente opino (y, por supuesto, estaré equivocado) que actualmente, si lo que quieres hacer es desarrollar aplicaciones de escritorio multiplataforma, la opción más robusta y respetuosa con los recursos hardware de tus usuarios es la plataforma .NET. Si quieres desarrollar para Windows, tienes .NET Framework con Windows Forms o WPF. Si quieres GNU/Linux, tienes tanto Mono con GtkSharp 2, o .NET Core con GtkSharp 3, y si lo que quieres es macOS, tienes Xamarin.Mac. La ventaja de todos ellos es que no reimplementan un dibujado propio de los elementos de GUI dependiendo de la plataforma, sino que, al final, todo son bindings con los elementos nativos correspondientes. Sí, por un lado tienes que planificar bien la arquitectura de tus aplicaciones para poder reutilizar tanto código como sea posible entre arquitecturas. Sí, también tienes que escribir la misma GUI hasta tres veces. Pero por el otro tienes aplicaciones que se sienten y se comportan como si fuesen completamente nativas a la plataforma, algo que, particularmente, cada vez echo más de menos. Echo de menos que las aplicaciones compartan un lenguaje común, echo de menos no tener que aprender dónde esconden sus ajustes y opciones cada una de las aplicaciones que uso a diario, y echo de menos los tiempos en los que una sencilla aplicación de chat no se comía medio _giga_ de RAM (¿puede por favor volver MSN Messenger? Hacía lo mismo que Slack, pero tenía zumbiditos molones).

Pero si lo que haces es dividir esfuerzos en programar tres GUIs distintas, ¿no nos estamos alejando del punto inicial del post, que era testear de manera sencilla la GUI? Si a duras penas encontramos soluciones propietarias para implementar los tests en una sola plataforma, ¿no estamos agrandando el programa al dividirnos en tres distintas? Bueno, la respuesta es relativamente sencilla. Si no te convence ningún framework de pruebas de GUI disponible en el mercado... programa el tuyo propio.
