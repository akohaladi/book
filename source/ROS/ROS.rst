
ROS2
-----

ROS1 and 2's native habitat is Ubuntu. ROS2 does install and run on
on Windows and Mac, and, there have been efforts to
port ROS1 to Windows or OSX.  The main simulation tool we will
introduce later, Veranda, has not been built for Mac or Windows
as of June, 2018, so in the next chapter we must use Linux but can use
any OS for this chapter.  Using Ubuntu might be the best choice for the long
run, however.

There are several ways to approach getting a Linux install of ROS2.
A standalone Linux system is the easiest. The author has
had good success with a virtual machine (Parallels on OSX). You can also
use for this chapter the OSX or Windows distribution.  Whatever you
select, the next step is to install ROS2.

Installation instructions can be found at
`ROS2 Install <https://github.com/ros2/ros2/wiki/Installation>`_. Please do
this now if not already completed. We will review the instructions here.
The final authority on ROS (installation and other) is `OSRF <ros.org>`_. The
instructions below can and will become out of date. They are included
here so we can discuss the steps.

Installing ROS2 via Debian Packages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Copied from the ROS2 install page`

As of Beta 2 we are building Debian packages for Ubuntu Xenial. They are
in a temporary repository for testing. The following links and
instructions reference the latest release - currently ardent.

Resources: - `Jenkins Instance <http://build.ros2.org/>`__ -
`Repositories <http://repo.ros2.org>`__ - Status Pages
(`amd64 <http://repo.ros2.org/status_page/ros_ardent_default.html>`__,
`arm64 <http://repo.ros2.org/status_page/ros_ardent_uxv8.html>`__)

Setup Sources
~~~~~~~~~~~~~

To install the Debian packages you will need to add our Debian
repository to your apt sources. First you will need to authorize our gpg
key with apt like this:

::

    sudo apt update && sudo apt install curl
    curl http://repo.ros2.org/repos.key | sudo apt-key add -

And then add the repository to your sources list:

::

    sudo sh -c 'echo "deb [arch=amd64,arm64] http://repo.ros2.org/ubuntu/main xenial main" > /etc/apt/sources.list.d/ros2-latest.list'

Install ROS 2 packages
~~~~~~~~~~~~~~~~~~~~~~~

The following commands install all ``ros-ardent-*`` package except ``ros-ardent-ros1-bridge`` and ``ros-ardent-turtlebot2-*`` since they
require ROS 1 dependencies. See below for how to also install those.

::

    sudo apt update
    sudo apt install `apt list "ros-ardent-*" 2> /dev/null | grep "/" | awk -F/ '{print $1}' | grep -v -e ros-ardent-ros1-bridge -e ros-ardent-turtlebot2- | tr "\n" " "`

Environment setup
~~~~~~~~~~~~~~~~~

::

    source /opt/ros/ardent/setup.bash

If you have installed the Python package ``argcomplete`` (version 0.8.5
or higher, see below for Xenial instructions) you can source the
following file to get completion for command line tools like ``ros2``:

::

    source /opt/ros/ardent/share/ros2cli/environment/ros2-argcomplete.bash

(optional) Install argcomplete >= 0.8.5 on Ubuntu 16.04
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you need to install ``argcomplete`` on Ubuntu 16.04 (Xenial), you'll
need to use pip, because the version available through ``apt-get`` will
not work due to a bug in that version of ``argcomplete``:

::

    sudo apt install python3-pip
    sudo pip3 install argcomplete

Choose RMW implementation
^^^^^^^^^^^^^^^^^^^^^^^^^^

By default the RMW implementation ``FastRTPS`` is being used. By setting
the environment variable ``RMW_IMPLEMENTATION=rmw_opensplice_cpp`` you
can switch to use OpenSplice instead.

Additional packages using ROS 1 packages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``ros1_bridge`` as well as the TurtleBot demos are using ROS 1
packages. To be able to install them please start by adding the ROS 1
sources as documented
`here <http://wiki.ros.org/Installation/Ubuntu?distro=kinetic>`__

If you're using Docker for isolation you can start with the image ``ros:kinetic`` or ``osrf/ros:kinetic-desktop`` This will also avoid the
need to setup the ROS sources as they will already be integrated.

Now you can install the remaining packages:

::

    sudo apt update
    sudo apt install ros-ardent-ros1-bridge ros-ardent-turtlebot2-*

Fundamental ROS Entities
~~~~~~~~~~~~~~~~~~~~~~~~~~

