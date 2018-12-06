This project is a sample that attempts to reproduce an issue that was raised for the serverless project.  See the following link:
https://github.com/serverless/examples/issues/279#issuecomment-444672189

The issue seems to be related to a serverless deployment that does not seem to fully deploy successfully due to some error. In the case of this project, which uses AWS, we want to purposely cause the serverless deployment to fail on the first try.  This is achieved by configuring a cloudwatch scheduled event rule for a lambda function that uses a name that is already in use (causing an error). From the serverless.yml: 

```yaml
    loader: 
        timeout: 300
        handler: shared/handler.initialize
        events: 
            - schedule: 
                name: ExistingRule
                rate: cron(0 1 * * ? *)
                enabled: true
```

### Prerequisites

1. Access to AWS (console or programmatic)
2. An existing CloudWatch event rule with the name "ExistingRule"

### Steps to reproduce

1. Execute deployment with stage: ```sls deploy --stage test```  <br>The deployment will fail and the serverless cf stack will eventually get into the update rollback complete state after successfully rolling back the deployment.  The serverless error will look as follows:   

```
Serverless Error ---------------------------------------
 
  An error occurred: LoaderEventsRuleSchedule1 - ExistingRule already exists.
```

2. Correct the event rule name for the loader lambda in the serverless.yml to something unique:

```yaml
            - schedule: 
                name: ExistingRule-new
                rate: cron(0 1 * * ? *)
                enabled: true
```

3. Redeploy:  ```sls deploy --stage test```

4. Once the deployment has successfully completed, run the invokAll script: ```./invokeAll test``` <br>This will use sls invoke on each function to try to reproduce the KMS decryption error which appears as follows:

```
Serverless: Recoverable error occurred (Lambda was unable to decrypt the environment variables due to an internal service error. ), sleeping for 5 seconds. Try 1 of 4
Serverless: Recoverable error occurred (Lambda was unable to decrypt the environment variables due to an internal service error. ), sleeping for 5 seconds. Try 2 of 4
Serverless: Recoverable error occurred (Lambda was unable to decrypt the environment variables due to an internal service error. ), sleeping for 5 seconds. Try 3 of 4
Serverless: Recoverable error occurred (Lambda was unable to decrypt the environment variables due to an internal service error. ), sleeping for 5 seconds. Try 4 of 4
 
  Serverless Error ---------------------------------------
 
  Lambda was unable to decrypt the environment variables due to an internal service error. 
 
  Get Support --------------------------------------------
     Docs:          docs.serverless.com
     Bugs:          github.com/serverless/serverless/issues
     Issues:        forum.serverless.com
 
  Your Environment Information -----------------------------
     OS:                     linux
     Node Version:           8.14.0
     Serverless Version:     1.32.0
```