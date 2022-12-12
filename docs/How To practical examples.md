# How To practical examples

{:.no_toc}

- This will be replaced with a table of contents
{:toc}

## Introduction

This section showcases practical examples on the journey to implementing MS-05 / IS-12 in a device and as such provides a practical example of how to implement an NMOS Control Framework Device on a Linux system using the NMOS Control Mock Device.

`Note`: The NMOS Control Mock Device application is a cross platform application and can run on Linux, MacOS and Windows.

The HOWTO is meant to provide a relatively simple, easy-to-follow "recipe" for getting a controllable NMOS Node up and running in a short period of time.
It is not intended to be a tutorial article, and therefore, it excludes explanation regarding the NMOS Control Framework. The reader is referred to the tutorial section for more depth coverage of the framework.

The reader is assumed to have some experience with an NMOS infrastructure including IS-04 and NMOS Registration and Discovery (RDS). Also some experience in JavaScript and/or typescript will be helpful although not entirely required.

This HOW-TO also demonstrates to a certain extent how to construct and operate an NMOS Controller that interacts with the NMOS Control Device. In the HOW-TO, you will be the controller - creating, issuing and reading information from JSON commands and responses sent and received via a standard WebSocket connecting you with the controlled device.

## HOWTO Steps

This HOWTO will present the steps to install, modify and try out the NMOS Control Framework. The HOW-TO uses a fresh Ubuntu 20.04 installation.

### Basic Installation

This section will cover the basic steps to get a mock NMOS Controllable Node running on your system:

- Install the NMOS Controllable Mock Device (NCMD)
- Install the EasyNMOS Docker container which provides an NMOS RDS
- Verify NCMD registers and exposes it's NMOS IS-12 Control endpoint in the NMOS RDS
- Install a Chrome WebSocket extension
- Verify the NCMD can be reached via the WebSocket Control endpoint listed in the RDS
- Locate one of the control classes provided by the NCMD
- Verify the ability to read, and write a property of the control class instance
- Add a subscription in order to receive notifications for any changes experienced by the properties of the control class instance
- Modify the value of a property and verify notification are received

### Addition of a Vendor Specific Control Class

This section will make modifications to the basic system and show how to add in a new control class to the mock node. The new control class will extend one of the NMOS Control Framework classes to add functionality. You will learn how to extend a standard control class in a manner that gives all of the functionality of the framework while providing additional control features to clients.

## Installing the Mock NMOS Control Device

