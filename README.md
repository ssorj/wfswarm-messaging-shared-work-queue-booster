# Messaging Shared Work Queue Mission for WildFly Swarm

## Purpose

This mission booster demonstrates how to dispatch tasks to a scalable
set of worker processes using a message queue. It uses the AMQP 1.0
message protocol to send and receive messages.

## Prerequisites

* The user has access to an OpenShift instance and is logged in.

* The user has selected a project in which the frontend and backend
  processes will be deployed.

## Deployment

Run the following commands to configure and deploy the applications.

```bash
find . | grep openshiftio | grep application | xargs -n 1 oc apply -f

oc new-app --template=amq63-basic -p APPLICATION_NAME=shared-work-queue -p IMAGE_STREAM_NAMESPACE=$(oc project -q) -p MQ_PROTOCOL=amqp -p MQ_QUEUES=requests,responses,worker-status -p MQ_USERNAME=shared-work-queue -p MQ_PASSWORD=shared-work-queue

oc new-app --template=wfswarm-messaging-shared-work-queue-frontend -p SOURCE_REPOSITORY_URL=https://github.com/ssorj/wfswarm-messaging-shared-work-queue -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=frontend

oc new-app --template=wfswarm-messaging-shared-work-queue-worker -p SOURCE_REPOSITORY_URL=https://github.com/ssorj/wfswarm-messaging-shared-work-queue -p SOURCE_REPOSITORY_REF=master -p SOURCE_REPOSITORY_DIR=worker
```

## Modules

The `resource-adapter` module serves as the JMS resource adapter for an
external AMQP message server.  It consists of two files.

* [resource-adapter/src/main/rar/META-INF/ra.xml](resource-adapter/src/main/rar/META-INF/ra.xml) -
  This is taken unaltered from the
  [generic JMS RA](https://github.com/jms-ra/generic-jms-ra) RAR
  module.

* [resource-adapter/pom.xml](resource-adapter/pom.xml) - This adds the
  dependencies necessary to use
  [Qpid JMS](http://qpid.apache.org/components/jms/index.html).

The `frontend` module serves the web interface and communicates with
workers in the backend.

* [frontend/src/main/java/io/openshift/booster/messaging/Frontend.java](frontend/src/main/java/io/openshift/booster/messaging/Frontend.java) -
  The main frontend code.  It use HTTP to communicate with browsers
  and AMQP to communicate with the backend.

* [frontend/src/main/java/io/openshift/booster/messaging/ResponseListener.java](frontend/src/main/java/io/openshift/booster/messaging/ResponseListener.java) -
  A message-driven bean for consuming the results of requests.

* [frontend/src/main/java/io/openshift/booster/messaging/WorkerStatusListener.java](frontend/src/main/java/io/openshift/booster/messaging/WorkerStatusListener.java) -
  A message-driven bean for handling worker status updates.

* [frontend/src/main/resources/project-defaults.yml](frontend/src/main/resources/project-defaults.yml) -
  The main Swarm configuration.  This deploys and configures the
  resource adapter.

* [frontend/pom.xml](frontend/pom.xml) - This adds the necessary
  Swarm fractions and the dependency on the Qpid JMS resource adapter.

The `worker` module implements the worker service in the backend.

* [worker/src/main/java/io/openshift/booster/messaging/Worker.java](worker/src/main/java/io/openshift/booster/messaging/Worker.java) -
  The main worker code.  It demonstrates the use of timers for
  periodically sending status updates.

* [worker/src/main/java/io/openshift/booster/messaging/RequestListener.java](worker/src/main/java/io/openshift/booster/messaging/RequestListener.java) -
  A message-driven bean for processing requests and sending responses.

* [worker/src/main/resources/project-defaults.yml](worker/src/main/resources/project-defaults.yml) -
  The main Swarm configuration.  This deploys and configures the
  resource adapter.

* [worker/pom.xml](worker/pom.xml) - This adds the necessary
  Swarm fractions and the dependency on the Qpid JMS resource adapter.
