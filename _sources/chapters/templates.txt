Working with Templates
======================
So far we've created several Django HTML templates for different pages in the application. You've probably already noticed that there is a lot of repeated HTML code in these templates.

While most sites will have lots of repeated structure (i.e. headers, sidebars, footers, etc) repeating the HTML in each template is a not good way to handle this. So instead of doing the same cut and paste hack job, we can minimize the amount of repetition in our code base by employing *template inheritance* provided by Django's Template Language.

The basic approach to using inheritance in templates is as follows.

#. Identify the re-occurring parts of each page that are repeated across your application (i.e. header bar, sidebar, footer, content pane)
#. In a *base template*, provide the skeleton structure of a standard page along with any common content (i.e. the copyright notice that goes in the footer, the logo and title that appears in the section), and then define a number of *blocks* which are subject to change depending on which page the user is viewing.
#. Create specific templates - all of which inherit from the base template - and specify the contents of each block.

Reoccurring HTML and The Base Template
--------------------------------------
Given the templates that we have created so far it should be pretty obvious that we have been repeating a fair bit of HTML code. Below we have abstracted away any page specific details to show the skeleton structure that we have been repeating within each template.

.. code-block:: html
	
	<!DOCTYPE html>
	
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	
	    <body>
	        <!-- Page specific content goes here -->
	    </body>
	</html>

Let's make this our base template, for the time being, and save it as ``base.html`` in Rango's ``templates`` directory (e.g. ``templates/rango/base.html``). 

.. note:: You should always aim to extract as much reoccurring content for your base templates. While it may be a bit more of a challenge for you to do initially, the time you will save in maintenance of your templates in the future far outweighs the initial overhead. Think about it: would you rather maintain one copy of your markup or multiple copies?

.. warning:: Remember that your page ``<!DOCTYPE html>`` declaration absolutely must be placed on the first line for your page! Not doing so will mean your markup will not comply with the W3C HTML5 guidelines.

Template Blocks
---------------
Now that we've identified our base template, we can prepare it for our inheriting templates. To do this, we need to include a Template Tag to indicate what can be overridden in the base template - this is done through the use of *blocks*.

Add a ``body_block`` to the base template as follows:

.. code-block:: html
	
	<!DOCTYPE html>
	
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	
	    <body>
	        {% block body_block %}{% endblock %}
	    </body>
	</html>

Recall that standard Django template commands are denoted by ``{%`` and ``%}`` tags. To start a block, the command is ``block <NAME>``, where ``<NAME>`` is the name of the block you wish to create. You must also ensure that you close the block with the ``endblock`` command, again enclosed within Django template tags.

You can also specify 'default content' for your blocks, if you so desire. Our ``body_block`` defined above presently has no default content associated with it. This means that if no inheriting template were to employ the use of ``body_block``, nothing would be rendered - as shown in the code snippet below.

.. code-block:: html
	
	<!DOCTYPE html>
	
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	
	    <body>
	        
	    </body>
	</html>

However, we can overcome this by placing default content within the block definition, like so:

.. code-block:: html
	
	<!DOCTYPE html>
	
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	
	    <body>
	        {% block body_block %}This is body_block's default content.{% endblock %}
	    </body>
	</html>

If a template were to inherit from the base template without employing the use of ``body_block``, the rendered outcome would now look something like the markup shown below.

.. code-block:: html
	
	<!DOCTYPE html>
	
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	
	    <body>
	        This is body_block's default content.
	    </body>
	</html>

Hopefully this all makes sense - and for now, we'll be leaving ``body_block`` blank by default. All of our inheriting templates will be making use of ``body_block``. You can place as many blocks in your templates as you so desire. For example, you could create a block for the page title, meaning you can alter the title of each page while still inheriting from the same base template.

Blocks are a really powerful feature of Django's template system to learn more about them check out the `official Django documentation on templates <https://docs.djangoproject.com/en/1.5/topics/templates/#id1>`_.

Abstracting Further
...................
Now that you have an understanding of Django blocks, let's take the opportunity to abstract our base template a little bit further. Reopen the ``base.html`` template and modify it to look like the following.

.. code-block:: html
	
	<!DOCTYPE html>
	
	<html>
	    <head>
	        <title>Rango - {% block title %}How to Tango with Django!{% endblock %}</title>
	    </head>

	    <body>
	        <div>
	            {% block body_block %}{% endblock %}
	        </div>
	        
	        <hr />
	        
	        <div>
	            <ul>
	            {% if user.is_authenticated %}
	                <li><a href="/rango/restricted/">Restricted Page</a></li>
	                <li><a href="/rango/logout/">Logout</a></li>
	                <li><a href="/rango/add_category/">Add a New Category</a></li>
	            {% else %}
	                <li><a href="/rango/register/">Register Here</a></li>
	                <li><a href="/rango/login/">Login</a></li>
	            {% endif %}
	                
	                <li><a href="/rango/about/">About</a></li>
	            </ul>
	        </div>
	    </body>
	</html>

