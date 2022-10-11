

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

This section will do the most basic steps to get a mock NMOS Controllable Node running on your system. 

- Install the NMOS Controllable Mock Device (NCMD)
- Install EasyNMOS Docker Container for NMOS RDS
- Verify NCMD registers and exposes it's NMOS IS-12 Control Endpoint in the NMOS RDS
- Install Chrome Websocket Extension
- Verify the NCMD can be reached via the WebSocket Control endpoint listed in the RDS
- Locate one of the control classes provided by the NCMD
- Verify ability to read, and write a parameter of the control class instance.

### Modifications to Basic Installation

This section will make modifications to the basic system and show how to add in a new property to one of the control classes provided by the mock node.

- Modify Mock Device to add in a new read/write property to an existing control class.
- Verify that the new property can be modified using IS-12 via the WebSocket channel.
- Add subscription in order to receive notifications for any changes experienced by the new property.
- Modify the value of the new property and verify notification received

### Addition of a vendor specific Control Class

This section will make modifications to the basic system and show how to add in a new control class to the mock node.  The new control class will allow checking the status of network connections and return some statistics about these interfaces. It will also allow clearing the packet counters on the interfaces.

- Add a new control class to the mock node
1.Create WebIDL
2.Create implementation based on WebIDL 
3.Map and expose the new control class implementation via the IS-12 protocol
- Verify that we can discover the new control class instance using a Chrome Websocket Plugin
- Verify we can retrieve interface statistics from the properties exposed by the new control class
- Clear the counters on the new control class
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

The output of `sudo docker ps` will show the EasyNMOS NMOS docker image is up and running and required ports are mapped from the host's network to the docker container network. 

```
CONTAINER ID   IMAGE                     COMMAND                 CREATED         STATUS         PORTS                                                                                                                   NAMES
9a8890946623   rhastie/nmos-cpp:latest   "/home/entrypoint.sh"   9 seconds ago   Up 8 seconds   1883/tcp, 11000-11001/tcp, 5353/udp, 0.0.0.0:8011->8011/tcp, :::8011->8011/tcp, 0.0.0.0:80->8010/tcp, :::80->8010/tcp   RDS

```

