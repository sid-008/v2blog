---
title: Creating a CRUD API with Go and MongoDB
date: 2023-03-20
description: Learn how to create a quick and easy CRUD API
---

# Setup
A CRUD API or a Create-Read-Update-Delete API is one of the first steps you take in your journey exploring the world of backend development. 

Get started with Golang by installing it on your computer if you haven’t already done so.

This tutorial will require some basic knowledge of golang.

Next we will install the packages needed for creating the CRUD API. We'll be using Gin Gonic to route the API endpoints.

```
go get github.com/gin-gonic/gin
```

Now we install the mongoDB driver: 

```
go get go.mongodb.org/mongo-driver/mongo
```
# Directory Structure
This is what your project directory should look like.
```
.
├── Collection
│   └── getCollection.go
├── databases
│   └── database.go
├── go.mod
├── go.sum
├── main.go
├── model
│   └── model.go
└── routes
    ├── create.go
    ├── delete.go
    ├── read.go
    └── update.go
```

# Connecting MongoDB to Go
All you need is your local mongoDB URI to connect to your DB. It typically looks a little like this:

```go 
Mongo_URI = "mongodb://127.0.0.1:27017"
```
Or
```go
 Mongo_URL = "mongodb://localhost:27017"
```

In my case I have used the latter.

Now create a new file named database, and create a database.go file in it with the following content

```go
package database

import (
	"context"
	"fmt"
	"log"
	"time"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

func ConnectDB() *mongo.Client {
	Mongo_URL := "mongodb://localhost:27017"
	client, err := mongo.NewClient(options.Client().ApplyURI(Mongo_URL))

	if err != nil {
		log.Fatal(err)
	}

	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	err = client.Connect(ctx)
	defer cancel()

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("connected to mongodb")
	return client
}
```
The ConnectDB function establishes a connection with the DB and returns a new MongoDB Client object.

**IMPORTANT NOTE:**
It's best practice to hide environment variables like the database connection string in a .env file using the dotenv package. For tutorial purposes we will not be doing this.

# Create Database Collection
Mongo stores data in collections which is a way to interface with the underlying DB.

Make a new folder, called Collection in your project root. Now create a new Go file, getCollection.go.

This will be short.

```go
package getcollection

import "go.mongodb.org/mongo-driver/mongo"

func GetCollection(client *mongo.Client, collectionName string) *mongo.Collection {
	collection := client.Database("DB1").Collection("Post")
	return collection
}
```
All this code does is return the collection thats requested as shown in the codeblock

```
collection := client.Database("DB1").Collection("Post")
```
So here its returning the collection "Post" from "DB1".

# Create the Database model

Make the model folder in project root. Create a file named model.go in it.
Your model in this case is a blog post with its title.

```go
package model

import "go.mongodb.org/mongo-driver/bson/primitive"

type Post struct {
	ID      primitive.ObjectID
	Title   string
	Article string
}
```

This just defines what kind of data we will be putting into the DB.

# Creating the CRUD API
 Now we move onto actually defining the routes and their functionalities.
 
 Create a file named routes. Inside it create 4 go files as shown:

```
routes
    ├── create.go
    ├── delete.go
    ├── read.go
    └── update.go
```

# CREATE endpoint
Creation of an entry in the DB is done by a `POST` request. We could say that we are quite literally posting a request to our DB. 
```go
package routes

import (
	"context"
	"log"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	getcollection "github.com/sid-008/monGoCRUD/Collection"
	database "github.com/sid-008/monGoCRUD/databases"
	model "github.com/sid-008/monGoCRUD/model"
	"go.mongodb.org/mongo-driver/bson/primitive"
)

func CreatePost(c *gin.Context) {
	var DB = database.ConnectDB()
	var postCollection = getcollection.GetCollection(DB, "Posts")

	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	post := new(model.Post)
	defer cancel()

	if err := c.BindJSON(&post); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"message": err})
		log.Fatal(err)
		return
	}

	postPayload := model.Post{
		ID:      primitive.NewObjectID(),
		Title:   post.Title,
		Article: post.Article,
	}

	result, err := postCollection.InsertOne(ctx, postPayload)

	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"message": err})
		return
	}

	c.JSON(
		http.StatusCreated,
		gin.H{"message": "Posted Successfully", "Data": map[string]interface{}{"data": result}},
	)

}
```

The most important thing you need to understand about this code is that `c.BindJSON("post")` is a 'JSONified' model instance that calls each model field as postPayload; this goes into the database. 

