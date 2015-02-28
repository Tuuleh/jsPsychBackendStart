# Tutorial - How to deploy your jsPsych experiment on Heroku as a Node.js application 
## ...or in other words, full stack JavaScript for psychologists

Clone this repository for a starting point for my tutorial on turning your jsPsych experiment into a 
Node.js application (with a bit of help from Express and Mongoose) and deploying it to Heroku. You can access the live demo <a href = "https://floating-oasis-6903.herokuapp.com" target = "new"> here</a>, or look at the code for the finished experiment in my <a href="https://github.com/Tuuleh/jsPsychBackendDemo">other repository</a>. This tutorial might feel a little overwhelming, so go through it little by little. It'll eventually be split into sections (4-5 blog posts) on my <a href="http://web-psychometrics.com/">website</a>.

My intention for this tutorial is to write a solution that is sufficiently simple so that with some self-study, anyone could deploy their jsPsych experiment with no previous back end experience. Both the solution and the language used in the tutorial are simplified, because my target audience consists of people who don't necessarily have any experience with the web. That being said, if you have ideas for improvements or find a problem, please contact me and tell me more!

I'm working through the tutorial on Ubuntu 14.10. The process should not be very different on a Mac OS, but if you're on Windows, you will have to find out how to install and manage the tools on your own. I'd recommend installing a <a href="https://www.virtualbox.org/">virtual machine</a>, which will allow you to run a Linux system inside your current operating system.

You'll need the following ingredients for your secret sauce (we'll install them together throughout the tutorial):

