# Low-Cost Basic Live Streaming Platform

## Features

* All-AWS setup
* RTMP input, HLS output
* Playback available over HTTP and/or HTTPS
* Ultra-fast, highly-available, Nginx-based on-the-fly HLS packager (https://github.com/arut/nginx-rtmp-module)
* Works with any HLS-capable player

## HTTP Setup 

### Deploying for HTTP delivery

1. Sign in to the [AWS Management Console](https://aws.amazon.com/console), then click the button below to launch the CloudFormation template. Alternatively you can [download](template.yaml) the template and adjust it to your needs.

[![Launch Stack](https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg)](https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=inexpensive-live-streaming-platform&templateURL=https://s3.amazonaws.com/lostshadow/inexpensive-live-streaming-platform/template.yaml)

2. Choose your instance type; your bottleneck will be the network throughput, so choose that depending on how much you will need to stream (use figures [here](https://cloudonaut.io/ec2-network-performance-cheat-sheet/) as reference); you will be able to change the instance size later, or just discard the stack and create a new one with a different setup
3. Leave all other settings to default
4. Check the `I acknowledge that AWS CloudFormation might create IAM resources` box. This confirms you agree to have some required IAM roles and policies created by *CloudFormation*.
5. Hit the `Create Stack` button. 
6. Wait for the `Status` to become `CREATE_COMPLETE`. Note that this may take **1-2 minutes** or more.
7. Under `Outputs`, notice the keys named `IngressEndpoint` and `HTTPEgressEndpoint`; write these down for using later

### Testing your HTTP delivery

1. Point your RTMP broadcaster (any of [these](https://support.google.com/youtube/answer/2907883) will work) to the rtmp `IngressEndpoint` output by CloudFormation above

	Note that, while some RTMP broadcasters require a simple URI, others (like [OBS Studio](https://obsproject.com)) require a **Server** and **Stream key**. In this case, split the URI above at the last *slash* character, as following:
	
	**Server**: `rtmp://[HOST]/live`  
	**Stream key**: `stream001`

	Also note that **the enpoint may not be ready** as soon as CloudFormation stack is complete; it may take a couple minutes more for the server software to be compiled, installed and started on the virtual server so be patient

## HTTPS Setup

### Deploying for HTTPS delivery

1. Sign in to the [AWS Management Console](https://aws.amazon.com/console), then click the button below to launch the CloudFormation template. Alternatively you can [download](template.yaml) the template and adjust it to your needs.

[![Launch Stack](https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg)](https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=inexpensive-live-streaming-platform&templateURL=https://s3.amazonaws.com/lostshadow/inexpensive-live-streaming-platform/template.yaml)

2. Choose your instance type; your bottleneck will be the network throughput, so choose that depending on how much you will need to stream (use figures [here](https://cloudonaut.io/ec2-network-performance-cheat-sheet/) as reference); you will be able to change the instance size later, or just discard the stack and create a new one with a different setup
3. Set `AvailableOverHTTP` to `no` and `AvailableOverHTTPS` to `yes`
4. Paste the domain name you want to use for HTTPS under `HTTPSDomain` (e.g. `livestreaming.mydomain.com`)
5. Paste the SSL Certificate corresponding to above domain (in PEM format) under `SSLCertificate`; a correct certificate should begin with `-----BEGIN CERTIFICATE-----` and end with `-----END CERTIFICATE-----`
6. Paste the SSL Private Key corresponding to above certificate (in PEM format) under SSLPrivateKey; a correct key should begin with `-----BEGIN RSA PRIVATE KEY-----` and end with `-----END RSA PRIVATE KEY-----`

7. Check the `I acknowledge that AWS CloudFormation might create IAM resources` box. This confirms you agree to have some required IAM roles and policies created by *CloudFormation*.
8. Hit the `Create Stack` button. 
9. Wait for the `Status` to become `CREATE_COMPLETE`. Note that this may take **1-2 minutes** or more.
10. Use the value of `ServerIP` under `Outputs` to set up the DNS record of the domain defined in step 4 (e.g. `livestreaming.mydomain.com`)
11. Under `Outputs`, notice the keys named `IngressEndpoint` and `HTTPSEgressEndpoint`; write these down for using later

### Testing your HTTPS delivery

1. Point your RTMP broadcaster (any of [these](https://support.google.com/youtube/answer/2907883) will work) to the rtmp `IngressEndpoint` output by CloudFormation above

	Note that, while some RTMP broadcasters require a simple URI, others (like [OBS Studio](https://obsproject.com)) require a **Server** and **Stream key**. In this case, split the URI above at the last *slash* character, as following:
	
	**Server**: `rtmp://[HOST]/live`  
	**Stream key**: `stream001`

	Also note that **the enpoint may not be ready** as soon as CloudFormation stack is complete; it may take a couple minutes more for the server software to be compiled, installed and started on the virtual server so be patient

2. Test the HTTPS video URL output by CloudFormation as `HTTPSEgressEndpoint` in your favorite HLS player or player tester. You may use [this one](http://player.wmspanel.com/) if not sure

## Setup for both HTTP and HTTPS

* Just follow the HTTPS setup, yet also set the `AvailableOverHTTP` to `yes`
* CloudFormation will output both a `HTTPEgressEndpoint` and a `HTTPSEgressEndpoint`, you can use them simultaneously according to your needs

## CDN delivery (CloudFront)

1. Follow the steps for `Deploying for HTTP delivery` above
2. Create a `Web` type CloudFront distribution with the following properties:
	* Origin Domain Name: the `Public DNS (IPv4)` of the EC2 instance that's been created by CloudFormation (e.g. ec2-xxx-xx-xxx-xx.compute-1.amazonaws.com)
	* Origin Protocol Policy: `HTTP Only`
	* Allowed HTTP Methods: `GET, HEAD, OPTIONS`
	
	(leave all other settings to default)
4. Broadcast to the RTMP URL output by CloudFormation as you normally would for a HTTP setup
3. Use the following format to compose your HLS playback link:

		http(s)://{CloudFront_domain_name}/hls/stream001.m3u8

e.g. if the `Domain Name` of your CloudFront distribution is 'd13p55o7ap557v.cloudfront.net', your URL would be

		https://d13p55o7ap557v.cloudfront.net/hls/stream001.m3u8



## Notes:

* All the above setup scenarios will output a `stream001` as the stream name. You can replace that with any (URL safe) string, as long as it's consistent on both ingress and egress. You can also have multiple streams running simultaneously simply by naming them differently. 
* CDN delivery is worthwile for consistent audiences of more than (say) 100 simultaneous viewers; using it agains a low audience will incur disproportionate edge transfer costs
* When streaming over CDN, the content will still be available directly; if you want to prevent it from being accessed as such you may want to adjust your security group to only allow traffic from CloudFront
* For use in browser, an HTTPS website will require that video content is delivered over HTTPS, while a HTTP website supports both HTTP and https  


