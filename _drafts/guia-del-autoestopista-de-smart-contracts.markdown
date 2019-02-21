---
layout: single
title: "Guía del autoestopista de los Smart Contracts"
date: 2019-02-21 09:00:00 +0100
categories:
    - programming
    - guide
tags:
    - ethereum
excerpt: "Cuando comienzas en una nueva tecnología, la cantidad de recursos puede ser abrumadora. ESTO es lo básico para probar Solidity, los Smart Contracts de Ethereum Blockchain."
header:
    overlay_image: assets/images/posts/004-guia-del-autoestopista-de-smart-contracts/header.jpg
    overlay_filter: rgba(255, 171, 0, 0.6)
    caption: "Foto: [**Crypto360**](https://flic.kr/p/25z5431) [(CC BY 2.0)](https://creativecommons.org/licenses/by/2.0/)"
---
¿A qué huele una unidad de gas? ¿Será inflamable el Ether? ¿De qué color es una transacción minada? Todas estas preguntas vagan por mi cerebro mientras hago búsqueda tras búsqueda en Google, intentando no meterme demasiado en el infierno que es la página de resultados de cualquier palabra clave remotamente relacionada con criptomonedas.

¿El denomindador común de todos ellos? Que están centrados en la especulación, y en intentar explicar de manera sencilla la tecnología a gente que no está interesada en la tecnología, sino en un gran beneficio económico a corto plazo.

> _¿Qué es Ethereum? Información para inversores_  
> _Gráfico precio Ethereum_  
> _La moneda Ethereum (lo que necesitas saber)_  
> _What is Ethereum? The most comprehensive beginners guide_

