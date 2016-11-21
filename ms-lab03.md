![alt text][RX-M LLC]

Microservices
==============================


## Lab 3 – Container Packaging

Operating-system-level virtualization is a server virtualization method in which the kernel of an operating system
allows the existence of multiple isolated user-space instances, instead of just one. These instances are typically
called containers and, less often, software containers, virtualization engines (VEs) or jails. Containers look
and feel like a real servers from the point of view of the service within the container.

On Unix-like operating systems, particularly Linux, this technology can be seen as an advanced implementation of the
standard chroot mechanism, namespaces on Linux. In addition to isolation mechanisms, the kernel often provides
resource-management features to limit the impact of one container's activities on other containers, CGroups in Linux.

Docker is an open-source software system that automates the deployment of applications inside software containers.
Containers abstract an application's operating environment from the underlying operating system. Containers eliminate
the overhead and startup latency of a full virtual machine while preserving most of the isolation benefits.

Docker can package an application and its dependencies into an image, which can be used to launch a container on any
Linux system. This enables flexibility and portability, allowing applications to run reliably across a number of Linux
distributions with various configurations in a range of cloud settings.

The recent Microsoft release of Windows 2016 has brought Docker containers for Windows to production as well.

In this lab we will migrate our microservice to a Docker container image to simplify running and deploying it.


### 1. Install Docker

Start your classroom virtual machine and log in as “user” with the password “user”. Launch a terminal and run the
following command to install Docker:

```
user@ubuntu:~$ wget -qO- https://get.docker.com/ | sh
[sudo] password for user:
apparmor is enabled in the kernel and apparmor utils were already installed
+ sudo -E sh -c apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
Executing: /tmp/tmp.vbiu35TgBk/gpg.1.sh --keyserver
hkp://ha.pool.sks-keyservers.net:80
--recv-keys
58118E89F3A912897C070ADBF76221572C52609D
gpg: requesting key 2C52609D from hkp server ha.pool.sks-keyservers.net
gpg: key 2C52609D: public key "Docker Release Tool (releasedocker) <docker@docker.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
+ break
+ sudo -E sh -c apt-key adv -k 58118E89F3A912897C070ADBF76221572C52609D >/dev/null
+ sudo -E sh -c mkdir -p /etc/apt/sources.list.d
+ dpkg --print-architecture
+ sudo -E sh -c echo deb \[arch=amd64\] https://apt.dockerproject.org/repo ubuntu-xenial main > /etc/apt/sources.list.d/docker.list
+ sudo -E sh -c sleep 3; apt-get update; apt-get install -y -q docker-engine
Hit:1 http://us.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease [94.5 kB]
Get:3 http://us.archive.ubuntu.com/ubuntu xenial-updates InRelease [95.7 kB]       
Hit:4 http://us.archive.ubuntu.com/ubuntu xenial-backports InRelease                           
Get:5 https://apt.dockerproject.org/repo ubuntu-xenial InRelease [30.2 kB]
Get:6 http://us.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [424 kB]
Get:7 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages [2,553 B]
Get:8 http://us.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages [418 kB]
Get:9 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages [361 kB]
Get:10 http://us.archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages [358 kB]
Fetched 1,783 kB in 3s (488 kB/s)                     
Reading package lists... Done
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  aufs-tools cgroupfs-mount
The following NEW packages will be installed:
  aufs-tools cgroupfs-mount docker-engine
0 upgraded, 3 newly installed, 0 to remove and 143 not upgraded.
Need to get 19.4 MB of archives.
After this operation, 102 MB of additional disk space will be used.
Get:1 http://us.archive.ubuntu.com/ubuntu xenial/universe amd64 aufs-tools amd64 1:3.2+20130722-1.1ubuntu1 [92.9 kB]
Get:2 http://us.archive.ubuntu.com/ubuntu xenial/universe amd64 cgroupfs-mount all 1.2 [4,970 B]
Get:3 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 docker-engine amd64 1.12.3-0~xenial [19.3 MB]
Fetched 19.4 MB in 27s (710 kB/s)
Selecting previously unselected package aufs-tools.
(Reading database ... 124422 files and directories currently installed.)
Preparing to unpack .../aufs-tools_1%3a3.2+20130722-1.1ubuntu1_amd64.deb ...
Unpacking aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../cgroupfs-mount_1.2_all.deb ...
Unpacking cgroupfs-mount (1.2) ...
Selecting previously unselected package docker-engine.
Preparing to unpack .../docker-engine_1.12.3-0~xenial_amd64.deb ...
Unpacking docker-engine (1.12.3-0~xenial) ...
Processing triggers for libc-bin (2.23-0ubuntu3) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for systemd (229-4ubuntu10) ...
Setting up aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
Setting up cgroupfs-mount (1.2) ...
Setting up docker-engine (1.12.3-0~xenial) ...
Processing triggers for libc-bin (2.23-0ubuntu3) ...
Processing triggers for systemd (229-4ubuntu10) ...
Processing triggers for ureadahead (0.100.0-19) ...
+ sudo -E sh -c docker version
Client:
 Version:      1.12.3
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   6b644ec
 Built:        Wed Oct 26 22:01:48 2016
 OS/Arch:      linux/amd64

Server:
 Version:      1.12.3
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   6b644ec
 Built:        Wed Oct 26 22:01:48 2016
 OS/Arch:      linux/amd64

If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker user

Remember that you will have to log out and back in for this to take effect!

user@ubuntu:~$
```

