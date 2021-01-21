# HackHERSCrudAppDemo

### Before starting this workshop, make sure you have done the prerequisities! 

## Read
We will be using Express, a framework used in conjunction with Node.js for web development. To use it, we have to require it.
In server.js:
```
const express = require('express')
const app = express()
```
We're building a web app! So, we need to be able to render and view in our browser. For this, we have our local host. 
```
app.listen(3000, function())
{
  console.log("running on port 3000")
})
```
Right now we're not displaying anything on our web page. Let's display something. 
```
app.get('/', (req, res) => {
  res.send('Hello World')
})
```
So far, we have written all of this in server.js

If you save and reload, you should see Hello World display (in ugly font) on your page. 

Since we're going to be actually displaying something cool and are making a web app, let's go to our index.html. This will help us form and serve up how our page will look. 

```
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html')
})
```
The above is also required to be in server.js

Let's set up our index.html. 

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>myApp</title>
</head>
<body>
  <h1>Let's be productive!</h1>
</body>
</html>
```

## Create
We want to be able to CREATE entries. This can only be done with POST requests that the server receives. Therefore, we have our form! 


```
<form action="/todos" method="POST">
  <input type="text" placeholder="task" task="task">
  <input type="text" placeholder="deadline" deadline="deadline">
  <button type="submit">Submit</button>
</form>
```

Our POST request is handled by a post function. After saving and running node server.js again, Hello! should be in the command line. 
```
app.post('/todos', (req, res) => {
  console.log('Hello!')
})
```

Remember how we installed body parser? We're going to use it now. This (body parser) is a middleware and helps us read in the data in the form. Add in the requirement for bodyParser in server.js
```
const express = require('express')
const bodyParser= require('body-parser')
const app = express()

// Make sure you place body-parser before your CRUD handlers!
app.use(bodyParser.urlencoded({ extended: true }))

// All your handlers here...
app.get('/', (req, res) => {/*...*/})
app.post('/todos', (req, res) => {/*...*/})
```

Since we are now using a middleware to handle receiving data from the form, let's log the actual body of the form entries. Let's update our app.post from before. 
```
app.post('/quotes', (req, res) => {
  console.log(req.body)
})
```

In the console, you should see something that resembles a database entry. 

Now that we have a form, we of course need a way of storing our data to delete and update it! This is where MongoDB comes into play. 

In server.js let's add a requirement for MongoClient, which will help us connect to our database. 
```
const express = require('express')
const app = express()

const bodyParser = require('body-parser')
const MongoClient = require('mongodb').MongoClient
```
Remember the long connection string I said to note down (if you could) in the prereq? We're going to need this now. It's fine if you forgot it. Log into your MongoDB Atlas account, go to your cluster, click connect, and copy the string. Remember, we need to replace user, password, and db name in the string. User and password should be the credentials I told you to note down. For now, we can make our db name just test. 

```
MongoClient.connect('mongodb-connection-string', (err, client) => {
  // ...
})
```
I mentioned that for the time being we could keep our db name in the connection string to test. Let's change it to what we actually want to call our database. Change it to hackhers! Now, your MongoClient code in server.js should look like this:

```
MongoClient.connect(connectionString, { useUnifiedTopology: true })
  .then(client => {
    console.log('Connected to Database')
    const db = client.db('hackhers')
  })
```

Ok, so we established a db variable. Now, how do we actual connect to our database? You see the then function? We need to put all our app.put, get etc into this. Let's update our MongoClient code again. 
```
MongoClient.connect(/* ... */)
  .then(client => {
    // ...
    const db = client.db('hackhers')
    app.use(/* ... */)
    app.get(/* ... */)
    app.post(/* ... */)
    app.listen(/* ... */)
  })
  .catch(console.error)
 ```
  
Just like in our car example, we had an inventory, we need an analogous item here. So, we create a collection. The inventory is our database and a collection is the individual cars. Let's name our collection todos, since we are creating a to-do app. Again, we make updates to MongoClient. 

```
MongoClient.connect(/* ... */)
  .then(client => {
    // ...
    const db = client.db('hackhers')
    const todoCollection = db.collection('todos')

    // ...
  })

```
Now we want to add into this collection, our different to-do items or tasks. 
```
app.post('/todos', (req, res) => {
  todoCollection.insertOne(req.body)
    .then(result => {
      console.log(result)
    })
    .catch(error => console.error(error))
})
```
If you go to your MongoDB cluster and click on the collections button, you should see an update! Our database name, hackhers, as well as the new collection name, todos. Enter something into the form. Refresh MongoDB. You should see an entry in the collection (a document, for finer terminology). 

Now, if you noticed, you can see that after hitting enter on an entry in the form, the page seems to keep loading. Let's fix that. In server.js:

```
app.post('/todos', (req, res) => {
  todoCollection.insertOne(req.body)
    .then(result => {
      res.redirect('/')
    })
    .catch(error => console.error(error))
})
```
### Our next step is to display our to-do items on our page. 
In server.js, let's update:
```
app.get('/', (req, res) => {
  db.collection('todos').find().toArray()
    .then(results => {
      console.log(results)
    })
    .catch(error => console.error(error))
  // ...
})
```
Now when you enter something into the form, it will reload properly, add to our collection in MongoDB, and we have logged it in our console. To actually see it on the webpage, we have a few more steps. 

Remember when we installed ejs before? We're going to be using that now. 
Let's go to server.js and before all your other handlers...

```
app.set('view engine', 'ejs')
\\..app.use...etc 
```

Now let's go into our index.ejs file. We will be doing virtually the same thing as index.html here, but this is going to be our main way of rendering so we will be making the rest of such visual changes here. 
In index.ejs:
```
<!-- index.ejs -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>To-Do App</title>
  </head>

  <body>
    <h1>Get it Done!</h1>

    <form action="/todos" method="POST">
      <input type="text" placeholder="task" task="task" />
      <input type="text" placeholder="deadline" deadline="deadline" />
      <button type="submit">Submit</button>
    </form>
  </body>
