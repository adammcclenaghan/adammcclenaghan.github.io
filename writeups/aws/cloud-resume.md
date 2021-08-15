<meta name="viewport" content="width=device-width, initial-scale=.5, maximum-scale=12.0, minimum-scale=.25, user-scalable=yes"/>
# The Cloud Resume Challenge

## Table of Contents
1. [Challenge Introduction](#challenge-introduction)
2. [Implementation](#implementation)
3. [Frontend Implementation](#frontend-implementation)
4. [Backend Implementation](#backend-implementation)
5. [Project Summary](#project-summary)

This year (2021) I have been focusing my learning on development with cloud technologies, specifically AWS.

This July I passed the AWS "Certified Cloud Practicioner" certification and I was keen to start putting what I'd learned into practice! While researching project ideas, I came across the ['Cloud Resume Challenge'](https://cloudresumechallenge.dev/) put together by Forrest Brazzeal.

I decided to take on this challenge, with the twist that I'd use Terraform instead of AWS SAM as my 'infrastructure as code' language of choice.
I chose to use Terraform because it seems to be hugely popular these days, it is used in my current workplace and it was highly loved in the
[Stackoverflow developer survey for 2020](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-other-frameworks-libraries-and-tools-loved3) so I was really excited to try it out!

In this blog post I want to give an overview of how I used Terraform and AWS to complete the challenge. I won't be giving away too many details on implementation
as Forrest has requested that we don't give too much away in the write up so that others can enjoy the challenge!

Finally, just to give a quick overview of how much experience I had before working on this challenge, I have a bacherlors degree in Computer Science and I have 4 years of professional experience working with Java on a large enterprise product. I have some exposure to cloud technologies in my workplace but I have not worked
with them too much directly, so a lot of this was new to me (and a lot of fun!). Hope you enjoy the rest of the post!

## The Finished Product

Before I delve into any details, if you'd just like to see my Resume, you can view it [here](https://adammcclenaghan.com)

If you'd like to learn more about the services behind this simple site, keep reading! 


## Challenge Introduction <a name="challenge-introduction"></a>

The basic goal of the Cloud Resume Challenge is to host your resume online using AWS's serverless offerings. 

The challenge has frontend and backend components. We'll use GitHub for version control, and add integrations with GitHub actions to ensure that both the frontend and backend are managed using CI/CD best practices.

This diagram provides an overview of the architecture we'll be building. We're going to use terraform to plan and manage this infrastructure for us!

<img src="{{site.url}}/resources/images/CloudResume/cloud-resume-design-diagram.png"/>

### The Frontend Element
<img src="{{site.url}}/resources/images/CloudResume/cloud-resume-frontend-components.png" width="419" height="268"/>

The frontend consists of:
1. A Resume written in HTML and styled with some CSS. The Resume will also contain some JS which will make calls to our backend to retrieve the number of visitors to the site.
2. An S3 bucket to host our resume as a static site.
3. A domain purchased and served via AWS Route53.
4. An AWS ACM Certificate which will ensure our site is SSL protected.
5. The site will be served via AWS CloudFront which is AWS's globally distributed serverless CDN offering.



### The Backend Element
<img src="{{site.url}}/resources/images/CloudResume/cloud-resume-backend-components.png" width="419" height="268"/>

In order to learn a little bit more about AWS, the resume will also display a visitor counter. Implementing this feature will makeup the backend element of the challenge.

The backend consists of:
1. A DynamoDB table used to keep track of the number of visits to the site.
2. A lambda function responsible for retrieving and setting values in the DyanmoDB table. This will be written in python
3. An AWS API Gateway endpoint. This will be responsibble for serving requests to retrieve the visitor count. It will make calls to the lambda function to handle this.


## Implementation <a name="implementation"></a>

As mentioned previously, I decided to use Terraform to manage the infrastructure for this project. In this section, I'll go into some detail about how I designed the frontend and backend in Terraform.

I will not provide exact details here of how I configured all of the resources in Terraform as Forrest has requested that we don't give away too much detail so that others can enjoy the challenge! For that reason, I will blur information in some of the terraform resources. However I will give a high level overview of the Terraform resources used and how they piece together!

In terms of repository layout, I used separate GitHub repositories for the frontend and backend. If you have a [Terraform cloud](https://www.terraform.io/cloud) account you can setup terraform integration with GitHub actions really easily! There is a great guide [here](https://learn.hashicorp.com/tutorials/terraform/github-actions) on how to set this up. The default GitHub actions template for Terraform integration is excellent and I didn't have to make any changes to it to get it working! With this integration, each time I make a change to a branch with an open Pull Request, I'm able to see the output of `terraform plan` for that PR, this allows me to determine whether the changes made in the PR are going to affect my AWS resources in the intended way. Additionally, once a PR is merged to the `main` branch, GitHub actions will automatically kick off a `terraform apply` for us, this means that all we have to do is merge our changes to the `main` branch and then Terraform will go off and configure our resources based on the changes! Awesome:) 

## Frontend Implementation <a name="frontend-implementation"></a>

When I began this project, I didn't have an HTML copy of my resume and to be honest I was really looking forward to getting stuck into some Terraform, so I decided that I'd leave creating the resume until the end, and instead I'd just use a simple hello world written in html:
```
<!DOCTYPE html>
<html>
   <head>Hello world! Running on serverless! <3</head>
</html>
```

### Purchasing a domain name

We need a domain name so that users can visit the site without having to use the long public endpoint URL that AWS static sites get by default. 

Route 53 makes it very easy to purchase a domain name, I was able to purchase `adammcclenaghan.com` for $15 per year. There's an additional fee of $0.50 per hosted zone per month.

### Configuring an S3 static site

We'll use S3 to store the HTML/CSS for the resume. AWS S3 buckets can be configured as static websites.

We can create a static site S3 bucket in Terraform with the `aws_s3_bucket` resource:

<img src="{{site.url}}/resources/images/CloudResume/frontend-s3-bucket-tf.png" />

As you can see, terraform makes it really simple to setup our S3 bucket as a static site by providing the `website` configuration argument.

Notice that there is also a policy configuration argument for the S3 bucket. This is also defined as a resource block in terraform:

<img src="{{site.url}}/resources/images/CloudResume/frontend-s3-bucket-policy-tf.png" />


Adding objects to the s3 bucket with terraform is also straightforward. We can add the index/error page using the `aws_s3_bucket_object` resource

This resource allows us to specify the bucket we want to add to and the object which we wish to add to it. 

<img src="{{site.url}}/resources/images/CloudResume/frontend-s3-bucket-object-tf.png" />

One interesting argument I want to point out here is the `etag` argument. The reason I've specified this argument is because when I modify the `index.html` file, I want terraform to detect this change so that when my GitHub actions terraform integration performs a `terraform plan/terraform apply` it automatically updates the S3 bucket with the latest version of my `index.html` file. The `etag` forces Terraform to calculate the md5 hash of the file's content as part of a `terraform apply/terraform plan` because by default Terraform does not detect changes in file contents.

With these 3 resource blocks (plus an extra `aws_s3_bucket_object` resource for the error.html file) our S3 coniguration is complete!

### Configuring a CloudFront Distribution

Next, we need to configure a CloudFront distribution. CloudFront is AWS's serverless CDN solution. With CloudFront, the resume site will be cached at the edge allowing faster access globally. Setting up the CloudFront resource in Terraform took me a little longer than the rest of the resources for the frontend, there are quite a lot of configuration options. We can hook our CloudFront distribution up to the S3 bucket and ensure that it serves the index.html file. There are options through Terraform to configure the caching behaviour of our distribution.

<img src="{{site.url}}/resources/images/CloudResume/frontend-aws-cloudfront-distro-tf.png" />

Note also the `viewer_certification` configuration argument. This references the ACM certificate that we'll set up soon. With this, content served from the CloudFront distribution will be SSL encrypted!

### Configuring Route 53 

Route 53 is Amazon's DNS service. We need to hook Route 53 up to the Cloudfront distribution so that visitors of adammcclenaghan.com see the cloud resume. 

To do this we need to use two Terraform resource blocks, the `aws_route53_zone` resource and the `aws_route53_record` resource. 

<img src="{{site.url}}/resources/images/CloudResume/frontend-route53-resources-tf.png" />

One thing to note is that by default when you purchase a domain through AWS, a hosted zone is created automatically by default. What we can do is declare the resource configuration in our terraform file and then use Terraform's `terraform import` command to import the state of the existing hosted zone into our terraform state file, this will allow Terraform to track updates to this existing AWS resource. Instructions to import the state of a hosted zone are detailed [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone) and you can read more about the `terraform import` command [here](https://www.terraform.io/docs/cli/import/index.html). 

We will need to add an additional `aws_route53_record` for certificate validation, but we'll cover that in the next section. 

### Configuring an ACM certificate

The final piece to configure for our backend is an ACM certificate for SSL verification. We need to create an ACM certificate and then we need to add a record to our hosted zone to enable DNS validation of the certificate.  You can read more about DNS validation [here](https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html). We also need to ensure that our Cloudfront resource has the `viewer_certificate` configuration block set to match with the ACM certificate that we create.

To create an ACM certificate we'll use the `aws_acm_certificate` resource and a `aws_acm_certificate_validation` resource to perform the certificate validation.

<img src="{{site.url}}/resources/images/CloudResume/frontend-acm-certificate-setup-tf.png" />

Then, we add another `aws_route53_record` to enable DNS validation of the cert! 

<img src="{{site.url}}/resources/images/CloudResume/frontend-acm-cert-validation-tf.png" />

I'd like to mention one thing which did trip me up while configuring the ACM certificate. Until this point in development I had been using a single aws provider configured to use the `eu-north-1` region. As it turns out, the ACM service is only available in Virginia (us-east) region. So I did have to configure an extra provider at this point and configure the `aws_acm_certificate` and `aws_acm_certificate_validation` resources to use the provider in Virginia!

### Frontend Implementation Summary

With all of these resources configured, we've successfully completed the frontend portion of the project! I used an iterative process to add each of these resources, using separate pull requests for most of the resources. This was a nice way to work as each push to the PR kicked off the Terraform GitHub actions job and displayed the `terraform plan` output so that I could validate whether my changes made sense before merging them to `main` (which results in a `terraform apply` kicking off automatically)

<img src="{{site.url}}/resources/images/CloudResume/frontend-terraform-gh-actions.png" />

Despite looking very simple, I was really happy to see this little `Hello world!` in my browser!

<img src="{{site.url}}/resources/images/CloudResume/frontend-summary.png" />

## Backend Implementation <a name="backend-implementation"></a>

### Configuring DynamoDB

We need a DynamoDB table to keep track of the number of visitors to the cloud resume, so that we can show the count to visitors. 

I decided to design my DynamoDB table with two basic entries, a `siteUrl` and a `visitorCount`:

```
{
   siteUrl: "someurl.com",
   visitorCount: 12345
}
```

With this design, our single DB can be used to track visitor counts to more than just the cloud resume site in future if we want, which is nice.

Creating the DB in Terraform is quick:

<img src="{{site.url}}/resources/images/CloudResume/backend-dynamodb-tf.png" />

That's really all there is to creating the dynamodb table!

### Configuring Lambda

We need a Lambda function to retrieve the visitor count from the DB. This same lambda will also be responsible for updating the visitor count in the DB.

Since we're writing the lambda in python, we'll use the AWS SDK for Python, [boto3](https://aws.amazon.com/sdk-for-python/) to facilitate communication between the lambda and the dynamodb.

For testing, I used the open source [moto](https://github.com/spulec/moto) library to handle mocking the AWS resources for me, this library is really great.

The Terraform resource configuration for the lambda was fairly simple, a lambda can be configured using the `aws_lambda_function` resource. We also need to add an IAM role and attach a policy to the role so that the lambda can read/write to the dynamoDB.

<img src="{{site.url}}/resources/images/CloudResume/backend-lambda-function-tf.png" />

I also added an `aws_cloudwatch_log_group` resource to the lambda. This ensures that logs from the lambda go to a Cloudwatch log group configured with a 1 day retention period. Using a Cloudwatch log group can really help to debug issues with your lambda.

### Configuring API Gateway

API gateway is going to be the entrypoint to the backend component of this project. The API gateway will receive a request for the visitor count, it will make calls to the lambda, and then return the visitor count passed from the lambda back to the caller.

Of all the backend resources, configuring the API gateway probably took the longest. Several different terraform resources are required:

- `aws_apigatewayv2_api` Defines the API gateway, we can provide the name of the gateway and the protocol type.
- `aws_apigatewayv2_integration` This resource links the API gateway to the lambda which it integrates with.
- `aws_lambda_permission` To ensure our API gateway has permissions to invoke the lambda function we created previously.
- `aws_apigatewayv2_stage` A stage for the API gateway. This is basically a deployment environment, think `dev` stage or `prod` stage where your API can have different deployments. I believe a common use case is to perform canary testing.
- `aws_apigatewayv2_route` Routes requests to the API gateway to the correct integration, in our case the `aws_apigatewayv2_integration` we define which routes to the lambda function.
- `aws_cloudwatch_log_group` A Cloudwatch log group, I configure the retention period to 1 day


Quite a few resources for our API entrypoint! 

<img src="{{site.url}}/resources/images/CloudResume/backend-api-gateway-tf.png" />

<img src="{{site.url}}/resources/images/CloudResume/backend-cloudwatch-api-gw-tf.png" />

### A little javascript

Finally to hook it all together we need to add some JS! I kind of saw this as part of the backend but it does live in the frontend repo so, its debatable I guess.

Anyway, not a whole lot to the JS for this, we just make an asynchronous fetch to the API gateway and then return the value we get back!

<img src="{{site.url}}/resources/images/CloudResume/backend-js.png" />

### Backend Implementation Summary

As with the frontend repo, I used the Terraform GitHub actions workflow to perform `terraform plan` on my PRs and `terraform apply` on merges to master. For the backend repo I also added a workflow for python tests. This was quite quick to setup, GitHub have great docs for this [here](https://docs.github.com/en/actions/guides/building-and-testing-python)

<img src="{{site.url}}/resources/images/CloudResume/backend-summary-gh-actions.png" />

So with all my resources configured and deployed I expected that I'd visit `adammcclenaghan.com` in my browser and be greeted with my visitor count! However... I was wrong. After looking in the chrome debug console, I saw there were CORS related errors. It took a bit of googling to solve this, and I had to make some changes to the responses from my lambda to ensure that the correct headers were set. I won't spoil the challenge.

With cors configured, finally we can see the visitor count when visiting the resume site! As you can see, there was quite a lot of configuration required just to get this simple response!

<img src="{{site.url}}/resources/images/CloudResume/backend-summary.png" />

### Frontend content

All that's left to finish out this blog post is to mention that I did then go and update the index.html with my resume details and added some styling with CSS.

Not too much to say here, if you want to see it you can check it out [here](https://adammcclenaghan.com) :) 

## Project Summary <a name="project-summary"></a>

I just want to wrap up this blog post by saying that I loved working on this challenge! Getting hands on with terraform and AWS was great, there's something really satisfying about making changes to a terraform file, pushing it to github and then watching your GitHub actions workflow go off and do all the configuring of AWS resources for you! If you've read the entire post then you can probably tell that this project also allowed me to get hands on with lots of different AWS resources, which was really great.

I'd absolutely recommend giving the challenge a shot, unless you have quite a bit of experience with AWS, I'm sure you'll learn something new!
