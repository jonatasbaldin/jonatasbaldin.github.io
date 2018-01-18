---
layout: post
title: "Django + Jupyter = ❤️"
date: 2018-01-18 06:00:00
tags: ["python", "django", "shell", "ipython"]
description: "Gosta de Django? Simmm! Gosta de Jupyter? Simmm! Então imagina os dois juntos..."
comments: True
---

[Jupyter Notebooks](https://jupyter.org/) são usados para criar e compartilhar código em mais de quartenta linguagens, criando um ambiente onde visualização e narrativas são cidadões de primeira classe.

[Django Shell](https://docs.djangoproject.com/en/2.0/ref/django-admin/#shell) é uma ferramenta que inicia o interpretador do Python, com todas as configurações do Django, possibilitando você chegar chegando com um `from app.models import Model; Model.objects.all()`.

Agora imagina poder ter o mesmo poder do Django Shell *dentro* de um notebook Jupyter...

O [django_extensions](https://github.com/django-extensions/django-extensions) é uma biblioteca que dá super poderes do Django, adicionando features como o Graph Models, que exporta as relações do seus modelos em um arquivo PNG. Porém, a funcão que eu mais gosto é o comando `python manage.py shell_plus`, que carrega o Django Shell com o iPython e automaticamente importa várias coisinhas úteis – incluindo *todos* os seus modelos – o que significa que você pode chegar chegando e mandar um `Model.objects.all()` direto.

```
$ python manage.py shell_plus

...more Models importing...
from topics.models import Topic
# Shell Plus Django Imports
from django.core.cache import cache
from django.conf import settings
from django.db import transaction
from django.db.models import Avg, Case, Count, F, Max, Min, Prefetch, Q, Sum, When
from django.utils import timezone
from django.urls import reverse
Python 3.6.2 (v3.6.2:5fd33b5926, Jul 16 2017, 20:11:06)
Type "copyright", "credits" or "license" for more information.

IPython 5.2.0 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]:
```

Porém, esse comando há um argumento escondido... o `--notebook`.

![Personagens de desenho animado com cara de suspeito](/img/django_jupyter_notebook_gif.gif){: .center-image }

E é isso mesmo o que você está imaginando: ele cria um Jupyter Notebook com *todo* o contexto do Django!

```
$ python manage.py shell_plus --notebook

[I 10:44:02.120 NotebookApp] Serving notebooks from local directory: /Users/jonatasbaldin/github/project
[I 10:44:02.121 NotebookApp] 0 active kernels
[I 10:44:02.121 NotebookApp] The Jupyter Notebook is running at:
[I 10:44:02.121 NotebookApp] http://localhost:8888/?token=4e259dd015f0ca5a89fefd8c8d2aa8feb2162400bea32a89
[I 10:44:02.121 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 10:44:02.122 NotebookApp]
 
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=4e259dd015f0ca5a89fefd8c8d2aa8feb2162400bea32a89
[I 10:44:02.304 NotebookApp] Accepting one-time-token-authenticated connection from ::1
```

*Caso algum erro ocorra, instale o Jupyter: `pip install jupyter`*

Automagicamente, vai abrir seu browser na tela inicial do Jupyter:

![Notebook](/img/django_jupyter_notebook.png)

Agora é só ir em `New > Django Shell-Plus` e voilà, Django Shell dentro de um Notebook 🤘

![Notebook com Shell-Plus](/img/django_jupyter_notebook_shell_plus.png)

*Imagine as possibilidades...*
