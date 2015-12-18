---
layout: post
title:	"Personal AWS Server Setup"
description: A guide to ditching traditional hosting providers and setting up a personal AWS EC2 server in the cloud.
permalink: /posts/personal-aws-server
date: 2015-12-19
---

After looking into  AWS pricing I realized how ridiculous it is that I've been running my personal website off of a traditional web host like Dreamhost for the past few years. Not only is AWS cheaper (and free for a year!), but also I get a full fledged server to play around with while exploring the AWS cloud.

But setting things up on AWS is more complicated and the potential to get billed for unexpected reasons is much greater. 

The goals I had for my server were as followed:

1) To not accidentally rack up a massive AWS bill  
2) To have security from nefarious people accessing my instance/account  
3) Simple ssh login (no remembering hostnames, IPs, or passwords)  

Since figuring everything out took awhile, I decided to document it. Here are the steps I used to get up and running.

<!--more-->

## Sign Up For AWS
* Head on over to the [AWS signup page](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) and follow the steps to create an account. A credit card is required.

## (Optional) Create An IAM User For Yourself
The AWS account you're using after signing up is your root account: it has full access to all AWS features including billing information. You can't restrict the permissions associated with your root account and it's also difficult to revoke credentials if they're ever accidentally compromised.

For these reasons, I suggest creating an Identity and Access Management (IAM) user for added security. IAM users are like subaccounts, allowing you to restrict AWS permissions and to revoke credentials. 

* To get started, head to the [IAM Dashboard](https://console.aws.amazon.com/iam/home) and click "Users"

* "Create New Users" and choose a user name when prompted (I chose 'connor'). Generate an access key for the user and download it on the following page. Store the credentials somewhere safe for future use.

<img src="/img/aws/iam-user.png" class="img-responsive" alt="AWS Signup">

Now it's time to give the user a password and some permissions: 

* Click on the newly created user on the IAM dashboard. Under the permissions box select "Attach Policy" and give the user "AdministratorAccess". This allows the user to do everything the root user can, except manage billing. 

Note: Ideally you would further restrict this user, but Admin is a good starting point as it allows you to play with all the AWS features. 

* Next, scroll down and assign the user a password by clicking "manage password". It's easiest to just assign a custom password at creation time, rather than having Amazon choose one for you.

* For added security security turn on Multi-Factor authentication by selecting the "virtual MFA device" option. This will require a code generate by an app on your phone to log in to your AWS account. Follow the directions for your mobile OS of choice.

Lastly we need to make it easier to find the login page for your account. Currently it's a url that includes your impossible to remember account number, so we want to alias that to a more memorable name.

* Back on the IAM Dashboard click "customize" next to the IAM users sign-in link. Change it to something you'll remember, I chose my own name.

<img src="/img/aws/iam-dashboard.png" class="img-responsive" alt="Custom IAM users sign-in link">

## Launch An Instance
Now that the boring stuff is over, let's get our server up and running.

* From the [AWS management console](https://us-west-2.console.aws.amazon.com/console/home) click EC2 (highlighted below) under the compute tab. Then click the blue 'Launch Instance' button on the next page.

<img src="/img/aws/aws-management-console.png" class="img-responsive" alt="AWS management console with EC2 highlighted">

* Now choose an Amazon Machine Image (AMI), which is template that includes an OS and applications. I personally like the Amazon AMI, because it's well integrated into AWS infrastructure and Amazon keeps it updated for us. You may also choose Ubuntu or any other 'Free tier eligible' AMI.

* Click through the next few steps on creating an instance, keeping the default settings and making sure to select the free tier eligible t2.micro instance type. Then, on step 5, feel free to give your instance a memorable name.

* On the 'Configure Security Group' step you may allow all traffic or, optionally, create a new security group that only allows services you will be using. I use the options shown in the image below.

<img src="/img/aws/security-group.png" class="img-responsive" alt="AWS management console with EC2 highlighted">

* Review and click launch. You will then be asked to select a key pair to access your instance. Give it a name and download it. It's important to hold onto this key as it's used to access your instance and can't be re-downloaded.

## (Optional) Create Elastic IP
While you wait for your instance to boot you can optionally create an [Elastic IP](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html). Elastic IPs are basically static public IP addresses that you can move between EC2 instances. They're free as long as you have them associated with an instance, so there's not much reason not to have one.

* From the EC2 (not AWS) dashboard click 'Elastic IPs' in the sidebar under Network & Security.

<img src="/img/aws/elastic-ip.png" class="img-responsive" alt="Location of Elastic IPs link">

* Click the blue button to 'Allocate New Address', and confirm that you want to create one.

* Select the newly allocated address and under the actions menu select 'Associate Address'. Associate it with the instance you created in the last step by selecting the instance from the drop down menu. Note: If you don't see your instance you may have to wait for it to complete its startup process.

<img src="/img/aws/associate-address.png" class="img-responsive" alt="Dropdown for associating elastic IP with instance">

## Access Your Instance
You *could* just follow the instructions provided by AWS on the instances page using a command like `ssh -i "location/of/private-key.pem" ec2-user@123.123.123.123`, but that requires remembering a username, IP, and key location. So lets make it super easy by using the ssh config file. 

Note: These instructions assume your are on a *nix operating system. If you're on windows, follow the instructions provided by Amazon.

* First, change the permissions on your private key file that you downloaded when creating your instance. To do this run `chmod 400 your-private-key.pem`.

* Move your private key to your `~/.ssh` folder.

* Now open up your ssh config file at `~/.ssh/config` in your favorite text editor, creating the file if it doesn't exist. Add the follow lines, substituting your server public IP address and key file location.

{% highlight bash%}
Host aws
  HostName 123.123.123.123
  User ec2-user
  IdentityFile ~/.ssh/aws-private-key.pem
{% endhighlight %}

* To connect to your instance, all you have to do is type `ssh aws` at the command line. Simple and easy.

<img src="/img/aws/login.png" class="img-responsive" alt="Easy ssh login to AWS">

## Set Up Billing Alerts
The more I've used AWS the less I've become worried about being unexpectedly billed, but that doesn't mean you shouldn't prepare for the unexpected. Ideally, I would like to give AWS a month budget that it couldn't go over, but that doesn't currently seem possible using Amazon provided tools. So we're going to use billing alerts instead, which will alert us via email if our estimated charges go over a certain threshold.

* From your root account (not your IAM account) go to the [Billing & Cost Management Dashboard](https://console.aws.amazon.com/billing/home).

* Click 'Preferences' in the sidebar and then check the box to 'Receive Billing Alerts'.

<img src="/img/aws/billing_preferences.png" class="img-responsive" alt="Enable receiving AWS billing alerts">

* Go back to the Billing & Cost Management Dashboard and in the 'Alerts & Notifications' section click to 'Set your first billing alarm'.

* Follow the instructions to set up the alarm. If you're using the free tier, set your alarm to go off if your AWS charges exceed $0 for the month. After a year, you will need to change this value to the cost of running a micro instance for a month.

## Done
You now have your own virtual server to use as you see fit. Throw your website on there and once it's up and running point your domain name's DNS A record at your servers public IP. 

* One useful command to run off the bat is: `sudo yum groupinstall "Development Tools"`. This will install git, gcc, and other common development tools.

You now also have a full AWS account. Amazon's cloud services are extensive and many of its features are available for free the first year. This is a great time to explore AWS and play around with what they have to offer.