<ol>
    <li>
        <a href = "http://www.jspsych.org">jsPsych</a> (d'oh!)
        <p>An open source JavaScript library for web based experiments in behavioral sciences.</p>
    </li>

    <li>
        The <a href = "http://nodejs.org/">Node.js</a> runtime environment and its awesome package manager, <a href="https://www.npmjs.com/">NPM</a>
        <p>Node.js® is built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js represents the flesh and bones on top of which we build the back end for our experiment. <a href = "http://expressjs.com/">Express</a> is a web framework for Node.js, and we'll be using <a href="http://mongoosejs.com/">Mongoose </a>- a MongoDB driver for Node.js.</p>
    </li>

    <li>
      <a href = "http://www.mongodb.org/" >MongoDB</a>
      <p>MongoDB is a cross-platform document-oriented NoSQL database. It will work as the storage for the data we get out of jsPsych. If you're interested, you could learn more about <a href = "https://www.youtube.com/watch?v=qI_g07C_Q5I">NoSQL</a> databases, <a href = "http://en.wikipedia.org/wiki/Relational_database">relational databases</a> or <a href = "http://json.org/">JSON</a>, since that's the format we're getting out of jsPsych. Or you could also not learn about any of those and still be able to complete this tutorial, but you will be none the smarter.</p>
      <p>For more advanced readers - I personally recommend using an <a href="http://en.wikipedia.org/wiki/Object-relational_mapping">ORM</a> between jsPsych and a relational database, as your experimental data can be aggregated in a way that is well suited for a relational DB. I like to make a table for each experiment, passing an identifier I use as a foreign key for the participant every time I make a request for a new page. If you use the relational approach and/or store data between tasks, you'll need to take care of some validation (making sure the participant doesn't repeat the same task several times, that there aren't several entries for the same ID etc), but the partial data gained from storing between the tasks affords you some valuable insight into performance and characteristics of participants who dropped out. </p>
    </li>

    <li>
      <a href = "https://dashboard.heroku.com/">Heroku</a> 
      <p>Heroku is a <a href = "http://en.wikipedia.org/wiki/Platform_as_a_service">platform as a service (PaaS)</a> that enables developers to build and run applications in the cloud. Heroku has a free tier that suits our needs and offers support for Node.js, so we are going to deploy our application to Heroku - this way, the entire experiment will be contained on the cloud and we do not need to worry about a web server - huzzah!</p>
    </li>

    <li>
      <a href="http://git-scm.com/">Git</a>
      <p>Git is a <a href = "http://en.wikipedia.org/wiki/Revision_control">version control</a> tool. In my opinion it's easiest to get started with Git by going to <a href = "https://github.com/">GitHub</a> and doing their Git bootcamp. We will need it to deploy the experiment to Heroku. </p>
    </li>
</ol>

##Track

1. Aligning stimuli in the center of the screen, and viewing the experiment in your browser
2. Setting up Node.js and serving your experiment locally
3. Database configuration - getting started with MongoDB and Mongoose
4. Setting up remote accounts, deploying to Heroku


##1. Horizontal and vertical aligntment, and viewing the file in your browser

You should get started by <a href="http://www.jspsych.org">downloading jsPsych</a> and doing the <a href="http://docs.jspsych.org/tutorials/hello-world/">Hello World</a> and <a href="http://docs.jspsych.org/tutorials/go-nogo-task/">go/no-go tutorials</a>. It'll be simplest to follow this tutorial if you clone this repository. If you get stuck at any point, you can peek at the finished solution <a href= "https://github.com/Tuuleh/jsPsychBackendDemo">here</a>. 

I'll take off from a point where you have created your experiment. You can view the static html file locally by pointing your browser to the location of the file. For instance, to view my go/no-go experiment, I point my browser to 'file:///home/tuuli/jsPsychBackendStart/public/views/go_no_go.html'. If you navigated to the file, you could see that you can go through the experiment locally and in the end, the results get written onto the screen as expected. 

You can see that I have jsPsych in its own folder, and all my static files in a folder called public, nested into their respective folders. The folder called 'views' is reserved for my html files, such as the experiments, an informed consent, a finish page... This is some pretty minimal structure and works to demonstrate that you should have a clear, modular structure for your project. This makes the project easy to read and allows you to change one part of your project without breaking its other parts. But note that if you do change the structure of your project, you will have to change paths accordingly! For example, the <img> tags in my instructions may have a different source path than yours, because my images are in the /public/img/ folder. 

###Centering stimuli

Before starting with the back end, we're going to change the styling in the experiment a little. In most experimental environments, the stimulus is placed aligned onto the center of the screen, both horizontally and vertically. This can be accomplished with nested <div>-elements. The HTML <div> element is a block level element that can be used as a container for other HTML elements. It's used to group block-elements to format them with CSS. You can see that I've specified a new CSS file, experiment.css in the /public/css folder. Add a link to that CSS file in the go_no_go.html-file. Experiment-specific CSS files should be linked under your higher level stylesheets as they can be used to override the elements. Your stylesheet imports should look like this now:

```html
  <link href="../../jsPsych/css/jspsych.css" rel="stylesheet" type="text/css"/>
  <link href="../css/experiment.css" rel="stylesheet" type="text/css"/>
```

You can refresh the page and see that things are now centered horizontally. Vertical centering will be achieved by styling centered ```<div>```-elements. You can see that in experiment.css, we specified styles for classes called parent and child. We'll need to add these divs into the body of our html file:

```html
<body>
    <div class = "parent">
      <div class = "child">
        <div id="jspsych_target"></div>
      </div>
    </div>
</body>
```
However, these changes alone are not enough to change the style. We need to tell jsPsych to append its targets (text and stimuli) to the ```<div>``` with the id "jspsych_target". We do this by adding a parameter to the object we pass to the ```jsPsych.init()```-call - we need to tell it that our display element is the one with the id we specified: 

```javascript
jsPsych.init({
  experiment_structure: experiment,
  display_element: $('#jspsych_target'),
  on_finish: function() {
    jsPsych.data.displayData();
  }
});
```

If you're confused by ```$('#jspsych_target')```, don't worry. The $ here is a synonym for jQuery, and # specifies that we're looking for a jQuery element with that id. (this allows to quickly identify the right element in the <a href = "http://www.w3.org/TR/DOM-Level-2-Core/introduction.html">DOM</a>). You can read more about jQuery <a href = "http://jquery.com/">here</a>. 

##2. Setting up Node.js and serving your experiment locally

To serve your experiment locally, the first thing you need to do is to <a href="http://nodejs.org/"> install Node.js</a> and its package manager, <a href = "https://www.npmjs.com/">NPM</a>, which you don't need to get separately since it's a part of the Node installation. 

After you have your Node all set up, go to the root folder of your project in the terminal, and type <code>npm init</code>. This will bring up a prompt for creating a file called ```package.json```, which is where you specify information and dependencies for your Node.js application. You can leave nearly all of the fields blank if you wish. When it prompts you for main, suggesting index.js as the default, you can instead type app.js, since that is what I will be using for the tutorial. We'll leave the dependencies empty for now, since we will use npm to install dependencies as we go - I find that this approach is more intuitive because it introduces each new module at its turn. Once you're finished with the prompts from npm init, your package.json file is ready, and you can fill it in or modify it in a text editor if you wish.

Npm makes installing dependencies very easy. You simply issue <code>npm install</code> and specify the package you want. Typing <code>npm install --save</code> will additionally add it to package.json, and <code>npm install -g</code> will install the dependency globally. Try npm out put by issuing <code>npm install nodemon</code>. <a href = "http://nodemon.io/">Nodemon</a> is a tool that restarts your server every time you save a change to your source code. You can develop without Nodemon, but since we are writing server side code, we have to restart our server (even if it is local) every time we make any changes to the server side code, which gets really boring really soon. Using --save when installing dependencies is important because we need to have our package.json up-to-date at the finial stage of the tutorial, when we deploy the application to Heroku. The reason we don't add Nodemon to our package.json is that we won't need it <i>in production</i>, after we've deployed - we only need it while we're developing our application. 

You can see that a new directory is also created into your project structure, called node_modules. This is a directory that contains all dependencies installed with npm. We don't actually want to push these dependencies onto our remote repository (because everything we need in production will be described in package.json), so we'll add a new file to the root of the folder, named .gitignore - this file should contain all of the contents you wish to hide from your remote repository. For now, the file should contain nothing but the text <code>node_modules</code>, specifying that want to hide your npm-installed Node modules. 

###Hello, Express

Add a file called ```app.js``` to the root of your project. This will be the main file for our back end - where we configure our server, our database and the routing (passing of data and requests) for our application. 
First, we'll install <a href="http://expressjs.com/">Express</a> - a light-weight <a href="http://en.wikipedia.org/wiki/Web_application_framework">web framework</a> for the Node runtime, by typing <code>npm install --save express</code>. Have a look at your package.json file - Express is now written under dependencies and you can see it in the node_modules directory.
In order to use Express, we need to import it into our Node application. In Node, modules are loaded with <code>require</code>, like so:
```javascript
var express = require('express');
```
Write this line into your app.js file. You've now required the previously installed Express framework into your app, and you can access its functionality. Next, we'll create the application and a server - in just four lines of code :) 