Verify the web server in the EasyNMOS docker container is up and running by opening [http://localhost/admin](http://localhost/admin) on your host. You should see the welcome screen for the open-source NVIDIA NMOS Commissioning Controller. 
 
Now install the mock device from the repo:

``` 
git clone https://github.com/AMWA-TV/nmos-device-control-mock.git
cd nmos-device-control-mock/code
npm install
npm run build-and-start

```

**Expected Output**

The output of npm run build-and-start will show status as the mock node is built and ran.  The last few lines of the output will show the mock device registering with the RDS running as part of the docker image started above.


```
App started
Configuration: Reading config.json
Configuration - CheckIdentifiers()
Configuration- Writing back config.json
RegistrationClient - RegisterOrUpdateResource(resourceType:node)
Server started on port 8080
RegistrationClient - RegisterOrUpdateResource(resourceType:device)
RegistrationClient - RegisterOrUpdateResource(resourceType:receiver)
Successfully wrote file


```
**Locate the NMOS Control WebSocket**

You now have all the NMOS items needed to interact with the NMOS Control mock node. Since IS-12 uses a WebSocket control endpoint we will next browse the RDS registry to find the advertised WebSocket endpoint and use a Chrome extension that allows opening that WebSocket and sending and receiving IS-12 JSON formated commands and responses.  

Navigate in your preferred browser to the devices query location:
`http://127.0.0.1/x-nmos/query/v1.3/devices/`

**Expected output**

```
[
    {
        "controls": [
            {
                "href": "http://127.0.0.1:8080/x-nmos/connection/v1.1/",
                "type": "urn:x-nmos:control:sr-ctrl/v1.1"
            },
            {
                "href": "http://127.0.0.1:8080/x-nmos/connection/v1.0/",
                "type": "urn:x-nmos:control:sr-ctrl/v1.0"
            },
            {
                "href": "ws://127.0.0.1:8080/x-nmos/ncp/v1.0/connect",
                "type": "urn:x-nmos:control:ncp/v1.0"
            }
        ],
        "description": "NC-01 device",
        "id": "[7977373c-70f6-4e62-b713-3431f1ac4a2f](http://127.0.0.1/x-nmos/query/v1.3/devices/7977373c-70f6-4e62-b713-3431f1ac4a2f)",
        "label": "NC-01 device",
        "node_id": "[e0f9e1a3-2a2f-4f00-b02e-76d8286e1d98](http://127.0.0.1/x-nmos/query/v1.3/nodes/e0f9e1a3-2a2f-4f00-b02e-76d8286e1d98)",
        "receivers": [
            "[18eae0e9-dcf4-40ff-88a3-cb553993d1b8](http://127.0.0.1/x-nmos/query/v1.3/receivers/18eae0e9-dcf4-40ff-88a3-cb553993d1b8)"
        ],
        "senders": [],
        "tags": {},
        "type": "urn:x-nmos:device:generic",
        "version": "1665129531:00000000"
    }
]

```

In the controls section of the JSON response you will find:

`ws://127.0.0.1:8080/x-nmos/ncp/v1.0/connect urn:x-nmos:control:ncp/v1.0`


This is the WebSocket used to interact with NMOS Control components.

**Install Chrome WebSocket Plugin**

We will install a Chrome WebSocket extension. Other WebSocket clients will work equally well. 

Details on installing Chrome extensions can be found [here](https://support.google.com/chrome_webstore/answer/2664769?hl=en). Follow the instructions to open the Chrome Web Store and search for WebSocket King Client. Install this extension. 

After completing the installation of WebSocket King open the extension in a new browser window. Copy and paste the WebSocket located in the RDS for the NC-01 NMOS Control Mock node into the connections field and click `connect`. The `Connect` button should turn to `Disconnect` indicating a successful connection to the NC-01 WebSocket endpoint.

Next we will verify the ability to read and write to the NMOS Control components running on the mock node.  We will focus on reading and writing to the Stereo Gain Block and related objects provided by the mock node.  Other aspects of control can also be explored by following the examples in the [IS-12 Specification](https://specs.amwa.tv/is-12/) example section. For purposes of this HOW-TO we will focus on working with the Stereo Control and adding code to extend this control then create a new control and interact with this new control.  

**Open a Session and Obtain Information on Control of Interest**

Use the JSON Command to open a new session to the NC-01 WebSocket control. Copy the JSON formatted message into the payload area. For more information about the format refer to [AMWA IS-12 NMOS Control Protocol](https://specs.amwa.tv/is-12/). Also note that in an actual system most of the manual steps we are performing here would be performed by an NMOS Controller using the IS-12 specification. For more information about implementing an IS-12 NMOS Controller see the HOW-TO section for Controller implementations.


```
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
  "messages": [
    {
      "handle": 1,
      "arguments": {
        "heartBeatTime": 5000
      }
    }
  ]
}

```

**Expected Output from WebSocket King**

```
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 1,
      "result": {
        "status": 0,
        "value": 3
      }
    }
  ]
}

```

We now have a WebSocket session `3` open for our WebSocket King client. Next retrieve the members of the root block in order to identify the location of the Stereo gain block.

Send the following JSON Formatted command to the NC-01 WebSocket Use the Session ID received in the previous command. In our case `3`:

```

{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 3,
      "oid": 1,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 2,
          "index": 10
        }
      }
    }
  ]
}

```


**Expected Output**

The device responds with a JSON containing member descriptors for the root block. The sub block we are interested in is the Stereo Gain block. We see the block is present along with its Object ID (oid). The oid is unique across all control elements and we will use it to further interogate the Stereo Gain block and find its members. The oid is 31.

```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 101,
  "messages": [
    {
      "handle": 3,
      "result": {
        "status": 0,
        "value": [
          {
            "role": "DeviceManager",
            "oid": 2,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                3,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Device manager",
            "owner": 1,
            "description": "The device manager offers information about the product this device is representing",
            "constraints": null
          },
          {
            "role": "ClassManager",
            "oid": 3,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                3,
                2
              ],
              "version": "1.0.0"
            },
            "userLabel": "Class manager",
            "owner": 1,
            "description": "The class manager offers access to control class and data type descriptors",
            "constraints": null
          },
          {
            "role": "SubscriptionManager",
            "oid": 5,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                3,
                4
              ],
              "version": "1.0.0"
            },
            "userLabel": "Subscription manager",
            "owner": 1,
            "description": "The subscription manager offers the ability to subscribe to events on particular objects and properties",
            "constraints": null
          },
          {
            "role": "ReceiverMonitor_01",
            "oid": 11,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                2,
                2
              ],
              "version": "1.0.0"
            },
            "userLabel": "Receiver monitor 01",
            "owner": 1,
            "description": "Receiver monitor worker",
            "constraints": null
          },
          {
            "role": "stereo-gain",
            "oid": 31,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Stereo gain",
            "owner": 1,
            "description": "Stereo gain block",
            "constraints": null,
            "blockSpecId": null
          },
          {
            "role": "DemoClass",
            "oid": 111,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                2,
                0,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Demo class",
            "owner": 1,
            "description": "Demo control class",
            "constraints": null
          }
        ]
      }
    }
  ]
}
```

**Read, Write, Modify Stereo Gain**

We then make use of the generic Get method (1m1) to find the members (2p10) of the Stereo gain block (oid: 31).  Send the following JSON Formatted command to the NC-1 control WebSocket.

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 3,
      "oid": 31,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 2,
          "index": 10
        }
      }
    }
  ]
}

