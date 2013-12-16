# hackernews site cloned in django
Exercise completed from Tuts+
##  1 intro
Django is the largest python web framework
##  2 virtualenv
virtualenv and virtualenvwrapper

    mkvirtualenv hackernews
##  3 hello world
    workon hackerners
    pip install django
    pip install ipython  #for shell
    django-admin.py startproject helloworld
    cd helloworld
    subl .             # opens in Sublime text
    chmod +x manage.py  # makes file executable # doesn't work
    python manage.py runserver   
Python maps urls to views.
You can create multiple apps within one project.
##  4 DB backend: create model
    django-admin.py startproject hackernews
    cd hackernews/
    setvirtualenvproject   # sets working dir as project dir
    cdproject  # virtenv command to go to project dir 
django docs model layer section, Field Types link  
Use this for setting up the data model in models.py  
For authentication: django docs scroll down to authentication link  

    python manage.py startapp stories  
Set up the db in settings.py file  

    python manage.py syncdb  
Create super user, email, password  

    python manage.py dbshell  # to open db app 
    .tables  # shows all tables created
    .schema auth_group #(any table) shows sql that created table
    .q # to quit dbshell
##  5 DB backend: using the ORM to perform CRUD on db
Create and save to db

    python manage.py shell  # opens shell 
    In [1]: from stories.models import *  # gets model
    In [2]: s = Story()        # creates an instance of model
    In [3]: s.title
    Out[3]: ''            # no title yet
    In [4]: from django.contrib.auth.models import User
    In [5]: User.objects.all()        
    Out[5]: [<User: kurt>]      # only user so far
    In [6]: moderator = User.objects.all()[0]    # sets active moderator
    In [7]: s.moderator = moderator             #sets moderator for s
    In [8]: s.save()                  # saves a story to db
    In [9]: Story.objects.all()            # shows db contents
    Out[9]: [<Story: Story object>]     

More
    
    In [1]: from django.contrib.auth.models import User
    In [2]: from stories.models import *
    In [3]: moderator = User.objects.all()[0]
    In [4]: moderator.story_set.all()
    Out[4]: [<Story: >]
    In [6]: s = Story.objects.all()[0]
    In [7]: s
    Out[7]: <Story: >
    In [8]: s.title = 'My First Story'
    In [9]: s
    Out[9]: <Story: My First Story>
    In [10]: s.save()
    In [11]: moderator.story_set.all()
    Out[11]: [<Story: My First Story>]
    In [12]: s.created_at
    Out[12]: datetime.datetime(2013, 12, 4, 20, 53, 39, 563137, tzinfo=<UTC>)
    In [13]: s.updated_at
    Out[13]: datetime.datetime(2013, 12, 4, 21, 5, 54, 245723, tzinfo=<UTC>)
    
    In [14]: s = Story.objects.create(title='My Second Story', moderator=moderator)
    In [15]: Story.objects.all()
    Out[15]: [<Story: My First Story>, <Story: My Second Story>]
    In [16]: moderator.story_set.all()
    Out[16]: [<Story: My First Story>, <Story: My Second Story>]
    
    In [17]: moderator.story_set.create(title='My Third Story')
    Out[17]: <Story: My Third Story>
    In [18]: moderator.story_set.all()
    Out[18]: [<Story: My First Story>, <Story: My Second Story>, <Story: My Third Story>]
    
    In [19]: s = Story(title='My Fourth Story', moderator=moderator)
    In [20]: moderator.story_set.add(s)
    In [21]: moderator.story_set.all()
    Out[21]: [<Story: My First Story>, <Story: My Second Story>, <Story: My Third Story>, <Story: My Fourth Story>]

list comprehension
    
    In [23]: [s.created_at for s in Story.objects.all()]
    Out[23]:
    [datetime.datetime(2013, 12, 4, 20, 53, 39, 563137, tzinfo=<UTC>),
     datetime.datetime(2013, 12, 4, 21, 8, 15, 228729, tzinfo=<UTC>),
     datetime.datetime(2013, 12, 4, 21, 9, 40, 7153, tzinfo=<UTC>),
     datetime.datetime(2013, 12, 4, 21, 11, 29, 321743, tzinfo=<UTC>)]

