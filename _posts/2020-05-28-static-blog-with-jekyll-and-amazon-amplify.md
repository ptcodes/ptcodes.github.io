---
layout: post
title: Static Blog with Jekyll and Amazon Amplify
categories: [Amazon,AWS,Amplify,awscli]
---

[AWS Amplify](https://aws.amazon.com/amplify/) is a development platform for building secure, scalable mobile and web applications. Amplify covers the complete application development workflow from version control, code testing, to production deployment, and it easily scales with your business from thousands of users to tens of millions. 

What is a typical workflow for deploying a static website to Amazon AWS? You create a bucket in S3, set up Route 53 and Cloudfront. If you want the site to be built and deployed on every commit you might want to add CodePipeline or a CI/CD tool of your choice such as Travis CI or Circle CI.

But with Amplify you can simplify and automate the whole process and deploy your site with just a few commands.

<!-- more -->

## Pricing

Amplify is very cheap. If you use it for a personal blog with low traffic you get it pretty much for free. At the moment of writing the pricing structure looks like this:

| **Free Tier**               | **Pay as you go**      |
| BUILD & DEPLOY               | BUILD & DEPLOY         |
| 1000 build minutes per month | $0.01 per build minute |
| HOSTING | HOSTING |
| 5 GB stored per month | $0.023 per GB stored per month |
| 15 GB served per month  | $0.15 per GB served |
| The free tier expires at the end of your 12 month AWS Free Tier term. | Multiple sites per project, public SSL certificates included at no extra cost |


## Jekyll

This blog is based on Jekyll which is a simple static site generator for personal, project, or organization sites. It's written in the Ruby language.

And if you have Ruby installed on your machine you can install Jekyll like this:
```bash
gem install bundler jekyll
```

And to generate a new site simply run:
```bash
jekyll new my-awesome-site
```

## Prerequirements

Alright, we still need to do a couple of things. Please create a git repository with your Jekyll site. Amazon Amlify works with Github, Gitlab, Bitbucket and private git repositories.

Please add a file called amplify.yml with the followent content:

```yaml
version: 0.1
frontend:
  phases:
    preBuild:
      commands:
        - bundle install
    build:
      commands:
        - bundle exec jekyll b
  artifacts:
    baseDirectory: _site
    files:
      - '**/*'
  cache:
    paths: []
```

I have my git repository on Gitlab. What we need to do now is to create an OAuth token in Gitlab to use with Amplify.

Please go to [Applications](https://gitlab.com/profile/applications) under your Settings and add a new application as seen below.

![Gitlab](/images/gitlab.png)

Save the generated OAuth token somewhere as you won't be able to get access to it after you close the page.

## AWS Amplify

Let's create an application with Amplify:

```bash
aws amplify create-app --name blog \
	--repository https://gitlab.com/ptcodes/ptimofeev.com \
	--access-token **YOUR_OAUTH_TOKEN** \
	--enable-branch-auto-build \
	--region us-east-1 
```

Output:
```json
{
    "app": {
        "appId": "d3lrx2kfkgliil",
        "appArn": "arn:aws:amplify:us-east-1:149740283335:apps/d3lrx2kfkgliil",
        "name": "blog",
        "repository": "https://gitlab.com/ptcodes/ptimofeev.com",
        "platform": "WEB",
        "createTime": "2020-05-28T11:47:16.925000-04:00",
        "updateTime": "2020-05-28T11:47:16.925000-04:00",
        "defaultDomain": "d3lrx2kfkgliil.amplifyapp.com",
        "enableBranchAutoBuild": true,
        "enableBasicAuth": false,
        "customRules": [],
        "enableAutoBranchCreation": false
    }
}
```

We need to specify a git branch used for deployments:

```bash
aws amplify create-branch --app-id d3lrx2kfkgliil \
  --branch-name master \
  --region us-east-1
```

Output:
```json
{
    "branch": {
        "branchArn": "arn:aws:amplify:us-east-1:149740283335:apps/d3lrx2kfkgliil/branches/master",
        "branchName": "master",
        "stage": "NONE",
        "displayName": "master",
        "enableNotification": false,
        "createTime": "2020-05-28T11:49:43.125000-04:00",
        "updateTime": "2020-05-28T11:49:43.125000-04:00",
        "enableAutoBuild": true,
        "totalNumberOfJobs": "0",
        "enableBasicAuth": false,
        "thumbnailUrl": "https://aws-amplify-prod-us-east-1-artifacts.s3.amazonaws.com/d3lrx2kfkgl
        "ttl": "5",
        "enablePullRequestPreview": false
    }
}
```

And let's assign a domain to our blog:

```bash
aws amplify create-domain-association --app-id d2fn0wj1x1v1n7 \
  --domain-name ptimofeev.com \
  --sub-domain-setting prefix=www,branchName=master \
  --region us-east-1
```

Output:
```json
{
    "domainAssociation": {
        "domainAssociationArn": "arn:aws:amplify:us-east-1:149740283335:apps/d2fn0wj1x1v1n7/domain
        "domainName": "ptimofeev.com",
        "enableAutoSubDomain": false,
        "domainStatus": "CREATING",
        "subDomains": [
            {
                "subDomainSetting": {
                    "prefix": "www",
                    "branchName": "master"
                },
                "verified": false,
                "dnsRecord": "www CNAME null"
            }
        ]
    }
}
```

Amazon will create and configure an SSL certificate for your site, verify ownership of the domain and propogate it to the Amazon Global CDN. The whole process takes around 10-15 minutes. You can use the same command to see the current status (look at the domainStatus value).

And finally to deploy our blog we should either start the deployment explicitly:
```bash
aws amplify start-deployment --app-id d3lrx2kfkglii \
  --branch-name master \
  --region us-east-1
```

Or make a commit to your repository.

## Useful Commands

To see a list of applications (along with their ids) under your account:
```bash
aws amplify list-apps --region us-east-1
```

Deploy a particular branch:
```bash
 aws amplify start-deployment --app-id d3lrx2kfkglii \
 --branch-name master \
 --region us-east-1
```

Alright, with a few simples steps we got our website up and running. We can add new posts and pages by simple adding respective files to the git repository.