```javascript
var app = express();
var server = app.listen(3000, function(){
    console.log("Listening on port %d", server.address().port);
});
```

The ```express()``` function is a top-level function exported by the express module, which creates an Express application. It has a method to <i>listen</i> on a specified port - in our case, port 3000 - which accepts a <i>callback function</i> as its second argument. A callback function is a function, which gets executed once its parent method or function completes - in this case, starting listening on port 3000. In our callback function, we tell it to log to the console which port we are listening on.

Now, start your service by typing <code>nodemon app.js</code> on the command line - you can also start your app with Node, typing <code>node app.js</code>, but as I wrote earlier, if you use Nodemon, you won't have to manually restart service every time you make a change. Point your browser to http://localhost:3000/ and see what happens.

Right, I'll spoil it for you. It says "Cannot GET /". What happened is that because you tried to access the application, it received a HTTP GET request with the URL "/", and you haven't speficied what to do when that request is received. GET is the most common <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html">HTTP</a> method; it says "give me this resource". Telling the application what to do when it gets a request is called routing, and we'll do that right now for when we receive a GET request at "/":

```javascript
app.get('/', function(request, response) {
    response.send('Hello, Express!');
});
```

Now whenever we get a GET request to "/", we send back a string as a response, reading "Hello, Express!" - this will get printed out onto the window if the service is restarted and the browser is again pointed at http://localhost:3000. But what happens if you point at http://localhost:3000/experiment? The server receives a HTTP GET request with the URL "/experiment" and it cannot handle it - because we haven't told it what to do. 

