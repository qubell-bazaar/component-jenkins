jenkins-ci
==========

![](://s3.amazonaws.com/qubell-images/jenkins-logo.png)

Installs and configures Jenkins CI Server

Version 1.1-41p

[![Install](https://raw.github.com/qubell-bazaar/component-skeleton/master/img/install.png)](https://express.qubell.com/applications/upload?metadataUrl=https://raw.github.com/qubell-bazaar/component-jenkins/1.1-41p/meta.yml)

Features
--------

 - Install and configure Jenkins CI Server on multiple compute
 - Setup Jenkins slave nodes both Windows and Linux
 - Install Jenkins plugins
 - Restore from previosly created backups

Configurations
--------------
 - Jenkins CI 1.xxx (latest), CentOS 6.4 (us-east-1/ami-ee698586), AWS EC2 m1.small, root
 - Jenkins CI 1.xxx (latest), amazon-linux (us-east-1/ami-1ba18d72), AWS EC2 m1.small, ec2-user
 - Jenkins CI 1.xxx (latest), Ubuntu 12.04 (us-east-1/ami-d0f89fb9), AWS EC2 m1.small, ubuntu


Pre-requisites
--------------
 - Configured Cloud Account a in chosen environment
 - Either installed Chef on target compute OR launch under root
 - Internet access from target compute:
   - Jenkins distribution
   - S3 bucket with Chef recipes: qubell-starter-kit-artifacts
   - If Chef is not installed: please install Chef 10.16.2 using http://www.opscode.com/chef/install.sh ```bash <($WGET -O - http://www.opscode.com/chef/install.sh) -v $CHEF_VERSION```

Implementation notes
--------------------
 - Installation is based on Chef recipes from https://github.com/opscode-cookbooks/jenkins/

Configuration parameters
------------------------
 - Server compatible AMI: You can select one of OS listed to run jenkins server.
 - Jenkins port: Select jenkins server listening port Default is 8080
 - Server compatible hardware: Select AWS instance type. Default is m1.small.
 - Jenkins server version: You can select specific version of Jenkins server to be installed.
   Note! Installation will be made from CentOS/Debian repo, so you have to set version like '1.612-1.1' instead of '1.612'.
 - URI to Jenkins backup: URL to jenkins backup to be restored
 - Jenkins backups restore type: You have to provide restoration type - 
   Single job - content of job folder should be packed to archive, archive name should be same as job name. 
   All jobs - directory 'jobs' should be packed to archive.
   Full Jenkins backup - content of Jenkins home folder shoud be packed to archive.
 - Linux slave compatible AMI: Allow set Linux jenkins slave AMI.
 - Windows slave compatible AMI: Allow set Windows jenkins slave AMI.
 - Windows administrator password: Password to 'Administrator' user in Windows OS
 - Windows slave compatible hardware: Here you can choose one of the AWS images types to run Windows OS server. Filed type is "string"
 - Plugins info: This field let specify list of plugins to install. 
   If specified like ["plugin_name1","plugin_name2"]. In this case default action always "install". Plugin's latest version will be installed from the Jenkins plugins repo.
   For more custom management plugins info can be specified in following format: [{"name":"plugin_name", "version":"version_number", "url":"Plugin's_URL", "action":"(install|delete)"}{another plugin}].
   All parameters are optional except name.

Example usage
-------------
```
- install-jenkins-server:
    action: chefsolo
    precedingPhases: [provision-server-vm]
    parameters:
      recipeUrl: "{$.recipe-url}"
      runList: [ "recipe[cookbook_qubell_jenkins::default]" ]
      roles: [ server ]
      jattrs:
        qubell_jenkins: 
          version: "{$.jenkins-version}"
          plugins: "{$.plugins-info}"
          backup_uri: "{$.backup-uri}"
          restore_type: "{$.restore-type}"
        jenkins:
          server:
            host: "{$.jenkins-server-hosts[0]}"
            port: "{$.jenkins-server-port}"
            install_method: "{$.install-method}"
    output:
      server-attrs: chefState

- install-jenkins-slave:
    action: chefsolo
    precedingPhases: [provision-slave-linux-vm, install-jenkins-server]
    parameters:
      recipeUrl: "{$.recipe-url}"
      runList: [ "recipe[cookbook_qubell_jenkins::node]" ]
      roles: [ slave-linux ]
      jattrs:
        jenkins:
          server:
            url: "http://{$.jenkins-server-hosts[0]}:{$.jenkins-server-port}"
            pubkey: "{$.server-attrs['*'].jenkins.server.pubkey[0]}"
          node:
            agent_type: "{$.slave-linux-agent-type}"
            availability: "{$.slave-linux-availability}"
          cli:
            username: "admin"
            password: "{$.server-attrs['*'].jenkins.server.admin_password[0]}"
    output:
      slave-attrs: chefState
``` 
