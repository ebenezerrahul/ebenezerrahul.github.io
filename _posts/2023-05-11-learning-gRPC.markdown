---
layout: post
title:  "Learning gRPC"
date:   2024-06-09 02:58:01 +0530
categories: jekyll update
author : ebenezerrahul
---
[Ebenezer Rahul](https://medium.com/@ebenezerrahul?source=post_page-----c550a55195b3--------------------------------)

> Disclaimer: This is my understanding of the protocol. So it’s not a meant to be a replacement for documentation

Introduction
============

gRPC is a Remote Procedure Call Protocol for implementing APIs between services. It uses HTTP2 as its transport protocol and protocol buffers to transfer data.


<br>


### Why gRPC instead of REST

I have little experience with building APIs, but as I understand it, REST is a design architecture (i.e. a set of rules that the design of API needs to follow) for implementing robust APIs. In REST API you set up routes for each type of data you need to retrieve, along with specifications like :

1.  How your request object needs to be structured
2.  How your response object is structured

these details are provided in some documentation by the service (API). So if need to check things like is the client sending the right request obj structure or is sending to the right route you need to provide a module or driver which checks for these things and assists and abstracts the requests for the client

And if you are writing the server you need to set up your service so that your service is listening to all the routes mentioned in the specification and structure your response so as to meet the specification.

and as it's something just mentioned in some sort of documentation you need to be the one to write the driver for the client, server and also write them however many languages you want to support with your API.

Also in REST API data transfer happens using JSON OBJECTS although they are relatively easy and efficient to parse gRPC provides better a way with protocol buffers.

There are also some things you can do using gRPC that are hard/impossible to do using normal REST architecture things as streams, and bidirectional streams. But these are things I haven’t experimented with at all. so now let's learn how gRPC deals with these.

### PROTOCOL BUFFERS (.proto files)

gRPC uses protocol buffers to transfer data between processes (services). the data is serialised and sent over using HTTP2.

The .proto files are used to describe the structures of data and describe services and define procedures you want the services to support. gRPC uses .proto files as they describe them in a language-agnostic way. So gRPC looks at these .proto files and generates drivers to support the services and methods to serialize and deserialize the defined structures so it's efficient in size and also cuts time in parsing.

But the important detail is, as “.proto” files provide a formal way to specify your API or service. it is now possible to look at it and generate code (i.e. drivers for server and client in many languages)to provide the interesting abstraction described below. with all the features described as lacking in REST (above).

### HIGH-LEVEL View of gRPC

The basic idea is to provide the client with an abstraction like it is calling a local function using a stub (_def_: like in interfaces where we just provide method declaration and don’t provide its definition so we can inherit from it and implement it there its something like that) with the right parameters and the gRPC module or the driver generated to trigger the corresponding procedure in the server.

And for the server to define all these procedures as local functions and for the driver or module to help expose these procedures as remote procedures for the client to call.

### UNDERSTANDING BETTER

To understand it better it is better to look at some actual code and walk through it and get a sense of what we are working with.

First, let's look at a normal API where we share some normal data

Let's first take a look at the “.proto” file :

```protobuf
syntax = "proto3";  
  
package my_package;  
  
message request_str {   
  string mystr = 1;   
}  
  
message Index {   
  int32 index = 1;   
}  
  
service clientService {   
  rpc whichIndex(request_str) returns (Index);  
}  
  
service serverService {   
  rpc getResponse(Index) returns (request_str);   
}
```

Now let's dissect it, These .protofiles are used to specify protocol buffers. There are many languages in which we can describe them like languages like Python, proto3 etc...

The first line :

```protobuf
syntax = "proto3";
```

tells it to use “proto3" syntax to describe the specification, gRPC uses this specification as its language-agnostic which helps to generate code in many languages

Then the second line package my_package is used to kind of limit the namespace to the file I guess not too sure.

Now when it comes to describing a structure we use this syntax

```protobuf
message person{   
  string name= 1;   
  int32 age= 2;  
}
```

message as the keyword to define the structure and then it can have multiple fields with type and unique field name and a unique number. (i.e. 1 and 2 for name and age respectively) this is field num. every field number must be unique.

> side-note : if every field num must be unique then why doesn’t it assign it under the hood instead of making us write every time The reason is to provide backward compatibility and future comapatibility. we can reserve fields for future use. and consider this use case say i have a message having 4 feilds but now i decide feild4 to depericate and add new feild and write a edit .proto file but i still want to talk to servers which are using old defination but i want to be able to generate a performent driver for the new client then i can just remove the feild and add a new feild and give it feild5 now if I’m talking to old service if I can just ignore the feild having feild number 4 but still be able to talk to it and also talk to services using new defination of the structure. and regarding future compatibity you can reserve feild numbers with reserve keyword.

I’m not sure of this but apparently, there is a less cost of sterilizing and deserializing fields with lower field numbers. Now we can also specify a field to be required with the required keyword and mention optional fields with optional keywords as

```protobuf
message person{   
  required string name= 1;   
  optional int32 age= 2;  
}
```

This is a neat feature which rest couldn’t enforce (as the writer of the driver needs to enforce) but now the generated driver can enforce such things

Now we can specify services using The service keyword like :

```protobuf
message Location {  
  required int32 x = 1;  
  required int32 y = 2;  
}  
message Data{  
  string data = 1;  
}  
service mapService {   
  rpc searchLocation(Location) returns (Data);  
  rpc searchRestaurents(Location) returns (Data);  
}
```

In the above example, the map service provides two RPCs (remote procedure calls) one searchLocation and one searchRestaurents which both take in Location and return Data.

we can define multiple such services in a .proto file now we can compile it.

**Case Study**

Now for example look at this code [here](https://github.com/Ebenezer-Rahul/grpc-basic-connect).

Let’s just Look at the static-codegen part. Now for just a background, we have two processes server and client using gRPC to communicate.

Let's first look at [communicate.proto](https://github.com/Ebenezer-Rahul/grpc-basic-connect/blob/main/proto/communicate.proto) file :

```protobuf
syntax = "proto3";  
  
package my_package;  
  
message request_str { string mystr = 1; }  
  
message Index { int32 index = 1; }  
  
service clientService { rpc whichIndex(request_str) returns (Index); }  
  
service serverService { rpc getResponse(Index) returns (request_str); }
```

So we need to compile it to get two js files that are in the proto folder which describe services and messages.

I have just implemented serverService so let's focus on that for now. Let's look at the server.js file

```js
const services = require("./../proto/communicate_grpc_pb");  
const message = require("./../proto/communicate_pb.js");  
let grpc = require('@grpc/grpc-js');  
let serverService = services.serverServiceService;  
  
let createResponse = function (myNum) {  
  
    let mymessage = new message.request_str();  
  
    if(myNum.getIndex() % 2 == 0) {  
        mymessage.setMystr("Hello, World!");  
    } else {  
        mymessage.setMystr("Bye, World!");  
    }  
  
    return mymessage;  
}  
  
let getResponse = function (call , callback) {  
    let err = null;  
    callback(err , createResponse(call.request));  
    console.log("sent message!");   
}  
  
  
let server = new grpc.Server();  
  server.addService(  
      serverService , {  
        getResponse : getResponse    
      }   
  );  
  
server.bindAsync('localhost:5001', grpc.ServerCredentials.createInsecure(), () => {  
  server.start();  
  console.log("server started");  
})
```

This is a little straightforward if you look into it :

message object contains all the message structures you have defined like say request_str to create a request_str message you call :

```js
let mymessage = new message.request_str();
```

And to access the field of these objects like getMystr (capitalize the field name) and to set field use the setMystr() method on the object as its binary serialized.

Now services contain all the services here we want serverService But there are two of them one which provides the service and one which is the client so we have serverServiceService and serverServcieClient.

Note: I have named my services badly so it may be hard to follow but you’ll get it once you have given enough thought.

As it is the one proving the service we chose serverServiceService

```js
let server = new grpc.Server();  
  server.addService(  
      serverService , {  
        getResponse : getResponse    
      }   
  );  
  
server.bindAsync('localhost:5001', grpc.ServerCredentials.createInsecure(), () => {  
  server.start();  
  console.log("server started");  
})
```

I think that this part is straightforward the only thing here is that getResponse is needed to have two parameters. As once the client calls the getResponse RPC it passes to whatever function we have mapped the service with. here we mapped it with the getResponse function. it calls the get response function

This part is kind of important it’s called two parameters :

1.  call: this has the data regarding the actual RPC call made like what parameters (in request obj)
2.  callback: it is a function which takes in input as an error and response as its parameters.

I think it's a good exercise to think about why it’s the case it's implemented this way. I have one explanation but love to hear what you think.

When we come to the client side there are no surprises its simple enough.

```js
const services = require("./../proto/communicate_grpc_pb");  
const message = require("./../proto/communicate_pb.js");  
  
let grpc = require('@grpc/grpc-js');  
  
let client = new services.serverServiceClient("localhost:5001", grpc.credentials.createInsecure());  
console.log("connected to client");  
  
  
let i = 0;  
while (i < 5) {  
    let index = new message.Index();   
    index.setIndex(i);  
  
    client.getResponse(index, (err, message) => {  
        if(!err) {  
            console.log(err, message.getMystr());  
            return ;  
        }  
        console.log(err);  
    });  
  
    i++;  
}
```

I think this is from me on this there are a lot more things I haven’t explored like streams and bi-directional streams but I think I will come back to it later as I’m very much interested in it.

I also haven’t mentioned how exactly to compile .proto files as I had to look it up and I’m not sure I understand all the parameters well.

```bash
grpc tools node protoc --js_out=import_style=commonjs,binary:. --grpc_out=grpc_js: . communicate.proto
```

That’s it for today.