Filter (gte is >=)

    In [24]: Story.objects.filter(created_at__gte='2013-12-4 21:9+00:00')
    Out[24]: [<Story: My Third Story>, <Story: My Fourth Story>]

    In [26]: Story.objects.filter(title__startswith='My F')
    Out[26]: [<Story: My First Story>, <Story: My Fourth Story>]

    In [27]: Story.objects.filter(created_at__gte='2013-12-4 21:9+00:00').filter(title__startswith='My F')
    Out[27]: [<Story: My Fourth Story>]

    In [28]: q = Story.objects.filter(title__startswith='My F')

    In [29]: q
    Out[29]: [<Story: My First Story>, <Story: My Fourth Story>]

    In [30]: print q.query
    SELECT "stories_story"."id", "stories_story"."title", "stories_story"."url", "stories_story"."points", "stories_story"."moderator_id", "stories_story"."created_at", "stories_story"."updated_at" FROM "stories_story" WHERE "stories_story"."title" LIKE My F% ESCAPE '\'

Foreign key example

    In [33]: Story.objects.filter(moderator__username='kurt')
    Out[33]: [<Story: My First Story>, <Story: My Second Story>, <Story: My Third Story>, <Story: My Fourth Story>]

    In [36]: User.objects.filter(story__title='My First Story')
    Out[36]: [<User: kurt>]

https://docs.djangoproject.com/en/1.6/topics/db/queries/  #query the db
##  6 the Admin app
Login to admin

    python manage.py runserver
    localhost:8000/admin    # in browser

Register app with the admin app to see stories in admin page
    
create admin.py file in stories directory
    
    from django.contrib import admin
    from stories.models import Story

    class StoryAdmin(admin.ModelAdmin):
	list_display = ('__unicode__', 'domain', 'moderator', 'created_at', 'updated_at')
	list_filter = ('created_at', 'updated_at')
	search_fields = ('title', 'moderator__username', 'moderator__first_name', 'moderator__last_name')

	fieldsets = [
		('Story', {
			'fields': ('title', 'url', 'points')
		}),
		('Moderator', {
			'classes': ('collapse',),
			'fields': ('moderator',)
		}),
		('Change History', {
			'classes': ('collapse',),
			'fields': ('created_at', 'updated_at')
		})
	]
	readonly_fields = ('created_at', 'updated_at')

    admin.site.register(Story, StoryAdmin)

Django Documentation > Admin section > sidebar 'Model Admin objects' link

##  7 first view
load some stories with the script   

    $ python loader.py
Created the main view for app.

Open views.py in stories directory
put in logic, but also temp html


##  8 adding a template
build a new template and a base template
Added a view function
added a custom filter

python ./manage.py syncdb --migrate


##  9 static files  - css
in production use apache  to serve css and js and static files  
in dev mode use the static files app  
4 ways to get context into project and have access to static variables. (yeah right)  
Collect static files

    python manage.py collect static
 
 Last 3 episodes:
 create a view function
 How to use templates to separate the visual representation from the code
 How to set up and use django's staticfiles app
 stataic files in dev and how to ready them for deployment
 
##  10 forms and submission page
added forms.py, updated views.py and both urls.py  
added csrf authentication  
Fairly easy to add a form for user input.

##  11 user authentication
 updated views.py  
 added login and logout functionality including login submit and user info on page header
 
##  12 using Ajax
Bug with csrf piece
 
##  13 wraping up the vote view
Fix 2 problems:  
1. user votes when not logged in - auth login required layup  
2. multi vote - limit voters to 1 vote each story.

2 episodes: added functionality for voting on stories 

##  14 DRYing up the code 
Replaced string urls with named tags on both url pages and all the html pages.  
 

## south db update
    pip install south
Add 'south', to settings.py>INSTALLED_APPS
    python ./manage.py syncdb
    python ./manage.py convert_to_south <table_name>
Make your changes to the model
    python ./manage.py schemamigration blog --auto
    python ./manage.py migrate blog
 
 
 
 