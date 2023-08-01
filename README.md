##How to run
[flask documentation](https://flask.palletsprojects.com/en/2.3.x/quickstart/)<br>
from main directory `microblog` set the `FLASK_APP` environment variable in terminal

`export FLASK_APP=microblog.py`

then from the terminal again type

`flask run`

The output from flask run indicates that the server is running on IP address 127.0.0.1, which is always the address of your own computer.

can also enter `http://localhost:5000/` or `http://localhost:5000/index` into the browser and get the same thing

instead of typing the env variable use python-dotenv package and save in `.flaskenv` file


####Use a class to store configuration variables
create a `config.py` file in the top-level directory to create the configuration Class

####Databases
The Flask-SQLAlchemy extension takes the location of the application's database from the `SQLALCHEMY_DATABASE_URI` configuration variable. Here I'm taking the database URL from the `DATABASE_URL` environment variable, and if that isn't defined, I'm configuring a database named app.db located in the main directory of the application, which is stored in the `basedir` variable.

The `SQLALCHEMY_TRACK_MODIFICATIONS` configuration option is set to `False` to disable a feature of Flask-SQLAlchemy that I do not need, which is to send a signal to the application every time a change is about to be made in the database.

db models
The `__repr__` method tells Python how to print objects of this class, which is going to be useful for debugging. You can see the `__repr__()` method in action below:
>>> from app.models import User <br>
>>> u = User(username='susan', email='susan@example.com') <br>
>>> u <br>
`<User susan>`


####DB Migration
Flask-Migrate exposes its commands through the flask command. The `flask db` sub-command is added by Flask-Migrate to manage everything related to database migrations. Create the migration repository for microblog by running `flask db init`:
follow this [link](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iv-database) for more details

The `flask db migrate` command does not make any changes to the database, it just generates the migration script. To apply the changes to the database, the `flask db upgrade` command must be used.
<br>When working with database servers such as `MySQL` and `PostgreSQL`, you have to create the database in the database server before running `upgrade`<br>
Note that Flask-SQLAlchemy uses a "snake case" naming convention for database tables by default. For the `User` model above, the corresponding table in the database will be named `user`. For a `AddressAndPhone` model class, the table would be named `address_and_phone`. If you prefer to choose your own table names, you can add an attribute named `__tablename__` to the model class, set to the desired name as a string.

Link for [Flask SQLAlchemy documentation](https://flask-sqlalchemy.palletsprojects.com/en/3.0.x/)
this will give more query options

use `flask shell` to test things in python without having to import these every time <br>
 `from app import app, db` <br>
 `from app.models import User, Post`<br>
  `app.app_context().push()`<br>

`(venv) $ flask shell` <br>
`>>> app`<br>
`<Flask 'app'>`

can add more commands in microblog.py with `@app.shell_context_processor`

###user logins
use `flask shell` and add user to db then test the login
`u = User(username='susan', email='susan@example.com')`<br>
`u.set_password('cat')`<br>
`db.session.add(u)`<br>
`db.session.commit()`<br>
added login, logout, register, validate passwords, etc

--------
####List of terminal commands thus far
- `flask run` : this runs the application
- `flask db init` : create migration repository
- `flask db migrate` : generates migration script
- `flask db upgrade` : applies migration (apply changes to db)
- `flask downgrade` : undoes last migration
- `flask shell` : allows you to test things in python without importing every time 
-----
####Action Items
1.  try to setup and connect to Postgres
----
###User Profile Pages
Using gravatar to create user images. Here is the [documentation](https://en.gravatar.com/site/implement/images) <br>
Use '_' prefix in a file name to indicate it is a sub-template '_name.html'

###Error Handling
when you are developing, enable debug mode, a mode in which Flask outputs a really nice debugger directly on your browser. To activate debug mode set the following environment variable
`export FLASK_ENV=development`
<br>That did not enable debug so used 
`flask run --debug` instead
If you run flask run while in debug mode, you can then work on your application and any time you save a file, the application will restart to pick up the new code.

enable the email logger when the application is running without debug mode, which is indicated by `app.debug` being `True`,

to test the email logger from 2nd terminal type <br>
`python -m smtpd -n -c DebuggingServer localhost:8025`<br>
go to 1st terminal and set `export MAIL_SERVER=localhost and MAIL_PORT=8025`
<br> run in production mode and can see the error printed to screen of 2nd terminal
<br>A second testing approach for this feature is to configure a real email server. Below is the configuration to use your Gmail account's email server:

export MAIL_SERVER=smtp.googlemail.com<br>
export MAIL_PORT=587<br>
export MAIL_USE_TLS=1<br>
export MAIL_USERNAME=<your-gmail-username><br>
export MAIL_PASSWORD=<your-gmail-password><br>

To make the logging more useful, lower the logging level to the INFO category, both in the application logger and the file logger handler. Logging categories are DEBUG, INFO, WARNING, ERROR and CRITICAL in increasing order of severity.

###Adding Followers
added complexity to db and implemented unit testing<br>
four tests that exercise the password hashing, user avatar and followers functionality in the user model. The `setUp()` and `tearDown()` methods are special methods that the unit testing framework executes before and after each test respectively.

a little hack to prevent the unit tests from using the regular database used for development. By setting the `DATABASE_URL` environment variable to `sqlite://`, I change the application configuration to direct SQLAlchemy to use an in-memory SQLite database during the tests.

The `setUp()` method then creates an application context and pushes it. This ensures that the Flask application instance, along with its configuration data is accessible to Flask extensions. 

The `db.create_all()` call creates all the database tables. This is a quick way to create a database from scratch that is useful for testing. For development and production create database tables through database migrations.<br>
to execute `python tests.py`<br>
every time a change is made to the application, re-run the tests to make sure the features that are being tested have not been affected. Also, each time another feature is added to the application, a unit test should be written for it.

###Pagination
after I process the form data, I end the request by issuing a redirect to the home page. So, why the redirect? It is a standard practice to respond to a `POST` request generated by a web form submission with a redirect. This helps mitigate an annoyance with how the refresh command is implemented in web browsers. All the web browser does when you hit the refresh key is to re-issue the last request. If a `POST` request with a form submission returns a regular response, then a refresh will re-submit the form. Because this is unexpected, the browser is going to ask the user to confirm the duplicate submission, but most users will not understand what the browser is asking them. But if a `POST` request is answered with a redirect, the browser is now instructed to send a `GET` request to grab the page indicated in the redirect, so now the last request is not a `POST` request anymore, and the refresh command works in a more predictable way. See [Post/Redirect/Get](https://en.wikipedia.org/wiki/Post/Redirect/Get)

One interesting aspect of the url_for() function is that you can add any keyword arguments to it, and if the names of those arguments are not referenced in the URL directly, then Flask will include them in the URL as query arguments.

###Email Support
Flask-Mail supports some features such as Cc and Bcc lists. [Flask-Mail Documentation](https://pythonhosted.org/Flask-Mail/) <br>
The `verify_reset_password_token()` is a static method, which means that it can be invoked directly from the class. A static method is similar to a class method, with the only difference that static methods do not receive the class as a first argument. <br>
Asynchronous Emails<br>
If you are using the simulated email server that Python provides  sending an email slows the application down considerably. All the interactions that need to happen when sending an email make the task slow, it usually takes a few seconds to get an email out, and maybe more if the email server of the addressee is slow, or if there are multiple addressees.<br>



What I really want is for the `send_email()` function to be asynchronous. when this function is called, the task of sending the email is scheduled to happen in the background, freeing the `send_email()` to return immediately so that the application can continue running concurrently with the email being sent.

Python has support for running asynchronous tasks, actually in more than one way. The `threading` and `multiprocessing` modules can both do this. Starting a background thread for email being sent is much less resource intensive than starting a brand new process, so going to go with that approach:

There are many extensions that require an application context to be in place to work, because that allows them to find the Flask application instance without it being passed as an argument. The reason many extensions need to know the application instance is because they have their configuration stored in the `app.config` object. This is exactly the situation with Flask-Mail. The `mail.send()` method needs to access the configuration values for the email server, and that can only be done by knowing what the application is. The application context that is created with the `with app.app_context()` call makes the application instance accessible via the `current_app` variable from Flask.

###Facelift
CSS frameworks [bootstrap](https://getbootstrap.com/) and [examples](https://getbootstrap.com/docs/3.3/getting-started/#examples)

###Timezones
[moment.js](https://momentjs.com/) is a small open-source JavaScript library that takes date and time rendering to another level, as it provides every imaginable formatting option, and then some. Flask-Moment, is a small Flask extension that makes it very easy to incorporate moment.js into your application.
The `scripts` block in `base.html` is where JavaScript imports are included<br>
`ISO 8601` format:<br> `{{ year }}-{{ month }}-{{ day }}T{{ hour }}:{{ minute }}:{{ second }}{{ timezone }}`
with UTC timezones, the last part is always going to be Z, which represents UTC in the ISO 8601 standard.<br>`t = moment('2021-06-28T21:45:23Z')`<br>
`moment('2021-06-28T21:45:23Z').format('L')`<br>
`"06/28/2021"`<br>
`moment('2021-06-28T21:45:23Z').format('LL')`<br>
`"June 28, 2021"`<br>
`moment('2021-06-28T21:45:23Z').format('LLL')`<br>
`"June 28, 2021 2:45 PM"`<br>
`moment('2021-06-28T21:45:23Z').format('LLLL')`<br>
`"Monday, June 28, 2021 2:45 PM"`<br>
`moment('2021-06-28T21:45:23Z').format('dddd')`<br>
`"Monday"`<br>
`moment('2021-06-28T21:45:23Z').fromNow()`<br>
`"7 hours ago"`<br>
`moment('2021-06-28T21:45:23Z').calendar()`<br>
`"Today at 2:45 PM"`<br>

###Language Translator
[flask babel](https://flask-user.readthedocs.io/en/v0.6/internationalization.html)<br>
Created commands to run from cli<br>
To add a new language, you use:<br>
`flask translate init <language-code>`<br>
To update all the languages after making changes to the _() and _l() language markers:<br>
This can be done manually or with something like [poedit](https://poedit.net/)<br>
`flask translate update`<br>
And to compile all languages after updating the translation files:<br>
`flask translate compile`<br>

###Ajax
[ajax](https://en.wikipedia.org/wiki/Ajax_(programming)/)
<br>Implementing live automated translations requires a few steps.  
- need a way to identify the source language of the text to translate
- need to know the preferred language for each user,  to show a "translate" link only for posts written in other languages.<br> 
When a translation link is offered and the user clicks on it,  need to send the Ajax request to the server, and the server will contact a third-party translation API. Once the server sends back a response with the translated text, the client-side javascript code will dynamically insert this text into the page.
<br>need to set up the [microsoft translator api](https://www.microsoft.com/en-us/translator/business/) the google one is no longer free

###Refactoring - Blueprints
no longer using a global variable for the application but use an application factory function instead to create the function at runtime.  The function accepts a configuration object as an argument and returns a flask application instance.<br>Write the blueprint in a separate Python package, then have a component that encapsulates the elements related to specific feature of the application
in the `email.py` need to do a small trick
 - need to access the real application instance that is stored inside the proxy object, and pass that as the app argument. The `current_app._get_current_object()` expression extracts the actual application instance from inside the proxy object, so that is what is passed to the thread as an argument.
 
 ----
 to recreate virtual env <br>
 `pip install -r requirements.txt`
 ------
 ###Elasticsearch
 start from terminal<br>`elasticsearch`<br> in python console <br>
 `>>> from elasticsearch import Elasticsearch`<br>
 `>>> es = Elasticsearch('http://localhost:9200')`<br>
 `>>> es.index(index='my_index', id=1, body={'text': 'this is a test'})`<br>
 add as many as you like just change `id` number<br>
 to issue a free-form search <br>
 `>>> es.search(index='my_index', body={'query': {'match': {'text': 'this test'}}})`<br>
 [For search params](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)<br>
 to delete the index<br>
 `>>>es.indices.delete(index='my_index')`<br>
 when setting up for the full-text search want to make it easy to switch from elasticsearch to other search engines so using a generic way.
 Add `__searchable__` class attribute that lists fields that need to be included in the index
 
 to test in console<br>
 set up an application context
with `app.app_context()`<br>
`>>> from flask import current_app`<br>
`>>> from app import create_app`<br>
`>>> app = create_app()`<br>
[2023-07-31 15:27:16,647] INFO in __init__: Microblog startup<br>
`>>> app.app_context().push()`<br>
`>>> from app.search import add_to_index, remove_from_index, query_index`<br>
`>>> from app.models import Post`<br>
`>>> for post in Post.query.all():`<br>
`...     add_to_index('posts',post)`<br>
`>>> query_index('posts', 'one two three four five', 1, 100)`<br>
`>>> app.elasticsearch.indices.delete(index='posts')`<br>
----
Integrating searches with SQLAlchemy<br>
1.  Elasticsearch results come as list of numeric IDs which is inconvenient.  Need to replace the numbers with models to pass down as templates for SQLAlchemy to render them 
2.  the solution requires the application to explicitly issue indexing as posts are added or removed.  This can cause a bug with indexing and the 2 databases can get out of sync

SOLUTION:  have these calls to be triggered automatically as changes are made in the SQLAlchemy db<br>
The problem of replacing the IDs with objects can be addressed by creating a SQLAlchemy query that reads those objects from the database<br>Will drive updates to the Elasticsearch index from SQLAlchemy [events](https://docs.sqlalchemy.org/en/20/core/event.html)<br>
EXAMPLE:  have a function in the app invoked by SQLAlchemy and in that function apply same updates made on SQLAlchemy session to Elasticsearch index<br>
Create a `SearchableMixin` that when attached to a model acts as a glue layer between SQLAlchemy and Elasticsearch worlds thus providing solutions to the 2 problems above
<br>The SQLAlchemy query that retrieves the list of objects by their IDs is based on a CASE statement from the SQL language, which needs to be used to ensure that the results from the database come in the same order as the IDs are given, see below<br>
`WHERE`<br>
   `x_field IN ('f', 'p', 'i', 'a') ...`<br>
`ORDER BY`<br>
   `CASE x_field`<br>
      `WHEN 'f' THEN 1`<br>
      `WHEN 'p' THEN 2`<br>
      `WHEN 'i' THEN 3`<br>
      `WHEN 'a' THEN 4`<br>
      `ELSE 5 -- fallback for values not inside the IN clause. eg : x_field = 'b'`<br>
   `END, id`<br>
   This is important because the Elasticsearch query returns results sorted from more to less relevant<br>
   From console, set up an application context
with `app.app_context()`, see above <br>
`>>>query, total = Post.search('water quack booo fccvvv', 1, 5)`<br>
`>>>total`<br>
3<br>
`>>>query.all()`<br>
`[<Post fccvvv>, <Post booo hooooo>, <Post water is blue>]`
---
creating a search form will use `GET` instead of `POST` since that is the request used for URLs. Will also put it in the navigation bar so it is on all pages of application. Use `q` as the argument in teh query string<br>
A form that has a text field, the browser will submit the form when you press Enter <br>
Forms have CSRF protection added by default, with the inclusion of a CSRF token that is added to the form via the form.hidden_tag() construct in templates. For clickable search links to work, CSRF needs to be disabled, so set meta to {'csrf': False} so that Flask-WTF knows that it needs to bypass CSRF validation for this form.<br>
This `g` variable provided by Flask is a place where the application can store data that needs to persist through the life of a request