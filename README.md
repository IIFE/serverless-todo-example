# serverless-todo-example

Simple to-do app using AWS API Gateway and AWS Lambda. This is the sample code
to the [detailed walkthrough](https://medium.com/2pax/fe0e10808732) on error
handling in a serverless architecture.

_**Note:** This project is mainly for demonstration purposes and includes only
the backend of a typical to-do app – limited to add and get single 
tasks. Also, the implementation uses a mock store and doesn't store tasks 
persistently._

_Feel free to play around with the sample code to extend the functionality or 
add an implementation for persistent storage, such as DynamoDB, PostreSQL or 
others._

_See [below](#try-yourself) for how test the API using `curl` or use a HTTP
client of your choice._


## Prerequisites

You'll need the following tools to build, run and deploy the app. In order to
deploy the app you must have access to an [AWS](http://aws.amazon.com) account.

- [AWS Command Line Interface](https://aws.amazon.com/cli)
- [AWS SAM Local](https://github.com/awslabs/aws-sam-local)
- Docker ([Mac](https://docs.docker.com/docker-for-mac), [Windows](https://docs.docker.com/docker-for-windows), see distro package manager on Linx)


## Build

The app has two commands, one to add and another to get a task. Each is a 
separate Lambda function and needs to be built separately.

    ./cmd/add-task/scripts/build.sh

    ./cmd/get-task/scripts/build.sh


## Invoke

Each command comes with example events and uses `example-event.json` by default.
To specify the file use `-e <PATH_TO_EVENT>` when calling the invoke script.

    ./cmd/add-task/scripts/invoke.sh


## Deploy

**Note:** Before deploying make sure to build both commands, see above.

In order to deploy the full stack, package the CloudFormation template using the
following command. Make sure to set `S3_BUCKET` correctly by specifying the name
of the bucket used to store all build artifacts – create a bucket if necessary.

    aws cloudformation package \
        --template-file cloudformation.yaml \
        --s3-bucket <S3_BUCKET> \
        --output-template-file packaged-template.yaml


The packaged template can then be used to bring up the CloudFormation stack.

    aws cloudformation deploy \
      --template-file packaged-template.yaml \
      --stack-name <STACK_NAME> --capabilities CAPABILITY_NAMED_IAM \
      --parameter-overrides StageName=<STAGE_NAME>


## Try yourself…

Replace the `API_GATEWAY`, `AWS_REGION` and `STAGE_NAME` in the cURL commands
with your own values. You can find the API Gateway identifier in the [console](console.aws.amazon.com/apigateway)
within the _Dashboard_ of your API and where it shows the `Invoke URL`.


### `201 Created` for a successful request

```sh
curl -X "POST" "https://<API_GATEWAY>.execute-api.<AWS_REGION>.amazonaws.com/<STAGE_NAME>/tasks" \
    -H 'Content-Type: application/json' \
    -d $'{
  "note": "Take the umbrella ☔"
}'
```

```json
{
  "id": "2b837aa9-9999-40b8-bc0c-42536f7272f7",
  "done": false,
  "note": "Take the umbrella ☔",
  "created_at": "2018-04-20T13:37:34.249306491Z",
  "updated_at": "2018-04-20T13:37:34.249306491Z"
}
```


### `400 Bad Request` for a missing request body

```sh
curl -X "POST" "https://<API_GATEWAY>.execute-api.<AWS_REGION>.amazonaws.com/<STAGE_NAME>/tasks" \
    -H 'Content-Type: application/json'
```

```json
{
  "code": "BAD_REQUEST_BODY",
  "message": "Invalid request body"
}
```


### `422 Unprocessable Entity` for an empty note

```sh
curl -X "POST" "https://<API_GATEWAY>.execute-api.<AWS_REGION>.amazonaws.com/<STAGE_NAME>/tasks" \
    -H 'Content-Type: application/json' \
    -d $'{
  "note": ""
}'
```

```json
{
  "code" : "INVALID_INPUT",
  "message" : "Invalid input"
}
```


### `403 Forbidden` for an unknown resource

```sh
curl -X "POST" "https://<API_GATEWAY>.execute-api.<AWS_REGION>.amazonaws.com/<STAGE_NAME>/wrong-path" \
    -H 'Content-Type: application/json' \
    -d $'{
  "note": "Take the umbrella ☔"
}'
```

```json
{
  "code": "MISSING_AUTHENTICATION_TOKEN",
  "message": "Missing Authentication Token"
}
```
