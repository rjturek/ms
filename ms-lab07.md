![alt text][RX-M LLC]

Microservices
==============================


## Lab 7 – Orchestration

Amazon EC2 Container Service (ECS) is a highly scalable, high performance container management service that supports
Docker containers and allows you to easily run applications on a managed cluster of Amazon EC2 instances. Amazon ECS
eliminates the need for you to install, operate, and scale your own cluster management infrastructure. With simple API
calls, you can launch and stop Docker-enabled applications, query the complete state of your cluster, and access many
familiar features like security groups, Elastic Load Balancing, EBS volumes, and IAM roles. You can use Amazon ECS to
schedule the placement of containers across your cluster based on your resource needs and availability requirements.
You can also integrate your own scheduler or third-party schedulers to meet business or application specific
requirements.

There is no additional charge for Amazon EC2 Container Service. You pay for AWS resources (e.g. EC2 instances or EBS
volumes) you create to store and run your application.

In this lab we will create an ECS cluster and launch our trash reporting service in the cloud. This will be our first
demonstration of a truly could native solution:

- Microservice oriented [our design]
- Container packaged [Docker images]
- Dynamically managed [AWS ECS]

Before starting this lab stop and remove any containers you have on your lab system.

```
$ docker rm --force `docker ps -aq`
```


### 1. Initialize an ECS Cluster

 You can create an ECS cluster using the aws CLI ecs subcommand and the create-cluster command. This will create a new
 ECS cluster with no compute instances and therefore no capacity. If you do not specify a cluster name (using the
--cluster-name switch) a default cluster will be created. If you specify a cluster that already exists the command will
simply return the details of that cluster.

Try running the aws ecs create-cluster command on you lab system:

```
user@ubuntu:~$ aws ecs create-cluster
{
    "cluster": {
        "status": "ACTIVE",
        "clusterName": "default",
        "registeredContainerInstancesCount": 3,
        "pendingTasksCount": 0,
        "runningTasksCount": 0,
        "activeServicesCount": 0,
        "clusterArn": "arn:aws:ecs:us-west-1:433017611331:cluster/default"
    }
}
user@ubuntu:~$
```

You can tell this is a preexisting default cluster because it already has 3 compute [EC2] instances. ECS is not
available in all regions, ours is running in our default region, us-west-1 (evident from the Amazon Resource Name
[ARN]).

If you just want to list your cluster information without creating a cluster try the describe-clusters command:

```
user@ubuntu:~$ aws ecs describe-clusters
{
    "clusters": [
        {
            "status": "ACTIVE",
            "clusterName": "default",
            "registeredContainerInstancesCount": 3,
            "pendingTasksCount": 0,
            "runningTasksCount": 0,
            "activeServicesCount": 0,
            "clusterArn": "arn:aws:ecs:us-west-1:433017611331:cluster/default"
        }
    ],
    "failures": []
}
user@ubuntu:~$
```

You can list the Container Instances in your cluster with the list-container-instances command (AWS refers to EC2
instances that are part of a cluster as "Container Instances"):

```
user@ubuntu:~$ aws ecs list-container-instances
{
    "containerInstanceArns": [
        "arn:aws:ecs:us-west-1:433017611331:container-instance/0ccf6b14-257f-446f-9f34-d0ba924bae16",
        "arn:aws:ecs:us-west-1:433017611331:container-instance/8139b024-a72f-4dbf-8ec6-20778803e80f",
        "arn:aws:ecs:us-west-1:433017611331:container-instance/c56d8e33-b68a-406b-81b7-ebd88b5b9d97"
    ]
}
user@ubuntu:~$
```

You can also get details for a particular container instance. Try it:

```
user@ubuntu:~$ aws ecs describe-container-instances --container-instances arn:aws:ecs:us-west-1:433017611331:container-instance/0ccf6b14-257f-446f-9f34-d0ba924bae16
{
    "failures": [],
    "containerInstances": [
        {
            "status": "ACTIVE",
            "registeredResources": [
                {
                    "integerValue": 2048,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "CPU",
                    "doubleValue": 0.0
                },
                {
                    "integerValue": 3954,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "MEMORY",
                    "doubleValue": 0.0
                },
                {
                    "name": "PORTS",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "51678",
                        "51679"
                    ],
                    "type": "STRINGSET",
                    "integerValue": 0
                },
                {
                    "name": "PORTS_UDP",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [],
                    "type": "STRINGSET",
                    "integerValue": 0
                }
            ],
            "ec2InstanceId": "i-0e42ee51",
            "agentConnected": true,
            "containerInstanceArn": "arn:aws:ecs:us-west-1:433017611331:container-instance/0ccf6b14-257f-446f-9f34-d0ba924bae16",
            "pendingTasksCount": 0,
            "remainingResources": [
                {
                    "integerValue": 2048,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "CPU",
                    "doubleValue": 0.0
                },
                {
                    "integerValue": 3954,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "MEMORY",
                    "doubleValue": 0.0
                },
                {
                    "name": "PORTS",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "51678",
                        "51679"
                    ],
                    "type": "STRINGSET",
                    "integerValue": 0
                },
                {
                    "name": "PORTS_UDP",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [],
                    "type": "STRINGSET",
                    "integerValue": 0
                }
            ],
            "runningTasksCount": 0,
            "agentUpdateStatus": "UPDATED",
            "attributes": [
                {
                    "name": "com.amazonaws.ecs.capability.privileged-container"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.17"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.20"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.21"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.22"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.23"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.json-file"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.syslog"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
                },
                {
                    "name": "com.amazonaws.ecs.capability.ecr-auth"
                },
                {
                    "name": "com.amazonaws.ecs.capability.task-iam-role"
                }
            ],
            "versionInfo": {
                "agentVersion": "1.13.1",
                "agentHash": "efe53c6",
                "dockerVersion": "DockerVersion: 1.11.2"
            }
        }
    ]
}
user@ubuntu:~$
```

This listing shows you the total resources the instance has and what is available, the ports in use and several other
useful features. At the bottom the output displays the ECS agent version and the Docker version configured on the
system. To add a node to an ECS cluster you must configure the ECS agent and Docker. The ECS agent is pre configured on
Amazon ECS-optimized AMIs. The agent will join the "default" cluster unless you modify its configuration. ECS instances
also require the "ecsInstanceRole" IAM role.


### 2. Create a container image for your final report microservice

In the prior lab we created a trash level report service and a trash level simulator. Both of these applications were
based on NodeJS and depended on the aws-sdk. If we want to containerize these services for use on ECS we might consider
creating a shared base image with the aws-sdk installed. This will allow both the rep.js and sim.js images to share the
a single copy of the aws-sdk.

We can quickly create a base image using a docker file. Create docker image build contexts (directories) for the shared
base (repbase), the trash level report services (rep) and the simulator (sim):

```
user@ubuntu:~$ cd trash-can/
user@ubuntu:~/trash-can$ cd report/
user@ubuntu:~/trash-can/report$ ls -l
total 8
-rw-rw-r-- 1 user user 2030 Nov 20 15:30 rep.js
-rw-rw-r-- 1 user user  608 Nov 20 09:02 sim.js
user@ubuntu:~/trash-can/report$ mkdir repbase
user@ubuntu:~/trash-can/report$ mkdir rep
user@ubuntu:~/trash-can/report$ mkdir sim
user@ubuntu:~/trash-can/report$ cp rep.js rep
user@ubuntu:~/trash-can/report$ cp sim.js sim
user@ubuntu:~/trash-can/report$
```

Now create a dockerfile for the base image that installs the aws-sdk on top of NodeJS. We'll base our image on
node:7.1.0-alpine, a light weight linux environment with NodeJS 7.1 installed:

```
user@ubuntu:~/trash-can/report$ cd repbase/
user@ubuntu:~/trash-can/report/repbase$ vim dockerfile
user@ubuntu:~/trash-can/report/repbase$ cat dockerfile
FROM node:7.1.0-alpine
RUN npm install aws-sdk

user@ubuntu:~/trash-can/report/repbase$ docker build -t repbase:v0.1 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM node:7.1.0-alpine
7.1.0-alpine: Pulling from library/node
3690ec4760f9: Pull complete
abc5c88aafef: Pull complete
Digest: sha256:dad2f53e80a3d1f0920f6d58ffe8a675979160519232a0c6cf37563c6f4e1f56
Status: Downloaded newer image for node:7.1.0-alpine
 ---> b1b3e6b6395f
Step 2 : RUN npm install aws-sdk
 ---> Running in 62c32b0cd364
npm info it worked if it ends with ok

...

npm info ok
 ---> 2d4a7d48462a
Removing intermediate container 62c32b0cd364
Successfully built 2d4a7d48462a
user@ubuntu:~/trash-can/report/repbase$
```

Using the repbase we can create an image for our simulator and our report service. Next create an image for the
simulator:

