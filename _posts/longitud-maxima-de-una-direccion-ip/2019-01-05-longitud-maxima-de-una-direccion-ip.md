---
layout: post
title: "Longitud máxima de una dirección IP"
date: 2019-01-05 09:29:20 -0600
categories: offtopic
---

Si estas diseñando una base de datos de un sitio WEB, y requieres almacenar en un campo de una tabla el valor de la IP de las personas que visitan el sitio, debes de especificar el tamaño máximo de ese campo, es ahí donde surge la pregunta: ¿Cuál es el tamaño máximo de una dirección IP?

Actualmente, los protocolos más usados en Internet son [IPv4](http://en.wikipedia.org/wiki/IPv4) u [IPv6](http://en.wikipedia.org/wiki/IPv6), por lo que usaremos estos como base.

## IPv4

Para el caso de IPv4, la dirección esta compuesta por 32bit o 4 octetos que cumple el siguiente formato: `255.255.255.255`

Por lo que la dirección ocupará una **longitud máxima 15 caracteres**.

## IPv6

Por su parte, IPv6 esta compuesta por 64bit de prefijo de red y 64bit de dirección, separados por el signo : (dos puntos), y cumple el siguiente formato: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

Por lo que la dirección ocupará una **longitud máxima de 39 caracteres**.

## Solución

Es aconsejable crear el campo de al menos **39 caracteres** para almacenar ambos tipos de protocolos.