Install the NMOS Mock control device from its location on the public GitHub [repo](https://github.com/AMWA-TV/nmos-device-control-mock).

Follow the steps below from any directory on your Ubuntu host:

### Install the required packages

You will need `docker` for running the NMOS RDS and `npm` for running the mock NMOS Controllable Node. We first assume a fresh install of Ubuntu 20.04 and so do an update for packages prior to installation of required dependencies.

```txt
sudo apt-get update
sudo apt install -y npm
sudo apt install -y docker.io

```

Now install an NMOS RDS. We will use the RDS from [EasyNMOS](https://github.com/rhastie/easy-nmos)

You will run the RDS from one terminal window and the mock node from another. Open a terminal window and perform the following commands:

```txt
sudo docker pull rhastie/nmos-cpp:latest
```

**Expected Output**

```txt
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

```txt
sudo docker run -d -p 80:8010 -p 8011:8011 --name RDS rhastie/nmos-cpp:latest
sudo docker ps
```

**Expected Output**

The output of `sudo docker ps` will show the EasyNMOS NMOS docker image is up and running and required ports are mapped from the host's network to the docker container network.

```txt
CONTAINER ID   IMAGE                     COMMAND                 CREATED         STATUS         PORTS                                                                                                                   NAMES
9a8890946623   rhastie/nmos-cpp:latest   "/home/entrypoint.sh"   9 seconds ago   Up 8 seconds   1883/tcp, 11000-11001/tcp, 5353/udp, 0.0.0.0:8011->8011/tcp, :::8011->8011/tcp, 0.0.0.0:80->8010/tcp, :::80->8010/tcp   RDS
```

Verify the web server in the EasyNMOS docker container is up and running by opening [http://localhost/admin](http://localhost/admin) on your host. You should see the welcome screen for the open-source NVIDIA NMOS Commissioning Controller.

Now install the mock device from the repo:

```txt
git clone https://github.com/AMWA-TV/nmos-device-control-mock.git
cd nmos-device-control-mock/code
npm install
npm run build-and-start
```

**Expected output**

The output of npm run build-and-start will show status as the mock node is built and ran. The last few lines of the output will show the mock device registering with the RDS running as part of the docker image started above.

```log
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

You now have all the NMOS items needed to interact with the NMOS Control mock node. Since IS-12 uses a WebSocket control endpoint we will next browse the RDS registry to find the advertised WebSocket endpoint and use a Chrome extension that allows opening that WebSocket and sending and receiving IS-12 JSON formatted commands and responses.

Navigate in your preferred browser to the devices query location:
`http://127.0.0.1/x-nmos/query/v1.3/devices/`

**Expected output**

```json
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

`ws://127.0.0.1:8080/x-nmos/ncp/v1.0/connect`

This is the WebSocket used to interact with NMOS Control components.

**Install Chrome WebSocket Plugin**

We will install a Chrome WebSocket extension. Other WebSocket clients will work equally well.

Details on installing Chrome extensions can be found [here](https://support.google.com/chrome_webstore/answer/2664769?hl=en). Follow the instructions to open the Chrome Web Store and search for WebSocket King Client. Install this extension.

After completing the installation of WebSocket King open the extension in a new browser window. Copy and paste the WebSocket located in the RDS for the NC-01 NMOS Control Mock node into the connections field and click `connect`. The `Connect` button should turn to `Disconnect` indicating a successful connection to the NC-01 WebSocket endpoint.

Next we will verify the ability to read and write to the NMOS Control components running on the mock node. We will focus on reading and writing to the Stereo Gain Block and related objects provided by the mock node. Other aspects of control can also be explored by following the examples in the [IS-12 Specification](https://specs.amwa.tv/is-12/) example section. For purposes of this HOW-TO we will focus on working with the Stereo Control and adding code to extend this control then create a new control and interact with this new control. Also note that in an actual system most of the manual steps we are performing here would be performed by an NMOS Controller using the IS-12 specification. For more information about implementing an IS-12 NMOS Controller see the HOW-TO section for Controller implementations.

Next retrieve the members of the root block in order to identify the location of the Stereo gain block.

Send the following JSON formatted command to the NC-01 WebSocket. In the JSON command the value of `oid` 1 indicates we are directing this command at the root block. The `methodId` with level 1 and index 1 is the `getter` method and the `id` level and index of 2 and 10 respectively targets the `2p10` `members` property of the root block.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

The device responds with a JSON containing `NcBlockMemberDescriptor` member descriptors for the root block. The sub block we are interested in is the Stereo Gain block. We see the block is present along with its Object ID (oid). The oid is unique across all control elements and we will use it to further interrogate the Stereo Gain block and find its members. The oid is 31.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 3,
      "result": {
        "status": 200,
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

You will now make use of the generic Get method `1m1` (level 1, index 1) to find the members `2p10` of the Stereo gain block (oid: 31). Send the following JSON formatted command to the NC-1 control WebSocket.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

**Expected Output**

The device responds to the above command with a JSON formatted response containing all the members of the Stereo Gain block.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 3,
      "result": {
        "status": 200,
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

Next drill down one more level to resolve the left and right gains for the Channel Gain block (oid = 21) using the same `Get` method but now targeted at the `channel-gain` `oid` 21.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

The JSON response to the above command gives us the two control blocks `left-gain` and `right-gain` as shown below:

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 2,
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

Now retrieve the set point gain value for the `right-gain` `oid` 23 using the generic Get method targeted at property (5p1).

Copy and paste the following into the WebSocket King Client. The level and index of the gain set point property is obtained from the definition of the `NcGain` class in the [MS-05-02](https://github.com/AMWA-TV/ms-05-02/blob/v1.0-dev/idl/NC-Framework.webidl) webIDL.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

The default value set in the mock device for the right-gain set point value is `0`, so we expect the returned value to be zero. The JSON response from `NC-01` confirms this:

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 200,
        "value": 0
      }
    }
  ]
}
```

Now we will set the `right-gain` set point gain (5p1) value to 11 and verify the change has taken effect. We will use the generic `Set` method for this `(1p2)`. Copy and paste the following JSON formatted command to set the new value:

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

The command should be accepted with no errors. The JSON response to the command should indicate a status of 0 (Ok).

Next retrieve the new set point gain value by copying and pasting the following into the WebSocket King client. The JSON command uses the `right-gain` `oid` 23 and the `get` methodId level and index (`1,1`). The argument for the `get` method is the id of the set point property of NcGain provided as level and index (`5,1`).

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

NC-01 returns the new value of the `right-gain` set point gain value in the response.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 200,
        "value": 11.0
      }
    }
  ]
}
```

**Subscribe to property changes for the `right-gain`**

Add a subscription notification to changes on the `right-gain` control by sending a subscription command to the SubscriptionManager. Paste in the JSON formatted command below to subscribe for changes. Note that the command is directed to the Subscription Manager's (oid 5) method (3m1) which is described in the tutorial section of this document and targets the `right-gain` by using its `oid` of 23 as the `emitterOid`. The `eventId` is provided as level 1 and index 1 which represents the PropertyChanged event defined in NcObject.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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
            "level": 1,
            "index": 1
          }
        }
      }
    }
  ]
}
```

**Expected Results**

The Subscription Manager will respond with a message indicating the subscription request was accepted.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 5,
      "result": {
        "status": 200
      }
    }
  ]
}
```

