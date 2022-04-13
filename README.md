# Introduction to Snyk Container and IaC Workshop

Snyk Container helps you find and fix vulnerabilities in container images. With snyk container you can scale your security capabilities by enabling developers to quickly eliminate a multitude of vulnerabilities by upgrading to a more secure base image or rebuilding when the base image is outdated. You may not always have access to the original source code that runs in your containers, but vulnerabilities in your code dependencies are still important. Snyk can detect and monitor open source dependencies for popular languages as part of the container scan.

Snyk Infrastructure as Code allows you to find and fix vulnerabilities in your Kubernetes, Helm, Terraform and CloudFormation configuration files. Developer focused infrastructure as code security with Snyk allows you to test and monitor Terraform modules and Kubernetes YAML, JSON, and Helm charts to detect configuration issues that could open your deployments to attack and malicious behavior.

In this hands-on workshop we will achieve the following:

* [Step 1 Fork the Springboot Employee API Application](#step-1-fork-the-springboot-employee-api-application)
* [Step 2 Configure GitHub Integration](#step-2-configure-github-integration)
* [Step 3 Configure Docker Hub Integration](#step-3-configure-docker-hub-integration)
* [Step 4 Test using the Add to Project Docker Hub Integration](#step-4-test-using-the-add-to-project-docker-hub-integration)
* [Step 5 Find vulnerabilities in Springboot Employee API Dockerfile](#step-5-find-vulnerabilities-in-springboot-employee-api-dockerfile)
* [Step 6 Fix the Dockerfile FROM tag using a Pull Request](#step-6-fix-the-dockerfile-from-tag-using-a-pull-request)
* [Step 7 View Kubernetes IaC config file scan from SCM](#step-7-view-kubernetes-iac-config-file-scan-from-scm)
* [Step 8 Snyk CLI scanning for container images](#step-8-snyk-cli-scanning-for-container-images)
* [Step 9 Snyk Kubernetes Integration](#step-9-snyk-kubernetes-integration)

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

## Step 1 Fork the Springboot Employee API Application

Navigate to the following GitHub repo - https://github.com/papicella/springbootemployee-api

* Click on the "**Fork**" button
* Ensure you are forking this repo to your public GitHub account
* Click done

![alt tag](https://i.ibb.co/G3q6ZKR/snyk-K8s-container-iac-workshop.png)

## Step 2 Configure GitHub Integration

First we need to connect Snyk to GitHub, so we can import our Repository. Do so by:

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

You may already have images in your Docker Hub Registries but let's go and add a new one to your Docker Hub account.

* Login to Docker Hub as shown below. These will be the same credentials you used in Step 3 above.

```bash
$ docker login -u DOCKER_HUB_USERNAME -p YOUR_ACCESS_TOKEN_OR_PASSWORD
Login Succeeded
```

* Pull down the following image as shown below.

```bash
$ docker pull pasapples/springbootemployee:multi-stage-add-layers
multi-stage-add-layers: Pulling from pasapples/springbootemployee
Digest: sha256:6e6f1a2df2034fabcb771cddb9dd9c6253c9edee692d208740cdc85573f56ce0
Status: Image is up to date for pasapples/springbootemployee:multi-stage-add-layers
docker.io/pasapples/springbootemployee:multi-stage-add-layers
```

* Run the following commands to TAG and PUSH the image to your Docker Hub account.

Note: Replace DOCKER_HUB_USERNAME with your Docker Hub username.

```bash
$ docker tag pasapples/springbootemployee:multi-stage-add-layers DOCKER_HUB_USERNAME/springbootemployee:multi-stage-add-layers

$ docker push DOCKER_HUB_USERNAME/springbootemployee:multi-stage-add-layers
The push refers to repository [docker.io/pasapples/springbootemployee]
e48e57561479: Layer already exists
a185292d58d7: Layer already exists
5f70bf18a086: Layer already exists
5de890f07ce7: Layer already exists
85b7d768b242: Layer already exists
0376aa443261: Layer already exists
1ffea8569203: Layer already exists
f18b02b14138: Layer already exists
multi-stage-add-layers: digest: sha256:6e6f1a2df2034fabcb771cddb9dd9c6253c9edee692d208740cdc85573f56ce0 size: 1996
```

* Return to the Snyk Dashboard and click on "**Add your Docker Hub Images to Snyk**"

* Search for "**springbootemployee**" and then select that container image using the tag "**multi-stage-add-layers**" and click "**Add Selected Repositories**"

![alt tag](https://i.ibb.co/ZLQx6RL/snyk-K8s-container-iac-workshop-2.png)

* This may take a few minutes, so you can view the Import using the link provided as follows

![alt tag](https://i.ibb.co/WxqFVGh/snyk-K8s-container-iac-workshop-3.png)

* Once complete your container scan should appear as follows.

_Note: Ignore the import warnings this will be explained in the workshop why this occurs_

![alt tag](https://i.ibb.co/hK7bZ61/snyk-K8s-container-iac-workshop-4.png)

When Snyk Container scans an image, using any of the available integrations, we first find the software installed in the image, including:

1. dpkg, rpm and apk operating systems packages.
1. Popular unmanaged software, ie. installed outside a package manager.
1. Application packages based on the presence of a manifest file.

_Note: the container does not need to be run as Snyk reads the info from the file system; therefore, no container or foreign code needs to be run in order to successfully scan._

After we have the list of installed software, we look that up against our vulnerability database, which combines public sources with proprietary research

* Let's go ahead and click on the "**multi-stage-add-layers**" link to view the container issues as part of the scan

![alt tag](https://i.ibb.co/d4Mczcv/snyk-K8s-container-iac-workshop-5.png)

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

## Step 5 Find vulnerabilities in Springboot Employee API Dockerfile

Snyk detects vulnerable base images by scanning your Dockerfile when importing a Git repository. This allows you to examine security issues before building the image, so helps solve potential problems before they land in your registry or in production.

Now that Snyk is connected to your GitHub Account, import the Repo into Snyk as a Project as this contains a Dockerfile.

* Navigate to Projects
* Click "**Add Project**" then select "**GitHub**"
* Click on the Repo "**springbootemployee**" that you forked earlier at Step 1.

![alt tag](https://i.ibb.co/q9Rsxsh/snyk-starter-open-source-3.png)

_Note: The import can take up to one minute, so you can view the import log while it's running as shown below_

![alt tag](https://i.ibb.co/S7G00S7/snyk-K8s-container-iac-workshop-6.png)

* Once imported you should see a reference for the Dockerfile as shown below.

![alt tag](https://i.ibb.co/FxbWNZR/snyk-K8s-container-iac-workshop-7.png)

* Go ahead and click on the Dockerfile this is similar to what a scan of a container from a registry looks like BUT this time we are scanning a Dockerfile itself versus the full container image.

In a Dockerfile project, you can find the relevant metadata of the Dockerfile and the base image used. If the base image is an [Official Docker image](https://docs.docker.com/docker-hub/official_images/), the results include recommendations for upgrades to resolve some of the discovered vulnerabilities

## Step 6 Fix the Dockerfile FROM tag using a Pull Request

Here we will go ahead and fix our Dockerfile using the "**Open a Fix PR**" button as follows:

* Click on "**Open a Fix PR**" for the base image "**openjdk:19-ea-15-jdk-oracle**" as per below

![alt tag](https://i.ibb.co/JRLPvgq/snyk-K8s-container-iac-workshop-8.png)

* Click on "**Open a Fix PR**" on the resulting page as shown below

![alt tag](https://i.ibb.co/GpbycFK/snyk-K8s-container-iac-workshop-9.png)

* A PR is then created as show below. "**Files Changed**" will show you what it's updating in the Dockerfile itself

![alt tag](https://i.ibb.co/RcDrr2r/snyk-K8s-container-iac-workshop-10.png)

* Click on "**Merge Pull Request**" button as shown below

![alt tag](https://i.ibb.co/4FVzZNq/snyk-K8s-container-iac-workshop-11.png)

* Return to the projects dashboard and you will see a new scan has occurred automatically and now our Dockerfile shows much less issues than previously. Of course until we build a new container and add it to the registry the container itself will still have the old base image in place.

![alt tag](https://i.ibb.co/bN9P3XQ/snyk-K8s-container-iac-workshop-12.png)

## Step 7 View Kubernetes IaC config file scan from SCM

Snyk IaC allows you to inspect, find and fix issues in configuration files for Terraform, Cloudformation, ARM Templates or Kubernetes config.

* Return to the projects view page as shown below and notice how we have a "**employee-K8s.yaml**" file which has been scanned. Go Ahead and click on it

![alt tag](https://i.ibb.co/mHPmgDv/snyk-K8s-container-iac-workshop-14.png)

* For each Vulnerability, Snyk displays the following ordered by Line no:

1. Each Vulnerability grouped by line no and severity 
2. Each Vulnerability links to the Snyk policy it was defined against including the path to the issue, what the issue is, the impact and how to resolve it
3. The ability to ignore issues you wish to remove from the list

![alt tag](https://i.ibb.co/Hg7nsFc/snyk-K8s-container-iac-workshop-15.png)

Go ahead and fix one of the IaC security issues identified by Snyk if you have time

## Step 8 Snyk CLI scanning for container images

The Snyk CLI can run a container test on containers sitting in a registry and even your local docker deamon if you like. All the Snyk CLI needs is access to the registry itself which is for public Docker Hub images only requires a "docker login" to achieve that. The following examples show how to use the Snyk CLI to issue a container test.

* Before we get started please make sure you have setup the Snyk CLI. There are various install options as per the links below. Using the prebuilt binaries means you don't have to install NPM to install the Snyk CLI.

1. Install Page - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
1. Prebuilt Binaries - https://github.com/snyk/snyk/releases

_Note: Make sure you have the following version installed or later_

```bash
$ snyk --version
1.898.0
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

* You have already built "**springbootemployee**" so go ahead and test that as shown below, please use your dockerhub account username rather than "**pasapples**"

```bash
❯ snyk container test pasapples/springbootemployee:multi-stage-add-layers

Testing pasapples/springbootemployee:multi-stage-add-layers...

✗ Low severity vulnerability found in xz-utils/liblzma5
  Description: CVE-2022-1271
  Info: https://snyk.io/vuln/SNYK-DEBIAN10-XZUTILS-2444279
  Introduced through: meta-common-packages@meta
  From: meta-common-packages@meta > xz-utils/liblzma5@5.2.4-1
  
....

Tested 91 dependencies for known issues, found 75 issues.

Base Image                   Vulnerabilities  Severity
openjdk:11.0.13-slim-buster  75               0 critical, 4 high, 1 medium, 70 low

Recommendations for base image upgrade:

Major upgrades
Base Image                  Vulnerabilities  Severity
openjdk:17.0.2-slim-buster  72               0 critical, 2 high, 0 medium, 70 low

Alternative image types
Base Image                    Vulnerabilities  Severity
openjdk:19-ea-13-oracle       1                0 critical, 0 high, 1 medium, 0 low
openjdk:18-jdk                1                0 critical, 0 high, 1 medium, 0 low
openjdk:oracle                1                0 critical, 0 high, 1 medium, 0 low
openjdk:11.0.14.1-jdk-oracle  1                0 critical, 0 high, 1 medium, 0 low


Learn more: https://docs.snyk.io/products/snyk-container/getting-around-the-snyk-container-ui/base-image-detection

```

* Finally, can monitor container images using the "**snyk container monitor**" command as shown below, please perform this step now using a different container image this time

```bash
$ snyk container monitor pasapples/springbootemployee:multi-stage-add-layers --org=workshops-admin-org

Monitoring pasapples/springbootemployee:multi-stage-add-layers (docker-image|pasapples/springbootemployee)...

Explore this snapshot at https://app.snyk.io/org/workshops-admin-org/project/ee854907-2a5c-4973-8327-97b6b3245146/history/7faf02db-dde6-42e9-b331-e38cba7f6d54

Notifications about newly disclosed issues related to these dependencies will be emailed to you.
```

* Return to Snyk App UI projects page to see the CLI scan result

![alt tag](https://i.ibb.co/s6nJ2VB/snyk-K8s-container-iac-workshop-13.png)

## Step 9 Snyk Kubernetes Integration

This step will be performed in the workshop by the host so please wait for that demo to be shown live

**Note: Snyk Kubernetes Integration is an Enterprise License feature and can't be used on FREE tier accounts**

Thanks for attending and completing this workshop

![alt tag](https://i.ibb.co/7tnp1B6/snyk-logo.png)

<hr />
Pas Apicella [pas at snyk.io] is a Principal Solution Engineer APJ at Snyk