Most of the concepts in ROS are based around a set of fundamental
entities, which will be discussed in this section. Understanding ROS,
the challenges which ROS attempts to overcome and the challenges of
using ROS are not possible without a firm understanding of these
materials.

Package
~~~~~~~

Pieces of software are, as the name suggests, known as packages in ROS.
Each package carries out a single or group of tightly related functions.
Packages need not include code at all; some packages are simply
metapackages for the purpose of ensuring that other packages are present
on the system, while other packages include startup routines for robots
or 3-D physical definitions which are used to render robots in
simulation.

Packages may be placed in different ROS environments, and a number of
these environments may be exposed simultaneously, allowing developers to
switch between different groups of packages with ease.

Node
~~~~

The node is quite possibly the single most important concept to
understand when discussing the use of ROS. Nodes are essentially
vertices in the computation graph that is implemented in ROS, and all of
the computation in ROS occurs in a node.

It is considered to be the best form in ROS for a single node to carry
out a single task. This helps to promote code reuse, as the node could
then be used to perform the same task as part of a completely different
system, ideally without any modification of the code.

It is quite common to see hundreds of nodes as part of a single ROS
environment, and it is also common of many of them to be active
simultaneously. For instance:

Master
~~~~~~

The ROS master provides a registration system for the nodes on a ROS
system, among other services. Think of it as the operator of a phone
network. When a node requests information, it asks the ROS master to
connect it to someone who can provide that information. The ROS master
doesn’t actually give the information to the node, it simply tells it
where it may be found. This communication happens over an XMLRPC
protocol.

A node does not typically communicate with the master once it has
finished initializing and is sending and receiving data. It does,
however, talk to the master whenever it needs a new data stream or
parameter information.

Worth noting is that while communication between the nodes and the
master is sparse, loss of communication with the master can be
devastating to a ROS system. If the master were to crash or become
otherwise unavailable, the entire ROS system would likely fail if any
master communication were to be attempted. The node which tried to
communicate with the master would fail, likely causing a domino effect
in nodes trying to request data streams that are sequentially becoming
unavailable.

In general, every ROS system must have exactly one master. There exist
methods of inter-master communication, but there is no built-in
methodology for this.

Message
~~~~~~~

Any data or information that is exchanged between nodes is known as a
message, which is defined as a combination of primitive data types or
other messages. Some messages include a common header, which includes a
sequence number, time stamp and a physical origin known as a frame ID.
For example, a *Twist* message contains 6 Float64 values; a 3-D vector
of linear velocities as well as a 3-D vector of angular velocities. This
message is widely used to describe the velocity of a body in ROS.

Any message defined in ROS is available in any of the supported language
in ROS. Once a node sends a message over ROS, the message can be
interpreted by another node even if the nodes are not written in the
same language or are running on the same operating system. On that note,
messages could be considered to be “data contracts” among nodes.

Topic
~~~~~

While a node may request a certain type of data from the master, it is
possible that multiple nodes could provide data of that type. The use of
“topics” is necessary to uniquely identify a data source to other nodes.
Therefore, when a node notifies the master of available data, it must
provide a topic name for that data. A connection between nodes is only
ever established if the nodes agree on a data type and a topic.

Topics can be thought of as being similar to a telephone number. When a
node registers its “number” with the master (a process known as
advertising), the master notes the message type that the “number”
corresponds to as well as the network address of the node that is
providing it. When another node “calls” that number (a process known as
subscribing), the master looks to see if there is a registered node
providing the requested message type, and tells the node what the
address of the other node is. The direct connection between the nodes is
then established and the data transfer begins.

It should be noted that while this seems to indicate that a topic
corresponds to a single server-client relationship, the topic system
allows for multiple subscribers as well as multiple publishers.
Therefore the relationship is generally referred to as
publisher-subscriber, or “pub-sub.”

Service
~~~~~~~

The publisher-subscriber (pub-sub) model is not always appropriate for
all types of data, and the service system exists in ROS to fill in the
gap. Services are, like pub-sub messages, exposed in ROS over topics.
The group of topic names is separate from the pub-sub topics, but the
structure remains.

Services are unique in that they are based on a call-and-response model
instead of pub-sub. A node can not only request information from another
node, but it can include a message to the other node containing
information about the request. The remote node then responds with a
single message back to the node that initiated the service call.

Services are useful in many ways, but should not be over-used. Each time
a service call is made, the node must request the address of the service
provider from the master. If service calls are made frequently, a
bottleneck could form in the computation graph at the master.
