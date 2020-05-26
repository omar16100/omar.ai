---
title: "Trigger Bitbucket Pipeline Only If Certain Files Are Changed (With Google Cloud Functions)"
date: 2020-05-19T22:38:14+08:00
draft: false
---

![Overview](/posts/images/overview0.png)

<sub><sup>Overview of what you will do.</sup></sub>

Having DevOps practices can make you really productive as a developer. It is a culture in an organization and there are countless tools we can use to implement such solutions.

In this article we will implement a Bitbucket CI/CD pipeline.

> Assumption : you have some familiarity with DevOps, CI/CD, Cloud Functions, Bitbucket and docker. If not, you may visit the [references](#definitions).

## Scenerio

You have multiple cloud functions in a repository and want an individual build-test-deployment option for each function.

![Screenshot of bitbucket repository](/posts/images/screenshot0.png)

> Note : these example functions are extended from the sample provided in GCP documentation.

### Let's say you have two functions which prints

```bash
Hello World!
```

and

```bash
Hello Mars!
```

These functions are hosted in Bitbucket and initially you can deploy them from your local environment :

```bash
gcloud functions deploy hello_world --allow-unauthenticated --runtime=python37 --memory=128MB --source hello_word/ --timeout=300 --trigger-http --entry-point=hello_world
```

But this is not the ideal process. Solution is to use version control and automated build, test and deployment.

Below is a new feature for Bitbucket pipelines at the time of writing which triggers the pipeline when certain files are changed :

```bash
condition:
    changesets:
        includePaths:
          - "path/*"

```

## Hands-on

### Steps

- Create a service account in GCP
- Export the service account credentials to Bitbucket and configure the bitbucket parameters
- Setup Bitbucket pipelines

### From GCP console

- IAM & Admin -> Service Accounts -> Create Service Account
- Choose an appropriate name and click create
- Select 'Service Account User' role
- Click 'Create Key' and save the .json file. Finally, press 'Done' to proceed

Your service account is created and you need to provide some permissions for the Bitbucket pipelines to work properly.

### From IAM & Admin

- Go to 'Roles'
- Select 'Create Role'
- Give suitable names 'Title' and 'ID'
- In 'Add Permissions' add 'Cloud Functions Admin' & 'Service Account User'
- Click 'Create' and your custom role is created which needs to be attached to the service account
- Go to 'IAM' and select your service account which was created earlier
- Click inheritence and add the role to it

Next up is configuring Bitbucket pipelines.

You will have to set up the environment variables first. We have to do this because hard-coding credentials is a bad practice.

Try to make the variables as verbose as possible, it helps when you have multi-environment variables. e.g. think if you had 10 functions from 10 different projects.

The following variables should be added.

```bash
GCP_PROJECT_PROJECT_ID
GCP_PROJECT_PRIVATE_KEY_ID
GCP_PROJECT_PRIVATE_KEY
GCP_PROJECT_EMAIL
GCP_PROJECT_CLIENT_ID
GCP_PROJECT_CERT_URL
GCP_PROJECT_REGION
```

Substitute the 'PROJECT' with your 'project name'.

As this is a how-to guide, I will not be using multi-environment variables. The default naming is not changed.

### In Bitbucket repository

- Go to 'Repository Settings' -> 'Repository variables'
- If enabling pipelines is required, follow the steps
- Commit the default pipeline provided which you are going to change soon
- Add the above mentioned names. The values are from the service account credential file and the region is the one you are using

Create a file named "sa-PROJECT.dist.json". Where 'sa' stands for 'service account' and 'PROJECT' for 'projet name'.

Contents of the file is a copy of your credentials from the service account but with the environment variables.

```bash
{
"type": "service_account",
"project_id": "$GCP_PROJECT_PROJECT_ID",
"private_key_id": "$GCP_PROJECT_PRIVATE_KEY_ID",
"private_key": "$GCP_PROJECT_PRIVATE_KEY",
"client_email": "$GCP_PROJECT_EMAIL",
"client_id": "$GCP_PROJECT_CLIENT_ID",
"auth_uri": "https://accounts.google.com/o/oauth2/auth",
"token_uri": "https://oauth2.googleapis.com/token",
"auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
"client_x509_cert_url": "$GCP_PROJECT_CERT_URL"
}
```

### Now you will work with the bitbucket-pipelines.yml file

> [They have a sweet tool to validate your pipelines.](https://bitbucket-pipelines.prod.public.atl-paas.net/validator)

bitbucket-pipelines.yml :

```bash
pipelines:
  branches:
    master:
      - step:
          name: Deploy hello_world
#          deployment: test/ staging/ production
          image: google/cloud-sdk:234.0.0
          script:
              - apt-get update && apt-get install -y gettext-base
              - envsubst < sa-PROJECT.dist.json > service-account.json
              - gcloud auth activate-service-account --key-file=service-account.json
              - gcloud functions deploy hello_world --runtime python37 --trigger-http --project $GCP_PROJECT_PROJECT_ID --region $GCP_PROJECT_REGION --source hello_world/
          condition:
              changesets:
                 includePaths:
                   - "hello_world/*"
      - step:
          name: Deploy hello_mars
#          deployment: test/ staging/ production
          image: google/cloud-sdk:234.0.0
          script:
              - apt-get update && apt-get install -y gettext-base
              - envsubst < sa-PROJECT.dist.json > service-account.json
              - gcloud auth activate-service-account --key-file=service-account.json
              - gcloud functions deploy hello_mars --runtime python37 --trigger-http --project $GCP_PROJECT_PROJECT_ID --region $GCP_PROJECT_REGION --source hello_mars/
          condition:
              changesets:
                 includePaths:
                   - "hello_mars/*"

```

You can find the code at this [Bitbucket repository](https://bitbucket.org/omar16100/bitbucket_pipelines_custom_trigger_cloud_functions/src/master/). And you can use it as template.

So now if you want to experiment you may modify 'hello_mars' only 'hello mars' will be deployed and vice versa.

And you are done triggering bitbucket pipelines when certain files are changed!

Thanks for reading!

> If you have any feedback, you may comment below or tweet [@omar161000](https://twitter.com/omar161000).
> Connect with me on [Linkedin](https://www.linkedin.com/in/omar16100).

Thanks for reviewing the drafts :

- [Sheikh Hanif](https://www.linkedin.com/in/sheikhhanif/) and
- [Tahmida Sumbula](https://www.linkedin.com/in/tahmida-sumbula-6711081a0/).

### References

- [An introduction to Bitbucket Pipelines](https://bitbucket.org/blog/an-introduction-to-bitbucket-pipelines)
- [Configure bitbucket-pipelines.yml](https://confluence.atlassian.com/bitbucket/configure-bitbucket-pipelines-yml-792298910.html)
- [Cloud Functions : How-to guides](https://cloud.google.com/functions/docs/how-to)
- [Testing & deploying Google Cloud Functions in BitBucket Pipelines](https://www.primitivesense.com/case-studies/ci-with-testing-and-deploying-google-cloud-functions-within-bitbucket-pipelines/)
- [Varibles in pipelines](https://confluence.atlassian.com/bitbucket/variables-in-pipelines-794502608.html)

### Definitions

- [YAML](https://en.wikipedia.org/wiki/YAML)
  - YAML Ain't Markup Language. It is commonly used for configuration files and in applications where data is being stored or transmitted. It uses both Python-style indentation to indicate nesting, and a more compact format that uses [] for lists and {} for maps[1] making YAML 1.2 a superset of JSON.
- [Docker](https://en.wikipedia.org/wiki/Docker_(software))
  - Is a form of delivering softwares in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels. All containers are run by a single operating system kernel and therefore use fewer resources than virtual machines. In this post, docker images have been used run all the commands in the yml file.
- [DevOps](https://en.wikipedia.org/wiki/DevOps)
  - DevOps is a set of practices that combines software development and information-technology operations which aims to shorten the systems development life cycle and provide continuous delivery with high software quality.
- [CI/CD](https://en.wikipedia.org/wiki/CI/CD)
  - generally refers to the combined practices of continuous integration and either continuous delivery and/or continuous deployment. [More detailed](https://opensource.com/article/18/8/what-cicd)
- Pipelines
  - the steps that needs to be done to perform CI/CD
- [Bitbucket](https://en.wikipedia.org/wiki/Bitbucket)
  - Bitbucket is a web-based version control repository hosting service owned by Atlassian, for source code and development projects that use either Mercurial or Git revision control systems. Bitbucket offers both commercial plans and free accounts.
- [Google Cloud Platform (GCP)](https://en.wikipedia.org/wiki/Google_Cloud_Platform)
  - Google Cloud Platform, offered by Google, is a suite of cloud computing services that runs on the same infrastructure that Google uses internally for its end-user products, such as Google Search, Gmail and YouTube.
- [Cloud functions](https://cloud.google.com/functions)
  - Cloud Functions is Google Cloud’s event-driven serverless compute platform. Run your code locally or in the cloud without having to provision servers. Go from code to deploy with continuous delivery and monitoring tools. Cloud Functions scales up or down, so you pay only for compute resources you use. Easily create end-to-end complex development scenarios by connecting with existing Google Cloud or third-party services.

[Go up.](#scenerio)
