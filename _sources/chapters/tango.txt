.. _tango-label:

Making Rango Tango! Exercises
=============================

So far we have been adding in different pieces of functionality to Rango. We've been building up the application in this manner to get you familiar with the Django Framework, and to learn about how to construct the various parts of a website that you are likely to make in your own projects. Rango however at the present moment is not very cohesive. In this chapter, we challenge you to improve the application and its user experience by bringing together functionality that we've already implemented alongside some awesome new additions.

To make Rango more coherent and integrated it would be nice to add the following functionality.

* Integrate the browsing and searching within categories, i.e.:
	* provide categories on every page;
	* provide some way to search through categories (see Chapter :ref:`ajax-label`); and
	* instead of have a disconnected search page, let users search for pages within a category so that they can then add these pages to the category (see Chapter :ref:`ajax-label`)
	
* Provide services for Registered Users, i.e.:
	* let users view their profile;
	* let users edit their profile; and
	* let users see the list of users and their profiles.

* Track the click throughs of Categories and Pages, i.e.:
	* count the number of times a category is viewed;
	* count the number of times a page is viewed via Rango; and
	* collect likes for categories (see Chapter :ref:`ajax-label`).

.. note:: We won't be working through all of these tasks right now. Some will be taken care of in Chapter :ref:`ajax-label`, while some will be left to you to complete as additional exercises.

Before we start to add this additional functionality we will make a todo list to plan our workflow for each task. Breaking tasks down into sub-tasks will greatly simplify the implementation and mean that we are attacking each one with a clear plan. In this chapter, we will provide you with the workflow for a number of the above tasks. From what you have learnt so far, you should be able to fill in the gaps and implement most of it on your own. The following chapter however includes the code walkthrough, along with notes on how we have implemented each task.


Providing Categories on Every Page
----------------------------------

It would be nice to show the different categories that users can browse through on any page, and not just the index page. There are a number of ways that this can be achieved, but given what we have learnt so far, we could undertake the following workflow:

* Create a ``category_list.html`` template and migrate the template code for presenting the categories list from  ``index.html`` to this new template. This template won't be a full HTML document, but a portion of one which we can include in other templates.
* Use the ``{% include "rango/category_list.html" %}`` to now include this template code into the ``base.html`` template within the sidebar. This means that all pages will now include categories, assuming the ``cat_list`` data is passed through in the context dictionary.
	* To handle the scenario where ``cat_list`` isn't available, add the conditional ``{% if cat_list %}`` to ensure that only templates providing ``cat_list`` attempt to render this component.
* Migrate the code that gathers the list of categories from the ``index()`` view and place it into your ``category()`` view. While it's really tempting to simply copy and paste, there's a much better way to go about this!
	* Create a helper function called ``get_category_list()`` within ``views.py`` that returns the list of categories.
	* We can then call this function whenever we want to get the category list to be presented in the sidebar. This saves a lot of code repetition!
* Pass the category list data (``cat_list``) through to the template to complete the process.

Searching Within a Category Page
--------------------------------
Rango aims to provide users with a helpful directory of page links. At the moment, the search functionality is essentially independent of the categories. It would be nicer however to have search integrated into category browsing. Let's assume that a user will first browse their category of interest first. If they can't find the page that they want, they can then search for it. If they find a page that is suitable, then they can add it to the category that they are in. Let's tackle the first part of this description here.

We first need to remove the global search functionality and only let users search within a category. This will mean that we essentially decommission the current search page and search view. After this, we'll need to perform the following steps.

* Remove the generic *Search* link from the menu bar.
* Take the search form and results template markup from ``search.html`` and place it into ``category.html``.
* Update the category view to handle a HTTP ``POST`` request. The view must then include any search results in the context dictionary for the template to render.

View Profile
------------
Another useful feature to add is a profile page, where users can view details of their Rango profile. Undertake the following steps to add this functionality.

* First, create a template called ``profile.html``. In this template, add in the fields associated with the user profile and the user (i.e. username, email, website and picture).
* Create a view called ``profile()``. This view will obtain the data required to render the user profile template.
* Map the URL ``/rango/profile/`` to your new ``profile()`` view.
* In the base template add a link called *Profile* into the menu bar, preferably on the right-hand side with other user-related links. This should only be available to users who are logged in (i.e. ``{% if user.is_authenticated %}``).
	
Track Page Click Throughs
-------------------------
Currently, Rango provides a direct link to external pages. This is not very good if you want to track the number of times each page is clicked and viewed. To count the number of times a page is viewed via Rango you will need to perform the following steps.

* Create a new view called ``track_url()``, and map it to URL ``/rango/goto/``.
* The ``track_url()`` view will examine the HTTP ``GET`` request parameters and pull out the ``page_id``. The HTTP ``GET`` requests will look something like ``/rango/goto/?page_id=1``.
	* Your view should then be able to find the relevant ``Page`` model for the selected page, and add 1 to the associated ``views`` field.
	* The view will then redirect the user to the specified URL using Django's ``redirect`` method.
	* In the scenario where no parameters are in the HTTP ``GET`` request for ``page_id``, or the parameters do not return a ``Page`` object, redirect the user to Rango's homepage.
* Update the ``category.html`` so that it uses ``/rango/goto/?page_id=XXX`` instead of using the direct URL.

Hint
....
If you're unsure of how to retrieve the ``page_id`` *querystring* from the HTTP ``GET`` request, the following code sample should help you.

.. code-block:: python
	
	if request.method == 'GET':
	    if 'page_id' in request.GET:
	        page_id = request.GET['page_id']

Always check the request method is of type ``GET`` first, then you can access the dictionary ``request.GET`` which contains values passed as part of the request. If ``page_id`` exists within the dictionary, you can pull the required value out with ``request.GET['page_id']``.