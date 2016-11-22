![alt text][RX-M LLC]

Microservices
==============================


## Lab 6. Stateful Microservices

Applications composed of microservices contain both stateless and stateful services. It is important to understand the
constraints and limitations of implementing stateful services. If a service relies on state, it should be separated
into a dedicated container that’s easily accessible.

One of the key advantages of microservices is the ability to scale rapidly. Like other distributed computing
architectures, microservices scale better when they are stateless. Within seconds, multiple containers can be launched
across multiple hosts. Each container running an instance of a stateless service is autonomous and doesn’t acknowledge
the presence of other instances of the service. This makes it possible to precisely scale the required service.
Stateless container replicas are ephemeral and can be destroyed at will with no loss of application function.

A microservices-based application will inevitably also require stateful services, storing state in an cluster of
collaborating in-memory services or in an RDBMS or NoSQL store. Services encapsulating stateful storage are also
typically packaged in containers when working with microservices. Stateful services can use "volumes" and drivers such
as Flocker, DRBD, Blockbridge, HPE 3Par, etc. to make the persistence layer host independent. State can also be
offloaded to highly available cloud data stores like DynamoDB to provide a persistence layer.

Cloud-centric datastores, such as DynamoDB, Redis, Cassandra, and IBM’s Cloudant, maximize availability with minimal
delay on consistency.

In this lab we will turn our trash level report service into a stateful container backed by the persistent/durable
DynamoDB key value store. Our report service will form a bounding context around the DynamoDB Table it uses,
encapsulating and isolating the data store. This will make it possible to change the data store or its schema at will
without impacting other services.


### 1. Configure a Dynamo DB aggregate

So far all of our containers are tracking state in memory. This is great for performance but not durability. If we want
to make our state durable we need to use a stateful container pattern. With Docker this usually means running a suitable
data store for the data we have and mapping the data store container's storage directory to a volume or using a SaaS
based data store like DynamoDB.

Similar to other database management systems, DynamoDB stores data in table aggregates. DynamoDB tables container
"Items". An item is a group of attributes that is uniquely identifiable among all of the other items by a key. Each
item is composed of its key and as many other attributes as you choose.

When you create a table, in addition to the table name, you must specify the primary key of the table. As in other
databases, a primary key in DynamoDB uniquely identifies each item in the table, so no two items can have the same
key. When you add, update, or delete an item in the table, you must specify the primary key attribute values for that
item.

DynamoDB supports two different kinds of primary keys:

- Partition Key:  A simple primary key composed of one attribute (known as the partition or HASH key)
- Partition/Sort Key:  A composite primary key composed of two attributes, the partition key and the sort key (in this
model multiple items can have the same Partition key as long as they have unique sort keys; data within a partition is
sorted by the value in the sort key)

The partition key of an item is also known as its hash attribute. The term "hash attribute" derives from DynamoDB's
usage of an internal hash function to evenly distribute data items across partitions, based on their partition key
values. The sort key of an item is also known as its range attribute. The term "range attribute" derives from the way
DynamoDB stores items with the same partition key physically close together, in sorted order by the sort key value.

In DynamoDB, you can read data in a table by providing primary key attribute values. If you want to read the data using
non-key attributes, you can use a secondary index to do this. After you create a secondary index on a table, you can
read data from the index in much the same way as you do from the table. By using secondary indexes, your applications
can use many different query patterns, in addition to accessing the data by primary key values.

We can use the aws client to create a new DynamoDB table for our trash level reports. The dynamodb subcommand offers a
create-table command which requires a table name a key definition and definitions for the key attributes.

Attributes (the data stored in our items) must be set to one of three types:

- S - String
- N - Number
- B - Binary

To keep things flexible we'll use the trash can ID as our partition key and set the type to String.

The create-table command also requires read and write capacity values. These values express the maximum number of
strongly consistent reads and writes per second allowed before DynamoDB returns a ThrottlingException.

