.. _tango-too-label:

Doing the Tango with Rango! 
===========================

Hopefully, you will have been able to complete the exercises given the workflows we provided, if not, or if you need a little help checkout snippets of code and use them within your version of Rango.



.. #########################################################################

List Categories on Each Page
----------------------------

Creating a Category List Template
.................................
Create a ``category_list.html`` template so that it looks like the code below:

.. code-block:: html
	
	<ul class="nav nav-list">
	    {% if cat_list %}
	        {% for cat in cat_list %}
	            <li><a href="/rango/category/{{ cat.url }}">{{ cat.name }}</a></li>
	        {% endfor %}
	    {% else %}
	        <li>No categories at present.</li>
	   {% endif %}
	</ul>

Updating the Base Template
..........................
In the sidebar ``<div>``, add ``{% include "rango/category_list.html" %}`` so that the ``base.html`` contains:

.. code-block:: html
	
	<div class="well sidebar-nav">
	   {% block sidebar %}
	   {% endblock %}
	   <div id="cats">
	       {% if cat_list %}
	           <ul class="nav nav-list"><li>Category List</li></ul>
	           {% include 'rango/category_list.html' with cat_list=cat_list %}
	       {% endif %}
	    </div>
	</div>

Creating Get Category List Function
...................................
In ``rango/views.py``, create a function called ``get_category_list()`` that returns the list of categories.

.. code-block:: python
	
	def get_category_list():
	    cat_list = Category.objects.all()
	    
	    for cat in cat_list:
	        cat.url = encode_url(cat.name)
	    
	    return cat_list

Updating Views
..............
Then call this function in each of the views that you want to display the category list in the sidebar, and pass the list into the context dictionary. For example, to have the categories showing on the index page, alter the ``index()`` view as follows:
	
.. code-block:: python
	
	def index(request): 
	    context = RequestContext(request)
	    cat_list = get_category_list()
	    
	    context_dict['cat_list'] = cat_list
	    
	    ....
	    
	    return render_to_response('rango/index.html', context_dict, context)
	
Note: to add the category list to all the other pages, you will need to do some refactoring to pass in all the context variables.
	
.. #########################################################################	

Searching Within a Category Page
--------------------------------
Rango aims to provide users with a helpful directory of page links. At the moment, the search functionality is essentially independent of the categories. It would be nicer however to have search integrated into category browsing. Let's assume that a user will first browse their category of interest first. If they can't find the page that they want, they can then search for it. If they find a page that is suitable, then they can add it to the category that they are in. Let's tackle the first part of this description here.

We first need to remove the global search functionality and only let users search within a category. This will mean that we essentially decommission the current search page and search view. After this, we'll need to perform the following.

Decommissioning Generic Search
..............................
Remove the generic *Search* link from the menu bar by editing the ``base.html`` template. You can also remove or comment out the URL mapping in ``rango/urls.py``.

Creating a Search Form Template
...............................
Take the search form from ``search.html`` and put it into the ``category.html``. Be sure to change the action to point to the ``category()`` view as shown below.

.. code-block:: html
	
	<div class="container-fluid">
	    <p>Search for a page.</p>
	    <form class="span8 form-search" id="search_form" method="post" action="/rango/category/{{ category_name_url }}/">
	        {% csrf_token %}
	        <input type="text" class="input-long search-query"  name="query" value="{{ category_name }}" id="query" />
	        <button type="submit" class="btn btn-success" name="submit" value="Search">Search</button>
	    </form>
	</div>

Also include a ``<div>`` to house the results underneath.

.. code-block:: html
	
	<div class="container-fluid">
	    {% if result_list %}
	        <!-- Display search results in an ordered list -->
	        <ol>
	            {% for result in result_list %}
	            <li>
	                <strong><a href="{{ result.link }}">{{ result.title }}</a></strong><br />
	                <p>{{ result.summary }}</p>
	            </li>
	            {% endfor %}
	        </ol>
	    {% else %}
	        <br/>
	        <p>No results found</p>
	    {% endif %}
	</div>

Updating the Category View
..........................
Update the category view to handle a HTTP ``POST`` request (i.e. when the user submits a search) and inject the results list into the context. The following code demonstrates this new functionality.
	
.. code-block:: python
	
	def category(request, category_name_url):
	    # Request our context
	    context = RequestContext(request)

	    # Change underscores in the category name to spaces.
	    # URL's don't handle spaces well, so we encode them as underscores.
	    category_name = decode_url(category_name_url)

	    # Build up the dictionary we will use as out template context dictionary.
	    context_dict = {'category_name': category_name, 'category_name_url': category_name_url}

	    cat_list = get_category_list()
	    context_dict['cat_list'] = cat_list

	    try:
	        # Find the category with the given name.
	        # Raises an exception if the category doesn't exist.
	        # We also do a case insensitive match.
	        category = Category.objects.get(name__iexact=category_name)
	        context_dict['category'] = category
	        # Retrieve all the associated pages.
	        # Note that filter returns >= 1 model instance.
	        pages = Page.objects.filter(category=category).order_by('-views')

	        # Adds our results list to the template context under name pages.
	        context_dict['pages'] = pages
	    except Category.DoesNotExist:
	        # We get here if the category does not exist.
	        # Will trigger the template to display the 'no category' message.
	        pass

	    if request.method == 'POST':
	        query = request.POST['query'].strip()
	        if query:
	            result_list = run_query(query)
	            context_dict['result_list'] = result_list

	    # Go render the response and return it to the client.
	    return render_to_response('rango/category.html', context_dict, context)