**Modify `right-gain` set point value and check notification is received**

Now whenever we modify the value of the `right-gain` set point property we can see notifications arriving.

Copy and paste the following command which will set the `right-gain` set point property to -3.0. The JSON command uses the `right-gain` oid of `23` and the `set` method to set the new value.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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

Since you registered for notifications for changes to the `right-gain` control you should see notifications of that change. Below is the expected result from invoking the `Set` method that you should see in your WebSocket Client when you send the command to set the value.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 200
      }
    }
  ]
}
```

You should also receive the notification of the change event as shown below. In the JSON notification you see the `oid` of the `right-gain` 23 along with the `propertyId` that was changed. Finally, the `changeType` and new `propertyValue` are provided in the change event notification.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 2,
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

### Section conclusions

In this section of the HOW-TO guide you have setup all required NMOS infrastructure to run and interacted with an NMOS Device that provides IS-12 NMOS Control functionality. You loaded an NMOS RDS so that the NMOS device can register its control endpoint for IS-12 in the form of a standard WebSocket. You have installed the NC-01 mock device and used it to explore how NMOS Control works for a simple Stereo Gain Control block. You have used manual copy-and-paste of JSON protocol messages to act as an human-in-the-loop NMOS Controller.

In addition you have subscribed to property changes for the gain control of interest and verified your WebSocket session has received notifications when you changed the property of interest.

In the next section you will add a new property to the Stereo Gain Block's left and right gains. You will become familiar with how to modify code to enhance existing NMOS Control classes.

## Modifications to the Stereo Gain Block

This section shows how to add in a simple `mute` property to the left and right gains that you worked with in the previous section. The requirements for this additional functionality are simple. Each of the stereo channels will have an additional `boolean` property that controls if the channel is muted. Turning on and off the muting does not effect the gain of the channel. The strategy we will take is the recommended practice for extending the framework. You will create a subclass of the NMOS Control Framework NcGain class and extend this class to add in a `mute` property.

**Steps to Implement**

- Edit the Features.ts file located in `code/src/NCModel` of the cloned mock node repo
- Create a subclass of NcGain called NcGainCustom
- Add in a boolean `mute` property to the NcGainCustom block
- Set the default value to `false`
- Update the Server.ts file located in `code/src` to use the new class in the StereoGain Block
- Restart the Mock Node.
- Explore the changes

**Editing Features.ts**

Open the Features.ts file with any editor. Make the following changes to the file to create the `NcGainCustom` class that extends the framework's NcGain class.

```typescript
export class NcGainCustom extends NcGain
{
    @myIdDecorator('6p1')
    public mute: Boolean;

    public classID: number[] = [ 1, 2, 1, 1, 1, 0, 1 ];
    public classVersion: string = "1.0.0";

    public constructor(
        oid: number,
        constantOid: boolean,
        owner: number | null,
        role: string,
        userLabel: string,
        lockable: boolean,
        lockState: NcLockState,
        touchpoints: NcTouchpoint[],
        enabled: boolean,
        ports: NcPort[] | null,
        latency: number | null,
        setPoint: number,
        mute: boolean,
        description: string,
        notificationContext: INotificationContext)
    {
        super(oid, constantOid, owner, role, userLabel, lockable, lockState, touchpoints, enabled, ports, latency, setPoint, description, notificationContext);

        this.mute = mute;
    }

