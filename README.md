# Wiki

Wiki personal / Jardín digital / Laboratorio.  
Me encontré con mucho contenido acerca de los jardines digitales, como [este texto de Barbi Couto](https://eneroenlaciudad.com.ar/el-software-los-jardines-digitales-las-redes-sociales-y-una-caotica-reflexion-sobre-la-libertad/), [este proyecto de 4lch4](https://github.com/4lch4/Digital-Garden) o [este ensayo de Maggie Appleton](https://maggieappleton.com/garden-history), y recuperar y mantener webs personales. Hace mucho tuve blogs y solía explorar y participar mucho en ese formato, hasta que caí en la trampa de las redes sociales...  
Además, paso mucho tiempo en internet tanto para "consumo personal" como para leer, aprender y probar herramientas y tecnologías que aplico en el trabajo o en otros proyectos que hago para distraerme y jugar un rato.

## Artículos y guías

- [Jardines digitales por el MIT](https://www.technologyreview.es/s/12606/jardines-digitales-la-respuesta-espiritual-la-futilidad-de-las-redes-sociales)
- [4lch4/Digital Garden](https://4lch4.garden/)
- [Today I Learned de jbranchaud](https://github.com/jbranchaud/til)
- [Vercel Doc: Deploying jekyll](https://vercel.com/guides/deploying-jekyll-with-vercel)
- [Repositorio theme](https://github.com/pmarsceill/just-the-docs)

## Proyecto

Entonces, luego de mucho navegar, leer y algunas pruebas, implementé esta wiki usando GitHub, archivos markdown, [Jekyll](https://jekyllrb.com/) y [Vercel](https://vercel.com/).  
No fue muy cómodo el tema ruby, node, gemas para poder servir jekyll localmente y en algún momento intentaré hacerlo mejor con docker, pero me funcionó 🔥.

## Implementación

Principalmente seguí esta [guía](https://vercel.com/guides/deploying-jekyll-with-vercel) de vercel:

- levantar template de sitio con jekyll desde vercel (requiere loguin)
- instalar vercel cli para aplicar automáticamente la configuración para jekyll ([ver más](#Instalar-node,-npm,-jekyll,-etc))

### Instalar node, npm, jekyll, etc

```shell
# Instalar nodejs, completar con la versión que corresponda
$ curl -sL <https://deb.nodesource.com/setup_12.x> | sudo -E bash -
$ apt install npm nodejs
$ nodejs -v
v10.19.0
$ npm -v
6.14.4
# Guía para instalar vercel cli: https://vercel.com/cli
$ npm i -g vercel
# Para deploy o autoconfiguración de un sitio ya existente
$ vercel
```

#### Ambiente local

- [Tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-jekyll-development-site-on-ubuntu-20-04)

```shell
# Instalar dependencias y configuraciones iniciales
$ sudo apt -y install make build-essential ruby ruby-dev
# Editar archivo .bashrc o .zshrc
$ code ~/.zshrc
# Luego ejecutar para aplicar los cambios
$ source ~/.zshrc
```

```bash
# Agregar al archivo .bashrc o .zshrc
# Ruby exports
export GEM_HOME=$HOME/gems
export PATH=$HOME/gems/bin:$PATH
```

```shell
$ gem install jekyll bundler
# node me mostraba un aviso por versiones distintas, entonces:
$ gem install bundler:2.2.4
# Para instalar el theme
$ sudo gem install just-the-docs
# Instalar dependencia que pedía node
$ gem install addressable -v 2.4.0
$ bundle install
# Levantar el sitio localmente, queda en ejecución y se actualiza automáticamente ante cada cambio
$ jeckyll serve
Configuration file: /_config.yml
            Source: /wiki
       Destination: /wiki/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 0.299 seconds.
 Auto-regeneration: enabled for '/wiki'
    Server address: http://127.0.0.1:4000/
```

### Troubleshooting

- Hice [esto](https://medium.com/@xiang_zhou/how-to-add-a-favicon-to-your-jekyll-site-2ac2179cc2ed) para agregarle el favicon, trascendental (?)
