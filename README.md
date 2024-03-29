# cookiecutter-aws-sam-python-improvment

improvment of cookiecutter-aws-sam-python

## Requirements

* AWS CLI with Administrator permission
* [Python 3 installed](https://www.python.org/downloads/)
* [Pipenv installed](https://github.com/pypa/pipenv)
    - `pip install pipenv`
* [Docker installed](https://www.docker.com/community-edition)
* [SAM Local installed](https://github.com/awslabs/aws-sam-local) 


Provided that you have requirements above installed, proceed by installing the application dependencies and development dependencies:

```bash
pipenv install
pipenv install -d
```


## Testing

`Pytest` is used to discover tests created under `tests` folder - Here's how you can run tests our initial unit tests:


```bash
AWS_XRAY_CONTEXT_MISSING=LOG_ERROR pipenv run python -m pytest tests/ -v
```

**Tip**: Commands passed to `pipenv run` will be executed in the Virtual environment created for our project.


## Packaging

AWS Lambda Python runtime requires a flat folder with all dependencies including the application. To facilitate this process, the pre-made SAM template expects this structure to be under `first_function/build/`:

```yaml
...
    FirstFunction:
        Type: AWS::Serverless::Function
        Properties:
            CodeUri: first_function/build/
            ...
```

With that in mind, we will:

1. Generate a hashed `requirements.txt` out of our `Pipfile` dep file
1. Install all dependencies directly to `build` sub-folder
2. Copy our function (app.py) into `build` sub-folder


```bash
# Create a hashed pip requirements.txt file only with our app dependencies (no dev deps)
pipenv lock -r > requirements.txt
pip install -r requirements.txt -t first_function/build/
cp -R first_function/app.py first_function/build/
```


### Local development

Given that you followed Packaging instructions then run the following to invoke your function locally:


**Invoking function locally through local API Gateway**

```bash
sam local start-api
```

If the previous command run successfully you should now be able to hit the following local endpoint to invoke your function `http://localhost:3000/first/REPLACE-ME-WITH-ANYTHING`.



## Deployment

First and foremost, we need a S3 bucket where we can upload our Lambda functions packaged as ZIP before we deploy anything - If you don't have a S3 bucket to store code artifacts then this is a good time to create one:

```bash
aws s3 mb s3://BUCKET_NAME
```

Provided you have a S3 bucket created, run the following command to package our Lambda function to S3:

```bash
aws cloudformation package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME
```

Next, the following command will create a Cloudformation Stack and deploy your SAM resources.

```bash
aws cloudformation deploy \
    --template-file packaged.yaml \
    --stack-name cookiecutter-aws-sam-python-improvment \
    --capabilities CAPABILITY_IAM
```

> **See [Serverless Application Model (SAM) HOWTO Guide](https://github.com/awslabs/serverless-application-model/blob/master/HOWTO.md) for more details in how to get started.**


After deployment is complete you can run the following command to retrieve the API Gateway Endpoint URL:

```bash
aws cloudformation describe-stacks \
    --stack-name cookiecutter-aws-sam-python-improvment \
    --query 'Stacks[].Outputs'
``` 


# Appendix



## AWS CLI commands

AWS CLI commands to package, deploy and describe outputs defined within the cloudformation stack:

```bash
aws cloudformation package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME

aws cloudformation deploy \
    --template-file packaged.yaml \
    --stack-name cookiecutter-aws-sam-python-improvment \
    --capabilities CAPABILITY_IAM \
    --parameter-overrides MyParameterSample=MySampleValue

aws cloudformation describe-stacks \
    --stack-name cookiecutter-aws-sam-python-improvment --query 'Stacks[].Outputs'
```

## Running 