```
user@ubuntu:~/trash-can/report/repbase$ docker images
REPOSITORY   TAG    IMAGE ID       CREATED         SIZE
repbase      v0.1   2d4a7d48462a   2 minutes ago   75.14 MB
...

user@ubuntu:~/trash-can/report/repbase$ cd ..
user@ubuntu:~/trash-can/report$ cd sim
user@ubuntu:~/trash-can/report/sim$ vim dockerfile
user@ubuntu:~/trash-can/report/sim$ cat dockerfile
FROM repbase:v0.1
COPY ./sim.js /
CMD node /sim.js

user@ubuntu:~/trash-can/report/sim$ docker build -t sim:v0.1 .
Sending build context to Docker daemon 3.584 kB
Step 1 : FROM repbase:v0.1
 ---> 2d4a7d48462a
Step 2 : COPY ./sim.js /
 ---> b9f015123f6e
Removing intermediate container 86e490f6a657
Step 3 : CMD node /sim.js
 ---> Running in 48283da5aac6
 ---> bf5f6d16430c
Removing intermediate container 48283da5aac6
Successfully built bf5f6d16430c
user@ubuntu:~/trash-can/report/sim$
```

Lastly, create an image for the trash level report service. The trash level report service depends on the aws-sdk from
rep-base but also requires the Express REST library:

```
user@ubuntu:~/trash-can/report/repbase$ cd ..
user@ubuntu:~/trash-can/report$ cd rep/

user@ubuntu:~/trash-can/report/rep$ vim dockerfile
user@ubuntu:~/trash-can/report/rep$ cat dockerfile
FROM repbase:v0.1
RUN npm install express
COPY ./rep.js /
CMD node /rep.js

user@ubuntu:~/trash-can/report/rep$ docker build -t rep:v0.1 .
Sending build context to Docker daemon 4.608 kB
Step 1 : FROM repbase:v0.1
 ---> 2d4a7d48462a
Step 2 : RUN npm install express
 ---> Running in ccf869e49c8c
npm info it worked if it ends with ok

...

npm info ok
 ---> 7ac55c81fbb8
Removing intermediate container ccf869e49c8c
Step 3 : COPY ./rep.js /
 ---> 1d6f9cfca038
Removing intermediate container e0612500409d
Step 4 : CMD node /rep.js
 ---> Running in c5c1d57ae623
 ---> d1012261acde
Removing intermediate container c5c1d57ae623
Successfully built d1012261acde
user@ubuntu:~/trash-can/report/rep$  
```

Display your three new Docker images:

```
user@ubuntu:~$ docker images
REPOSITORY   TAG    IMAGE ID       CREATED          SIZE
sim          v0.1   bf5f6d16430c   6 minutes ago    75.14 MB
rep          v0.1   d1012261acde   9 minutes ago    78.17 MB
repbase      v0.1   2d4a7d48462a   20 minutes ago   75.14 MB
```

Note that our base image is 75MB but the sim image adds almost no additional bytes. The rep image shares all of the base
bytes and adds another approximately 3MB for the express library. We can use the docker history command to view an
image's ancestry and the sizes of the various layers it is built on:

```
user@ubuntu:~/trash-can/report$ docker history sim:v0.1
IMAGE          CREATED       CREATED BY                                      SIZE
bf5f6d16430c   2 hours ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "node    0 B
b9f015123f6e   2 hours ago   /bin/sh -c #(nop) COPY file:f673a78535811592d   608 B
2d4a7d48462a   2 hours ago   /bin/sh -c npm install aws-sdk                  19.84 MB
b1b3e6b6395f   10 days ago   /bin/sh -c #(nop)  CMD ["node"]                 0 B
<missing>      10 days ago   /bin/sh -c addgroup -S node     && adduser -D   50.5 MB
<missing>      10 days ago   /bin/sh -c #(nop)  ENV NODE_VERSION=7.1.0       0 B
<missing>      10 days ago   /bin/sh -c #(nop)  ENV NPM_CONFIG_LOGLEVEL=in   0 B
<missing>      4 weeks ago   /bin/sh -c #(nop) ADD file:7afbc23fda8b0b3872   4.803 MB
user@ubuntu:~/trash-can/report$
```


### 3. Run the simulator image

Now that we have a proper image for our simulator we can run it to begin generating simulated trash can levels:

```
user@ubuntu:~$ docker run -d -e AWS_ACCESS_KEY_ID=AKIAIE54Z4AUBXNVVW2Q -e AWS_SECRET_ACCESS_KEY=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 --name sim sim:v0.1
79db6dc6f46ab3144ad80bcbab56e01a06572d4dfc490710e98078a1cf6baf05
user@ubuntu:~$
```

