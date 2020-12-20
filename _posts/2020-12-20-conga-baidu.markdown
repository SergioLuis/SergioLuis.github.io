---
layout: single
title: "Pi-hole y Cecotec Conga en la misma LAN"
date: 2020-12-20 20:30:00 +0100
categories:
    - iot
tags:
    - pihole
    - raspberry
    - smarthome
excerpt: "Desde que instalé Pi-hole en casa para bloquear publicidad y malware,
dejé de poder usar mi Cecotec Conga. El porqué no te sorprenderá."
header:
  overlay_image: assets/images/posts/007-conga-baidu/header.jpg
  overlay_filter: rgba(255, 0, 0, 0.4)
  caption: "Foto: [**Daniel Cukier**](https://flic.kr/p/NCcbLp) [(CC BY-ND 2.0)](https://creativecommons.org/licenses/by-nd/2.0/)"
---
Desde que monté en mi red local una Raspberry Pi con Pi-hole para filtrar
dominios sospechosos (y publicidad) a nivel de DNS, dejé de poder usar mi
[Cecotec Conga](https://cecotec.es/conga-3290-titanium) desde la aplicación
oficial para el teléfono ([Android](https://play.google.com/store/apps/details?id=es.cecotec.s3590),
[iOS](https://apps.apple.com/es/app/conga-3000/id1469759495)) (que es...
_subóptima_, por decirlo de alguna forma) cuando este estaba conectado a la
misma LAN que el robot.

Sospechaba que estaba filtrando algún dominio que no debía, pero hasta ahora el
espíritu procastinador ha sido más fuerte que mi interés por arreglarlo. Además,
si el dominio había caído en alguna blacklist, descubrir cuál era no iba a hacer
más que empeorar mi humor, así que ¿por qué andar por ese camino? ¡Todavía podía
usar el robot desconectando el teléfono de la red WiFi! Pero todo inconveniente
termina superando su límite de tolerancia, así que por fin he depurado el
problema.

El dominio que estaba bloqueando, y que para la Conga es absolutamente necesario
para funcionar es...

```text
www.baidu.com
```

Descubrirlo es sencillo:

1. Monitorizar las peticiones DNS realizadas desde mi teléfono en el momento de
   abrir la aplicación.
2. De entre los dominios bloqueados, escoger el que tuviese un aspecto más
   _chungo_.
3. Añadirlo a la _allowlist_ de Pi-hole (sigue llamándose _whitelist_ en el
   panel).
4. Probar de nuevo.
5. ???
6. Profit!

Para monitorizar las peticiones, aconsejo usar directamente la terminal. El
panel web de Pi-hole está bastante bien para bastante poco, y bastante mal para
bastante mucho. Personalmente ejecuto Pi-hole en un container de Docker, así que
para mí el primer paso es ejecutar Bash en una sesión interactiva contra es
container. Necesitaremos también saber la dirección IP del teléfono en la LAN -
lo usaremos para filtrar el log. En la línea siguiente a la query DNS será donde
veremos qué ha decidido hacer Pi-hole con ella, si bloquearla o darla paso, así
que usaremos el flag `-A` para dar un contexto de las dos líneas posteriores a
la que ha hecho match con el string de búsqueda.

```bash
$ docker exec -it pihole /bin/bash
$ pihole -t | grep 192.168.0.142 -A 2
```

El resultado (parcial):

```text
Dec  3 20:09:40: query[type=65] api-glb-euw3c.smoot.apple.com from 192.168.0.142
Dec  3 20:09:40: cached api-glb-euw3c.smoot.apple.com is <CNAME>
Dec  3 20:09:40: forwarded api-glb-euw3c.smoot.apple.com to 8.8.8.8
Dec  3 20:09:40: query[A] api-glb-euw3c.smoot.apple.com from 192.168.0.142
Dec  3 20:09:40: cached api-glb-euw3c.smoot.apple.com is <CNAME>
Dec  3 20:09:40: cached smoot-searchv2-euw3c.v.aaplimg.com is 15.188.167.1
Dec  3 20:09:40: query[type=65] smoot-searchv2-euw3c.v.aaplimg.com from 192.168.0.142
Dec  3 20:09:40: forwarded smoot-searchv2-euw3c.v.aaplimg.com to 8.8.8.8
Dec  3 20:09:41: query[A] www.baidu.com from 192.168.0.142
Dec  3 20:09:41: gravity blocked www.baidu.com is 0.0.0.0
Dec  3 20:09:41: query[A] bugly.qq.com from 192.168.0.142
Dec  3 20:09:41: gravity blocked bugly.qq.com is 0.0.0.0
Dec  3 20:09:41: query[type=65] cecotec-ota.3irobotix.net from 192.168.0.142
Dec  3 20:09:41: forwarded cecotec-ota.3irobotix.net to 8.8.8.8
Dec  3 20:09:41: query[A] cecotec-ota.3irobotix.net from 192.168.0.142
Dec  3 20:09:41: cached cecotec-ota.3irobotix.net is 8.209.81.208
```

De este log podemos sacar que:

- Se hacen peticiones a servidores de Apple. No nos interesan, porque además se
  están permitiendo.
- Se hacen peticiones a servidores de Cecotec (`cecotec-ota.3irobotix.net`) que,
  además de ser esperables, tampoco nos interesan, porque se están permitiendo.
- Se hacen peticiones a `bugly.qq.com` que son bloqueadas. Primer candidato.
- Se hacen peticiones a `www.baidu.com` que son bloqueadas. Segundo candidato.

Investigando un poco sobre Bugly, veo que es una herramienta de monitorización
del comportamiento de aplicaciones (logs, crashes, etc.). Estaría feo que una
herramienta de monitorización paralice la aplicación completa, pero no lo
descarto.

El que me eriza un poco el pelo de la nuca (de manera irracional, todo hay que
decirlo) es el segundo: Baidu.

Cuando uno compra una aspiradora _inteligente_ a una compañía española como es
Cecotec, podría pensar (aunque solo sea por la fuerza de la costumbre) que toda
su parte _smart_ pasa por algún cajón con luces en una nave de Amazon AWS,
dentro de Europa, y relativamente cerca geográficamente al usuario. No es el
caso con las Cecotec Conga. El mapa de tu casa creado por el robot está
igualmente en una caja con luces, pero la nave pertenece a Baidu, y está en PRC,
que no se refiere al Partido Regionalista Cántabro, sino a la _People's Republic
of China_.

Así que si ~~sufres~~ tienes una aspiradora inteligente Conga, y se da también
la casualidad de que usas Pi-hole, recuerda permitir la resolución de nombre de
`www.baidu.com`.
