# Setup Guide for Workshop Instructors

Follow the guide on this [workshop by AWS](https://catalog.us-east-1.prod.workshops.aws/workshops/0720c7c4-fb23-4e43-aa9f-036fc07f46b2).

If that Workshop doesn't exist, you can use the following instructions (as of 25/03/2025).

## Alternative setup guide

First you need to create the AWS Account using a new email and your credit card (or use an existing AWS Account).

Now you should create an AWS Organization and an admin user for that organization (so you only login to that organization with an user different from root).

Install the base infrastructure using that new user with the following [Cloudformation Stack](https://console.aws.amazon.com/cloudformation/home#/stacks/new?&templateURL=https%3A%2F%2Fws-assets-prod-iad-r-iad-ed304a55c2ca1aee.s3.us-east-1.amazonaws.com%2F0720c7c4-fb23-4e43-aa9f-036fc07f46b2%2Finfra%2Fpackage.yaml) (or use the package.yaml file in this repository).

Choose **Next** in the bottom right of the page. In **Specify stack details** page, enter `bedrock-guardrails-workshop-<student ID>` for **Stack name**. Let everything else as default setting, and then choose **Next**.

Choose **Next** in **Configure stack** options page.

In the last page, select all checkboxes in **Capabilities and transforms** pane, and then choose **Submit**. Wait until stack status is `CREATE_COMPLETE`. It will take around 25-30 minutes for the stack creation to finish.

You need to deploy the infrastructure for each one of the students that will be attending so they can all experiment with their own webapp, the output of this deployment will be a URL for a Cloudfront origin for the student to test the prompts / guardrails.

## Cleanup

Open [Amazon S3 console](https://us-east-1.console.aws.amazon.com/s3/buckets).

Empty the bucket bedrock-guardrails-workshop-static-website-<student-ID> (do this for each one of the student's buckets).

Open AWS CloudFormation console .

Delete stack bedrock-guardrails-workshop-<student-ID> (do this for each one of the student's buckets).

Open Amazon CloudWatch Log groups.

Delete all log group with prefix /aws/lambda/bedrock-guardrails-workshop.