```
We use oid 31 which we discovered was the oid for Stereo Gain when interrogating the root block.

 **Expected Output**

The device responds to the above command with a JSON formatted response containing all the members of the Stereo Gain block.

 ```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 3,
      "result": {
        "status": 0,
        "value": [
          {
            "role": "channel-gain",
            "oid": 21,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Channel gain",
            "owner": 31,
            "description": "Channel gain block",
            "constraints": null,
            "blockSpecId": null
          },
          {
            "role": "master-gain",
            "oid": 24,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                2,
                1,
                1,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Master gain",
            "owner": 31,
            "description": "Master gain",
            "constraints": null
          }
        ]
      }
    }
  ]
}

 ```

Next drill down one more level to resolve the left and right gains for the Channel Gain block (oid = 21)

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 3,
      "oid": 21,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 2,
          "index": 10
        }
      }
    }
  ]
}

```

**Expected Results**

The JSON response to the above query gives us the two leaf control blocks `left-gain` and `right-gain` as shown below:

```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 3,
      "result": {
        "status": 0,
        "value": [
          {
            "role": "left-gain",
            "oid": 22,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                2,
                1,
                1,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Left gain",
            "owner": 21,
            "description": "Left channel gain",
            "constraints": null
          },
          {
            "role": "right-gain",
            "oid": 23,
            "constantOid": true,
            "identity": {
              "id": [
                1,
                2,
                1,
                1,
                1
              ],
              "version": "1.0.0"
            },
            "userLabel": "Right gain",
            "owner": 21,
            "description": "Right channel gain",
            "constraints": null
          }
        ]
      }
    }
  ]
}

```

Now retrieve the value for the `right-gain` using the generic getter for the property.

Copy and paste the following into the WebSocket King Client.  The level and index of the gain value parameter is obtained from the mock node source code and will be more fully described when we modify the gain control to add a new parameter in the next sections.

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 1
        }
      }
    }
  ]
}

```

**Expected Results**

The default value set in the mock device for the right-gain is `0` so we expect the returned value to be zero. The  JSON response from `NC-01` confirms this:

```

{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 0,
        "value": 0
      }
    }
  ]
}
```

Now we will set the `right-gain` to a value of 11 and verify the change has taken effect.  Copy and paste the following JSON formatted command to set the value of the right-gain:

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 2
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 1
        },
        "value": "11.0"
      }
    }
  ]
}

```
**Expected Results**

The command should be accepted with no errors.  JSON Response to the command should indicate status of 0.

Next retrieve the new gain value by copying and pasting the following into the WebSocket King client:

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 1
        }
      }
    }
  ]
}

```

**Expected Results**

NC-01 returns the new value of the `right-gain` parameter for the Stereo Gain Block:

```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 0,
        "value": "11.0"
      }
    }
  ]
}

```

**Subscribe to Change Event for the `right-gain` Parameter**

Add a subscription notification to changes on the `right-gain` parameter by opening a second  connection in WebSocket King Client.  Paste in the WebSocket for the NCMN and click `Connect`. Next paste in the JSON formatted command to open a new Session to the NCMN.  Next copy and paste the command below to subscribe for changes to the `right-gain` SetPoint parameter.  Note that the command being issues it directed to the Subscription Manager's (oid 5) method 3m1 which is described in the tutorial section of this document. 

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 4,
  "messageType": 2,
  "messages": [
    {
      "handle": 5,
      "oid": 5,
      "methodId": {
        "level": 3,
        "index": 1
      },
      "arguments": {
        "event": {
          "emitterOid": 23,
          "eventId": {
            "level": 5,
            "index": 1
          }
        }
      }
    }
  ]
}

```
**Expected Results**

The Subscription Manager will respond with a message indicating the subscription request was accepted. The session will be notified of any changes.

```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 4,
  "messages": [
    {
      "handle": 5,
      "result": {
        "status": 0
      }
    }
  ]
}

```

