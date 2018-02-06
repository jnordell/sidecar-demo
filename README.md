# Simple sidecar demo
===============================

A simple demo showcasing a sidecar pattern to consume log files.

*DISCLAIMER: This is just a proof of concept and not meant to be used as is in production*

-------
## Overview

In OpenShift there exists an option of deploying a [logging aggeration](https://docs.openshift.com/container-platform/latest/install_config/aggregate_logging.html) solution that utilizes the EFK stack. The stack can aggregate both host logs and applications logs. Making them accessible for developers and cluster administrators on a cluster level.

This is the recommended way to aggregate logs in OpenShift but sometimes there is a need for option that is not included in the default logging flow at all.

Some of the options to realize this could be:

* Configure a logging framework to send logs directly to an external destination.
* Add logging logic to the actual application container.
* Add a sidecar container with the logging logic to the pod.

In this demo the sidecar pattern is used. It will behave the same regardless of what logging framework is available to the application and it does not disrupt the application container lifecycle.

The sidecar container will run fluentbit and will use the tail plugin to consume a specific log file. This log file exists on a emptyDir volume that is shared in the pod between the containers.

## Prerequisite

The following instruction assumes an existing OpenShift environment.

This demo was build using the [Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview/). The version of OpenShift included in the CDK used was 3.6.173.0.21.

The template will deploy one pod with the two relevant containers. As endpoint for the log a simple elasticsearch/kibana deployment will be done in the same project. The output plugin can be configured in the configurationMap.

## Setup

1. Create a project in OpenShift

        $ oc new-project sidecar-demo

2. Add the default service account to the SCC anyuid. This is needed as both elasticsearch and kibana needs to set users by them selfs.

        $ oc adm policy add-scc-to-user anuid system:serviceaccount:sidecar-demo:default

3. Create an ElasticSearch instance

        $ oc new-app docker.io/elasticsearch

4. Create an Kibana instance

        $ oc new-app docker.io/kibana
        
5. Expose kibana service

        $ oc expose svc/kibana

Wait for both elasticsearch and kibana to finish deploying before continue with the next step.

6. Process the template 

        $ oc process -f https://raw.githubusercontent.com/jnordell/sidecar-demo/master/openshift/sidecar-demo.yaml | oc create -f-
    
    This will create a deploymentconfiguration describing a pod with two containers. The container named "log-producer" will write a date entry to /mnt/logdir/logfile.txt every five seconds. The second container named "fluentbit-sidecar" will run [fluentbit](http://fluentbit.io) and use the tail plugin to read the contents of the file logfile.txt. Both containers shares the volume called "logdir" of the type emptyDir and while the first container is able to write to the file the second will mount it read-only. 

    The included configuration will send logs to the local elasticsearch installation.

7. When the pod is deployed and running access the kibana route.

        When accessing kibana for the first time it will ask to create an index pattern. Do not change anything and click the "Create" button. This will create an index pattern called logstash-*. Once this has been created select "Discover" in the left menu.

        ![Kibana](https://github.com/jnordell/sidecar-demo/blob/master/docs/images/kibana.png?raw=true)
