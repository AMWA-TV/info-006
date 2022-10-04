

{:.no_toc}

- This will be replaced with a table of contents
{:toc}

## Introduction

This section showcases practical examples on the journey to implementing MS-05 / IS-12 in a device and as such provides a practical example of how to implement an NMOS Control Framework Device on a Linux system using the NMOS Control Mock Device.

The HOWTO is meant to provide a relatively simple, easy-to-follow "recipe" for getting a controllable NMOS Node up and running in a short period of time.
It is not intended to be a tutorial article, and therefore, it excludes explanation regarding the NMOS Control Framework. The reader is referred to the tutorial section for more depth coverage of the framework.

The reader is assumed to have some experience with an NMOS infrastructure including IS-04 and NMOS Registration and Discovery (RDS).  Also some experience in javascript and/or typescript will be helpful although not entirely required.  

We hope you find this how-to guide to be useful.

## HOWTO Steps

This HOWTO will present the steps to install, modify and try out the NMOS Control Framework. The HOW-TO uses a fresh Ubuntu 20.04 installation.  

### Basic Installation

This section will do the most basic steps to get a NMOS Controllable Node running on your system. 

- Install the NMOS Controllable Mock Device (NCMD)
- Install EasyNMOS Docker Container for NMOS RDS
- Verify NCMD registers and exposes it's NMOS IS-12 Control Endpoint in the NMOS RDS
- Install Chrome Websocket Extension
- Verify the NCMD can be reached via the WebSocket Control endpoint listed in the RDS
- Add subscription for notification on change to control parameter
- Verify notification event received when parameter changes

### Modifications to Basic Installation

This section will make modifications to the basic system and show how to add in a parameter to one of the controls provided by the mock node.

- Modify Mock Device to add in extra read/write parameter to existing control 
- Verify extra parameter can be modified via websocket
- Add subscription notification to the new parameter
- Modify the parameter and verify notification received

### Addition of Custom Control

This section will make modifications to the basic system and show how to add in a new control to the mock node.  It will use a realistic control point that will check the status of network connections and return some statistics about these interfaces. It will also allow clearing the packet counters on the interfaces.

- Add a new control to the mock node
1.Create WebIDL
2.Create implementation based on WebIDL 
3.Extend protocol to connect to the control implementation
- Verify the control can be seen using Chrome Websocket Plugin
- Verify values are correct for interface statistics
- Clear the counters on the control
- Verify statistics provided by the new controller are correct


## Installing Mock NMOS Control Device

Install the NMOS Mock control device from its location on the public github [repo](https://github.com/AMWA-TV/nmos-device-control-mock).  

Follow the steps below from any directory on your Ubuntu host:
### Install Needed Packages

You will need `docker` for running the NMOS RDS and `npm` for running the mock NMOS Controllable Node.  We first assume a fresh install of Ubuntu 20.04 and so do an update for packages prior to installation of required dependencies.

```
sudo apt-get update
sudo apt install -y npm
sudo apt install -y docker.io

```

Now install an NMOS RDS. We will use the RDS from [EasyNMOS](https://github.com/rhastie/easy-nmos)

You will run the RDS from one terminal window and the mock node from another.  Open a terminal window and perform the following commands:

```
sudo docker pull rhastie/nmos-cpp:latest
```

**Expected Output**
```
latest: Pulling from rhastie/nmos-cpp
3b65ec22a9e9: Pull complete 
964e9f4b2501: Pull complete 
5312be12420b: Pull complete 
037321a10163: Pull complete 
4f4fb700ef54: Pull complete 
Digest: sha256:bd2cdeb5263d555cfe0e427099251d287f3f343af09789e82354deb3049d4a2d
Status: Downloaded newer image for rhastie/nmos-cpp:latest
docker.io/rhastie/nmos-cpp:latest

```
**Run and Verify the RDS**

```
sudo docker run -d -p 80:8010 -p 8011:8011 --name RDS  rhastie/nmos-cpp:latest
sudo docker ps
```

**Expected Output**
```
CONTAINER ID   IMAGE                     COMMAND                 CREATED      STATUS      PORTS                                                                                                                   NAMES
cca7720881dd   rhastie/nmos-cpp:latest   "/home/entrypoint.sh"   2 days ago   Up 2 days   1883/tcp, 11000-11001/tcp, 5353/udp, 0.0.0.0:8011->8011/tcp, :::8011->8011/tcp, 0.0.0.0:80->8010/tcp, :::80->8010/tcp   RDS

```

Now install the mock device from the repo:

``` 
git clone https://github.com/AMWA-TV/nmos-device-control-mock.git
git checkout release-1.0.0
cd nmos-device-control-mock/code
npm install
npm run build-and-start

```

## Recap of HOWTO

This HOWTO has shown how to add controllability to an NMOS node using the NMOS Control Framework. ...



