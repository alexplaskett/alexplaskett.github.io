---
layout: post
title:  Fuzzing - Serverless Crash Triage
categories: [Lambda,Serverless,Python]
---

In order to learn about serverless architecture, I experimented with implementing a quick proof of concept crash triaging tool using AWS [Lambda Functions](https://aws.amazon.com/lambda/). There are many benefits of serverless architecture when you really don't want to manage underlying infrastructure components and often cost saving advantages which can be made. These concepts lend themselves well to certain components of a continuous fuzzing architecture (such as Google's [Clusterfuzz](https://google.github.io/clusterfuzz/)).   

When triaging a crash we often want to see at glance the type of crash it is in order to prioritise and determine if the crash is worth further manual investigation. For example, with something like [browser fuzzing](https://github.com/alexplaskett/Publications/blob/master/mwri-t2-big-game-fuzzing-pwn2own-safari-final.pdf), we often end up with a significant amount of crashes where a large majority are not interesting from a security perspective (i.e. not exploitable). We either want to identify interesting crashes or filter out non-interesting ones without having to manually triage each crash. This article focuses on how AWS constructs can be used to trigger triage events. 

This article is just going to demonstrate brief extraction of crash metadata in response to a trigger. In general for crash triage we care about the following information:
* Registers
* Stack Frame
* Faulting Instruction (if available)
* Type of crash (read/write etc)

If we have this information at a glance, we can see if the crash would be good for further manual investigation and root cause analysis. 

[Lambda functions](https://aws.amazon.com/lambda/) allow us to apply event driven processing to a crash at soon as it is available for processing and therefore are perfect for this task. Whilst we could have implemented this logic on the client side, this blog post we describes the creation of a Lambda function and hook it up with our crash data store to perform processing [DynamoDB](https://aws.amazon.com/dynamodb/). 

## Architecture Diagram

We will assume the following simple hypothetical fuzzing architectural design:

![Fuzzing High Level Architecture]({{site.url}}/images/arch.svg)

In this architecture we will assume the following from each component:

* **Fuzz Nodes** - The fuzz nodes are our EC2 instances running a fuzzer, performing crash detection and adding the crash to DynamoDB. 
* **Amazon DynamoDB** - This database will store all crashes found by the fuzzer nodes.
* **Amazon CloudWatch Logs** - This will be used for logging for both the Lambda Function and DynamoDB. 
* **IAM Roles** - An IAM role is an IAM entity which defines the set of permissions for the AWS service which the role is assigned to. 

We will assume once our crash harness detects a crash, then the crash is added to the database. This allows triggering of our post-processing triage lamda. 

The general approach is as follows:
1. A fuzzer generates a crash record and enters it into the Crashes table in DynamoDB.
2. A stream record is written to show that the crash has been entered into Crashes table. 
3. The stream record will fire an AWS Lambda Function. 
4. The Lambda Function will add additional metadata to the crash. 

With this high level architecture in mind, I will start to outline each of the component implementations in detail. 

## DynamoDB Table Creation

One big design choice when building a scalable fuzzing solution is the choice of data storage. When building a cloud based fuzzing platform, the obvious choice is to use one of the cloud native stores (AWS - [DynamoDB](https://aws.amazon.com/dynamodb/), GCP - [Google Datastore](https://cloud.google.com/datastore/docs/concepts/overview) or a cloud based file storage ([S3 bucket](https://aws.amazon.com/s3/) , [Google Cloud Storage](https://cloud.google.com/storage)) etc. The database method seems a better choice and allows queries to be performed across the entire crash corpus. It also allowed for AWS Lambda's to be used to perform post processing of the crash information. 

In order for a Lambda to be executed on a certain DynamoDB event, then it is necessary to enable streams on the table. Streams allow applications to benefit from the ability to capture changes to items stored within a DynamoDB table. A DynamoDB stream is an ordered flow of information changes to an items in a DynamoDB table. 

For this blog post we will assume that new table within DynamoDB will be created from scratch. This can be performed either using the SDK CLI or the GUI interface. 

We will create a DynamoDB table with the following information:
```
Table Name: Crashes 
Primary Key: UUID (String)
```

We use a UUID to represent a unique crash record within the database and therefore will also use this as the primary key. 

This can be done via the AWS CLI as follows:
```bash
$ aws dynamodb create-table \
    --table-name Crashes \
    --attribute-definitions \
        AttributeName=UUID,AttributeType=S \
    --key-schema \
        AttributeName=UUID,KeyType=HASH \
	--provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5 \
    --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES
```

This will create a table and enable streams for the table. For more information about the Dynamo DB streams then [this article](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html) provides a good introduction.

One thing to note here is the usage of NEW_AND_OLD_IMAGES, this specifies:
* NEW_AND_OLD_IMAGES - Both the new and the old item images of the item are written to the stream.

It is also important that we take a note of this stream ARN (as this will be used in the IAM policies):

```
arn:aws:dynamodb:eu-west-2:***********:table/Crashes/stream/2020-02-25T16:22:27.122
```

We can list the streams using the following API:
```bash
$ aws dynamodbstreams list-streams
```

Then describe a stream as follows for more information on it:
```bash
$ aws dynamodbstreams describe-stream \
    --stream-arn arn:aws:dynamodb:eu-west-2:******:table/Crashes/stream/2020-04-07T12:57:19.229
```

## The Triage Lambda Itself

Now the interesting part! We will write code to parse an [Address Sanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) log file and extract relevant parts, then hook up this code to be triggered on DB entry. I will use as an example parsing the log files from ASAN, however, any log file parsing could be used (asan/ksan/gdb/lldb/whatever). The ideal architecture would handle various types of log parsing dependant on the software under test at the time and output the data in a generic form. 

To simulate a fuzzer entering a crash to the Crashes table the following code can be used. Normally the crash harness part of the fuzzer would be performing this process. 

First we convert our log file into the right JSON format to put into the database using the following:

```python
import json
import uuid

fn = "test.log"
contents = None

with open (fn, "r") as myfile:
    contents=myfile.read()

uuid_str = str(uuid.uuid4())

json = """{
	"UUID" : {"S" : "%s" },
	"asan_log" : {"S": %s }
}
""" % (uuid_str,json.dumps(contents))

print(json)
```

and run it as follows:

```bash
$ python log_to_json.py > post.json
```
Once we have our JSON, we can then enter it into DynamoDB using the AWS CLI tools: 

```bash
$ aws dynamodb put-item \
	--table-name Crashes \
	--item file://post.json \
	--return-consumed-capacity TOTAL
```

This will simulate a crash being created within the DB for this item and can be used for testing the upcoming Lambda function code. 

## IAM Role and Policy 

An IAM role is an AWS Identity and Access Management (IAM) entity with permissions to make AWS service requests. IAM roles cannot make direct requests to AWS services; they are meant to be assumed by authorized entities, such as IAM users, applications, or AWS services such as EC2. 

We now need to create a IAM policy and role for our lambda to execute under to allow it both read and write access to the database table. 

We need to make a note of the role ARN so it an be used in the lambda deployment step. 

We now need to create two policies, firstly a trust policy (trust-policy.json):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This json can then be used for creation as follows:

```bash
$ aws iam create-role --role-name lambda-ex --assume-role-policy-document file://trust-policy.json
```
We also want to give the Lambda read/write access to DynamoDB. We also need to give it permissions to log to CloudWatch. We can do this by creating a permission policy (permission-policy.json) (and updating the ARN with the ARN of the table):
```json
{
	"Version": "2012-10-17",
	"Statement": [{
			"Effect": "Allow",
			"Action": [
				"dynamodb:BatchGetItem",
				"dynamodb:GetItem",
				"dynamodb:Query",
				"dynamodb:Scan",
				"dynamodb:BatchWriteItem",
				"dynamodb:PutItem",
				"dynamodb:UpdateItem"
			],
			"Resource": "arn:aws:dynamodb:eu-west-1:123456789012:table/Crashes"
		},
		{
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogStream",
				"logs:PutLogEvents"
			],
			"Resource": "arn:aws:logs:eu-west-1:123456789012:*"
		},
		{
			"Effect": "Allow",
			"Action": "logs:CreateLogGroup",
			"Resource": "*"
		}
	]
}
```

The permissions can then be attached to the role as follows:

```bash
$ aws iam put-role-policy --role-name lambda-ex --policy-name LambdaDynamoDBPolicy --policy-document file://permission-policy.json
```

At this point we have a lambda execution role with the correct polices attached.

We can now create a basic Lambda (lambda_handler.py) in go which will log the event data to CloudWatch.

```python  
def my_handler(event, context):
    print(event)

    return { 
        'message' : event
    }
```

This is useful as it will print to the CloudWatch logs the format of the event from the DynamoDB stream when executed. 

This can then be packaged and deployed as follows:

```bash
$ zip function.zip lambda_handler.py

$ aws lambda create-function --function-name TriageLambda --runtime python3.7 \
  --zip-file fileb://function.zip --handler lambda_handler.my_handler \
  --role arn:aws:iam::123456789012:role/lambda-ex 
```

## Creating a Lambda Trigger

We now need to hook up our Lambda with DynamoDB so that we get can process the stream data when available. 

We need to add 'AWSLambdaDynamoDBExecutionRole' to the role to allow this:

```bash
$ aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaDynamoDBExecutionRole --role-name lambda-ex
```
We also need to give the AWS lambda permissions to write to the cloudwatch logs too:


```bash
$ aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole --role-name lambda-ex
```
This can be performed as follows:
```bash
$ aws dynamodb describe-table --table-name Crashes
```
In this output you should see the 'LatestStreamArn' value. 

We can then create an event source mapping as follows (where streamARN is the ARN returned above):

```bash
$ aws lambda create-event-source-mapping \
    --function-name TriageLambda \
    --event-source streamARN  \
    --batch-size 1 \
    --starting-position TRIM_HORIZON
```

A stream mapping can be viewed (and deleted - if needed!) as follows:
```bash
$ aws lambda list-event-source-mappings
$ aws lambda delete-event-source-mapping --uuid ****** 
```
We can then test our lambda function is working by running the following script to simulate a fuzzer putting an item to the database:

```bash
$ aws dynamodb put-item \
	--table-name Crashes \
	--item file://post.json \
	--return-consumed-capacity TOTAL
```

We can check to see if our function has executed correctly as follows:

```bash
$ aws lambda list-event-source-mappings --function-name TriageLambda
{
    "EventSourceMappings": [
        {
            "UUID": "*****",
            "BatchSize": 1,
            "MaximumBatchingWindowInSeconds": 0,
            "ParallelizationFactor": 1,
            "EventSourceArn": "arn:aws:dynamodb:eu-west-2:***********:table/Crashes/stream/2020-04-08T09:30:19.619",
            "FunctionArn": "arn:aws:lambda:eu-west-2::***********::function:TriageLambda",
            "LastModified": "2020-04-09T10:37:00+01:00",
            "LastProcessingResult": "OK",
            "State": "Enabled",
            "StateTransitionReason": "User action",
            "DestinationConfig": {
                "OnFailure": {}
            },
            "MaximumRecordAgeInSeconds": 604800,
            "BisectBatchOnFunctionError": false,
            "MaximumRetryAttempts": 10000
        }
    ]
}
```

The list returns all of the event source mappings you created, and for each mapping it shows the LastProcessingResult, among other things. This field is used to provide an informative message if there are any problems. Values such as No records processed (indicates that AWS Lambda has not started polling or that there are no records in the stream) and OK (indicates AWS Lambda successfully read records from the stream and invoked your Lambda function) indicate that there are no issues. If there are issues, you receive an error message.

Looking in Cloudwatch logs we can now see the following output:
```json 

{'Records': [{'eventID': '99c8a9f0d9eef8fcf70a0e9faeb62894', 'eventName': 'INSERT', 'eventVersion': '1.1', 'eventSource': 'aws:dynamodb', 'awsRegion': 'eu-west-2', 'dynamodb': {'ApproximateCreationDateTime': 1586339400.0, 'Keys': {'UUID': {'S': '*****-c820-4e81-9909-157e3dea06d4'}}, 'NewImage': {'asan_log': {'S': "=================================================================\n==1382==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffee99d8a40 at pc 0x00010628a26b bp 0x7ffee99d8910 sp 0x7ffee99d80c0\nWRITE of size 1024 at 0x7ffee99d8a40 thread T0\n    #0 0x10628a26a in __asan_memset (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0x5f26a)\n    #1 0x106227e48 in main overflow.c:4\n    #2 0x7fff6ba707fc in start (libdyld.dylib:x86_64+0x1a7fc)\n\nAddress 0x7ffee99d8a40 is located in stack of thread T0 at offset 288 in frame\n    #0 0x106227d0f in main overflow.c:2\n\n  This frame has 1 object(s):\n    [32, 288) 'buf' (line 3) <== Memory access at offset 288 overflows this variable\nHINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork\n      (longjmp and C++ exceptions *are* supported)\nSUMMARY: AddressSanitizer: stack-buffer-overflow (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0x5f26a) in __asan_memset\nShadow bytes around the buggy address:\n  0x1fffdd33b0f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b100: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b110: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b120: 00 00 00 00 f1 f1 f1 f1 00 00 00 00 00 00 00 00\n  0x1fffdd33b130: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n=>0x1fffdd33b140: 00 00 00 00 00 00 00 00[f3]f3 f3 f3 f3 f3 f3 f3\n  0x1fffdd33b150: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b160: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b170: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b180: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\n  0x1fffdd33b190: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00\nShadow byte legend (one shadow byte represents 8 application bytes):\n  Addressable:           00\n  Partially addressable: 01 02 03 04 05 06 07 \n  Heap left redzone:       fa\n  Freed heap region:       fd\n  Stack left redzone:      f1\n  Stack mid redzone:       f2\n  Stack right redzone:     f3\n  Stack after return:      f5\n  Stack use after scope:   f8\n  Global redzone:          f9\n  Global init order:       f6\n  Poisoned by user:        f7\n  Container overflow:      fc\n  Array cookie:            ac\n  Intra object redzone:    bb\n  ASan internal:           fe\n  Left alloca redzone:     ca\n  Right alloca redzone:    cb\n  Shadow gap:              cc\n==1382==ABORTING\n"}, 'UUID': {'S': 'fb82e2bb-c820-4e81-9909-157e3dea06d4'}}, 'SequenceNumber': '400000000007861311075', 'SizeBytes': 2485, 'StreamViewType': 'NEW_AND_OLD_IMAGES'}, 'eventSourceARN': 'arn:aws:dynamodb:eu-west-2:***********:table/Crashes/stream/2020-04-08T09:30:19.619'}]}
```
Now we can write our log extraction and database modification code to perform extraction of the log file. 

## Log Parsing 

As a quick example we will demonstrate parsing a ASAN log file. The log file here is just from a simple stack overflow.

The code will extract the following:
* Stack Trace
* Faulting Instruction
* Type of Crash 

It will also create a hash to allow for crash binning (we could also bucket crashes in the database like this). 

Firstly we need to obtain the log file passed as the 'asan_log' event parameter from the event: 

```python
"""
This lambda function demonstrates log parsing and updating the database according to crash logs.
"""

import json

def my_handler(event, context):
    #print(event)
    #asan_log = event['asan_log']
    #print(asan_log)
    message = "test"
    for record in event['Records']:
        if record['eventName'] == "INSERT":
            message = json.dumps(record['dynamodb']['NewImage']['asan_log']['S'])
            print(message)

    return { 
        'message' : message
    }
```
Now we can implement our log parsing logic to extract the relevant parts of the log. 

```python
import json
import boto3
import json
import re
import hashlib
import binascii

class asanlogparse:
    def __init__(self,log_data):
        print("++ parsing asan output")
        self.log_data = log_data
        self.symbolized_lines = []
        self.error_line = ""
        self.read_or_write = ""

    def print_stack(self):
        for l in self.symbolized_lines:
            print(l)

    def process_log(self):
        read_or_write = False

        for line in self.log_data:
            line = line.strip()
            #print(line)

            stack_trace_line_format = ('^( *#([0-9]+) *)(0x[0-9a-f]+) in (.*)')
            match = re.match(stack_trace_line_format, line)

            if not match:
                if "ERROR: AddressSanitizer" in line:
                    self.error_line = line
                    self.parse_error_line()
                elif "READ" in line or "WRITE" in line:
                    if not read_or_write:
                        self.read_or_write = line
                #print("++ no match")
                continue

            _, frameno_str, addr, symbol = match.groups()
            self.symbolized_lines.append(symbol)
            #print frameno_str, addr, symbol

    """ ==3909== ERROR: AddressSanitizer heap-buffer-overflow on address 0x7fd82049edbd at pc 0x40bcbb bp 0x7fff08117a00 sp 0x7fff081179e8 """
    def extract_error_line(self):
        return self.error_line

    def parse_error_line(self):
        pattern = ".*(0x[0-9a-f]+).*(0x[0-9a-f]+).*(0x[0-9a-f]+).*(0x[0-9a-f]+)"
        match = re.match(pattern,self.error_line)
        if not match:
             address = ""
             pc = ""
             bp = ""
             sp = ""
        else:
            address, pc, bp, sp = match.groups()
        return "Fault Address: " + address + " PC: " + pc + " BP: " + bp + " SP: " + sp

    def type_of_fault(self):
        if "heap-buffer-overflow" in self.error_line:
            return "heap-buffer-overflow"
        elif "heap-use-after-free" in self.error_line:
            return "heap-use-after-free"
        elif "stack-buffer-overflow" in self.error_line:
            return "stack-buffer-overflow"
        else:
            return "UNKNOWN"

    def get_read_or_write(self):
        return self.read_or_write

    def stack_hash(self):
        i = 0
        stack_hash = hashlib.sha1()
        for l in self.symbolized_lines:
            print(l)
            if i > 5:
                break
            else:
                stack_hash.update(l.encode('utf-8'))
                i += 1

        return binascii.hexlify(stack_hash.digest())

    def get_stackframe(self):
        stack_lines = []
        i = 0
        for l in self.symbolized_lines:
            if i > 10:
                break
            else:
                stack_lines.append(l)
                i += 1
        return stack_lines
```

## Database Update Code

Putting this all together, we can add the update DynamoDB code with new fields to provide this metadata to allow for queries. 

```python 

""" 
This lambda function demonstrates log parsing and updating the database according to crash logs.
"""

import json
import boto3
import json
import re
import hashlib
import binascii

class asanlogparse:
    def __init__(self,log_data):
        print("++ parsing asan output")
        self.log_data = log_data
        self.symbolized_lines = []
        self.error_line = ""
        self.read_or_write = ""

    def print_stack(self):
        for l in self.symbolized_lines:
            print(l)

    def process_log(self):
        read_or_write = False

        for line in self.log_data:
            line = line.strip()
            #print(line)

            stack_trace_line_format = ('^( *#([0-9]+) *)(0x[0-9a-f]+) in (.*)')
            match = re.match(stack_trace_line_format, line)

            if not match:
                if "ERROR: AddressSanitizer" in line:
                    self.error_line = line
                    self.parse_error_line()
                elif "READ" in line or "WRITE" in line:
                    if not read_or_write:
                        self.read_or_write = line
                #print("++ no match")
                continue

            _, frameno_str, addr, symbol = match.groups()
            self.symbolized_lines.append(symbol)
            #print frameno_str, addr, symbol

    """ ==3909== ERROR: AddressSanitizer heap-buffer-overflow on address 0x7fd82049edbd at pc 0x40bcbb bp 0x7fff08117a00 sp 0x7fff081179e8 """
    def extract_error_line(self):
        return self.error_line

    def parse_error_line(self):
        pattern = ".*(0x[0-9a-f]+).*(0x[0-9a-f]+).*(0x[0-9a-f]+).*(0x[0-9a-f]+)"
        match = re.match(pattern,self.error_line)
        if not match:
             address = ""
             pc = ""
             bp = ""
             sp = ""
        else:
            address, pc, bp, sp = match.groups()
        return "Fault Address: " + address + " PC: " + pc + " BP: " + bp + " SP: " + sp

    def type_of_fault(self):
        if "heap-buffer-overflow" in self.error_line:
            return "heap-buffer-overflow"
        elif "heap-use-after-free" in self.error_line:
            return "heap-use-after-free"
        elif "stack-buffer-overflow" in self.error_line:
            return "stack-buffer-overflow"
        else:
            return "UNKNOWN"

    def get_read_or_write(self):
        return self.read_or_write

    def stack_hash(self):
        i = 0
        stack_hash = hashlib.sha1()
        for l in self.symbolized_lines:
            print(l)
            if i > 5:
                break
            else:
                stack_hash.update(l.encode('utf-8'))
                i += 1

        return binascii.hexlify(stack_hash.digest())

    def get_stackframe(self):
        stack_lines = []
        i = 0
        for l in self.symbolized_lines:
            if i > 10:
                break
            else:
                stack_lines.append(l)
                i += 1
        return stack_lines

def update_dynamodb(uuid,stack_hash,error_line):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('Crashes')
    table.update_item(
        Key={
            'UUID' : uuid
        },
        UpdateExpression='SET stack_hash = :stack_hash, error_line = :error_line',
        ExpressionAttributeValues={
            ':stack_hash': stack_hash,
            ':error_line' : error_line
        }
    )

def my_handler(event, context):
    print(event)
    #asan_log = event['asan_log']
    #print(asan_log)
    message = "aaa"
    for record in event['Records']:
        # We need to ensure we don't keep updating over an over again (due the stream listener). 
        if record['eventName'] == "INSERT" and "asan_log" in record['dynamodb']['NewImage']:
            uuid = record['dynamodb']['Keys']['UUID']['S']
            message = record['dynamodb']['NewImage']['asan_log']['S']
            print(message)

            # Now parse the log file and extract the relevant parts
            asan = asanlogparse(message.split("\n"))
            asan.process_log()
            stack_hash = str(asan.stack_hash())
            error_line = asan.extract_error_line()

            update_dynamodb(uuid,stack_hash,error_line)

    return { 
        'message' : message
    }
```

The function can then be redeployed and tested. It should be noted in this architecture the Lambda function gets called twice, once on INSERT events and once on MODIFY events. Since we only care about new records being added to the stream, we have to handle this within the code, to prevent constantly calling the lamba again and again. 

## Testing

We can then test the lambda functionality by sending an event to simulate an incoming crash being passed from DynamoDB.

```bash
$ aws dynamodb put-item \
    --table-name Crashes \
    --item file://post.json \
    --return-consumed-capacity TOTAL
```

We can then see by viewing DynamoDB that the document has been updated with our crash metadata! Further post-processing is left up to your imagination.   

## References 

* [AWS Lambda Python](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html)
* [Dynamo DB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.Lambda.Tutorial.html)