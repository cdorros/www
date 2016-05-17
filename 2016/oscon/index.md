<script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
      (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
      m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

      ga('create', 'UA-57004739-1', 'auto');
      ga('send', 'pageview');
</script>

This page contains the instructions and general information for my OSCON 2016 tutorial:  
[Don't fix it. Throw it away! Introduction to disposable infrastructure](http://conferences.oreilly.com/oscon/open-source-us/public/schedule/detail/49043)

[Please complete this survey after the course! Thanks!](https://www.surveymonkey.com/r/ZKKYJHT)

* **Slides**: coming soon
* I plan to leave this site up after the course in case you want to use the tutorial again later.

## Prerequisites
The following are the prerequisites for the course. PLEASE come to the course with all of this sorted out. Don't hesitate to contact me with questions. It's a short lesson, so we don't have time to spend playing IT admin.

### Hardware
Something to hack on. Preferably a laptop.

### Operating System
You are free to choose what you'd like to run this software on. It will be much easier for both of us if you're on OS X or Linux though, as I haven't touched Windows in years. If you're using a VM, make sure it can reach the Internet. Note that I'll be doing the tutorial on a Mac.

### Software


-----------
If you're on the OSCON Wifi, you can download these from the local OSCON server: [http://172.16.0.20/oscon/DorrosChris/](http://172.16.0.20/oscon/DorrosChris/). For the Vagrant box add, you can run this:

		vagrant box add ubuntu/trusty64 \
		http://172.16.0.20/oscon/DorrosChris/trusty-server-cloudimg-amd64-vagrant-disk1.box
-----------

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
  - Ensure your user account has privilege to create / manage AWS services (in IAM->Users->youruser->Permissions->Attach Policy->Administrator Access)
  - **IAM "access key ID" and "secret access key" for your user**
    (Getting Your Access Key ID and Secret Access Key)

* **github.com** (optional but recommended)


## 0. Get Setup
1. Ensure you have setup all the required tools and AWS pieces in the Prerequisites section
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


***STOP HERE! We'll cover sections 1-4 during the tutorial.***

## 1. Vagrant App
In this step we are going to spin up a new Vagrant box, using Ubuntu 14.04, and deploy a web server (nginx) via Puppet.

1. Start in the base directory, [gh_repo]
2. Create a new vagrant instance  
   `vagrant init ubuntu/trusty64`
3. Open the Vagrantfile, and uncomment this line:
   `config.vm.network "forwarded_port", guest: 80, host: 8080`
4. Start the virtual machine and SSH in  
   `vagrant up && vagrant ssh`
5. `exit` out of the SSH session
6. Open the Vagrantfile and add the Puppet provisioning steps:

        config.vm.provision "puppet" do |puppet|
          puppet.manifests_path = "puppet/manifests"
          puppet.manifest_file = "app.pp"
          puppet.module_path = "puppet/modules"
        end
7. Apply the Puppet module  
   `vagrant destroy && vagrant up`
   `Are you sure you want to destroy the 'default' VM? [y/N] y`
8. From outside VM: `curl localhost:8080`  
   You should see the default nginx page
9. Done! Let's tear down the Vagrant instance for now  
   `vagrant destroy`


## 2. Packer
2. `cd [gh_repo]/packer`
3. Open "variables.json" and add the values for aws\_vpc\_id and aws\_subnet\_id that we saved during the setup. For the subnet ID you can pick either one here, it's just where the AMI will be built.
3. Set AWS credentials into environment variables (if not already done)
	`export AWS_ACCESS_KEY_ID='youraccesskeyid'`
	`export AWS_SECRET_ACCESS_KEY='yoursecretkey'`
	
	If you're using Windows, you can add these directly to the variables.tf file [see example here](https://www.terraform.io/docs/providers/aws/index.html). **Be careful though**, you don't want to check this into source control!
4. Let's test out our AWS setup by building a plain Ubuntu 14.04 AMI
   ```packer validate -var-file=variables.json app.json```
   ```packer build -var-file=variables.json app.json```  
   This should spit out an AMI ID at the end if successful.
5. Now we tell Packer to apply our Puppet config during build.  
	Add the following to variables.json: (mind the commas)
	`"puppet_manifest_file": "../puppet/manifests/app.pp"`
6. 	Update the contents of the app.json file to look like the code below. Notice we added the "provisioners" block.
  
		{
		  "builders": [
		    {
		      "type": "amazon-ebs",
		      "region": "{{user `aws_region`}}",
		      "vpc_id": "{{user `aws_vpc_id`}}",
		      "subnet_id": "{{user `aws_subnet_id`}}",
		      "source_ami": "{{user `ubuntu_1404_ami`}}",
		      "instance_type": "{{user `aws_instance_type`}}",
		      "ssh_username": "{{user `aws_ssh_username`}}",
		      "ami_name": "{{user `ami_prefix`}}-{{timestamp}}",
		      "iam_instance_profile": "{{user `iam_instance_profile`}}"
		    }
		  ],
		  "provisioners": [
		    {
		      "type": "shell",
		      "inline": [
		        "sleep 10",
		        "sudo apt-get update",
		        "sudo apt-get install -y puppet"
		      ]
		    },
		    {
		      "type": "puppet-masterless",
		      "manifest_file": "{{user `puppet_manifest_file`}}",
		      "module_paths": [
		        "../puppet/modules"
		      ]
		    }
		  ]
		}
7. Build a new AMI image
   ```packer validate -var-file=variables.json app.json```
   ```packer build -var-file=variables.json app.json```  
   This will spit out an AMI ID at the end (ami-xxxxxxxx). **Save this! Note down that it's the "default nginx image"**
   

## 3. Terraform

1. `$ cd [gh_repo]/terraform`
3. Set AWS credentials into environment variables (if not already done)
	`export AWS_ACCESS_KEY_ID='youraccesskeyid'`
	`export AWS_SECRET_ACCESS_KEY='yoursecretkey'`
3. Copy over the infrastructure files:

		cp 3/* .
5. Open the "ec2_instances.tf" file and update the ami for the "app-1" instance to the AMI ID you saved during the last Packer build.
6. Run Terraform in NOOP mode
   `terraform plan`
7. Build our infrastructure!
   `terraform apply`
8. Once the Terraform run in complete, note the public IP of the app-1 instance outputted. (this can also be found in the terraform.tfstate file or the EC2 section of the AWS GUI)
9. After waiting a few minutes for the instance to boot up (you'll get connection refused until it's finished):  
   `curl [public_ip]`
10. Note down the DNS name of the ELB (elastic load balancer) (elb_fqdn) in the Terraform output
11. This should return no content, since we haven't added the instance behind the load balancer yet:
	`curl [dns_name]`  
Note that `curl -v` would show a 503 (service unavailable) response. Also note you may get a DNS resolution error until the new record is propogated.
12. Add the instance to the ELB by editing "elb.tf". In the "instances" array, add: `"${aws_instance.app-1.id}"`
13. Apply our changes
   `terraform plan`,
   `terraform apply`
14. Try curling the ELB again. Note that it will take some time for this to work, as the instance needs to be healthy for a set period before the load balancer will direct traffic to it. (you could run this under the `watch` command if you have that installed)  
	`curl [dns_name]`

## 4. Making Changes!
Our website is pretty boring. Let's change the content up. We'll now get to exercise the entire deployment workflow we've just built!  

2. `cd [gh_repo]/puppet/modules/app/manifests`  
3. Create a new file to hold our web server config named "config.pp" with the following content  
	
		# config for our OSCON demo app
		#
		class app::config {
		
		  file { "/etc/nginx/sites-available/default":
		    source => "puppet:///modules/${module_name}/default-proxy",
		    mode => "0644",
		    owner => "root",
		    group => "root",
		    ensure => present,
		    notify => Service['nginx'],
		    ;
		  }	
		}
	
4. Add the new config to the "init.pp" file  
   `include app::config`
5. Note that the new configuration is already in the Puppet directory, we just weren't using it. To see it:
  `cat ../files/default-proxy`
6. Test out the changes locally in Vagrant:  

		cd [gh_repo] #(base)
		vagrant up
		curl localhost:8080
   You should see a weather forecast for Austin, TX
7. Bulid a new AMI using Packer:

		cd packer
		packer validate -var-file=variables.json app.json
		packer build -var-file=variables.json app.json
	
 **Save the AMI ID! Note down that it's the "weather app image"**
8. `cd ../terraform`
9. Create a new instance in the Terraform "ec2_instance.tf" config by copy/pasta the app-1 block to an app-2 block (including the "output"). Bump the version number too. Use the new AMI ID for this one.
10. Spin up the new app version instance
    `terraform plan`
    `terraform apply`
11. Grab the public IP address of your new instance from the tfstate file or the GUI, and curl it to ensure it's functioning properly
12. Add the new instance to the ELB (elb.tf, instances array)
13. Open a new terminal to watch the ELB, if you don't already have one open
    `watch curl [elb_dns_name]`
14. `terraform plan`, `terraform apply` (you know the drill)
15. In the curl/watch window, you should see some switching of responses, based on which instance the ELB forwarded your request to. (keepalives, HTTP/2, DNS, and some other technicalities can make this quite variable, so don't worry so much if you don't see much exciting here)
16. Remove the old (app-1) instance from the ELB (elb.tf)
17. `terraform plan`, `terraform apply`
18. Eventually your curl/watch window should ONLY be showing the new, weather app (may take a few minutes). Note the the weather app doesn't display nicely underneath "watch" - run a regular curl to see it in all its glory.

UPGRADE DONE! Notice that we didn't touch any existing production instance to make this change. We didn't even SSH into an EC2 instance once during this whole upgrade!

If you want to cleanup everything we created in Terraform during the tutorial, just run `terraform destroy`
