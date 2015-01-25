# Provisioning and creating images #

Cananga uses Ansible for provisioning machines and Packer for creating Amazon
machine images (AMIs).

The typical usage is to create a set of AMIs using Packer and Ansible that are
ready to be launched into a cluster.  However it is also possible to run
Ansible independently of Packer to provision machines that are already running.


## Requirements ##

To provision and create images you will need a **build host**.  This is a
machine that has Packer and Ansible installed and which has an IAM profile with
sufficient permissions to launch instances and create images.


### Installing Packer ###

Follow the instructions at [Install
Packer](https://packer.io/docs/installation.html).


### Installing Ansible ###

Follow the instructions at [Installation -- Ansible
Documentation](http://docs.ansible.com/intro_installation.html).  There are
instructions for various distributions including
[Ubuntu](http://docs.ansible.com/intro_installation.html#latest-releases-via-apt-ubuntu).


### IAM profile ###

Create an IAM profile for your build instance according to the instructions at
[Amazon AMI Builder](https://packer.io/docs/builders/amazon.html).


## Usage ##

To build EC2 AMIs cd to the build directory and run Packer:

    cd cananga/build
    packer build -var 'source_ami=ami-1711732d' -var 'provision_role=client' ec2_build.json

The source AMI should be a recent version of Ubuntu or similar in your
preferred AWS region.

Repeat for different provision roles -- master and slave etc.:
    
    packer build -var 'source_ami=ami-1711732d' -var 'provision_role=master' ec2_build.json
    packer build -var 'source_ami=ami-1711732d' -var 'provision_role=slave' ec2_build.json


### Provisioning an existing host ###

*NOTE: This is an optional procedure if you want to run Ansible provisioning on
some host that exists.*  If you only want to create Amazon AMIs from the
existing Packer/Ansible scripts you can skip this section.

Running Ansible provisioning directly is handy when you're developing Ansible
playbooks.  You could do this if you wanted to make changes to the provided
Ansible roles (add your own packages etc.).  There are two modes that can be
used:

* "provision": Apply changes to a host 

* "test": Test that the changes had the desired effect.

In true TDD-style you can write failing tests in the tests playbook (at
cananga/build/test), then provide an implementation in the provision playbook
(at cananga/build/provision) that makes the tests pass.

An example of how to do this would be to set some environment variables that
direct the process.  For example, to run (failing?) playbook tests you could do
the following:

    target=127.0.0.1   # Provide a proper hostname/IP here
    role=client
    mode=test
    ansible-playbook --private-key ssh-key-for-target-host.pem  -i "$mode/inventory" -e "target_host=$target" -e "provision_role=$role" "$mode/playbook.yml"

Note that you'll need a SSH key on the build host that can be used to connect
to the target host.

Then after providing a working implementation switch the mode to "provision"
and run the same command again:

    mode=provision
    ansible-playbook --private-key ssh-key-for-target-host.pem  -i "$mode/inventory" -e "target_host=$target" -e "provision_role=$role" "$mode/playbook.yml"

After provisioning, you would expect the tests to pass.

