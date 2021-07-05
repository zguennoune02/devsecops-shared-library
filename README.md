# DevSecOps shared library

This shared library aims to simplify the usage of security tools within the GitLab-CI pipeline.


## About DevSecOps shared library

Implementing and setting up security tools within a CI/CD pipeline can be difficult and time-consuming for developpers.
From this observation, I and Théo Cusnir created a shared library that permits to easily integrate high-quality open-source security tools, within the GitLab-CI pipeline.

This shared library is composed of templates, that can be imported by every project to their own ```.gitlab-ci.yml``` file.
As each tool has its own template, you can import only the tools you need for your project.

### Security tools available

| Security tools                     | Description |
| ------------------------------------- | ------------------------- |
| [OWASP ZAP](https://github.com/zaproxy/zaproxy/) | Analyze running web applications for known vulnerabilities (detection of basic XSS, basic SQL injection, basic sensible file disclosure...)
| [Anchore](https://github.com/anchore/anchore-engine) | Analyze your Docker images for known vulnerabilities.|
| [Dependency Check](https://github.com/jeremylong/DependencyCheck)                 | Analyze your dependencies for known vulnerabilities.
| [Docker Bench](https://github.com/docker/docker-bench-security)                   | Analyze your Docker containers for known vulnerabilities. |
| [FFUF](https://github.com/ffuf/ffuf)                     | Fuzz code for vulnerabilities (find publicly exposed files that might be sensitive) |
| [Nmap](https://nmap.org/)           | Scan networks to map open ports and running services|
| [Sqlmap](https://github.com/sqlmapproject/sqlmap)                       | Perform advanced SQL injection test on running web applications. |
| [SSLyze](https://github.com/nabla-c0d3/sslyze)                       | Analyze SSL/TLS servers' configuration.|
| [Arachni](https://www.arachni-scanner.com)                       | Perform advanced XSS attack tests (active checks only) on running web applications. |


## Requirements

To integrate the security tools, you need to setup your machine and prepare your ```.gitlab-ci.yml``` file :

1. Install [GitLab Runner](https://docs.gitlab.com/runner/install/) with a [Shell Executor](https://docs.gitlab.com/runner/executors/shell.html) configured and set a tag name.

2. Install [Docker](https://docs.docker.com/engine/install/) on the same machine that the Runner is installed on as the environments needed to run the stages are dockerized.
> **Note:** All required dependencies for your builds need to be installed manually on the same machine that the Runner is installed on.

3. Add the following in the ```before_script``` tag to your ```.gitlab-ci.yml``` file:
```yml
before_script:
  - mkdir reports
```

## OWASP ZAP

### Overview
The [OWASP Zed Attack Proxy](https://github.com/zaproxy/zaproxy/) (ZAP) is one of the world’s most popular free security tools and is actively maintained by a dedicated international team of volunteers. It can help you automatically find security vulnerabilities in your web applications while you are developing and testing your applications.

> **Note:** You need a test user on your application

> **IMPORTANT:** You should NEVER run an authenticated scan against a production server. When an authenticated scan is run, it may perform any function that the authenticated user can (ie: modifying/deleting data, submitting forms, following links...). Only run an authenticated scan against a test server.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```WEBSITE```         | Main page URL of your website (e.g., https://mywebsite.com) |
| ```LOGIN_URL```       | Login page URL of your website (e.g., https://mywebsite.com/login) |
| ```USERNAME```        | Username to authenticate on the website to test. **YOU SHOULD SET THIS AS ENVIRONMENT VARIABLE IN YOUR GITLAB PROJECT** |
| ```PASSWORD```        | Password to authenticate on the website to test. **YOU SHOULD SET THIS AS ENVIRONMENT VARIABLE IN YOUR GITLAB PROJECT** |

To integrate OWASP ZAP tool, you will need to import and call the OWASP ZAP template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dast-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use OWASP ZAP in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### OWASP ZAP VARIABLES
  WEBSITE: "https://mywebsite.com"
  LOGIN_URL: "https://mywebsite.com/login"
  USERNAME: $USERNAME # With USERNAME as environment variable in your GitLab Project
  PASSWORD: $PASSWORD # With PASSWORD as environment variable in your GitLab Project

# Other variables below...
```

3. Add a ```dast``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - dast
  # Other stages below...
```

4. Call the ```dast``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Docker** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# DAST CALL
dast:
  stage: dast
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dast-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### OWASP ZAP VARIABLES
  WEBSITE: "https://mywebsite.com"
  LOGIN_URL: "https://mywebsite.com/login"
  USERNAME: $USERNAME # With USERNAME as environment variable in your GitLab Project
  PASSWORD: $PASSWORD # With PASSWORD as environment variable in your GitLab Project

# Other variables below...

stages:
  # Other stages above...
  - dast
  # Other stages below...

# Other stage calls above...

# DAST CALL
dast:
  stage: dast
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## Anchore

### Overview

[Anchore Engine](https://github.com/anchore/anchore-engine) is an open source tool for deep image inspection and vulnerability scanning. It allows users to perform detailed analysis of container images, producing reports and defining policies that can be used in CI/CD pipelines.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```ANALYZED_DOCKER_IMAGE``` | Name and version of the image to test (e.g., mydockerimage:myversion) |


To integrate Anchore tool, you will need to import and call the Anchore template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/anchore-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use Anchore in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### ANCHORE VARIABLES
  ANALYZED_DOCKER_IMAGE: "mydockerimage:myversion"

# Other variables below...
```

3. Add a ```anchore``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - anchore
  # Other stages below...
```

4. Call the ```anchore``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# ANCHORE CALL
anchore:
  stage: anchore
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/anchore-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### ANCHORE VARIABLES
  ANALYZED_DOCKER_IMAGE: "mydockerimage:myversion"

# Other variables below...

stages:
  # Other stages above...
  - anchore
  # Other stages below...


# Other stage calls above...

# ANCHORE CALL
anchore:
  stage: anchore
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## Dependency Check

### Overview
[Dependency Check](https://github.com/jeremylong/DependencyCheck) is a Software Composition Analysis (SCA) tool that attempts to detect publicly disclosed vulnerabilities contained within a project’s dependencies. It does this by determining if there is a Common shared-assets Enumeration (CPE) identifier for a given dependency. If found, it will generate a report linking to the associated CVE entries.

Dependency Check is a tool evaluating the security level of the dependencies of source code files.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```PROJECT_SOURCES_FOLDER``` | Name of the folder containing the source code to evaluate (e.g., src_folder) |

To integrate Dependency Check tool, you will need to import and call the Dependency Check template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dcheck-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use Dependency Check in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### DEPENDENCY CHECK VARIABLES
  PROJECT_SOURCES_FOLDER: "src_folder"

# Other variables below...
```

3. Add a ```dcheck``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - dcheck
  # Other stages below...
```

4. Call the ```dcheck``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Docker** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# DEPENDENCY CHECK CALL
dcheck:
  stage: dcheck
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dcheck-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### DEPENDENCY CHECK VARIABLES
  PROJECT_SOURCES_FOLDER: "src_folder"

# Other variables below...

stages:
  # Other stages above...
  - dcheck
  # Other stages below...

# Other stage calls above...

# DEPENDENCY CHECK CALL
dcheck:
  stage: dcheck
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## Docker Bench

### Overview

The [Docker Bench](https://github.com/docker/docker-bench-security) for Security is a script that checks for dozens of common best-practices around deploying Docker containers in production. The tests are all automated, and are inspired by the [CIS Docker Benchmark v1.2.0](https://www.cisecurity.org/benchmark/docker/).

Docker Bench is a tool evaluating the security level of the dependencies of Docker containers.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```CUSTOM_DOCKER_NETWORK``` | Name of the Docker network within the Docker container to test  (e.g., custom-host) |

To integrate Docker Bench tool, you will need to import and call the Docker Bench template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dbench-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use Docker Bench in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### DOCKER BENCH VARIABLES
  CUSTOM_DOCKER_NETWORK: "custom-host"

# Other variables below...
```

3. Add a ```dbench``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - dbench
  # Other stages below...
```

4. Call the ```dbench``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# DOCKER BENCH CALL
dbench:
  stage: dbench
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dbench-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### DOCKER BENCH VARIABLES
  CUSTOM_DOCKER_NETWORK: "custom-host"

# Other variables below...

stages:
  # Other stages above...
  - dbench
  # Other stages below...

# Other stage calls above...

# DOCKER BENCH CALL
dbench:
  stage: dbench
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## FFUF

### Overview

[FFUF](https://github.com/ffuf/ffuf) is a fast web fuzzer written in Go.

FFUF is a tool detecting any sensitive file that might be publicly accessible on a website.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```WEBSITE``` | Main page URL of your website (e.g., https://mywebsite.com) |

To integrate FFUF tool, you will need to import and call the FFUF template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/ffuf-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use FFUF in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### FFUF VARIABLES
  WEBSITE: "https://mywebsite.com"

# Other variables below...
```

3. Add a ```ffuf``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - ffuf
  # Other stages below...
```

4. Call the ```ffuf``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# FFUF CALL
ffuf:
  stage: ffuf
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/ffuf-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### FFUF VARIABLES
  WEBSITE: "https://mywebsite.com"

# Other variables below...

stages:
  # Other stages above...
  - ffuf
  # Other stages below...

# Other stage calls above...

# FFUF CALL
ffuf:
  stage: ffuf
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## Nmap

### Overview

[Nmap](https://nmap.org/) is a free and open source (license) utility for network discovery and security auditing. Many systems and network administrators also find it useful for tasks such as network inventory, managing service upgrade schedules, and monitoring host or service uptime. Nmap uses raw IP packets in novel ways to determine what hosts are available on the network, what services (application name and version) those hosts are offering, what operating systems (and OS versions) they are running, what type of packet filters/firewalls are in use, and dozens of other characteristics. It was designed to rapidly scan large networks, but works fine against single hosts.

Nmap is a tool scanning a domain and permitting to map any open-ports, with their version name and number.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```DOMAIN``` | Main domain of your application (e.g., mywebsite.com) |

To integrate Nmap tool, you will need to import and call the Nmap template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/nmap-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use Nmap in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### NMAP VARIABLES
  DOMAIN: "mywebsite.com"

# Other variables below...
```

3. Add a ```nmap``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - nmap
  # Other stages below...
```

4. Call the ```nmap``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# NMAP CALL
nmap:
  stage: nmap
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/nmap-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### NMAP VARIABLES
  DOMAIN: "mywebsite.com"

# Other variables below...

stages:
  # Other stages above...
  - nmap
  # Other stages below...

# Other stage calls above...

# NMAP CALL
nmap:
  stage: nmap
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```


## Sqlmap

### Overview

[Sqlmap](https://github.com/sqlmapproject/sqlmap) is an open source penetration testing tool that automates the process of detecting and exploiting SQL injection flaws and taking over of database servers.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```TARGET``` | Page URL of your website to test containing parameters (e.g., https://mywebsite.com/page=1) |

To integrate sqlmap tool, you will need to import and call the sqlmap template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sqlmap-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use sqlmap in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### SQLMAP VARIABLES
  TARGET: "https://mywebsite.com/page=1"

# Other variables below...
```

3. Add a ```sqlmap``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - sqlmap
  # Other stages below...
```

4. Call the ```sqlmap``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# SQLMAP CALL
sqlmap:
  stage: sqlmap
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sqlmap-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### SQLMAP VARIABLES
  TARGET: "https://mywebsite.com/page=1"

# Other variables below...

stages:
  # Other stages above...
  - sqlmap
  # Other stages below...

# Other stage calls above...

# SQLMAP CALL
sqlmap:
  stage: sqlmap
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## SSLyze

### Overview

[SSLyze](https://github.com/nabla-c0d3/sslyze) is a fast and powerful SSL/TLS scanning library. It allows you to analyze the SSL/TLS configuration of a server by connecting to it, in order to detect various issues (bad certificate, weak cipher suites, Heartbleed, ROBOT, TLS 1.3 support, etc.).

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```HOST``` | [IP address or domain]:[port] (e.g., mywebsite.com, mywebsite.com:443) |

To integrate SSLyze tool, you will need to import and call the SSLyze template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sslyze-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use SSLyze in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### SSLYZE VARIABLES
  HOST: "mywebsite.com"

# Other variables below...
```

3. Add a ```sslyze``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - sslyze
  # Other stages below...
```

4. Call the ```sslyze``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# SSLYZE CALL
sslyze:
  stage: sslyze
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sslyze-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### SSLYZE VARIABLES
  HOST: "mywebsite.com"

# Other variables below...

stages:
  # Other stages above...
  - sslyze
  # Other stages below...

# Other stage calls above...

# SSLYZE CALL
sslyze:
  stage: sslyze
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## Arachni

### Overview

[Arachni](https://www.arachni-scanner.com) is a feature-full, modular, high-performance Ruby framework aimed towards helping penetration testers and administrators evaluate the security of modern web applications. It is versatile enough to cover a great deal of use cases, ranging from a simple command line scanner utility, to a global high performance grid of scanners, to a Ruby library allowing for scripted audits, to a multi-user multi-scan web collaboration platform.

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```WEBSITE``` | Main page URL of your website (e.g., https://mywebsite.com) |

To integrate Arachni, you will need to import and call the Arachni template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/arachni-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use Arachni in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### ARACHNI VARIABLES
  WEBSITE: "https://mywebsite.com"

# Other variables below...
```

3. Add a ```arachni``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - arachni
  # Other stages below...
```

4. Call the ```arachni``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# ARACHNI CALL
arachni:
  stage: arachni
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/arachni-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### ARACHNI VARIABLES
  WEBSITE: "https://mywebsite.com"

# Other variables below...

stages:
  # Other stages above...
  - arachni
  # Other stages below...

# Other stage calls above...

# ARACHNI CALL
arachni:
  stage: arachni
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## SonarQube

### Overview

[SonarQube](https://www.sonarqube.org) (formerly Sonar) is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities on 20+ programming languages. SonarQube offers reports on duplicated code, coding standards, unit tests, code coverage, code complexity, comments, bugs, and security vulnerabilities. It can record metrics history and provides evolution graphs.

### Requirements

You need to install and set up [SonarQube server](https://docs.sonarqube.org/latest/setup/install-server/).

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```SONAR_TOKEN``` | Authentication token of a SonarQube user with Execute Analysis permission on the project. **YOU SHOULD SET THIS AS ENVIRONMENT VARIABLE IN YOUR GITLAB PROJECT** |
| ```SONAR_HOST_URL``` | Server URL (e.g., http://your-sonarqube-instance.org)|
| ```SONAR_PROJECT_KEY``` | Unique identifier for the project. Allowed characters are: letters, numbers, -, _, . and :, with at least one non-digit. |
| ```SONAR_PROJECT_NAME``` | Name of the project that will be displayed. Default : $CI_PROJECT_NAME |
| ```SONAR_SOURCES``` | Comma-separated relative paths from $CI_PROJECT_DIR containing main file. Default to . |

To integrate SonarQube, you will need to import and call the SonarQube template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sonarqube-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use SonarQube in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### SONARQUBE VARIABLES
  SONAR_TOKEN: "my-sonarqube-token"
  SONAR_HOST_URL: "http://your-sonarqube-instance.org"
  SONAR_PROJECT_KEY: "my-project-key"
  SONAR_PROJECT_NAME: "my-project"
  SONAR_SOURCES: "."

# Other variables below...
```

3. Add a ```sonarqube``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - sonarqube
  # Other stages below...
```

4. Call the ```sonarqube``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# SONARQUBE CALL
sonarqube:
  stage: sonarqube
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sonarqube-stage.yml'

    # Other include statements...

variables:

# Other variables above...

### SONARQUBE VARIABLES
  SONAR_TOKEN: "my-sonarqube-token"
  SONAR_HOST_URL: "http://your-sonarqube-instance.org"
  SONAR_PROJECT_KEY: "my-project-key"
  SONAR_PROJECT_NAME: "my-project"
  SONAR_SOURCES: "."

# Other variables below...

stages:
  # Other stages above...
  - sonarqube
  # Other stages below...

# Other stage calls above...

# SONARQUBE CALL
sonarqube:
  stage: sonarqube
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

## Dependency Check integration with SonarQube

### Overview
It is possible to integrate Dependency Check reports in SonarQube v7.6 or higher with the use of [Dependency-Check Plugin for SonarQube](https://github.com/dependency-check/dependency-check-sonar-plugin).

> **Notes**: This SonarQube plugin does not perform analysis, rather, it reads existing Dependency-Check reports.

### Requirements

1. Import Dependency Check stage.

2. Install and set up [SonarQube server](https://docs.sonarqube.org/latest/setup/install-server/).

2. Install the plugin [Dependency-Check Plugin for SonarQube](https://github.com/dependency-check/dependency-check-sonar-plugin)

### Configuration

You need to configure a few environment variables in your ```.gitlab-ci.yml``` file:

| Environment variables  | Description                               |
| --------------------- | ----------------------------------------- |
| ```SONAR_TOKEN``` | Authentication token of a SonarQube user with Execute Analysis permission on the project. **YOU SHOULD SET THIS AS ENVIRONMENT VARIABLE IN YOUR GITLAB PROJECT** |
| ```SONAR_HOST_URL``` | Server URL (e.g., http://your-sonarqube-instance.org)|
| ```SONAR_PROJECT_KEY``` | Unique identifier for the project. Allowed characters are: letters, numbers, -, _, . and :, with at least one non-digit. |
| ```SONAR_PROJECT_NAME``` | Name of the project that will be displayed. Default : $CI_PROJECT_NAME |
| ```SONAR_SOURCES``` | Comma-separated relative paths from $CI_PROJECT_DIR containing main file. Default to . |
| ```SONAR_DCHECK_SKIP``` | Set to "true" to enable the using of the plugin. Default to "false" |
| ```SONAR_DCHECK_SUMMARIZE``` | Summarize all vulnerabilities of one dependency into one issue. Ideal for large project. Default to "false" |
| ```SONAR_DCHECK_SEVERITY_BLOCKER```<br>```SONAR_DCHECK_SEVERITY_CRITICAL```<br>```SONAR_DCHECK_SEVERITY_MAJOR```<br>```SONAR_DCHECK_SEVERITY_MINOR``` | Minimum score of each severity of the created issues. To disable severity, set to -1. Default to 9, 7, 4, 0 |
| ```SONAR_DCHECK_SECURITY_HOTSPOT``` | Set to "true" to enable review process in the team if working with [Security-Hotspots](https://docs.sonarqube.org/latest/user-guide/security-hotspots/) . Default to "false" |

To integrate SonarQube with Dependency Check plugin, you will need to import and call the SonarQube with Dependency Check plugin template to your ```.gitlab-ci.yml``` file:

1. Add the following include statement to your ```.gitlab-ci.yml``` file:

```yml
include:

    # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sonarqube-dcheck-stage.yml'

    # Other include statements...
```

2. Add the variables needed to use SonarQube with Dependency Check plugin in your ```.gitlab-ci.yml``` file:

```yml
variables:

# Other variables above...

### SONARQUBE WITH DEPENDENCY CHECK VARIABLES
  SONAR_TOKEN: "my-sonarqube-token"
  SONAR_HOST_URL: "http://your-sonarqube-instance.org"
  SONAR_PROJECT_KEY: "my-project-key"
  SONAR_PROJECT_NAME: "my-project"
  SONAR_SOURCES: "."

  SONAR_DCHECK_SKIP: "false"
  SONAR_DCHECK_SUMMARIZE: "false"

  # Specify a score of -1 to completely disable a severity
  SONAR_DCHECK_SEVERITY_BLOCKER: 9
  SONAR_DCHECK_SEVERITY_CRITICAL: 7
  SONAR_DCHECK_SEVERITY_MAJOR: 4
  SONAR_DCHECK_SEVERITY_MINOR: 0

  SONAR_DCHECK_SECURITY_HOTSPOT: "false"

# Other variables below...
```

3. Add a ```sonarqube``` stage, to the stage list of your ```.gitlab-ci.yml``` file:

```yml
stages:
  # Other stages above...
  - sonarqube
  # Other stages below...
```

4. Call the ```sonarqube``` stage and replace ```<YOUR_SHELL_RUNNER_NAME>``` with your **Shell** runner tag name in your ```.gitlab-ci.yml``` file:

```yml
# Other stage calls above...

# SONARQUBE WITH DEPENDENCY CHECK CALL
sonarqube:
  stage: sonarqube
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```

Your final ```.gitlab-ci.yml``` file should be like:
```yml
include:

  # Other include statements...

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dcheck-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sonarqube-dcheck-stage.yml'

  # Other include statements...

variables:

# Other variables above...

### DEPENDENCY CHECK VARIABLES
  PROJECT_SOURCES_FOLDER: "src_folder"

### SONARQUBE WITH DEPENDENCY CHECK VARIABLES
  SONAR_TOKEN: "my-sonarqube-token"
  SONAR_HOST_URL: "http://your-sonarqube-instance.org"
  SONAR_PROJECT_KEY: "my-project-key"
  SONAR_PROJECT_NAME: "my-project"
  SONAR_SOURCES: "."

  SONAR_DCHECK_SKIP: "false"
  SONAR_DCHECK_SUMMARIZE: "false"

  SONAR_DCHECK_SEVERITY_BLOCKER: 9
  SONAR_DCHECK_SEVERITY_CRITICAL: 7
  SONAR_DCHECK_SEVERITY_MAJOR: 4
  SONAR_DCHECK_SEVERITY_MINOR: 0

  SONAR_DCHECK_SECURITY_HOTSPOT: "false"

# Other variables below...

stages:
  # Other stages above...
  - dcheck
  - sonarqube
  # Other stages below...

# Other stage calls above...

# DEPENDENCY CHECK CALL
dcheck:
  stage: dcheck
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# SONARQUBE WITH DEPENDENCY CHECK CALL
sonarqube:
  stage: sonarqube
  dependencies:
    - dependency-check
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# Other stage calls below...
```


## Examples

This section is composed of security tools integration with the library, within realistic context.

### Integration with Docker Compose stage

The following example shows the integration of all the security tools, within a pipeline using docker-compose to build the application.

We can note a few things from this file:
- Each tool is imported one by one within the ```ìnclude``` statement
- Each tool variables are set within the ```variables``` statement
- Each tool stage is declared in the ```stages``` statement
- The ```before_script``` stage is declared and called before all the other tools stages
- Each tool stage is called with a specific runner tag

This example file can be found on ```examples/docker-compose-example.yml```:

```yml
include:   
  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dast-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/anchore-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dcheck-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/dbench-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/ffuf-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/nmap-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sqlmap-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sslyze-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/arachni-stage.yml'

  - project: 'shared-assets/shared-library'
    ref: master
    file: '/templates/sonarqube-stage.yml'

variables:

# BUILD AND DEPLOY VARIABLES

### DOCKER REGISTRY VARIABLES
  ### DOCKER REGISTRY DOMAIN, TO STORE BUILD DOCKER IMAGE 
  DOCKER_REGISTRY: 
  ### USERNAME TO LOG INTO DOCKER REGISTRY- YOU HAVE TO SET IT AS ENV VARIABLE IN GITLAB
  DOCKER_REGISTRY_USER: "$DOCKER_REGISTRY_USERNAME"
  ### PASSWORD TO LOG INTO DOCKER REGISTRY - YOU HAVE TO SET IT AS ENV VARIABLE IN GITLAB
  DOCKER_REGISTRY_PASS: "$DOCKER_REGISTRY_PASSWORD"

# SECURITY TOOLS VARIABLES

### DEPENDENCY CHECK VARIABLES (see https://owasp.org/www-project-dependency-check/) (default is "src")
  ### PROJECT SOURCES FOLDER
  PROJECT_SOURCES_FOLDER: "src"

### DOCKER BENCH VARIABLES (see https://github.com/docker/docker-bench-security) (default is "custom-host")
  CUSTOM_DOCKER_NETWORK: "custom-host"

### ANCHORE VARIABLES (see https://anchore.com/)
  ANALYZED_DOCKER_IMAGE: "myapplication:myversion"

### DAST VARIABLES (for OWASP ZAP, FFUF and ARACHNI)
  ### WEBSITE TO ANALYZE URL
  WEBSITE: "$WEBSITE_URL"
  ### LOGIN PAGE URL, TO ACCESS WEBSITE TO ANALYZE
  LOGIN_URL: "$WEBSITE_LOGIN_URL"
  ### USERNAME TO LOG INTO WEBSITE - YOU HAVE TO SET IT AS ENV VARIABLE IN GITLAB
  USERNAME: "$OWASP_USERNAME"
  ### PASSWORD TO LOG INTO WEBSITE - YOU HAVE TO SET IT AS ENV VARIABLE IN GITLAB
  PASSWORD: "$OWASP_PASSWORD"

### NMAP VARIABLES
  DOMAIN: "$DOMAIN"

### SQLMAP VARIABLES
  ### PAGE URL WITH PARAMETERS
  TARGET: "$WEBSITE_LOGIN_URL"

### SSLYZE VARIABLES
  HOST: "$DOMAIN"

### SONARQUBE VARIABLES
  SONAR_TOKEN: "my-sonarqube-token"
  SONAR_HOST_URL: "http://your-sonarqube-instance.org"
  SONAR_PROJECT_KEY: "my-project-key"
  SONAR_PROJECT_NAME: "my-project"
  SONAR_SOURCES: "."

# STAGES DEFINITION
stages:
  - up
  - dast
  - anchore
  - dcheck
  - dbench
  - ffuf
  - nmap
  - sqlmap
  - sslyze
  - arachni
  - sonarqube
  - clean

before_script:
  - mkdir reports

 # RUN DOCKER
docker-compose-up:
  image: docker/compose
  stage: up
  allow_failure: true
  script:
    - docker login -u $DOCKER_REGISTRY_USER -p $DOCKER_REGISTRY_PASS $DOCKER_REGISTRY
    - docker ps -a -q | xargs docker rm -fv
    - docker-compose up -d
  tags:
    - oflxrun-s01

# DEVSECOPS INIT
devsecops-init:
  stage: devsecops-init
  tags:
    - oflxrun-s01

# DAST STAGE
dast:
  stage: dast
  tags:
    - oflxrun-s01

# ANCHORE STAGE
anchore:
  stage: anchore
  tags:
    - oflxrun-s01

# DEPENDENCY CHECK STAGE
dependency-check:
  stage: dcheck
  tags:
    - oflxrun-s01

# DOCKER BENCH STAGE
bench:
  stage: dbench
  tags:
    - oflxrun-s01

# FFUF STAGE
ffuf:
  stage: ffuf
  tags:
    - oflxrun-s01

# NMAP STAGE
nmap:
  stage: nmap
  tags:
    - oflxrun-s01

# SQLMAP STAGE
sqlmap:
  stage: sqlmap
  tags:
    - oflxrun-s01

# SSLYZE STAGE
sslyze:
  stage: sslyze
  tags:
    - oflxrun-s01

# ARACHNI STAGE
arachni:
  stage: arachni
  tags:
    - oflxrun-s01

# SONARQUBE CALL
sonarqube:
  stage: sonarqube
  tags:
    - <YOUR_SHELL_RUNNER_NAME> # You have to replace this value, with your own "Shell" GitLab runner tag name.

# CLEANUP STAGE
cleanup:
  image: docker/compose
  stage: clean
  script:
    - docker ps -a -q | xargs docker rm -fv
  tags:
    - oflxrun-s01
```