    //'1m1'
    public override Get(oid: number, propertyId: NcElementId, handle: number) : CommandResponseNoValue
    {
        if(oid == this.oid)
        {
            let key: string = `${propertyId.level}p${propertyId.index}`;

            switch(key)
            {
                case '6p1':
                  return new CommandResponseWithValue(handle, NcMethodStatus.OK, this.mute, null);
                default:
                    return super.Get(oid, propertyId, handle);
            }
        }

        return new CommandResponseNoValue(handle, NcMethodStatus.InvalidRequest, 'OID could not be found');
    }

    //'1m2'
    public override Set(oid: number, id: NcElementId, value: any, handle: number) : CommandResponseNoValue
    {
        if(oid == this.oid)
        {
            let key: string = `${id.level}p${id.index}`;

            switch(key)
            {
              case '6p1':
                    this.mute = value;
                    this.notificationContext.NotifyPropertyChanged(this.oid, id, this.mute);
                    return new CommandResponseNoValue(handle, NcMethodStatus.OK, null);
              default:
                    return super.Set(oid, id, value, handle);
            }
        }

        return new CommandResponseNoValue(handle, NcMethodStatus.InvalidRequest, 'OID could not be found');
    }

    public static override GetClassDescriptor(): NcClassDescriptor 
    {
        let baseDescriptor = super.GetClassDescriptor();

        let currentClassDescriptor = new NcClassDescriptor("NcGainCustom class descriptor",
            [ 
                new NcPropertyDescriptor(new NcElementId(6, 1), "mute", "NcBoolean", false, false, false, false, null, "TRUE iff muted"),
            ],
            [],
            []
        );

        currentClassDescriptor.properties = currentClassDescriptor.properties.concat(baseDescriptor.properties);
        currentClassDescriptor.methods = currentClassDescriptor.methods.concat(baseDescriptor.methods);
        currentClassDescriptor.events = currentClassDescriptor.events.concat(baseDescriptor.events);

        return currentClassDescriptor;
    }
}
```

Key modifications to the code for the derived class include the following code snippet:

```typescript
export class NcGainCustom extends NcGain
{
    @myIdDecorator('6p1')
    public mute: Boolean;

    public classID: number[] = [ 1, 2, 1, 1, 1, 0, 1 ]; // classID, 1 extra level below NcGain (includes the authority key 0 in this case)
    public classVersion: string = "1.0.0";
}
```

Here you have created a subclass of NcGain which is a standard class in the NMOS Control Framework. The new class has all the features of the framework including discoverability, event notification subscriptions and communications via the IS-12 protocol.

The code snippet below shows the additions needed to override the `get` and `set` methods inherited from the base class `NcObject`:

```typescript
 //'1m1'
    public override Get(oid: number, propertyId: NcElementId, handle: number) : CommandResponseNoValue
    {
        if(oid == this.oid)
        {
            let key: string = `${propertyId.level}p${propertyId.index}`;

            switch(key)
            {
                case '6p1':
                    return new CommandResponseWithValue(handle, NcMethodStatus.OK, this.mute, null);
                default:
                    return super.Get(oid, propertyId, handle);
            }
        }

        return new CommandResponseNoValue(handle, NcMethodStatus.InvalidRequest, 'OID could not be found');
    }

    //'1m2'
    public override Set(oid: number, id: NcElementId, value: any, handle: number) : CommandResponseNoValue
    {
        if(oid == this.oid)
        {
            let key: string = `${id.level}p${id.index}`;

            switch(key)
            {
              case '6p1':
                    this.mute = value;
                    this.notificationContext.NotifyPropertyChanged(this.oid, id, this.mute);
                    return new CommandResponseNoValue(handle, NcMethodStatus.OK, null);
              default:
                    return super.Set(oid, id, value, handle);
            }
        }

        return new CommandResponseNoValue(handle, NcMethodStatus.InvalidRequest, 'OID could not be found');
    }
