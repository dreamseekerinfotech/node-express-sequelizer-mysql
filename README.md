# How to intigrate mysql using sequelizer with nodejs express

Hello Everyone, in this blog we will learn how to intigrate mysql with node express using very known ORM sequelizer.
we will learn setp by step process to setup project node express project using express generator and intigrate sequelizer and make CRUD APIs.

In this blog we will make Todos application CRUD APIs, so let's start our blog

1st step: install express generator and create node express project

you can run express generator by npx comman as follow

npx express-generator

or you can install it globally by following command

npm install -g express-generator

genarete express project by following command

express [projectname] --view=[view type]
express mynodeexpress --view=pug

for more option you can visit <link>(https://expressjs.com/en/starter/generator.html)

2nd setp: install required dependency for sequelizer

now in this step install required dependencies for sequelizer and mysql

npm install sequelize mysql2 body-parser cors --save

install sequelizer cli package globally

npm install -g sequelize-cli

3rd step: initialize sequelizer and configure database

let's init sequelizer by following command

sequelize init

image sequelizerinit.png

now conffigure our database in config.json file which is newly created into config folder

"development": {
"username": "root",
"password": "password",
"database": "nodesequel",
"host": "127.0.0.1",
"dialect": "mysql"
},

4th step: create model and migrations

to create model and migration we need to fire following command od sequelizer

sequelize model:create --name [modelname] --attributes [attr1]:[datatype],[attr2]:[datatype]

for more sequelizer command please check <link>(https://sequelize.org/master/)

sequelize model:create --name Todos --attributes name:string,category:string,startDate:date,endDate:date

because of this command two file generated one in migration directory and another one in model directory

let's check migration and model file and make final change if requied.

migration file containes
migrations\20210503133202-create-todos.js

'use strict';
module.exports = {
up: async (queryInterface, Sequelize) => {
await queryInterface.createTable('Todos', {
id: {
allowNull: false,
autoIncrement: true,
primaryKey: true,
type: Sequelize.INTEGER
},
name: {
type: Sequelize.STRING
},
category: {
type: Sequelize.STRING
},
startDate: {
type: Sequelize.DATE
},
endDate: {
type: Sequelize.DATE
},
createdAt: {
allowNull: false,
type: Sequelize.DATE
},
updatedAt: {
allowNull: false,
type: Sequelize.DATE
}
});
},
down: async (queryInterface, Sequelize) => {
await queryInterface.dropTable('Todos');
}
};

model file containes
models\todos.js

'use strict';
const {
Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
class Todos extends Model {
/\*\*
_ Helper method for defining associations.
_ This method is not a part of Sequelize lifecycle.
_ The `models/index` file will call this method automatically.
_/
static associate(models) {
// define association here
}
};
Todos.init({
name: DataTypes.STRING,
category: DataTypes.STRING,
startDate: DataTypes.DATE,
endDate: DataTypes.DATE
}, {
sequelize,
modelName: 'Todos',
});
return Todos;
};

next need to migrate database by following command

sequelize db:migrate

image migratedb.png

5th setp : start create routing and APIs

app.js file will be look like as follow
app.js

var createError = require("http-errors");
var express = require("express");
var path = require("path");
var cookieParser = require("cookie-parser");
var logger = require("morgan");

var apiRouter = require("./routes/api");

var app = express();

// view engine setup
app.set("views", path.join(\_\_dirname, "views"));
app.set("view engine", "pug");

app.use(logger("dev"));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(\_\_dirname, "public")));

app.use("/api", apiRouter);

// catch 404 and forward to error handler
app.use(function (req, res, next) {
next(createError(404));
});

// error handler
app.use(function (err, req, res, next) {
// set locals, only providing error in development
res.locals.message = err.message;
res.locals.error = req.app.get("env") === "development" ? err : {};

// render the error page
res.status(err.status || 500);
res.render("error");
});

module.exports = app;

chnage in api.js file in router directory
routes\api.js

var express = require("express");
var router = express.Router();
const todoController = require("../controllers/todoController");

// Create a new todo
router.post("/todo", todoController.create);

// Retrieve all todoController
router.get("/todo", todoController.findAll);

// Retrieve a single todo with id
router.get("/todo/:id", todoController.findOne);

// Update a todo with id
router.put("/todo/:id", todoController.update);

// Delete a todo with id
router.delete("/todo/:id", todoController.delete);

module.exports = router;

make changes in todoController.js file
controllers\todoController.js

const db = require("../models");
const Todos = db.Todos;
const Op = db.Sequelize.Op;

// Create and Save a new Todos
exports.create = (req, res) => {
// Validate request
if (!req.body.title) {
res.status(400).send({
message: "Content can not be empty!",
});
return;
}

// Create a todo
const todo = {
name: req.body.name,
category: req.body.category,
startDate: req.body.startDate,
endDate: req.body.endDate,
};

// Save todo in the database
Todos.create(todo)
.then((data) => {
res.send(data);
})
.catch((err) => {
res.status(500).send({
message: err.message || "Some error occurred while creating the Todo.",
});
});
};
// Retrieve all Todos from the database.
exports.findAll = (req, res) => {
Todos.findAll()
.then((data) => {
res.send(data);
})
.catch((err) => {
res.status(500).send({
message: err.message || "Some error occurred while retrieving todos.",
});
});
};

// Find a single Todos with an id
exports.findOne = (req, res) => {
const id = req.params.id;

Todos.findByPk(id)
.then((data) => {
res.send(data);
})
.catch((err) => {
res.status(500).send({
message: "Error retrieving Todo with id=" + id,
});
});
};

// Update a Todosby the id in the request
exports.update = (req, res) => {
const id = req.params.id;

Todos.update(req.body, {
where: { id: id },
})
.then((num) => {
if (num == 1) {
res.send({
message: "Todo was updated successfully.",
});
} else {
res.send({
message: `Cannot update Todo with id=${id}. Maybe Todo was not found or req.body is empty!`,
});
}
})
.catch((err) => {
res.status(500).send({
message: "Error updating Todos with id=" + id,
});
});
};

// Delete a Todoswith the specified id in the request
exports.delete = (req, res) => {
const id = req.params.id;

Todos.destroy({
where: { id: id },
})
.then((num) => {
if (num == 1) {
res.send({
message: "Todos was deleted successfully!",
});
} else {
res.send({
message: `Cannot delete Todos with id=${id}. Maybe Todos was not found!`,
});
}
})
.catch((err) => {
res.status(500).send({
message: "Could not delete Todos with id=" + id,
});
});
};

following are the routes that we created in this blog

get 127.0.0.1:3000/api/todo
post 127.0.0.1:3000/api/todo
get 127.0.0.1:3000/api/todo/:id
put 127.0.0.1:3000/api/todo/:id
delete 127.0.0.1:3000/api/todo/:id

that is it for this blog. hope you are clear about on sequelizer and mysql connect to you nodejs project.

If you have any query then hit me up on my social media account or you can mail me on learning@dreamseekerinfotech.com
