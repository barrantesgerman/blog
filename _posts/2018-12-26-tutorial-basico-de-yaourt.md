---
layout: post
title: "Tutorial básico de Yaourt"
date: 2018-12-26 09:29:20 -0600
categories: linux manjaro
---

## ¿Qué es Yaourt?

`yaourt` (*Yet AnOther User Repository Tool*, **Yogurt** en francés) es un programa de interfaz de línea de comandos que completa `pacman` para la instalar software en Archlinux y derivadas.

A diferencia de `pacman` que instala paquetes solamente de los repositorios oficiales, `yaourt` permite instalar paquetes desde **Arch User Repository** ([AUR](https://wiki.archlinux.org/index.php/Arch_User_Repository_(Espa%C3%B1ol))) que es un repositorio promovido por los usuarios de la comunidad de Arch.

> Actualmente, el proyecto se encuentra descontinuado, por lo que se recomiendan otras herramientas similares como  [aurman](https://github.com/polygamma/aurman), [pakku](https://github.com/kitsunyan/pakku) o [yay](https://github.com/Jguer/yay).

## Como instalar

<del>`yaourt` se puede instalar desde `pacman` ejecutando: `$ sudo pacman -S yaourt`</del>

**[Actualización 19/04/2019]**

> `yaourt` ya no se encuentra dentro de los repositorios oficiales de **Manjaro Linux** por lo que no se puede instalar directamente mediante el uso de `pacman`, se recomienda el uso de otras herramientas similares, sin embargo, aún existe la posibilidad de instalarlo siguiendo la guía de [LinOxide](https://linoxide.com/linux-how-to/install-yaourt-arch-linux-2018/ "LinOxide").

## Uso básico

Una de las ventajas que tiene `yaourt` es que no requiere iniciar la orden con `sudo` a diferencia de `pacman`, sino que nos solicitará los privilegios de administrador solo si los requiere.

Ahora bien, esta es una lista de comandos básicos comparados contra los tradicionales comandos de `pacman`:

### Buscar paquetes

Se pueden buscar paquetes o grupos de paquetes por su nombre, y nos retornará una lista con todos los resultados encontrados para el nombre de paquete indicado.

```bash
# con yaourt
$ yaourt -Ss paquete
# o simplemente
$ yaourt paquete
# con pacman
$ sudo pacman -Ss paquete
```

### Actualizar sistema

Se pueden actualizar todos los paquetes del sistema a sus versiones más actualizadas.

```bash
# con yaourt
$ yaourt -Syyu
# actualizar tambien paquetes de AUR
$ yaourt -Syyua
# con pacman
$ sudo pacman -Syyu
```

### Instalar/reinstalar paquete

Ya identificado el paquete que se quiere instalar, simplemente se debe ejecutar el siguiente comando, el cual se encargará no solo de instalarlo, sino instalar todas las dependencias requeridas para su funcionamiento.

```bash
# con yaourt
$ yaourt -S paquete
# con pacman
$ sudo pacman -S paquete
```

### Eliminar paquetes y sus dependencias

Si por algún motivo ya no se requiere o desea tener un paquete instalado, se pude ejecutar el siguiente comando para eliminar el paquete y todas las dependencias y configuraciones que fueron instaladas con él.

```bash
# con yaourt
$ yaourt -Rnsc paquete
# con pacman
$ sudo pacman -Rnsc paquete
```

### Buscar paquetes huérfanos

En algunas ocasiones quedan paquetes huérfanos, los cuales son paquetes que fueron instalados como dependencias de otros paquetes que posteriormente fueron eliminados, sin embargo, no se eliminaron sus dependencias y ya no son dependencias de ningún otro paquete, por lo que ya no son necesarios.

```bash
# con yaourt
$ yaourt -Qdt
# con pacman
$ sudo pacman -Qdt
```

Finalmente les dejo el [Link](https://github.com/archlinuxfr/yaourt) de repositorio en **Github**.
