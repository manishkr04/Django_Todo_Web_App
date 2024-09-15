Created CICD Pipeline to Django Web Application using Docker, Jenkins and AWS
-------------------------------------------------------------------------


Create an ec2 instance with amazon2 ami

Connect to the instance using any ssh connection tools

make a directory
# mkdir projects
# cd projects

Clone the code  from github
# git clone https://github.com/shreys7/django-todo.git


install Django
# pip3 install django
oR
# sudo apt install python3-django


Once you have downloaded django, go to the cloned repo directory and run the following command

# cd django-todo


```bash
$ python manage.py makemigrations
```

This will create all the migrations file (database migrations) required to run this App.

Now, to apply this migrations run the following command
```bash
$ python manage.py migrate
```

One last step and then our todo App will be live. We need to create an admin user to run this App. On the terminal, type the following command and provide username, password and email for the admin user
```bash
$ python manage.py createsuperuser
```

That was pretty simple, right? Now let's make the App live. We just need to start the server now and then we can start using our simple todo App. Start the server by following command

```bash
$ python manage.py runserver 
        OR
$ python manage.py runserver 0.0.0.0:8000
```

In the security group of Instance you have to open the port 8000 for every Outside IP 

Now Once you will try to access the web app you will get and ERROR
- Invalid HTTP_HOST header: '13.235.114.45:8000'. You may need to add '13.235.114.45' to ALLOWED_HOSTS. 

For this you have to Edit settings.py and add HOST'S IP in OR * i.e even if the IP changes it will be allowed- ALLOWED_HOSTS = ['*']
Once this is done You can refresh the page and you will be able to access the web application 


with this method your terminal will be engaged so if you want to run this in Backgroud 
1. Is you can you pm2 Process management 
2. #nohup python3 manage.py runserver 0.0.0.0:8000 &
    just by running this command the app will run in background and you will have the access to terminal too.

```bash
$ lsof -i:8000 
```  
// command to check what are the process    running on this port number

```bash 
$ kill -9 1306(PID)
```


Now Install Docker 
In Amazon linux 2
```bash
# sudo amazon-linux-extras install docker
```
In Ubuntu
```bash

$ sudo apt install docker.io
$ sudo service docker start
```
Add the ubuntu user to the docker group so that you can run Docker commands without using sudo.
```bash
sudo usermod -a -G docker ubuntu
```


now create a Docker file in Project Directory Itself
```bash
$ vi Dockerfile
    FROM python:3
    RUN pip install django==3.2

    COPY . .

    RUN python manage.py migrate

    CMD ["python","manage.py","runserver","0.0.0.0:8000"]
```

Build the docker Image 
```bash
$ docker build .  -t todoapp
```


Check if there is any running container 
```bash
$ docker ps
```

Run the container 
```bash
docker run -p 8000:8000 todoapp
     OR
docker run -d -p 8000:8000 todoapp  //demon to run in background     
nohup docker run -p 8000:8000 todoapp &  //to run in background
```

some more docker commands
```bash
sudo docker kill Container ID
sudo docker rm container ID
```

Now Install Jenkins,
Follow theses steps inorder to Install Jenkins in Ubuntu (AWS EC2 )
Update the Repository 
Install OpenJDK and check if it is installed 

```bash
$ sudo apt update
$ sudo apt install openjdk-11-jre
java --version
```

Next is to install Jenkins, For that You can also refer to official Jenkins Documentation of Command listed below

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```


Installing # Jenkins using Docker 

```bash
$ sudo docker pull jenkins/jenkins
$ sudo docker run  -d -p 8080:8080 docker.io/jenkins/jenkins:latest
$ docker exec <container_name_or_id> cat /var/jenkins_home/secrets/initialAdminPassword


```
so With the help of Just Two command In Docker we are able to Install Jenkins 

Open Port 8080 in security Group of EC2 instance 

Now http://PublicIP:8080/  using this you will be able to acces Jenkins 



Now Setup jenkins Following some steps

    ```bash
    cat /var/lib/jenkins/secrets/initialAdminPassword
    ```

    copy the Authentication Password and paste it to Jenkins page

    Click On Install Suggested Plugins 

    Create First Admin User 

Terms:
  Node: Node is a environment which allows you to create a pipeline.

Set Up an agent
Enter the Node Name 
Type
  Permanent Agent 
Click on Create

Further COnfigure some mode details
Description
  - 
Number of executors
  - 1
Remote root directory
  - /home/ubuntu
Labels
  - Todo_dev
Usage
  Select - Use this node as much as possible 
Lunch Method
  Select - Launch agent by connecting to the controller
Availability
  select - Keep this agent online as much as possible 
Node Properties
  - Leave this uncheck
Save

Go to Dashboard
Create a job
Description 
  - This job is to Automate Deployment Todo Web App

We are only demonstrating Continuous Deployment so for now leave uncheck Github Project
Source Code Management 
    - None
  
Build Triggers
  - Leave Unchecked
Build Environment 
  - Leave Unchecked

Build Steps
    Select - Execute Shell
  
Jenkins would not have the permission to  run all the commands on this project Directory so Before u list the command for the job give permission to all the user who can run the commands

On Ec2 Ubuntu
```bash
  ubuntu@ip-192-168-3-223:~/project$ chmod 777 django-todo/
  ```

List the command here
```bash
cd /home/ubuntu/projects/django-todo
sudo docker build . -t todo-dev
sudo docker run -d -p 8000:8000 todo dev
```
save 

Now CLick on Build 
Therefore Your Jenkins Todo Pipeline is ready


----------------------------------------------------

Make sure the proper permission is given to all the users 
1- Edit this file and add the some line to it
```bash
  $ sudo visudo
    ubuntu  ALL=(ALL) NOPASSWD:/usr/bin/docker
    jenkins ALL=(ALL) NOPASSWD:/usr/bin/docker

