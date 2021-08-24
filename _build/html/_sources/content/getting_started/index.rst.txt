Using the Demokit
*****************

This section provides information on how to use the demokit.

Getting started with the Demokit
================================

Requirements
------------

For simple applications:

- Having go (Golang) installed: https://golang.org/doc/install
- Docker / Docker compose (for the network, and it eases the development)

For applications involving hardware:

- RaspberryPI with I2C enabled (if you want to use the SenseHat). This can be done through *raspi-config*.


Installation
------------

Bootstrapping the demokit network
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The first thing to do is to set up the development network for the demokit.
You can issue the following command from the demokit-infra folder:

.. code-block:: bash

     $ docker-compose -f docker-compose.dev.yml up -d

Running the hello world example
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We will run the hello-world example with docker. The first thing to do is to build the image.
Going in the demokit-examples/hello-world folder, do:

.. code-block:: bash

     $ docker build -t demokit-hello-world .

You then need a way to see the messages. To do so, we will use the demokit-utils monitor application, which can be built like so:

.. code-block:: bash

     $ go build -o build/demokit-utils main.go

from the demokit-utils folder.

Then, in a terminal, you can start it:

.. code-block:: bash

     $ ./build/monitor-utils monitor

In another terminal, issue the following command to start the hello-world node:

.. code-block:: bash

    docker run -e RABBIT_MQ_USERNAME="guest" -e RABBIT_MQ_PASSWORD="guest" -e RABBIT_MQ_HOST="rabbit_mq" -e RABBIT_MQ_PORT="5672" --network=demokit demokit-hello-world:latest

You should see something similar to this:

.. code-block::

    time="2021-08-24T08:46:37Z" level=info msg="Starting node..." node=twilight-moon
    time="2021-08-24T08:46:37Z" level=info msg="Starting API Server..." node=twilight-moon
    [GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
     - using env:   export GIN_MODE=release
     - using code:  gin.SetMode(gin.ReleaseMode)

    [GIN-debug] Listening and serving HTTP on :8081
    time="2021-08-24T08:46:37Z" level=debug msg="queue amq.gen-LYOOfIt0X5ZPrZ8KT-FG5A declared" component=event-network node=twilight-moon
    time="2021-08-24T08:46:37Z" level=debug msg="Successfully bound to queue amq.gen-LYOOfIt0X5ZPrZ8KT-FG5A" component=event-network node=twilight-moon
    time="2021-08-24T08:46:37Z" level=info msg="Listening for events..." component=event-network node=twilight-moon
    time="2021-08-24T08:46:37Z" level=info msg="Broadcasting hello world..." node=twilight-moon
    time="2021-08-24T08:46:37Z" level=info msg="Node ready!" node=twilight-moon
    time="2021-08-24T08:46:39Z" level=info msg="Broadcasting hello world..." node=twilight-moon
    time="2021-08-24T08:46:41Z" level=info msg="Broadcasting hello world..." node=twilight-moon
    time="2021-08-24T08:46:43Z" level=info msg="Broadcasting hello world..." node=twilight-moon
    time="2021-08-24T08:46:45Z" level=info msg="Broadcasting hello world..." node=twilight-moon
    time="2021-08-24T08:46:47Z" level=info msg="Broadcasting hello world..." node=twilight-moon

And on the monitor:

.. code-block::

    INFO[0000] Listening for events...                       component=event-network name=cli
    2021.08.24 10:46:39 - EVENT NAME: HELLO_WORLD    	Emitter: twilight-moon       	Receiver: *                   	Payload:
    2021.08.24 10:46:41 - EVENT NAME: HELLO_WORLD    	Emitter: twilight-moon       	Receiver: *                   	Payload:
    2021.08.24 10:46:43 - EVENT NAME: HELLO_WORLD    	Emitter: twilight-moon       	Receiver: *                   	Payload:
    2021.08.24 10:46:45 - EVENT NAME: HELLO_WORLD    	Emitter: twilight-moon       	Receiver: *                   	Payload:
    2021.08.24 10:46:47 - EVENT NAME: HELLO_WORLD    	Emitter: twilight-moon       	Receiver: *                   	Payload:


In your applications
^^^^^^^^^^^^^^^^^^^^

The demokit comes as a Go module. To add it as a dependency:

.. code-block:: bash

     $ go get github.com/SINTEF-Infosec/demokit/core

From there, you can compose your node with a core.Node, which gives you access to the demokit functionalities.

Here is a minimal code that runs but does nothing:

.. code-block:: go

    package main

    import (
        "github.com/SINTEF-Infosec/demokit/core"
        "github.com/sirupsen/logrus"
        "time"
    )

    func main() {
        node := NewHelloNode()
        node.Configure()
        node.Start() // Start() needs to be called in order to configure the node internally
    }

    // HelloNode is composed by a core.Node and thus "inherits" its functionalities
    type HelloNode struct {
        *core.Node
    }

    func NewHelloNode() *HelloNode {
        // By default, the logging level is Info
        logrus.SetLevel(logrus.DebugLevel)
        return &HelloNode{
            Node: core.NewDefaultNode(),
        }
    }

    func (n *HelloNode) Configure() {
        // Configure your node here
    }
