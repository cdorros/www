<script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

      ga('create', 'UA-57004739-1', 'auto');
      ga('send', 'pageview');
</script>

This page will contain information in preparation for my OSCON 2016 tutorial:  
[Don't fix it. Throw it away! Introduction to disposable infrastructure](http://conferences.oreilly.com/oscon/open-source-us/public/schedule/detail/49043)

I will add the tutorial steps right before the conference. For now, please work through the prerequisites and section 0.

## Prerequisites
The following are the prerequisites for the course. PLEASE come to the course with all of this sorted out. Don't hesitate to contact me with questions. It's a short lesson, so we don't have time to spend playing IT admin.

### Hardware
Something to hack on. Preferably a laptop.

### Operating System
You are free to choose what you'd like to run this software on. It will be much easier for both of us if you're on OS X or Linux though, as I haven't touched Windows in years. If you're using a VM, make sure it can reach the Internet. Note that I'll be doing the tutorial on a Mac.

### Software
Install the latest version of this software:

* [Vagrant](https://www.vagrantup.com/downloads.html)
* [Terraform](https://www.terraform.io/downloads.html)
* [Packer](https://www.packer.io/downloads.html)

If you're on a Mac, note that most of these are available via 'brew':

	brew cask install virtualbox
	brew cask install vagrant
	brew install packer
	brew install terraform

Install the Ubuntu Trusty 64 box. **This one is important to do before the session**, as it's a large OS image file, and we don't want to overload the conference WiFi. (DoS Techniques is a different course number)
`vagrant box add ubuntu/trusty64`

We'll use Puppet during the tutorial, but it isn't necessary to install this on your laptop as the HashiCorp tools support this.

### Accounts
* **Active AWS account**
  - **This is probably the most important pre-req!** It's the most time consuming of all the setup.
  - You can setup a fresh account - take a look at the Free Tier (https://aws.amazon.com/free/). We will try to spin up resources in AWS that are covered under the Free Tier - I can't guarantee there will be no costs associated with the activity, though (would be very minimal if there is)
  - If you're reusing an existing account, that's fine, but just note that there may be some extra setup in the beginning for you. Also note that **you're at your own risk** with whichever account you decide to use, so you may want to create a fresh account just for the tutorial.
  - Ensure your user account has privilege to create / manage AWS services
  - **IAM "access key ID" and "secret access key" for your user**
    (Getting Your Access Key ID and Secret Access Key)

* **github.com** (optional but recommended)


## 0. Get Setup
1. Ensure you have setup all the required tools and AWS pieces in the [Prequisites](#prerequisites) section
2. Install the Ubuntu Trusty 64-bit Vagrant box:  
   `vagrant box add ubuntu/trusty64`
3. Checkout [this github repo](https://github.com/cdorros/oscon2016) to a place of your choosing. From here on we will refer to this path as [gh_repo].
4. Upload your SSH key to AWS. In the GUI this is under EC2->Key Pairs. Note the name of the SSH key in AWS.
5. Edit the terraform/variables.tf file to add the name of your SSH key, replacing "your-key-name"
*Now We'll setup the networking components of AWS required for the tutorial (VPC, subnets, routing, etc). Don't worry so much about the Terraform commands at this point - we'll cover Terraform in more detail in section 3.*
5. `cd terraform`  
6. Set AWS credentials into environment variables
	`export AWS_ACCESS_KEY_ID='youraccesskeyid'`
	`export AWS_SECRET_ACCESS_KEY='yoursecretkey'`
7. `terraform plan`
8. `terraform apply`
If this fails due to IP CIDR conflict, navigate in the GUI to "VPC->Your VPC" and "VPC->Subnets" and look for a free RFC-1918 range (172.16.0.0/12, 10.0.0.0/8, 192.168.0.0/16). Make sure the subnet is actually a subnet of the VPC range (in my config I'm using 10.0.0.0/16 for the VPC and 10.0.1.0/24 and 10.0.2.0/24 as the two subnets.
9. **Note down the VPC and Subnet IDs outputted at the end, we'll need these later**
