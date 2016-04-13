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

I will add the tutorial steps right before the conference. For now, here's the prerequisites:

Prerequisites
-------------
The following are the prerequisites for the course. PLEASE come to the course with all of this sorted out. Don't hesitate to contact me with questions. It's a short lesson, so we don't have time to spend playing IT admin.

### Hardware
Something to hack on. Preferably a laptop.

### Operating System
You are free to choose what you'd like to run this software on. Linux - sure. Windows - go for it. If you're using a VM, make sure it can reach the Internet. Just note that I'll be doing the tutorial on a Mac.

### Software
Install the latest version of this software:

* [Vagrant](https://www.vagrantup.com/downloads.html)
* [Terraform](https://www.terraform.io/downloads.html)
* [Packer](https://www.packer.io/downloads.html)

If you're on a Mac, note that most of these are available via 'brew'.

We'll use Puppet during the tutorial, but it isn't necessary to install this on your laptop as the HashiCorp tools support this.

### Accounts
* **Active AWS account**
  - This is probably the most important pre-req! It's the most time consuming of all the setup.
  - Take a look at the Free Tier (https://aws.amazon.com/free/). We will only spin
  up resources in AWS that are covered under the Free Tier
  - Ensure your user account has privilege to create / manage AWS services
  - IAM "access key ID" and "secret access key" for your user
    (Getting Your Access Key ID and Secret Access Key)

* **github.com** (optional but recommended)