**Modify `right-gain` set point value and check notification is received**

Now whenever we modify the value of the `right-gain` set point parameter we can see notifications arriving.

Copy and paste the following command which will set the `right-gain` set point parameter to -3.0:

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 2
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 1
        },
        "value": "-3.0"
      }
    }
  ]
}

```

**Expected Results**

Since you registered for notifications for changes to the `right-gain` control you should see notifications of that change. Below is the expected result from invoking the Set method:
```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 0
      }
    }
  ]
}
```
The notification of the change event is shown below:

```

{
  "protocolVersion": "1.0.0",
  "messageType": 6,
  "sessionId": 3,
  "messages": [
    {
      "type": 0,
      "oid": 23,
      "eventId": {
        "level": 1,
        "index": 1
      },
      "eventData": {
        "propertyId": {
          "level": 5,
          "index": 1
        },
        "changeType": 0,
        "propertyValue": "-3.0"
      }
    }
  ]
}
```



 **Conclusions for Section One HOW-TO**

 In this section of the HOW-TO guide you setup all required NMOS infrastructure to run and interact with an NMOS Device that provides IS-12 NMOS Control functionality. You loaded an NMOS RDS so that the NMOS device can register its control endpoint for IS-12 in the form of a standard WebSocket. You have installed the NC-01 mock device and used it to explore how NMOS Control works for a simple Stereo Gain Control Block. You have used manual copy-and-paste of JSON protocol messages to act as an human-in-the-loop NMOS Controller.  

 In addition you have subscribed for a change event on a control block parameter of interest and verified your WebSocket session monitoring any changes received notification on a change event when you changed the parameter of interest.

 In the next section you will add a new parameter to the Stereo Gain Block's left and right gains.  You will become familiar with how to modify code to enhance existing control blocks to add functionality to an existing NMOS Control Block.


 #### Modifications to the Stereo Gain Block 

This section shows how to add in a simple `muted` parameter to the left and right gains that you worked with in the previous section.Note that this section is for illustrative purposes and in practice you would not modify the existing framework but rather create a derived class for new functionality - something you will explore in the next section of this HOW-TO.
 
 The requirements for this additional functionality are simple.  Each of the stereo channels will have an additional `boolean` parameter that controls if the channel is muted.  Turning on and off the muting does not effect the gain of the channel.  

**Steps to Implement**

- Edit the Features.ts file located in `code/src/NCModel` of the cloned mock node repo
- Add in a boolean parameter to the gain blocks
- Set the new parameter's level and index
- Set the default value to `false`
- Restart the Mock Node.
- 

**Editing Features.ts**

Open the Features.ts file with any editor.   Make the following changes to the file:

```
@@ -144,11 +144,11 @@ try
             new NcGain(22, true, 21, "left-gain", "Left gain", false, NcLockState.NoLock, [], true, [
                 new NcPort('input_1', NcIoDirection.Input, null),
                 new NcPort('output_1', NcIoDirection.Output, null),
-            ], null, 0, "Left channel gain", sessionManager),
+            ], null, 0, false, "Left channel gain", sessionManager),
             new NcGain(23, true, 21, "right-gain", "Right gain", false, NcLockState.NoLock, [], true, [
                 new NcPort('input_1', NcIoDirection.Input, null),
                 new NcPort('output_1', NcIoDirection.Output, null),
-            ], null, 0, "Right channel gain", sessionManager)
+            ], null, 0, false, "Right channel gain", sessionManager)
         ],
         [ 
             new NcPort('stereo_gain_input_1', NcIoDirection.Input, null),
@@ -189,7 +189,7 @@ try
                     new NcPort('input_2', NcIoDirection.Input, null),
                     new NcPort('output_1', NcIoDirection.Output, null),
                     new NcPort('output_2', NcIoDirection.Output, null),
-                ], null, 0, "Master gain", sessionManager)
+                ], null, 0, false, "Master gain", sessionManager)
             ],
             [ 
                 new NcPort('block_input_1', NcIoDirection.Input, null),

```

Modify the file `code/src/Server.ts` to make the following changes

```
@@ -162,6 +162,9 @@ export class NcGain extends NcActuator
     @myIdDecorator('5p1')
     public setPoint: number;
 
+    @myIdDecorator('5p2')
+    public mute: boolean;
+
     public classID: number[] = [ 1, 2, 1, 1, 1 ];
     public classVersion: string = "1.0.0";
 
@@ -178,12 +181,14 @@ export class NcGain extends NcActuator
         ports: NcPort[] | null,
         latency: number | null,
         setPoint: number,
+       mute: boolean,
         description: string,
         notificationContext: INotificationContext)
     {
         super(oid, constantOid, owner, role, userLabel, lockable, lockState, touchpoints, enabled, ports, latency, description, notificationContext);
 
         this.setPoint = setPoint;
+       this.mute = mute;
     }
 
     //'1m1'
