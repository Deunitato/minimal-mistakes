---
published: true
---
Another part of my internship is to learn flask for python. Apparently we need to be know abit of html,css and of course python. Although python is not my main language, I do have some sort of knowledge of it. (Although my python class are not the best)

For the first part of the search, I follow a tutorial on youtube in the creation of a "Task Master" web application.

# Udemy Course

## Starting
- Importing
`from flask import Flask`
- Initialising
`app = Flask(__name__)`

- Standard decorators
```
@app.route("/")
```
> This decorator tells where the function should return to. Without this, the program will not know where to return the string. 
/ signifies the route.


- Functions
```
def welcome():
	return 'This is first Flask app`
```

- Running the app `app.run()`

Running `python app.py` will create a server at the local webserver at port 5000

> Note, if we specify the route as "/potato", we have to go into `<ipaddress>/potato` for it to work
  
We can do something like this:

```
@app.route('potato')
def welcome():
	return 'Hello world'

@app.route('/')
def index():
	return 'This is my root page!'

@app.route('/bob')
def bob():
	return 'hi bob'
    
app.run()
```


## Handling HTTP request (GET,POST)

By default, we are doing a GET request.
By using POST, we are sending data to the server
We can do this by defining a method

```
@app.route('/method',methods=['GET', 'POST'])
def method():
	if request.method == 'POST':
    	return "This is a post request!"
    else:
    	return "This might be a GET request"

```

But we realised that this method does not ask for an input. Thus running the ipadd/method will give us the fetch request.

> We can use a form to allow for input

Note that because we only specify get and post as our methods, other methods would not be allowed.

## Folder Hierachy
- create a template folder which we can store our html files.
- CSS stuff should exist in other folders (static)



# Youtube - Making an todo List
## Starting
- Python 
- Pip
- Virtualenv

1. Install virtual env:

`pip3 install virtualenv`

2. Start

`virtaulenv env`

`source \env\Scripts\activate.bat`

We can see the (env) that signifies that we are in virtual enviorment

3. Install flask

`pip3 install flask flask-sqlalchemy`

4. Create `app.py`

Sample flask application:

app.py:
```
      from flask import Flask

      app = Flask(__name__) #reference this file

      @app.route('/') #url string of our route
      def index(): #define fn for this route
          return "Hello world!"

      if __name__ == "__main__":
          app.run(debug=True) #debug will be true
 ```   

Running this will allow us to start a webpage that says "Hello world" in `localhost:5000`

## Templates and static content

1. import `render_template`

2. Change the index() function:
```
		def index():
           return render_template('index.html')
```
3. Create an index.html


- This will load the template instead of a string

### Template inheritence
- Creating a master that is a skeleton of what each page should look like and insert only code where we need. 
- Ensures that we only need to write what is relevant
1. Create skeleton `base.html`
Using Jinja2 syntax
- This will specify where we are going to fill up the infomation using other html files

2. Change our index.html

![Flask_1.PNG]({{site.baseurl}}/img/Flask_1.PNG)

- We do not need to repeat ourselves each time 
- We can include css by including it in the base html 

Including stylesheet:

`<link rel = "stylesheet" href = "{{url_for('<location of css>',  filename = 'css/main.css')}}"`

## SQLAlchemy

### Create

1. Import 

`from flask_sqlalchemy import SQLAlchemy`

2. Add the config under 

```
app = Flask()..'

      	app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite : /// test.db' #Tells the app where our database is added
        db = SQLALchemy(app) #init the database
        
        
        class Todo(db.Model):
			id = db.Column(db.Integer, primary_key= True) #Reference the id of each entry
            content = db.Column(db.String(200), nullable = False) # cannot be empty and has a size of 200
            date_create = db.column(db,DateTime, default = datetime.utcnow) # Optional book keeping, must import datatime
            
            def __repr__(self): #everytime we create a new class, it will run this and return it's id
            	return '<Task %r>' % self.id
```
        
> Four / means absolute path

3. Set up the database

//Not covered here

4. Add lines to index.html
```
		<div class = "content">
        	<h1> Task Master </h1>
            
            <table>
            	<tr>
                   <th>Task</th>
                   <th>Added</th>
                   <th>Actions</th>
                </tr>
                <tr>
                	<td></td>
                    <td></td>
                    <td>
                    	<a href = ""> Delete </a>
                        <br>
                        <a href = "">Update</a>
                    </td>
                 </tr>
              </table>
         </div>
  ```                  
 - This will create a table with Delete and Update  
5. Add methods

- We are going to post and get our db

Edit the route

`@app.route('/', methods = ['POST','GET'])`
- Get is by default
- Now our app accepts two methods

We can now update our index.html to include a form
```
		<form action="/" method = "POST">
 			<input type = "text" name= "content" id="content">
            <input type = "submit" value="Add Task">`
```
- Edit the index() method
```
		def index():
			if request.method =='POST':
            	return 'Hello'
            else:
            	return render_template('index.html')
```
> Remember to insert request in the import file

- We are able to click on the 'submit' button

We can update our request.method
```
			task_content = reuqest.form['content"]
            new_task = ToDo(content = task_content)
            
            try:
            	db.session.add(new_task) #add the task to database
                db.session.commit() #commit to db
                return redirect('/') # return back to our index page
            except:
                return 'There was an issue adding your task'
```                
Update our else method:
```
			task = Todo.query.order_by(Todo.date_created).all() #grabs all the task in db by date created 
            return render_template('index.html', tasks = tasks) # parse it as parameters
```
Add Jinja2 into our index.html:
![Flask_2.PNG]({{site.baseurl}}/img/Flask_2.PNG)
From our prev example excerpt:
![Flask_3.PNG]({{site.baseurl}}/img/Flask_3.PNG)

> The creation is now done, we can create task now


### Delete 

> Remember the id in the ToDo class

1. Add a new route for delete

`@app.route('/delete/<int:id>')`

2. Create our method
```
		def delete(id):
			task_to_delete = Todo.query.get_or+404(id) #attempt to get the task by id else it will 404
            
            try:
            	db.session.delete(task_to_delete)
                db.session.commit()
                return redirect('/')
            except:
            	return 'There was a prob deleting the task'
```
3. Update our html file

`<a href = ""> Delete </a>`

edit it to

`<a href = "/delete/{{task.id}}"> Delete </a>`


### Update
1. Add a new route for Update

`@app.route('/update/<int:id>', methods = ['GET', 'POST'])`

2. Create our method
```
		def update(id):
        	task = Todo.query.get_or_404(id) # get the task
			if request.method == 'POST':
				#Update logic here
                task.content = request.form['content'] 
               try:
                db.session.commit()
                return redirect('/')
            except:
            	return 'There was a prob update' 
                
            else:
            	return render_template('update.html', task=task)
 ```           
the task

3. Update our html file

`<a href = ""> Update </a>`

edit it to

`<a href = "/update/{{task.id}}"> Delete </a>`


4. Create a new update.html to serve the update

> Hint: We can copy the form from the index.html
```
		<form action="/update/{{task.id}}" method = "POST">
 			<input type = "text" name= "content" id="content" value = "{{task.content}}">
            <input type = "submit" value="Add Task">
         </form>
```


# Findings/reflection
- Create a route each time a new method is made
- Route directs the user to another page
- We can use classes to objectify the items used
- For each method, create a new function to make code cleaner
- We can use jinja2 to add "logic code" into our html file

# References
[Youtube tutorial - learn flask for python](https://www.youtube.com/watch?v=Z1RJmh_OqeA)
[Udemy Flask For Beginners](https://www.udemy.com/course/python-flask-for-beginners/learn/lecture/8281338#content)