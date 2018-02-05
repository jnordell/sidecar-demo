# Simple sidecar demo
===============================

A simple demo showcasing a sidecar pattern to consume logfiles.

*DISCLAIMER: This is just a proof of concept and not meant to be used as is in production*

-------
## Overview

In OpenShift there exists an option of deploying a [logging aggeration](https://docs.openshift.com/container-platform/latest/install_config/aggregate_logging.html) solution that utilizes the EFK stack. The stack can aggreagte both host logs and applications logs. Making them accessible for developers and cluster administrators on a cluster level.

This is the recommended way to aggregate logs in OpenShift but sometimes there is a need for option that is not included in the default logging flow at all.

Some of the options to realize this could be:

* Configure a logging framework to send logs directly to an external destination.
* Add logging logic to the actual application container.
* Add a sidecar container with the logging logic to the pod.

In this demo the sidecar pattern is used. It will behave the same regardless of what logging framework is available to the application and it does not dissurupt the application container lifecycle.

The sidecar container will run fluentbit and will use the tail plugin to consume a specific logfile. This logfile exists on a emptyDir volume that is shared in the pod betwen the containers.

## Prerequisite

The following instruction asumes an existing OpenShift environment.

This demo was build using the [Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview/). The version of OpenShift included in the CDK used was 3.6.173.0.21.

## Setup

1. Create a project in OpenShift

        $ oc new-project sidecar-demo

2. Process the template 

        $ oc process lalala
