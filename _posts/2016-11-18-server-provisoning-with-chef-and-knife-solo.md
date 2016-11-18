---
layout: post
title:  "Server provisoning with chef and knife-solo"
date:   2016-11-18 15:00:00 +0900
categories: jekyll update
---
In this blog post I will show you how I provision my servers with a handy tool called knife-solo. I will not explain how Chef works, but I will mainly focus on the knife-solo tool.

Typically, Chef is set up as master-slave, with a central Chef server and Chef clients, called nodes. The Chef server contains the versioned cookbooks, client information and some other stuff like data bags. The clients are fairly "dumb" and simply contact the Chef server on each run, to check if there is a new cookbook version and report the result of those runs.

For a lot of small companies, this master-slave setup is a bit of an overkill. The master server only adds a couple of features that, in my opinion, you don't really need if you are only managing a couple of servers. Alternatively, Chef has a chef-solo command that does not require a Chef server, but the main disadvantage of this "solo-mode" is that the cookbooks need to be stored locally on the clients (because there's no Chef server they can connect to). Luckily, knife-solo adds a few additional commands to the knife tool that can help you solve that problem.

What knife-solo basically does, is rsync the cookbooks from your workstation to the client over ssh, and executes the chef-solo command with the correct node configuration.

**Note**: All of the following commands should be executed on your workstation and not on your servers. Also, before you start provisioning production servers, test it on a virtual machine first.

## Installing knife-solo
First you will need to install Chef, knife-solo and librarian-chef. We will use Librarian to download and manage community cookbooks. Because these are all based on ruby, the easiest way to install them is using the gem command:

{% highlight ruby %}
> gem install chef
> gem install knife-solo
> gem install librarian-chef
{% endhighlight %}
Then double check the installation by executing knife solo --help.

Preparing the Chef kitchen
Create a folder, and prepare your kitchen:
{% highlight ruby%}
> mkdir knife-solo-example
> cd knife-solo-example
> knife solo init .
Creating kitchen...
Creating knife.rb in kitchen...
Creating cupboards...
{% endhighlight %}
This will leave you with the following folder structure:
{% highlight ruby %}
> ls -al
total 9
drwxr-xr-x  10 minhpd  root   340 23 nov 09:28 .
drwxr-xr-x+ 51 minhpd  root  1734 23 nov 09:30 ..
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 .chef
-rw-r--r--   1 minhpd  root    12 23 nov 09:28 .gitignore
-rw-r--r--   1 minhpd  root   329 23 nov 09:50 Cheffile
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 cookbooks
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 data_bags
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 environments
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 nodes
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 roles
drwxr-xr-x   3 minhpd  root   102 23 nov 09:28 site-cookbooks
{% endhighlight %}
Some further explanation:

 **.chef** - contains a basic knife.rb configuration file.<br/>
 **Cheffile** - the Librarian-Chef configuration file.<br/>
 **cookbooks** - directory for all vendor cookbooks that are installed using Berkshelf or Librarian.<br/>
 **data_bags** - storage for data bag files<br/>
 **environments** - environment configuration (optional).<br/>
 **nodes** - node configuration, knife-solo will automatically create files here for each server you provision.<br/>
 **roles** - role configuration (optional).<br/>
 **site-cookbooks** - directory for all your custom cookbooks.<br/>
## Preparing a server
 Before you can start provisioning your server, you will need to prepare it by executing the knife solo prepare command from your workstation. This will install Ruby and the latest Chef version on your remote server.

{% highlight ruby %}
> knife solo prepare user@host
Bootstrapping Chef...
Installing Chef 11.12.8
Thank you for installing Chef!
Generating node config 'nodes/host.json'...
{% endhighlight %}
Afterwards, you will see that your nodes directory contains a host.json file with the following content:
{% highlight json %}
{
    "run_list": [

    ]
}
{% endhighlight %}
Adding a cookbook
Let's install nginx on the remote server, using the community cookbook and Librarian. Open your Cheffile and add the nginx cookbook like this:
{% highlight ruby %}
site 'http://community.opscode.com/api/v1'
cookbook 'nginx'
{% endhighlight %}
And download the cookbook with all its dependencies with:

{% highlight ruby %}
> librarian-chef install
{% endhighlight %}
Configuring your node
Now that we have the cookbook, we need to tell Chef to install nginx on our server by editing the nodes/host.json file. The run_list section contains all roles and/or recipes for this specific node. Here we will add the nginx::package recipe which will install nginx using the default package manager.

Cookbooks usually have attributes that are used in the recipes for a finer grade of configuration. In our case, the nginx cookbook has several attributes such as the installed version, the number of worker processes, and so on. More information about the available nginx attributes here.
{% highlight json %}
{
    "run_list": [
        "recipe[nginx::package]"
    ],
    "nginx": {    
        "version": "1.4.4",
        "default_site_enabled": false,
        "worker_processes": 4,
        "gzip_comp_level": 5
    }
}
{% endhighlight %}
Let's start cooking
We're ready to provision our server; the cookbooks are downloaded, the node configuration is in place. Let's start cooking:
{% highlight text %}
> knife solo cook user@host
Running Chef on host...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef...
{% endhighlight %}
That should be it. You now have provisioned a remote server using Chef Solo. But remember, don't forget to version your infrastructure. You can easily add the entire kitchen to a git repository so that you do have a central versioned storage for your cookbooks and node configuration.
