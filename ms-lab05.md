![alt text][RX-M LLC]

Microservices
==============================


## Lab 5. Messaging and Loosely Coupled Systems

Amazon Simple Queue Service (SQS) is a reliable, scalable, fully-managed message queuing service. Amazon SQS uses a
distributed architecture within Amazon's high-availability data centers, so queues will be available when applications
need them. To prevent messages from being lost, all messages are stored redundantly across multiple servers and data
centers. Amazon SQS enables an unlimited number of services to read and write an unlimited number of messages at any
time.

Amazon also offers an IoT service specifically designed to provide secure transmission of MQTT messages from device
gateways. AWS IoT can publish to SQS making it easy to integrate IoT device traffic with microservices which consume SQS
messages.

In this lab we will use an SQS queue and simulate IoT gateway traffic, generating random trash can level reports to
test our service. The trash can level service will consume the device reports and (later) expose an interface enabling
other services to retrieve trash can levels.


### 1. Creating an SQS Queue

To begin we will need to create a queue for our message traffic. We can create, list and delete queues using the aws
CLI. The "aws sqs" subcommand has create-queue, list-queues and delete-queue commands. A standard SQS queue might be a
good choice for this application. Standard queues can support massive throughput, however they may deliver a message
twice in some rare corner cases. Since writing a trash can level twice is harmless in our use case, this seems like a
good trade. FIFO queues (the other supported queue type) are guaranteed to deliver messages in order and only once, at
the expense of throughput. It is important to select the appropriate queue type up front because it cannot be changed.

Another thing we might want to consider when creating the queue is the holding period. Let's assume our trash cans
report in every hour. The default queue holding time for SQS messages is 2 days. Why would we want trash can levels from
2 days ago? The only thing we would ever need is the most recent level. If our trash level service crashes the last
thing we want it to have to do when it restarts is read 2 days of old trash levels. We'll create a queue with a maximum
holding period of 1 hour (3600 seconds).

Use the aws client to create a trash-level queue with your initials (replacing the "wra" initials used in the example):

```
user@ubuntu:~$ aws sqs create-queue --queue-name trash-level-wra --attributes '{"MessageRetentionPeriod":"3600"}'
{
    "QueueUrl": "https://us-west-1.queue.amazonaws.com/433017611331/trash-level-wra"
}
user@ubuntu:~$
```

You can always use the list queues command to display your queue URLs:

```
user@ubuntu:~$ aws sqs list-queues
{
    "QueueUrls": [
        "https://us-west-1.queue.amazonaws.com/433017611331/trash-level-wra"
    ]
}
user@ubuntu:~$
```


### 2. Creating a gateway traffic simulator

In our development cycle we want to be able to simulate trash can level data before we actually have our trash can IoT
network running. In this step we'll build a simulator to generate random trash can level data and send it to SQS.
Imagine that in the trash level reporting bounded context we have decided to use JavaScript. So we'll create our trash
can level report simulator using NodeJS.

Begin by creating a working directory for the trash can level reporting code:

```
user@ubuntu:~/trash-can$ mkdir report
user@ubuntu:~/trash-can$ cd report/
user@ubuntu:~/trash-can/report$
```

Next create a simple program to simulate IoT messages from trash cans containing their ID and percentage full:

```
user@ubuntu:~/trash-can/report$ vim sim.js
user@ubuntu:~/trash-can/report$ cat sim.js
```
```JavaScript
var AWS = require('aws-sdk');
var url = "https://us-west-1.queue.amazonaws.com/433017611331/trash-level-wra";
var queue = new AWS.SQS({apiVersion: '2012-11-05', region: 'us-west-1', params: {QueueUrl: url}});


function trash_level() {
    var lvl = Math.random() * 100;
    var can_id = (Math.random() * 1000).toFixed();
    var msg = '{"can_id":"' + can_id + '", "level":"' + lvl + '"}';
    console.log('publishing: ' + msg);
    queue.sendMessage({MessageBody: msg}, function (err, data) {
        if (err) console.log('sendMessage failed:' + err);
    });
}


setInterval(trash_level, 4000);
```
```
user@ubuntu:~/trash-can/report$
```

This simple program generates a random trash level and a random 3 digit can ID and then packs it into a JSON message
for transmission to the SQS queue. the NodeJS setInterval() function calls our message generator every 4 seconds
creating a constant stream of messages.