Basically, remember that model we defined before? We use that to shape our request. This is the entire crux of the api we are building. It allows us to help the DB and request sender communicate with each other.


# Read endpoint
This is the code for the `READ` endpoint. This is done via a `GET` request.

```go
package routes

import (
	"context"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	getCollection "github.com/sid-008/monGoCRUD/Collection"
	database "github.com/sid-008/monGoCRUD/databases"
	model "github.com/sid-008/monGoCRUD/model"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
)

func ReadOnePost(c *gin.Context) {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	var DB = database.ConnectDB()
	var postCollection = getCollection.GetCollection(DB, "Posts")

	postId := c.Param("postId")
	var result model.Post

	defer cancel()

	objId, _ := primitive.ObjectIDFromHex(postId)

	err := postCollection.FindOne(ctx, bson.M{"id": objId}).Decode(&result)

	res := map[string]interface{}{"data": result}

	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"message": err})
		return
	}

	c.JSON(http.StatusCreated, gin.H{"message": "Success!", "Data": res})

}
```

The `postId` variable is a parameter declaration(these are also called query strings). It gets a document's object ID as `objId` as shown in the line

```go
objId, _ := primitive.ObjectIDFromHex(postId)
```

# Update endpoint

Now its time to use a `PUT` request. This is similar to how we created an entry in our DB. This time we update an existing entry using its unique object ID using our `PUT` request.

```go
package routes

import (
	"context"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	getcollection "github.com/sid-008/monGoCRUD/Collection"
	database "github.com/sid-008/monGoCRUD/databases"
	model "github.com/sid-008/monGoCRUD/model"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
)

func UpdatePost(c *gin.Context) {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	var DB = database.ConnectDB()
	var postCollection = getcollection.GetCollection(DB, "Post")

	postId := c.Param("postId")
	var post model.Post

	defer cancel()

	objId, _ := primitive.ObjectIDFromHex(postId)

	if err := c.BindJSON(&post); err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"message": err})
		return
	}

	edited := bson.M{"title": post.Title, "article": post.Article}

	result, err := postCollection.UpdateOne(ctx, bson.M{"id": objId}, bson.M{"$set": edited})

	res := map[string]interface{}{"data": result}

	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"message": err})
		return
	}

	if result.MatchedCount < 1 {
		c.JSON(http.StatusInternalServerError, gin.H{"message": "Data doesn't exist"})
		return
	}

	c.JSON(http.StatusCreated, gin.H{"message": "data updated!", "Data": res})
}
```

A JSON format of the model instance (which is post here) calls each model field from the database. The result variable uses the MongoDB $set operator to update a required document called by its object ID.

The `result.MatchedCount` condition prevents the code from running if there's no record in the database or the passed ID is invalid.

# Delete endpoint
Finally we have the delete endpoint. It removes an entry based on object ID passed as URL parameter

```go
package routes

import (
	"context"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	getcollection "github.com/sid-008/monGoCRUD/Collection"
	database "github.com/sid-008/monGoCRUD/databases"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
)

func DeletePost(c *gin.Context) {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	var DB = database.ConnectDB()
	postId := c.Param("postId")

	var postCollection = getcollection.GetCollection(DB, "Posts")
	defer cancel()
	objId, _ := primitive.ObjectIDFromHex(postId)
	result, err := postCollection.DeleteOne(ctx, bson.M{"id": objId})
	res := map[string]interface{}{"data": result}

	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"message": err})
	}

	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"message": "No data to delete"})
	}

	c.JSON(http.StatusCreated, gin.H{"message": "Article deleted successfully", "Data": res})
}
```
This code deletes a record using the `DeleteOne` function. The result.DeletedCount property helps us check if the DB is empty or requested object ID is invalid.

# Creating the API runner file
Now in `main.go` add the following code. This will handle the router execution for each endpoint.

```go
package main

import (
	"log"

	"github.com/gin-gonic/gin"
	"github.com/sid-008/monGoCRUD/routes"
)

func main() {
	router := gin.Default()

	router.POST("/", routes.CreatePost)

	router.GET("/getOne/:postId", routes.ReadOnePost)

	router.PUT("/update/:postId", routes.UpdatePost)

	router.DELETE("/delete/:postId", routes.DeletePost)

	err := router.Run("localhost: 3000")
	if err != nil {
		log.Fatal(err)
	}
}
```

Now you can run the server by typing the following command in your project root

```
$ go run main.go
```

There you have it! A fully working mongoDB + Go CRUD API

You can test the API using the curl command, postman or insomnia.















