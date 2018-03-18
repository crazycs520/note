# [MongoDB Manual](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)

# Install

## On Mac OS

```shell
brew install mongodb
```



# mongo shell

## start

start the mongo shell

```shell
mongo
```

display the datebase you are using 

```Shell
db
show dbs #show the all datebase
```

To switch databases, issue the `use <db>`,if the <db> don't exit,it will create a new database.

```shell
use test
show collections #show the all collections in the datebase
```

create a new collection when you insert, the following creates the collection `student`and insert the data

```shell
db.students.insert({
	name:'Jone',
	age: 12,
	home:'china'
});
```

you can use an alternate syntax to find the date

```Shell
db["students"].find();	
db.students.find();
db.getCollection("students").find()
```

Format Printed Results,add the `pretty()` to the operation:

```shell
db.students.find().pretty();
```

## insert

[`db.collection.insert()`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#db.collection.insert) inserts a single document or multiple documents into a collection.

```Shell
db.students.insert(
{
	name:'crazycs',
	age: 21,
	home:'HuNan'
});
```

`db.collection.insertMany([{},{}])` insert many document:

```Shell
db.students.insertMany([
{
	name:'Jak',
	age: 31,
	home:'Hongkon'
},
{
	name:'Marry',
	age: 09,
	home:'Japan'
}
]);
```