Our code begins by requiring the JavaScript aws-sdk. This provides us with a queue object we can use to send and receive
messages. We initialize the queue with our queue URL, the region we created the queue in and the API version we want to
use. The current AWS SQS API version for JavaScript is 2012-11-05.

Now we can use a Node JS container to test our program:

```
user@ubuntu:~/trash-can/report$ docker run -it -v ~/trash-can/report:/app --name=sim -e AWS_ACCESS_KEY_ID=AKIAIE54Z4AUBXNVVW2Q -e AWS_SECRET_ACCESS_KEY=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  node /bin/bash
Unable to find image 'node:latest' locally
latest: Pulling from library/node
386a066cd84a: Pull complete
75ea84187083: Pull complete
88b459c9f665: Pull complete
1e3ee139a577: Pull complete
bd2400e7d880: Pull complete
b8c6812ba469: Pull complete
1882bd83135c: Pull complete
Digest: sha256:873c24bdb94e9b198bbec3c298fb7b2dfc64bce7b2dea71348b62f47814e16fb
Status: Downloaded newer image for node:latest
root@89bc5e566d9e:/#
```

The command we issued runs the node image from Docker Hub and mounts our working directory with our sim.js program into
the container using the -v switch. We also set the AWS key and secret environment variables needed by the SDK to
authenticate us. Unless you make policy changes to your queue, only the queue's creator can use it.

The /bin/bash argument executes a bash shell with an interactive tty (-it) so that we can configure our application.

Use the node package manager to install the NodeJS AWS SDK in the container:

```
root@89bc5e566d9e:/# npm install aws-sdk
npm info it worked if it ends with ok
npm info using npm@3.10.9
npm info using node@v7.1.0

...

npm info lifecycle aws-sdk@2.7.5~postinstall: aws-sdk@2.7.5
/
`-- aws-sdk@2.7.5
  +-- buffer@4.9.1
  | +-- base64-js@1.2.0
  | +-- ieee754@1.1.8
  | `-- isarray@1.0.0
  +-- crypto-browserify@1.0.9
  +-- jmespath@0.15.0
  +-- querystring@0.2.0
  +-- sax@1.1.5
  +-- url@0.10.3
  | `-- punycode@1.3.2
  +-- xml2js@0.4.15
  `-- xmlbuilder@2.6.2
    `-- lodash@3.5.0

npm WARN enoent ENOENT: no such file or directory, open '/package.json'
npm WARN !invalid#1 No description
npm WARN !invalid#1 No repository field.
npm WARN !invalid#1 No README data
npm WARN !invalid#1 No license field.
npm info ok
root@89bc5e566d9e:/#
```

We can ignore the warnings, "info ok" means we're all set. Note that we have not installed Node, NPM or the AWS SDK on
our host system.

Run your simulator:

```
root@89bc5e566d9e:/# node /app/sim.js
publishing: {"can_id":"333", "level":"30.65517941767528"}
publishing: {"can_id":"783", "level":"48.39960843546247"}
publishing: {"can_id":"777", "level":"59.4727637457233"}
publishing: {"can_id":"131", "level":"31.789566396147695"}
publishing: {"can_id":"968", "level":"1.193210915398879"}
publishing: {"can_id":"111", "level":"73.35593870113127"}
publishing: {"can_id":"182", "level":"87.46566203990642"}
publishing: {"can_id":"51", "level":"19.156573712062407"}
publishing: {"can_id":"208", "level":"90.97420104829932"}
publishing: {"can_id":"582", "level":"40.2300373916086"}
publishing: {"can_id":"905", "level":"22.548580501105462"}
publishing: {"can_id":"667", "level":"98.4137410601164"}
publishing: {"can_id":"175", "level":"4.469152152295597"}
publishing: {"can_id":"823", "level":"80.78328000523909"}
publishing: {"can_id":"907", "level":"19.08970106954433"}
publishing: {"can_id":"374", "level":"13.905318338056238"}
publishing: {"can_id":"22", "level":"92.23543185756564"}
publishing: {"can_id":"985", "level":"76.44769283928574"}
publishing: {"can_id":"908", "level":"77.96205801727758"}
publishing: {"can_id":"477", "level":"66.49639039621418"}

