# Specification for a network of nodes with IOTA as payment solution

**Status of the project:** This repo contains rough specifications for a network of IOTA light wallet nodes.

# Specifications

## Preconditions:

In sense of the CAP theorem this specification prefers availability and partion tolerance over consistency. It is designed to be eventual consistent.

## Naming Convention:

## Node

A node is a embedded device which can be sensor- or payment-node. Node is the generic name for both.

## SensorNode

A sensor node is a small embedded device which collect some data about the sourounding environment. This could be the temperature, as well as the traffic of a specific street.

## PaymentNode

A payment node is a small embedded device which accepts IOTA payments and provides some service. This could be a car park barrier, as well as a snack automat.

## NodeGroup

A NodeGroup is a group of Sensor- or PaymentNodes which fullfil the same purpose in a given area.

## NodeGroup Member

A NodeGroup Member is a Node within a NodeGroup.

## NodeGroup Neighbor

All other Nodes within a NodeGroup are the Nodes Neighbors.

##### Examples

**Temperature sensors:** Temperature sensors within a given area. This could be a district of a city or postcode area. These sensors provide information which are equal from the outside perspective. (temperature of a given area)

## NodeID

Each node has its own independent NodeID within a NodeGroup.

## Gateway

A Gateway is a more powerful embedded device, which gives the nodes indirect access to the tangle.

## SeedNode

A SeedNode is a Node within a NodeGroup which holds the configuration for new NodeGroup Members. It is also responsible for collecting the status of the NodeGroup for a Gateway.

## Address Milestone

A Address Milestone is an ID for a given Iota address which will be set by a Gateway. A Gateway sets this ID on every [X]th transaction.

## NodeGroup Seed Key

A NodeGroup Seed key is a shared Iota seed. Therefore every NodeGroup Member uses the same seed on the tangle.

# Routing

This specification prefers Content Centric Networking for routing purposes, but can also used with a traditional TCP/IP stack or other network-layer solutions like 5LoWPAN.

# Short function description

## Communication

Nodes can communicate peer to peer within their NodeGroup. Each node can also communicate to any Gateway. Nodes cannot talk to other Node outside of their NodeGroup.


# Status Refresh Request
A Gateway requests every x seconds the status of a NodeGroup by its SeedNode. The response of one or more SeedNode contains the following information:
- HealthStatus of every Member
- Current used addresses
- Prices of the services or sensor data
- New added Member, if there were any between current status call and last one.


# Address transactions
A Gateway gets the current used addresses by one or more SeedNode within its Status Requests. A Gateway subscribes to all known addresses and gets therefore all incoming transactions. The Gateway caches these transactions, but doesn't forword them directly after they came in. The NodeGroup Member which is interested in an incoming transaction can request an addresses transactions since a specific Account Milestone. The response contains all new transactions since the given Address Milestone. It can request a Gateway and/or its NodeGroup Neighbors for this information. The Member which is intersted in an incoming transaction should use an timeout and should request the addresses transactions again, if it is still waiting for it. A NodeGroup Member is also able to force its NodeGroup Neighbors to forword its request to any available Gateway.

# Process of registration

## Add a new Member to an existing NodeGroup

### Step 1
If a new Member gets added to an existing NodeGroup, it requests a list of all existing SeedNodes by a Gateway. It communicate to this given SeedNodes and gives them all necassary information to operate within the NodeGroup. One or more SeedNode responses with all necassary information for the new added NodeGroup Member, such as the NodeGroups Seed Key and active addresses.

### Step 2
A Gateway gets the new added Member of the NodeGroup by a Status Refresh Request.

## Creation of a new NodeGroup

### Step 1
A new NodeGroup needs to be added to a Gateway. A new NodeGroup can not be created by a Node itself. The first Member of a new created NodeGroup must be a SeedNode. The Member of the created NodeGroup can request a Gateway for initial configuration. A new Iota seed key will be generated by the Member itself, not a Gateway.

### Step 2
A Gateway gets the new added Member of the NodeGroup by a Status Refresh Request.

## Getting Sensor Data
A Gateway keeps track of the prices of the sensor data.

