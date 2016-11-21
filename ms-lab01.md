![alt text][RX-M LLC]

Microservices
==============================


## Lab 1 – Microservice Hello World

Microservices represent a specialization of service-oriented architecture (SOA) used to build cloud native  distributed
applications. Services in a microservice architecture are processes that communicate with each other over
technology-agnostic network interfaces. The microservices architectural style has been shaped by, and enables, the
DevOps movement, integrating particularly well with Agile teams and continuously deployed systems. Because of its focus
on incremental and localized changes, Microservices enable rapid innovation in large scale systems, where gridlock might
otherwise ensue.

In this lab we will build a simple service to explore and demonstrate some of the key properties of microservices using
the classroom supplied virtual lab machine. The class room VM is an Ubuntu 16.04 server with a basic desktop installed.
Versions of the lab system are available for Virtual Box and VMware desktop hypervisors. If you do not have a copy of
the lab VM, instructions for downloading it can be found here:

- https://s3-us-west-1.amazonaws.com/rx-m-class-setup/rx-m-lab-sys-setup.html
- N.B. If you cannot access the lab VM, any base Ubuntu 16.04 system will work

The Lab system user account is "user" and the password is "user" (with full sudo permissions).


### 1. Login to the Lab System

Start your classroom virtual machine and log in as "user" with the password "user".

![alt text](./images/desktop_login.png "Desktop Login")


### 2. Conceptualizing a microservice based system

Microservice based applications are known for being highly scalable. This makes them a good fit for systems that
interact with the growing population of connected things in the real world, like cell phones, cars and even trash cans.
The microservice architectural style is also a key attribute of cloud native applications, typically described as:

- Microservice oriented
- Container packaged
- Dynamically orchestrated

We will take a look at each of these aspects in this lab and the labs ahead.


#### 2.a The Trash Can App

Cites can save a large amount of money by simply knowing when their trash cans are full. This would avoid trash
overflowing on the streets and also keep the city from sending maintenance personal out to collect trash from trash cans
that are not full. In this lab and the labs ahead we will incrementally build a microservice based application to create
a cloud native solution to the trash problem in cities around the world.

*Problem Statement: Cities currently schedule trash pickups by calendar time not by necessity. Some trash cans overflow,
polluting, and some are checked by staff even though they are empty, wasting time and fuel.*

Our goal is to solve this problem.

Our initial system requirements include the following:
- Manage trash can inventory
- Track trash can locations
- Track trash levels in cans
- Produce optimized trash pick up routes for trash trucks

Optimally all of the trash cans deployed in our city will be "smart", and able to tell us their trash levels via
Low-Power Wide-Area Network (LPWAN). We can use an algorithmic approach for "dumb" trash cans by having staff report
trash levels at various time intervals, allowing us to predict trash levels with better than random accuracy.


#### 2.b System Design

Most software architects and designers working with modern technology prefer to avoid Big Design Up Front (BDUF).
However spending some time understanding the problem domain and establishing the basic subsystems involved is almost
always an essential first step. System design is the process of establishing the basic subsystems of the overall
solution and defining their relationships. These subsystems should map directly to the bounded contexts identified
within the problem domain.

> Bounded Context is a central pattern in Domain-Driven Design [DDD] (term coined by Eric Evans in his 2003 book). DDD
decomposes complex software systems into bounded contexts based on natural partitions in the real world problem domain.
Various other authors have suggested similar approaches over the years, for example James Rumbaugh's OMT breaks systems
down into subsystems with defined interfaces.

Also in this early design phase we may want to select some initial communications schemes, as well as programming
languages and tools. In a microservice architecture we generally want to let each development team build their services
the way they think best, after all, they are (or will be) the experts on implementing the services they create. This is
also a particularly Agile approach. However, for the entirety of the application to work well together we'll need to
think about the ways in which the services communicate, the types of data stores we want to use, which messaging
platforms we will work with and other similar things that give the overall application interoperability and control the
scope of systems we must support and monitor.

