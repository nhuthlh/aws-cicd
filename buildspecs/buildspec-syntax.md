# Buildspec syntax

https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html

Buildspec files must be expressed in YAML format. If a command contains a character, or a string of characters, that is not supported by YAML, you must enclose the command in quotation marks (""). The following command is enclosed in quotation marks because a colon (:) followed by a space is not allowed in YAML. The quotation mark in the command is escaped (\").

```
"export PACKAGE_NAME=$(cat package.json | grep name | head -1 | awk -F: '{ print $2 }' | sed 's/[\",]//g')"
```

The buildspec has the following syntax:

```
version: 0.2

run-as: Linux-user-name

env:
  shell: shell-tag
  variables:
    key: "value"
    key: "value"
  parameter-store:
    key: "value"
    key: "value"
  exported-variables:
    - variable
    - variable
  secrets-manager:
    key: secret-id:json-key:version-stage:version-id
  git-credential-helper: no | yes

proxy:
  upload-artifacts: no | yes
  logs: no | yes

batch:
  fast-fail: false | true
  # build-list:
  # build-matrix:
  # build-graph:
        
phases:
  install:
    run-as: Linux-user-name
    on-failure: ABORT | CONTINUE
    runtime-versions:
      runtime: version
      runtime: version
    commands:
      - command
      - command
    finally:
      - command
      - command
  pre_build:
    run-as: Linux-user-name
    on-failure: ABORT | CONTINUE
    commands:
      - command
      - command
    finally:
      - command
      - command
  build:
    run-as: Linux-user-name
    on-failure: ABORT | CONTINUE
    commands:
      - command
      - command
    finally:
      - command
      - command
  post_build:
    run-as: Linux-user-name
    on-failure: ABORT | CONTINUE
    commands:
      - command
      - command
    finally:
      - command
      - command
reports:
  report-group-name-or-arn:
    files:
      - location
      - location
    base-directory: location
    discard-paths: no | yes
    file-format: report-format
artifacts:
  files:
    - location
    - location
  name: artifact-name
  discard-paths: no | yes
  base-directory: location
  exclude-paths: excluded paths
  enable-symlinks: no | yes
  s3-prefix: prefix
  secondary-artifacts:
    artifactIdentifier:
      files:
        - location
        - location
      name: secondary-artifact-name
      discard-paths: no | yes
      base-directory: location
    artifactIdentifier:
      files:
        - location
        - location
      discard-paths: no | yes
      base-directory: location
cache:
  paths:
    - path
    - path
 ```
 
### CodeBuild buildspec Elements:
- **version:** (required) Defines the version of your buildspec file. You can choose between version 0.1 or 0.2 (latest).
- **run-as:** (optional) Specifies a Linux user that runs commands in this buildspec file, the scope can be the entire buildspec file if defined at the begging or within phases blocks if defined in phases block.
- **env:** (optional) Specifies the custom environment variables that you want to expose during your build.
  - env/shell: Specifies the supported shell for Linux or Windows operating systems. For Linux, you can use bash or /bin/sh and for Windows, you can use cmd.exe or powershell.exe.
  - env/variables: Specifies the actual environment variable in the form of a key (variable name) value (variable value) pair format in plain text.
  - env/parameter-store: Specifies AWS provided parameter store config which you can use as a source of your environment variable.
  - env/secrets-manager: Specifies AWS provided secret manager config which you can use as a source of your environment variable.
  - env/exported-variables: Specifies environment variable that can be exported to post-build phase in your artifacts.
  - env/git-credential-helper: Specifies if CodeBuild uses its Git credential helper to provide Git credentials. yes if it is used.
- **proxy:** (optional) Used to represent settings if you run your build in an explicit proxy server.
  - proxy/upload-artifacts: Set to yes if you want your build in an explicit proxy server to upload artifacts. The default is no.
  - proxy/logs: Set to yes for your build in an explicit proxy server to create CloudWatch logs. The default is no.
- **phases:** (required) Represents the commands CodeBuild runs during each phase of the build.
  - phases/*/run-as: Specifies a Linux user that runs commands inside the build phase.
  - phases/install: You can define all your package installation commands that are required by your build.
  - phases/install/runtime-versions: Specify the runtime version of your packages that you want to install.
  - phases/install/commands: The commands to execute to install your package.
  - phases/install/finally: The commands to execute at the end of the install phase.
  - phases/pre_build: Represents the commands, if any, that CodeBuild runs before the build.
  - phases/pre_build/commands: The commands to execute during pre-build phase.
  - phases/pre_build/finally: The commands to execute at the end of pre-build phase.
  - phases/build: Represents the commands, if any, that CodeBuild runs during the build.
  - phases/build/commands: The commands to execute during build phase.
  - phases/build/finally: The commands to execute at the end of build phase.
  - phases/post_build: Represents the commands, if any, that CodeBuild runs during the post build.
  - phases/post_build/commands: The commands to execute during post-build phase.
  - phases/post_build/finally: The commands to execute at the end of post-build phase.
- **reports:** (optional) Specifies the CodeBuild test report configuration for your build testing.
  - report-group-name-or-arn: Specifies the report group that the reports are sent to.
  - reports/<report-group>/files: Represents the locations that contain the raw data of test results generated by the report.
  - reports/<report-group>/file-format: Represents the report file format. If not specified, JUNITXML is used.
  - reports/<report-group>/base-directory: Represents one or more top-level directories, relative to the original build location, that CodeBuild uses to determine where to find the raw test files.
  - reports/<report-group>/discard-paths: Specifies if the report file directories are flattened in the output. If this contains yes, all of the test files are placed in the same output directory.
- **artifacts:** (optional) Represents information about where CodeBuild can find the build output and how CodeBuild prepares it for uploading to the S3 output bucket.
  - artifacts/files: Represents the locations that contain the build output artifacts in the build environment.
  - artifacts/name: Specifies a name for your build artifact.
  - artifacts/discard-paths: Specifies if the build artifact directories are flattened in the output. If this contains yes, all of the build artifacts are placed in the same output directory.
  - artifacts/base-directory: Represents one or more top-level directories, relative to the original build location, that CodeBuild uses to determine which files and subdirectories to include in the build output artifact.
  - artifacts/secondary-artifacts: Represents one or more artifact definitions as a mapping between an artifact identifier and an artifact definition.
- **cache:** (optional) Represents information about where CodeBuild can prepare the files for uploading cache to an S3 cache bucket.
  - cache/paths: Represents the locations of the cache.
