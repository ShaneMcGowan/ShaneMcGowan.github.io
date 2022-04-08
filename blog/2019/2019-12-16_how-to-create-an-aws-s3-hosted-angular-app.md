---
title: How to create an AWS S3 hosted Angular App with a Custom Domain, HTTPS and Continuous Deployment
published: true
description: A guide on how to host an Angular app in AWS S3 with a custom domain, HTTPS and continuous deployment from a GitHub repo. (accurate as of 16th of December 2019 _common era_)
tags: AWS, Angular, GitHub, HTTPS
---

A guide on how to host an Angular app in AWS S3 with a custom domain, HTTPS and continuous deployment from a GitHub repo. (accurate as of 16th of December 2019 _common era_)

#Requirements

- A GitHub Account
- A GitHub Repo containing an Angular App created in the root directory
- An AWS Account
- A domain name with the ability to update it's DNS settings
 

# Basic Static Site Setup

#### S3
 - Create a bucket with site name (example.com)
 - Go to the `'Properties'` tab
 - Select `'Static Website'`
 - Select `'Use this bucket to host a website'` and enter index.html for both Index document and Error document (for single page app)
 - Manually add an index.html file to the bucket for testing purposes
 - Make note of the Endpoint, we will be needing this later when setting up our CDN.
 - Go to the `'Permissions'` tab
 - Select `'Bucket Policy'`
 - Copy the following json replacing `BUCKET_NAME` with the name of the bucket (example.com). 

```javascript
{
    "Statement": [
        {
            "Sid": "AllowPublicRead",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::BUCKET_NAME/*"
            ]
        }
    ]
}
```

*__Note__: The policy found in bucket-policy.json will cause all objects in this bucket to be __publicly available by default__, this is required later by our deployment script as normally objects are made private by default. If you do not do this, your site will not be accessible.*

 - Create bucket with site name and subdomain (www.example.com)
 - Go to `'Properties'` tab
 - Select `'Static Website'`
 - Select `'Redirect requests'` and choose `'http'` for the protocol


# Custom Domain and HTTPS

#### Route 53 and DNS
 - Click `'Create Hosted Zone'` and enter your domain example.com with type `'Public Hosted Zone'`
 - Go to the Domain Registrar where you got your domain (eg hover.com)
 - Copy the `ns` values from your newly created hosted zone to nameserver settings for your domain  <br/>

#### AWS Certificate Manager (SSL Certs)
 - Ensure the `us-east-1` region is selected as Cloud Front needs this (top right of dashboard). More info on this here https://aws.amazon.com/certificate-manager/faqs/
 - Add your domain example.com and your domain with the `'www'` subdomain www.example.com then click `'Next'`
 - Select `'DNS Validation'` then click `'Next'`
 - Click `'Review'`
 - Click `'Confirm and Request'`
 - Expand the area for each domain and click `'Create record in route53'`
 - Click `'Continue'`

#### AWS Cloud Front (CDN)
 - Click `'Create Distribution'`
 - Under `'Web'` click `'Get Started'`
 - Under `'Origin Domain Name'` enter the S3 Site Endpoint you made a note of earlier. It should be in the format **bucket.name**.s3-website-**aws-region**.amazonaws.com
 - Under `'Viewer Protocol Policy'`, select `'redirect http to https'`
 - Under `'Price class'` select `'use only US, Canada and Europe'`. Users outside these regions will experience more latency but this is cheaper
 - Under `'Alternate Domain Names (CNAMEs)'` enter your apex domain (example.com) and then on a new line enter your domain with the `'www'` subdomain (www.example.com)
 - Under `'SSL Certificate'` select `'Custom SSL Certificate'` and choose the Certificate we just created in AWS Certificate Manager
 - Under `'Default Root Object'` enter `'index.html'`
 - Click `'Create Distribuion'`

*__Note__: It will take 15 to 20 minutes to create your distribution, be patient and wait before moving on to the next step.*

*__Note__: Your first 1,000 invalidation path requests each month are free, but invalidations after this will cost $0.005 per additonal path invalidated.*

_**Note**: You can have a max of 3,000 path invalidation requests in progress at once. A wildcard '*' request can invalidate a max of 15 paths at once so if your app contains more than 15 files, you will need to remove each path manually in your deployment script, not using the wild card path._

#### Back to Route 53
 - Click `'Hosted Zones'`
 - Click your domain example.com
 - Click `'Create Record Set'` and leave name blank
   Set `'Alias'` to `'Yes'`
   Select `'Alias Target'` and select the Cloud Front deployment that we created earlier from the dropdown
   Click Create
 - Repeat as above but enter `'www'` as the name this time.

