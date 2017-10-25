# jenkins

Jenkins server running in a Docker container


## Introduction

The Jenkins server that is built into a Docker image, saved to an [ECR](https://aws.amazon.com/ecr/) repo,
and deployed to [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) running [Docker](https://www.docker.com/).
Then the Datadog agent and SumoLogic collector get provisioned on the master EC2 instance using `Ansible`.


## Features

After all of the AWS resources are created:

__CodePipeline__ will:

  * Get the Jenkins repo from GitHub (https://github.com/autopayeng/jenkins)
  * Build a Docker image from it (https://github.com/autopayeng/jenkins/blob/master/Dockerfile)
  * Save the Docker image to the ECR repo (https://github.com/autopayeng/jenkins/blob/master/buildspec.yml)
  * Deploy the Docker image from the ECR repo to Elastic Beanstalk running Docker stack (https://github.com/autopayeng/jenkins/blob/master/buildspec.yml)
  * Monitor the GitHub repo for changes and re-run the steps above if new commits are pushed into it


__DataPipeline__ will run on the specified schedule and will backup all Jenkins files to an S3 bucket by doing the following:

  * Spawn a new EC2 instance (`t2.micro` in the example below)
  * Mount the EFS filesystem to a directory on the EC2 instance
  * Backup the directory to an S3 bucket
  * Notify about the status of the backup (Success or Failure) via email
  * Destroy the EC2 instance


Once a new Docker image is deployed from the ECR repo to Elastic Beanstalk, __Elastic Beanstalk__ will:

  * Receive the file `Dockerrun.aws.json` and the entire directory `.ebextensions` from the CodePipeline (https://github.com/autopayeng/jenkins/blob/master/buildspec.yml)
  * Mount the EFS filesystem to a directory on the EC2 instance (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/01-efs-mount.config)
  * Setup Java ENV vars (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/02-java.config)
  * Install [Ansible](https://www.ansible.com/) (via `pip`) to configure [Datadog](https://www.datadoghq.com) and [SumoLogic](https://www.sumologic.com/) collectors on the EC2 instance (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/03-ansible.config)
  * Start the `Jenkins` server inside a Docker container


When configuring the Datadog and SumoLogic collectors on the EC2 instance, __Ansible__ does the following:

  * Using `ansible-galaxy` command (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/03-ansible.config), installs all the required roles from the public Cloud Posse repos (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/ansible/requirements.yml). These `Ansible` roles include:
    * https://github.com/cloudposse/ansible-ntp
    * https://github.com/cloudposse/ansible-datadog
    * https://github.com/cloudposse/ansible-sumologic
  * Installs NTP on the EC2 instance by executing the NTP playbook (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/ansible/playbooks/ntpd.yml)
  * Installs Datadog agent on the EC2 instance by executing the Datadog playbook (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/ansible/playbooks/datadog.yml)
  * Installs SumoLogic collector on the EC2 instance by executing the SumoLogic playbook (https://github.com/autopayeng/jenkins/blob/master/.ebextensions/ansible/playbooks/sumologic.yml)


After the Docker image is deployed and Jenkins server starts, __Jenkins__ will:

  * Setup initial admin user, security mode (Matrix), and the number of job executors (https://github.com/autopayeng/jenkins/blob/master/init.groovy)
  * Configure [`Amazon EC2 plugin`](https://wiki.jenkins.io/display/JENKINS/Amazon+EC2+Plugin) to start slaves on demand (https://github.com/autopayeng/jenkins/blob/master/init-ec2.groovy)


## References

* http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_image.html
* http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html
* http://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html
* http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-jvm-ttl.html
* http://docs.oracle.com/javase/7/docs/technotes/guides/net/properties.html
* https://aws.amazon.com/articles/4035
* https://stackoverflow.com/questions/29579589/whats-the-recommended-way-to-set-networkaddress-cache-ttl-in-elastic-beanstalk
* http://docs.ansible.com/ansible/latest/intro_installation.html
* https://cloudacademy.com/blog/get-started-with-ansible-on-the-cloud/
* http://docs.ansible.com/ansible/latest/playbooks_intro.html
* http://docs.ansible.com/ansible/latest/playbooks.html
* https://docs.ansible.com/ansible-container/container_yml/template.html
* http://codeheaven.io/15-things-you-should-know-about-ansible/
* https://wiki.jenkins-ci.org/display/JENKINS/Amazon+EC2+Plugin
* https://plugins.jenkins.io/ec2
* https://aws.amazon.com/getting-started/projects/setup-jenkins-build-server/
* https://aws.amazon.com/marketplace/pp/B00NNZUF3Q?qid=1508255204514&sr=0-2&ref_=srh_res_product_title
* https://d1.awsstatic.com/Projects/P5505030/aws-project_Jenkins-build-server.473cb2b76bb7b7018aa52b5b92bdeaaaa806aeee.pdf
* http://www.pugme.co.uk/index.php/2017/07/07/automating-the-jenkins-ec2-plugin-using-groovy/
* https://www.cloudbees.com/blog/setting-jenkins-ec2-slaves
* http://www.bogotobogo.com/DevOps/Jenkins/Jenkins_on_EC2_setting_up_master_slaves.php
* http://jmaitrehenry.ca/how-to-install-a-jenkins-master-that-spawn-slaves-on-demand-with-aws-ec2/
* https://dzone.com/articles/setting-up-jenkins-ec2-slaves
* http://artsy.github.io/blog/2012/07/10/on-demand-jenkins-slaves-with-amazon-ec2/
* https://www.atlantbh.com/blog/using-ec2-on-demand-with-jenkins/
* https://gist.github.com/vrivellino/97954495938e38421ba4504049fd44ea
* https://gist.github.com/mujina/d1e677a44571d6315b6337ae5bbe59a5