We introduce two new features into the template.

* The first is a new Django template block, ``title``. This will allow us to specify a custom page title for each page inheriting from our base template. If an inheriting page does not make use of this feature, the title is defaulted to ``Rango - How to Tango with Django!``
* We also bring across the list of links from our current ``index.html`` template and place them into a HTML ``<div>`` tag underneath our ``body_block`` block. This will ensure the links are present across all pages inheriting from the base template. The links are preceded by a *horizontal rule* (``<hr />``) which provides a visual separation between the ``body_block`` content and the links. 

Also note that we enclose the ``body_block`` within a HTML ``<div>`` tag - we'll be explaining the meaning of the ``<div>`` tag in Chapter :ref:`css-course-label`. Our links are also converted to an unordered HTML list through use of the ``<ul>`` and ``<li>`` tags.

Template Inheritance
--------------------
Now that we've created a base template with a block, we can now update the templates we have created to inherit from the base template. For example, let's refactor the template ``rango/category.html``.

To do this, first remove all the repeated HTML code leaving only the HTML and Template Tags/Commands specific to the page. Then at the beginning of the template add the following line of code:

.. code-block:: html
	
	{% extends 'rango/base.html' %}

The ``extends`` command takes one parameter, the template which is to be extended/inherited from (i.e. ``rango/base.html``). We can then modify the ``category.html`` template so it looks like the following complete example.

.. note:: The parameter you supply to the ``extends`` command should be relative from your project's ``templates`` directory. For example, all templates we use for Rango should extend from ``rango/base.html``, not ``base.html``.

.. code-block:: html
	
	{% extends 'rango/base.html' %}
	
	{% block title %}{{ category_name }}{% endblock %}
	
	{% block body_block %}
	    <h1>{{ category_name }}</h1>
	    
	    {% if pages %}
	    <ul>
	        {% for page in pages %}
	        <li><a href="{{ page.url }}">{{ page.title }}</a></li>
	        {% endfor %}
	    </ul>
	    {% else %}
	        <strong>No pages currently in category.</strong>
	    {% endif %}
	    
	    {% if user.is_authenticated %}
	       <a href="/rango/category/{{category_name_url}}/add_page/">Add a Page</a>
	    {% endif %}
	    
	{% endblock %}

Now that we inherit from ``base.html``, all that exists within the ``category.html`` template is the ``extends`` command, the ``title`` block and the ``body_block`` block. You don't need a well-formatted HTML document because ``base.html`` provides all the groundwork for you. All you're doing is plugging in additional content to the base template to create the complete HTML document which is sent to the client's browser.

.. note:: 

 	Templates are very powerful and you can even create your own template tags. Here we have shown how we can minimise the repetition of structure HTML in our templates.

	However, templates are can also to minimise code within views too. For example, if you had a list of items generated from a database table that you would like to be presented on each page, it is then possible to construct templates that make the call to a specific view to render that part of the part. This saves you from calling the functions to retrieve the data and passing that data to the template for every view that displays that list.
	
	To learn more about the extensive functionality offered by Django's template language, check out the official `Django documentation on templates <https://docs.djangoproject.com/en/1.5/topics/templates/>`_. 

Exercises
---------
Now that you've worked through this chapter, we've got several exercises for you to work through. After completing them, you'll be a Django templating messiah.

* Update all other existing templates within Rango's repertoire to extend from the ``rango/base.html`` template. Follow the same process as we demonstrated above. Once completed, your templates should all inherit from ``base.html``, as demonstrated in Figure :num:`fig-rango-template-inheritance`. While you're at it, make sure you remove the links from our ``index.html`` template. We don't need them anymore! You can also remove the link to Rango's homepage within the ``about.html`` template.
* Convert the restricted page to use a template. Call the template ``restricted.html``, and ensure that it too extends from our ``base.html`` template.
* Add another link to our growing link collection that allows users to navigate back to Rango's homepage from anywhere on the website.

.. warning:: Remember to add ``{% load static %}`` to the top of each template that makes use of static media. If you don't, you'll get an error! Django template modules must be imported individually for each template that requires them - *you can't make use of modules included in templates you extend from!*

.. _fig-rango-template-inheritance:

.. figure:: ../images/rango-template-inheritance.svg
	:figclass: align-center
	
	A class diagram demonstrating how your templates should inherit from ``base.html``.