Use the aws client to create a table, substituting your initials for the wra initials in the example:

```
user@ubuntu:~/trash-can/report$ aws dynamodb create-table --table-name trash-levels-wra --attribute-definitions AttributeName=id,AttributeType=S --key-schema AttributeName=id,KeyType=HASH --provisioned-throughput ReadCapacityUnits=3,WriteCapacityUnits=3
{
    "TableDescription": {
        "TableArn": "arn:aws:dynamodb:us-west-1:433017611331:table/trash-levels-wra",
        "AttributeDefinitions": [
            {
                "AttributeName": "id",
                "AttributeType": "S"
            }
        ],
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "WriteCapacityUnits": 3,
            "ReadCapacityUnits": 3
        },
        "TableSizeBytes": 0,
        "TableName": "trash-levels-wra",
        "TableStatus": "CREATING",
        "KeySchema": [
            {
                "KeyType": "HASH",
                "AttributeName": "id"
            }
        ],
        "ItemCount": 0,
        "CreationDateTime": 1479677484.887
    }
}
user@ubuntu:~/trash-can/report$
```

Beyond the key definition, DynamoDB is schemaless, so we can insert any additional attributes we choose with each key.
In our case we'll want to insert the trash level today. However, it is nice to know that we can add arbitrary data in
the future without making any schema changes.


### 2. Create the stateful report service

Now we need to update our trash level report service to save its state in the DynamoDB. This way if the report
service crashes we can simply restart it somewhere else and reattach it to the DynamoDB as if nothing ever happened.
DynamoDB itself is implemented using robust HA technology and can be replicated across regions if desired.

Update your report server code to look something like this:

```
user@ubuntu:~/trash-can/report$ vim rep.js
user@ubuntu:~/trash-can/report$ cat rep.js
```
```JavaScript
var AWS = require('aws-sdk');
var url = "https://us-west-1.queue.amazonaws.com/433017611331/trash-level-wra";
var queue = new AWS.SQS({apiVersion: '2012-11-05', region: 'us-west-1', params: {QueueUrl: url}});
var table = new AWS.DynamoDB({apiVersion: '2012-08-10', region: 'us-west-1', params: {TableName: 'trash-levels-wra'}});

var http = require('http');
var express = require('express');
var app = express();

setInterval(function (){
    for (x=0; x<10; x++) {
        queue.receiveMessage({}, function(err, msgs) {
            if (err) console.log(err, err.stack);
            else {
                if (msgs.Messages && msgs.Messages.length == 1) {
                    console.log('Received report: ' + msgs.Messages[0].Body);
                    var rep = JSON.parse(msgs.Messages[0].Body);
                    var itemParams = {
                       Item: {
                         id: {S: rep.can_id},
                         level: {S: rep.level}
                       }
                    };
                    table.putItem(itemParams, function(err,data) {
                        if (err) console.log(err);
                    });
                    queue.deleteMessage({"ReceiptHandle": msgs.Messages[0].ReceiptHandle},
                                        function(err, data){if (err) console.log(err);});
                } else return;
            }
        });
    }
},1000);

app.get('/reports/:can_id', function(req, res) {
    table.getItem({Key: {id: {S: req.params.can_id}}}, function(err, data) {
        if (err || !data.Item || !data.Item.level || !data.Item.level.S) {
            console.log(err, data);
            res.statusCode = 404;
            return res.send('Error: Trash Can not found\n');
        } else {
            console.log(data); // print the item data
            res.send('{"can_id":"' + req.params.can_id + '", "level":"' + data.Item.level.S + '"}');
        }
    });
});

http.createServer(app).listen(9090, function() {
    console.log('Listening on port 9090');
});
```
```
user@ubuntu:~/trash-can/report$
```

Our new server code uses three libraries:
- aws-sdk: to provide access to our SQS queue and DynamoDB
- express: a nodeJS framework for building RESTful services
- http: used to create the web server our REST API will listen to

