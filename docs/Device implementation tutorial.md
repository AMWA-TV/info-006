# Device implementation tutorial

{:.no_toc}

- This will be replaced with a table of contents
{:toc}

This section covers the basis for quickly building an MS-05 / IS-12 device implementation.

## Guidance

This section provides guidance in select focus areas required for device implementations.

For full definitions of models referred to in this document please check the models published in the [Framework](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Framework.html) or the [Feature Sets Register](https://specs.amwa.tv/nmos-control-feature-sets/).

The basic device workflow follows the diagram below where individual steps are detailed in the following subsections.

| ![Basic device sequence](images/basic-device-sequence.png) |
|:--:|
| _**Basic device sequence**_ |

### Modelling the control classes

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/NcObject.html) specification all control classes inherit from `NcObject`.

This base control class exposes important properties but also [generic methods](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/NcObject.html#generic-getter-and-setter) for getting and setting property values.

`NcObject` also defines the [PropertyChanged](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/NcObject.html#propertychanged-event) event which is fundamental for subscriptions and notifications to work.

As per the [MS-05-01](https://specs.amwa.tv/ms-05-01/branches/v1.0-dev/docs/Identification.html) specification there are different types of identifiers which ultimately can be split into two categories:

- dynamic identifiers (object identifiers)
- persistent identifiers (roles, class identities and data type names)

| ![Identities](images/identities.png) |
|:--:|
| _**Identities**_ |

#### Block control classes

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Blocks.html) specification blocks are containers for other control classes.

All devices have at the very least a `root block` which is the top most block in the device model. The root block has an `oid` of 1 and the role of `root`.

Control classes which are nested inside a block are advertised using descriptors in the `members` property of [NcBlock](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Framework.html#ncblock).

The `members` property in blocks enables [device model discovery](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Blocks.html#device-model-discovery) of the device structure.

Blocks are also useful for quickly finding a particular control class by using the [search methods](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Blocks.html#search-methods) provided.

#### Manager control classes

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Managers.html) specification managers are special classes which collate information which pertains to the entire device.

Typical managers included in the root block are:

- Device manager (holds general information about the device including product information and serial numbers)
- Class manager (offers means of class and data type discovery)

#### Worker control classes

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Workers.html) specification workers are special classes which handle control or monitoring features for a particular specific device domain.

Different devices will need to use different workers depending on their functionality set.

Indeed, sometimes devices might also need to expose vendor specific functionality by creating non-standard worker classes (see [Non-standard classes](#non-standard-classes-used-to-model-vendor-specific-functionality)).

#### Context identity mapping (Receiver monitor example)

[MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/NcObject.html#touchpoints) specifies an identity mapping mechanism available in the base `NcObject` class. This touchpoint mechanism can be used to associate identities from outside contexts with entities inside the control structure of the device.

One such example is the [ReceiverMonitor](https://specs.amwa.tv/nmos-control-feature-sets/branches/main/monitoring/#ncreceivermonitor) control class which is used to express connection and payload statuses for an attached stream receiver.

This allows for a `Receiver monitor` to be associated with a specific [NMOS IS-04](https://specs.amwa.tv/is-04/) receiver.

A device is expected to offer touchpoints to map identities wherever relevant (For example if the device has a specific worker class like the `Receiver monitor` which is linked to a resource in another context or specification like an NMOS IS-04 receiver).

| ![Context identity mapping](images/context-identity-mapping.png) |
|:--:|
| _**Context identity mapping**_ |

#### Minimum requirements

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Blocks.html) specification all MS-05 / IS-12 devices need to expose a structure starting with the root block which always has an `oid` of 1.

A minimal implementation of a device will have at least two [managers](Device%20implementation%20tutorial.md#manager-control-classes) listed in the root block:

- Device manager
- Class manager

| ![Typical device structure](images/typical-device-structure.png) |
|:--:|
| _**Typical device structure**_ |

A device is expected to allow its structure to be discovered (see [Block control classes](Device%20implementation%20tutorial.md#block-control-classes)) by exposing its capabilities in nested blocks starting with the `root block`.

#### Non-standard classes used to model vendor specific functionality

Non-standard control classes can be created by branching off from a standard control class and following the class ID generation guidelines specified in [MS-05-01](https://specs.amwa.tv/ms-05-01/branches/v1.0-dev/docs/Appendix_A_-_Class_ID_Format.html).

Here is an example of a new worker control class called `DemoClassAlpha`. It inherits from [NcWorker](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Framework.html#ncworker) which has an identity of `[1, 2]` and adds the authority key (in this case 0, but would be a negative number if the vendor has an OUI or CID) followed by the index 1.

```json
{
  "role": "DemoClassAlpha",
  "oid": 111,
  "constantOid": true,
  "classId": [
      1,
      2,
      0,
      1
    ],
  "userLabel": "Demo class alpha",
  "owner": 1,
  "description": "Demo control class alpha",
  "constraints": null
}
```

A subsequent non-standard worker would look like this:

```json
{
  "role": "DemoClassBeta",
  "oid": 150,
  "constantOid": true,
  "classId": [
      1,
      2,
      0,
      2
    ],
  "userLabel": "Demo class beta",
  "owner": 1,
  "description": "Demo control class beta",
  "constraints": null
}
```

ensuring class identity uniqueness.

| ![Non-standard branching](images/non-standard-branching.png) |
|:--:|
| _**Non-standard branching**_ |

### Exposing models through the protocol

After a device has initiated its device model structure and allocated oids to every control class instance, it then either waits for external commands which interact with these entities (e.g. get/set values, invoke actions) or sends notifications for properties which have changed if there are subscriptions.

[IS-12](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/Protocol_messaging.html) defines the protocol messaging behavior but also what the different JSON representations are for specific [data types](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/Data_type_marshalling.html).

#### Control endpoint advertisement (in NMOS IS-04)

The [NMOS IS-12](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/IS-04_interactions.html) specification explains that the control endpoint is advertised in the controls array as part of the NMOS device resource. The schema for the NMOS device resource is available in the [NMOS IS-04](https://specs.amwa.tv/is-04/branches/v1.3.1/APIs/schemas/with-refs/device.html) specification.

It is expected that an IS-12 enabled device exposes a `urn:x-nmos:control:ncp` control type in the controls array for its NMOS device resource.

Control endpoint example:

```json
{
  ...
    "senders": [
        ...
    ],
    "receivers": [
        ...
    ],
    "controls": [
        {
            "type": "urn:x-nmos:control:ncp/v1.0",
            "href": "ws://hostname/example"
        }
    ],
    "type": "urn:x-nmos:device:generic",
    "id": "58f6b536-ca4c-43fd-880a-9df2501fc125",
  ...
}
```

#### Mapping commands and returning responses

As per the [NMOS IS-12](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/Protocol_messaging.html#command-message-type) specification a device is expected to respond to [Commands](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/Sending_commands.html) sent by a controller.

`Note`: Multiple commands can be sent in the commands array.

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/NcObject.html#generic-getter-and-setter) specification all control classes inherit from `NcObject` which specifies generic `Get` and `Set` methods.

These methods can be used by a controller to get the value of a property in a control class or set the value of a property in a control class if write allowed. Furthermore, any control class could have other methods which can be invoked in the same way as the generic methods.

As specified by `MS-05-02` any method response inherits from the base data type [NcMethodResult](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/Framework.html#ncmethodresult).

| ![Command example](images/command-example.png) |
|:--:|
| _**Command example**_ |

#### Subscriptions, events and notifications

As per the [MS-05-02](https://specs.amwa.tv/ms-05-02/branches/v1.0-dev/docs/NcObject.html#propertychanged-event) specification all control classes inherit from `NcObject` which specifies the `PropertyChanged` event.

This means any object in the device model can be subscribed to in order to receive property change notifications.
A device is expected to allow controllers to [Subscribe](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/Subscribing_to_events.html) to object ids it is interested in by correctly handling `Subscription` messages and sending back `SubscriptionResponse` messages as specified in [NMOS IS-12](https://specs.amwa.tv/is-12/branches/v1.0-dev/docs/Protocol_messaging.html).

A device is also expected to use the underlying WebSocket control protocol context and the subscriptions received in order to determine when a notification message needs to be sent to a controller.

## How to

HOW TO practical examples are available [here](How%20To%20practical%20examples.md).
