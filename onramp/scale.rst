Scale Cluster
-----------------

Everything up to this point has been done as part of the Quick Start
configuration, with all the components running in a single server (VM
or physical machine). We now describe how to scale Aether to run on
multiple servers, where we assume this cluster-based configuration
throughout the rest of this guide. Before continuing, though, you need
to remove the Quick Start configuration by typing:

.. code-block::

   $ make aether-uninstall

There are two aspects of our deployment that scale independently. One
is Aether proper: a Kubernetes cluster running the set of
microservices that implement SD-Core and AMP (and optionally, other
edge apps). The second is gNBsim: the emulated RAN that generates
traffic directed at the Aether cluster. The assumption in this section
is that there are at least two servers—one for the Aether cluster and
one for gNBsim—with each able to scale independently. For example,
having four servers would support a 3-node Aether cluster and a 1-node
workload generator. This example configuration corresponds to the
following ``hosts.ini`` file:

.. code-block::

   [all]
   node1 ansible_host=172.16.144.50 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
   node2 ansible_host=172.16.144.71 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
   node3 ansible_host=172.16.144.18 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether
   node4 ansible_host=172.16.144.93 ansible_user=aether ansible_password=aether ansible_sudo_pass=aether

   [master_nodes]
   node1

   [worker_nodes]
   node2
   node3

   [gnbsim_nodes]
   node4

The first block identifies all the nodes; the second block designates
which node runs the Kubernetes control plane (and where you invoke
``kubectl`` commands); the third block designates the worker nodes in
the Kubernetes cluster; and the last block indicate which nodes run
the gNBsim workload generator (gNBsim scales across multiple Docker
containers, but these containers are **not** managed by Kubernetes).

Although not a requirement, this and the following sections make the
simplifying assumption that you install OnRamp and invoke Make targets
on the ``master_nodes``. (In general, the Ansible client that OnRamp
uses to deploy Aether need not run on one of the servers listed in
``hosts.ini``.) Also note that having ``master_nodes`` and
``gnbsim_nodes`` contain exactly one common server (as we did
previously) is what triggers Ansible to instantiate the Quick Start
configuration. (In general, the node groups need not be disjoint, so
for example, a single node could be part of ``worker_nodes`` and
``gnbsim_nodes``.)

You need to modify ``hosts.ini`` to match your target deployment.
Once you've done that (and assuming you deleted your earlier Quick
Start configuration), you can re-execute the same set of targets you
ran before:

.. code-block::

   $ make aether-k8s-install
   $ make aether-5gc-install
   $ make aether-amp-install
   $ make aether-gnbsim-install
   $ make aether-gnbsim-run

This will run the same gNBsim test case as before, but originating in
a separate VM. We will return to options for scaling up the gNBsim
workload in a later section, along with describing how to run physical
gNBs in place of gNBsim. Note that if you are primarily interested in
the latter, you can still run Aether on a single server, and then
connect that node to one or more physical gNBs.

Finally, apart from being able able to run SD-Core and gNBsim on
separate nodes—thereby cleanly decoupling the Core from the RAN—one
question we have not yet answered is why you might want to scale the
Aether cluster to multiple nodes. One answer is that you are concerned
about availability, so want to introduce redundancy.

A second answer is that you want to run some other edge application,
such as an IoT or AI/ML platform, on the Aether cluster.  Such
applications can be co-located with SD-Core, with the latter providing
local breakout. For example, OpenVINO is a framework for deploying
inference models to process local video streams streams, for example,
detecting and counting people who enter the field of view for
5G-connected cameras. Just like SD-Core, OpenVINO is deployed as a set
of Kubernetes pods.

.. _reading_openvino:
.. admonition:: Further Reading

   `OpenVINO Toolkit <https://docs.openvino.ai>`__.

A third possible answer is that you want to scale SD-Core itself, in
support of a scalable number of UEs. For example, providing
predictable, low-latency support for hundreds or thousands of IoT
devices requires horizontally scaling the AMF. OnRamp provides a way
to experiment with exactly that possibility by taking advantage of
*Horizontal Pod Autoscaling (HPA)*. Note that scaling Aether is an
area of active research, as documented in the Aether Wiki.

.. _reading_hpa:
.. admonition:: Further Reading

   `Horizontal Pod Autoscaling
   <https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/>`__.

   `Aether Wiki: Research <https://wiki.aetherproject.org/display/HOME/Research>`__.




