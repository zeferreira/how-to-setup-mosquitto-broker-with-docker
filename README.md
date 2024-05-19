# How to setup Mosquitto MQTT Broker using Docker Compose for development

These is a configuration for a developement environment. Don't use it for production environment. 
It works with for Windows and Linux Environments (tested with windows).

The scenario is simple, the mosquitto broker will start without peristence and with basic authentication for one user (or more).
After it, we will test the local environment with common mtqq clients and tools (you can choose witch one you want to use).

## The test scenario:
 - One client will Subscribe the topic home/bedroom/temperature. 
 - One Client will Publish one message with the value (42 - meaning of the life) to the same topic.

## Obs.:
** I tested it on Windows running without WSL. **
** I tested some commands running directly on conteiners (mosquitto_sup/pub) and outside the container (curl and MQTT Explorer). **
** In some cases I will add notes about the tools. **
<!-- ** I wrote one little C# application to test the flow with the broker. Not yet, but I'm working on it' -->
** I'm using **Visual Studio Code (VScode)** because I like to see the highlight sintax of the files and I already have it on my machine, but you can use whatever you want to **create the configuration/password files** . Choose one text editor make the samples for shell commands more simple when I created this file. For god sake, give a break. It's just a text editor.

You can download it [here](https://code.visualstudio.com/download) from the oficial website. 
Teach about how to install the VSCode is out of the scope of this document.

## 1. Docker environment

Docker installation is out of the scope of this document, as well as the commands and it's sintax. Please, check the links below if you need to understand bacic concepts about docker or if you don't have Docker on your machine.

### Docker Desktop / Docker Compose 

Latest intructions are [here] (https://docs.docker.com/desktop/) on docker website.

Latest intructions are [here] (https://docs.docker.com/compose/) on docker website.

**Make sure that you understande what is a container, how to run, restart or delete a container and what is a docker compose file and how to start/stop it. You also need to understand the sintax for docker compose files, the relation between the folders on your host machine and the path that you are going to provide inside the compose file. 

It's also important the image that we are going to use and you always need to check his information on docker hub. Do your homework.

If you don't know the basic concepts about docker compose files check it [here](https://docs.docker.com/compose/gettingstarted/). 

### Windows Install

Latest instructions are [here](https://docs.docker.com/desktop/install/windows-install/) on docker website.

### Linux Install

Latest instructions are [here](https://docs.docker.com/desktop/install/linux-install/) on docker website.
  
## Mosquitto Broker

I supose that you know what is MQTT protocol, a Broker and (more specificaly) Mosquitto Broker. But, if you don't know it (really?) these links below will help you:

**Introduction about MQTT Protocol [here] (https://mqtt.org/getting-started/) on mqtt.org website.

**Main page of Eclipse Mosquitto [here] (https://mosquitto.org/) on mosquitto.org website.

** What is a Message Broker [here](https://www.ibm.com/topics/message-brokers#:~:text=A%20message%20broker%20is%20software,messages%20between%20formal%20messaging%20protocols.) by IBM.

Now is the time to setup the envirmonment for the container.


## 2. Create the folder for Mosquitto configuration files

### Windows 

** Windows Commmand Shel

```shell

md mosquitto
cd mosquitto

# for storing mosquitto.conf and pwfile (for username/password)
md config 

```

### Linux

** Linux Commmand Shel

```bash

mkdir mosquitto
cd mosquitto

# for storing mosquitto.conf and pwfile (for username/password)
mkdir config

```

** Pay attention to this folders, you will use it inside the docker compose to expose the files with the initial configuration.

## 3. Create Mosquitto configuration file - mosquitto.conf

Mosquitto has a configuration file named **mosquitto.conf** and you need to create the file before you test our scenario (or you can change it later and restart it, just in case of a mistake). You can use this file to choose the service configurations for your broker. In our case, we will choose the ports that will listem to the clients, if we accept anonymous or authenticated and the username and password to access our server (remember our scenario).

If you want to know more about Mosquitto configuration options check the [here](https://mosquitto.org/man/mosquitto-conf-5.html) on the official website.  

Make sure that you are creating the file inside the **config folder** that we created:

### Windows / Linux

** Windows Commmand Shel

```shell
# just if you are not on the right folder 
cd mosquitto
cd config 
#create the empty configuration file and open it on VSCode 
code mosquitto.conf

```

Copy and past the content below inside the file. If you read the section about the configuration options, it's basically self explanatory. We are bascally telling the server that we don't want to accept connections from unauthenticated clients, the path to the file what will store usernames and passwords, ports that we want to use to listen for new connections and a protocol that we accept.

Basic configuration file content 
```
allow_anonymous false
password_file /etc/mosquitto/config/passwd
listener 1883
listener 9001
protocol websockets

```

## 4. Create the empty Mosquitto password file - pwfile

### Windows / Linux

```shell
# just if you are not on the right folder 
cd mosquitto
cd config 
#create the empty password file and open it on VSCode 
code pwfile
```

**Yes, the file is empty. Don't worry little padawan. The force will be with you. Save it empty and go ahead.


## 5. Create docker-compose file called 'docker-compose.yml'

Now, we have the folder structure to receive Mosquitto information and the basic files to setup the server and the users. (mosquitto.conf and pwfile, both files inside the config folder.). We also have installed the docker and docker compose and we understand the basic concept behind it. 

We also have one single text editor (VSCode) that keeps open each single file the we created, so it's easy to create another one and test it.

Keep it in mind and let's create the file.

### Windows / Linux

```shell
# just if you are not on the right folder 
cd mosquitto #yes, just mosquitto, not config.

#create the empty docker compose file and open it on VSCode 
code docker-compose.yml
```
### Content of Docker compose file

Copy and paste the content below into your **docker-compose.yml** file. The file is very simple, we are just using the broker's official image, exposing the protocol ports (these numbers are standard for MQTT) and mapping the folder/file structure that we created previously.

Some details you could pay attention to:
- We exposed the same ports that we configured inside the configuration file. Without it, well, you know, you have no 'doors', right? With it, the container will accept connections on these ports and the service inside the container will also accept connections on the same ports. If you are running other containers on your docker, you need to check if some is already using these ports.

- We don't have persistence. Inside our mosquitto configuration file, we don't have a session with the configuration for persistence, you don't have information stored on your disk about the content of the messages. No logs, no storage files. If you want to enable persistence for your test server, take a look [here](https://mosquitto.org/man/mosquitto-conf-5.html). Seach for the 'persistence' tag and study the options that you need.

You must create the structure for storage folders and files and change your configuration file to use the same structure as well as your docker compose to expose the folders for your container.

- The structure of folders and files correspond to the structure we created previously.

```yml

version: "3.7"

services:
  mosquitto:
    image: eclipse-mosquitto
    hostname: mosquitto
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto:/etc/mosquitto
      - ./mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf

```

## 6. Create and run docker container with mosquitto broker

### Windows 

```shell
# just if you are not on the right folder 
cd mosquitto #yes, just mosquitto, not config.

# Run the docker container 
docker docker-compose up mosquitto

```
### Linux

```bash
# just if you are not on the right folder 
cd mosquitto #yes, just mosquitto, not config.

# Run the docker container 
sudo docker docker-compose up mosquitto

```

### Check if the container is up 

### Windows 

```shell

docker ps

```

### Linux


```bash

sudo docker ps

```
You must see status Up :)


## 7. Let's create our user in the pwfile to access our server 

As you know, our use case has a server with authentication. So we need to choose the username and password for our mosquitto server.
We need to use mosquitto_passwd tool for it. Don't worry, Mosquitto comes with a set of tools and one of them is mosquitto_passwd.
To use it, we need to access the container to get access to the tool or just execute the it using docker exec with the right arguments:

My advice is simple, if you want to make a set of changes inside your container, connect to it and go ahead. If you just want to change something specific, do it with docker exec. 

Now, remember that the pwfile is empty, so we need to add users to our configuration file using the command:

### Windows 

```shell
docker exec mosquitto mosquitto_passwd -b /etc/mosquitto/passwd guest guest

```

### Linux 

```shell
sudo docker exec mosquitto mosquitto_passwd -b /etc/mosquitto/passwd guest guest

```
In our case, we created the user guest with the guest password. If you want to add more than one user, it's a good opportunity to learn how to connect to the container. The idea is the same, you need to execute a program that will provide access to send commands to the container. The shell.

The nexte session will provide commands to connect to the container and create more users. You can skip this session if you want.

## X.1 Connecting to the container to create more users

This is simple, call the terminal (sh) using docker exec and use the same tool to create so many users you want.  

### Windows 

```shell

#calling SH (mosquitto is our container name, please, pay attention)
docker exec -it mosquitto sh

# now you don't need to use docker exec, just call the mosquitto tool to create the new user
mosquitto_passwd -b /etc/mosquitto/passwd username2 passw2

#repeat the process every time that you need to create one user
# type 'exit' when you finish

#restart the container (mosquitto is the name of the container)
docker restart mosquitto

```
## Linux 

For linux you basically need to add **sudo** for each docker command.

## 8. Testing the environment 

Ok, now you have everything that you need to start using the broker. It's time to test our setup using some tools (clients) and we are done. You can use it to develop your mqtt aplications. 

### A little bit about MQTT and the test use case

For our testing scenario, we basically want to send a message using one client and receive the same message using another client. Simple.

Communication is asynchronous. The message is sent to the broker and the broker delivers the message to the second program. One customer does not know the other.

The idea here is that you have a program "waiting" for new messages to be processed and the broker is a 'Carrier' that handles the senders for you. You could have more than one program listening to a topic or more than one device (with a program) sending messages to the topic. Topic is the structure you use to receive messages within the broker. 

Some people might think that it doesn't make sense to put another program in between the other, but in real scenarios, you could work with more than one protocol and your broker would separate your second program from the first. Your processing layer does not need to know all the different protocols than the device layer.

Most of the time you also need to implement Patterns to control the flow of events between services and the broker will provide the framework for this with abstraction for storage, concurrency, different protocols and it is also a good way to scale the architecture layer that hosts your events.

If you don't understand those lines above, you should take a look at the pub/sub pattern or take a look at mosquitto documentation [here](https://mosquitto.org/man/mqtt-7.html).


Remember the test scenario:
 - One client will Subscribe the topic home/bedroom/temperature. 
 - One Client will Publish one message with the value (42 - meaning of the life) to the same topic.

Our mosquitto image already have the basic tools (mosquitto_pub/mosquitto_sub) to test the broker you don't need to install them if you want to use it. 

### Let's start the program to "listen" the messages that we send to the topic. (subscriber)

Remember that we configurated our broker with authentication, so we need to call the program with the right credentials.

### Windows
```shell

# Subscribe to the topic with authentication
docker exec mosquitto mosquitto_sub -u guest -P guest -v -t 'home/bedroom/temperature'

```

### Linux 
```shell

# Subscribe to the topic with authentication
sudo docker exec mosquitto mosquitto_sub -u guest -P guest -v -t 'home/bedroom/temperature'

```
Keep the window open. 

### Let's start the program to "publish" the messages that we send to the topic. (publisher)

### Windows
```shell

# Subscribe to the topic with authentication
docker exec mosquitto mosquitto_pub -u guest -P guest  -t 'home/bedroom/temperature' -m '42'
```

### Linux 
```shell

# Subscribe to the topic with authentication
sudo docker exec mosquitto mosquitto_pub -u guest -P guest  -t 'home/bedroom/temperature' -m '42'

```

You should see: 'home/bedroom/temperature' '42' on the windows running the subscriber (mosquitto_sub).
Keep the window open and try new messages. 


## 9. That is it.

That is it. If you did every thing right you have a MQTT broker running over docker you can use it to develop your MQTT Applications.
Thanks for you attention and let me know if you have some suggestion for this document or if you found a error. If you want to send me something, open one issue or tag me on a discussion.

I really appreciate your time to read this document and hope it's usefull for you.

