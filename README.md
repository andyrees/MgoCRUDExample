<script src="https://cdn.rawgit.com/google/code-prettify/master/loader/run_prettify.js"></script>
# MongoDB simple CRUD example in Go

### Overview

This is a quick CRUD example that I put together for the purpose of having a easy reference for MGO. It uses Go's version of OOP to:

1. Create an intial connection, that can be defer closed in the main function.
2. Copy the session across each object method, use it and defer close it in the method.

### Example

<pre class="prettyprint lang-Go">
package main

import (
	"fmt"
	"log"
	"time"

	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

// Person
type Person struct {
	Name  string `bson:"name" json:"name"`
	Phone string `bson:"phone" json:"phone"`
}

// MySession allows for shared connections
type MySession struct {
	mainSession *mgo.Session
}

func (s *MySession) CreateNewRecord(p Person) {
	fmt.Println("Creating new record")

	// Copy & defer close the main session here
	session := s.mainSession.Copy()
	defer session.Close()

	conn := session.DB("MYDB").C("CRUD")
	err := conn.Insert(p)
	if err != nil {
		log.Fatal(err)
	}
}

func (s *MySession) ReadRecord() {
	fmt.Println("Reading new record")

	// Copy & defer close the main session here
	session := s.mainSession.Copy()
	defer session.Close()

	conn := session.DB("MYDB").C("CRUD")

	result := Person{}
	err := conn.Find(bson.M{"name": "Eric"}).One(&result)
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("%+v\n", result)
}

func (s *MySession) UpdateRecord(p Person) {
	fmt.Println("Updating record")

	// Copy & defer close the main session here
	session := s.mainSession.Copy()
	defer session.Close()

	p.Name = "New Name"
	conn := session.DB("MYDB").C("CRUD")

	err := conn.Update(bson.M{"name": "Eric"}, bson.M{"$set": p})
	if err != nil {
		log.Fatal(err)
	}
}

func (s *MySession) DeleteRecord(p Person) {
	fmt.Println("Deleting new record")

	// Copy & defer close the main session here
	session := s.mainSession.Copy()
	defer session.Close()

	conn := session.DB("MYDB").C("CRUD")

	err := conn.Remove(bson.M{"name": "New Name"})
	if err != nil {
		log.Fatal(err)
	}
}

var mainDBConn *mgo.Session

// Create initial connection to MGO here
// hand over to global var mainDBConn
// and most importantly remember to defer close the conn
// in the main function
func init() {
	ms, err := mgo.Dial(mgoURL)
	if err != nil {
		log.Fatal(err)
	}
	mainDBConn = ms
}

const (
	mgoURL = "mongodb://127.0.0.1"
)

func main() {
	//create initial connection to MGO
	defer mainDBConn.Close()
	
	// Create a new MySession object and give it the main
	// MGO conn
	mys := MySession{
		mainSession: mainDBConn,
	}

	// Create a person new object
	person1 := Person{Name: "Eric", Phone: "12345"}

	// Create the record
	mys.CreateNewRecord(person1)
	time.Sleep(5 * time.Second)

	// Read the record
	mys.ReadRecord()
	time.Sleep(5 * time.Second)

	// Update the record
	mys.UpdateRecord(person1)
	time.Sleep(5 * time.Second)

	// Delete the record
	mys.DeleteRecord(person1)
	time.Sleep(5 * time.Second)
}

</pre>