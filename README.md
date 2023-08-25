## Installation
To follow along with the instruction after the event has ended, you can follow the official instructions to install Minikube and Helm on your local machine. This repository also contains Ansible code to deploy an AWS instance just like the one used in the workshop.

### Local Installation


### AWS Installation
* To test your connections and make sure you can connect to the AWS Instances, run:
  * `ansible -m ping -i ./tracydevs.aws_ec2.yml aws_ec2`
* Then to install everything, run `ansible-playbook workshop_environment.yml`