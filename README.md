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
If you save and reload, you should see Hello World display (in ugly font) on your page. 

Since we're going to be actually displaying something cool and are making a web app, let's go to our index.html. This will help us form and serve up how our page will look. 

```
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html')
})
```

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