Let's assume that the City already has several software systems and front ends. In order to ensure that our trash app
can easily integrate with existing parts of the City's platform we will provide API based access to the trash app. In
essence the trash app is itself a bounded context. Imagine that after discussion we decide to make the interface to our
application as open as possible and that high performance is not a concern. Choosing *REST* as an API approach might
make sense in this situation.

While letting our developers work in programming languages they like and are familiar with is very important, languages
are complex and we don't want to have too many in a single system if we can avoid it. One or two is great. Three or four
could also be justified but the justification should be clear. Five or six could begin to produce diminishing returns,
making it hard for developers to change teams or debug/augment other services they need to use. Mastering a programming
language is no small task and lack of experience can produce bugs and leave us with no experienced coders to do the
debugging.

Let's assume our team is familiar with *Python* and that this is a suitable language for the requirements we have
established. For now, we'll focus on building our application in Python.


#### 2.c Microservice or monolith

To begin our project we may start by building several individual microservices or we may start by creating a single
monolithic service that solves the entire trash problem. Wait! Isn't this a class on microservices? Why on earth would
we build a monolith?

Often it is easier to create a basic solution for a given bounded context as a single program. It may not scale cleanly
or be easy to maintain but it can teach us many things we do not yet know about the application and the problem domain.
Once completed we can separate out the microservices, decomposing the monolith into its essential components. Our
existing problem is simple enough that we can probably start right out with services but this is not always the case.


#### 2.d The first service

Ward Cunningham (revered coder and inventor of the Wiki) suggests that you should always build the simplest thing that
could possible work first. While this advice is directed at the process of constructing software, it mirrors the
microservice/monolith thought process. The goal is to gain experience before making decisions that will be expensive to
countermand. It is easy to change the implementation of a service behind an interface, it is expensive to change the
responsibilities of services.

Perhaps the simplest thing we could build to get started with our trash app is a service that inventories all of the
trash cans in the city (meeting requirement 1). If we were to flesh out the requirement we might unearth a user story
something like this:

- As a city engineer
- I want to keep track of all of the city's trash cans
- So that I can efficiently deploy and service trash cans throughout the city

Following a brief interview, we discover that the city engineers are interested in tracking the following trash can
attributes:

- ID - The identifying number of the trash can itself
- Deployed - True if the trash can is deployed for use in the city or False if the trash can is in the warehouse
- Power Source - How is the trash can's embedded trash level detector powered (grid/solar/wind/battery/none)
- Latitude - The trash can's position to 1000s of a degree (e.g. 31.776, south latitudes are negative)
- Longitude - The trash can's position to 1000s of a degree (e.g. 35.217, west longitudes are negative)
- Capacity - How many liters of trash the container can hold

This trash can "model" is a conceptual representation. It is used within the bounded context of our application but also
consumed and produced by our users. In other words, users will want to retrieve trash can information and add new trash
cans to the system from time to time. This makes the trash can model part of our interface. In some API terms, the trash
can would be a resource, in others, a type, in still others, a class. Regardless of the nomenclature, we will need to
exchange these "trash can" things with the outside world. This means that the representation should be extensible.


#### 2.e Languages and Tools

So we are coding in *Python*, we are exposing a *REST* API and we need to inventory the city's trash cans. While it is
probably clear that the trash can data should be persisted we'll begin by simply storing the trash can data in memory.
This will allow us to grow confident about the size and shape of the trash can data before committing to a storage
platform. Also, and critically, it will keep us from letting the storage solution drive our design decisions. Ultimately
the business will evolve around its natural boundaries (bounded contexts), not around our database choices.

Creating a REST API from scratch would be reinventing the wheel. There are many good Python frameworks available for
this. For example *Flask*. Flask is a BSD licensed microframework for Python based on Werkzeug and Jinja 2. Werkzeug is
a toolkit for WSGI, the standard Python interface for web applications, and Jinja2 renders templates. Sounds like a fit.