Normal user accounts must use the sudo command to run command line tools as root. For our in-class purposes, eliminating
the need for sudo execution of the docker command will simplify our practice sessions. To make it possible to connect to
the local docker daemon domain socket without sudo we need to add our user id to the “docker” group. To add the “user”
user to the “docker” group execute the following command:

```
user@ubuntu:~$ sudo usermod -a -G docker user
```

Now display the ID and Group IDs for "user":

```
user@ubuntu:~$ id user
uid=1000(user) gid=1000(user) groups=1000(user),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),999(docker)
```

As you can see from the “id user” command, the account “user” is now a member of the “docker” group. Now try running
“id” without an account name:

```
user@ubuntu:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

While the “docker” group was added to your group list, your login shell maintains the old groups. After updating your
user groups you will need to restart your login shell to ensure the changes take effect. Reboot your system to complete
the installation.

```
user@ubuntu:~$ sudo reboot
...
```

While a reboot is not completely necessary it verifies that your system is properly configured to boot up with the
docker daemon running.

When the system has restarted log back in as “user” with the password “user”.


### 2. Run a test Container

To further test our system we will run a simple container. Use the docker run command to run the hello-world image:

```
user@ubuntu:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c04b14da8d14: Pull complete
Digest: sha256:0256e8a36e2070f7bf2d0b0763dbabdd67798512411de4cdcf9431a1feb60fd9
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

You now have a working Docker installation! Food for thought:

- Where might you get more examples and ideas for using Docker?
- Did the Docker Engine run the container from a hello-world image it found locally?
- In the docker output the hello-world image is listed with a "suffix", what is the suffix?
- What do you think this suffix represents?


### 3. Create a Dockerfile for your inventory microservice

Dockerfiles are usually placed at the root of a folder hierarchy containing everything you need to build one or more
docker images. This folder is called the build context. The entire build context will be packaged and sent to the Docker
Engine for building. In simple cases the build context will have nothing in it but a Dockerfile.

Navigate to your inventory project directory:

```
user@ubuntu:~$ cd trash-can/
user@ubuntu:~/trash-can$ cd inv/
user@ubuntu:~/trash-can/inv$
```

A Dockerfile contains a list of instructions for the Docker build command to perform when creating the target image. We
will build the following Dockerfile:

```
FROM ubuntu:16.04
RUN apt-get update -y && \
    apt-get install -y python-pip && \
    pip install --upgrade pip && \
    pip install flask && \
    rm -rf /var/lib/apt/lists/*
ENV FLASK_APP=inv.py    
COPY . /app
WORKDIR /app
ENTRYPOINT ["python"]
CMD ["-m", "flask", "run", "-h", "0.0.0.0", "-p", "8080"]
EXPOSE 8080
```

Dockerfile statements can be loosely organized into three groups:

* **Filesystem Setup** – Steps that copy or install files into the image’s file system (programs, libraries, etc)
* **Image Metadata** – Descriptive image data outside of the filesystem (exposed ports, labels, etc.)
* **Execution Metadata** – Commands and environment settings that tell Docker what to do when users run the image

Our Dockerfile has two statements in the **Image Metadata** category, FROM and EXPOSE. Dockerfiles begin with a FROM
statement which defines the parent image for the new image. We will base our service on the “Ubuntu 16.04” image, just
like the host we developed it on. You can create images with no parent by using the reserved name “scratch” as the
parent image (e.g. FROM scratch). No matter where users run our container (RedHat EL, SUSE, etc.), inside the container,
it will look like Ubuntu 16.04.

The EXPOSE instruction describes ports that the container’s service(s) typically listen on, in our case 8080.

The **File System Setup** parts of our Dockerfile will need to run apt-get commands to update the system and pip
commands to configure the necessary Python libraries. Note that because our service will now have its own private
container, we do not need to bother with Virtual environments, otherwise the commands are identical to those we ran on
the lab system earlier.

