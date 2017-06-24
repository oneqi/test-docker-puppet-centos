# test-docker-puppet-centos
Test environment for puppet in centos

## Environment

This test lab uses Docker for Windows     
However, you can use the scripts in other test labs where you can create a Centos7 VM or server;         
* client 
* workstation     
Where client is a puppet node with the puppet agent installed ... this will be a docker container  
Workstation is your computer where you have installed Docker    
We can use 'client' to test puppet configs, manifests, etc    
  
The image used will be "centos:7" and it will be labelled 'client' within docker and given a hostname of 'puppet'     
The network will be the default (NAT)     
On your workstation place all scripts under c:\admin, i.e.  
puppet.conf  
helloworld.pp  
testfile.pp  

## Client

### Docker Image

We need to pull the latest CentOs image from Docker  
```docker pull centos:7```

### Docker Container

When we launch the container, we need to share the c:\admin folder on your 'workstation' with the 'client' container  
```docker run -it -v c:/admin:/home/ -h puppet --name client centos:7 /bin/bash```

### Inside the Docker Container

Now that we are on the bash prompt of the client we can;   
Check the c:\Admin folder is mounted  
You should see the two .pp files as well as the .conf file !    
```
cd /home
ls -lL
```
Update and download tree  
```
yum -y update
yum install -y tree
```

We now need the URL for the latest Puppet Collection package repository .. the list is available from  
https://docs.puppet.com/puppet/latest/puppet_collections.html#enterprise-linux-7  
We can then download the latest for our CentOS  
```rpm -Uvh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm```

Now to check that our repo is present .. e.g. check for 'Puppet Labs PC1 Repository el 7 - x86_64'  
```yum repolist```

Now we can install the puppet agent
```
cd /
yum install -y puppet-agent
```

Then we can check the dependencies are present    
```ls -la /opt/puppetlabs/bin/```  

This is an example of the output ...  
lrwxrwxrwx 1 root root 33 Apr 6 4:41 facter -> /opt/puppetlabs/puppet/bin/facter # evaluates a system and provides a number of facts about it  
lrwxrwxrwx 1 root root 32 Apr 6 4:41 hiera -> /opt/puppetlabs/puppet/bin/hiera # wrt manifests and modules .. allows you to provide default values and then override or expand them through a customizable hierarchy  
lrwxrwxrwx 1 root root 30 Apr 6 4:41 mco -> /opt/puppetlabs/puppet/bin/mco # Marionette Collective, or MCollective, is an orchestration framework   
lrwxrwxrwx 1 root root 33 Apr 6 4:41 puppet -> /opt/puppetlabs/puppet/bin/puppet    

Next, we need to adjust the PATH  
```$PATH```  
Take note of the contents and add '/opt/puppetlabs/bin' at the end, e.g.
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/puppetlabs/bin
$PATH
export PATH
```

Now check puppet if is working  
```puppet --version```

We can also replace the default puppet.conf file
This will be copied from c:\admin  
```
cp /etc/puppetlabs/puppet/puppet.conf /etc/puppetlabs/puppet/puppet.conf.old
rm -f /etc/puppetlabs/puppet/puppet.conf
cp /home/puppet.conf /etc/puppetlabs/puppet
```

#### Testing Puppet

Find all the 'hard' facts about the 'client' OS  
```facter```

Create a test environment and copy some files from c:\admin      
```
mkdir /etc/puppetlabs/code/environments/test
mkdir /etc/puppetlabs/code/environments/test/manifests/
cp /home/helloworld.pp /etc/puppetlabs/code/environments/test/manifests/
cp /home/testfile.pp /etc/puppetlabs/code/environments/test/manifests/
```

Test a desired state setting ...  
```puppet apply /etc/puppetlabs/code/environments/test/manifests/helloworld.pp```

Create a 'desired file' ...  
This will create a file in the /tmp folder with the name of 'testfile.txt' with fixed permissions  
```puppet apply /etc/puppetlabs/code/environments/test/manifests/testfile.pp```

To check if this has been created ...  
```cat /tmp/testfile.txt```

### Log out of the 'client' container

To log out press CTRL + P + Q  

### Save settings of container as a 'template'

First find out he 'container ID' of the container 'client'  
```docker ps -a```  
Then create a 'template' ...  
```docker commit <container ID of 'client'> puppet-template```  
To create another container from the 'tempalte' ...  
```docker run -it -h puppet-n1 --name puppet-n1 puppet-template /bin/bash```