You can verify the operation of your simulator by displaying its log output:

```
user@ubuntu:~$ docker logs sim
publishing: {"can_id":"276", "level":"64.35707295866004"}
publishing: {"can_id":"500", "level":"88.9210609995484"}
user@ubuntu:~$
```


### 4. Push the report image to ECR

In this step we'll push the rep image to the ECR, create an ECS task for it and run the task on ECS.

To begin we need to retag the rep image to home it to the ECR registry and then push it into the AWS cloud:

```
user@ubuntu:~$ docker tag rep:v0.1 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:repv0.1
user@ubuntu:~$ docker push 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:repv0.1
The push refers to a repository [433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can]
ffbef747f373: Preparing
0e72f42a170b: Preparing
38e23dcbf498: Preparing
d5b66603fcd9: Preparing
011b303988d2: Preparing
denied: Your Authorization Token has expired. Please run 'aws ecr get-login' to fetch a new one.
```

In the above example our login failed. You can generate a new login token anytime your docker login token expires and
then retry the push (if your push succeeded you can skip this):

```
user@ubuntu:~$ aws ecr get-login | sh
Flag --email has been deprecated, will be removed in 1.13.
Login Succeeded

user@ubuntu:~$ docker push 433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:repv0.1
The push refers to a repository [433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can]
ffbef747f373: Pushed
0e72f42a170b: Pushed
38e23dcbf498: Pushed
d5b66603fcd9: Pushed
011b303988d2: Pushed
repv0.1: digest: sha256:6f77dc793b56f9e63486d53b3b5a1ff0eb0a5920f26075026ed618cc44a00b2a size: 7119
user@ubuntu:~$
```


### 5. Create an ECS Task

ECS tasks define one or more containers to run and other application level factors associated with the application. For
example, you can constrain the memory available to the container, define port mappings and other useful features.

You can use the aws ecs register-task-definition command with the --generate-cli-skeleton switch to display all of the
possible task options:

```
user@ubuntu:~$ aws ecs register-task-definition --generate-cli-skeleton
{
    "family": "",
    "taskRoleArn": "",
    "networkMode": "",
    "containerDefinitions": [
        {
            "name": "",
            "image": "",
            "cpu": 0,
            "memory": 0,
            "memoryReservation": 0,
            "links": [
                ""
            ],
            "portMappings": [
                {
                    "containerPort": 0,
                    "hostPort": 0,
                    "protocol": ""
                }
            ],
            "essential": true,
            "entryPoint": [
                ""
            ],
            "command": [
                ""
            ],
            "environment": [
                {
                    "name": "",
                    "value": ""
                }
            ],
            "mountPoints": [
                {
                    "sourceVolume": "",
                    "containerPath": "",
                    "readOnly": true
                }
            ],
            "volumesFrom": [
                {
                    "sourceContainer": "",
                    "readOnly": true
                }
            ],
            "hostname": "",
            "user": "",
            "workingDirectory": "",
            "disableNetworking": true,
            "privileged": true,
            "readonlyRootFilesystem": true,
            "dnsServers": [
                ""
            ],
            "dnsSearchDomains": [
                ""
            ],
            "extraHosts": [
                {
                    "hostname": "",
                    "ipAddress": ""
                }
            ],
            "dockerSecurityOptions": [
                ""
            ],
            "dockerLabels": {
                "KeyName": ""
            },
            "ulimits": [
                {
                    "name": "",
                    "softLimit": 0,
                    "hardLimit": 0
                }
            ],
            "logConfiguration": {
                "logDriver": "",
                "options": {
                    "KeyName": ""
                }
            }
        }
    ],
    "volumes": [
        {
            "name": "",
            "host": {
                "sourcePath": ""
            }
        }
    ]
}
user@ubuntu:~/trash-can/report$
```

We can run our rep service with a fairly simple task definition. Create the following task definition:

```
user@ubuntu:~/trash-can/report$ vim aws-rep-task.json
user@ubuntu:~/trash-can/report$ cat aws-rep-task.json
{
    "family": "wra-trash-services",
    "containerDefinitions": [
        {
            "name": "trash-level-report-service",
            "image": "433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:repv0.1",
            "cpu": 100,
            "memory": 500,
            "portMappings": [
                {
                    "containerPort": 9090,
                    "hostPort": 80
                }
            ]
        }
    ],
    "taskRoleArn": "arn:aws:iam::433017611331:role/ecs-synamo-sqs-role"
}
```

