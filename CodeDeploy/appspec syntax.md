# CodeDeploy AppSpec File reference

https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html

The application specification file (AppSpec file) is a YAML-formatted or JSON-formatted file used by CodeDeploy to manage a deployment.

### AppSpec files on an EC2/on-premises compute platform
If your application uses the EC2/On-Premises compute platform, the AppSpec file must be a YAML-formatted file named **appspec.yml** and it must be placed in the root of the directory structure of an application's source code. Otherwise, deployments fail. It is used by CodeDeploy to determine:

- What it should install onto your instances from your application revision in Amazon S3 or GitHub.

- Which lifecycle event hooks to run in response to deployment lifecycle events.

## AppSpec File example for an Amazon ECS deployment
Here is an example of an AppSpec file written in YAML for deploying an Amazon ECS service.
```
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "arn:aws:ecs:us-east-1:111222333444:task-definition/my-task-definition-family-name:1"
        LoadBalancerInfo:
          ContainerName: "SampleApplicationName"
          ContainerPort: 80
# Optional properties
        PlatformVersion: "LATEST"
        NetworkConfiguration:
          AwsvpcConfiguration:
            Subnets: ["subnet-1234abcd","subnet-5678abcd"]
            SecurityGroups: ["sg-12345678"]
            AssignPublicIp: "ENABLED"
        CapacityProviderStrategy:
          - Base: 1
            CapacityProvider: "FARGATE_SPOT"
            Weight: 2
          - Base: 0
            CapacityProvider: "FARGATE"
            Weight: 1
Hooks:
  - BeforeInstall: "LambdaFunctionToValidateBeforeInstall"
  - AfterInstall: "LambdaFunctionToValidateAfterInstall"
  - AfterAllowTestTraffic: "LambdaFunctionToValidateAfterTestTrafficStarts"
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeAllowingProductionTraffic"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterAllowingProductionTraffic"
  ```
## AppSpec File example for an AWS Lambda deployment
Here is an example of an AppSpec file written in YAML for deploying a Lambda function version.
```
version: 0.0
Resources:
  - myLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "myLambdaFunction"
        Alias: "myLambdaFunctionAlias"
        CurrentVersion: "1"
        TargetVersion: "2"
Hooks:
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeTrafficShift"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterTrafficShift"
  ```
## AppSpec File example for an EC2/On-Premises deployment
Here is an example of an AppSpec file for an in-place deployment to an Amazon Linux, Ubuntu Server, or RHEL instance.

Note
Deployments to Windows Server instances do not support the runas element. If you are deploying to Windows Server instances, do not include it in your AppSpec file.
```
version: 0.0
os: linux
files:
  - source: Config/config.txt
    destination: /webapps/Config
  - source: source
    destination: /webapps/myApp
hooks:
  BeforeInstall:
    - location: Scripts/UnzipResourceBundle.sh
    - location: Scripts/UnzipDataBundle.sh
  AfterInstall:
    - location: Scripts/RunResourceTests.sh
      timeout: 180
  ApplicationStart:
    - location: Scripts/RunFunctionalTests.sh
      timeout: 3600
  ValidateService:
    - location: Scripts/MonitorService.sh
      timeout: 3600
      runas: codedeployuser
```
For a Windows Server instance, change os: linux to os: windows. Also, you must fully qualify the destination paths (for example, c:\temp\webapps\Config and c:\temp\webapps\myApp). Do not include the runas element.

Here is the sequence of events during deployment:

1. Run the script located at Scripts/UnzipResourceBundle.sh.

2. If the previous script returned an exit code of 0 (success), run the script located at Scripts/UnzipDataBundle.sh.

3. Copy the file from the path of Config/config.txt to the path /webapps/Config/config.txt.

4. Recursively copy all the files in the source directory to the /webapps/myApp directory.

5. Run the script located at Scripts/RunResourceTests.sh with a timeout of 180 seconds (3 minutes).

6. Run the script located at Scripts/RunFunctionalTests.sh with a timeout of 3600 seconds (1 hour).

7. Run the script located at Scripts/MonitorService.sh as the user codedeploy with a timeout of 3600 seconds (1 hour).