...

```

Perfect!

Leave the simulator running. Because we are using messaging, our microservices are loosely coupled and
atomically deployable. The simulator has no idea whether the report server is running or not, nor does it care.
You can verify the message stream operation using the aws tool from your lab system in a new shell:

```
user@ubuntu:~/trash-can/report$ aws sqs receive-message --queue-url https://us-west-1.queue.amazonaws.com/433017611331/trash-level-wra
{
    "Messages": [
        {
            "Body": "{\"can_id\":\"274\", \"level\":\"75.40692033260528\"}",
            "ReceiptHandle": "AQEBNZe71H+KWHE6mBSfbJ0KaPYW0JHASBgJOl0yZrrQKBrhBh77XShuTx57i85G1tsWtLFGV3jNZE8gy6llVHVYYo1ezM5OGOoyxZ58J2FJCsqmy7rLIw57zxIpv96QbtlghwUZ4xBjDkE56jKwTII0lim8VOKsmcKB+hFbJfYkfFKrULb0tIKeNTa42w1AYX0n7GiLs1+TsbxxH02qF+vmJzPB9wWX0I7iAfa3b7QLLDt3JFA/swqVecqD5bmmCvr6cbARfbccrrdSPrw12EBCnh4bhTuyKZljiRoDARDPFX6bwcEqTnFQaCC0L1VTORjd3FkZW+almFBZ/pGuyRokXdmlh2ZPQ+hAZzxDABeEPD8XAVbiTGt1dgAo3tRMADWxb6P5nNAaj59GxyCby062ng==",
            "MD5OfBody": "eb5fb945a2586322f3e811832dc89e5a",
            "MessageId": "80c6a5d3-55dd-48f0-b6f4-1ee4f0a01eda"
        }
    ]
}
user@ubuntu:~/trash-can/report$
```

For each message returned, the response includes the following:
- Message body
- Receipt handle
- MD5 digest of the message body
- Message ID (this is also returned to the sender of the message)
- Message attributes (if any)
- MD5 digest of the message attributes (if any)

Amazon SQS doesn't automatically delete a message after retrieving it for you, in case you don't successfully receive
the message (for example, the receiver could crash or lose connectivity during message retrieval). To delete a message,
you must send a separate request which acknowledges that you no longer need the message because you've successfully
received and processed it. The ReceiptHandle is used to delete a message once received.


### 4. Creating a trash level report service

Next we need to create the trash level reporting service. The AWS ReceiveMessage method retrieves one or more messages,
with a default of 1 and a maximum limit of 10 messages, from the specified queue. Long poll support is enabled by using
the WaitTimeSeconds parameter. Long polling allows you to specify a number of seconds to wait for messages to be
returned.

Short poll is the default behavior where a weighted random set of machines is sampled on a ReceiveMessage call. This
means only the messages on the sampled machines are returned. If the number of messages in the queue is small (less
than 1000), it is likely you will get fewer messages than you requested per ReceiveMessage call. If the number of
messages in the queue is extremely small, you might not receive any messages in a particular ReceiveMessage response.

Our simple queue reader will use short polling and retrieve a single message (the default) at a time, looping over and
over to ensure continuous message flow.

Create the following rep.js file to implement the report server on your lab system:

```
user@ubuntu:~/trash-can/report$ vim rep.js
user@ubuntu:~/trash-can/report$ cat rep.js
```
```JavaScript
var AWS = require('aws-sdk');
var url = "https://us-west-1.queue.amazonaws.com/433017611331/trash-level-wra";
var queue = new AWS.SQS({apiVersion: '2012-11-05', region: 'us-west-1', params: {QueueUrl: url}});