2 - Add Jenkins and Ubuntu user to Docker Group

    sudo usermod -aG docker jenkins
    sudo usermod -aG docker ubuntu

3 - chmod 755 ubuntu 

4 - chmod 777 django-todo/

```

After Building I encountered this error

```bash
+ cd /home/ubuntu/project/django-todo
/tmp/jenkins8459013625774312786.sh: 2: cd: can't cd to /home/ubuntu/project/django-todo
Build step 'Execute shell' marked build as failure
```
So the Solution to the above error is : 

Well the problem was the permission of the ubuntu directory. The home and project directory had the 755 permission but the ubuntu directory had 400. Used

chmod 755 ubuntu

------------------------------------------------------------------


Now there is a scenario that So changes in the code is made by the developer So how will you reflect that change in running Web application 

First of all kill the Process that is running 
```bash
$ sudo docker kill <container ID>
```

now go to jenkins job u have created lately just click on Play Button( Build Scheduled)






****************************************************************

DevOps Project 
  Personal Access Token
  Jenkins and Github Integration 
  CI/CD Pipeline


Create GitHub Personal Access Token
  Click on Profile Icon on Top-Right
  Go to Setting
  Go to Developer Settings
  Click on Personal Access Tokens
    Tokens (Classic)
    click to Generate new Token > Classic
    Check on first Two Box
    Click to Generate Token
    Copy the Token ro save to some safer place or you will not be able to access it afterwards


Create a repository to Push the Code that is running locally including Docker File

Create Repository
Copy the URL 
  https://github.com/manishkr04/Django_Todo_Web_App.git
Point the git remote to Repository URL you have Created
  ```bash
    $ git remote -v // check where git remote is pointing currently

    $ git remote set-url origin https://github.com/manishkr04/Django_Todo_Web_App.git
  
    $ git status

    $ sudo git add .

    $ sudo git commit -m "Added Server Code"    

    $ sudo git push origin develop
      Enter the Username and Password (Personal Access Token)
  
  ```


Jenkins and GitHub Integrations
  Login To Jenkins 
  Dashboard
  Manage Jenkins
  System Configuration
    System 
      GitHub
        GitHub Server
          Credentials > Add  > Kind ( Secret Text)> Secret (Githun Personal Access Token) > Add > ID (Jenkins-github-cicd)
          select the created Credentials
    Save and Apply



----------------------------------------------------------------
Docker Compose is a tool that simplifies the management of multi-container Docker applications by defining them in a single docker-compose.yml file. Instead of starting multiple containers manually, Docker Compose lets you define services (containers), networks, and volumes in a YAML file and start them with one command.

For example, in a typical web application, you might need a web server, database, and caching system. With Docker Compose, you can define each service in docker-compose.yml:


```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example

```
Running docker-compose up will start both services. It simplifies orchestration, especially in development environments.



Installing Docker compose

```bash
$ sudo apt install docker-compose
$ docker-compose -v
```

create a Docker Compose File
```bash
$ vi docker-compose.yml
``` 
```yaml
version : "3.3"
services :
  web :
    build : .
    ports :
        - "8000:8000"
```

Check if the writter docker-compose file is correct
```bash
$ docker-compose config
```

Now Run that docker compose file to Run your application 
```bash
$ sudo docker-compose up
$ sudo docker-compose down
or
$ sudo docker stop container_id
```


Now Integrating Docker Compose and Jenkins

Firstly push the docker-compose.yml file to Git hub as githun and jenkins is already integrated done in earliar project

```bash
$ git status
$ git add docker-compose.yml
$ git commit -m "messege"
$ git pull origin develop
$ git push origin develop
```
After this go to jenkins


add this command to Build Steps in Execute Shell
```bash
sudo docker-compose down
sudo docker-compose up -d --force-recreate --no-deps --build web
```






```bash
sudo yum update
sudo yum install -y python3
sudo yum install httpd-devel
sudo yum install -y mod_wsgi
cd /etc/httpd/modules     (verify that mod_wsgi is there)
cd /var/www/
sudo mkdir myApp
sudo chown ec2-user myApp
cd myApp
sudo pip3 install virtualenv
virtualenv myprojectenv
source myprojectenv/bin/activate
sudo pip3 install django==2.1.1
django-admin startproject myApp
cd myApp
python manage.py migrate
python manage.py runserver
wget http://127.0.0.1:8000/      (works correctly as it should and I receive test page)
python manage.py startapp hello
```