```

In this code you override the base class `Get` and `Set` methods to also handle the new `mute` property before passing back up to the base class for other inherited properties.
Finally in the following code section you override the `GetClassDescriptor` to provide information about your new derived class specific to the new class (in this case the additional `mute` property).

```typescript
public static override GetClassDescriptor(): NcClassDescriptor
{
      let baseDescriptor = super.GetClassDescriptor();

      let currentClassDescriptor = new NcClassDescriptor("NcGainCustom class descriptor",
          [ 
              new NcPropertyDescriptor(new NcElementId(6, 1), "mute", "NcBoolean", false, false, false, false, null, "TRUE iff muted"),
          ],
          [],
          []
      );

      currentClassDescriptor.properties = currentClassDescriptor.properties.concat(baseDescriptor.properties);
      currentClassDescriptor.methods = currentClassDescriptor.methods.concat(baseDescriptor.methods);
      currentClassDescriptor.events = currentClassDescriptor.events.concat(baseDescriptor.events);

      return currentClassDescriptor;
}
```

Next, Modify the file `code/src/Server.ts` to make the following changes that plug in your new `NcGainCustom` block into the overall controls provided by the NMOS device control mock code only in the replacement of the framework's `NcGain` with your new extended `NcGainCustom`.

```typescript
const channelGainBlock = new NcBlock(
        false,
        21,
        true,
        31,
        'channel-gain',
        'Channel gain',
        false,
        NcLockState.NoLock,
        null,
        true,
        null,
        null,
        null,
        null,
        null,
        false,
        [
            new NcGainCustom(22, true, 21, "left-gain", "Left gain", false, NcLockState.NoLock, [], true, [
                new NcPort('input_1', NcIoDirection.Input, null),
                new NcPort('output_1', NcIoDirection.Output, null),
            ], null, 0, false, "Left channel gain with mute", sessionManager),
            new NcGainCustom(23, true, 21, "right-gain", "Right gain", false, NcLockState.NoLock, [], true, [
                new NcPort('input_1', NcIoDirection.Input, null),
                new NcPort('output_1', NcIoDirection.Output, null),
            ], null, 0, false, "Right channel gain with mute", sessionManager)
        ],

        ... 
        [ 
            new NcPort('stereo_gain_input_1', NcIoDirection.Input, null),
            new NcPort('stereo_gain_input_2', NcIoDirection.Input, null),
            new NcPort('stereo_gain_output_1', NcIoDirection.Output, null),
            new NcPort('stereo_gain_output_2', NcIoDirection.Output, null)
        ],
        [
            new NcSignalPath('left_gain_input', 'Left gain input', new NcPortReference([], "stereo_gain_input_1"), new NcPortReference(['left-gain'], 'input_1')),
            new NcSignalPath('left_gain_output', 'Left gain output', new NcPortReference(['left-gain'], 'output_1'), new NcPortReference([], "stereo_gain_output_1")),
            new NcSignalPath('right_gain_input', 'Right gain input', new NcPortReference([], "stereo_gain_input_2"), new NcPortReference(['right-gain'], 'input_1')),
            new NcSignalPath('right_gain_output', 'Right gain output', new NcPortReference(['right-gain'], 'output_1'), new NcPortReference([], "stereo_gain_output_2")),
        ],
        "Channel gain block with Mute",
        sessionManager);

        const stereoGainBlock = new NcBlock(
            false,
            31,
            true,
            1,
            'stereo-gain',
            'Stereo gain',
            false,
            NcLockState.NoLock,
            null,
            true,
            null,
            null,
            null,
            null,
            null,
            false,
            [
                channelGainBlock,
                new NcGainCustom(24, true, 31, "master-gain", "Master gain", false, NcLockState.NoLock, [], true, [
                    new NcPort('input_1', NcIoDirection.Input, null),
                    new NcPort('input_2', NcIoDirection.Input, null),
                    new NcPort('output_1', NcIoDirection.Output, null),
                    new NcPort('output_2', NcIoDirection.Output, null),
                ], null, 0, false, "Master gain with mute", sessionManager)
            ],
            [ 
                new NcPort('block_input_1', NcIoDirection.Input, null),
                new NcPort('block_input_2', NcIoDirection.Input, null),
                new NcPort('block_output_1', NcIoDirection.Output, null),
                new NcPort('block_output_2', NcIoDirection.Output, null)
            ],
            [
                new NcSignalPath('block-in-1-to-left-gain-in', 'Block input 1 to left gain input', new NcPortReference([], "block_input_1"), new NcPortReference(['stereo-gain'], 'stereo_gain_input_1')),
                new NcSignalPath('left-gain-out-to-master-gain-in-1', 'Left gain output to master gain input 1', new NcPortReference(['stereo-gain'], 'stereo_gain_output_1'), new NcPortReference(['master-gain'], "input_1")),
                new NcSignalPath('master-gain-out-1-to-block-out-1', 'Master gain output 1 to block output 1', new NcPortReference(['master-gain'], "output_1"), new NcPortReference([], 'block_output_1')),
                new NcSignalPath('block-in-2-to-right-gain-in', 'Block input 2 to right gain input', new NcPortReference([], "block_input_2"), new NcPortReference(['stereo-gain'], 'stereo_gain_input_2')),
                new NcSignalPath('right-gain-out-to-master-gain-in-2', 'Right gain output to master gain input 2', new NcPortReference(['stereo-gain'], 'stereo_gain_output_2'), new NcPortReference(['master-gain'], "input_2")),
                new NcSignalPath('master-gain-out-2-to-block-out-2', 'Master gain output 2 to block output 2', new NcPortReference(['master-gain'], "output_2"), new NcPortReference([], 'block_output_2'))
            ],
            "Stereo gain block with Mute",
            sessionManager);
