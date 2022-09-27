# Device implementation tutorial

{:.no_toc}

- This will be replaced with a table of contents
{:toc}

This section covers the basis for quickly building an MS-05 / IS-12 device implementation.

## Guidance

This section provides guidance in select focus areas required for device implementations.

### Modelling the control classes

Summary of inheritance model in the framework.

Cover base NcObject, NcWorker, core managers and blocks

#### Minimum requirements

Mention the minimum structure, root block, managers

#### Vendor specific control classes

Guidance of class identity and how to create a vendor specific control class.

### Exposing models through the protocol

Summary of how models are mapped in IS-12

#### Control endpoint advertisement (in NMOS IS-04)

How the endpoint is advertised in IS-04

#### Keeping track of control sessions

Overview of control sessions

#### Mapping commands and returning responses

How commands are mapped to methods in a control classes and response types

#### Subscriptions, events and notifications

How to handle subscriptions in the Subscription manager and how events are mapped through notifications.

#### Handling heartbeats

Overview of heartbeats

## How to

A practical example HOW TO is available [here](Device%20implementation%20How%20To.md).
