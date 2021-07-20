# Blog

Blog Personal de [Herman Barrantes](https://www.hermanbarrantes.dev/).

## Motivación

Este blog lo creé con la idea de que sirva como un repositorio de las cosas que voy aprendiendo en el día a día de mi trabaja y que puedan ser de utilidad a más personas.

## Herramientas

- Sitio estático generado con [Jekyll](https://jekyllrb.com/)
- Desplegado en [Vercel](https://vercel.com/)
- Plantilla del sitio basada en [jekyll-klise](https://github.com/hermanbarrantes/jekyll-klise)
- Esquema de colores modificado en base a [Nord Theme](https://www.nordtheme.com/)

## Instalación

Ejecutar localmente usando [Jekyll](https://jekyllrb.com/):

```bash
$ git clone https://github.com/barrantesgerman/blog.git
$ cd blog
$ bundle install
$ bundle exec jekyll serve
```

Ejecutar localmente usando [Docker](https://github.com/envygeeks/jekyll-docker):

```bash
$ git clone https://github.com/barrantesgerman/blog.git
$ cd blog
$ docker run --rm -v $PWD:/srv/jekyll -p 4000:4000 -it jekyll/jekyll jekyll serve
```

Navega a `localhost:4000` para probar el sitio.

## Contribuir

Si ve algún error tipográfico o de formato en una publicación, o si desea ayudar con cualquier otro problema o sugerencia, no dude en abrir una solicitud de extracción y solucionarlo. Por favor, lea [Conviértete en colaborador](./CONTRIBUTING.md) y [Código de Conducta](./CODE_OF_CONDUCT.md) antes de hacer el PR.

## Licencia

Este proyecto es de código abierto y está disponible bajo la [Licencia MIT](LICENSE).