The RUN instruction is provided a string of commands to execute in sequence. This ensures that we update the
package indexes (apt-get update generates about 30MB in index data) and clear them (`rm .../lists/*`) before committing
the image each Dockerfile line creates. We also need to copy our Python program into the container image, the /app
directory is used in our Dockerfile at the destination directory.

The **Execution Metadata** section of our Dockerfile will need to setup an environment variable for the Flask app
(inv.py). This section also sets the working directory to /app and then runs "python" with the necessary parameters
to launch our service on port 8080, listening on all interfaces inside the container (-h 0.0.0.0).

Create a Dockerfile for your service as per below:

```
user@ubuntu:~/trash-can/inv$ vim dockerfile
user@ubuntu:~/trash-can/inv$ cat dockerfile
FROM ubuntu:16.04
RUN apt-get update -y && \
    apt-get install -y python-pip && \
    pip install --upgrade pip && \
    pip install flask && \
    rm -rf /var/lib/apt/lists/*
ENV FLASK_APP=inv.py    
COPY . /app
WORKDIR /app
ENTRYPOINT ["python"]
CMD ["-m", "flask", "run", "-h", "0.0.0.0", "-p", "8080"]
EXPOSE 8080
```

You can use the Dockerfile reference to look up any commands you would like more information on:
https://docs.docker.com/reference/builder

Before we build the dockerfile we need to create a .dockerignore file. This file serves the same purpose as the
.gitignore. If we do not tell docker to ignore certain files it will copy them into our image when the COPY instruction
runs. The only file we want to copy into our image is the inv.py file at present. Create the following .dockeringnore
file:

```
user@ubuntu:~/trash-can/inv$ vim .dockerignore
user@ubuntu:~/trash-can/inv$ cat .dockerignore
venv/
.git/
.gitignore
dockerfile
.dockerignore
*.pyc
```

This will keep the copy command from copying the .git and .venv Python virtual environment we previously installed into
the container image.

Now build your Docker image:

```
user@ubuntu:~/trash-can/inv$ docker build -t trash-can/inv:0.1 ~/trash-can/inv/
Sending build context to Docker daemon 6.656 kB
Step 1 : FROM ubuntu:16.04
16.04: Pulling from library/ubuntu

aed15891ba52: Pull complete
773ae8583d14: Pull complete
d1d48771f782: Pull complete
cd3d6cd6c0cf: Pull complete
8ff6f8a9120c: Pull complete
Digest: sha256:35bc48a1ca97c3971611dc4662d08d131869daa692acb281c7e9e052924e38b1
Status: Downloaded newer image for ubuntu:16.04
 ---> e4415b714b62
Step 2 : RUN apt-get update -y &&     apt-get install -y python-pip &&     pip install --upgrade pip &&     pip install flask &&     rm -rf /var/lib/apt/lists/*
 ---> Running in 1d3904883947

...

 ---> dc75435b1351
Removing intermediate container 1d3904883947
Step 3 : ENV FLASK_APP inv.py
 ---> Running in 68107879f748
 ---> a82200fc04a5
Removing intermediate container 68107879f748
Step 4 : COPY . /app
 ---> b652a9f5af01
Removing intermediate container c81cf4d1d23b
Step 5 : WORKDIR /app
 ---> Running in c0169398923c
 ---> 58aee51b68aa
Removing intermediate container c0169398923c
Step 6 : ENTRYPOINT python
 ---> Running in d9502014b04e
 ---> 19e29636c6b4
Removing intermediate container d9502014b04e
Step 7 : CMD -m flask run -h 0.0.0.0 -p 8080
 ---> Running in 108adc6a9133
 ---> 2dd1c7315e2d
Removing intermediate container 108adc6a9133
Step 8 : EXPOSE 8080
 ---> Running in 968cb6fc3c59
 ---> 68088503fd42
Removing intermediate container 968cb6fc3c59
Successfully built 68088503fd42
user@ubuntu:~/trash-can/inv$
```

This will take some time, as the Docker base image for Ubuntu:16.04 is fairly stripped down and many packages will need
to be install to support pip and flask respectively.

Use the docker images command to display the images on you lab system after the build completes:

```
user@ubuntu:~/trash-can/inv$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
trash-can/inv       0.1                 68088503fd42        About a minute ago   408.5 MB
ubuntu              16.04               e4415b714b62        3 days ago           128.1 MB
hello-world         latest              c54a2cc56cbb        4 months ago         1.848 kB
user@ubuntu:~/trash-can/inv$
```

The three images are the hello-world image we ran to test docker, the ubuntu image we based our service on and the
trash-can/inv repository which contains the image we tagged 0.1 during the docker build.

Perfect.


### 4. Run the containerized service

To run our service we want to tell docker to execute it in the background and to forward connections destined for port
8080 on the host machine to the container. We should also give the container a name, use "inv". Launch your service with
the following command:

```
user@ubuntu:~/trash-can/inv$ docker run --name=inv -d -p=8080:8080 trash-can/inv:0.1
77cc537b781afb433c549820a2f438670150a4a9afd06633976d494705435350
```

Use the docker ps command to view your container:

```
user@ubuntu:~/trash-can/inv$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
77cc537b781a        trash-can/inv:0.1   "python -m flask run "   6 seconds ago       Up 5 seconds        0.0.0.0:8080->8080/tcp   inv
user@ubuntu:~/trash-can/inv$
```

The ps command shows our container running, displays the image we used and the command that it ran as well as the port
mapping information.

You can use the docker logs command to view the console log:

```
user@ubuntu:~/trash-can/inv$ docker logs inv
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
```

Now try running a typical curl command to test the containerized microservice:

```
user@ubuntu:~$ curl -vv http://localhost:8080/
*   Trying ::1...
* Connected to localhost (::1) port 8080 (#0)
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
< Date: Mon, 03 Oct 2016 06:47:09 GMT
<
{
  "version": "0.1"
}
* Closing connection 0
user@ubuntu:~$
```


### 5. Clean up and commit

Now that we have an image built we can dispose of our microservice container without worries. We can run it again any
time we like from the image. We can also save the image to a file on the host with `docker save`, or push the image to
registry server with `docker push`.

For now we'll simply stop and delete all containers:

```
user@ubuntu:~/trash-can/inv$ docker rm --force `docker ps -aq`
77cc537b781a
...
user@ubuntu:~/trash-can/inv$
```

This command runs docker ps in a sub shell and then forces the removal of all containers listed. The -a switch lists
all containers, running or exited, and the -q switch outputs only the container ids.

Now run the git status command:

```
~/trash-can/inv$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.dockerignore
	dockerfile

nothing added to commit but untracked files present (use "git add" to track)
user@ubuntu:~/trash-can/inv$
```

We have yet to commit our dockerfile and the .dockerignore file. Add both to the git index and commit them to the
inventory service source code repo, they are now a critical part of our microservice build:

```
user@ubuntu:~/trash-can/inv$ git add .dockerignore
user@ubuntu:~/trash-can/inv$ git add dockerfile
user@ubuntu:~/trash-can/inv$ git commit -m "dockerfile and .dockerignore for trash can inventory service"
[master 3f46535] dockerfile and .dockerignore for trash can inventory service
 2 files changed, 19 insertions(+)
 create mode 100644 .dockerignore
 create mode 100644 dockerfile
user@ubuntu:~/trash-can/inv$ git status
On branch master
nothing to commit, working directory clean
user@ubuntu:~/trash-can/inv$
```


### 6. Push to the cloud

If we want to execute our microservice in the cloud we will need to push the service's container image to a registry
server accessible by the cloud we want to run on. The Amazon EC2 Container Registry (ECR) is a fully-managed Docker
container registry housed in AWS. ECR is integrated with AWS Identity and Access Management (IAM), providing
resource-level control of each Docker image repository.

In this step we'll push our container image to the AWS ECR so that we can run the image in later labs.

We will need to authenticate in order to push our image to AWS. We can use the pip installer to install the Python based
AWS client, which will allow us to generate an ECR login. Install the awsclient on your lab system:

```
user@ubuntu:~$ sudo -H pip install awscli
Collecting awscli
  Downloading awscli-1.11.19-py2.py3-none-any.whl (1.0MB)
    100% |████████████████████████████████| 1.0MB 141kB/s
Collecting s3transfer<0.2.0,>=0.1.9 (from awscli)
  Downloading s3transfer-0.1.9-py2.py3-none-any.whl (53kB)
    100% |████████████████████████████████| 61kB 5.4MB/s
Collecting botocore==1.4.76 (from awscli)
  Downloading botocore-1.4.76-py2.py3-none-any.whl (2.8MB)
    100% |████████████████████████████████| 2.8MB 404kB/s
Collecting rsa<=3.5.0,>=3.1.2 (from awscli)
  Downloading rsa-3.4.2-py2.py3-none-any.whl (46kB)
    100% |████████████████████████████████| 51kB 13.2MB/s
Collecting PyYAML==3.12 (from awscli)
  Downloading PyYAML-3.12.tar.gz (253kB)
    100% |████████████████████████████████| 256kB 2.7MB/s
Collecting colorama<=0.3.7,>=0.2.5 (from awscli)
  Downloading colorama-0.3.7-py2.py3-none-any.whl
Collecting docutils>=0.10 (from awscli)
  Downloading docutils-0.12.tar.gz (1.6MB)
    100% |████████████████████████████████| 1.6MB 892kB/s
Collecting futures<4.0.0,>=2.2.0; python_version == "2.6" or python_version == "2.7" (from s3transfer<0.2.0,>=0.1.9->awscli)
  Downloading futures-3.0.5-py2-none-any.whl
Collecting python-dateutil<3.0.0,>=2.1 (from botocore==1.4.76->awscli)
  Downloading python_dateutil-2.6.0-py2.py3-none-any.whl (194kB)
    100% |████████████████████████████████| 194kB 4.3MB/s
Collecting jmespath<1.0.0,>=0.7.1 (from botocore==1.4.76->awscli)
  Downloading jmespath-0.9.0-py2.py3-none-any.whl
Collecting pyasn1>=0.1.3 (from rsa<=3.5.0,>=3.1.2->awscli)
  Downloading pyasn1-0.1.9-py2.py3-none-any.whl
Collecting six>=1.5 (from python-dateutil<3.0.0,>=2.1->botocore==1.4.76->awscli)
  Downloading six-1.10.0-py2.py3-none-any.whl
Installing collected packages: futures, six, python-dateutil, docutils, jmespath, botocore, s3transfer, pyasn1, rsa, PyYAML, colorama, awscli
  Running setup.py install for docutils ... done
  Running setup.py install for PyYAML ... done
Successfully installed PyYAML-3.12 awscli-1.11.19 botocore-1.4.76 colorama-0.3.7 docutils-0.12 futures-3.0.5 jmespath-0.9.0 pyasn1-0.1.9 python-dateutil-2.6.0 rsa-3.4.2 s3transfer-0.1.9 six-1.10.0
user@ubuntu:~$
```

Now we can use the AWS client to authenticate with AWS (using the RX-M lab account) and then generate ECR registry
login credentials for use with Docker. Run the aws configure command as below (using the RX-M lab credentials as shown).

```
user@ubuntu:~$ aws configure
AWS Access Key ID [None]: AKIAIE54Z4AUBXNVVW2Q
AWS Secret Access Key [None]: 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
Default region name [None]: us-west-1
Default output format [None]: json
user@ubuntu:~$
```

This saves the credentials in the hidden ~/.aws directory:

```
user@ubuntu:~$ cat ~/.aws/credentials
[default]
aws_access_key_id = AKIAIE54Z4AUBXNVVW2Q
aws_secret_access_key = 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
user@ubuntu:~$ 
```

Now we can use the aws client to build a docker login command which will authenticate us with ECR:

```
user@ubuntu:~$ aws ecr get-login --region us-west-1
docker login -u AWS -p AQECAHijEFXGwF1cipVOacG8qRmJoVBPay8LUUvU8RCVV0XoHwAAAw0wggMJBgkqhkiG9w0BBwagggL6MIIC9gIBADCCAu8GCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMZeIk9Kyr95A4hMt3AgEQgIICwFsDpAeCw+6/hSJGBQtAs1krGiZThr/LxzRz4TsN7BwRJX8Q0j1nsH58Vz1s1bdjBYMoYUuBcaEn4ekiP/7Mxi6MmVay3YKfQEPirVKl5bbpROlbfR7h+fFAcI0dDctxBFqqGsm2jdbQgWpA173LlAK+uuScuxvr1UQScpLETELIzkboEDhRqqdfBqHzL41zmk18UOOl+P9Nq35snv6NlP6tYxbAq4H5LzJPtOM28eZsh1D6ukhVkNbxfLPIXYSpVvqP55j/lgUmVtyAgUNp5eGLzd0NMISlIpG78ziGptAFkexNdOdyk2dz3XI8+vJY3PEAUETzePyIyfL0WROdPw3euBi6QH2jHfCpMu6YDwAWbJagWOu1AtODS0D/ZMxASf+zJVwYjC6SINcfFJkbLBzejYmAb6tGnNCR9yqqjj07SiNZo8g90yjibt/pK8X3CBPQ47wSrAfOz7+Ibn3TZB8i9gVNf4kdHnuCW5fG0wqBe3C7gWOIvkN+CATVeD2l8Is3gHDCoC/i15WHDELWeQXMsFzT/i/6KCiP3afTn8dN71avopK6O5vSu0V4QLC25RGZ7gTEYJeWb6jBhqbPzUa8xmV4pN4c+eq6VWtf+M0OgSqf4IA7R9IAOo6HLKGX51CA/sDA0T7z27EN3XTPp/oRqRhCOrPBZos+JmZXfbKi7eKZn7/nF2Nwk3oy6QYlWda17egsi4kDSHsp3SgcoUWq0sDyK492p6D9HazeiLDy3AIGm0lYVBAJ8uUR+5JY+srf25yAolp4XoJw24jf+iCdCa5lC2Ju4N2JYbSabwW6BOreWkwKioWhkEDReKSC8zxDqgpJVgyD1Yjoa4KZLUMnQz2+B4VvolC0xyvu1hUkjraLgMEpeS2PmjPQqehSLJK4up2DBzPDYqIOHbse/0D7NE7Tx6wjtb8z49y+I+Pj -e none https://433017611331.dkr.ecr.us-west-1.amazonaws.com
user@ubuntu:~$
```

