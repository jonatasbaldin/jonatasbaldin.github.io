---
layout: post
title: "AWS Serverless Events into Knative (or anywhere!)"
date: 2020-04-29 05:00:00
description: "Today we will take over the means of production by extracting the events from AWS, packing them into a well known format and send them to Knative."
image: "/img/knative-konnek.png"
---

A lot of services inside AWS produces events to be consumed by services inside AWS, that's one of the Serverless foundations. The classic example is executing an AWS Lambda function to resize an image after being uploaded to AWS S3. Receive the event, process the data. Event-driven. Serverless-styles.

## Introducing Konnek
Konnek is this little project I've been working on to extract events from cloud providers – like AWS and GCP –, package them into [CloudEvents](https://cloudevents.io/) and send them anywhere – including Knative.

It works by deploying a central AWS Lambda receiving events _from anywhere inside AWS_ in a generic way, parsing those events according to the CloudEvents spec and send them via an HTTP POST to a webserver. [Here](https://konnek.github.io/docs/#/supported-events/aws) is the list of AWS events supported by Konnek.

## Konnek -> Knative
Consuming events emitted by Konnek into Knative can be quite simple. For the next steps, I assume you already have a Knative platform up and running with Service and Eventing installed and with a Broker named `default` in the default namespace. You can achieve all these steps by following the [Knative Installation docs](https://knative.dev/docs/install/any-kubernetes-cluster/).

Here it's how it looks like:
![](/img/knative_integration.png)

### On Knative
First, let's create a `receiver` Knative Service. It will be responsible to receive the event from Konnek.

Create a file named `receiver.yaml` with the following content and apply it with `kubectl -f receiver.yaml`.
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: konnek-receiver
spec:
  template:
    metadata: 
      annotations:
        # avoiding cold start for fun and profit
        autoscaling.knative.dev/minScale: "1"
    spec:
      containers:
      # You can find the code for the `knative-receiver` service here
      # https://github.com/konnek/knative-receiver
      - image: konnek/knative-receiver
```

Next, we will deploy a `SinkBinding`. What it does is to connect a Kubernetes (or Knative) component that wants to generate events, called the `subject`, to a resource that can consume events, called the `sink`. In our case, the `receiver` Service is forwarding the events to the `default` Broker – a central place in Knative to manage events:

Create a file named `sinkbinding.yaml` with the following content and apply it with `kubectl -f sinkbinding.yaml`.
```yaml
apiVersion: sources.knative.dev/v1alpha1
kind: SinkBinding
metadata:
  name: konnek-sinkbinding
spec:
  subject:
    apiVersion: serving.knative.dev/v1
    kind: Service
    name: konnek-receiver
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1beta1
      kind: Broker
      name: default
```

The third step is to set up a Knative Trigger. It will watch the Broker for an event with a specific type and _trigger_ the code that will finally consume the event. In our example, we will consume an SQS event.

Create a file named `trigger.yaml` with the following content and apply it with `kubectl -f trigger.yaml`.
```yaml
apiVersion: eventing.knative.dev/v1beta1
kind: Trigger
metadata:
  name: konnek-trigger-aws-sqs
spec:
  broker: default
  filter:
    attributes:
      # Here we say we want SQS events
      type: com.amazon.sqs
  subscriber:
    ref:
      # And trigger the consumer function
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: konnek-consumer
```

Finally, let's deploy the Knative `consumer` Service, which will log the event in the logs – but we could do anything with it!

Create a file named `consumer.yaml` with the following content and apply it with `kubectl -f consumer.yaml`.
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: konnek-consumer
  labels:
    serving.knative.dev/visibility: cluster-local
spec:
  template:
    spec:
      containers:
      - image: konnek/consumer
```

Before deploying the Konnek function in AWS, we need the address from the `receiver` Knative Service – since Konnek will forward the AWS events there. Let's fetch it and add into the `KONNEK_CONSUMER` environment variable.
```bash
export KONNEK_CONSUMER=$(kubectl get ksvc konnek-receiver -o jsonpath="{.status.url}")
```

## On AWS
The setup is way simpler in AWS. We will use the Serverless Framework to deploy the function, so make sure it is [installed](https://serverless.com/framework/docs/getting-started/) and [configured](https://serverless.com/framework/docs/providers/aws/cli-reference/config-credentials/).

First, get the latest version of `konnek-aws`. For now, it's `v0.0.3`, but you can check the latest version [here](https://github.com/konnek/konnek-aws/releases):
```bash
wget https://github.com/konnek/konnek-aws/releases/download/v0.0.3/konnek-aws-v0.0.3.zip -O konnek.zip
```

Get the official Konnek serverless.yml file:
```bash
wget https://raw.githubusercontent.com/konnek/konnek-aws/master/config/serverless-framework/serverless.yml
```

Make sure the KONNEK_CONSUMER environment variable is set to the Knative `receiver` Service and deploy the function!
```bash
export KONNEK_CONSUMER=$(kubectl get ksvc konnek-receiver -o jsonpath="{.status.url}")
serverless deploy
```

That should be it! Let's give it a spin!

First, start looking into the Knative `consumer` logs with [stern](https://github.com/wercker/stern), since the event will end up there:
```bash
stern -l serving.knative.dev/service=konnek-consumer -c user-container
```

In another terminal, get a SQS mock data:
```bash
wget https://raw.githubusercontent.com/konnek/konnek-aws/master/testdata/sqs.json
```

And invoke our Konnek function with the Serverless Framework `invoke` command using the mock data:
```bash
serverless invoke -f konnek -p sqs.json
```

Look in the `stern` terminal, you should see something like:
```
... user-container 2020/04/25 18:35:50 Validation: valid
... user-container Context Attributes,
... user-container   specversion: 1.0
... user-container   type: com.amazon.sqs
... user-container   source: arn:aws:sqs:eu-central-1:123456789012:MyQueue
... user-container   id: 1b6a181f-42bb-40e0-a95a-b54baf2795f0
... user-container   time: 2020-04-25T18:35:50.821658085Z
... user-container   datacontenttype: application/json
... user-container Extensions,
... user-container   knativearrivaltime: 2020-04-25T18:35:50.826800825Z
... user-container   knativehistory: default-kne-trigger-kn-channel.default.svc.cluster.local
... user-container   traceparent: 00-1fdfa2325a20d35b622a8ad2262566ff-d8445a2b188ba72f-00
... user-container Data,
... user-container   {
... user-container     "Records": [
... user-container       {
... user-container         "attributes": {
... user-container           "ApproximateFirstReceiveTimestamp": "1523232000001",
... user-container           "ApproximateReceiveCount": "1",
... user-container           "SenderId": "123456789012",
... user-container           "SentTimestamp": "1523232000000"
... user-container         },
... user-container         "awsRegion": "eu-central-1",
... user-container         "body": "Hello from SQS!",
... user-container         "eventSource": "aws:sqs",
... user-container         "eventSourceARN": "arn:aws:sqs:eu-central-1:123456789012:MyQueue",
... user-container         "md5OfBody": "7b270e59b47ff90a553787216d55d91d",
... user-container         "messageAttributes": {},
... user-container         "messageId": "19dd0b57-b21e-4ac1-bd88-01bbb068cb78",
... user-container         "receiptHandle": "MessageReceiptHandle"
... user-container       }
... user-container     ]
... user-container   }
```

WE DID IT! An AWS event directly into your Knative infrastructure :o 

To receive events from a real SQS queue, just add its AWS ARN in the `serverless.yaml` file, it will look like this:
```yaml
...

functions:
  konnek:
    handler: main
    events:
      - sqs: arn:aws:sqs:us-east-1:123456789012:queue1
```

And redeploy your function :)

## What can I do with it?
We've just opened the AWS Serverless world to Knative! Take a look at the source code of the `consumer` Knative Service:
```go
package main

import (
	"context"
	"log"

	cloudevents "github.com/cloudevents/sdk-go"
)

func display(event cloudevents.Event) {
	log.Print(event.String())
}

func main() {
	client, err := cloudevents.NewDefaultClient()
	if err != nil {
		log.Fatalf("could not create client: %v\n", err)
	}

	ctx := context.Background()

	log.Printf("server started on port %d", 8080)

	err = client.StartReceiver(ctx, display)
	if err != nil {
		log.Fatalf("failed to start server, %v", err)
	}
}
```

In this case, the `display` function is printing the function to the stdout, but you can implement whatever you want! Add the event to a database, send a message to Slack, resize images and all the cool Serverless things ⚡️

## Knative (or anywhere!)
Today we saw an example to receive the events in the Knative platform, but Konnek can send them to any webserver that understands CloudEvents 😮 [Try it out receiving in your local machine!](https://konnek.github.io/docs/#/getting-started/setting-up-local-consumer).