.. #########################################################################

View Profile 
------------
To add the view profile functionality, undertake the following steps.

Creating the Profile Template
.............................
First, create a new template called ``profile.html``. In this template, add the following code.

.. code-block:: html
	
	{% extends "rango/base.html" %}

	{% block title %}Profile{% endblock %}

	{% block body_block %}
	<div class="hero-unit">
	    <h1> Profile <h1> <br/>
	    <h2>{{ user.username }}</h2>
	    <p>Email: {{ user.email }}</p>
        
	    {% if userprofile %}
	        <p>Website: <a href="{{ userprofile.website }}">{{ userprofile.website }}</a></p>
	        <br/>
	        {% if userprofile.picture %}
	            <img src="{{ userprofile.picture.url }}"  />
	        {% endif %}
	    {% endif %}
	</div>
	{% endblock %}


Creating Profile View
......................
Create a view called ``profile`` and add the following code.

.. code-block:: python
	
	from django.contrib.auth.models import User
	
	@login_required
	def profile(request):
	    context = RequestContext(request)
	    cat_list = get_category_list()
	    context_dict = {'cat_list': cat_list}
	    u = User.objects.get(username=request.user)
	
	    try:
	        up = UserProfile.objects.get(user=u)
	    except:
	        up = None
	
	    context_dict['user'] = u
	    context_dict['userprofile'] = up
	    return render_to_response('rango/profile.html', context_dict, context)

Mapping the Profile View and URL
................................
Create a mapping between the URL ``/rango/profile`` and the ``profile()`` view. Do this by updating the ``urlpatterns`` tuple in ``rango/urls.py`` so that it includes the following entry.

.. code-block:: python
	
	url(r'^profile/$', views.profile, name='profile'),

Updating the Base Template
..........................
In the ``base.html`` template, update the code to put a link to the profile page in the menu bar.

.. code-block:: html
	
	{% if user.is_authenticated %}
	    <li><a href="/rango/profile">Profile</a></li>
	{% endif %}	
	
.. #########################################################################

Track Page Click Throughs
-------------------------
Currently, Rango provides a direct link to external pages. This is not very good if you want to track the number of times each page is clicked and viewed. To count the number of times a page is viewed via Rango you will need to perform the following steps.

Creating a URL Tracking View
............................
Create a new view called ``track_url()`` in ``/rango/views.py`` which takes a parameterised HTTP ``GET`` request (i.e. ``rango/goto/?page_id=1``) and updates the number of views for the page. The view should then redirect to the actual URL.

.. code-block:: python	
	
	def track_url(request):
	    context = RequestContext(request)
	    page_id = None
	    url = '/rango/'
	    if request.method == 'GET':
	        if 'page_id' in request.GET:
	            page_id = request.GET['page_id']
	            try:
	                page = Page.objects.get(id=page_id)
	                page.views = page.views + 1
	                page.save()
	                url = page.url
	            except:
	                pass
	
	    return redirect(url)

Be sure that you import the ``redirect()`` function to ``views.py`` if it isn't included already!

.. code-block:: python
	
	from django.shortcuts import redirect

Mapping URL
...........
In ``/rango/urls.py`` add the following code to the ``urlpatterns`` tuple.

.. code-block:: python
	
	url(r'^goto/$', views.track_url, name='track_url'),


Updating the Category Template
...............................
Update the ``category.html`` template so that it uses ``rango/goto/?page_id=XXX`` instead of providing the direct URL for users to click.

.. code-block:: html
	
	{% if pages %}
	<ul>
	    {% for page in pages %}
	    <li>
	        <a href="/rango/goto/?page_id={{page.id}}">{{page.title}}</a>
	        {% if page.views > 1 %}
	            - ({{ page.views }} views)
	        {% elif page.views == 1 %}
	            - ({{ page.views }} view)
	        {% endif %}
	    </li>
	    {% endfor %}
	</ul>
	{% else %}
	<strong>No pages currently in category.</strong><br/>
	{% endif %}

Here you can see that in the template we have added some control statements to display ``view``, ``views`` or nothing depending on the value of ``page.views``.

Updating Category View
......................
Since we are tracking the number of click throughs you can now update the ``category()`` view so that you order the pages by the number of views. To confirm this works, click on a link and refresh the category view - the link you clicked should jump up the rankings.
