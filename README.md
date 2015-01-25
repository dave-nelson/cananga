# cananga #

Cloud platform for scalable computing.


## Purpose ##

Cananga is designed to provide a system for scalable ad-hoc computing.  A
typical use case would be a researcher or analyst who needs to do a large
amount of computation over the space of hours or days, when the same task might
take months or years if performed on his or her desktop.  The idea is to pspin
up a cluster of machines, use them, then dispose of them when finished.

Cananga is oriented towards AWS technologies but is not tightly coupled to
Amazon: it uses systems (such as S3) that are also available on other cloud
platforms.


## Infrastructure ##

Cananga provides a set of build scripts that can be used to create machine
images, which can then be launched into a cluster.  The technologies used are:

* [Mesos](http://mesos.apache.org/): This provides a hardware abstration layer.
  It enables you to pool a set of machines into one large cluster.

* [Marathon](http://mesosphere.github.io/marathon/): Marathon is a Mesos
  framework that facilitates launching and managing long-running tasks.  Each
  application can run as a number of tasks that you specify, including how much
  resource (CPU and RAM) is allocated to each task.  Marathon provides an
  easy-to-use management layer that gets things running and keeps them going.

* [Docker](https://www.docker.com/): Docker images contain your built code, to
  be run on the cluster.  Docker plays a key role in the stack: it enables you
  to run software on the cluster without having to install it on the cluster
  machines in advance.

There are three types of machine involved:

* **client**: This is the machine with which the user interacts: building
  Docker images and running code on the cluster.

* **master**: A small machine that runs the Mesos master, initiates Marathon,
  and which subsequently mediates communication in the cluster.

* **slave**: A pool of powerful slave instances (spot market recommended)
  provide the cluster computing.  They run the user's code as Docker containers
  within Marathon and Mesos.

The build scripts (and instructions) can be found in the `build/` directory.


## Usage ##

Cananga aims to provide a workflow that is friendly to researchers.  There are
three basic operations:

* **build**: Compile the user's code, scripts and perhaps small amounts of file
  data into a Docker image.

* **run-local**: Run Docker containers locally (on the user's machine).  This
  enables the user to ensure that everything is working as expected.

* **deploy**: Launch a cluster application using the built Docker images.

You will find a sample application in this repository that illustrates these
steps.

So to actually use a Cananga cluster a user would perform the following
actions:

1. Start a client instance.

2. Write some code/prepare some data, put dependencies into a Dockerfile.  (The
sample application can be used as a starting point.)

3. Build a Docker image containing the code.

4. Run the container locally, on the client instance.  Ensure that it meets
expectations.

5. Launch a master and a number of slaves -- as many slaves as are needed (and
budget allows).

6. Deploy the application to the cluster.

7. Open the Mesos and Marathon web consoles and monitor progress.

When finished, the slave machines can typically be disposed of, while the
client and master machines may be kept for later usage.


## License ##

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.  See the GNU General Public License for more details.

The GNU General Public License is provided here in the file `LICENSE`.
