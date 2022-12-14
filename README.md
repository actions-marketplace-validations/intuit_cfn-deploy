# cfn-deploy
<img src="https://github.com/intuit/cfn-deploy/blob/master/.github/cfn-deploy-logo.png" align=right alt="cfn-deploy" width="250"> 
A simple github action to deploy cloudformation yaml files to AWS

## Usage

An example workflow for deploying a cloudformation template follows.

```
 - uses: intuit/cfn-deploy@master
      env:
        AWS_REGION: us-east-2
        STACK_NAME: cfn-deploy
        TEMPLATE_FILE: ec2.yml
        PARAMETERS_FILE: parameter.json
        CAPABLITIES: CAPABILITY_IAM
        AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}
        AWS_SECRET_ACCESS_KEY: ${{secrets.AWS_SECRET_ACCESS_KEY}}
        SLACK_WEBHOOK_URL: ${{secrets.SLACK_WEBHOOK_URL}}
        GITHUB_JOB_LINK: ${{github.server_url}}/${{github.repository}}/actions/runs/${{github.run_id}}
        NOTIFICATION_ARNS: ("arn:aws:sns:us-east-2:012345678910:topic1" "arn:aws:sns:us-east-2:012345678910:topic2")
```

**Note**: The stack will created if it does not exist. If the initial stack creation fails for some reason then it will be deleted instead of rolled back.

## Secrets

 - `AWS_ACCESS_KEY_ID` – (Required) The AWS access key part of your credentials [more info](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)
 
 - `AWS_SECRET_ACCESS_KEY` – (Required) The AWS secret access key part of your credentials [more info](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)

## Environment variables

All environment variables listed in the official documentation are supported.

The custom env variables to be added are:

`AWS_REGION` - Region to which you need to deploy your app<br>
`STACK_NAME` - Cloudformation Stack Name <br>
`TEMPLATE_FILE` - Cloudformation template yaml file<br>
`PARAMETERS_FILE` - (If required) Input parameters to the cloudformation stack as json file<br>
`CAPABLITIES` - IAM capablities for the cloudformation stack<br>
#### Optional
`WAIT_TIMEOUT` - Timeout in seconds to exit from "wait" of create/update stack<br>
`SLACK_WEBHOOK_URL` - Webhook url for Slack Notification. Refer [Slack Documentation](https://api.slack.com/tutorials/slack-apps-hello-world) <br>
`GITHUB_JOB_LINK` - Link to the Gihub Job<br>
`NOTIFICATION_ARNS` - List of SNS topic ARNS to get stack updates

## Nested Stacks

Nested stacks creation is supported. You create a nested stack within another stack by using the `AWS::CloudFormation::Stack` resource.

Your root stack template should contain resource definitions for your nested stacks with the **S3 URLs** of their templates.

An example of a root stack template:
```
{
    "AWSTemplateFormatVersion" : "2010-09-09",
    "Resources" : {
        "myStack" : {
	       "Type" : "AWS::CloudFormation::Stack",
	       "Properties" : {
	        "TemplateURL" : "https://s3.amazonaws.com/cloudformation-templates-us-east-1/S3_Bucket.template",
              "TimeoutInMinutes" : "60"
	       }
        }
    },
    "Outputs": {
       "StackRef": {"Value": { "Ref" : "myStack"}},
       "OutputFromNestedStack" : {
             "Value" : { "Fn::GetAtt" : [ "myStack", "Outputs.BucketName" ] }
       }
    }
}
```

Refer to AWS docs for additional help: [Working with nested stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)

More example template snippets: [AWS CloudFormation template snippets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-cloudformation.html#w2ab1c27c21c19b5)

## Test Github Actions while development

### Prerequisites

Depend on your machine follow the installation process.

#### [Act](https://github.com/nektos/act) Setup

For macOS
```
brew install act
```

for Windows
```
choco install act-cli
or
scoop install act
```

Other installation options

```
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash
```

#### Docker Setup

`Act` uses docker to run workflows.

For macOS, follow instructions given [here](https://docs.docker.com/desktop/mac/install/)

For Windows, follow instructions given [here](https://docs.docker.com/desktop/windows/install/)

### How to start

[Act](https://github.com/nektos/act) is an open source project which allows you to run project workflow locally. 
This workflow(s) are in the form of YAML file (can be either `.yml` or `.yaml`) residing in `.github/workflows/` folder of your 
project directory.

For more on workflow syntax, follow [here](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)

To test your setup, is working fine or not, run below command. It will list the actions

```shell
act -l
```

It will list all workflows

```shell
ID               Stage  Name
linter           0      linter
TestOnYourLocal  0      TestOnYourLocal
```

To run `Act` specific workflow or yaml/yml file within `.github/workflows/` folder on your branch, you need [github token](https://github.com/settings/tokens/new?scopes=repo&description=wiki%20page%20creator%20token)

```shell
act -W .github/workflows/testlocally.yml
```

Dryrun mode

```shell
act -n -W .github/workflows/testlocally.yml
```

If you want to run for specific branch, you need to mention same in workflow file and also need to pass github token, 
which can be generated [here](https://github.com/settings/tokens/new?scopes=repo&description=wiki%20page%20creator%20token)

```shell
act -n -W .github/workflows/testlocally.yml -s GITHUB_TOKEN=$GITHUB_TOKEN
```


Here is the sample for reference
![Demo](/docs/img/demo.gif)

## Contributing

See [Contributing](https://github.com/intuit/cfn-deploy/blob/master/.github/CONTRIBUTING.md)