The -u switch sets the username, which is always AWS, and the -p switch sets the password, which is actually a secure
access token identifying your account to ECR. The last two arguments include the -e switch, which sets the optional
email address to none, and the IRI of the ECR. The -e switch can be removed as it does nothing in the current version of
docker and will be removed (generating an error in docker 1.13).

Run the docker login command to establish your ECR credentials:

```
user@ubuntu:~$ aws ecr get-login --region us-west-1 | sh
Flag --email has been deprecated, will be removed in 1.13.
Login Succeeded
user@ubuntu:~$
```

The docker login command asks the docker daemon to test the login against the registry IRI and, if successful, writes
the credentials to the hidden .docker directory in you account's home directory.

```
user@ubuntu:~$ ls -la
total 108
drwxr-xr-x 17 user user 4096 Nov 20 03:33 .
drwxr-xr-x  3 root root 4096 Aug  4 17:18 ..
drwxrwxr-x  2 user user 4096 Nov 19 19:37 .aws
-rw-------  1 user user 3328 Nov 19 17:50 .bash_history
-rw-r--r--  1 user user  220 Aug  4 17:18 .bash_logout
-rw-r--r--  1 user user 3771 Aug  4 17:18 .bashrc
drwx------ 10 user user 4096 Nov 19 16:32 .cache
drwx------ 12 user user 4096 Aug  4 17:24 .config
drwxr-xr-x  2 user user 4096 Aug  4 17:23 Desktop
-rw-r--r--  1 user user   25 Aug  4 17:23 .dmrc
drwx------  2 user user 4096 Nov 20 03:33 .docker
...
```

A file named config.json stores all of the login credentials you have established for various docker registries:

```
user@ubuntu:~$ ls -la .docker
total 12
drwx------  2 user user 4096 Nov 20 03:33 .
drwxr-xr-x 17 user user 4096 Nov 20 03:33 ..
-rw-------  1 user user 1564 Nov 20 03:33 config.json
user@ubuntu:~$ cat .docker/config.json
{
	"auths": {
		"https://433017611331.dkr.ecr.us-west-1.amazonaws.com": {
			"auth": "QVdTOkFRRUNBSGlqRUZYR3dGMWNpcFZPYWNHOHFSbUpvVkJQYXk4TFVVdlU4UkNWVjBYb0h3QUFBdzB3Z2dNSkJna3Foa2lHOXcwQkJ3YWdnZ0w2TUlJQzlnSUJBRENDQXU4R0NTcUdTSWIzRFFFSEFUQWVCZ2xnaGtnQlpRTUVBUzR3RVFRTXdPR2tpRDk3V1VCTjFEV0dBZ0VRZ0lJQ3dHaDU3MDJCT2VVS0tUajlCTmpvOTA0VkxSMGR5ci9QUUVmSksySW1vV3VSU3JUaENTcFEyMUlzRzRMMVNEd2cvZXAvbXVHVElsTUhPM05sMS9zREQxY21mVEhmUElVV0NuWTFHVUhENjEyOEVkaHRWaWZaaThCOG9UeHZ0Q3E3RWtWeW9HMWp4cjZzNkh3NGkzV2x5SkdtR1VjTUY2d3pCWTVoVW5BSEdiNkw1ZEpXSmVqYU4vRm1TNllsZFpMVmhpcnloRTA0R3h2TVcwdDZXenFyTVBoeUtQMFRlcXpzL1pSWXoydDlrby9mTzdwYzNqRHBtRzQ2ak1Iam9JYUVWMnRvY0VQTEtVdjIyVGNCZEYyazJtS3lZcTYrUUR6cjZCbFA3SHBQWmRFYTVBWjFoZzhxbTYvdm5heHpSWlAzTml5b2s3cDBxaGRnMUloTXhMUDEvMkZFVTMraUNuUkZXMXBYWmh6Um13YnZDN1EzR1NkY1VLZzZBaDc0dnJpdGR6cVZoOEQ2TEtyNXVLUU02eTNoRU9BRUlTRng3Q3gyMWtIZTQxUFRCR0pYUDU4NW95Z3VuOHlFdTdsUXVJMWlnS2RDYTRnTnFVa2UzNnI1cVdZdXFuNkRhYXZVVFdrVVlkR1YvTlUwQnRUMW9kV0tOUzhTbmg2Q2loOXc0RHU0bTRjdWt1TVExdEtxVElFTnptZEZYWUtkcnBZbkpCUWw3dThIMFV0bE1kaXFLNEdJekhWVjA4ckIrZEtZR0VmY3RwR3dNZnRFQWZaeVFtU0VjS0d0V0tObWVjMUdVbjlOUWhBMmMyYjlGVUUrQ0pza3VUYXRBTmZLNDhEZEJmVHI1U2ZKTG1kWXF3M2VvcS9LMkhkeStIMEtHZXl2cDhBRDBkdUpQbjl4em5KNmU2bnZhQnVJQXU1SGo3eUUvUWpxVExpUU44NXpUVTc0TXF6SExiNFIyOUFidE9BRk9lR1VGeWV5YXJHdnRpSW1xbnR3Q1p2WWlTSHdjSktEa2R2VWF6dENzamtqcko0TXZuQUNoSlQ1VHZ4bjlyT2FPM1hsMlNVc1AxZVJzSko4dWZwZjVMYUxTSTcwMVNlWW1aU1R4SGxHVkYrRFRMWVBOUFUwZmR1M2RWSnRCNG5OY1lCWjFSSnFnNkRNZU9qbitjVlluSHhoR1RhUFRwOUJOUUd3WXJXdmgwZ2hjbEp1TDdHZmptSmVNU3UrQmRsS1VkcXB4dklxSXlUcGErZG4="
		}
	}
}
user@ubuntu:~$
```

