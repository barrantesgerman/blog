---
layout: post
title: "Git Flow Cheat Sheet"
date: 2021-07-16 12:57:00 -0600
categories: desarrollo
---

## Inicializar Git Flow

```bash
$ git flow init
```

Inicializa un repositorio de `git` existente para usar [git flow](https://danielkummer.github.io/git-flow-cheatsheet/).

## Trabajar una funcionalidad (feature)

```bash
$ git flow feature start <feature>
```

Empezar a desarrollar una nueva **feature**. Crea una nueva rama basada en **develop**.

```bash
$ git flow feature finish <feature>
```

Finaliza el desarrollo de una **feature**, la **feature** se fusiona en **develop**, se elimina la rama **feature** y cambia hacia la rama **develop**.

```bash
$ git flow feature publish <feature>
```

Publica una **feature** en el servidor remoto para que otros también puedan trabajar en ella.

```bash
$ git flow feature pull <remote> <feature>
```

Obtiene una **feature** publicada en su máquina local.

```bash
$ git flow feature track <feature>
```

Sigue una **feature** en **origin**. 

## Hacer un lanzamiento (release)

```bash
$ git flow release start <version>
```

Inicia una rama de **release**. Crea una nueva rama basada en **develop**. 

```bash
$ git flow release publish <version>
```

Publica un **release** en el servidor remoto para que otros puedan verla.

```bash
$ git flow release finish <version>
```

Termina el desarrollo de un **release**, el **release** se fusiona en **master**, se crea un **tag** en **master** con la `<version>`, el **master** se fusiona en **develop** y se elimina la rama **release**, finalmente cambia hacia la rama **develop**.

## Corregir incidencias (hotfix)

```bash
$ git flow hotfix start <version>
```

Inicia una nueva rama de **hotfix**. Crea una nueva rama basada en **master**.

```bash
$ git flow hotfix finish <version>
```

Termina un **hotfix**, el **hotfix** se fusiona en **develop** y **master**. se crea un **tag** en **master** con la `<version>`.

## Resumen de Comandos

{% mermaid %}
graph LR;
    GF[git flow] --> SP1((_));
    SP1 --> IN[init] & FE[feature] & RE[release] & HF[hotfix];
    FE & RE & HF --> SP2((_));
    SP2 --> ST[start] & FI[finish] & PB[publish] & PL[pull];
    ST & FI & PB & PL --> SP3((_));
    SP3 --> NM([name]);
{% endmermaid %}