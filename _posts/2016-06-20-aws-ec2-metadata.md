---
layout: post
title: "AWS EC2 Metadata"
date: 2016-06-20 11:00:00
tags: ['aws', 'ec2', 'devops', 'metadata']
description: "You need to create a script for sending custom metrics do CloudWatch, separated by the instance name. But this script should be generic and applied in various EC2 instances. How can we know the instance ID or its public IP? We may use EC2 Metadata!"
---

You need to create a script for sending custom metrics do CloudWatch, separated by the instance name. But this script should be generic and applied in various EC2 instances. How can we know the instance ID or its public IP? We may use EC2 Metadata!

## Is all about the metadata
Metadata is data that provides information about other data. In a general way, it can be the owner of a file, its size or encode method. 

In EC2, this data is all about the instance itself. You can get its ID, hostname, IAM information, IPs, MACs. It can be very useful to create software or scripts that depends dynamically of this information.

Besides the metadata, EC2 also provides *dynamic-data*, such as instance identity document.

## Accessing it
AWS offers an unique HTTP structure to access the data. This information is available inside the EC2 only, and each instance can access just their own metadata. Also, *any* user who has access to the console can view this information and it isn't protected by any encryption method, so you may not store sensitive information, such as passwords or keys.

The URL used is `http://169.254.169.254/latest/meta-data/`

You can use `curl` to get the index directly from the console:

```bash
$ curl http://169.254.169.254/latest/meta-data/
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
hostname
iam/
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/
```

If the line ends with a bar '/', means that it contains sublinks. If not, the information is right there:

```bash
$ curl http://169.254.169.254/latest/meta-data/ami-id
ami-9abea4fb

$ curl http://169.254.169.254/latest/meta-data/iam/
info
security-credentials/

$ curl http://169.254.169.254/latest/meta-data/iam/info
{
  "Code" : "Success",
  "LastUpdated" : "2016-06-20T11:57:43Z",
  "InstanceProfileArn" : "arn:aws:iam::xxxxxxxxxxxx:instance-profile/xxxxxxxxx",
  "InstanceProfileId" : "xxxxxxxxxxxxxxxxxxxxx"
}
```

## How it can be used?
Easy and awesome, right? Here's a example of how it can be used in a Python script to send custom metrics to CloudWatch. The following snippet is as generic as it can be, showing how we could grab the metadata and use it inside Python's language.

```python
import boto3
from urllib.request import urlopen
from socket import gethostname
from time import time

# Get instance-id
# It comes in UTF-8, so we need to decode it
instance_id = urlopen('http://169.254.169.254/latest/meta-data/instance-id').read().decode('utf-8')

...

# The strctured JSON could be
metric_data = [
    {
        'MetricName': metric_name,
        'Dimensions': [
            {
                'Name': 'InstanceName',
                'Value': gethostname(),
            },
            {
                'Name': 'InstanceID',
                'Value': instance_id,
            },
        ],
        'Timestamp': time(),
        'Value': metric_value,
        'Unit': metric_unit,
    },
]
```

That is it for today :) A simple but powerful feature inside EC2 that makes scripting really easy.
