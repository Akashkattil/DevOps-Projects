## Install and configure ansible on ec2 instance.
1. Update Name tag on  Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
2. In  GitHub account create a new repository and name it ansible-config-mgt.
3. Instal Ansible
  ``` 
sudo apt update
sudo apt install ansible
```
Check  Ansible version by running ansible --version
4. Configure Jenkins build job to save  repository content every time we change it – this will solidify  Jenkins configuration skills acquired in Project 9.
- Create a new Freestyle project ansible in Jenkins and point it to  ‘ansible-config-mgt’ repository.
- Configure Webhook in GitHub and set webhook to trigger ansible build.
- Configure a Post-build job to save all (**) files, like we did it in Project 9.
5. Test  setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
```
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
Note: Trigger Jenkins project execution only for /main (master) branch.
 Every time we stop/start  Jenkins-Ansible server – we have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to  Jenkins-Ansible server. Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once we terminate  EC2 Instance.

- Prepare  development environment using Visual Studio Code

First part of ‘DevOps’ is ‘Dev’, which means we will require to write some codes and we shall have proper tools that will make  coding and debugging comfortable – we need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, we can choose whichever we are comfortable with, but we recommend one free and universal editor that will fully satisfy  needs – Visual Studio Code (VSC), we can get it here.

After we have successfully installed VSC, configure it to connect to  newly created GitHub repository.

Clone down  ansible-config-mgt repo to  Jenkins-Ansible instance
```
git clone <ansible-config-mgt repo link>
```





