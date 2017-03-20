---
layout: default
title: Sobre open source y otras yerbas
---

Sobre open source y otras yerbas
===

{{ page.date | date: "%-d %B %Y"}}

Desde hacía mucho tiempo quería comenzar a contribuir en algún proyecto [open source (OS)](https://es.wikipedia.org/wiki/C%C3%B3digo_abierto){:target="_blank"}, pero la verdad es que no encontraba ninguno con el que me sintiese cómodo. Es bastante común encontrar por ahí artículos o _podcasts_ que hablen de los beneficios de contribuir a proyectos de código abierto pero desde aquí quiero compartir mi humilde experiencia.

[Github](https://github.com){:target="_blank"} es, hoy en día, la plataforma de control de versiones en la nube más popular que hay. Mi intención no es la de ofrecer una guía acerca de cómo [contribuir en proyectos en esta plataforma](https://guides.github.com/activities/contributing-to-open-source/){:target="_blank"}, sino más bien comentar porqué creo es es una buena idea hacerlo.

Hace poco más de un año comencé a dar mi primeros pasos programando en Python, y a día de hoy programo a tiempo completo en este lenguaje. Entre las librerías que comencé a utilizar se encuentra [Eve](http://python-eve.org/){:target="_blank"}, desarrollada por [Nicola Iarocci](https://twitter.com/nicolaiarocci){:target="_blank"}. Eve es un framework que permite construir una API REST de forma sencilla. El motor de Eve es el conocido framework de desarrollo web para Python llamado [Flask](http://flask.pocoo.org/){:target="_blank"}. La idea principal de Eve es poder construir nuestra API REST simplemente definiendo un [schema](http://python-eve.org/config.html#schema){:target="_blank"}, como por ejemplo:

{% highlight python %}
people = {
    'schema' = {
        'firstname': {
            'type': 'string',
            'minlength': 1,
            'maxlength': 10,
        },
        'lastname': {
            'type': 'string',
            'minlength': 1,
            'maxlength': 15,
            'required': True,
            'unique': True,
        }
    }
}
{% endhighlight %}

Con este schema nuestra API tendrá un recurso llamado <code>people</code>. Para poder realizar un __POST__ en este recurso, la estructura del _body_ deberá verificar ese schema, es decir, deberá contener un campo llamado _firstname_, de tipo _string_, con un largo entre 1 y 10 caracteres. De forma similar ocurre con el otro campo, llamado _lastname_, que es obligatorio (_required_), y además, único en la base de datos (_unique_).

Este sistema de validación de schema lo realiza una herramienta llamada [Cerberus](http://docs.python-cerberus.org/en/stable/){:target="_blank"}, también desarrollada por Nicola y también OS.

En los últimos meses estuve usando Cerberus bastante y se me ocurrió una mejora que quizá comente en otro post, pero que básicamente consiste en hacer Cerberus 100% compatible con el estándar para APIs REST llamado [OpenAPI](https://github.com/OAI/OpenAPI-Specification){:target="_blank"}, que no es más que la especificación desarrollada por [Swagger](http://swagger.io/specification/){:target="_blank"}.

Debido a que esta mejora implicaría bastante tiempo y conocimiento del código de Cerberus, es que decidí comenzar a contribuir de forma más comedida, viendo los [issues](https://github.com/pyeve/cerberus/issues){:target="_blank"} abiertos en el proyecto.

> Al momento de elegir un proyecto al cual contribuir, elige un proyecto que utilices comúnmente.

Esto tiene varias ventajas. La primera es que conocerás más o menos su funcionamiento de antemano. Sabes para qué sirve y cómo se utiliza. Con esto reduces en gran medida la fricción inicial que conlleva corregir un _bug_ o agregar una funcionalidad en un programa que no sabemos muy bien qué hace. Luego está claro que si es una herramienta que utilizas diariamente, estarás corrigiendo errores y agregando funcionalidades a tu propio proyecto de forma indirecta. Esto incluso podría ser un factor a favor si deseas plantear en tu empresa contribuir al proyecto en horario de oficina. Por último, es la mejor forma de agradecer al o los desarrolladores que ofrecen la herramienta de forma gratuita.

Elegir un issue no es una tarea sencilla. Principalmente porque _a priori_ no sabes qué implica corregirlo.

> Para comenzar, elige el issue más fácil que encuentres.

Y esto puede ser tan sencillo como corregir un _link_ o un error tipográfico. Esto servirá, principalmente, para familiarizarnos con el _flow_ del proceso y perderle un poco el miedo. Este tipo de errores generalmente no tienen un issue asociado pero es común encontrarlos recorriendo la documentación. Otra tarea muy solicitada por este tipo de proyectos es la de traducción o ampliación de documentación.

En mi caso decidí por comenzar con el [issue más sencillo que encontré](https://github.com/pyeve/cerberus/issues/271){:target="_blank"}. Lo primero que hay que hacer es (una vez entendido los pasos básicos que se explican en la guía de Github) leer las ["reglas"](https://github.com/pyeve/cerberus/blob/master/CONTRIBUTING.rst){:target="_blank"} propias establecidas por el proyecto a la hora de querer enviar tu trabajo. Allí se establecen por ejemplo, reglas de estilo, de comentarios, etc.

El issue que elegí es el siguiente: Cerberus no ofrece soporte de [normalización](http://docs.python-cerberus.org/en/stable/normalization-rules.html){:target="_blank"} para [tuplas](https://docs.python.org/2/library/functions.html#tuple){:target="_blank"} (una tupla es, básicamente, una lista inmutable). Esto significa que, si en nuestro esquema definimos un atributo de tipo _list_ y le pasamos un objeto de tipo _tuple_, recibimos una _exception_ de tipo [TypeError](https://docs.python.org/2/library/exceptions.html#exceptions.TypeError){:target="_blank"}:

{% highlight shell %}
TypeError: 'tuple' object does not support item assignment
{% endhighlight %}

Esto es sencillamente por este trozo de código:

{% highlight python %}
for i in result:
    mapping[field][i] = result[i]
{% endhighlight %}

<code>mapping[field]</code> es la tupla a la cual se le está intentando asignar un elemento (normalizado) por posición. Esto lo podemos hacer con una lista, pero no con un objeto inmutable. <code>result</code> es un diccionario con los valores normalizados.

Mi primera solución fue la [siguiente](https://github.com/pyeve/cerberus/commit/371104953de17ec8d9a301985a49af67edbc0243#diff-795369a6cae83c62a70eafa3d3a1852e){:target="_blank"}:

{% highlight python %}
if type(mapping[field]) is tuple:
    mapping[field] = tuple(result.values())
else:
    mapping[field] = result.values()
{% endhighlight %}

El cambio no es más que, en lugar de iterar los valores del diccionario y _reasignarlos_ a la lista, asignar directamente los valores del diccionario mediante el método <code>values()</code>, el cual ya nos devuelve una lista. De esta manera evitamos asignar un valor por posición a la tupla.

Debo destacar en este punto que, la gran mayoría de los proyectos de código abierto, medianamente importantes, dan soporte a las últimas versiones de Python. Python hace unos años hizo un cambio importante en su librería estándar por lo que pasó de la versión 2.x a la versión 3.x. El EOL (End Of Life, es decir, la fecha en la cual dejará de tener soporte) para Python 2.7 [será en el año 2020](https://hg.python.org/peps/rev/76d43e52d978){:target="_blank"}, por lo que la comunidad está moviéndose a Python 3. De todas formas, Cerberus [soporta todas las versiones de Python](https://github.com/pyeve/cerberus/blob/master/setup.py){:target="_blank"} 2 y 3.

Una vez satisfecho con mis cambios realicé el correspondiente _pull request_ en Github (esto es, envié una solicitud de aceptación de mis cambios en el proyecto oficial). Cerberus tiene un sistema de CI ([Integración Continua](https://es.wikipedia.org/wiki/Integraci%C3%B3n_continua){:target="_blank"}), llamado [Travis](https://travis-ci.com/){:target="_blank"} que se ejecuta cada vez que se realiza un pull request. En este proceso se ejecutan todos los tests para todas las versiones de Python. Una vez enviado mi pull request pude ver, rápidamente, que mis cambios hacía fallar los tests.

![CI fallando miserablemente]({{ site.baseurl }}/assets/img/ci.png)

> Usa Tox

En el [README](https://github.com/pyeve/cerberus/blob/master/README.rst#testing){:target="_blank"} del proyecto claramente dicen que debes ejecutar [Tox](https://tox.readthedocs.io/en/latest/){:target="_blank"} para pasar los tests. Tox es una herramienta que automatiza los tests para diferentes configuraciones de entorno (diferentes librerías instaladas, diferentes versiones de Python, etc.) Como es evidente, yo no hice caso, y ahí está el error.

El problema está en que yo ejecuté los tests solamente para Python 2 <code>python setup.py test</code>, cuando debería haberlo hecho, como mínimo, también para Python 3 <code>python3 setup.py test</code>. En Python 2 el método <code>values()</code> de un diccionario devuelve una [lista](https://docs.python.org/2/library/stdtypes.html#dict.values){:target="_blank"} mientras que en Python 3, el mismo método devuelve un [view object](https://docs.python.org/3.6/library/stdtypes.html#dict-views){:target="_blank"}. Este cambio, como dicen en la documentación, es _para proveer de una vista dinámica de los valores del diccionario, lo que significa que cuando el diccionario cambia, la vista refleja estos cambios_.

{% highlight shell %}
Python 2.7.12
>>> a_dict = {'name': 'Sebastian'}
>>> elements = a_dict.values()
>>> elements
['Sebastian']
>>> a_dict['name'] = 'Sebastian Rajo'
>>> elements
['Sebastian']
{% endhighlight %}

{% highlight shell %}
Python 3.5.2
>>> a_dict = {'name': 'Sebastian'}
>>> elements = a_dict.values()
>>> elements
dict_values(['Sebastian'])
>>> a_dict['name'] = 'Sebastian Rajo'
>>> elements
dict_values(['Sebastian Rajo'])
{% endhighlight %}


Si queremos una lista, como en Python 2, debemos construirla así <code>list(result.values())</code>

Ahora debía _cancelar_ el pull request, y hacer [uno nuevo](https://github.com/pyeve/cerberus/pull/301){:target="_blank"} (o eso creía yo). Procedí a realizar un _close_ del primer pull request, e hice el cambio correspondiente en el código:

{% highlight python %}
if type(mapping[field]) is tuple:
    mapping[field] = tuple(result.values())
else:
    mapping[field] = list(result.values())
{% endhighlight %}

Confieso que el código quedó bastante feo, y lo notaba, pero no veía la solución. Abrí el segundo pull request.

Cerberus, además de Nicola, tiene otro gran [_contributor_](https://github.com/pyeve/cerberus/graphs/contributors){:target="_blank"}, _funkyfuture_ (Frank Sachsenheim). Lo primero que me recomendó fue _refactorizar_ el código utilizando únicamente el método [<code>type()</code>](https://docs.python.org/2/library/functions.html#type){:target="_blank"} de la siguiente manera.

{% highlight python %}
mapping[field] = type(mapping[field])(result.values())
{% endhighlight %}

Es decir, ¡sustituir las 4 líneas anteriores por solamente esa! Si al método le pasamos un argumento, éste debe ser un objeto, lo que nos devolverá _el tipo de un objeto_. Esto quiere decir:

{% highlight shell %}
>>> list == type([])
True
>>> tuple == type(())
True
{% endhighlight %}

En nuestro caso, <code>mapping[field]</code> a veces será una lista y a veces será una tupla. El código dinámicamente creará el tipo que corresponda y, al ejecutar el tipo, hace las veces de _constructor_, por lo que nos creará la instancia correspondiente.

{% highlight shell %}
>>> type([])([1,2,3])
[1, 2, 3]
>>> type(())([1,2,3])
(1, 2, 3)
{% endhighlight %}

Otra anotación que me hizo fue que, <code>if type(mapping[field]) is tuple:</code> no funciona para _subclases_ de _tuple_. Para eso es mejor utilizar [<code>isinstance(object, classinfo)</code>](https://docs.python.org/2/library/functions.html#isinstance){:target="_blank"} como claramente recomiendan en la documentación.

La tercera recomendación (habrá visto que había cancelado el primer pull request) fue que, luego de realizar los cambios, realizara un [squash o fixup](https://robots.thoughtbot.com/git-interactive-rebase-squash-amend-rewriting-history#squash-commits-together){:target="_blank"} de mis _commits_. ¿Cómo? Pues... sí. _git_ [tiene más de unos pocos comandos](https://xkcd.com/1597/){:target="_blank"} y no siempre los conocemos bien. En mi caso, nunca lo había usado. En realidad el comando en cuestión es el famoso [<code>rebase</code>](https://robots.thoughtbot.com/git-interactive-rebase-squash-amend-rewriting-history#rebase-example){:target="_blank"}. Entre sus opciones podemos realizar un squash o un fixup. Squash sirve para _unir_ varios commit, pudiéndole corregir a su vez los comentarios. Fixup, es lo mismo, pero eliminando directamente los comentarios de los commits _unidos_.

De esta experiencia no me llevo más que cosas positivas. He aprendido a utilizar [git](https://git-scm.com/){:target="_blank"} y Github de mejor forma, así como la librería estándar de Python. He contribuido a un proyecto que utilizo diariamente. He conseguido material para escribir este post. He agregado algo de _movimiento_ a mi cuenta de Github, lo que siempre es importante ya que hay muchas empresas que se fijan en ello, y aunque está claro que no trabajar en proyectos OS no te hace peor candidato, hacerlo puede que sea el factor decisivo al momento de elegirte. Y todo esto, invirtiendo unas pocas horas.

Es importante tener en cuenta que pasamos muchas horas frente al ordenador en nuestra actividad laboral, y seguir programando fuera del horario de oficina es costoso, tanto sicológica como físicamente. Pero aún así creo que es muy provechoso. La nuestra es una actividad en la que nunca paramos de aprender, y si queremos destacar debemos realizar esfuerzos extras en este sentido.

Por último remarcar el tiempo y guía que _funkyfuture_ me ofreció. Para alguien que está comenzando a contribuir es muy importante este tipo de recibimientos. Así que si eres responsable de un proyecto OS, _be kind_, sé como _funkyfuture_.

