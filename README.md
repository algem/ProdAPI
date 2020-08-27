# ProdAPI
Sample API Code

In this article, we are going to learn how to create a simple API in Node.js and using MS SQL as a data source on which we are going to perform CRUD operation.
Why Node.js if we already have other frameworks?
•	It is super-fast and highly scalable e.g. PayPal, Uber, Netflix
•	Builds twice as fast with fewer people
•	33% fewer lines of code
•	40 % fewer files
•	2x request/sec
•	35% faster response
Large ecosystem of the open source library
JavaScript everywhere (front + back)
You can write an entire application using JavaScript, no need to learn a new language.
 Work Flow
 
 

SQL Server Configuration (Microsoft SQL Server 2019)

 
 
 


Create Table

USE [ProductDB]
GO

/****** 
Object:  Table [dbo].[Products]    
Script Date: 27/08/2020 10:10:54 PM 
By: Algem Mojedo
******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[Products](
	[ProductId] [int] IDENTITY(1,1) NOT NULL,
	[ProductName] [nvarchar](450) NULL,
	[ProductPrice] [numeric](18, 2) NULL
) ON [PRIMARY]
GO

Create Store Procedure Insert
CREATE PROC [dbo].[Usp_InsertProduct]
@ProductName varchar(450),
@ProductPrice decimal(18,2)
AS
BEGIN
INSERT INTO [Products]
	( [ProductName]
 	,[ProductPrice])
VALUES
	(@ProductName
	,@ProductPrice)
END
Create Store Procedure Update
CREATE PROC [dbo].[Usp_UpdateProduct]
@ProductID int,
@ProductPrice decimal(18,2)
AS
BEGIN
	UPDATE [dbo].[Products]
	SET [ProductPrice] = @ProductPrice
	WHERE ProductID = @ProductID
END
Create Store Procedure Delete
CREATE PROC [dbo].[Usp_DeleteProduct]
@ProductID int
AS
BEGIN
	DELETE FROM [dbo].[Products]
     WHERE ProductID = @ProductID
END


Installation Requirements
1.  Node.js version 6.14.4
2.  npm version 6.14.4
3.  Visual Studio Code Editor
============================

1. Lunch command with Admin righs
2. Make directory c:Users\al\source\ProdAPI   		This is mine.  This maybe deference in yours
 3. Change directory c:Users\al\sourceProdAPI
4. Check the version of Node.js 
	node -v 				 -> Must be 12.18.3 or higher
5. Check the version of npm  
	npm -v  				-> Must be 6.14.6 or higher
6. Create a package.json
   	 Npm init
package name: (prodapi)
version: (1.0.0)
description: Product API
entry point: (index.js)
test command:
git repository:
keywords:
author: Algem Mojedo
license: (ISC)
About to write to C:\Users\al\source\ProdAPI\package.json:
{
  "name": "prodapi",
  "version": "1.0.0",
  "description": "Product API",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Algem Mojedo",
  "license": "ISC"
}
Is this OK? (yes) Yes

Installing modules
1.	Express
Fast, un opinionated, minimalist web framework for node.
	npm install express --save

More Details: https://www.npmjs.com/package/express
2.	body-parser
Node.js body parsing middleware.
Parse incoming request bodies in a middleware before your handlers, available under
the req.body property.
  	npm install body-parser --save
3.	Mssql
Microsoft SQL Server client for Node.js Joi
npm install mssql joi  --save
Opening the Project in Visual Studio Code from the Command Prompt
Code . 
then  Enter
Above command creates a package.json
{
  "name": "prodapi",
  "version": "1.0.0",
  "description": "Product API",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Algem Mojedo",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.19.0",
    "express": "^4.17.1",
    "joi": "^17.2.1",
    "mssql": "^6.2.1"
  }
}


Create two sub directory in PRODAPI
•	connection
•	controller
Create a file in sub directory connection
Connect.js
var sql = require("mssql");
var connect = function()
{ 
    var conn = new sql.ConnectionPool({
        user: 'sa',
        password: 'w3bus3r@1957',
        server: 'ALGEM',
        database: 'ProductDB',
        port: 1433,
        connectionTimeout: 3000,
        requestTimeout: 3000,
        pool: {
            max: 100,
            min: 0, // 1 = don't close all the connections.
            idleTimeoutMillis: 1000,
            evictionRunIntervalMillis: 1500000
        },
        _options: {
            enableArithAbort: true
        },
        get options() {
            return this._options;
        },
        set options(value) {
            this._options = value;
        },
    });
    return conn;
};
module.exports = connect;

/*
Important note to check in your connection:
Built-in Service
    Local System
    Local Service
    Network Service <- This is my SQL configuration installation
*/


