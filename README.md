# trek10-teamdays-cloudfront

This repository will demo three different setups for a static site.

1. No default object for subdirectories
2. CloudFront Function to retrieve default object (index.html) for subdirectories
3. Lambda@Edge function to retrieve default object (index.html) for subdirectories

## What does it mean to retrieve a default object for subdirectories?

CloudFront allows for a [`DefaultRootObject`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cloudfront-distribution-distributionconfig.html#cfn-cloudfront-distribution-distributionconfig-defaultrootobject). By default, going to a CloudFront url, it will show the index.html in the bucket. If you have index.html files in subdirectories, CloudFront will not automatically reroute to the index.html.

This can be seen in the first example. By clicking on the links in the website spun up, they won't properly resolve. You can see where they SHOULD resolve to by manually changing the url to include `/index.html` on the end of the url.

With both of the CloudFront options (CloudFront Function, Lambda@Edge), the Lambda function will see that the url doesn't route to index.html and automatically append it to the request.

## Doesn't S3 have this functionality?

S3 DOES have this functionality when setting up S3 as a static website and if you use the S3 static website url to back a CloudFront distribution, you can get this functionality without using one of the Lambda options described here. HOWEVER, you then need to leave the bucket open to the public and people can go to the HTTP version of your site by visiting the bucket url.

Preferrably, the bucket should be locked down and NOT configured as a static site so that people can only get to your static site through HTTPS on CloudFront. And this is what these examples here will show.

# How to follow along

There is a single CloudFormation template with multiple CloudFormation parameter files. Each parameter file will include the specifics to create the three different setups above. If you look into `.github/workflows/deploy.yaml`, you can see three separate deployments happening on commit. In practice, you will only need the one option you are going with but for demo purposes of this repository, all three are getting deployed.

First, visit broken.charlieguse.com and try to use any of the sublinks on the site. You will see that links are broken and don't work. You can then manually, in the url bar, add `index.html` to the end of it to see the page that SHOULD have come up.

Secondly, visit cf-function.charlieguse.com and you can see how the links now properly redirect you. You will see the same at lambdaedge.charlieguse.com and see that it works identically to cf-function.charlieguse.com.

## Why demo two approaches that accomplish the same thing?
The implementation details will drive AWS costs. If you don't want to consider it deeply, use CloudFront Functions and you are likely to be ok. It will add roughly 10% to your CloudFront invocation charges and an additional 0% to data egress fees. Whether the files are cached or not, the CloudFront Function will always trigger.

By using Lambda@Edge, you can set it on `origin-request` so that your Lambda@Edge function will only trigger if CloudFront needs to hit the S3 bucket origin. The CloudFront Function cannot be set to `origin-request` and can only use `viewer-request` (and `viewer-response`). Lambda@Edge can use `origin-request/response` and `viewer-request/response`. Lambda@Edge is six times more expensive and should only be considered over CloudFront Functions in situatiions where you need to intercept the request to CloudFront on the way to or from the origin. If CloudFront Functions get the ability to work against the origin, then there might not be any practical reason to use Lambda@Edge over CloudFront (at least not in this example).

# How to deploy
You need to create an AWS User via IAM that has the proper permissions to deploy the CloudFormation stacks or you can follow the steps in `.github/workflows/deploy.yaml` and deploy one (or all) of the stacks locally.

# Site content
[Elder.js](https://github.com/Elderjs/elderjs#getting-started) is the sample site being used. There is no modification to it.