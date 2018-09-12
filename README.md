# Amazon Connect & VoiceIt 2.0 Voice Integration Demo
### Andrew Lee
---

We created this repository for our clients who want to integrate our services with Amazon Connect's Call Center functionality.

Both programs are intended to run on AWS Lambda as part of a serverless architecture. They are in a sense, intended to run together, and utilize AWS DynamoDB to "glue" together their logic as well as ensure a secure way to signal a successful authentication from Twilio's server back to Amazon Connect.

The directory structure is as follows:

| *Path* | *Language* | *Description* |
| -- | -- | -- |
| [/connect-twilio-initial](./connect-twilio-initial) | Go | Server which encompasses the initial server side logic when the user calls Amazon Connect |
| [/twilioserver](./twilioserver) | NodeJS | Server that handles call recording using using Twilio's API |

If you need help setting up your workflow, see [exampleconnectvoiceit](./exampleconnectvoiceit) for inspiration.
![example_screenshot](./screenshot.png)

---

## All Use Cases
> In Amazon Connect Call Center Console...


---

## Deploying Example

### Prerequisites
1. Amazon Connect Call Instance with a free phone number
2. AWS DynamoDB table (we named ours `ConnectTwilio` which will be referenced throughout code) with the primary key `phoneNumber` of type String
3. Twilio account with a free phone number
4. Clone this repository somewhere on your local work environment
5. Locally installed [Go](https://golang.org/doc/install) and `$GOPATH` environment variable set up
6. Locally installed [node/npm](https://nodejs.org/en/download/)

### Deploying the Amazon Connect Initial Server

> In Amazon Connect Call Center Console...
1. Create a new contact flow, and use `Import flow` option to upload [our Contact Flow file](./exampleconnectvoiceit)

> In AWS Lambda web UI...

2. Create a new Lambda Function with the `Go 1.x` runtime (Either create a new role, or use an existing role with Full Access to DynamoDB).
3. Add an API Gateway trigger (Create a new API with Security=Open [other options can be left to default values]) & Add

> In `aws cli`...

4. Add permission to allow your Lambda Function to be accessed from Amazon Connect

```shell
aws lambda add-permission --function-name function:[your_lambda_function_name] --statement-id 1 --principal connect.amazona
ws.com --action lambda:InvokeFunction --source-account [source_account_number_of_lambda_function] --source-arn [arn:of_amazon_connect_instance]
```

> In your local machine...

5. Install VoiceIt Go wrapper, AWS-Lambda library, and AWS-SDK

```shell
go get -u github.com/voiceittech/VoiceIt2-Go
go get -u github.com/aws/aws-lambda-go/...
go get -u github.com/aws/aws-sdk-go/...
```

6. `cd` into the `connect-twilio-initial` directory in this repository, build the executable for Lambda, and package it into a ZIP file

```shell
cd [root_of_cloned_repository]/connect-twilio-initial
GOOS=linux go build -o main
zip deployment.zip main
```

> Back in AWS Lambda web view for the function you created...

7. Upload the `deployment.zip` file you just created under the "Function Code" section, and change the `Handler` attribute from 'hello' to 'main'
8. Add the environment variables `VIAPIKEY` and `VIAPITOKEN` (which correspond to the API 2.0 key/token credentials you can view at [https://voiceit.io/settings](https://voiceit.io/settings))
9. Save
10. Take note of the Lambda ARN at the top of the page, which will look like `arn:aws:lambda:[location]:00000000000:function:[functionname]` which we will plug into the Contact Flow

> Back in Amazon Connect Call Center Console Contact Flow view...

11. Change the `Function ARN` parameter in the "Invoke AWS Lambda function" designer element to match the above arn for the Lambda function you created
12. Change the `Phone number` parameter in the "Transfer to phone number" designer element to match the Twilio phone number you created
13. Save & Publish
14. Go to Phone Numbers, and select the Contact Flow you just created for the `Contact flow/IVR` parameter
15. Save

**Congratulations, you now have the initial logic portion of the Amazon Connect Call center up and running! Now it's time to set up the Twilio server which handles the actual voice enrollment/verification to VoiceIt API 2.0.**

---

### Deploying the VoiceIt (Twilio) Server

> In your local machine...

1. change directories into the `twilioserver` directory

```shell
cd [root_of_cloned_repo]/twilioserver
```

2. Install AWS Serverless Express Node Middleware, AWS SDK, body-parser,  moment.js (for converting time into string), the Twilio library, and VoiceIt wrapper

```shell
npm i
```

3. Modify the phone numbers in `app.js` to contact your Amazon Connect phone number. (Any time you see `twiml.dial('786-864-5177');`, just replace the phone number with the phone number you defined in your Amazon Connect Call Center for this example)
4. Zip the deployment as `deployment.zip`

```shell
zip -r deployment.zip *
```

> In AWS Lambda web UI...

5. Create a new Lambda Function  with the `Node.js 8.10` runtime (Either create a new role, or use an existing role with Full Access to DynamoDB), and the name "twilioserver".
**Note: In the name of brevity, we do not include more complex API routing calls and instead focus on using the same exact endpoint of `/twilioserver` throughout our code. If you choose to change the Lambda function name, you must change all endpoint calls to match this as well as modify the next Twilio section accordingly**

6. Add an API Gateway trigger (Create a new API with Security=Open [other options can be left to default values]) & Add
7. Under "Function code", change the `Code entry type` to be "Upload a .ZIP file" and upload `deployment.zip` you created for Node.js
8. Add the environment variables `VIAPIKEY` and `VIAPITOKEN` (which correspond to the API 2.0 key/token credentials you can view at [https://voiceit.io/settings](https://voiceit.io/settings)) as well as the `PHRASE` environment variable as "never forget tomorrow is a new day"
9. Save
10. Take note of the API endpoint which looks like `https://0000000000.execute-api.[location].amazonaws.com/default/twilioserver` as we will need to add it to Twilio's API later

> In Twilio web console...

11. Route the phone number to do a `POST` request to `https://0000000000.execute-api.[location].amazonaws.com/default/twilioserver` as you saw in the API endpoint above
