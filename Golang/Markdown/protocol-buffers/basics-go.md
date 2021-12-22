---
title: 'Protocol Buffer Basics: Go'
date: '2021-12-22'
categories:
 - golang
 - protocol buffers
publish: true
---

# Protocol Buffer Basics: Go

This tutorial provides a basic Go programmer’s introduction to working with protocol buffers, using the `proto3` version of the protocol buffers language. By walking through creating a simple example application, it shows you how to 

- Define message formats in a `.proto` file
- Use the protocol buffer compiler
- Use the Go protocol buffer API to write and read message

## 1. Why use protocol buffers 

The example we are going to use is a very simple “address book” application that can read and write people’s contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:

- Use `gobs` to serialize the Go data structures. This is good solution in a Go-specific environment, but it doesn’t work well if you need to share data with applications written for other platforms
- You can invent an ad-hoc way to encode the data items into a single string - such as encoding 4 ints as “12:3:-23:67”. This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing impose a small run-time cost. This works best for encoding very simple data
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you wan to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be

Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code still read data encoded with the old format.

## 2. Defining your protocol format

To create your address book application, you will need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, the specify a name and a type for each field in the message. In our example, the `.proto` file that defines the message is `addressbook.proto`.

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects.

```protobuf
syntax = "proto3"
package tutorial

import "google/protobuf/timestamp.proto";
```

The `go_package` option defines the import path of the package w