Given that multiple teams may be building Python services, we may run into version of library conflicts. What if team A
wants (needs) to use Python 2.7 and team B uses Python 3.4? What is team C needs Python 3.3. What about conflicting
library versions? These are common problems. One popular way to solve these problems is with Python Virtual
environments. (What about containers you say? Try this first and you be happy to switch to containers when we cover
them in a later lab!)

Exchanging data over a REST API involves some form of data serialization. For REST APIs this typically involves XML or
JSON. While XML is mature and has wide spread tool support, it is not as easy to read by humans, takes longer to parse
and consumes more bytes than the equivalent JSON. Given the groundswell of support for JSON in the industry we will use
*JSON* exclusively.

When dealing with text formats like JSON it is also often important to select a character encoding scheme. Here again
the industry has shown wide spread support for utf-8 on the wire. We'll use utf-8 for all string data in our API
interfaces to make string exchange easy, compact and consistent.

We should also choose a source code management system, testing approaches and tools, CI/CD solutions, etc. Due to time
limitations, we'll restrict our additional tooling to a source code version control system. Git is the most popular tool
in this space so lets choose it for simplicity.

We have now selected and configured our key languages, tools and technologies for our microservice development:

- Microservice architecture
- Python Language for (at least some) service development
- REST API
- Python Virtualenv (to support multiple isolated Python version installs)
- Flask Python REST library
- JSON data serialization
- utf-8 character encoding
- Git version control


### 3. Setting up a development environment

Before we begin coding we will need to prepare a development environment, but which Python version will we use and which
Flask version? What if we need to change versions? What if other programs running on the system install incompatible
versions of Python or Flask?

Hmmm. Running lots of independent programs on the same computer could be a challenge. Yet creating a VM for each
microservice would be pretty inefficient.

One possible solution is to use a language specific isolation scheme. For example Ruby offers Ruby Virtual Environments
(RVM), NodeJS uses NPM to install libraries on a per project basis and Python offers Virtualenv. Virtualenv enables
multiple side-by-side installations of Python and its libs. We'll configure our initial development environment with
Virtualenv.

Open a terminal on your lab system and create a working directory for your Trash Can system.

```
user@ubuntu:~$ mkdir trash-can
user@ubuntu:~$ cd trash-can/
user@ubuntu:~/trash-can$
```

Now create a subdirectory for the inventory service:

```
user@ubuntu:~/trash-can$ mkdir inv
user@ubuntu:~/trash-can$ cd inv/
user@ubuntu:~/trash-can/inv$
```

Now we can install the Python PIP package manager (the standard tool for installing Python libraries):

```
user@ubuntu:~/trash-can/inv$ sudo apt-get install python-pip
...
```

We may not have the latest version of PIP when installed from Ubuntu packages. Ask PIP to upgrade itself in case a newer
version is available.

```
user@ubuntu:~/trash-can/inv$ sudo -H pip install --upgrade pip
Collecting pip
  Downloading pip-9.0.1-py2.py3-none-any.whl (1.3MB)
    100% |████████████████████████████████| 1.3MB 981kB/s
Installing collected packages: pip
  Found existing installation: pip 8.1.1
    Not uninstalling pip at /usr/lib/python2.7/dist-packages, outside environment /usr
Successfully installed pip-9.0.1
user@ubuntu:~/trash-can/inv$
```

Next install Python Virtual Environments using PIP:

```
user@ubuntu:~/trash-can/inv$ sudo -H pip install virtualenv
Collecting virtualenv
  Downloading virtualenv-15.1.0-py2.py3-none-any.whl (1.8MB)
    100% |████████████████████████████████| 1.8MB 733kB/s
Installing collected packages: virtualenv
Successfully installed virtualenv-15.1.0
user@ubuntu:~/trash-can/inv$
```

