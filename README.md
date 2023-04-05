How to Deploy multiple application Containers using Docker Compose On Amazon EC2


What is Docker Compose?

Docker Compose is a tool that was developed to help define and share multi-container applications. With Compose, we can create a YAML file
to define the services and with a single command, can spin everything up or tear it all down.

Docker Compose is used for running multiple containers as a single service. Each of the containers here runs in isolation but can interact with each other when required.

For example:

If you have an application that requires an NGINX server and a Redis database, you can create a Docker Compose file that can run both containers as a service without the need to start each one separately.

In this blog, we  will learn


How to set up an Amazon EC2 Instance
Install Docker and Docker Compose on EC2 Instance
How to deploy multiple applications using Docker compose


Prerequisites
To follow along, you must have the following:
An AWS account
Basic knowledge of the terminal.
Some experience with Docker and Docker Compose.


Step 1: Setting up EC2 Instance:
     We will first create an AWS Instance (Amazon Linux) free tier eligible using the AWS console.

Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
Choose Launch Instance.

Once the Launch an instance window opens, provide the name of your EC2 Instance:
         
   4. Choose the Amazon Linux 2023 AMI
       
5. Choose an Instance Type. Select t2.micro for our use case which is also free-tier eligible.

6. Select an already existing key pair or create a new key pair. In my case, I will select an existing key pair.



7. Edit the Network Settings, Create a new Security Group, and allow ssh traffic, HTTPS, and HTTP from anywhere (later on we can modify the rules)



8. Leave the rest of the options as default and click on the Launch instance button:


9. On the screen you can see a success message after the successful creation of the EC2 instance, click on Connect to instance button:







10. Now connect to instance wizard will open, go to SSH client tab and copy the provided chmod and SSH command:


11. Open the Command Prompt or PowerShell in your local machine and paste the following two commands and you will be able to access your EC2 machine:



Step 2: Installing Docker and Docker Compose:


Apply pending updates using the yum command:
                           $ sudo yum update
     2. Search for the Docker package:
                          $ sudo yum search docker	
     3. Get version information:
                       $sudo yum info docker


Output:


4. To install Docker, run
                  $ sudo yum install docker



5. Add group membership for the default ec2-user so you can run all docker commands without using the sudo command:
                        $ sudo usermod -a -G docker ec2-user
6. Enable docker service at AMI boot time
                        $ sudo systemctl enable docker. service
7. Start the Docker service:
                        $ sudo systemctl start docker. service
8. To verify the docker service status on your AMI instance, run
                        $ sudo systemctl status docker.service


9. To download and install Compose standalone, run:
   
sudo curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose



10. Apply executable permissions to the standalone binary in the target path for the installation.
                $ sudo chmod +x /usr/local/bin/docker-compose

11. Verify both docker-compose and docker on your AWS Linux AMI:
              


Step 3:Deploying a multi-container application using Docker Compose on Amazon EC2
We will first clone the multi-container application code from the GitHub repo on our EC2 instance with the following command:

                 $ git clone https://github.com/mudasirhaji/multi-container-application-using-Docker-Compose-on-Amazon-EC2. git

After cloning the project, run the following command to change into the multi-container-using-Docker-Compose-on-Amazon-EC2 directory where the compose.yaml file is
                $ cd multi-container-application-using-Docker-Compose-on-Amazon-EC2/

Once in the project directory, start the containers using compose.yaml file and the following command:
                 $ docker-compose up -d
As soon you run the above command, Docker will start pulling the images and creates the network for the containers to communicate with each other, as shown in the screenshots below:


And this:


To confirm that both WordPress and db containers are running, run $ docker ps  and you should see an output similar to the image below.




We can also confirm the availability of two separate volumes by running the $ docker volume ls command as shown below: 




Step 3:Testing your deployment
The application we just deployed talks on port 8081 as mentioned in our compose.yaml file but if you visit port 8081 with the public address of your EC2 instance, you won’t be able to view it. This is because of the security group settings which currently only allow inbound traffic from SSH, HTTP, and HTTPS:
So to access our application from our browser, go to the EC2 instance console, with your EC2 instance selected, move to the Security tab and then click on the security group as in the image below.



After clicking on the security group, in the next menu, click on Edit Inbound rules to add the rule that allows inbound traffic through port 8081.


And then, add the rule as seen in the screenshot below.



With that done, when you visit your_instance_public_ip:8081 you should see the WordPress Installation page as in the image below.

You can go ahead with the configuration of the WordPress website and fill in all the required information and click on Install WordPress as shown below:

After Installing the WordPress you will be presented with the login screen and go ahead and log in.
After successful login will be on the home page of WordPress and now let's create our first post.


In our first post upload any image of your choice and put the title of the post of your choice and should look something like this:


This post will be stored as an actual image that you used in the Container Database while in the host machine, it will be stored as its metadata and both are required for this WordPress website to be fully functional.
Now we will demonstrate that if we destroy both of our containers the data will live on past the lifecycle of these containers.

Let's go to our EC2 machine terminal and issue the below command:
                           $ docker-compose down


Output:


As you can see in the above output both the containers have been removed.
If we check the $ docker ps command it will show we have got no running containers: 


However, if we check the $ docker volume ls command it will show both our named volumes:


Now if we create two new containers and set them to use these named volumes then the containers will use the data contained in these volumes which contains the WordPress Post created earlier. Let's test with the docker-compose up -d command which will utilize the images which we have stored locally, in addition, it will utilize the existing name volumes and won’t create new ones.

As you can notice from the output that this time around the deployment was very fast.
Now if we go back to the browser and refresh the page we would see the same post. This is only possible to see as we have utilized the named volumes which persist for the lifetime of the existing containers.

Conclusion
In this blog, you learned how to deploy multi-container applications using docker-compose on Amazon EC2 Instance.

Hope you would like this!


Thanks