###Serving your experiment

Here's what we'll do. When we get the default "/" request, we want to serve a page with an informed consent. The informed consent will have a button, which - when we click it and accept the terms of participation - will <i>redirect</i> the user to the go/no go experiment. In order to render a html page as a response to a GET request, we need to modify the callback function we have specified in app.get('/'), like so:

```javascript
app.get('/', function(request, response) {
    response.render('index.html');
});
```

But there are a few things that need to be taken care of first. We need a way to serve static html pages. One of the beautiful ideas behind Node.js is that it's very modular - you import things you need, and it doesn't burden you with anything extra. This same principle carries over to Express. You might be surprised that introducing the funcionality to serve static html pages requires a few steps.

The first thing we need to do is to tell Express where it can find our static files - css, images and such. We'd also have our custom scripts at that location, if we had any. Here we tell it to use static middleware, and we tell it that it can find static files inside the /public folder.

```javascript
app.use(express.static(__dirname + '/public'));
```

This isn't enough, though, because we need <i>different</i> static files when we refer to the static jsPsych files. Those are of course not in our /public folder, but in our /jsPsych folder - so we specify another folder for static files that need to be fetched when the we look for something starting with '/jsPsych':

```javascript
app.use('/jsPsych', express.static(__dirname + "/jsPsych"));
```

We can't just tell it to render 'index.html' without telling it where it is, so we tell Express that our <i>views</i> can be found in the /public/views folder.

```javascript
app.set('views', __dirname + '/public/views');
```

Finally - Express utilizes template engines. A <a href="http://www.simple-is-better.org/template/">template engine<a/> is a tool that makes writing html files cleaner and additionally allows you to mix logic inside your html. Template engines can bring some very interesting features into your project, but we are not going to dive into their use in this part of the tutorial. For our current purpose, raw html would do just fine. However, Express is expecting us to specify a template engine, and so we will tell it that we are going to use an engine called EJS, but that we are going to be serving files with .html file extension. This works all right, because EJS works fine with raw HTML. 

EJS is a node module, and so it has to be installed with npm:

<code>
npm install --save EJS
</code>

For the upcoming second part of the tutorial, we will be passing a unique identifier for each participant inside a script tag - this allows us to recognize their data across different tasks in a test battery in the database - and for that purpose, we will eventually use some EJS functionality. But for now, just think of all this as raw HTML, and add this to your app.js:

```javascript
app.engine('html', require('ejs').renderFile);
app.set('view engine', 'html');
```

Now, let's make the informed consent. In your /public/views/ directory, make a file called index.html and create your informed consent. You can also add some styling - put your stylesheet in /public/css/ and link to it in your .html file. If you're completely out of inspiration, you can ~~copy~~ plagiarize my index.html and its associated stylesheet in the GitHub repository for the <a href="https://github.com/Tuuleh/jsPsychBackendDemo">tutorial demo</a>. Below the consent text inside the <body> tag, add this button element:


```html
<button type='button' onclick = "window.location = 'experiment'">I agree to donate my body to science</button>
```

The ```onclick``` specifies an event that happens when the button is clicked. We're specifying here that when the button gets clicked, we want to redirect to 'experiment' - this issues a HTTP GET request to the server, with the URL "/experiment". In order to handle this, we need to go to app.js and set up <i>routing</i> for this request. The routing for '/experiment' is not very different from the routing for '/' - we just tell it to render our experiment instead of the index page.

Your app.js file should now look something like this - but note that I separated creating the app with express() and starting the server, since configuring middleware in between makes for a more logical structure:

```javascript
// --- LOADING MODULES
var express = require('express');

// --- INSTANTIATE THE APP
var app = express();

// --- STATIC MIDDLEWARE 
app.use(express.static(__dirname + '/public'));
app.use('/jsPsych', express.static(__dirname + "/jsPsych"));

// --- VIEW LOCATION, SET UP SERVING STATIC HTML
app.set('views', __dirname + '/public/views');
app.engine('html', require('ejs').renderFile);
app.set('view engine', 'html');

// --- ROUTING
app.get('/', function(request, response) {
    response.render('index.html');
});

app.get('/experiment', function(request, response) {
    response.render('go_no_go.html');
});

// --- START THE SERVER 
var server = app.listen(process.env.PORT, function(){
    console.log("Listening on port %d", server.address().port);
});
```

Congrats! You now have a functional Node.js application.


##3. Database configuration - getting started with MongoDB and Mongoose

Your application is now running locally, but you're still only printing out the data on the screen at the end of the experiment instead of using a persistent storage solution. We're going to alter the experiment so that it pipes the data into a <a href = "http://www.mongodb.org/">MongoDB<a/> database. MongoDB is a document database, which serves well as data storage solution to our problem, because our use is extremely simple: we run the experiment, post the entire jsPsych output JSON object into the database, and just access it later for analysis. If your experimental battery is more complex (e.g. requires validation or has several tasks), I'd recommend using a relational database with an ORM instead. I've used <a href="http://sequelize.readthedocs.org/en/latest/">Sequelize</a> between a Node-based jsPsych experiment and a MySQL database before, and while it does take more setting up, it pays off for more complicated use cases. You can also use it for Postgres.

###MongoDB

The first thing to do is to <a href="http://docs.mongodb.org/manual/installation/">set up MongoDB locally</a>. Once you have it set up, go to the console and enter the Mongo shell by typing <code>mongo</code>. You can view databases that are available to you by issuing <code>show databases</code>. You can access an existing database or create a new one with the <code>use</code> expression. We'll create a local database called jspsych:
<code>use jspsych</code>.

Once you're in the database, you can view its contents by typing <code>show collections</code>. Since the database is empty, Mongo will not respond with anything in particular. Let's add an a collection called 'fruit_shop' with a few documents, and re-issue the command to show collections:
```
db.fruit_shop.insert([{item:'banana', quantity:15, price: '0.25'}, {item:'orange', quantity:8, price :'0.35'}, {item:'apple', quantity: 25, price:'0.12'}])
```
You can now issue <code>show collections</code> again and you'll see we've created a collection called fruit_shop. 
If you issue <code>db.fruit_shop.find()</code>, you'll receive a list of the objects in the fruit_shop collection - the ones we inserted there a moment ago. 

```
{ "_id" : ObjectId("54ef903d8e352a6695c499b0"), "item" : "banana", "quantity" : 15, "price" : "0.25" }
{ "_id" : ObjectId("54ef903d8e352a6695c499b1"), "item" : "orange", "quantity" : 8, "price" : "0.35" }
{ "_id" : ObjectId("54ef903d8e352a6695c499b2"), "item" : "apple", "quantity" : 25, "price" : "0.12" }
```
You'll also notice that each object has a new parameter, called "_id". If the document insert does not specify an _id field, then MongoDB will add the _id field and assign a unique object id for the document before inserting. If the document contains an _id field, the _id value must be unique within the collection to avoid duplicate key error. 

The reason I'm illustrating how the data is laid out in the DB is so you'll have an idea of how we'll store the jsPsych output. For a single participant, the document would look like this (except with a number of trials that corresponds to the length of the experiment):
```javascript
{
    "_id" : ObjectID("54ef903d8e352a6695c499b3"), 
    "data": [
        {
            "rt": 1275,
            "key_press": 13,
            "trial_type": "text",
            "trial_index": 0,
            "trial_index_global": 0,
            "time_elapsed": 1291,
            "internal_chunk_id": "0-0"
        },
        {
            "rt": 708,
            "key_press": 70,
            "trial_type": "text",
            "trial_index": 0,
            "trial_index_global": 1,
            "time_elapsed": 2552,
            "internal_chunk_id": "0-0"
        },
        {
            "rt": 387,
            "stimulus": "../img/orange.png",
            "key_press": 70,
            "response": "no-go",
            "trial_type": "single-stim",
            "trial_index": 0,
            "trial_index_global": 2,
            "time_elapsed": 4627,
            "internal_chunk_id": "0-0"
        },
        ...   
        ...
        ...
    ]
}
```
MongoDB simply works as a data storage, where every participant has their own document, containing their data and object id. Now that this has been established, let's look at how to remove elements from the collection:

<code>
db.fruit_shop.remove({"item":"banana"})
</code>

Our fruit shop is now out of bananas. You can see this by issuing <code>db.fruit_shop.find()</code>. But how about we stop with this fruit vendor business altogether and first look at how to empty our inventory

<code>
db.fruit_shop.remove({})
</code>

...and then how to delete the entire fruit_shop collection so we can instead populate the database with something more useful:

<code>
db.fruit_shop.drop()
</code>

Be very careful when dropping <i>anything</i>! You will not be able to recover the lost data. Let's exit the shell - we don't need to do anything more than this with our MongoDB instance for now. Let's instead go to our go_no_go.html file and figure out what we need to change in order to send data to our back end (our Node application in app.js) instead of displaying it on the screen at the end of the experiment. 

###An AJAX call in the on_finish callback

The section you should be looking at is this one, where you start the experiment:
```javascript
jsPsych.init({
  experiment_structure: experiment,
  display_element: $('#jspsych_target'),
  on_finish: function() {
    jsPsych.data.displayData();
  }
});
```
And actually, you should just be looking at the callback function defined in ```on_finish```. Remember that callback functions are functions that get executed once their 'parent functions' are done - this one gets called once your experiment has completed.
So, in this callback, instead of telling jsPsych to display data on the screen, we should tell it to send it to app.js. We'll use a technique with one of the dumbest abbreviations in the history of the WWW - AJAX, or Asynchronous Javascript And XML. It's dumb because its second 'A' stands for 'and', and because it's most frequently used with JSON now that no-one wants to be seen wearing XML in public.

I've written the new callback ready - let's go through it row by row:

```javascript
jsPsych.init({
  display_element: $('#jspsych_target'),
  experiment_structure: experiment,
  on_finish: function() {
    $.ajax({
      type: "POST",
      url: "/experiment-data",
      data: JSON.stringify(jsPsych.data.getData()),
      contentType: "application/json"
    })
    .done(function() {
      window.location.href = "finish";
    })
    .fail(function() {
      alert("A problem occurred while writing to the database. Please contact the researcher for more information.")
      window.location.href = "/";
    })
  }
});
```

The ```$.ajax()```-section specifies an AJAX call. In our context, $ stands for jQuery, so we're doing a <a href="http://api.jquery.com/jquery.ajax/">jQuery AJAX</a> call here - this performs an HTTP request. We specify that it should submit a POST request to the URL "/experiment-data". The POST request method is designed to request that a web server accepts the data enclosed in the request message's body for storage. To read a bit more about HTTP requests, shuffle to the request methods section <a href="http://code.tutsplus.com/tutorials/http-headers-for-dummies--net-8039">here</a>.

In the data parameter of the object we pass to the AJAX call, we specify what data to send in the request body. We want to send the data we received from the jsPsych experiment, and we get the jsPsych data by calling the jsPsych.data.getData() method from the <a href="http://docs.jspsych.org/core_library/jspsych-data/">jsPsych.data module</a>. We first turn it from a JavaScript value to a JSON string by calling JSON.stringify() on it.  

The callback function defined inside the ```.done()``` method gets executed if the AJAX call completes succesfully. In our case, we redirect the page to a finish page where we'll thank the participants for their time and give again them the researcher's contact information. If you had several tasks, you'd use this callback to send a GET request to route to your next task. The callback inside the ```.fail()``` method gets called if the post in the AJAX call fails. For this example, we simply alert the user that there was a problem and redirect them to the front page, but usually you'd do some more complicated error handling in case the post request would fail. 