Now we can create a virtual environment for our inventory project:

```
user@ubuntu:~/trash-can/inv$ virtualenv venv
New python executable in /home/user/trash-can/inv/venv/bin/python
Installing setuptools, pip, wheel...done.
user@ubuntu:~/trash-can/inv$ ls -la
total 12
drwxrwxr-x 3 user user 4096 Oct  1 07:41 .
drwxrwxr-x 3 user user 4096 Oct  1 07:24 ..
drwxrwxr-x 6 user user 4096 Oct  1 07:41 venv
user@ubuntu:~/trash-can/inv$
```

This installs a shell script that we can use to set the Python executable and library path to our private venv
directory. To activate the virtual environment you will need to run this script (every time you want to run, debug or
test your Python program). Enable the Python virtual environment:

```
user@ubuntu:~/trash-can/inv$ . venv/bin/activate
(venv) user@ubuntu:~/trash-can/inv$ python --version
Python 2.7.12
(venv) user@ubuntu:~/trash-can/inv$ which python
/home/user/trash-can/inv/venv/bin/python
(venv) user@ubuntu:~/trash-can/inv$
```

This prepares a Python virtual environment and ensures that our project will always have a stable version of Python and
dependent packages. This is a nice utility and widely used but it will not protect us from all changes to the host
system that indirectly impact Python or its libraries. It is also not easily deployable. If someone needs to run our
Python application we will have to tar/zip the virtual environment and the app together before sharing it to guarantee
consistency. We'll look at better more flexible means for microservice packaging later.

As you work within the lab system remember that you can enable the virtual environment and disable it with the following
commands:

- `. venv/bin/activate`
- `deactivate`

For example:

```
(venv) user@ubuntu:~/trash-can/inv$ deactivate
user@ubuntu:~/trash-can/inv$ . venv/bin/activate
(venv) user@ubuntu:~/trash-can/inv$
```

Now that we have PIP installed and a virtual Python environment active, we can install the Flask framework into our
virtual environment:

```
(venv) user@ubuntu:~/trash-can/inv$ pip install Flask
Collecting Flask
  Downloading Flask-0.11.1-py2.py3-none-any.whl (80kB)
    100% |████████████████████████████████| 81kB 297kB/s
Collecting itsdangerous>=0.21 (from Flask)
  Downloading itsdangerous-0.24.tar.gz (46kB)
    100% |████████████████████████████████| 51kB 1.9MB/s
Collecting click>=2.0 (from Flask)
  Downloading click-6.6.tar.gz (283kB)
    100% |████████████████████████████████| 286kB 560kB/s
Collecting Werkzeug>=0.7 (from Flask)
  Downloading Werkzeug-0.11.11-py2.py3-none-any.whl (306kB)
    100% |████████████████████████████████| 307kB 600kB/s
Collecting Jinja2>=2.4 (from Flask)
  Downloading Jinja2-2.8-py2.py3-none-any.whl (263kB)
    100% |████████████████████████████████| 266kB 567kB/s
Collecting MarkupSafe (from Jinja2>=2.4->Flask)
  Downloading MarkupSafe-0.23.tar.gz
Building wheels for collected packages: itsdangerous, click, MarkupSafe
  Running setup.py bdist_wheel for itsdangerous ... done
  Stored in directory: /home/user/.cache/pip/wheels/fc/a8/66/24d655233c757e178d45dea2de22a04c6d92766abfb741129a
  Running setup.py bdist_wheel for click ... done
  Stored in directory: /home/user/.cache/pip/wheels/b0/6d/8c/cf5ca1146e48bc7914748bfb1dbf3a40a440b8b4f4f0d952dd
  Running setup.py bdist_wheel for MarkupSafe ... done
  Stored in directory: /home/user/.cache/pip/wheels/a3/fa/dc/0198eed9ad95489b8a4f45d14dd5d2aee3f8984e46862c5748
Successfully built itsdangerous click MarkupSafe
Installing collected packages: itsdangerous, click, Werkzeug, MarkupSafe, Jinja2, Flask
Successfully installed Flask-0.11.1 Jinja2-2.8 MarkupSafe-0.23 Werkzeug-0.11.11 click-6.6 itsdangerous-0.24
(venv) user@ubuntu:~/trash-can/inv$
```