</html>
```
Remember, we have to render it!
Go back to server.js:
```
app.get('/', (req, res) => {
  db.collection('todos').find().toArray()
    .then(/* ... */)
    .catch(/* ... */)
  res.render('index.ejs', {})
})

```
To properly render our entries on the page, we need to put the to-do tasks in index.ejs
Let's go back to server.js and make that update
```
app.get('/', (req, res) => {
  db.collection('todos').find().toArray()
    .then(results => {
      res.render('index.ejs', { todos: results })
    })
    .catch(/* ... */)
})

```
Now to display the quotes using index.ejs, we need to use a for loop. In index.ejs:
```
<h2> ToDos </h2>

<ul class="todos">
  <!-- Loop through todos -->
  <% for(var i = 0; i < todos.length; i++) {%>
    <li class="todo">
      <!-- Output name from the iterated task object -->
      <span><%= todos[i].task %></span>:
      <!-- Output quote from the iterated deadline object -->
      <span><%= todos[i].deadline %></span>
    </li>
  <% } %>
</ul>
```

## Update
Navigate to index.ejs. We will be making some functionalities for updating our entries. 
```
<section data-position="update-task">
      <div>
        <h2>Add some inspiration!</h2>
        <p>
          Replace a stressful task with some inspriational quotes. 
        </p>
        <button id="update-button">Replace a Task</button>
      </div>
    </section>
```

Now let's go into our public folder and edit main.js
Let's add in all the functionalities for our new update, delete, etc functions we will need. 
```
const update = document.querySelector('#update-button')
const deleteButton = document.querySelector('#delete-button')
const messageDiv = document.querySelector('#message')
```
Now go into server.js: we need to add in more app.use function. 
```
app.use(express.static('public'))
```
Now, to wire it all up, let's add main.js to index.ejs like so:
```
<body>
  <!-- ... -->
  <script src="/main.js"></script>
</body>
```
We're not done with our update portion yet. How is our update going to work exactly? Let's go over that.
Add this to main.js
```
update.addEventListener('click', _ => {
  // Send PUT Request here
})
```

We need a PUT request to our browser, which is done through a Fetch. 
```
update.addEventListener('click', _ => {
  fetch('/todos', {
    method: 'put'
  })
})
```
As maybe you can tell from the set up of our inputs in our database so far, we are sending in JSON data. What does this mean?
We need to tell the servers this!

```
update.addEventListener('click', _ => {
  fetch('/todos', {
    method: 'put',
    headers: { 'Content-Type': 'application/json' },
  })
})
```
  
 Now we need to stringify our JSON to display it.
 ```
 update.addEventListener('click', _ => {
  fetch('/quotes', {
    method: 'put',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: 'Joe Biden',
      quote: 'Failure at some point in your life is inevitable, but giving up is unforgivable.'
    })
  })
})
 ```
 Remember middleware? We need to use that again so that our server can accept the JSON data. We add the bodyParser's JSON middleware. 
 ```
 app.use(bodyParser.json())
 ```
We also want the feature where we can change our to-do items to inspirational quotes if we're feeling stressed. To be able to do that, we need to operate on our MongoDB collection. The operators we use will be $set, $inc, and $push. Currently, we are changing from task to quote, so we will use the $set.  

```
app.put('/todos', (req, res) => {
      todoCollection.findOneAndUpdate(
        {
          $set: {
            task: req.body.task,
            quote: req.body.quote
          }
        },
        {
          upsert: true
        }
      )
        .then(result => res.json('Success'))
        .catch(error => console.error(error))
    })
```
We also need to use another THEN object to get a server response. Let's go to main.js and in our update function, add this:
```
 .then(res => {
      if (res.ok) return res.json()
    })
    .then(response => {
      window.location.reload(true)
    })
```
To now look like this:
```
update.addEventListener('click', _ => {
  fetch('/todos', {
    method: 'put',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: 'Joe Biden',
      quote: 'Failure at some point in your life is inevitable, but giving up is unforgivable.'
    })
  })
    .then(res => {
      if (res.ok) return res.json()
    })
    .then(response => {
      window.location.reload(true)
    })
})

```
Almost there! It's time for delete!
## Delete
Maybe we want to get rid of a quote, so we have no distractions. 
```
<section data-position="delete-quote">
      <div>
        <h2>Remove a Quote</h2>
        <p>
          
        </p>
        <button id="delete-button">Delete a quote</button>
      </div>
      <div id="message"></div>
    </section>
```
And of course, we need to call a MongoDB command in server.js for this process to actually work. 
```
app.delete('/todos', (req, res) => {
  todoCollection.deleteOne(/* ... */)
    .then(result => {
      if (result.deletedCount === 0) {
        return res.json('No quote to delete')
      }
      res.json(`Deleted a quote`)
    })
    .catch(error => console.error(error))
})
```
Remember how we had an event listener for update? We need to do the same thing for delete in main.js
```
deleteButton.addEventListener('click', _ => {
  fetch('/todos', {
    method: 'delete',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
    })
  })
    .then(res => {
      if (res.ok) return res.json()
    })
    .then(response => {
      if (response === 'No quote to delete') {
        messageDiv.textContent = 'Nothing to delete'
      } else {
        window.location.reload(true)
      }
    })
    .catch(console.error)
})
```
And now we're done! You can play around with the different MongoDB calls an experiment, or even with the CSS! 