We're issuing a GET request to "/finish" on the call that gets triggered with a successful post, so make a finish.html into your views and set up routing for that. It's a simple GET request so you should be able to figure out how to do it from your previous routes - you can cheat by looking at the code in the solution, but we'll instead continue with what to do with the POST request with our jsPsych data. In the AJAX call, we specifified that we want to send the jsPsych data in the body of the post request to the URL "/experiment-data". Since we want to make a request to a new URL, we will need to configure routing for this request in app.js. 

For a post request, we call ```app.post()``` instead of ```app.get()```, and in the callback function, we need to write a function to parse the request body for the data and to send it into our database. We need two new dependencies: body-parser and mongoose. <a href="https://www.npmjs.com/package/body-parser">Body-parser</a> is middleware for Node.js that allows you to parse key-value data contained in the request body, and <a href="http://mongoosejs.com/">mongoose</a> is a MongoDB driver for the Express framework. 

To install the new dependencies, go to your console and do your usual magic: 

<code>
npm install --save body-parser mongoose
</code>

We need to load the modules in app.js, so update the first lines where you load your modules:

```javascript
var express = require('express'),
    mongoose = require('mongoose'),
    body_parser = require('body-parser');
```

We also need to specify the use of our body parsing middleware, below the static middleware: 

```javascript
app.use(body_parser.json());
```

Finally, we'll start working with the routing for the post request. First, let's just see what's contained in the request body by logging it to the terminal (remember that when you call ```console.log``` on server side code, it logs to your terminal instead of the browser console). This should log the jsPsych output JSON onto your terminal:

```javascript
app.post('/experiment-data', function(request, response) {
    console.log(request.body);
})
```

###Mongoose

Now we just need to figure out how to pipe it into our MongoDB. Generally, you need to design your database so that it supports your data storage. Usually, this means specifying a <a href="http://stackoverflow.com/questions/2674222/what-is-purpose-of-database-schema">schema</a>. A schema is a description of the organization of data as a blueprint of how a database is constructed. Schemas are important and in fact most database solutions necessarily require for you to specify one. They bring structure and constraints to your database, allow you to set permits etc. However, for the sake of simplicity, we are going to use a schemaless MongoDB instance. This means that we do not specify how the data we enter into the database should look like, what parameters it should have etc, but we will simply use MongoDB like a bucket where we pour our jsPsych output. Mongoose doesn't directly allow you to use a schemaless database, so we'll make a work-around by using an empty schema with the ```strict``` parameter set to ```false```. In your app.js, write your Mongoose configuration above the lines where you set up middleware.

```javascript
var emptySchema = new mongoose.Schema({}, { strict: false });
var Entry = mongoose.model('Entry', emptySchema);
```

We'll then open a connection to our local MongoDB, called jspsych, at mongodb://localhost/jspsych, log 'connection error' if the connection fails, and 'database opened' on its first opening:

```javascript
mongoose.connect('mongodb://localhost/jspsych');
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error'));
db.once('open', function callback() {
    console.log('database opened');
});
```

Now we'll need to actually pipe in the data in the routing for a POST request to /experiment-data by using the Entry model and the empty schema we specified in Mongoose configuration. Remember that request.body contains nothing but the jsPsych output we sent from our front end - go_no_go.html - in the AJAX call. We simply send a JSON object to MongoDB, with a key - 'data' - that contains our experiment output for this given participant.

```javascript
app.post('/experiment-data', function(request, response){
    Entry.create({
        "data":request.body
    });    
    response.end();
})
```

Let's try this out. Connect to mongo and make sure your jspsych database is empty. Once you've done that, restart your application, and complete the experiment once. Once you're done, go to the shell and reconnect to your mongodb:

```bash
> mongo
MongoDB shell version: 2.6.7
connecting to: test
> use jspsych
switched to db jspsych
> show collections
entries
system.indexes
> db.entries.find()
```