The SQS code is as before but now we save all of the received trash levels in the DynamoDB before we delete them. The
main block of code associated with the app.get() function is our express web services. We accept requests to the
"/reports" IRI for a given can_id, returning that can's level. We use the DynamoDB table putItem() and getItem()
methods to interact with the database.

This microservice is now following the CQRS pattern. Asynchronous messages (Commands) are received to update data and
Synchronous REST calls are used to retrieve it.


### 3. Start the simulation

To test our new report service we'll want to run the trash can report simulator. You can restart the container you used
in the previous lab as follows:

```
user@ubuntu:~/trash-can/report$ docker start -i sim
root@89bc5e566d9e:/# node /app/sim.js
publishing: {"can_id":"432", "level":"70.69237046207665"}
publishing: {"can_id":"550", "level":"29.907124165084564"}
publishing: {"can_id":"596", "level":"97.90823776039335"}
publishing: {"can_id":"594", "level":"41.13776602592276"}
...
```

If you deleted the simulation container you can follow the instructions in the prior lab to recreate and rerun it.


### 4. Run the new stateful report service

To run the new report service code we need to start a Node container and then install all of the required libraries
using the Node Package Manager (NPM). Then we can run our app. Remove the old report server container and run a new
container with the appropriate switches:

```
user@ubuntu:~/trash-can/report$ docker rm rep
rep
user@ubuntu:~/trash-can/report$ docker run -it -v ~/trash-can/report:/app -e AWS_ACCESS_KEY_ID=AKIAIE54Z4AUBXNVVW2Q -e AWS_SECRET_ACCESS_KEY=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 --name=rep node /bin/bash
root@a9cf8abdc5e8:/#
```

Next install the necessary libraries:

```
root@a9cf8abdc5e8:/# npm install aws-sdk
npm info it worked if it ends with ok
npm info using npm@3.10.9
npm info using node@v7.1.0

...

npm info ok
root@a9cf8abdc5e8:/# npm install express
npm info it worked if it ends with ok
npm info using npm@3.10.9
npm info using node@v7.1.0

...

npm info ok
root@a9cf8abdc5e8:/#
```

Finally run the new report service:

```
root@a9cf8abdc5e8:/# node /app/rep.js
Listening on port 9090

```

Great, our new report server is up and making use of the DynamoDB table!


### 5. Test the report service REST API

Now we can use curl to test the report service. In a new shell on your lab system, list the containers running:

```
user@ubuntu:~$ sudo docker ps
CONTAINER ID   IMAGE   COMMAND       CREATED             STATUS             PORTS   NAMES
f44b1041e111   node    "/bin/bash"   3 minutes ago       Up 3 minutes               sick_lalande
91fc7fb173f9   node    "/bin/bash"   About an hour ago   Up About an hour           naughty_austin
user@ubuntu:~$
```

In a practical setting we would create docker files for the simulator and report server and build them into images that
we could just run, but our ad hoc environment works fine for testing and development.

Next discover the IP address of your report server (substitute the ID for the container on your lab system):

```
user@ubuntu:~$ sudo docker inspect f44b1041e111 | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.18.0.5",
user@ubuntu:~$
```

Now use curl to retrieve one of the trash can reports (use an ID that you see from one of the logs):

```
user@ubuntu:~$ curl -s http://172.18.0.5:9090/reports/420
{"can_id":"420", "level":"88.81585003341274"}
user@ubuntu:~$
```

It works!

Now try an ID that doesn't exist:

```
user@ubuntu:~/trash-can/report$ curl -s http://172.18.0.5:9090/reports/-1
Error: Trash Can not found
user@ubuntu:~/trash-can/report$
```

Perfect!


*[OPTIONAL]*
Use the 'docker rm --force' command to delete all of your containers. Then rerun them and retrieve some old data (saved
before they were restarted) to demonstrate the stateful persistence of your solution.


<br>

Congratulation you have successfully completed Lab 6!

<br>

_Copyright (c) 2013-2016 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