@@ -197,6 +202,8 @@ export class NcGain extends NcActuator
             {
                 case '5p1':
                     return new CommandResponseWithValue(handle, NcMethodStatus.OK, this.setPoint, null);
+               case '5p2':
+                   return new CommandResponseWithValue(handle, NcMethodStatus.OK, this.mute, null);
                 default:
                     return super.Get(oid, propertyId, handle);
             }
@@ -218,7 +225,11 @@ export class NcGain extends NcActuator
                     this.setPoint = value;
                     this.notificationContext.NotifyPropertyChanged(this.oid, id, this.setPoint);
                     return new CommandResponseNoValue(handle, NcMethodStatus.OK, null);
-                default:
+              case '5p2':
+                    this.mute = value;
+                    this.notificationContext.NotifyPropertyChanged(this.oid, id, this.mute);
+                    return new CommandResponseNoValue(handle, NcMethodStatus.OK, null);
+              default:
                     return super.Set(oid, id, value, handle);
             }
         }

```

Now restart `npm run build-and-start` and explore the changes you have made to the `right-gain` control.  Note that since you have added in a `mute` to the base NCGain all objects of this type will have a `mute` capability including `left-gain`, and `master-gain` in the control hierarchy.  

**Read the Default Value for Mute**

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 2
        }
      }
    }
  ]
}

```

**Expected Value**

```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 0,
        "value": false
      }
    }
  ]
}

```

**Set the Mute for Right Channel to True**

```
{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 2
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 2
        },
        "value": "true"
      }
    }
  ]
}

```

**Read New Value**

```

{
  "protocolVersion": "1.0.0",
  "sessionId": 3,
  "messageType": 2,
  "messages": [
    {
      "handle": 2,
      "oid": 23,
      "methodId": {
        "level": 1,
        "index": 1
      },
      "arguments": {
        "id": {
          "level": 5,
          "index": 2
        }
      }
    }
  ]
}

```

**Expected Value**

The retrieved value for the mute on the `right-gain` shows the new value for `mute` as true as expected.

```
{
  "protocolVersion": "1.0.0",
  "messageType": 3,
  "sessionId": 3,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 0,
        "value": "true"
      }
    }
  ]
}

```



**Conclusions**

In this section you have started to explore the code for the NMOS Control Framework by adding a new parameter to the existing framework. You have modified the code to add the parameter to an existing Control Block and then used IS-12 Protocol to read, write and verify your code changes are worked as expected. 

You have seen how the overall framework supports additional functionality in a seamless manner. Some 15 odd code line changes provides full access to a new control parameter including read, write and notifications of changes.

In the next section you will follow another path to extension of the framework by creating a derived class based on the `NCGain` Control Block you worked with in the previous section. This is the expected path of extension to the NMOS Control Framework envisioned by AMWA and the NMOS Community. Implementors and Venders can derive from the existing framework class structure to add functionality and remain discoverable and controllable by other venders that use the framework.  

**Create a Derived Class from `NCGain`**

In the previous section we modified the framework `NCGain` class to add a parameter `mute`.  We now want to create our own class called `MyComGain`.  The new class will fit into the overall framework with all the functionality of   `NCGain` but with an additional `mute` parameter.  


**Create the Derived Class**
TODO
**Verify Class is Discoverable**
TODO
**Verify Existing Parameter Mods Work**
TODO
**Verify New Parameter Works**
TODO
**Set and Read**
TODO
**Subscription to Changes Work**
TODO



###Overall Conclusions

This HOW-TO has shown how to work with the NMOS Control Framework.  You have created a simple NMOS Device that uses NMOS IS-04 to advertise its control endpoint with an NMOS RDS.  You have worked with the IS-12 protocol to discover a NMOS Control for Stereo Gain and modified a parameter of one leg of the Stereo Gain Block.  You have gained experience with making simple modifications to the code for a TypeScript implementation of a NMOS Controllable Device based on the open-sourced NMOS Control Mock Node.  

We encourage you to continue exploration of how the NMOS Control Framework can enable compelling User-Stories for your customers and differentiate your products in the expanding [NMOS](https://www.amwa.tv/nmos-overview) community of venders and users while at the same time contributing to an open-standards based approach. 