This definition includes several keys:

- *family*  [Type: string, Required: yes] this is like a name for all of the versions of the task definition, each of
which will have a revision number
- *containerDefinitions*  [Type: array, Required: yes] an array of containers to run
- *taskRoleArn*  [Type: string, Required: no] an IAM role to associate with the task's containers

Our container definition includes a name and container image as well as memory and cpu settings:

- *memory*  [Type: integer, Required: no] hard limit (in MiB) of memory to present to the container (the container is
killed if it attempts to eceed this level)
- *cpu*  [Type: integer, Required: no] The minimum amount of cpu to reserve for the container (1,024 = 1 CPU core)

We also map our container port to a port on the host (if this port is in use your task deployment will fail).

Before we can run our task we must register it. Use the register-task-definition command to prepare ECS to run your
task:

```
user@ubuntu:~/trash-can/report$ aws ecs register-task-definition --cli-input-json file://~/trash-can/report/aws-rep-task.json
{
    "taskDefinition": {
        "status": "ACTIVE",
        "family": "wra-trash-services",
        "requiresAttributes": [
            {
                "name": "com.amazonaws.ecs.capability.ecr-auth"
            },
            {
                "name": "com.amazonaws.ecs.capability.task-iam-role"
            }
        ],
        "volumes": [],
        "taskRoleArn": "arn:aws:iam::433017611331:role/ecs-synamo-sqs-role",
        "taskDefinitionArn": "arn:aws:ecs:us-west-1:433017611331:task-definition/wra-trash-services:1",
        "containerDefinitions": [
            {
                "name": "trash-level-report-service",
                "mountPoints": [],
                "image": "433017611331.dkr.ecr.us-west-1.amazonaws.com/wra/trash-can:repv0.1",
                "cpu": 100,
                "portMappings": [
                    {
                        "protocol": "tcp",
                        "containerPort": 9090,
                        "hostPort": 80
                    }
                ],
                "memory": 500,
                "essential": true,
                "volumesFrom": []
            }
        ],
        "revision": 1
    }
}
user@ubuntu:~/trash-can/report$
```

If the registration succeeds you should see the "status": "ACTIVE" field in the output.

To run the task use the run-task command:

```
user@ubuntu:~/trash-can/report$ aws ecs run-task --cluster default --task-definition wra-trash-services:1
{
    "failures": [],
    "tasks": [
        {
            "taskArn": "arn:aws:ecs:us-west-1:433017611331:task/50605cc5-8225-48b4-814d-01275999ea3f",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "trash-level-report-service"
                    }
                ]
            },
            "lastStatus": "PENDING",
            "containerInstanceArn": "arn:aws:ecs:us-west-1:433017611331:container-instance/8139b024-a72f-4dbf-8ec6-20778803e80f",
            "createdAt": 1479782904.469,
            "clusterArn": "arn:aws:ecs:us-west-1:433017611331:cluster/default",
            "desiredStatus": "RUNNING",
            "taskDefinitionArn": "arn:aws:ecs:us-west-1:433017611331:task-definition/wra-trash-services:1",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-west-1:433017611331:container/dfefbac6-f93b-43f5-8448-442d4f5bfc27",
                    "taskArn": "arn:aws:ecs:us-west-1:433017611331:task/50605cc5-8225-48b4-814d-01275999ea3f",
                    "lastStatus": "PENDING",
                    "name": "trash-level-report-service"
                }
            ]
        }
    ]
}
```

ECS responds with the "lastStatus": "PENDING" output. You can examine the tasks on your cluster with the list-tasks
command:

```
user@ubuntu:~/trash-can/report$ aws ecs list-tasks
{
    "taskArns": [
        "arn:aws:ecs:us-west-1:433017611331:task/50605cc5-8225-48b4-814d-01275999ea3f"
    ]
}
```

Display the details of your task with the describe-tasks command:

