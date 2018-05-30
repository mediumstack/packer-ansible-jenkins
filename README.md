# packer-ansible-jenkins

A packer project to pack an AMI based on CentOS 7 using ansible, with jenkins installed natively along with Docker & awscli.

## Getting Started

Export your AWS environment variables (if they aren't already in .aws), make sure to add a space before the command so it doesn't show up in history.

```
 export AWS_ACCESS_KEY_ID=
 export AWS_SECRET_ACCESS_KEY=
```

Fork the below repo if you would like to make any changes to the playbook
```
https://github.com/maxstack/ansible-playbook-jenkins.git
```

Edit jenkins-packer-ec2.json and change the region / source AMI if required.

Next validate your file then build!
```
packer validate jenkins-packer-ec2.json
packer build jenkins-packer-ec2.json
```

To add the build job to jenkins you can use the following command
```
cat _jenkins/packer-ansible-jenkins-pipeline.xml | java -jar /opt/jenkins-cli.jar -s http://admin:somecomplexpassword@localhost:8080 create-job jenkins-packer-pipeline
```