The "auth" key and value are passed as an HTTP header anytime you interact with the registry IRI identified,
"https://433017611331.dkr.ecr.us-west-1.amazonaws.com" in the example above.

When pushing docker images to a registry docker examines the repository name of the image to determine where to send it.
Repository names have three parts, the registry IRI, the namespace and the repository name. For example, the image name
"reg1.example.com/bobsmith/ubuntu:custom7" has these parts:

- Registry IRI:  reg1.example.com
- Namespace:  bobsmith
- Repository name:  ubuntu
- Image Tag:  custom7

Images with names like "ubuntu" or "bobsmith/ubuntu" will be pushed to Docker Hub, the default registry. To push our
image to ECR we will need to give it a name something like this:
"433017611331.dkr.ecr.us-west-1.amazonaws.com/bobsmith/inv:0.1"

The docker tag command allows you to create new names for an existing image. This allows you to setup many names for the
same image, allowing you to push it to multiple places.

List the images on your system:

```
user@ubuntu:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
trash-can/inv       0.1                 68088503fd42        9 hours ago         408.5 MB
ubuntu              16.04               e4415b714b62        3 days ago          128.1 MB
hello-world         latest              c54a2cc56cbb        4 months ago        1.848 kB
user@ubuntu:~$
```

Notice that trash-can/inv:0.1 has image ID 68088503fd42 (it may be different on your system). Now add a new tag name for
this image that is homed to ECR, using your initials as the namespace (in place of wra in the example) and trash-can as
the repo name:

```
user@ubuntu:~$ docker tag 68088503fd42 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv
user@ubuntu:~$ docker images
REPOSITORY                                                 TAG     IMAGE ID      CREATED       SIZE
433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can inv     68088503fd42  10 hours ago  408.5 MB
trash-can/inv                                              0.1     68088503fd42  10 hours ago  408.5 MB
ubuntu                                                     16.04   e4415b714b62  3 days ago    128.1 MB
hello-world                                                latest  c54a2cc56cbb  4 months ago  1.848 kB
user@ubuntu:~$
```

As you can see, both of our trash can image names point to the same image ID. Before we can push an image to ECR the
repository name must be created on AWS. You can create the repository using the AWS console (in the ECS section) or
using the aws command line utility. Create the repository using aws (again, substituting your initials for wra) and then
push your image using docker:

```
user@ubuntu:~$ aws ecr create-repository --repository-name wra/trash-can
{
    "repository": {
        "registryId": "433017611331",
        "repositoryName": "wra/trash-can",
        "repositoryArn": "arn:aws:ecr:us-west-1:433017611331:repository/wra/trash-can",
        "createdAt": 1479644771.0,
        "repositoryUri": "433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can"
    }
}

user@ubuntu:~$ docker push 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv
The push refers to a repository [433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can]
3ea5fd7e4fec: Pushed
516398e17d18: Pushed
c1bd37d01c89: Pushed
943edb549a83: Pushed
bf6751561805: Pushed
f934e33a54a6: Pushed
e7ebc6e16708: Pushed
inv: digest: sha256:080f9575e05a81f14c1e35c99cf71fea5353c49d725170ccb6cb4079da512ce0 size: 9059
user@ubuntu:~$
```

You can push as many image tags to your repository as you like once the repository is created. You can also use the aws
tool to display and delete images once they have been uploaded. To list your repositories try:

```
user@ubuntu:~/trash-can/levels$ aws ecr describe-repositories
{
    "repositories": [
        {
            "registryId": "433017611331",
            "repositoryName": "marvel/spiderman",
            "repositoryArn": "arn:aws:ecr:us-west-1:433017611331:repository/marvel/spiderman",
            "createdAt": 1479611401.0,
            "repositoryUri": "433017611331.dkr.ecr.us-west-1.amazonaws.com/marvel/spiderman"
        },
        {
            "registryId": "433017611331",
            "repositoryName": "wra/trash-can",
            "repositoryArn": "arn:aws:ecr:us-west-1:433017611331:repository/wra/trash-can",
            "createdAt": 1479644771.0,
            "repositoryUri": "433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can"
        }
    ]
}
user@ubuntu:~/trash-can/levels$
```

To list the images within a given repository try:

```
user@ubuntu:~/trash-can/levels$ aws ecr list-images --repository-name wra/trash-can
{
    "imageIds": [
        {
            "imageTag": "inv",
            "imageDigest": "sha256:080f9575e05a81f14c1e35c99cf71fea5353c49d725170ccb6cb4079da512ce0"
        }
    ]
}
user@ubuntu:~/trash-can/levels$
```

To get more image information use the describe command:

```
user@ubuntu:~/trash-can/levels$ aws ecr describe-images --repository-name wra/trash-can
{
    "imageDetails": [
        {
            "imageSizeInBytes": 166536619,
            "imageDigest": "sha256:080f9575e05a81f14c1e35c99cf71fea5353c49d725170ccb6cb4079da512ce0",
            "imageTags": [
                "inv"
            ],
            "registryId": "433017611331",
            "repositoryName": "wra/trash-can",
            "imagePushedAt": 1479644977.0
        }
    ]
}
user@ubuntu:~/trash-can/levels$
```


### 7. Test your cloud image

To ensure that our image was pushed properly we can delete it locally and then try to run the image with docker. When
docker fails to find the image locally it will automatically try to pull it from the embedded registry IRI.

First delete the image tag homed to ECR:

```
user@ubuntu:~$ docker images
REPOSITORY                                                  TAG     IMAGE ID      CREATED       SIZE
433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can  inv     68088503fd42  10 hours ago  408.5 MB
trash-can/inv                                               0.1     68088503fd42  10 hours ago  408.5 MB
ubuntu                                                      16.04   e4415b714b62  3 days ago    128.1 MB
hello-world                                                 latest  c54a2cc56cbb  4 months ago  1.848 kB

user@ubuntu:~$ docker rmi 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv
Untagged: 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv
Untagged: 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can@sha256:080f9575e05a81f14c1e35c99cf71fea5353c49d725170ccb6cb4079da512ce0

user@ubuntu:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
trash-can/inv       0.1                 68088503fd42        10 hours ago        408.5 MB
ubuntu              16.04               e4415b714b62        3 days ago          128.1 MB
hello-world         latest              c54a2cc56cbb        4 months ago        1.848 kB
user@ubuntu:~$
```

Now, even though the image is no available locally, we can run it with docker by passing the ECR homed repo name and
tag. Try it:

```
user@ubuntu:~$ docker run -d 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv
Unable to find image '433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv' locally
inv: Pulling from wra/trash-can
aed15891ba52: Already exists
773ae8583d14: Already exists
d1d48771f782: Already exists
cd3d6cd6c0cf: Already exists
8ff6f8a9120c: Already exists
baef339a14a0: Already exists
ed618a72ce3c: Already exists
Digest: sha256:080f9575e05a81f14c1e35c99cf71fea5353c49d725170ccb6cb4079da512ce0
Status: Downloaded newer image for 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv
eced46681b267d2e3e77c756ad2fc90853166176bfc6464d54aa3bb6ae44fe30
user@ubuntu:~$
```

Docker can not find the named image but when it contacts ECS and downloads the image manifest docker discovers that it
already has all of the image layers needed (recall that trash-can/inv:0.1 points tot the same bits), so docker simply
restores the repository name and tag and runs the image.

When you delete the last tag pointing to a given image id, docker actually removes the image bits from the local disk.
However, even then we could pull the image down from our ECR.

List stop and remove the container you just ran to complete the lab.

```
user@ubuntu:~$ docker ps
CONTAINER ID  IMAGE                                                          COMMAND                CREATED      
eced46681b26  433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:inv "python -m flask run " 3 minutes ago
user@ubuntu:~$ docker stop eced
eced
user@ubuntu:~$ docker rm eced
eced
user@ubuntu:~$ docker ps
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
user@ubuntu:~$
```


<br>

Congratulation you have successfully completed Lab 3!

<br>

_Copyright (c) 2013-2016 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