*__Note__: This may take several minutes to propogate, I have experienced site without www subdomain working before site with www subdomain. Eventually the www subdomain will work also so be patient.*

**Your site should now be live under https://example.com and https://www.example.com**


# Continuous Delivery

#### AWS Code Pipeline
 - Navigate to AWS Code Pipeline
 - Enter a name for your pipeline
 - Select `'New service role'`, the default name is fine
 - Ensure `'Allow AWS CodePipeline to create a service role so it can be used with this new pipeline'` is checked
 - Click `'Next'`


 - In the `'Source provider'` dropdown, select `'GitHub'`
 - Click `'Connect To GitHub'` and allow AWS access to your GitHub account
 - In the `'Repository'` dropdown, select your repo
 - In the `'Branch'` dropdown, select your desired branch
 - Under `'Change detection options'` ensure `'GitHub webhooks'` is selected
 - Click `'Next'`


 - In the `'Build provider'` dropdown, select `'AWS CodeBuild'`
 - In the `'Region'` dropdown, select your desired AWS region
 - Under `'Project name'`, click `'Create project'`. This will bring you to a popup window.

 - In the Popup Window, under `'Project Name'` enter a desired name
 - Under `'Environment image'` select `'Managed image'`
 - Under `'Operating System'` select `'Ubuntu'`. This includes the language runtimes we need for building our Angular App
 - Under `'Runtimes'` select `'Standard'`
 - Under `'Image'` select `'aws/codebuild/standard:3.0'`. This contains NodeJS version 12 which we want.
 - Under `'Image version'` select `'Always use the latest image for this runtime version'`
 - Under `'Environment Type'` select `'Linux'`
 - Under `'Service role'` select `'New Service Role'` and enter an appropriate name (for example 'codebuild-service-role-s3-angular')
 - Under `'Build specifications'` select `'User a buildspec file'`
 - Click `'Continue to CodePipeline'`

 - You should be brought back to CodePipeline
 - Click `'Next'`

 - Click `'Skip deploy stage'` as we covered this in our build pipeline.

 - Click `'Create pipeline'`


#### AWS CodeBuild
 - Create a `buildspec.yaml` file in the root of your repo. This will be used during the CodeBuild part of our CD pipeline to build your angular app and copy it to the root of the S3 bucket.


_Copy the following to the `buildspec.yaml` file.
Replace `@SITE_NAME` with your site name (example.com).
Replace `@ANGULAR_APP_NAME` with the name of the folder which contains your angular app after running ng build which can be found inside the `dist` folder for default angular apps
Replace `@ANGULAR_APP_ENVIRONMENT` with the build environment for your angular app, `prod` being the default value
Replace `@CLOUDFRONT_DISTRIBUTION_ID` with the ID of the cloudfront distrubtion you created earlier. This can be found in your Cloudfront Dashboard._

```yaml
version: 0.2

env:
  variables:
    S3_BUCKET: "@SITE_NAME"
    APP_NAME: "@ANGULAR_APP_NAME"
    BUILD_ENV: "@ANGULAR_APP_ENVIRONMENT"
    CDN_DISTRIBUTION_ID: "@CLOUDFRONT_DISTRIBUTION_ID"

phases:
  install:
    runtime-versions:
      # NodeJS included in the docker image
      nodejs: 12
    commands:
      # Install node dependancies.
      - npm install

  build:
    commands:
      # Builds Angular application.
      - node ./node_modules/@angular/cli/bin/ng build --$BUILD_ENV

  post_build:
    commands:
      # Clear S3 bucket.
      - aws s3 rm s3://$S3_BUCKET --recursive
      # Copy files from dist to S3
      - cd dist
      - aws s3 cp $APP_NAME s3://$S3_BUCKET --recursive
      # Begin cloudfront invalidation
      - aws cloudfront create-invalidation --distribution-id $CDN_DISTRIBUTION_ID --paths "/*"
```

#### CodePipeline IAM Role Permissions
*An IAM Role will have been created for both Code Pipeline and CodeBuild. You need to give the CodeBuild role the permission to access S3 and Cloudfront*

 - Go to the IAM Dashboard
 - Click on `'Roles'`
 - In the table, click on the role that was created during the CodeBuild in the previous step.
 - Click `'Attach Policy'`
 - Enter `'S3'` into the search bar and click the checkbox beside `'AmazonS3FullAccess'`.
 - Enter `'CloudFront'` into the search bar and click the checkbox beside `'CloudFrontFullAccess'`.
 - Click `'Attach Policy'` to add the policy to this role.


# The End
You did it, good work friend ❤️

Psst... Follow me on twitter https://twitter.com/TheShaneMcGowan