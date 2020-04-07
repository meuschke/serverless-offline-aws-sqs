# serverless-offline-sqs

This Serverless-offline plugin emulates AWS λ and SQS queue on your local machine by using ElasticMQ. To do so, it listens SQS queue and invokes your handlers.

## Installation

First, add `serverless-offline-aws-sqs` to your project:

```sh
yarn add serverless-offline-aws-sqs
```

Then inside your project's `serverless.yml` file, add following entry to the plugins section before `serverless-offline` (and after `serverless-webpack` if presents): `serverless-offline-aws-sqs`.

```yml
plugins:
  - serverless-webpack
  - serverless-offline-aws-sqs
  - serverless-offline
```

## Configure

### Functions

Ths configuration of function of the plugin follows the [serverless documentation](https://serverless.com/framework/docs/providers/aws/events/sqs/).

```yml
functions:
  mySQSHandler:
    handler: handler.default
    events:
      - sqs:
          queueName: MyQueue
          arn:
            Fn::GetAtt:
              - MyQueue
              - Arn
resources:
  Resources:
    MyQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: MyQueue
```

Inside your functions you could use a wrapper to switch between local and (aws) production environment.  

```js
const AWS = require('aws-sdk');

let options = {};

// connect to local ElasticMQ if running offline
if (process.env.IS_OFFLINE) {
  options = {
    apiVersion: '2012-11-05',
    region: 'localhost',
    endpoint: "http://0.0.0.0:9324",
    sslEnabled: false,
  };
}

const client = new AWS.SQS(options);

export default client
```


### Local SQS (ElasticMQ)

The configuration of [`aws.SQS`'s client](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SQS.html#constructor-property) of the plugin is done by defining a `custom: serverless-offline-sqs` object in your `serverless.yml` with your specific configuration.

You could use [ElasticMQ](https://github.com/adamw/elasticmq) with the following configuration:

```yml
custom:
  serverless-offline-aws-sqs:
    apiVersion: '2012-11-05'
    endpoint: http://0.0.0.0:9324
    region: eu-west-1
    accessKeyId: root
    secretAccessKey: root
    skipCacheInvalidation: false
```

Before you start your serverless functions you ElasticMQ needs to run.

```
docker run -it -p 9324:9324 s12v/elasticmq:latest

```

Queues will be automatically created. The QueueUrl for a SendMessage command sent by a client would be `http://0.0.0.0:9324/queue/MyQueue`.


## Roadmap

- install ElasticMQ automatically or start docker automatically

PLEASE HELP ME TO GET THIS DONE! EVERY PR IS WELCOME.

## Credits
This is a custom fork from [CoorpAcademy](https://github.com/CoorpAcademy/serverless-plugins) Project
