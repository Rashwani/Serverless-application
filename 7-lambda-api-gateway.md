# Lab 1 - Invoking AWS Lambda Functions

## Invoke function synchronously

aws lambda invoke --function-name lambda-invoke-demo --payload lambda-invoke-demo response.json

## Invoke function asynchronously

aws lambda invoke --function-name my-node-16-function --invocation-type Event --payload lambda-invoke-demo response.json

## Create versions and an alias and invoke alias ARN

aws lambda invoke --function-name arn:aws:lambda:us-east-1:ACCOUNT_ID:function:my-node-16-function:1 response.json

## Code to return name (make sure the function uses Node.js 16.x)

console.log('Loading function');
exports.handler = (event, context, callback) => {
   console.log('Received event:',
      JSON.stringify(event, null, 2));
   console.log('name =', event.name);
   var name = '';
   if ('name' in event) {
      name = event['name'];
   } else {
      name = 'World';
   }
   var greetings = 'Hello ' + name + '!';
   console.log(greetings);
   callback(null, greetings);
};

## JSON payload for above

{
    "name": "YOUR-NAME"
}

# Lab 2 - Create API Gateway REST API

## Create the AWS Lambda function

1. Create the Lambda function
- Name: helloWorldFunction
- Runtime: Python 3.9

2. Add the code:

```python
import json
import random

def lambda_handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))
    messages = [
        "Hello, World!",
        "Hi there, how are you today?",
        "Greetings from Lambda!",
        "Hello, nice to meet you!",
        "Hi, hope you're having a great day!"
    ]
    return {
        'statusCode': 200,
        'headers': { 'Content-Type': 'application/json' },
        'body': json.dumps({ 'message': random.choice(messages) })
    }
```

3. Build the API
- Type: REST API
- Name: myAPI

4. Configure the resource, method, and integration
- Resource: hello
- Method: GET
- Integration: LAMBDA_PROXY to the 'helloWorldFunction'

5. Deploy API to stage 'prod'

6. Run a test event for the GET method

7. Run a test using the invoke URL, add: '/hello'

8. Delete the API and recreate using the following Swagger code(update the account ID):

```yaml
swagger: "2.0"
info:
  version: "2023-06-24T17:34:51Z"
  title: "myAPI"
host: "k5xnsrq6h8.execute-api.us-east-1.amazonaws.com"
basePath: "/prod"
schemes:
- "https"
paths:
  /hello:
    get:
      produces:
      - "application/json"
      responses:
        "200":
          description: "200 response"
          schema:
            $ref: "#/definitions/Empty"
      x-amazon-apigateway-integration:
        httpMethod: "POST"
        uri: "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:ACCOUNT_ID:function:helloWorldFunction/invocations"
        responses:
          default:
            statusCode: "200"
        passthroughBehavior: "when_no_match"
        contentHandling: "CONVERT_TO_TEXT"
        type: "aws_proxy"
definitions:
  Empty:
    type: "object"
    title: "Empty Schema"
```
9. Run a test event for the GET method. Does it work? If not, fix it