This installs Flask and all of its dependencies into our localized venv directory.

When our service is required to receive or return data we will need to define a format. Modern REST services generally
use JSON for this purpose. We can make displaying JSON data easy with the JQ package. Install JQ:

```
user@ubuntu:~$ sudo apt-get install jq
...
```

We will use jq at various points in the labs ahead.


### 4. Building a test service

Let's code a very simple Flask service to test our installation. Once we have this hello service working we can set
about building a proper Trash Inventory service (in the next lab).

Using your favorite editor (VIM is good for the CLI folks, GEdit should work for the GUI inclined and you can always
install something else if you prefer) create the following Python program:

```
(venv) user@ubuntu:~/trash-can/inv$ vim inv.py
(venv) user@ubuntu:~/trash-can/inv$ cat inv.py
```
```Python
""" Trash Can Inventory Service
"""

from flask import Flask
from flask import jsonify
from flask import request

app = Flask(__name__)

VERSION = "0.1"

@app.route('/')
def version():
    """ Root IRI returns the API version """
    return jsonify(version=VERSION)
```
```
(venv) user@ubuntu:~/trash-can/inv$
```

This program begins by importing the Flask class from the Python Flask package. An instance of this class will be our
Web Service application. Next we import the jsonify function which we will use to return JSON representations of our
API responses. Finally the request import will allow us to inspect the request data sent by clients.

After the import statements we create an instance of the Flask class with the `__name__` argument, which in Python
resolves to the name of the module (.py file). Flask will look for templates, static files, and so on here. We also set
a constant API version of 0.1.

We then use the route() decorator to tell Flask what URL should trigger our function. The function simply returns the
versions string in this case. The jsonify function allows us to convert Python key value pairs into legal JSON and it
also sets the content type header to application/json in the response object.

To run the service we need to export the FLASK_APP environment variable (setting it to the file name of our Python
program). Then we can run Flask:

```
(venv) user@ubuntu:~/trash-can/inv$ export FLASK_APP=inv.py
(venv) user@ubuntu:~/trash-can/inv$ python -m flask run -p 8080
 * Serving Flask app "inv"
 * Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)
```

In the example above we ask Python to run the installed Flask module (-m flask) and then ask flask to run our service on
port 8080 (run -p 8080).

Flask reports that it is running the "inv" app and that it is listening on the local loopback interface (127.0.0.1),
port 8080. You can choose a host interface (other than the default loopback) using the -h switch.

Now open another terminal window and test your service with curl:

```
user@ubuntu:~$ curl -s localhost:8080
{
  "version": "0.1"
}
user@ubuntu:~$
```

We can use the curl -v verbose, or -vv very verbose, switches to see the HTTP headers and other data exchanged between
curl and our service:

```
user@ubuntu:~$ curl -vv localhost:8080/
*   Trying ::1...
* connect to ::1 port 8080 failed: Connection refused
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.47.0
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Content-Type: application/json
< Content-Length: 23
< Server: Werkzeug/0.11.11 Python/2.7.12
< Date: Sun, 02 Oct 2016 19:02:50 GMT
<
{
  "version": "0.1"
}
* Closing connection 0
user@ubuntu:~$
```

So far so good.


### 5. Checking the code into source control

At this point normal programmers will start becoming nervous. We have written several lines of code and yet we have not
checked this code into a source code control system to protect it.

For this project we'll use the git source code control system to ensure that we do not accidentally delete or overwrite
our work. To begin we'll use the git init command to create a repository for our code (the hidden .git folder):

