# serverless-bug-2984

This repository contains serverless configuration files that can be used to reproduce the bug described here:

https://github.com/serverless/serverless/issues/2984

Please note that deploying the below services will bill some (very tiny) expenses to your AWS account, the author takes no responsibility for any costs accrued.

## Steps to reproduce bug

Initialize:

    (cd service1/ && yarn)
    (cd service2/ && yarn)

Deploy `service1`. This service will create an S3 bucket and export its Arn as `ServerlessBug2984BucketArn`:

    (cd service1/ && npx sls deploy)

Deploy `service2`. This service will use the exported bucket arn from `service1` to configure IAM:

    (cd service2/ && npx sls deploy)

Now try to remove `service1`. AWS will not allow you to remove the `service1` stack, since `service2` is dependent upon it:

    (cd service1/ && npx sls remove)

However - **THIS IS THE BUG** - serverless says that everything was removed correctly:

    Serverless: Getting all objects in S3 bucket...
    Serverless: Removing objects in S3 bucket...
    Serverless: Removing Stack...
    Serverless: Checking Stack removal progress...
    ..
    Serverless: Stack removal finished...

You can verify that `service1` is still around:

    aws cloudformation describe-stacks \
        --stack-name serverless-bug-2984-service1-dev

Note that the stack's `StackStatusReason` is set to:

    Export ServerlessBug2984BucketArn cannot be deleted as it is in use by serverless-bug-2984-service2-dev

## Cleanup

To cleanup, just make sure to remove the services in the correct order:

    (cd service2/ && npx sls remove)
    (cd service1/ && npx sls remove)
