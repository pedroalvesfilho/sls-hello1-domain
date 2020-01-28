2020-01-28
----------

# sls-hello1-domain
Serverless domain name for Lambda - hello

How to set up a custom domain name for Lambda & API Gateway with Serverless:
https://serverless.com/blog/serverless-api-gateway-domain/

Where: `sls-hello1-domain`

### Create your serverless backend
Before you go any further, you should have a Serverless service with at least one function that has an HTTP event trigger. If you don't have that, you can use the code below. This example is in Python, but any runtime will work.

In a clean directory `sls-hello1-domain`, add a `handler.py` file with the following contents:
  (...)
  
Create two simple functions, `hello` and `goodbye`, to demonstrate how to write 
HTTP handlers in Serverless. 
Now, let's connect them with a Serverless service. 
Create a `serverless.yml` file with the following contents:
  (...)

This `serverless.yml` file configures the functions to respond to HTTP requests. 
It says that the `hello` function will be triggered on the `/hello` path of the API Gateway, 
while the `goodbye` function will be triggered on the `/goodbye` path.

Run `sls deploy` to send the function to production:

$ sls deploy
  (...)


Once the deploy is finished, you will see the Service Information output. 
This includes the API Gateway domain where you can trigger the functions. 
In the example above, the hello function is available at
- https://7bbn6avk11.execute-api.us-east-1.amazonaws.com/dev/hello
I can visit that in the browser.

Finally, the path is odd as well -- `/dev/hello` includes the stage (`dev`) as well as 
the actual page (`hello`). 
I'd rather have a cleaner path. This shows the need for using a custom domain.


### Create a custom domain in API Gateway
Do this with Serverless with the serverless-domain-manager plugin:
- https://github.com/amplify-education/serverless-domain-manager

To use the plugin, first make sure you have a `package.json` file in your service.
Run `npm init -y` to generate one.
```
sls-hello1-domain $ ls
handler.py  README.md  serverless.yml
sls-hello1-domain $ npm init -y
Wrote to .../sls-hello1-domain/package.json:
{
  "name": "sls-hello1-domain",
  "version": "1.0.0",
  "description": "2020-01-28\r ----------",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
sls-hello1-domain $ ls
handler.py  package.json  README.md  serverless.yml
sls-hello1-domain $
```

Then, you'll need to install the plugin in your service:

$ npm install serverless-domain-manager --save-dev

Then, configure it into your serverless.yml:
---
plugins:
  - serverless-domain-manager

custom:
  customDomain:
    domainName: <registered_domain_name>
    basePath: ''
    stage: ${self:provider.stage}
    createRoute53Record: true
---

You can create your custom domain with a single command:

$ sls create_domain
Serverless: Domain was created, may take up to 40 mins to be initialized


Once your domain name is ready, run sls deploy again to redeploy your
service.

Voila! You have a much cleaner URL for your endpoints.

--------------------

domain $ export aws_access_key_id=AKIA5JIMLSYY6PTSYM7A
domain $ export aws_secret_access_key=dcpreEa6PydVAkQdOOGbugcOg70bxJoQkpLPyRg5 domain $ sls deploy
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
........
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service serverless-http.zip file to S3 (1.57 KB)...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
................................................
Serverless: Stack update finished...
Service Information
service: serverless-http
stage: dev
region: us-east-1
stack: serverless-http-dev
resources: 17
api keys:
  None
endpoints:
  GET - https://7ndp2bz8v4.execute-api.us-east-1.amazonaws.com/dev/hello
  GET - https://7ndp2bz8v4.execute-api.us-east-1.amazonaws.com/dev/goodbye
functions:
  hello: serverless-http-dev-hello
  goodbye: serverless-http-dev-goodbye
layers:
  None
Serverless: Run the "serverless" command to setup monitoring, troubleshooting and testing.
domain $
domain $ ls
handler.py  README.md  serverless.yml
domain $