There was a new collection called 'entries'. Searching through it will show you that it contains one entry with two or three parameters: "_id", which is the object ID MongoDB adds, "data", containing the jsPsych data we parsed from the request body, and possibly <a href="http://stackoverflow.com/questions/12495891/what-is-the-v-field-in-mongodb">"__v"</a>, which is a version key parameter added by Mongo, containing the internal revision of the document. You will get a document like this, entered to the 'entries'-collection, for every participant who completes the experiment.

##4. Setting up remote accounts and deploying to Heroku

Awesome, you now have a fully fledged application you can run locally, database configuration and all! Now you only have to take care of a few formalities in order to deploy your application and start sharing the link to recruit participants. 

First, the local MongoDB instance has worked great for demonstration purposes, but we're really going to need a remote database. We'll set up a free account on <a href = "https://mongolab.com/">MongoLab</a> - a cloud-based database-as-a-service (<a href="http://www.scaledb.com/dbaas-database-as-a-service.php">DBaaS</a>) for MongoDB. Just navigate to their website and <a href="https://mongolab.com/signup/">make an account</a>. Once you've done that, log in, and under your MongoDB deployments, select "create new". Select your favorite cloud provider (perhaps one with a server close to your location), and change your plan to single-node. Pick a free plan and give your database a name - I called mine jspsych_db - and click the 'Create new deployment'-button. Then, select the db you just created from your MongoDb deployments tab. You have to make a user in order to access the database, so click the 'Users'-tab, and add a new user. You need write permits, so don't tick the read-only checkbox.

Now, let's set up Heroku. You should go to their <a href="https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction">Node.js</a> starter page to get started. If you choose to do their starter tutorial, just make sure you're not in your project directory when you clone their repository. 

To get started with deploying your own project, go to the root of your project folder and type <code>heroku create</code> to get started. But in order to get our application on Heroku, we'll need to make a few changes. First of all, we need a Procfile. Make a file to the root of the project, called Procfile (with no extension), and the following content: 

<code>web: node app.js</code>.

Next, we specified that we want our server to listen on port 3000. Heroku will not necessarily have the application running on port 3000, so we'll need to find out the port from the process environment and use that value instead. In Node, you can access Heroku configuration variables with ```process.env```. So in order to find out the port, we use ```process.env.PORT```: 

```javascript
var server = app.listen(process.env.PORT, function(){
    console.log("Listening on port %d", server.address().port);
});
```

Next, go to <a href = "https://mongolab.com">MongoLab</a>, to the database you set up for the experiment, and look for the URI string: 

<img src="http://i.imgur.com/Kxi68ya.png"/> 

We want this connection string to be accessible to Heroku, but we want to hide it from our public Git repository. We'll use Heroku's config variables for this purpose. Go to your terminal and type...

<code>heroku config:set CONNECTION = [MongoLabs_URI]</code>

... where ```[MongoLabs_URI]``` is your URI string - replacing <dbuser> and <dbpassword> with their respective values. Type <code>heroku config</code> again to verify you got it right (there should now be a CONNECTION variable), and then modify your mongoose connection in app.js:

<code>
mongoose.connect(process.env.CONNECTION);
</code>

Now, push your repository both to GitHub and to Heroku:

```bash
> git push
...
> git push heroku master
...
```

... and make sure that the app is running on at least one dyno:
```bash
> heroku ps:scale web=1
```

...and then view it live in a browser!
```bash
> heroku open
```

If you get an application error after deploying, go to your console and issue <code>> heroku logs</code>. Debugging heroku can be a little annoying, but the logs should help you define what the problem is. It's often an iterative process and requires a bit of work. A frequent error with the default go/no-go experiment is that once you progress to the experiment, the page is empty, and you can see on the browser console that the page on heroku was loaded over HTTPS, but it's doing an insecure HTTP request for jQuery - so make sure to correct the jQuery loading script on your experiment .html file into HTTPS instead of HTTP. 

#The end
This is the end of the tutorial. Comments and feedback are welcome at tuuli.pollanen@gmail.com!