```

The changes to the original Server.ts file are simply replacements of the NMOS Control Framework's NcGain with the new NcGainCustom. A clearer picture of the changes is shown below as a standard `diff` format.

```typescript
@@ -14,7 +14,7 @@ import { SessionManager } from './SessionManager';
 import { NcBlock, RootBlock } from './NCModel/Blocks';
 import { NcClassManager, NcDeviceManager, NcSubscriptionManager } from './NCModel/Managers';
 import { NcIoDirection, NcLockState, NcPort, NcPortReference, NcSignalPath, NcTouchpointNmos, NcTouchpointResourceNmos } from './NCModel/Core';
-import { NcDemo, NcGain, NcReceiverMonitor } from './NCModel/Features';
+import { NcDemo, NcGainCustom, NcReceiverMonitor } from './NCModel/Features';
 
 export interface WebSocketConnection extends WebSocket {
     isAlive: boolean;
@@ -141,14 +141,14 @@ try
         null,
         false,
         [
-            new NcGain(22, true, 21, "left-gain", "Left gain", false, NcLockState.NoLock, [], true, [
+            new NcGainCustom(22, true, 21, "left-gain", "Left gain", false, NcLockState.NoLock, [], true, [
                 new NcPort('input_1', NcIoDirection.Input, null),
                 new NcPort('output_1', NcIoDirection.Output, null),
-            ], null, 0, "Left channel gain", sessionManager),
-            new NcGain(23, true, 21, "right-gain", "Right gain", false, NcLockState.NoLock, [], true, [
+            ], null, 0, false, "Left channel gain with mute", sessionManager),
+            new NcGainCustom(23, true, 21, "right-gain", "Right gain", false, NcLockState.NoLock, [], true, [
                 new NcPort('input_1', NcIoDirection.Input, null),
                 new NcPort('output_1', NcIoDirection.Output, null),
-            ], null, 0, "Right channel gain", sessionManager)
+            ], null, 0, false, "Right channel gain with mute", sessionManager)
         ],
         [ 
             new NcPort('stereo_gain_input_1', NcIoDirection.Input, null),
@@ -162,7 +162,7 @@ try
             new NcSignalPath('right_gain_input', 'Right gain input', new NcPortReference([], "stereo_gain_input_2"), new NcPortReference(['right-gain'], 'input_1')),
             new NcSignalPath('right_gain_output', 'Right gain output', new NcPortReference(['right-gain'], 'output_1'), new NcPortReference([], "stereo_gain_output_2")),
         ],
-        "Channel gain block",
+        "Channel gain block with Mute",
         sessionManager);
 
         const stereoGainBlock = new NcBlock(
@@ -184,12 +184,12 @@ try
             false,
             [
                 channelGainBlock,
-                new NcGain(24, true, 31, "master-gain", "Master gain", false, NcLockState.NoLock, [], true, [
+                new NcGainCustom(24, true, 31, "master-gain", "Master gain", false, NcLockState.NoLock, [], true, [
                     new NcPort('input_1', NcIoDirection.Input, null),
                     new NcPort('input_2', NcIoDirection.Input, null),
                     new NcPort('output_1', NcIoDirection.Output, null),
                     new NcPort('output_2', NcIoDirection.Output, null),
-                ], null, 0, "Master gain", sessionManager)
+                ], null, 0, false, "Master gain with mute", sessionManager)
             ],
             [ 
                 new NcPort('block_input_1', NcIoDirection.Input, null),
@@ -205,7 +205,7 @@ try
                 new NcSignalPath('right-gain-out-to-master-gain-in-2', 'Right gain output to master gain input 2', new NcPortReference(['stereo-gain'], 'stereo_gain_output_2'), new NcPortReference(['master-gain'], "input_2")),
                 new NcSignalPath('master-gain-out-2-to-block-out-2', 'Master gain output 2 to block output 2', new NcPortReference(['master-gain'], "output_2"), new NcPortReference([], 'block_output_2'))
             ],
-            "Stereo gain block",
+            "Stereo gain block with Mute",
             sessionManager);
 
     const rootBlock = new RootBlock(
@@ -497,4 +497,4 @@ try
 catch (err) 
 {
     console.log(err);
-}
\ No newline at end of file
+}
```

Now restart `npm run build-and-start` and explore the changes you have made to the `right-gain`. Since you've used the new NcGainCustom for `master gain` and `left-gain` these will now also have the new `mute` property.

The `oids` remain the same with this change but we must use the level and property index for the derived class to interact with the `mute` property. The level is now 6 in the class hierarchy and the property for `mute` is 1 as indicated in the code `@myIdDecorator('6p1')`.

**Read the Default Value for Mute**

Now paste in the JSON formatted command below into your WebSocket client. Note that the command is directed to the NcGainCustom's (oid 23) method (1m1) which is the `get` method. The command requests the value of the new `mute` property (level 6, index 1).

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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
          "level": 6,
          "index": 1
        }
      }
    }
  ]
}
```

