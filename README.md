# lambda-test
[![AWS CodePipeline Status](https://s3-eu-west-1.amazonaws.com/lambda-test-123-cfn/badges/current_state.svg)](https://s3-eu-west-1.amazonaws.com/lambda-test-123-cfn/badges/current_state.svg)


This project has build on the **GitOps** principles, so this Git repo act as your **Source of Truth**.....Elaborate on DR, IAC,....

## Technology stack

### AWS SAM

This project uses [AWS Serverless Application Model (AWS SAM)](https://github.com/awslabs/serverless-application-model).

> **See [Serverless Application Model (SAM) HOWTO Guide](https://github.com/awslabs/serverless-application-model/blob/master/HOWTO.md) for more details in how to get started.**

### CodePipeline, CodeBuild & CodeDeploy

A pipeline is included in this stack in order to deploy your code on AWS. The pipeline is consists out of tree major steps:

1. **Source**: listen for Github code changes
1. **Build**: 
	- clean the workspace
	- lint your code and run unit tests
	- package the code for deployment
1. **Deploy**: 
	- Create a CloudFormation changeset
	- Deploy your code through CloudFormation on AWS
	- Use CodeBuild to parse swagger.yaml and deploy the API docs

### Swagger
[Swagger](https://swagger.io/) is used both to configure AWS API Gateway and to document your API endpoints. The API docs will be exposed under `Prod/api-docs`.

## Getting Started

### Prerequisites

* AWS CLI already configured with at least PowerUser permission
* [Python 3 installed](https://www.python.org/downloads/)
* [Pipenv installed](https://github.com/pypa/pipenv)
    - `pip install --user pipenv`
* [Docker installed](https://www.docker.com/community-edition)
* [AWS SAM CLI installed](https://github.com/awslabs/aws-sam-cli) 
	- `pip install --user aws-sam-cli`

### Installation

1. Provided that you have the requirements above installed, proceed by installing the application dependencies and development dependencies:

	```bash
	pipenv install
	pipenv install -d
	```
1. [Add your code to Github](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)
	- In your GitHub create a new repository named `lambda-test`
	- Push your code to Github:

		```bash
		git init
		git add .
		git commit -m "Initial commit"
		git remote add origin git@github.com:glnds/lambda-test.git
		git push -u origin master
		```
1. Setup AWS CodePipeline: in order to install the pipeline a GitHub token is required. To create a token go to: https://github.com/settings/tokens and create a token with `repo` and `admin:repo_hook` permissions.

	For OS X run:

	```bash
	make create-pipeline OAUTH_TOKEN=your_github_token 
	```
	
	For Window run:
	```bash
	aws cloudformation create-stack ^
	--stack-name lambda-test-123-pipeline ^
	--template-body file://pipeline.yaml --parameters ^
	ParameterKey=ApplicationName,ParameterValue=lambda-test ^
	ParameterKey=ArtifactBucket,ParameterValue=lambda-test-123-cfn ^
	ParameterKey=GitHubOAuthToken,ParameterValue={replace_with_oauth_token} ^
	ParameterKey=GitHubUser,ParameterValue=glnds ^
	ParameterKey=GitHubRepository,ParameterValue=lambda-test ^
	ParameterKey=GitHubBranch,ParameterValue=master ^
	--region eu-west-1 --capabilities CAPABILITY_NAMED_IAM --output text
	```
	
You're all set. The newly created AWS CodePipeline will be triggered automatically and in minutes your application will deployed on AWS.

## Running the tests

`Pytest` is used to discover tests created under `tests` folder.
Here's how you can run tests our initial unit tests:

```bash
make test
```

## Local development

In order to run your code locally all dependencies need to be build and packaged on your local machine:
```
make build
```

Afterwards invoking your functions locally through a local API Gateway can be achieved by running:

```bash
make sam
```

If the previous command ran successfully you should now be able to hit the following local endpoint to invoke your functions:
- `http://localhost:3000/rest/one`
- `http://localhost:3000/rest/two`


## Usage


TODO: add all make targets

To install the CodePipeine:

```bash
make create-pipeline OAUTH_TOKEN=your_github_token 
```
To update the CodePipeline:

```bash
make update-pipeline OAUTH_TOKEN=your_github_token
```

To remove the pipeline run:

```bash
make delete-pipeline
```

## Project Layout

Find the most important project files with their description below:
```
.
├── Makefile				# Makefile to make life easier :)
├── buildspec.yml			# Buildspec for building your project with AWS CodeBuild
├── pipeline.yaml			# CloudFormation template to setup AWS CodePipeline 
├── scripts
│   └── swagger_build.sh	# Script triggered by AWS CodeBuild to build the Swagger documentation
├── src
│   └── my_module			# All Lambda source code
│       └── lambda_*.py
├── swagger.yaml			# Swagger file to deploy AWS API Gateway
├── targets.txt				# Config file to run load tests with Vegeta (see appendix)
├── template.yaml			# CloudFormation template for AWS SAM
└── tests					# Lambda Unit Tests
    └── test_handler.py
 ```

## Appendix

### Load Testing
The ```targets.txt``` file included in the project can be used to configure [Vegeta](https://github.com/tsenart/vegeta) to run load tests on the API Gateway endpoints.

**For canary releasing it is crucial to have load on your application.** If there is no load on your application, tests will never fail and your deployments will always pass regardless there's an error or not.

To fire 20 request per second during 30 seconds:
```
vegeta attack -rate=20 -duration=30s -targets=targets.txt | vegeta report
```

To fire 50 requests per second (default) forever:
```
vegeta attack -duration=0 -targets=targets.txt | vegeta report
```

### Makefile

It is important that the Makefile created only works on OSX/Linux but the tasks above can easily be turned into a Powershell or any scripting language you may want too.

Find all available targets: `make help`


