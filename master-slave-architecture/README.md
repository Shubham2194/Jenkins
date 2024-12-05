                                      Jenkins Master Slave Architecture


![image](https://github.com/user-attachments/assets/5fbebf00-4c7f-4e45-9cea-473045c96c25)

Step 1: 
**Create two VM on Cloud with linux Image (AWS , GCP , Azure)**

Step 2: 
**Rename One of them as Jenkins-master and another as Jenkins-slave**

Step 3:
**Install Java on the master as well as on slave VM**

-  As Jenkins is written in Java, installing Java in the system is mandatory.
  
```sh
sudo apt update -y
sudo apt install default-jre -y
java -version
```

Step 4:
**Install Jenkins on VM (Jenkins-master)**

```sh
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl status jenkins.service 
```

![image](https://github.com/user-attachments/assets/e5e03047-0d0c-49a3-aac6-71c2b4afa565)


Step5: 
**SSH to slave VM and Create ssh-keys to Select the Launch agent via SSH**

![image](https://github.com/user-attachments/assets/ebcc63b4-c17f-47e6-ac16-1295e57374e9)

```sh
ssh-keygen -t ed25519 -C "you-grmail@gmail.com"
```
This will Create SSH-Keys inside ~/.ssh folder

```sh
cat ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub 
```

Step 6: 
**Copy id_ed25519.pub file and paste in authorized_keys**

```sh
cat id_ed25519.pub >> authorized_keys
```

Step 7: 
**Acces jenkins on VM ip and port**

Note : Make sure to Open 8080 port from security group ( i have configured nginx and add proxy pass as 8080 and mapped with subdomain)

![image](https://github.com/user-attachments/assets/e6b54a58-85e2-43b3-8419-a81a3751cd4b)

(Get the Jenkins admin password from - sudo cat /var/lib/jenkins/secrets/initialAdminPassword)



Step 8:
**Login to Jenkins in the Master node, then in Jenkins Dashboard go to set up an agent**


![image](https://github.com/user-attachments/assets/0302d65f-0a5b-4bc6-92b8-22b333f31ba2)

Step 9:
**Enter the name of the agent and click create**


![image](https://github.com/user-attachments/assets/9c65fa49-2602-42d0-b295-27cc8d5569cb)


Step 10:
**After clicking on the button Create, The form will open for configurations**

![image](https://github.com/user-attachments/assets/e600f2a8-d921-4c45-9fad-a143d6e5a4ee)

Name: The name of our agent.

Description: The description of your Agent.

Remote root directory: Location on the agent machine where Jenkins will store and execute its job workspace

Label: For categorizing and grouping agents.



![image](https://github.com/user-attachments/assets/8b37c79c-86d9-4f72-8fc8-7153d0e34e9f)


Launch Method: Select the Launch agent via SSH.

Host: IP address of the Worker node.

Credentials: Paste the private key as the credentials.

Save the Configurations.

Click on Launch the agent.

Note: You need to create credentials by clicking on Add > jenkins > choose SSH username with Private Key.

Enter username by checking $whoami on slave node and in Private key copy the id_ed25519 and paste here and save.


![image](https://github.com/user-attachments/assets/f4f3b850-72dc-415b-bb48-cb34c7025fd1)

![image](https://github.com/user-attachments/assets/22a466fe-d4a4-4b39-a2b8-2ab5b8c76ad7)



Step 11:
Go to Log of your node and you can see :

![image](https://github.com/user-attachments/assets/8debb545-91d3-4866-b838-dbda8b136770)

Step 12:

***To deploy resources on this agent we need to use agent (i have added dev-docker in my agent configuration):***

```sh

pipeline {
    agent {
        label "dev-docker"
    }    
    environment {
        GIT_PROJECT = "https://github.com/repo/@.git"  
    }
    
    stages {
        stage('Pull Code') {
            steps {
                git branch: "${params.Branch}", credentialsId: 'github-cred', url: GIT_PROJECT
                
            }
        }
        stage('Build Docker') {
            steps {
                script{
                    sh "docker-compose -f docker-compose-algo.yaml up -d --build"
                }
            }
        }
    }
}
```

**We Have succcessfully Setup master-slave architecture !!**
