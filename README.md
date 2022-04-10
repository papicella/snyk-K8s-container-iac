# Introduction to Snyk Container and IaC Workshop

TODO://

In this hands-on workshop we will achieve the follow:

* [Step 1 Fork the Goof Application](#step-1-fork-the-goof-application)
* [Step 2 Configure GitHub Integration](#step-2-configure-github-integration)
* [Step 3 Configure Docker Hub Integration](#step-3-configure-docker-hub-integration)
* [Step 4 Test using the Add to Project Docker Hub Integration](#step-4-test-using-the-add-to-project-docker-hub-integration)
* [Step 5 Find vulnerabilities in Goof’s Dockerfile](#step-5-find-vulnerabilities-in-goofs-dockerfile)
* [Step 6 Fix the Dockerfile FROM tag using a Pull Request](#step-6-fix-the-dockerfile-from-tag-using-a-pull-request)
* [Step 7 Container Test using the Snyk CLI](#step-7-container-test-using-the-snyk-cli)

## Prerequisites

* public GitHub account - http://github.com
* git CLI - https://git-scm.com/downloads
* snyk CLI - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
* Registered account on Snyk App - http://app.snyk.io
* Docker Hub Account - https://hub.docker.com/
* Docker Desktop running locally - https://www.docker.com/products/docker-desktop

_NOTE: Please ensure you have meet the Prerequisites prior to starting this workshop_

# Workshop Steps

_Note: It is assumed your using a mac for these steps but it should also work on windows or linux with some modifications to the scripts potentially_

## Step 1 Fork the Goof Application

Navigate to the following GitHub repo - https://github.com/papicella/springbootemployee-api

* Click on the "**Fork**" button
* Ensure you are forking this repo to your public GitHub account
* Click done

![alt tag](TODO://boot employee app)

## Step 2 Configure GitHub Integration

First we need to connect Snyk to GitHub so we can import our Repository. Do so by:

* Login to http://app.snyk.io Sign up if you haven't already.
* Navigating to Integrations -> Source Control -> GitHub
* Fill in your Account Credentials to Connect your GitHub Account.

![alt tag](https://i.ibb.co/bPqqybM/snyk-starter-open-source-1.png)

## Step 3 Configure Docker Hub Integration

Enable integration between Docker Hub and Snyk, to start managing your container vulnerabilities. To do that we must first connect to Docker Hub

* Navigating to Integrations -> Container Registries -> Docker Hub
* Enter your Docker Hub username and Access Token and then click Save

Snyk tests the connection values and the page reloads, now displaying Docker Hub integration information and the Add your Docker Hub images to Snyk button. A confirmation message that the details were saved also appears in green at the top of the screen. In addition, if the connection to Docker Hub failed, a notification appears

Note: As the access token, you can either use your DockerHub password or an [access token](https://docs.docker.com/docker-hub/access-tokens/) created in DockerHub. In case 2FA is activated on your account, access token only is applicable

![alt tag](https://i.ibb.co/hYyb7RD/snyk-container-1.png)

![alt tag](https://i.ibb.co/pWkKGmh/snyk-container-2.png)

* Snyk Container registry integration supports detecting application vulnerabilities in container images. To enable that follow the instructions here.

[Detecting application vulnerabilities in container images](https://support.snyk.io/hc/en-us/articles/360008593457-Detecting-application-vulnerabilities-in-container-images)

## Step 4 Test using the Add to Project Docker Hub Integration

You may already have images in your Dockerhub Registries but lets go and add a new one to your Docker Hub account.

* Login to Docker Hub as shown below. These will be the same credentials you used in Step 3 above.

```bash
$ docker login -u DOCKER_HUB_USERNAME -p YOUR_ACCESS_TOKEN_OR_PASSWORD
Login Succeeded
```

* Pull down the following image as shown below.

```bash
$ docker pull pasapples/docker-goof
Using default tag: latest
latest: Pulling from pasapples/docker-goof
Digest: sha256:be2d6b0f5041315f632f44e3528ea513c452cc95ed4a40bc3d70f943ab293e5f
Status: Downloaded newer image for pasapples/docker-goof:latest
docker.io/pasapples/docker-goof:latest
```

* Run the following commands to TAG and PUSH the image to your Docker Hub account.

Note: Replace DOCKER_HUB_USERNAME with your Docker Bub username.

```bash
$ docker tag pasapples/docker-goof:latest DOCKER_HUB_USERNAME/docker-goof:latest

$ $ docker push DOCKER_HUB_USERNAME/docker-goof:latest
The push refers to repository [docker.io/pasapples/docker-goof]
1bc5d83ccce7: Layer already exists
35bda1fbb3d0: Layer already exists
5f70bf18a086: Layer already exists
fbd39f37d7c2: Layer already exists
938fc2ad056c: Layer already exists
c24944d2eccc: Layer already exists
02a318dedbea: Layer already exists
7972420bc26e: Layer already exists
17c76043bf23: Layer already exists
496d6557f1e3: Layer already exists
867786449541: Layer already exists
92d17ee6d9da: Layer already exists
e54368741774: Layer already exists
5a6c4d956b5d: Layer already exists
86ab2c6c5d58: Layer already exists
latest: digest: sha256:be2d6b0f5041315f632f44e3528ea513c452cc95ed4a40bc3d70f943ab293e5f size: 3466
```

* Return to the Snyk Dashboard and click on "**Add your Docker Hub Images to Snyk**"

* Search for "**docker-goof**" and then select it and click "**Add Selected Repositories**"

![alt tag](https://i.ibb.co/mq421V8/snyk-container-3.png)

* This may take a few minutes, so you can view the Import using the link provided as follows

![alt tag](https://i.ibb.co/PQy4pzq/snyk-container-4.png)

* Once complete your container scan should appear as follows

![alt tag](https://i.ibb.co/NTX9KV2/snyk-container-5.png)

When Snyk Container scans an image, using any of the available integrations, we first find the software installed in the image, including:

1. dpkg, rpm and apk operating systems packages.
1. Popular unmanaged software, ie. installed outside a package manager.
1. Application packages based on the presence of a manifest file.

_Note: the container does not need to be run as Snyk reads the info from the file system; therefore, no container or foreign code needs to be run in order to successfully scan._

After we have the list of installed software, we look that up against our vulnerability database, which combines public sources with proprietary research

* Let's go ahead and click on the "**latest**" link to view the container issues as part of the scan

![alt tag](https://i.ibb.co/HGS4MSP/snyk-container-7.png)

One thing you will notice is recommendations for upgrading the base image. This is handy as we can remove a substantial amount of issues just by using an alternative base image from minor upgrades to major upgrades if available will be shown including what issues will remain if the basde image is changed and the container re-built.  

The supported base images can be found at this [link](https://snyk.io/docker-images/)

For each Vulnerability, Snyk displays the following ordered by our [Proprietary Priority Score](https://snyk.io/blog/snyk-priority-score/) :

1. The module (O/S, base image or user instruction layer) that introduced it and, in the case of transitive dependencies, its direct dependency
1. Details on the path and proposed Remediation, as well as the specific vulnerable functions
1. Overview
1. Exploit maturity
1. Links to CWE, CVE and CVSS Score
1. Social Trends
1. Plus more ...

## Step 5 Find vulnerabilities in Goof’s Dockerfile

Snyk detects vulnerable base images by scanning your Dockerfile when importing a Git repository. This allows you to examine security issues before building the image, so helps solve potential problems before they land in your registry or in production.

Now that Snyk is connected to your GitHub Account, import the Repo into Snyk as a Project as this contains a Dockerfile.

* Navigate to Projects
* Click "**Add Project**" then select "**GitHub**"
* Click on the Repo "goof" that you forked earlier at Step 1.

![alt tag](https://i.ibb.co/q9Rsxsh/snyk-starter-open-source-3.png)

_Note: The import can take up to one minute, so you can view the import log while it's running as shown below_

![alt tag](https://i.ibb.co/RQsX6jZ/snyk-starter-open-source-14.png)

* Once imported you should see a reference for the Dockerfile as shown below.

![alt tag](https://i.ibb.co/1rNMFhC/snyk-container-9.png)

* Go ahead and click on the Dockerfile this is similar to what a scan of a container from a registry looks like BUT this tim we are scanning a Dockerfile itself versus the full container image.

In a Dockerfile project, you can find the relevant metadata of the Dockerfile and the base image used. If the base image is an [Official Docker image](https://docs.docker.com/docker-hub/official_images/), the results include recommendations for upgrades to resolve some of the discovered vulnerabilities

## Step 6 Fix the Dockerfile FROM tag using a Pull Request

Here we will go ahead and fix our Dockerfile using the "**Open a Fix PR**" button as follows:

* Click on "**Open a Fix PR**" for the base image "**node:16.6.0-slim**" as per below

![alt tag](https://i.ibb.co/5kY26FR/snyk-container-13.png)

* Click on "**Open a Fix PR**" on the resulting page as shown below

![alt tag](https://i.ibb.co/C0tn01C/snyk-container-14.png)

* A PR is then created as show below. "**Files Changed**" will show you what it's updating in the Dockerfile itself

![alt tag](https://i.ibb.co/py4GdJS/snyk-container-15.png)

* Click on "**Merge Pull Request**" button as shown below

![alt tag](https://i.ibb.co/hCwDCFP/snyk-container-16.png)

* Return to the projects dashboard and you will see a new scan has occurred automatically and now our Dockerfile shows much less issues than previously. Of course until we build a new container and add it to the registry the container itself will still have the old base image in place.

![alt tag](https://i.ibb.co/pbqmR1v/snyk-container-17.png)

## Step 7 Container Test using the Snyk CLI

The Snyk CLI can run a container test on containers sitting in a registry and even your local docker deamon if you like. All the Snyk CLI needs is access to the registry itself which is for public Docker Hub images only requires a "docker login" to achieve that. The following examples show how to use the Snyk CLI to issue a container test.

* Before we get started please make sure you have setup the Snyk CLI. There are various install options as per the links below. Using the prebuilt binaries means you don't have to install NPM to install the Snyk CLI.

1. Install Page - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
1. Prebuilt Binaries - https://github.com/snyk/snyk/releases

_Note: Make sure you have the following version installed or later_

```bash
$ snyk --version
1.675.0
```

* Authorize the Snyk CLI with your account as follows

```bash
$ snyk auth

Now redirecting you to our auth page, go ahead and log in,
and once the auth is complete, return to this prompt and you'll
be ready to start using snyk.

If you can't wait use this url:
https://snyk.io/login?token=ff75a099-4a9f-4b3d-b75c-bf9847672e9c&utm_medium=cli&utm_source=cli&utm_campaign=cli&os=darwin&docker=false

Your account has been authenticated. Snyk is now ready to be used.
```

_Note: If you are having trouble authenticating via a browser with the Snyk App you can setup authentication using the API token as shown below
[Authenticate using your API token](https://support.snyk.io/hc/en-us/articles/360004008258-Authenticate-the-CLI-with-your-account#UUID-4f46843c-174d-f448-cadf-893cfd7dd858_section-idm4557419555668831541902780562)_

_Note: Testing container images through the CLI performs the following steps, so it can take a few minutes on the first scan [Test images with the Snyk Container CLI](https://support.snyk.io/hc/en-us/articles/360003946917-Test-images-with-the-Snyk-Container-CLI)_

1. Downloads the image if it’s not already available locally in your Docker daemon
2. Determines the software installed in the image
3. Sends that bill of materials to the Snyk Service
4. Returns a list of the vulnerabilities in your image

* You have already built "docker-goof" so go ahead and test that as shown below, please use your dockerhub account username rather than "**pasapples**"

```bash
$ snyk container test pasapples/docker-goof:latest

...

Tested 412 dependencies for known issues, found 889 issues.

Base Image  Vulnerabilities  Severity
node:14.1   889              45 critical, 186 high, 196 medium, 462 low

Recommendations for base image upgrade:

Minor upgrades
Base Image  Vulnerabilities  Severity
node:14.17  530              9 critical, 46 high, 40 medium, 435 low

Major upgrades
Base Image   Vulnerabilities  Severity
node:16.5.0  344              3 critical, 31 high, 49 medium, 261 low

Alternative image types
Base Image                Vulnerabilities  Severity
node:16.6.0-slim          60               2 critical, 8 high, 5 medium, 45 low
node:16.6.1-buster-slim   60               2 critical, 8 high, 5 medium, 45 low
node:16.6.0-buster        343              3 critical, 30 high, 49 medium, 261 low
node:16.6.0-stretch-slim  78               6 critical, 11 high, 8 medium, 53 low
```

* The following container test is for a Spring Boot application

```bash
$ snyk container test pasapples/springbootemployee:cnb

....

Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/springbootemployee
Docker image:      pasapples/springbootemployee:cnb
Platform:          linux/amd64
Base image:        ubuntu:bionic-20210325
Licenses:          enabled

Tested 97 dependencies for known issues, found 32 issues.

Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210325  31               0 critical, 1 high, 6 medium, 24 low

Recommendations for base image upgrade:

Minor upgrades
Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210723  25               0 critical, 0 high, 3 medium, 22 low

Major upgrades
Base Image    Vulnerabilities  Severity
ubuntu:20.04  15               0 critical, 0 high, 0 medium, 15 low
```

* There is also a Distroless version if you would like to try with that

```bash
$ snyk container test pasapples/spring-crud-thymeleaf-demo:distroless

...

Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/spring-crud-thymeleaf-demo
Docker image:      pasapples/spring-crud-thymeleaf-demo:distroless
Platform:          linux/amd64
Licenses:          enabled

Tested 20 dependencies for known issues, found 38 issues.
```

* The CLI also allows us to report vulnerabilities of provided level or higher. Now let's go ahead and set that to HIGH using "**--severity-threshold=high**". Only issues tagged as HIGH or CRITICAL will appear on this test run.

```shell
$ snyk container test --severity-threshold=high pasapples/springbootemployee:cnb

Testing pasapples/springbootemployee:cnb...

✗ High severity vulnerability found in systemd/libsystemd0
  Description: Allocation of Resources Without Limits or Throttling
  Info: https://snyk.io/vuln/SNYK-UBUNTU1804-SYSTEMD-1320128
  Introduced through: systemd/libsystemd0@237-3ubuntu10.46, apt/libapt-pkg5.0@1.6.13, procps/libprocps6@2:3.3.12-3ubuntu1.2, util-linux/bsdutils@1:2.31.1-0.4ubuntu3.7, util-linux/mount@2.31.1-0.4ubuntu3.7, systemd/libudev1@237-3ubuntu10.46
  From: systemd/libsystemd0@237-3ubuntu10.46
  From: apt/libapt-pkg5.0@1.6.13 > systemd/libsystemd0@237-3ubuntu10.46
  From: procps/libprocps6@2:3.3.12-3ubuntu1.2 > systemd/libsystemd0@237-3ubuntu10.46
  and 5 more...
  Fixed in: 237-3ubuntu10.49

✗ High severity vulnerability found in openssl
  Description: Buffer Overflow
  Info: https://snyk.io/vuln/SNYK-UBUNTU1804-OPENSSL-1569474
  Introduced through: openssl@1.1.1-1ubuntu2.1~18.04.9, ca-certificates@20210119~18.04.1, openssl/libssl1.1@1.1.1-1ubuntu2.1~18.04.9
  From: openssl@1.1.1-1ubuntu2.1~18.04.9
  From: ca-certificates@20210119~18.04.1 > openssl@1.1.1-1ubuntu2.1~18.04.9
  From: openssl/libssl1.1@1.1.1-1ubuntu2.1~18.04.9
  and 1 more...
  Fixed in: 1.1.1-1ubuntu2.1~18.04.13



Organization:      pas.apicella-41p
Package manager:   deb
Project name:      docker-image|pasapples/springbootemployee
Docker image:      pasapples/springbootemployee:cnb
Platform:          linux/amd64
Base image:        ubuntu:bionic-20210325
Licenses:          enabled

Tested 97 dependencies for known issues, found 2 issues.

Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210325  31               0 critical, 1 high, 6 medium, 24 low

Recommendations for base image upgrade:

Minor upgrades
Base Image              Vulnerabilities  Severity
ubuntu:bionic-20210723  25               0 critical, 0 high, 3 medium, 22 low

Major upgrades
Base Image    Vulnerabilities  Severity
ubuntu:20.04  15               0 critical, 0 high, 0 medium, 15 low

```

The severity threshold allows you to break a build in the event of this threshold being meet using the exit code of the command.

```
Possible exit codes and their meaning:

0: success, no vulns found
1: action_needed, vulns found
2: failure, try to re-run the command
3: failure, no supported projects detected
```

* Finally, can monitor container images using the "**snyk container monitor**" command as shown below, please perform this step now using a different container image this time

```bash
$ snyk container monitor pasapples/spring-crud-thymeleaf-demo:latest --project-name=spring-crud-thymeleaf-demo-container

Monitoring pasapples/spring-crud-thymeleaf-demo:latest (spring-crud-thymeleaf-demo-container)...

Explore this snapshot at https://app.snyk.io/org/workshops-admin-org/project/1cce2457-a6ac-4f09-8465-9ca596a966fa/history/a253804a-bdb2-4ed1-a7d6-54a321e7f725

Notifications about newly disclosed issues related to these dependencies will be emailed to you.
```

![alt tag](https://i.ibb.co/vY63PXR/snyk-container-12.png)

* Return to the Snyk App and this time you will see the container image

## Step 8 Container Reporting Dashboard

_Note: It can take up to an hour for report pages to show full details so if you see very little detail that would be why_

* Back to the Snyk App navigate to the projects page and select "**View report**" for the "**docker-goof**" project as shown below

* The following report page for the "**docker-goof**" container should be displayed

![alt tag](https://i.ibb.co/yPFVyt2/snyk-container-11.png)

Thanks for attending and completing this workshop

![alt tag](https://i.ibb.co/7tnp1B6/snyk-logo.png)

<hr />
Pas Apicella [pas at snyk.io] is an Solution Engineer at Snyk APJ