Create a file in sub directory controller
productController.js
var express = require('express');
var router = express.Router();
var sql = require("mssql");
var conn = require("../connection/connect")();

var routes = function () {
    router.route('/')
        .get(function (req, res) {
            conn.connect().then(function () {
                var sqlQuery = "SELECT * FROM Products";
                var req = new sql.Request(conn);
                req.query(sqlQuery).then(function (recordset) {
                    res.json(recordset.recordset);
                    conn.close();
                })
                    .catch(function (err) {
                        conn.close();
                        res.status(400).send(JSON.stringify(err));
                    });
            })
                .catch(function (err) {
                    conn.close();
                    res.status(400).send(JSON.stringify(err) );
                });
        });

    router.route('/')
        .post(function (req, res) {
            conn.connect().then(function () {
                var transaction = new sql.Transaction(conn);
                transaction.begin().then(function () {
                    var request = new sql.Request(transaction);
                    request.input("ProductName", sql.VarChar(50), req.body.ProductName);
                    request.input("ProductPrice", sql.Decimal(18, 0), req.body.ProductPrice);
                    request.execute("Usp_InsertProduct").then(function () {
                        transaction.commit().then(function (recordSet) {
                            conn.close();
                            res.status(200).send(req.body);
                        }).catch(function (err) {
                            conn.close();
                            res.status(400).send(JSON.stringify(err));
                        });
                    }).catch(function (err) {
                        conn.close();
                        res.status(400).send(JSON.stringify(err));
                    });
                }).catch(function (err) {
                    conn.close();
                    res.status(400).send(JSON.stringify(err));
                });
            }).catch(function (err) {
                conn.close();
                res.status(400).send(JSON.stringify(err));
            });
        });

    router.route('/:id')
        .put(function (req, res) {
            var _productID = req.params.id;
            conn.connect().then(function () {
                var transaction = new sql.Transaction(conn);
                transaction.begin().then(function () {
                    var request = new sql.Request(transaction);
                    request.input("ProductID", sql.Int, _productID);
                    request.input("ProductPrice", sql.Decimal(18, 2), req.body.ProductPrice);
                    request.execute("Usp_UpdateProduct").then(function () {
                        transaction.commit().then(function (recordSet) {
                            conn.close();
                            res.status(200).send(req.body);
                        }).catch(function (err) {
                            conn.close();
                            res.status(400).send(JSON.stringify(err));
                        });
                    }).catch(function (err) {
                        conn.close();
                        res.status(400).send(JSON.stringify(err));
                    });
                }).catch(function (err) {
                    conn.close();
                    res.status(400).send(JSON.stringify(err));
                });
            }).catch(function (err) {
                conn.close();
                res.status(400).send(JSON.stringify(err));
            });
        });

        router.route('/:id')
        .delete(function (req, res) {
            var _productID = req.params.id;
            conn.connect().then(function () {
                var transaction = new sql.Transaction(conn);
                transaction.begin().then(function () {
                    var request = new sql.Request(transaction);
                    request.input("ProductID", sql.Int, _productID);
                    request.execute("Usp_DeleteProduct").then(function () {
                        transaction.commit().then(function (recordSet) {
                            conn.close();
                            res.status(200).json("ProductID:" + _productID);
                        }).catch(function (err) {
                            conn.close();
                            res.status(400).send(JSON.stringify(err));
                        });
                    }).catch(function (err) {
                        conn.close();
                        res.status(400).send(JSON.stringify(err));
                    });
                }).catch(function (err) {
                    conn.close();
                    res.status(400).send(JSON.stringify(err));
                });
            });
        });

    return router;
};
module.exports = routes;

Create a file in sub directory PRODAPI
Server.js

var express = require('express');
var app = express();
var port = process.env.port || 1440;

var bodyParser = require('body-parser');

// create application/x-www-form-urlencoded parser
app.use(bodyParser.urlencoded({ extended: true }));

// create application/json parser
app.use(bodyParser.json());

var productController = require('./Controller/ProductController')();
app.use("/api/products", productController);

app.listen(port, function () {
    var datetime = new Date();
    var message = "Server runnning on Port:- " + port + "\nStarted at :- " + datetime;
    console.log(message);
});


In command prompt:
 
Browser Test
http://localhost:1440/api/products
 

Postman Test

 
 

 

 
 
Now we have learned how to create Node.js API in a simple step, we have started with creating Node.js application, after that we have to create a simple GET API in server.js file. Next we have created a product controller in that we have created route and move entire logic of API in to this controller, and also in SQL Server we have created stored procedure for Inserting, updating and deleting data. Finally, we have used POSTMAN for testing our API which we have created.
I hope you have liked my article to kick start Node.js. In the next article, you will learn how to validate API Request.











