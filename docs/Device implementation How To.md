# Device implementation How To

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

This HOWTO will present the steps:

### Basic Installation

This section will do the most basic steps to get a NMOS Controllable Node running on your system. 

- Install the NMOS Controlable Mock Device (NCMD)
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


...


## Recap of HOWTO

This HOWTO has shown how to add controllability to an NMOS node using the NMOS Control Framework. ...



