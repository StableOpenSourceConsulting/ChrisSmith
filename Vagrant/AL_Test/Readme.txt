README.txt
----------
Instructions to run vagrant vm for Automation Logic test by Chris Smith 12/4/18

Setup environment to recreate test:

- Install VirtualBox5.2 and Vagrant2.03 onto your chossen desktop
  Download and install rpm or deb files from Oracle Virtualbox and Hashicorp Vagrant Download sites 

- create a directory for the tar files to be moved to
  i.e 'mkdir ~/vagrant'

- Download Vagrant 'box' to use as base build of test vm
  'vagrant init puppetlabs/ubuntu-14.04-64-puppet'

- Copy the Vagranfile and .yml and .txt files from tar directory file into directory for vagrant:
  'cd ~/vagrant'
  'cp ~/Downloads/ChrisSmith-AutomationLogic-test-04122018.tar  ./ '
  tar -xvf *.tar

- Run vagrant vm and install all software packages from ~/vagrant directory:

use: 'vagrant up' to start the vm, 
It will auto configure, update and install software automatically.
  
Software includes vim,htop,ansible and nginx + php hello world script.

- 'vagrant ssh' to login to the new vagrant ubuntu vmserver (Hostname:automationlogic-test-ubuntu-cs-1) 

- Test Nginx is running and installed a php app

  open a browser on your vagrant desktop/laptop browser and point to
http://localhost:808/index.php

Gotchas, work arounds and why I did things I met along the way
--------------------------------------------------------------

Ansible version must be the latest otherwise bugs and security vulnerbilities will exisit like:
vagrant local doesnt work in ansible 1.5 fixed 
https://github.com/hashicorp/vagrant/issues/3096#issuecomment-37237086


Ansible Sudoers changes:
by default on the puppet box, vagrant user can already 
sudo su without password due to 
/etc/sudoers.d/10_vagrant

but I have ammended /etc/sudoers for admin group to prompt for passwd
Firstly sudoers file is backed up incase of ansible corrupting file.

changed %admin ALL=(ALL) to
%admin ALL=(ALL) PASSWD: ALL
This was done by using regex to find the line in file that needed to be changed and then adding the ammened sudoers config. Finally the sudoers was validated to make sure the file was Ok'd by visudo
            regexp: '^%admin\s'
            line: '%admin ALL=(ALL) PASSWD: ALL'
            validate: '/usr/sbin/visudo -cf %s'

php packages and php nginx config and php 'hello world' script are all installed via one playbook: app-copy-nginx-reconfig-for-php.yml

Tricky'ist parts in this playbook where getting the correct syntax /layout for installing multiple php packages installed needed for php script to run on nginx by using a list. 
I also used regex to find a location and replace a block of code in the nginx default config file.

Once vagrant has finished provisioning server you should get a hostname displayed on webpage
This is used to test load balancing later.
i.e Each seperate Webserver instance will display its own local hostname when a browser connects to it and this will change if load balacing or FT changes.
 You can then use wget or curl to test operation of webiste or jmeter for that matter!

Most of this test was completed by eith using old vagrantfiles/playbooks I have generated in the past, reading documentation on vagrant or ansible site or using dedicated devops forums.

I hope you enjoted the Automation Logic I used to complete this task.

I would make sense to repackage unique build of the box's after it has completed, so it can be installed faster in a real world application.