setInterval(function (){
    for (x=0; x<10; x++) {
        queue.receiveMessage({}, function(err, msgs) {
            if (err) console.log(err, err.stack);
            else {
                if (msgs.Messages && msgs.Messages.length == 1) {
                    console.log('Received report: ' + msgs.Messages[0].Body);
                    queue.deleteMessage({"ReceiptHandle": msgs.Messages[0].ReceiptHandle},
                                        function(err, data){;});
                } else return;
            }
        });
    }
},1000);
```
```
user@ubuntu:~/trash-can/report$
```

Our message reader uses the same AWS SDK and queue code as our writer. The reader however, requires much more complex
message operations. First of all, we do not want to sit in a tight loop in JavaScript programs as they only have one
thread to use for executing user code. So we read a maximum of 100 messages before letting a 100 millisecond interval
go by (setInterval(...,100)).

Next we use the receiveMessage() function to attempt to retrieve a message. If this fails we display the error and try
again. If the read succeeds but the Messages array is not present, there were no messages, so we exit the read loop
until the next interval. We could make things more efficient by increasing the number of messages we will accept each
cycle to 10 and by setting a time out (say 10 seconds) to enable long polling. As it is we hit the server very
frequently. For now we'll leave the code simple to make it easy to read and understand.

The last thing you may note is that, after a successful read, the program calls deleteMessage(). Unlike many systems,
reading a message from SQS does not delete it, rather it hides it. The default visibility timeout is 30 seconds, after
which the message will become visible again and other programs (or this one!) could read it again.

Here's an example queue console showing a queue with several messages which have been read but not deleted [Messages in
Flight (Not Visible)]:

![Hidden Messages](./images/in-flight.png)

In distributed systems, producer/consumer queues are difficult to design for single delivery and high scalability.
Since many use cases (like ours) prefer the at least once guarantee with high performance over the exactly one or at
most once guarantee, and because AWS can not know when we have successfully committed the message internally within our
application the burden is on us to delete it when finished. If we do not AWS assumes we have crashed or been partitioned
and allows some other application to process the message.

Run another Node container, install the AWS JavaScript SDK and test run the report server code (make sure the simulator
is still running in another shell and container):

```
user@ubuntu:~/trash-can/report$ docker run -it -v ~/trash-can/report:/app -e AWS_ACCESS_KEY_ID=AKIAIE54Z4AUBXNVVW2Q -e AWS_SECRET_ACCESS_KEY=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 --name=rep node /bin/bash

root@ea6b2e94ec16:/# npm install aws-sdk
npm info it worked if it ends with ok
npm info using npm@3.10.9
npm info using node@v7.1.0

...

npm info ok

root@ea6b2e94ec16:/# node /app/rep.js
Received report: {"can_id":"169", "level":"87.78012747959187"}
Received report: {"can_id":"341", "level":"5.03677337043944"}
Received report: {"can_id":"707", "level":"81.4820242189711"}
Received report: {"can_id":"63", "level":"71.42767411199134"}
Received report: {"can_id":"230", "level":"24.90842795780861"}
Received report: {"can_id":"221", "level":"72.29054832544676"}
Received report: {"can_id":"469", "level":"47.40519427370191"}
Received report: {"can_id":"367", "level":"27.29792541108438"}
Received report: {"can_id":"573", "level":"98.92857398205186"}
Received report: {"can_id":"415", "level":"41.90742748631913"}
Received report: {"can_id":"319", "level":"73.12309948409236"}
Received report: {"can_id":"283", "level":"4.054741063055078"}
Received report: {"can_id":"185", "level":"36.96278920655469"}
...
```

Great, we now have a trash level simulator and a loosely coupled reporting server.


### 5. Commit your code

We've created a fair bit of code. Let's use git to create a repository for our report project.

```
user@ubuntu:~/trash-can/report$ git init
Initialized empty Git repository in /home/user/trash-can/report/.git/
user@ubuntu:~/trash-can/report$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	rep.js
	sim.js

nothing added to commit but untracked files present (use "git add" to track)
user@ubuntu:~/trash-can/report$
```

Now add and commit your source code:

```
user@ubuntu:~/trash-can/report$ git add -A
user@ubuntu:~/trash-can/report$ git commit -m "initial trash level reporting service"
[master (root-commit) 4dd70e9] initial trash level reporting service
 2 files changed, 36 insertions(+)
 create mode 100644 rep.js
 create mode 100644 sim.js
user@ubuntu:~/trash-can/report$ git status
On branch master
nothing to commit, working directory clean
user@ubuntu:~/trash-can/report$
```


### [OPTIONAL] Create Docker images for the simulator and the report service

If you have time, package both of your services in container images. Push them to ECR if you like!


<br>

Congratulation you have successfully completed Lab 5!

<br>

_Copyright (c) 2013-2016 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
