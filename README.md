# Notes on Jekyll


## Resources

- [Jekyll documentation](https://jekyllrb.com/docs/)

## Getting set up

These instructions assume Jekyll is installed

1. make the directory where you want the site to be. ie `mkdir my-jelyll-site`

2. in the directory, add an `index.html` file

3. run jekyll   
  - `jekyll build` will output the site to a directory called `_site`
  - use `jekyll serve` for development. this will continuously update the site as you work
  
4. view the site at `http://localhost:4000`

5. it's a good idea, though not necessary to include a *config.yml* file in the root directory

## congfig.yml

The *config.yml* file contains site settings. (including global variables). If you mane any changes here, you will have to stop `jekyll serve` and start it again for them to take effect. Be careful, it's `.yml` so proper syntax spacing is critical.

## Liquid

Jekyll uses the *liquid* templating language developed by Shopify. Liquid is similar, but not identical to *twig* which is used for Drupal 8 templating

### Objects

Objects tell liquid where content goes. and are denoted with curly braces. ie. `{{` and `}}`.

For example, `{{ page.title }}` will output a variable called title.

### Tags

Tags are used for logic. and use a single curly bracket with a % sign. `{% tag %}` ie:

```
  {% if page.content %}
    <div>
    {{ page.content }}
    </div>
  {% endif %}

```

Jekyll has some [built in tags](https://jekyllrb.com/docs/liquid/tags/) and you can also use any of the [tags available to liquid](https://shopify.github.io/liquid/tags/control-flow/).

### Filters
Filters change the output of a liquid object and are seperatd by a `|`. ie:
`{{ "hi" | capitalize }}` will output "Hi"

Jekyll has it's own [list of filters](https://jekyllrb.com/docs/liquid/filters/) and you can use any filters [available to liquid](https://shopify.github.io/liquid/filters/abs/)

## Front Matter
Front matter is `YAML` that sets variables for the page. Set them at the top of the document between two rows of `---`. ie
```
  ---
  my_number: 5
  ---
```
so you could do something like

```
  ---
  title: Home
  ---
  <html>
    <head>
      <title>{{ page.title }}</title>
    </head>
  </html>
```
you can also use a *front matter* on markdown pages. Jekyll will generate the html accordingly.

## Creating layouts
Layouts are templates that wrap around content. create them in a directory called `_layouts` ie `_layouts/default.html`

A simple layout would look like 

```
  <!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ page.title }}</title>
  </head>
  <body>
    {{ content }}
  </body>
</html>
```

`{{ content }}` is a special variable that jekyll uses to show the content of a page.

to include the layout in your html file, use a *front matter* ie. 

```
---
layout: default
title: Home
---
<h1>{{ "Hello World!" | downcase }}</h1>
```

## using *includes*
Includes let you include content from other files. For example, you probably want the navigation the same everywhere.

1. create directory `_includes`

2. make the file you want to include, say `_includes/navigation.html`

3. add the *include* to the template, ie:    
  ```
  <body>
    {% include navigation.html %}
    {{ content }}
  </body>
  ```
## data files

Jekyll supports loading data from YAML, JSON, and CSV files located in a `_data` directory.

For a navigation array:

1. create `_data/navigation.yml`

2. format as follows:
```
- name: Home
  link: /
- name: About
  link: /about.html
```
[more information on formatting yaml](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

Jekyll makes this data file available to you at `site.data.navigation`. 

now we can iterate over the data file. In `_includes/navigation.html` use the following

```
<nav>
  {% for item in site.data.navigation %}
    <a href="{{ item.link }}" {% if page.url == item.link %}style="color: red;"{% endif %}>
      {{ item.name }}
    </a>
  {% endfor %}
</nav>
``` 

## Assets
Jekyll handles assets nicely and will even compile SASS for you. 

user the following structure for assets

```
.
├── assets
|   ├── css
|   ├── images
|   └── js
```

Link to your assets in the layout file, ie: in `_layouts/default.html` add `<link rel="stylesheet" href="/assets/css/styles.css">`

### Getting a sassmap to show up in code inspector

In testing there was a big issue with how Jekyll compiles SASS. Jekyll does not generate a source map and will over-write it on compile. This means that css is almost impossible to debug. 

so for development, do the following:

1. at `assets/css/styles.scss` disable the *front matter* so it isn't tracked by jekyll
2. at `assets/css/styles.scss` comment out `//@import "main";` and replace with `@import "../../_sass/main";`
3. in `config.yml` exclude `_sass`
4. in `config.yml` exclude `assets/css/styles.scss`
5. if it's running, stop `jekyll serve`
6. start `jekyll serve`

You can now use whatever compiler you like and it will generate a sass map that can be used in the inspector.

This should work fine, but if you want to do things the *jekyll way*, reverse these steps for production.


### if using codekit
There are some things you will need to setup in codekit to work with jekyll

1. go to general settings and add `_site` to the ignore list
2. in languages, go to markdown and under markdown output, ignore markdown files on change/build (because jekyll does this)
3. in languages, go to sass and make sure sass compiles to the same folder as the source

## things to watch out for 

1. if you have any *liquid* code in a markdown or html document, jekyll will try to render it. In most cases this is fine, but if you actually want to see the code (such as in this file) you need to make sure the file is ignored in config.yml

