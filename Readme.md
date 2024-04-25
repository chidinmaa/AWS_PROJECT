## MEAN STACK DEPLOYMENT ON UBUNTU
- ### MONGODB
- ### EXPRESS
- ### ANGULAR
- ### NODE.JS
#### STEP 1
- Create an ubuntu 22.04 server on Ec2 micro family,remember to use a keypair.
### INSTALL NODE.JS
```
sudo apt update
sudo apt upgrade
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
nvm install 20
```
- Always make sure to check the version of your node.js
```
node -v
npm -v
```
- The version should be the latest.
### INSTALL MONGODB
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
```
- To use the wget you have to change to roots.
```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```
- Update and Upgrade the machine
```
apt-get update
apt-get upgrade -y 
```
- Now lets install the mongodb app
```
sudo apt install mongodb-org
```
- Verify to see mongodb has been installed
- Then enable MongoDB to start at system startup and initiate its start as well.
```
sudo systemctl enable mongod
sudo service mongod start
```
- Now check the mongodb status.
```
sudo service mongod status
```
![alt text](<mongodb active.PNG>)
### INSTALL MONGODB AND START
```
sudo apt install -y mongodb
sudo service mongodb start
```
- Make sure the service is up and running.
```
sudo systemctl status mongodb
```
![alt text](mongoose.PNG)
### Install npm – Node package manager.
```
sudo apt install -y npm
```
### Install body-parser package
We need ‘body-parser’ package to help us process JSON files passed in requests to the server.
```
sudo npm install body-parser
```
### Create a folder named Books
```
mkdir Books 
cd Books
```
###In the Books directory, Initialize npm project.
```
npm init
```
![alt text](<npm init.PNG>)
-  
Add a file to it named server.js
```
vi server.js
```
- Copy and paste the web server code below into the server.js file.
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
### INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER
Step 3: Install Express and set up routes to the server
- We will use Mongoose to establish a schema for the database to store data of our book register.
```
sudo npm install express mongoose
```
- In ‘Books’ folder, create a folder named apps.
```
mkdir apps
cd apps
```
Create a file named routes.js
```
vi routes.js
```
- Copy and paste the code below into routes.js
```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```
- In the ‘apps’ folder, create a folder named models.
```
mkdir models 
cd models
```
- Create a file named book.js.
```
vi book.js
```
Copy and paste the code below into ‘book.js’
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```
###Step 4 – Access the routes with AngularJS
-  Change the directory back to ‘Books’
```
cd ../..
```
Create a folder named public
```
mkdir public 
cd public
```
Add a file named script.js
```
vi script.js
```
 
- Copy and paste the Code below (controller configuration defined) into the script.js file.
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
- In the public folder, create a file named index.html;
```
vi index.html
```
 
- Copy and paste the code below into index.html file.
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>
 
        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>
 
          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
Change the directory back up to Books.
```
cd ..
```
Start the server by running this command:
```
node server.js
```
- The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what the curl command returns locally.
```
curl -s http://localhost:3300
```
### EDIT INBOUND RULES
- Open port 3300.
![alt text](project4.PNG)


 








 