# Work Queue sample application #

This sample application presents a fully working example of a clustered work
queue application.  This is a simple processing model that could be used to do
various sorts of distributed computing in a cluster.


## Topology ##

This application consists of an AMQP message broker and one or more workers.


### Message broker component ###

The AMQP message broker maintains a queue (or queues) of messages, each one
detailing some item of work to be performed.


### Worker component ###

The workers consume messages from one of the broker's queues. On success the
worker application returns success which results in a message being consumed;
on failure, the current message is left in the queue for processing by another
node.


## Usage ##

Both the message broker and worker components can be run locally, or on the
cluster.

Note that the commands shown below can generally be run with the command-line
option "-h" to display help.  For example:

    ./build -h
    Build and tag a Docker image.
    Usage: ./build -t tag
        tag : Docker image tag


### Building and running the message broker ###

Use the provided ./build script to build a Docker image for the AMQP message
broker (based on RabbitMQ):

    cd components/broker/
    ./build -t broker:v1

The broker can be run locally (i.e. on the client host -- no cluster required):

    ./run-local -t broker:v1

(Note that the tag should be the name of a previously-built image.)

If desired the broker can be deployed to the compute cluster, to run as a
Marathon task:

    ./deploy -t broker:v1


### Building and running the workers ###

Use the provided ./build script to build a Docker image for the worker (based
on the AMQP tools package):

    cd components/worker/
    ./build -t worker:v1

The worker can be run locally:

    ./run-local -t worker:v1 -q queue

This assumes that a broker is already running on localhost (see above).  You
can point the worker to a different broker using either the -u (specific broker
AMQP URL) or -b (broker name) options.  The -b option allows the discovery of
the host:port of a broker that was previously deployed onto the cluster (see
./deploy above).  The default name for the broker on the Marathon cluster is
"broker", so you could run the worker locally using the cluster broker as
follows:

    ./run-local -t worker:v1 -b broker -q queue

To deploy the worker to the cluster use ./deploy as follows:

    ./deploy -t worker:v1 -b broker -q queue

This will create a Marathon cluster app that will look up the AMQP queue called
"broker" and consume messages from the queue called "queue".  The initial
configuration scales the app to one instance; you would typically use the
Marathon console to scale it up to as many worker instances as you
wanted/needed.


## Altering the implementation ##

You would typically want to make some changes to the implementation of the
worker, since it doesn't do much useful by default.  Tailoring the system to
your needs would require the following:

* Provide a useful message processing program
* Tailor the build of the Docker image to suit -- adding any required packages etc.


### Message processing implementation ###

This system uses the `amqp-consume` utility to drive the processing of
messages.  Basically `amqp-consume` works like this:

1. It launches a program/script whenever there is a message ready to be processed

2. It writes the body of the message to the STDIN of the program/script

3. If the program/script returns success (exit 0) then the message is consumed:
it is removed from the queue.  Or on failure (exit non-0) the message is not
consumed: it is left to be processed by another worker.

So customisation essentially involves providing your own processing program or
script.  How this is done is up to you.  For compiled programs it may be
necessary to build executables from the Dockerfile and install them into the
image.  In simpler cases it may only be a matter of copying scripts into the
image.

The default (not useful) implementation of message processing is in the script
`components/worker/scripts/process-message`.  You can see it writes the body of
the message to a file with a random name.  The usage of this command is
configured near the end of the Marathon app specification
`components/worker/app.json`:

    "cmd": "/usr/local/bin/cluster-worker -b {{BROKER}} -q {{QUEUE}} /usr/local/bin/process-message /tmp"

So a simple way to provide an alternative script would be to put it in the
`components/worker/scripts/` and edit `components/worker/app.json`.  Remove the
part that says "/usr/local/bin/process-message /tmp" and replace it with your
own script/program.

Note that your script will need to fulfil `amqp-consume`'s requirements:

* It will read a single message from STDIN
* It will do something and exit 0 (success) if successful, non-0 otherwise.


### Altering the build ###

The processing script you provide may have certain requirements, such as tools
and libraries.  For example you might want to use a Python script that requires
NumPy and other libraries.  There are some packages that are installed in the
existing Dockerfile in the line like this:

    RUN apt-get -y install amqp-tools curl

It may simply be a matter of adding more packages to that list.

In the case of compiled code you may want to make more changes to the
Dockerfile to build and install your software into the image.


### Checking functionality ###

Note that the default ./run-local command also uses the provided default
implementation of message processing.  See `components/worker/run-local`:

    docker run -i -t -v "/tmp:/tmp" "$tag" bash -c \
        "/usr/local/bin/cluster-worker -u $amqp -q $queue /usr/local/bin/process-message /tmp"

You will want to edit this file too so that you can run the worker in local
mode with your alternative processing script.  Remove the part that says
"/usr/local/bin/process-message /tmp" and replace it with your own
script/program that you set up.