Toda esta (inútil y desmoralizante) cantidad de información hace, si tienes el ánimo bajo, que termines desistiendo rápidamente de introducirte en el mundo de las criptomonedas. ¿Acaso necesito ser rico para encontrar una aplicación práctica al conocimiento que adquiera? _¡Un Bitcoin es casi el doble de mi sueldo mensual!_ Y es que ya sea por azares del destino o por mala suerte, me he metido a desarrollar Smart Contracts justo cuando las criptomonedas en general, y [Ethereum](https://www.thestreet.com/investing/bitcoin/attack-against-ethereum-classic-14832327) [en particular](https://www.investopedia.com/news/hackers-use-unsafe-clients-steal-20m-ether/), no están pasando por su mejor momento. Parece que la burbuja de especulación creada alrededor de esta tecnología está a punto de reventar, con [los chinos apagando sus granjas](https://www.coindesk.com/600k-bitcoin-miners-shut-down-in-last-2-weeks-f2pool-founder-estimates) porque la electricidad invertida en minar es más cara que la recompensa. Si no hay especulación no hay mercado, y sin mercado, lo que ahora es algo _mainstream_ (hemos oído hablar de criptomonedas en la cola de la frutería) corre el riesgo de volver a convertirse en un _hobby_ de nicho. Menos mal que esto lo estoy haciendo por cuestiones académicas, y no en busca de ese dulce, dulce [Wei](https://forum.ethereum.org/discussion/304/what-is-wei) que sumar a mi [EOA](https://ethereum.stackexchange.com/questions/5828/what-is-an-eoa-account) de Ethereum.

Vamos a tratar de sumergirnos un poco en la parte de desarrollo que tiene este mundillo, pero sin ahogarnos. Básicamente, como si estuviésemos en la piscina y no quisiésemos mojarnos la tripa porque acabamos de comer.

# Breve introducción al _blockchain_

Asumo que si te ha interesado el blogpost hasta aquí, tienes al menos un conocimiento básico de qué es la tecnología _blockchain_ y dónde radica su interés. Si no, este vídeo lo resume de manera bastante didáctica:

<iframe width="560" height="315" src="https://www.youtube.com/embed/SSo_EIwHSd4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Por ponerlo en bullet points:

* Un bloque agrupa transacciones.
* Estas transacciones pueden representar cualquier cosa...
  * ...aunque normalmente representan que Alice le ha enviado divisas de la criptomoneda de turno a Bob.
* Todas las transacciones nuevas van a un _pool_ de transacciones sin confirmar.
* Cada cierto tiempo, los nodos que participan en la _blockchain_, que pueden ser operados por _cualquier hijo de vecino_, cogen unas cuantas transacciones de dicho _pool_, las agrupan en un bloque, e intentan minarlo.
  * Antes de comenzar a minarlo, cada nodo añade una transacción que representa la _recompensa_. En Bitcoin, es así como se acuña nueva criptomoneda, aunque otras _blockchains_ usan otros métodos, como un impuesto sobre las transacciones que se minen.
* La dificultad de minar un bloque radica en que el hash del mismo ha de comenzar por un número determinado de ceros.
  * Esto es lo que se conoce como _proof of work_, una forma elegante de decir _"mira qué de [FPGAs](https://en.wikipedia.org/wiki/Field-programmable_gate_array) me he comprado"_.
* Para modificar el hash del bloque, como no se pueden modificar las transacciones, se modifica una parte variable llamada _nonce_.
* Cuando un nodo encuentra un _nonce_ que haga que el hash del bloque satisfaga el _proof of work_, lo distribuye al resto de nodos, que aseguran que el hash concuerda con el nonce y las transacciones.
* Si todos los nodos están de acuerdo en que el bloque es correcto, este se añade a la cadena, los nodos abortan el minado fallido, recogen más transacciones del _pool_, y comienza el minado de un nuevo bloque.

Una transacción no está _realmente_ confirmada cuando entra en un bloque: el último bloque de la cadena es el más débil, ya que está sujeto a escenarios excepcionales tales como que dos mineros minen un bloque simultáneamente. Si esto sucediese, cuando se mine el siguiente bloque se llegará al consenso de cuál de los dos bloques simultáneos anteriores es el que pervive, y las transacciones del descartado volverán al _pool_. Pero eso queda fuera del alcance de este post, así que haced como que no os he dicho nada.

# ¿Qué es un Smart Contract?

El objetivo de una _blockchain_ concreta (Bitcoin, por ejemplo) puede ser únicamente la transferencia de moneda digital. Sin embargo, algunas _blockchains_ (y, en concreto, Ethereum), además de a dicha transferencia, dan soporte a una cosa llamada _Smart Contracts_: la capa de lógica de negocio que se ejecuta sobre la cadena de bloques.

Bueno, no exactamente _"sobre"_ la cadena de bloques. Al fin y al cabo, esta no es más que datos replicados en los nodos participantes. Donde se ejecuta realmente un _Smart Contract_ es en una máquina virtual (Ethereum Virtual Machine, a.k.a. EVM) que se ejecuta en dichos nodos.

Estos _Smart Contracts_ suelen estar escritos en un lenguaje de programación sencillo (Solidity, entre otros), no muy lejano a Java o C#, y con unas capacidades relativamente limitadas - a pesar de ser lenguajes _Turing-completos_, la EVM impone restricciones sobre el tiempo que puede ejecutarse, el número de operaciones que puede realizar en ese tiempo, y los datos a los que puede acceder.

## ¿Cuál es el objetivo de un Smart Contract?

El concepto de _Smart Contract_ no surgió con el de _blockchain_, sino que es mucho anterior. Fueron [planteados por primera vez](https://firstmonday.org/ojs/index.php/fm/article/view/548/469) por Nick Szabo en la década de los 90. La idea principal es que, mientras que el cumplimiento de un contrato _tradicional_ solo puede ser asegurado mediante métodos coercitivos (normalmente con el Estado de por medio, con medidas tales como sanciones económicas o penas de cárcel, siempre al amparo de la ley), un _Smart Contract_, valiéndose de métodos digitales, aseguraría su cumplimiento por la sencilla inevitabilidad de este.

Si tenemos una pieza de código que...:

* Una vez puesta en producción, no puede modificarse.
* No es controlada por ninguna de las partes, sino por una red de nodos distribuida operada por agentes anónimos que han de alcanzar un consenso sobre el resultado de la ejecución de dicho código.
* Al ejecutarse realizará una acción para la que normalmente usaríamos contratos tradicionales:
  * Transferencia de bienes.
  * Obtención de licencias.
  * Contratación de seguros.
  * ...y un largo etcétera.
* Puede comprobar precondiciones para asegurar que su ejecución es posible.
  * Por ejemplo, que una persona tenga tanto o más dinero del que intenta gastar.
* Puede comprobar postcondiciones para asegurar que su ejecución fue correcta.
  * Por ejemplo, que al destinatario efectivamente le ha llegado el _asset_ que estaba comprando.
* Su resultado es público, pudiendo consultarse y verificarse en cualquier momento por cualquiera.

...entonces tenemos un _Smart Contract_ en el sentido más estricto del concepto.

Sin embargo, los _Smart Contracts_ son la parte de una _blockchain_ que un desarrollador puede programar, por lo que, en realidad, su aplicación va más allá que simplemente reemplazar los aburridos y farragosos contratos tradicionales. Pensad en ellos como programas que funcionan sobre una nube pública a la que cualquiera, con una conexión a Internet, tiene acceso. Y, además, podemos acceder a sus estado y ejecutar sus funciones mediante APIs REST o JavaScript. ¿La cosa empieza a parecer más interesante?
