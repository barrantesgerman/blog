---
layout: post
title: "Usar chmod en Linux"
date: 2019-04-11 09:29:20 -0600
categories: linux
---

Todos los sistemas de archivos basados en **Unix**, como lo es **Linux**, manejan tres tipos de permisos: lectura, escritura y ejecución. Los mismos se otorgan a tres categorías posibles: el usuario (*usuario*), el grupo al que pertenece el usuario (*grupo*) y todos los demás usuarios del sistema (*global*).

Para ver los permisos de un directorio podemos ejecutar el comando `ls -l` en un terminal y obtener una salida como la siguiente:

```bash
$ ls -l
lrwxrwxrwx  1 usuario grupo    14 may 21  2018  Datos -> /datos/usuario/
-rw-r--r--  1 usuario grupo     0 abr 11 17:11  ejemplo.txt
drwxr-xr-x  2 usuario grupo  4096 mar 24 11:19  Escritorio
```

Los primeros 10 caracteres representan los permisos, para el ejemplo anterior corresponden a:

| Tipo de archivo      | Usuario | Grupo | Global |
| -------------------- | ------- | ----- | ------ |
| `l` Enlace simbólico | `rwx`   | `rwx` | `rwx`  |
| `-` Archivo          | `rw-`   | `r--` | `r--`  |
| `d` Directorio       | `rwx`   | `r-x` | `r-x`  |

El primero corresponde al tipo de archivo. Los restantes 9 agrupados en 3 grupos de 3 letras representan los permisos del usuario, el grupo y todos los demás (global) respectivamente. Cada letra representa:

- `r` Lectura (**r**ead)
- `w` Escritura (**w**rite)
- `x` Ejecución (e**x**ecute)

Ya comprendido lo anterior, la sintaxis a seguir para asignar permisos a un archivo o directorio es la siguiente:

```bash
$ chmod [u,g,o,a][+,-,=][r,w,x] archivo_o_directorio
```

Donde los parámetros corresponden a:

| Parámetro | Significado |
| --------- | ----------- |
| `u`       | Usuario     |
| `g`       | Grupo       |
| `o`       | Global      |
| `a`       | Todos       |
| `+`       | Otorgar     |
| `-`       | Revocar     |
| `=`       | Copiar      |
| `r`       | Lectura     |
| `w`       | Escritura   |
| `x`       | Ejecución   |

Por ejemplo, si un usuario quiere otorgarle permisos de escritura al archivo *ejemplo.txt* a todos los usuarios de su mismo grupo, debe ejecutar `chmod` con los parámetros `g+w` de la siguiente manera:

```bash
$ chmod g+w ejemplo.txt
```

Para revocar el permiso se cambiar el parámetro por `g-w`:

```bash
$ chmod g-w ejemplo.txt
```

Si lo que desea es que el grupo tenga los mismos permisos que el usuario sobre el directorio *Escritorio*, debe ejecutar `chmod` con los parámetros `g=u`:

```bash
$ chmod g=u Escritorio
```

Si se indica la categoría `a` o no se indica ninguna categoría, los permisos se aplica a todas las categorías:

```bash
# Se otorga permiso de ejecución a usuario, grupo y global
$ chmod a+x ejemplo.txt
# Equivalente a
$ chmod +x ejemplo.txt
```

Se pueden especificar diferentes permisos al mismo tiempo utilizando coma:

```bash
$ chmod g+w,o-rw,a+x ejemplo.txt
```

Y se pueden aplicar permisos recursivamente utilizando el parámetro `-R`:

```bash
$ chmod -R +r Escritorio
```

El anterior ejemplo, le otorga permisos de lectura al directorio *Escritorio* al igual que a todo su contenido.

Otro método de establecer permisos es mediante el uso de la **notación octal**, la cual consiste en representar los 3 permisos (lectura, escritura y ejecución) en binario, donde 0 es que no tiene el permiso y 1 que si tiene el permiso y pasarlos luego a su representación en base 8, utilizando los dígitos del 0 al 7.

| Binario | Octal | Permiso |
| ------- | ----- | ------- |
| 000     | 0     | `---`   |
| 001     | 1     | `--x`   |
| 010     | 2     | `-w-`   |
| 011     | 3     | `-wx`   |
| 100     | 4     | `r--`   |
| 101     | 5     | `r-x`   |
| 110     | 6     | `rw-`   |
| 111     | 7     | `rwx`   |

La sintaxis de la notación octal sería igual a la anterior, remplazado los parámetros por el valor octal:

```bash
$ chmod 750 ejemplo.txt
```

Así por ejemplo el parámetro `750` significa que el usuario tiene permisos `rwx`, el grupo tiene permisos `r-x` y los otros no tiene permisos `---`.

Las notaciones octales más frecuentes son:

| Octal | Permisos                                                     |
| ----- | ------------------------------------------------------------ |
| 777   | `rwxrwxrwx` Todos tienen todos los permisos                  |
| 755   | `rwxr-xr-x` Usuario tiene todos los permisos, los demás solo leer y ejecutar |
| 644   | `rw-r--r--` Usuario puede leer y escribir, los demás solo leer |
| 600   | `rw-------` Usuario puede leer y escribir, los demás sin permisos |
