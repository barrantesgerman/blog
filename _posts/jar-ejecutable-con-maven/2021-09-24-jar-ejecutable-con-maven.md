---
layout: post
title: "Jar Ejecutable con Maven"
date: 2021-09-24 12:35:20 -0600
categories: java
---

Los archivos `.jar` son empaquetados de clases de Java, se podría comparar con un simple archivo `.zip` que contiene todos los `.class` de nuestra aplicación. En el pueden o no haber una o varias clases principales.

Las clases principales son las que contienen un método con la firma `public static void main(String[] args)`, por ejemplo:

```java
package com.example.executable;
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

La manera tradicional de ejecutar esta clase principal es:

```shell
$ java -cp example.jar com.example.executable.Main
Hello World
```

Si damos doble clic o ejecutamos un `java -jar nombre-archivo-jar` obtendremos un error:

```shell
$ java -jar example.jar
no hay ningún atributo de manifiesto principal en example.jar
```

Para poder ejecutarlo con doble clic o `java -jar nombre-archivo-jar` se debe incluir dentro del JAR el archivo de manifiesto, localizado en `META-INF/MANIFEST.MF` y en su contenido el atributo `Main-Class` indicar el nombre de la clase principal que se debe ejecutar.

> **NOTA:** El archivo `META-INF/MANIFEST.MF` tiene que terminar con una línea en blanco para que sea correctamente interpretado por Java.

Adicionalmente, el proyecto puede requerir dependencias de terceros, estas dependencias se debe agregar también en el archivo de manifiesto, en el contenido del atributo `Class-Path` con la ruta de los JARs requeridos, separando cada uno con espacio.

Por ejemplo, si agregamos dependencias a [SLF4J](http://www.slf4j.org/) con [Log4j](http://logging.apache.org/log4j/1.2/) en el [pom.xml](https://github.com/barrantesgerman/executable-jar-maven/blob/main/pom.xml)

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.30</version>
</dependency>
```

Modificamos la clase principal `com.example.executable.Main`:

```java
package com.example.executable;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class Main {
    private static final Logger LOG = LoggerFactory.getLogger(Main.class);
    public static void main(String[] args) {
        LOG.info("Hello World");
    }
}
```

Y luego modificamos el archivo `META-INF/MANIFEST.MF`:

```properties
Manifest-Version: 1.0
Class-Path: libs/slf4j-log4j12-1.7.30.jar libs/slf4j-api-1.7.30.jar libs/log4j-1.2.17.jar
Main-Class: com.example.executable.Main

```

Se deben copiar todas las dependencias en una carpeta llamada *libs* junto al archivo JAR.

Ya con esto podemos ejecutar la aplicación dando doble clic en ella o ejecutando  `java -jar nombre-archivo-jar` sin problemas.

## Copiando Dependencias

[Maven](https://maven.apache.org/) cuenta con plugins que nos facilitan la tarea de crear el archivo de manifiesto y copiar las dependencias.

Con `maven-jar-plugin` podemos indicar la clase principal en el atributo `mainClass` y el nombre de la carpeta de dependencias en el atributo `classpathPrefix`.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>libs/</classpathPrefix>
                <mainClass>com.example.executable.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

Por su parte, el plugin `maven-dependency-plugin` nos ayuda a copiar todas las dependencias del proyecto en la carpeta indicada.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.2.0</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>
                    ${project.build.directory}/libs
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```


## Generar un FatJar con Plugin

Un **FatJar** (también conocido como **UberJar**) es un archivo JAR que contiene no solo sus clases, sino que también contiene dentro de él mismo todas sus dependencias. Su ventaja consiste en ser un único archivo (.jar) para distribuir la aplicación (no se requiere copiar la carpeta *libs* como en el caso anterior).

Existen varios plugins en Maven que nos permiten generar un FatJar, los siguientes son los más populares.

### Apache Maven Assembly Plugin

Se debe agregar el siguiente plugin al proyecto:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.3.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.example.executable.Main</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Apache Maven Shade Plugin

Se debe agregar el siguiente plugin al proyecto:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <executions>
        <execution>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <shadedArtifactAttached>true</shadedArtifactAttached>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.executable.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Onejar Maven Plugin

Se debe agregar el siguiente plugin al proyecto:

```xml
<plugin>
    <groupId>com.jolira</groupId>
    <artifactId>onejar-maven-plugin</artifactId>
    <version>1.4.4</version>
    <executions>
        <execution>
            <goals>
                <goal>one-jar</goal>
            </goals>
            <configuration>
                <mainClass>com.example.executable.Main</mainClass>
                <attachToBuild>true</attachToBuild>
                <filename>${project.build.finalName}-one.${project.packaging}</filename>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## Código Fuente de Ejemplo

📦 [executable-jar-maven](https://github.com/barrantesgerman/executable-jar-maven)

## Documentación de los Plugins

* [Apache Maven Dependency Plugin](https://maven.apache.org/plugins/maven-dependency-plugin/)
* [Apache Maven JAR Plugin](https://maven.apache.org/plugins/maven-jar-plugin/)
* [Apache Maven Assembly Plugin](https://maven.apache.org/plugins/maven-assembly-plugin/)
* [Apache Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/)
* [Onejar Maven Plugin](https://github.com/jolira/onejar-maven-plugin)

## Referencia

* [How to Create an Executable JAR with Maven](https://www.baeldung.com/executable-jar-with-maven)