**Expected Value**

The JSON formatted response should be returned by the NMOS device with the default value for `mute` which we set in the code to be `false`.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 200,
        "value": false
      }
    }
  ]
}
```

**Set the Mute for Right Channel to True**

Now paste in the JSON formatted command below into your WebSocket client. Note that the command is directed to the NcGainCustom instance (oid 23) method (1m2) which is the `set` method. The command sets the value of the new `mute` property (level 6, index 1) to `true`.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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
          "level": 6,
          "index": 1
        },
        "value": "true"
      }
    }
  ]
}
```

**Read New Value**

Now paste in the JSON formatted command below to re-read the value of the `mute` property. The method is again `level` 1 `index` 1 which is the `get` method. The `arguments` for the command indicate the target property which is the new `mute` property (level 6, index 1).

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 0,
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
          "level": 6,
          "index": 1
        }
      }
    }
  ]
}
```

**Expected Value**

The retrieved value for the mute on the `right-gain` shows the new value for `mute` as true as expected.

```json
{
  "protocolVersion": "1.0.0",
  "messageType": 1,
  "messages": [
    {
      "handle": 2,
      "result": {
        "status": 200,
        "value": true
      }
    }
  ]
}
```

### Section conclusions

In this section you learned how to extend the NMOS Control framework though class inheritance. You created a derived class that added functionality to the `NcGain` control clock to allow a client to `mute` the master, left or right channels independently. You saw how simple inheritance can create additional capabilities in the framework while the overall interaction with control blocks remains the same and all the functionality provided by the framework including discoverability, subscription for change events and control over a standard WebSocket come for free.

## Overall conclusions

This HOW-TO has shown how to work with the NMOS Control Framework. You have created a simple NMOS Device that uses NMOS IS-04 to advertise its control endpoint with an NMOS RDS. You have worked with the IS-12 protocol to discover an NMOS Control Stereo Gain and modified a property of one leg of the Stereo Gain Block. You have gained experience with making simple modifications to the code for a TypeScript implementation of an NMOS Controllable Device based on the open-sourced NMOS Control Mock Node.

## Further Directions

All code and APIs provided by NMOS are completely open and free for any purposes, private or commercial. You are encouraged to continue exploration of how the NMOS Control Framework can enable compelling User-Stories for your customers and differentiate your products in the expanding [NMOS](https://www.amwa.tv/nmos-overview) community of vendors and users while at the same time contributing to an open-standards based approach.
