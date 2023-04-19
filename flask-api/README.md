<h1>Flask RESTful CRUD API</h1>

<img src=https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OoT87Kw_gdGXrhGepWC3sA.jpeg>

This past weekend I was getting ready to live stream on my [YouTube channel](https://www.youtube.com/c/dennisIvy) and decided to write a guide for myself to use during the stream. This guide turned into an article called [“My First CRUD App With Fast API”.](https://medium.com/@dennisivy/my-first-crud-app-with-fast-api-74ac190d2dcc)

Well today I decided to continue this mini series and create the same API using Flask. Unlike Fast API this is NOT the first time I am using Flask. Nevertheless, I’m writing this guide for you crazies that will watch this stream, or I guess just anybody that finds this article, however it is that you end up here…

You can find the stream recording [here](https://www.youtube.com/watch?v=xeuW-IitQTQ),

and the source code [here](https://github.com/divanov11/Simple-REST-API-with-Flask).

If “here” isn't a link yet that means the I haven't streamed yet, so check back later.

Also, if you are trying to read this article without following the stream it may end up being a little difficult to read. This is meant to be more so of a stream guide and less of a detailed article.

Now that you have been warned, here's a quick look at the structure of this guide:

- Building API’s with Flask
- Let’s setup our project
- Adding Flask RESTful
- Create Read, Update & Delete
- Database Configuration
- Modeling data
- Working with the database
- Response marshaling (aka serializing data)
- Finish up CRUD
<h2>Building API’s with Flask</h2>
There are several ways we can go about building API’s with Flask but in this particular example we will use the [Flask RESTful](https://flask-restful.readthedocs.io/en/latest/) with Flask. Flask RESTful is an extension for flask that adds extra features and gives us the tools we need for building API’s. For those of you that know Django think of it as what [Django REST Framework](https://www.django-rest-framework.org/) is to [Django.](https://www.djangoproject.com/)

So technically you can build an API with just flask without any extra packages but Flask RESTful will make things easier so that's what I will use.

Let’s setup our project
We’ll start by creating an empty folder for our project, opening it in our text editor and adding a app.py file.

```
flaskapp
   app.py
```
Then we’ll install Flask..

```
pip install flask
```
and finally, we’ll create a simple app and start our server just to make sure things are setup right. We’ll start by importing the Flask class and then creating and instance of our WSGI application by using the __name__ shortcut.

After our first route we will call app.run() to start our server when the script is executed.

```
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return '<h1>Hello, World!</h1>'
if __name__ == '__main__':
    app.run(debug=True)
```
Now to run our server..

```
(env) C:\Users\Dennis Ivy\Desktop\FlaskApp> app.py
Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://127.0.0.1:5000 (Press CTRL+C to quit)        
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 681-583-112
```

You’re application should now be running on port 5000.

Another way we can run our server is to use the “flask run” command. For this we will first have to tell our terminal which application to work with by exporting the FLASK_APP environment variable and then calling flask run.

For windows CMD

```
set FLASK_APP=main
flask run
 * Running on http://127.0.0.1:5000/
Debug mode:
```

The flask run command can do more than just start the development server. By enabling debug mode, the server will automatically reload if code changes, and will show an interactive debugger in the browser if an error occurs during a request.
```
> set FLASK_ENV=development
> flask run
```
<h2>Adding Flask RESTful</h2>
Alright now its time to create our API using FlaskRESTful.

We’ll start by first installing FlaskRESTful, then modifying our routes to return some JSON data.

```
pip install flask-restful
```
In main.py import Resource & API from flask_restful, then create an instance of the Api class.

We will update our routes to use class base views now and inherit from Resource. These class based views will help us make our API more restful by separating our logic by the type of http method. For now each view will only process “get” requests

```
from flask import Flask
from flask_restful import Resource, Api
app = Flask(__name__)
api = Api(app)
fakeDatabase = {
    1:{'name':'Clean car'},
    2:{'name':'Write blog'},
    3:{'name':'Start stream'},
}
class Items(Resource):
    def get(self):
        return fakeDatabase
class Item(Resource):
    def get(self, pk):
        return fakeDatabase[pk]
api.add_resource(Items, '/')
api.add_resource(Item, '/<int:pk>')
if __name__ == '__main__':
    app.run(debug=True)
```

<h2>Create Read, Update & Delete</h2>
Lets start by handling post requests so we can add data to our placeholder database.

<b>POST</b>

First we want to import “request” so we can read the data that's sent over in the request body

```
from flask import Flask, request
class Items(Resource):
    def get(self):
        return fakeDatabase 
    def post(self):
        data = request.json
        itemId = len(fakeDatabase.keys()) + 1
        fakeDatabase[itemId] = {'name':data['name']}
        return fakeDatabase
```

PUT

```
class Item(Resource):
    def get(self, pk):
        return fakeDatabase[pk]
    def put(self, pk):
        data = request.json
        fakeDatabase[pk]['name'] = data['name']
        return fakeDatabase
```

Delete

```
class Item(Resource):
    def get(self, pk):
        return fakeDatabase[pk]
    def put(self, pk):
        data = request.json
        fakeDatabase[pk]['name'] = data['name']
        return fakeDatabase
    def delete(self, pk):
        del fakeDatabase[pk]
        return fakeDatabase
```

<h2>Database Configuration</h2>
Time to move on from our place holder database and configure something more practical. For this we will use SQLite.

We wont focus on file structure here so lets just keep adding to our main.py file for simplicity.

We will be using SQLAlchemy as our ORM so lets start by importing that first:

```
pip install flask-sqlalchemy
```

Now in our main.py file:

```
from flask_sqlalchemy import SQLAlchemy
......
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///todo.db'
db = SQLAlchemy(app)
```

That’s it for now. We have the basic configuration we need but before we create the database we want to create some models to establish what kind of tables and columns we need.

<h2>Modeling our data</h2>
Still in our main.py file lets create a class that inherits from db.model.

```
class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(200), nullable=False)
    
    def __repr__(self):
        return self.name
```

Once you have this model. Lets go to our terminal and actual create this database.

```
(env) C:\Users\Dennis Ivy\Desktop\FlaskApp> python
>>> from app import db
>>> db.create_all()
>>> exit()
```

Calling db.create_all() will create our database based on the configuration we set in app.config[‘SQLALCHEMY_DATABASE_URI’] and will also create a table based on the model “Task” that we have set.

```
flaskapp
   app.py
   todo.db
```

<h2>Working with the database</h2>
Now that we have our database, its time to learn how to add, modify and query data.

Let start with making a request to get all the items in the database.

Querying items in a table

```
class Items(Resource):
    def get(self):
        tasks = Task.query.all()
        return tasks
    ...
class Item(Resource):
    def get(self, pk):
        task = Task.query.filter_by(id=pk).first()
        return task
    ...
```

If we have now data in our database yet we should see an empty array return from this request. We are about to face an issue when we add some data so lets give that a test.

Adding data

```
class Items(Resource):
    def get(self):
        tasks = Task.query.all()
        return tasks
def post(self):
        data = request.json
        task = Task(name=data['name'])
        db.session.add(task)
        db.session.commit()
        tasks = Task.query.all()
        # itemId = len(fakeDatabase.keys()) + 1
        # fakeDatabase[itemId] = {'name':data['name']}
        return tasks
```
If we send a request from the frontend here everything will work fine. The issue here is when we try returning the list of tasks now.

<h2>Response marshaling (aka serializing data)</h2>
Now that we are returning back some objects we need to do something called “response marshaling” or “serializing”, which will essentially convert our objects into a more simplified version of our data so it can be rendered out as JSON data.

To serialize our data we will first have to import the “marshal_with” decorator and “fields” from flask_restful and then create a new datastructure for our task object.

```
from flask_restful import Resource, Api, marshal_with, fields
...
taskFields = {
    'id':fields.Integer,
    'name':fields.String,
 }
```

Now place the “@marshal_with” decorator above any view function that will return a task instance or a list of task instances.

```
class Items(Resource):
    @marshal_with(taskFields)
    def get(self):
        ...
    
    @marshal_with(taskFields)
    def post(self):
        ...
class Item(Resource):
    @marshal_with(taskFields)
    def get(self, pk):
        ...
    @marshal_with(taskFields)
    def put(self, pk):
        ...
    @marshal_with(taskFields)
    def delete(self, pk):
        ...
```

Finish up CRUD
Ok so now that we took care serializing our data, lets move on to working with our database.

<strong>Modifying data</strong>

```
class Item(Resource):
    ...
    @marshal_with(taskFields)
    def put(self, pk):
        data = request.json
        task = Task.query.filter_by(id=pk).first()
        task.name = data['name']
        db.session.commit()
        return task
    ...
```

<strong>Deleting data</strong>

```
class Item(Resource):
    ...
    @marshal_with(taskFields)
    def delete(self, pk):
         task = Task.query.filter_by(id=pk).first()
         db.session.delete(task)
         db.session.commit()
         tasks = Task.query.all()
         return tasks
```

Ok that about sums it up!

<button border-radius=10%>Flask</button> <button border-radius=10%>Flask Restful</button> <button border-radius=10%>API</button> <button border-radius=10%>Web Development</button>
