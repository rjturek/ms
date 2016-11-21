![alt text][RX-M LLC]

Microservices
==============================


## Lab 4. Modern RPC

Apache Thrift is a modern, microservice friendly RPC solution. Apache Thrift uses a C like IDL, but the Apache Thrift
IDL is designed to represent interfaces for cross language implementation. Also at the time that Thrift was being
developed, all languages in mainstream use included collections such as map, set and list. Their ubiquity is a testament
to their usefulness. Apache Thrift makes it possible to pass collections, structures, unions, and even exceptions to
identify various application level error cases encountered in the service handler.

Perhaps the most powerful feature of Apache Thrift is its interface evolution capability. By encoding each element of a
struct or parameter list with an ordinal value, Thrift gains the ability to add, and occasionally remove elements from
an interface without damaging compatibility with existing clients.

In this lab we will build a simple Python microservice for capturing and reporting on the trash can fill levels as well
as a test client using Apache Thrift.


### 1. Defining and building an interface

Most RPC systems require you to define your interface in IDL. This is usually done before or while you code your
service. This is actually beneficial because it allows you to focus on the interface and its business requirements, not
how it might be implemented. This interface will become the access layer for some bounding context within your
microservice based application.

For our service we need to be able to return all of the trash cans above a certain threshold and set trash can levels.
Our IDL might look something like this:

```
typedef i64 CanId

struct can_levels {
    1: i32 count
    2: map<CanId, double> can_levels
}

service CanLevels {
    can_levels get_cans_above_threshold(1: double percent_full)
    oneway void update_can_level(1: CanId can_id, 2: double percent_full)
}
```

The IDL begins with a type def for CanIds. This will ensure that developers always use the same IDL type for the CanId
semantic. Next we define a structure for a list of cans and fill levels, we can return this struct from functions. This
is important because RPC functions can typically only return one thing per call, by creating a struct we can include as
much data within that thing as we like.

The last element defined in the IDL is the service, CanLevels. This service has two functions, the first returns the can
levels for all cans over a given percent full threshold. The second function is a oneway function and can be called but
never returns, similar to an async message. This function updates a given can's level.

Create a new working directory for the levels service under the trash-can application directory:

```
user@ubuntu:~$ cd
user@ubuntu:~$ cd trash-can/
user@ubuntu:~/trash-can$ mkdir levels
user@ubuntu:~/trash-can$ cd levels
user@ubuntu:~/trash-can/levels$
```

Now create the thrift IDL file from the above listing:

```
user@ubuntu:~/trash-can/levels$ vim levels.thrift
user@ubuntu:~/trash-can/levels$ cat levels.thrift
typedef i64 CanId

struct can_levels {
    1: i32 count
    2: map<CanId, double> can_levels
}

service CanLevels {
    can_levels get_cans_above_threshold(1: double percent_full)
    oneway void update_can_level(1: CanId can_id, 2: double percent_full)
}
user@ubuntu:~/trash-can/levels$
```

We can now compile our Apache Thrift IDL for any number of languages. However we need to have access to the Apache
Thrift IDL compiler to build our IDL into code. The /usr/bin/thrift compiler installed by system package managers like
yum and apt is often old. Fortunately there is a docker image on Docker hub for Apache Thrift.

Use the command below to build your IDL with the thrift docker image:

```
user@ubuntu:~/trash-can/levels$ docker run -v "$PWD:/data" thrift thrift -o /data --gen py /data/levels.thrift

Unable to find image 'thrift:latest' locally
latest: Pulling from library/thrift
0fbab137f56a: Pull complete
0e4a6fb76508: Pull complete
Digest: sha256:7f5e8f81d9e0d59792cb8039c150f29feda25fd285a974e06bce1a00881d37e0
Status: Downloaded newer image for thrift:latest

user@ubuntu:~/trash-can/levels$ ls -la
total 16
drwxrwxr-x 3 user user 4096 Oct  3 00:47 .
drwxrwxr-x 4 user user 4096 Oct  3 00:31 ..
drwxr-xr-x 3 root root 4096 Oct  3 00:47 gen-py
-rw-rw-r-- 1 user user  256 Oct  3 00:39 levels.thrift
user@ubuntu:~/trash-can/levels$
```

This docker command is a little complex so lets analyze it in parts:

- v: maps a volume (directory) from the host, in our case the current working directory ($PWD), to a path in the
  container, /data in this case
- thrift: in the first case is the name of the image repository on docker hub
- thrift: in the second case is the program inside the container we want to run
- -o: sets the thrift output directory
- --gen: tells thrift which language to emit, py for Python in our case
- /data/levels.thrift: is the path within the container to our IDL file

Because we have never run this image before it takes a moment to download from docker hub. Now that is has been pulled
it will execute instantly. The listing shows that thrift has created a gen-py directory with our service stubs for use
by clients and servers.

Notice that the gen-py directory is owned by root. This will be a problem when we try to use the files in gen-py with
our normal user. Container processes run as root by default. We need to change the owner of all of the files generated
back to our user id before we proceed:

```
user@ubuntu:~/trash-can/levels$ sudo chown -R user *
```


### 2. Create a Thrift microservice

Before we can progress with Apache Thrift for Python we will need to install the Thrift support libraries for our chosen
language. Use the pip command to install the Apache Thrift Python libraries at the system level:

```
user@ubuntu:~/trash-can/levels$ sudo -H pip install thrift
Collecting thrift
  Downloading thrift-0.9.3.tar.gz
Building wheels for collected packages: thrift
  Running setup.py bdist_wheel for thrift ... done
  Stored in directory: /root/.cache/pip/wheels/e5/20/32/cbe6d90e33b19825ea6d251ff0b1c778df8941750f5d5c3d3f
Successfully built thrift
Installing collected packages: thrift
Successfully installed thrift-0.9.3
user@ubuntu:~/trash-can/levels$
```

This installs version Thrift 0.9.3 which should be the same as our IDL compiler version. Re run the thrift image with
the version switch to verify this:

```
user@ubuntu:~/trash-can/levels$ docker run thrift thrift --version
Thrift version 0.9.3
```

Now enter the following code for the levels microservice:

```
user@ubuntu:~/trash-can/levels$ vim levels-server.py
user@ubuntu:~/trash-can/levels$ cat levels-server.py
```
```Python
import sys
sys.path.append("gen-py")

from thrift.transport import TTransport
from thrift.transport import TSocket
from thrift.protocol import TBinaryProtocol
from thrift.server import TServer

from levels import CanLevels
from levels import ttypes

wcans = {}

class CanLevelsHandler:
    def get_cans_above_threshold(self, percent_full):
        return ttypes.can_levels(count=2, can_levels=wcans)

    def update_can_level(self, can_id, percent_full):
        wcans[can_id] = percent_full


handler = CanLevelsHandler()
proc = CanLevels.Processor(handler)
trans_ep = TSocket.TServerSocket(port=9095)
trans_fac = TTransport.TBufferedTransportFactory()
proto_fac = TBinaryProtocol.TBinaryProtocolFactory()
server = TServer.TThreadedServer(proc, trans_ep, trans_fac, proto_fac)

print("[Server] Started")
server.serve()
```

The above version of our level server implements the CanLevels service interface with some test code. The updated levels
are saved in a dictionary called wcans, however all requests for can levels return all cans at present. We'll update the
server and integrate it with the rest of our application later.

Run your new Thrift microservice:

```
user@ubuntu:~/trash-can/levels$ python levels-server.py
[Server] Started

```

Perfect!


### 3. Create a Thrift microservice client

To test our RPC microservice we will create a simple client which submits two levels and then retrieves them. Open a
new terminal, change to the levels project directory and enter the following code into the levels-client.py file:

```
user@ubuntu:~/trash-can/levels$ vim levels-client.py
user@ubuntu:~/trash-can/levels$ cat levels-client.py
```
```Python
import sys
sys.path.append("gen-py")
from levels import CanLevels

from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol

trans = TSocket.TSocket("localhost", 9095)
trans = TTransport.TBufferedTransport(trans)
proto = TBinaryProtocol.TBinaryProtocol(trans)
client = CanLevels.Client(proto)

trans.open()
client.update_can_level(42, 0.85)
client.update_can_level(57, 0.89)
res = client.get_cans_above_threshold(0.8)
print("[Client] received: %d results" % (res.count))
print(res)
trans.close()
```

This Python client simply connects to the Thrift server on its listening port (9095), using the server's serialization
protocol, binary rather than JSON, and then begins to make normal function calls. The client sets two trash levels first
then retrieves all of the levels above 0.8 (80%).

Try running your client while the server is running in another terminal:

```
user@ubuntu:~/trash-can/levels$ python levels-client.py
[Client] received: 2 results
can_levels(count=2, can_levels={57: 0.89, 42: 0.85})
user@ubuntu:~/trash-can/levels$
```

While this may seem like a bit more work than our REST service it is probably about an order of magnitude faster. It is
also complete with IDL to document the interface contract. Notice we did not have to do any JSON to int/float
conversions, Thrift knows what the correct types are and ensures that only those types are accepted in the interface.
Also consider the fact that you would probably need to document your REST API using OAI or RAML, mirroring the IDL
effort made here with Thrift.

REST is king for request/response in public interfaces but many large internet firms use Thrift (Facebook, Twitter,
Evernote among them) or gRPC/ProtoBufs (Google, Docker, Square, among them) for internal services that are under heavy
load or are latency sensitive.


### [OPTIONAL] Challenge step

Create a REST version of the service in this lab then create a client that makes 1 million REST "update can level" calls
in a loop. Run this under the "time" command to see how long it takes. Then try the same thing with Apache Thrift.


<br>

Congratulation you have successfully completed Lab 4!

<br>

_Copyright (c) 2013-2016 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
