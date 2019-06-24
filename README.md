# AWS-CF-CloudFormationPipeline
Provide generic pipeline to automated Github to CloudFormation deployement

- Main benefit over manualy doing the pipeline is that it can rely on the token of a service account instead of a personal one
- You can chose your CodeBuild Image (https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html#runtime-versions-buildspec-file)
- It does not rely on webhook on purpose, as those require admin privilege on the repository to setup
- If build is set to true, it will create a codebuild project. The codebuild project will rely on the builspec.yaml to know what to do
    - The name of the s3 bucket storing artifacts will be stored in the environnement variable BucketName. This can be used if you want to manually manipulate files
    - Common use case is packaging nested stack
- Template file and configuration file needs to be prefixed by the artifact name followed by ::, there are 2 valid artifact names:
    - source: containing files imported from github
    - build: containing output of the build phase
    - Example: source::Mytemplate.yaml
- Artifact bucket can be accessed by the CodeBuild during the building step, its name is made available in the variable $BucketName
- Container registry, when selected, can be accessed by the CodeBuild during the building step, its name is made available in the variable $ContainerRegistryName
- The AWS account id is exposed in the environment variable AWS_ACCOUNTID. This one is not present in the default variables (https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-env-vars.html)

***Buildspec.yaml example:
````yaml
    version: 0.2
    phases:
    build:
        commands:
        - cd code
        - pip install pytz -t .
        - zip -r -9 ../ec2rds-scheduler.zip *
        - cd ..
    post_build:
        commands:
        - aws cloudformation package --template-file EC2RDS-Scheduler.yaml --s3-bucket $BucketName --output-template-file packaged.yaml
    artifacts:
    files:
        - packaged.yaml