---
layout: post
title: Quick setup for django pegasus
description: Notes for quickly setting up a django pegasus project
toc: 
  sidebar: right
tags: django python
categories: 
featured: true
---
[SaaS Pegasus](https://www.saaspegasus.com/projects/) is a configurable (commercial) template for django. There is good [official documentation](https://docs.saaspegasus.com/) available from the software author.

The author also provides great [guides on how the use django in a modern webstack](https://www.saaspegasus.com/guides/) with a plethora of backend and frontend options from APIs over django core functionality to modern frontends like react, vue or htmx. I highly recommend to read through the guides if you want to better understand choices and tradeoffs in modern webstacks, in particular from a django perspective.

SaaS Pegasus is a pretty good starting point for a django project that will come out of the box configured with many things you otherwise might have to set up manually. For example user management, different frontend options and backend options can be configured out of the box with reasonable choices and example code ready for use.

However some finer points regarding configuration choices are a bit hidden in the docs and additionally I prefer to use a bit of a different setup in some details. So here are my notes for exactly that purpose.

So these notes are most likely only useful for you, if you have bought access to SaaS Pegasus and want to, for example use `python poetry` to manage the environment instead of `venv` which is covered in the official docs.

## Initialization from pegasus

Starting a new pegasus project, is straight forward for the most part, just log into the web interface, give your project a name, pick some options, download it and you are good to go.

Here I go into some consequences for the available options and explain alternatives (e.g. `poetry` instead of `venv`).

### Configuration in pegasus

There are a couple of setting that are not very obvious in their consequences:

Option `Example pages`
: Pulling in example pages results in a much larger code base. If you just want to inspect them, it's better to look at them in a separate project.

Option `Include static files`
: This pulls in static css  and js files of the chosen frontend framework as opposed to pulling in only the sass files and generate the css on-the-fly. If changes to css via sass are planned, then it's better to leave this unchecked and follow the instructions in the docs under <https://docs.saaspegasus.com/front-end/>. This will require setting up npm and running the webpack compiler whenever the css changes. The same goes for `webpack`: if the static files are included there is no webpack setup, for example if you want to include your own javascript.

### git / github

Configure and download project, unpack and run:

``` bash
git init
git add .
git commit -am "initial project creation"
git branch pegasus  # for easy merging of future updates
git branch -M main
# head over to github (or whereever) and create repo
git remote add origin https://<created-github-repo>
```

In case remote and local default branches have different names, you can rename the local branch `git branch -m master main`. If pushing to main doesn't recognize the branch right away it's possible to set it explicitly using `git branch --set-upstream-to=origin/main main`.
'''
Then sync both branches, main and pegasus.

It's best to create the repo empty on the github side, otherwise there is some need to merge e.g. gitignore files.

### python env using poetry

`poetry` is not officially supported but can be used via the `requirement.ini` files.

First initialize poetry by running `poetry init`, make sure to select the proper python version and dont specifiy any dependencies during initialization. The new poetry configuration will be appended to the existing `pyproject.toml`.

The python versions for pegasus versions are:

+ pegasus-2023.11.1 (since 2023.4): python-3.11

To create the virtual env _inside_ the project directory in `.venv`, check that the option `virtualenvs.in-project` is set to `true`:

```bash
poetry config --list  # check config settings
poetry virtualenvs.in-project true  # set explicitly
```

Then add the required packages via poetry as described below.

`requirement.txt` contains fixed versions of  _all_ packages of the resolved dependency tree, similar to what is recorded in `poetry.lock`.

We could run `poetry add` with the fully-resolved list of packages and their version constraints in the `requirements/*requirements.txt` files, but it's more generic to add the actual requirements from the `requirements/*requirements.in` files.

So if we want to use the minimal, generic dependencies, we just run `poetry add` for each line of the `.in` files. Note that this will not result in the exact same package versions as in the `requirements.txt` files, since poetry will run the resolver and pull the latest available packages fitting the constraints. This may or may not work with the code base (but often will).

To run `poetry add` on each line use the following commands. `grep` excludes links to constraints and `sed` drops comments.

```bash
cat requirements/dev-requirements.in | grep -v '\-c '| sed 's/\s#.*//' | xargs poetry add --group dev
```

```bash
cat requirements/requirements.in | grep -v '\-c ' | sed 's/\s#.*//' | xargs poetry add
```

Lastly, to select the correct virtual environment in visual studio code, select `Python: Select Interpreter` from the command palette (Shift-Ctrl-P) and pick `./.venv` from the list.

### Setting up a `webpack` workflow

The reasoning and tradeoffs on how to integrate js (and frontend libraries) are covered in the guide about [Modern JavaScript for Django Developers](https://www.saaspegasus.com/guides/).

Remember, to take this approach, do __not__ select the option `Include static files` in the project configuration.

The details of the setup for SaaS Pegasus are covered in two parts of the documentation:

1. For javascript in the section [The Front End](https://docs.saaspegasus.com/front-end/)
2. For css in the section [CSS](https://docs.saaspegasus.com/css/)

If the `Include static files` option was not in the project configuration, the project package comes complete with a `webpack-config.js` file configured for the chosen css frontend (e.g. bootstrap).

The next step is to set up a javascript environment for the project. That works similar to setting up a python environment.

The [pegasus docs](https://docs.saaspegasus.com/front-end/#prerequisites-to-building-the-front-end) mention `nvm`, the `npm` version manager (similar to `virtualenv`) but don't go into details.

`nvm` installation is described in detail on [this github webseite](https://github.com/nvm-sh/nvm) - the gist is to install `nvm` on the user level and switch `npm` versions on the go, depending on the directory one works in. To this end, the `nvm` installer script appends some lines to the `.bashrc` file.

```bash
cd ~/
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source .bashrc
```

After installation of `nvm` any instructions recommending running `npm` as root should instead be run as user.

How to use `nvm` is covered in the [Usage section of the official docs](https://github.com/nvm-sh/nvm#usage), but basically you can then install the latest version of node:

```bash
nvm install node
which nvm  # will point to ~/.nvm/versions/...
```

Once this is done, change into the project directory and run

```bash
npm install
```

This will read the included `package.json` file and install the javascript packages configured there.

Then you can proceed to build the javascript (and css) bundles using webpack via the preconfigure command `npm run dev` or `npm run dev-watch`. To build for production, run `npm run build`.

To configure the webpack build system look at the section [Long-term best practicse](https://docs.saaspegasus.com/front-end/#long-term-best-practices) of the pegasus front-end docs. This includes instructions on how the set up own javascript code, compile per-page etc.

In particular, the `webpack.config.js` file contains definitions for various `entry` (= source files) and `output` (= destination files). For example to add a special script `special.js` add the script to the `entry` part like this:

```js
module.exports = {
  entry: {
    // more entries
    'special': './assets/javascript/special.js',
  }
}
```

`assets/javascript/special.js` will then be compiled into `static/js/special-bundle.js`

The convention for the location of `entry` and `output` directories is to have the former in the `assets/javascript/` folder and the latter in the `static` folder.

The output js files need to be loaded on the respective html pages:

```html
{% raw %}
{% load static %}
{% block page_js %}
  <script src="{% static 'js/app-bundle.js' %}"></script>
  <script src="{% static 'js/special-bundle.js' %}"></script>
{% endblock page_js %}
{% endraw %}
```

### Passing data from django to javascript

This can be done either directly via django templates or via api calls. The first option is great for small amounts of data that is directly available in a django model, the second option is better for larger data sets (e.g. graphs) that can be fetched asynchronously from an api.

Her is [documentation on how to pass data to javascript via django templates](https://www.saaspegasus.com/guides/modern-javascript-for-django-developers/integrating-django-react/#passing-data-directly-with-djangos-template-system). Basically you build a dict in django and render it in the template using the `json_script` template tag.

The simplest way to pass single variable `mystringvar` is, to just render it directly in the template like this:

```html
{% raw %}
<script id="mystringvar" type="application/json">"{{ simulation.mystringvar }}"</script>
<script src="{% static 'rheo_three-bundle.js' %}"
{% endraw %}
```

The variable can then be retrieved in the javascript by

```js
const mystringvar = document.getElementById('mystringvar').textContent;
// for more complex json dicts:
//const mystringvar = JSON.parse(document.getElementById('mystringvar').textContent)
```

To include all this via webpack in an external script file and log `mystringvar` to the console as well as display it on the page as content of a div with the `id="mystringvar-div"` you can do the following:

```js
// assets/javascript/special.js
function foo() {
    // this is populated from django and rendered into the django template - we retrieve it here in js
    const mystringvar = document.getElementById('mystringvar').textContent;
    console.log(mystringvar);
}
foo();  // will print mystringvar to the javascript console

// will show mystringvar as inner text content of <div id="mystringvar-div"></div>
const mystringvar = document.getElementById('mystringvar').textContent;
document.getElementById('mystringvar-div').innerText = mystringvar
```

```html
{% raw %}
<div id="mystringvar-div"></div>
<script id="mystringvar" type="application/json">"{{ simulation.mystringvar }}"</script>
<script src="{% static 'rheo_three-bundle.js' %}"
{% endraw %}
```

### Customizing the theme

I'm exclusively using bootstrap 5 here since it is the most prevalent framework out there.

Pegasus uses sass via `webpack` (see previous chapter) and additionally splits the sass files into two parts:

+ css-framework independent styles: `/assets/styles/app/base.sass` --> `static/site-base.css`
+ css-framework overrides: `/assets/styles/app/bootstrap/` --> `static/site-bootstrap.css`

To override bootstrap styles, edit `assets/styles/site-bootstrap.scss` according to the bootstrap docs. Edits will only apply if you run `npm run dev` (`dev-watch`).

If you change any bootstrap scss variables, you need to do so before the boostrap css files are included or they will have no effect. Bootstrap class overrides go into the file after bootsrap code is included.

### User setup

Start a first user by starting the server `django runserver` (see below how to set up the `django`-shortcut) and going through the registration process. Then elevate the user to superuser with the script supplied by pegasus: `django promote_user_to_superuser yourname@example.com`. After that, you can log in and access `http://localhost:8000/admin`.

## Add a custom app

Pegasus organizes all apps in the `apps` directory. To add an app there specify a path when calling startapp:

```bash
mkdir apps/projects
django startapp projects apps/projects
```

The broad next steps are then:

+ Add a model
+ Create a view for the model
+ Create a `urls.py` file and add the url scheme
+ Include the new `apps/projects/urls.py` in the _project_ `urls.py`
+ Add a template in `templates/web/...`
+ Create an object via the admin interface
+ Start migrations and test

Lets go over the code in detail quickly. This uses generic class based views.

### Create a model

```python
# apps/project/models.py
from django.db import models

# Create your models here.

# basic project model
class Project(models.Model):
    name = models.CharField(
        max_length=128,
        help_text="Descriptive project name.",
        )
    created = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.name} (created: {self.created})"
```

### Create a view for the model

We just implement the DetailView here:

```python
# apps/project/views.py
from django.views.generic import DetailView
from .models import Project

# Create your views here.

class ProjectDetailView(DetailView):
    model = Project
    template_name = "web/project.html"
```

### Create and link urls

Create `urls.py`:

```python
# apps/project/urls.py
from django.urls import path
from .views import ProjectDetailView

urlpatterns = [
    path("project/<int:pk>", ProjectDetailView.as_view(), name="project"),
]
```

Now include this new `urls.py` in the _project_ file in `project_name/project_name/urls.py` at the end of the `urlpatterns`:

```python
urlpatterns = [
    # redirect Django admin login to main login page
    # ... path includes ..
    # myproject projects
    path("", include("apps.projects.urls"))
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### Add a template

Pegasus keeps templates in a global `template` directory. It's possible to either create a subdirectory there for the app, or if there are no namespace clashes, I can add the template directly in the existing structure. In this case I will just use a single Detail view, so I put the template in `templates/web/project.html`. To make things easier to maintain I break this down into sections:

```html
{% raw %}
<!-- templates/web/project.html -->
{% extends "web/base.html" %}
{% load static %}
{% block body %}
  {% include 'web/components/projects_detail.html' %}
{% endblock %}
{% endraw %}
```

Then in `projects_detail.html` I can show the details:

```html
{% raw %}
<!-- templates/web/components/projects_detail.html -->
{% load i18n %}
{% load static %}
<div class="container text-center">
  <div class="row justify-content-center">
    <div class="col col-md-8 align-self-center">
      <div class="row align-items-center justify-content-center py-4 border-dark border-top border-bottom border-opacity-25">
        <div class="col">
          Project: <h1>{{ project.name }}</h1>
          <p>Project creation time: {{ project.created}} </p>
          <p>Project primary key: {{ project.pk }} </p>
        </div>
      </div>
    </div>
  </div>
</div>
{% endraw %}
```

### Migrations and test

Run `django makemigrations && django migrate`, start the server `django runserver`. Then add a object instance from `127.0.0.1:8000/admin` and check the view according to the url-scheme under `127.0.0.1:8000/projects/1`.

## Misc. hints

### 'django' shortcut command

Instead of running `python manage.py ...` all the time we can abbreviate this to a `django` command by appending the following to our `.bashrc`:

```bash
# wrapper command for django's "python manage.py"
django(){
        python manage.py "$@"
}
```