```
(venv) user@ubuntu:~/trash-can/inv$ git init
Initialized empty Git repository in /home/user/trash-can/inv/.git/
```

The init command creates a hidden '.git' folder in your project directory. We can use the git status command to see the
state of our project:

```
(venv) user@ubuntu:~/trash-can/inv$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	inv.py
	inv.pyc
	venv/

nothing added to commit but untracked files present (use "git add" to track)
```

The status command displays all of the files that are not being tracked for version control by git. In our case the .py
file is our source code and the only thing we would like to track. The pyc is a temporary compiled version of our source
and the venv/ directory is the Python virtual environment. We can tell git to ignore pyc files and the venv directory by
adding them to a .gitignore file. Try it:

```
(venv) user@ubuntu:~/trash-can/inv$ echo "*.pyc" > .gitignore

(venv) user@ubuntu:~/trash-can/inv$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore
	inv.py
	venv/

nothing added to commit but untracked files present (use "git add" to track)
```

Adding `*.pyc` to the .gitignore caused git to no longer show .pyc files in its statistics. This did add the .gitignore
file itself to the list of unchecked in files. Because we don't want to lose this file we should commit it.

Now add the venv/ directory to the ignore list:

```
(venv) user@ubuntu:~/trash-can/inv$ echo "venv/" >> .gitignore

(venv) user@ubuntu:~/trash-can/inv$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore
	inv.py

nothing added to commit but untracked files present (use "git add" to track)
(venv) user@ubuntu:~/trash-can/inv$
```

Now git status shows us only the files that we care about, the python source code and our .gitignore.

Git allows you to stage a set of files to be committed at once. Stage the .gitignore and the .py file:

```
(venv) user@ubuntu:~/trash-can/inv$ git add .gitignore
(venv) user@ubuntu:~/trash-can/inv$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   .gitignore

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	inv.py

(venv) user@ubuntu:~/trash-can/inv$ git add inv.py
(venv) user@ubuntu:~/trash-can/inv$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   .gitignore
	new file:   inv.py

(venv) user@ubuntu:~/trash-can/inv$
```

Now both of our changed files are staged for commit. To commit them use the commit command:

```
(venv) user@ubuntu:~/trash-can/inv$ git commit -m "initial trash can commit"

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'user@ubuntu.(none)')
(venv) user@ubuntu:~/trash-can/inv$
```

As you can see, git is not interested in committing code from anonymous parties. As suggested in the output, add your
name and email address to the git configuration.

```
(venv) user@ubuntu:~/trash-can/inv$ git config --global user.email "me@example.com"
(venv) user@ubuntu:~/trash-can/inv$ git config --global user.name "Bat Man"
```

Now retry your commit:

```
(venv) user@ubuntu:~/trash-can/inv$ git commit -m "initial trash can commit"
[master (root-commit) 7515493] initial trash can commit
 2 files changed, 11 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 inv.py
(venv) user@ubuntu:~/trash-can/inv$
```

You can use the git log command to view the commits in your repository:

```
(venv) user@ubuntu:~/trash-can/inv$ git log
commit 75154931fe467f35521ab312dcc863da7ff73d44
Author: Bat Man <me@example.com>
Date:   Sat Oct 1 13:14:51 2016 -0700

    initial trash can commit
(venv) user@ubuntu:~/trash-can/inv$
```

If you know git already these steps were probably second nature. If not you will get a chance to pick up the basics as
we progress through the labs in this course. However learning Git properly is another class...

We now have a proper development environment and a tested microservice skeleton exposing a REST API with JSON formatted
data exchange. All checked into a Git repository. In the next lab we'll implement the Trash Can Inventory functionality
using the tools we have configured here.


<br>

Congratulation you have successfully completed Microservices Lab 1!

<br>

_Copyright (c) 2013-2016 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
