---
layout: post.html
title: Building a portable dev environment using Vagrant and Berkshelf
tags: [vagrant, virtual-box, berkshelf, devops]
---

## The problem
For the development of our newest application I wanted to use what I've learned using Git and Chef to create an easy to setup dev environment for our developers. I accomplished this by creating Vagrant file inside of the chef repository that contains the cookbooks for our applications. While this works it's fairly clunky, first the developer has to clone two seperate repos also one of these repos contains all the other cookbooks we use for all of other applications. Quite a bit of unnecessary overhead.

## The solution
To simplify this all I'd like to keep the tools to build the dev environment in the project and make sure only the required cookbooks are downloaded.

## The process
First we need to install berkshelf
```
    gem install berkshelf
```
Next we install the vagrant-berkshelf plugin
```
    vagrant plugin install vagrant-berkshelf
```
Now we can configure Berkshelf
```
    berks configure
```
Berkshelf will now run you through the configuration. Remember because we are using Chef solo in vagrant we don't need to specify a chef_server_url as it will be hijacked by vagrant-berkshelf.

The options you need to pay attention to are box and box\_url. Be sure to choose a box that will best mimic the environment you intend to run at production. Myself  I chose box: opscode-ubuntu-12.04-chef-11.4.4, box\_url: https://opscode-vm-bento.s3.amazonaws.com/vagrant/opscode\_ubuntu-12.04\_chef-11.4.4.box to mimic our AWS OpsWorks environment.

Once we have berkshelf configured we can use `berks init` to create our Berksfile and Vagrantfile

Once our Vagrant file is created we want to sync the directory of our git managed application to our dev server as well as forward any ports the application may use. My options look something like this:
```
    config.vm.synced_folder "~/web", "/var/app"
    config.vm.network "forwarded_port", guest: 9080, host: 9080
    config.vm.network "forwarded_port", guest: 9443, host: 9443
    config.vm.network "forwarded_port", guest: 9081, host: 9081
    config.vm.network "forwarded_port", guest: 9444, host: 9444
```
Also we need to make sure that our runlist we are using in production as well as our json used to configure the roles are in the Vagrant file. At the bottom of our Vagrant file the two sections we are looking for are `config.vm.provision` and `chef.run_list`. Below is my configuration for reference.
```
    config.vm.provision :chef_solo do |chef|
      chef.json = {
        "app" => {
          "name" => 'cool_new_app',
          "env" => 'development',
          "url" => 'localhost',
          "branch" => 'dev'
        },
         "redisio" => {
         "default_settings" => {
          }
        },
      }
      chef.run_list = [
        "recipe[ohai]",
        "recipe[apt]",
        "recipe[build-essential]",
        "recipe[runit::default]",
        "recipe[git::default]",
        "recipe[nodejs::install_from_package]",
        "recipe[nodejs::npm]",
        "recipe[mongodb::10gen_repo]",
        "recipe[mongodb::session]",
        "recipe[ulimit::default]",
        "recipe[redisio::install]",
        "recipe[redisio::enable]",
        "recipe[postfix]"
      ]
    end
```
For more information on setting up your Vagrantfile the Vagrant documentation is incredibly helpful ([Vagrant Docs](http://docs.vagrantup.com/v2/)).

Now that our Vagrantfile is setup we need to tell Berkshelf what cookbooks we want and where to get them. For this we need to edit our Berksfile.

Here we want to make sure we include all of the cookbooks that we have listed in our `chef.run_list`. We want to make sure we are pulling the same version of the cookbooks as we use in production as to avoid any incompatibilities moving to deployment. Also, if you have any custom recipes in these cookbooks you want them to be pulled down as well. The way I accomplish this is by telling my Berksfile to pull the cookbooks from the same repository I pull them from in OpsWorks. Below is a sample of my Berksfile.
```
    cookbook "ohai", github: "organiztion/cookbooks", protocol: :ssh, rel: "ohai"
    cookbook "apt", github: "organization/cookbooks",   protocol: :ssh, rel: "cookbooks"
    cookbook "build-essential", github: "organization/cookbooks", protocol: :ssh, rel: "build-essential"
```
For more information on Berksfile see the Berkshelf website ([Berkself](http://berkshelf.com/))

Assuming that we have all of our config files setup correctly we should be able to run `vagrant up` and `vagrant provision` and we should have a fully functioning dev environment. Your developers should be able to code locally as they normally do and connect to our new virtual server to test their code.