```
user@ubuntu:~/trash-can/report$ aws ecs describe-tasks --tasks arn:aws:ecs:us-west-1:433017611331:task/50605cc5-8225-48b4-814d-01275999ea3f
{
    "failures": [],
    "tasks": [
        {
            "taskArn": "arn:aws:ecs:us-west-1:433017611331:task/50605cc5-8225-48b4-814d-01275999ea3f",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "trash-level-report-service"
                    }
                ]
            },
            "lastStatus": "RUNNING",
            "containerInstanceArn": "arn:aws:ecs:us-west-1:433017611331:container-instance/8139b024-a72f-4dbf-8ec6-20778803e80f",
            "createdAt": 1479782904.469,
            "clusterArn": "arn:aws:ecs:us-west-1:433017611331:cluster/default",
            "startedAt": 1479782910.122,
            "desiredStatus": "RUNNING",
            "taskDefinitionArn": "arn:aws:ecs:us-west-1:433017611331:task-definition/wra-trash-services:1",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-west-1:433017611331:container/dfefbac6-f93b-43f5-8448-442d4f5bfc27",
                    "taskArn": "arn:aws:ecs:us-west-1:433017611331:task/50605cc5-8225-48b4-814d-01275999ea3f",
                    "lastStatus": "RUNNING",
                    "name": "trash-level-report-service",
                    "networkBindings": [
                        {
                            "protocol": "tcp",
                            "bindIP": "0.0.0.0",
                            "containerPort": 9090,
                            "hostPort": 80
                        }
                    ]
                }
            ]
        }
    ]
}
```

Your task should now report "lastStatus": "RUNNING". The output also displays the task's containerInstanceArn. You can
use this to display container details:

```
user@ubuntu:~/trash-can/report$ aws ecs describe-container-instances --container-instances arn:aws:ecs:us-west-1:433017611331:container-instance/8139b024-a72f-4dbf-8ec6-20778803e80f
{
    "failures": [],
    "containerInstances": [
        {
            "status": "ACTIVE",
            "registeredResources": [
                {
                    "integerValue": 2048,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "CPU",
                    "doubleValue": 0.0
                },
                {
                    "integerValue": 3954,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "MEMORY",
                    "doubleValue": 0.0
                },
                {
                    "name": "PORTS",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "51678",
                        "51679"
                    ],
                    "type": "STRINGSET",
                    "integerValue": 0
                },
                {
                    "name": "PORTS_UDP",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [],
                    "type": "STRINGSET",
                    "integerValue": 0
                }
            ],
            "ec2InstanceId": "i-0742ee58",
            "agentConnected": true,
            "containerInstanceArn": "arn:aws:ecs:us-west-1:433017611331:container-instance/8139b024-a72f-4dbf-8ec6-20778803e80f",
            "pendingTasksCount": 0,
            "remainingResources": [
                {
                    "integerValue": 1948,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "CPU",
                    "doubleValue": 0.0
                },
                {
                    "integerValue": 3454,
                    "longValue": 0,
                    "type": "INTEGER",
                    "name": "MEMORY",
                    "doubleValue": 0.0
                },
                {
                    "name": "PORTS",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "80",
                        "51678",
                        "51679"
                    ],
                    "type": "STRINGSET",
                    "integerValue": 0
                },
                {
                    "name": "PORTS_UDP",
                    "longValue": 0,
                    "doubleValue": 0.0,
                    "stringSetValue": [],
                    "type": "STRINGSET",
                    "integerValue": 0
                }
            ],
            "runningTasksCount": 1,
            "agentUpdateStatus": "UPDATED",
            "attributes": [
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.syslog"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.json-file"
                },
                {
                    "name": "com.amazonaws.ecs.capability.privileged-container"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.17"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
                },
                {
                    "name": "com.amazonaws.ecs.capability.ecr-auth"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.20"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.21"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.22"
                },
                {
                    "name": "com.amazonaws.ecs.capability.task-iam-role"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.23"
                }
            ],
            "versionInfo": {
                "agentVersion": "1.13.1",
                "agentHash": "efe53c6",
                "dockerVersion": "DockerVersion: 1.11.2"
            }
        }
    ]
}
```

We can use the "ec2InstanceId": "i-0742ee58" field to discover the instance's public IP.

```
user@ubuntu:~/trash-can/report$ aws ec2 describe-instances --instance-ids i-0742ee58 | grep PublicIpAddress
                    "PublicIpAddress": "54.193.16.134",
user@ubuntu:~/trash-can/report$
```

Now test your containerized ECS cloud service:

```
user@ubuntu:~/trash-can/report$ curl -s http://54.193.16.134:80/reports/420
{"can_id":"420", "level":"48.18605700470173"}

user@ubuntu:~/trash-can/report$
```

<br>

Congratulations, you have completed the ECS lab!    

<br>

_Copyright (c) 2013-2016 RX-M LLC, Cloud Native Consulting, all rights reserved_

[RX-M LLC]: http://rx-m.io/rxm-cnc.svg "RX-M LLC"
