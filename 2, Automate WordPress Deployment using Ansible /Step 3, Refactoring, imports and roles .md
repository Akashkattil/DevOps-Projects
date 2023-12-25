# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project we will continue working with ansible-config-mgt repository and make some improvements of  code. Now we need 
to refactor  Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use 
previously created playbooks in a new playbook – it allows we to organize  tasks and reuse them when needed.

### Code Refactoring
Refactoring ( https://en.wikipedia.org/wiki/Code_refactoring ) is a general term in computer programming. It means making changes to 
the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, 
increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

In  case, we will move things around a little bit in the code, but the overal state of the infrastructure remains the same.

Let us see how we can improve  Ansible code!

Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory
which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each
subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require 
Copy Artifact ( https://plugins.jenkins.io/copyartifact/ ) plugin.

1. Go to  Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts 
after each build.

```
sudo mkdir /home/ubuntu/ansible-config-artifact
```

2. Change permissions to this directory, so Jenkins could save files there –  
```
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this 
plugin without restarting Jenkins

4. Create a new Freestyle project (we have done it in Project 9) and name it save_artifacts.

5. This project will be triggered by completion of  existing ansible project. Configure it accordingly:

Note: we can configure number of builds to keep in order to save space on the server, for example, we might want to keep only last
2 or 5 build results. we can also make this change to  ansible job.

6. The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, 
create a Build step and choose Copy artifacts from other project, specify ansible as a source project and 
/home/ubuntu/ansible-config-artifact as a target directory.

7. Test  set up by making some change in README.MD file inside  ansible-config-mgt repository (right inside master branch).
If both Jenkins jobs have completed one after another – we shall see  files inside /home/ubuntu/ansible-config-artifact directory
and it will be updated with every commit to  master branch.

Now  Jenkins pipeline is more neat and clean.

#
