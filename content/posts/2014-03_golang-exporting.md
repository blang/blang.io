---
title: Golang Exporting
date: 2014-03-28
comments: true
---

While i'm learning golang and searching the web for every possible talk and docs about it, i experienced that i can't find a good doc about the exporting features.
By export i'm refering to the mechanism to export types, method and properties to the world outside the package.

-----

## Exporting

You already might know that you can export types by starting its name with an upcase letter. But there are many edge cases like a public struct with private fields which i will talk about in this post.


### Everying public

Let's begin with a public struct with public fields.

```go
package north
type Record struct {
	Firstname string
	Lastname string
}
```


It's quite clear that outside the 'north' package you can access the type and all properties:

```go
package south
import "north"

// Init empty and set properties
i:=&north.Record{}
i.Firstname = "Foo"
i.Lastname = "Bar"

// Init with literal
i:=&north.Record{"Foo","Bar"}
```

It's also clear that if the type and the properties are private you can't instantiate the struct or access the properties directly, later more.

### Private properties

But what if the struct is public and the members are private?

```go
package north
type Record struct {
	firstname string
	lastname string
}
```

Now you can create a new instance of Record but can't use the literal to init the fields. Also there's no read or write access to those fields

```golang
package south
import "north"

// The empty literal is ok
i:=&north.Record{}

//Compile time error if accessing private fields
i.firstname = "Foo"
i.lastname = "Bar"

// Nope! Init with literal
// Compile time error
i:=&north.Record{"Foo","Bar"}
```

### Private type, public properties

But what if the type is private and the members are public?

```go
package north
type record struct {
	Firstname string
	Lastname string
}

func MakeRecord() record {
	return &record{}
}
```

Now you can't create a new instance outside the north package but you can access the properties of an existing instance!

```go
package south
import "north"

// Compile time error, private struct
i:=north.record{}

// Ok if instance is made inside the north pkg
i:=north.MakeRecord()
i.Firstname = "Foo"
i.Lastname = "Bar"
```

That's very useful if you need to initialize every new struct with non-default values like pointers.

### Everything private, but getter/setter

If the struct and its properties are private you can still provide access functions like getter and setter to better control the manipulation of the object.

```go
package north
type record struct {
	firstname string
	fastname string
}

func MakeRecord() record {
	return &record{}
}

func (r *record) SetFirstname(n string) {
	r.firstname = n
}

func (r *record) Firstname() string {
	return r.firstname
}
```

Now you can restrict and control the access via methods:

```go
package south
import "north"

i:=north.MakeRecord()
i.SetFirstname("Foo")
i.Firstname() // "Foo"

// Compile time error
i.firstname
```

Well and that's it. I really like go's exporting feature because it gives you quite a good feeling in the robustness of the system if entry-points of your api are secured by proper guards of object instantiation and manipulation.