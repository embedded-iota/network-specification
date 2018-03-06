# Specification for a network of embedded devices with IOTA as payment solution

## Preconditions

### Wording

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

### CAP
In sense of the CAP theorem, this specification prefers availability and partion tolerance over consistency. It is designed to be eventual consistent.

### Routing

This specification prefers Content Centric Networking for network purposes, but can also used with a traditional TCP/IP stack or other network-layer solutions like 6LoWPAN.

### Usage of the Specifications

This Specification is UNSTABLE. It SHALL not be used for production implementations.

## Naming Convention

The following naming convention SHALL be used in the implementation. The naming conevention within the implementation MUST be CamelCase or Underscore _.

### Node

A Node MUST be one physical device. A Node SHOULD be an embedded device.

#### Properties

###### Node Class

The Node Class MUST be a property on a Node and MUST be Sensor Node or Payment Node.

###### NodeType

A NodeType MUST be Seed Node or WorkerNode.

###### Node ID

Each Node MUST have its own independent Node ID within a Node Group.

#### Other

###### Sensor Node

A Sensor Node SHOULD be a small embedded device which MUST collect some data about the sourounding environment.

###### Payment Node

A Payment Node SHOULD be a small embedded device. It MUST accepts IOTA payments and MUST provide some service or access to goods.

###### Node Group

A Node Group MUST be group of Sensor- or Payment Nodes. They MUST fullfil the same or similar purpose. They MUST be within physical limited area.

###### Node Group Member

A Node Group Member MUST be a Node within a Node Group.

###### Seed Node

A Seed Node MUST be a Node within a Node Group. It MUST holds the configuration for new Node Group Members. It is also MUST be responsible for collecting the status of the Node Group for a Gateway.

###### WorkerNode

A WorkerNode MUST be Node Group Member which is not a Seed Nodes.


###### Node Group Neighbor

All other Nodes MUST be called Node Group Neighbor.

### Gateway

A Gateway MUST be on physical device. A Gateway SHOULD be a more powerful embedded device. It SHALL give a Node Group Member indirect access or direct to the tangle.

### Status Codes

###### HEALTHY

The Status HEALTHY MUST return by Node, if  everything operates like expected.

###### DEAD

The Status DEAD MUST return by a Nodes Node Group Neighbors, if they cannot contact this Node Group Member directly or indirect anymore.

###### BROKEN

The Status BROKEN MUST return by a Node, if the Node is still available but not able to fullfil its purpose.

### IOTA Tangle Network

###### IOTA Address

An IOTA Address MUST be the address which is used on the Iota tangle.

###### IOTA Transaction

An IOTA Transaction MUST be a transaction on the IOTA tangle. It SHOULD be a confirmed transaction. A IOTA Transaction MUST, at least, contain the following properties:
- Tag
- Message

###### IOTA Address Milestone

A IOTA Address Milestone MUST be an ID for a given Iota Address which SHOULD be set by a Gateway. It SHALL NOT be set by a Node Group Member.

###### Seed Key

A Seed Key is the private IOTA Seed which is used as base for generating IOTA addresses, transactions etc. Is known just as Seed in IOTA.

### Communication

#### Gateway

###### Health Status Request

A Health Status Request MUST be a regular request which a Gateway sends. It MUST be used by a Gatway to get the health of a Node Group.

###### Health Status Response

The Health Status Response MUST contain the following information:
- Status: BROKEN, HEALTHY or DEAD

###### Status Refresh Request

A Status Refresh Request MUST be a regular request which a Gateway sends. It MUST be used by a Gatway to get information about the Node Group.

###### Status Refresh Response

The Status Refresh Response MUST contain the following information:
- Current used addresses
- Prices of the services, sensor data or goods
- New added Member, if there were any between current status call and last one.


#### Nodes

###### IOTA Address Transactions Update Request

A IOTA Address Transactions Update Request SHALL be send by a Node which is interested in a new transaction. It MUST , at least, contain the following information:
- Latest Milestone

## Functional description

### Communication

Nodes SHALL communicate peer to peer within their Node Group. Nodes within one Node Group SHALL NOT be able to communicate with Members of another Node Group. Each Node Group Member MUST also able communicate to one or more Gateway.

##### Gateway <-> IOTA Tangle

A Gateway MUST subscribe to the transaction feed of all known IOTA Addresses. The Gatway MUST receive all of the incoming transactions to these IOTA Addresses. The Gateway MUST cache these transactions. A Gateway SHALL NOT forward these transactions directly to the Node Group. It MUST wait for a Address Transactions Update Request.

##### Gateway <-> Node Group Member communication

###### Health Status Requests

A Gatway SHOULD requests every x seconds the health status of a Node Group. If one Member doesn't respond, it MUST forces the other Node Group Members to get the status of this Node Group Neighbor. The Members MUST respond with their current saved status of this Node Group Member. If, at least, one Node Group Member responses with the status Healthy, the Member MUST NOT marked as Dead. If no Node Group Member responses with a Health status, the Gatway SHOULD retry the Health Status Request. The retry SHOULD have a timeout and it SHOULD be tried several times before the Node Group Member will marked as Dead.

###### Status Refresh Requests
A Gateway SHOULD requests every x seconds the status of a Node Group by its Seed Node. These requests MUST be called Status Refresh Requests. It MUST response one Seed Node. It is OPTIONAL that more Seed Nodes responses.


#### Seed Node <-> Seed Node communication

###### Seed Node Gossip

Every Seed Node MUST request the Seed Node Gossip on a regular base from the other Seed Nodes. It SHOULD do it every x seconds. The node which requests the newest Seed Node Gossip selects a random Seed Node Neighbor. This Seed Node Neighbor SHOULD respond to this request. If it doesn't respond to this request, SHOULD be marked as DEAD. The Seed Node Gossip Response MUST contain the following information:
- Latest known transactions since the last milestone.
- Latest known milestone
- Its own status
- New added Worker Nodes (a new Worker Node SHOULD be deleted from this response after x hours.)

If a Seed Node detected an issue on its side, it MUST response with a BROKEN status within the Seed Node Gossip.

The SeedNodes SHOULD retry contacting the not responding Seed Node. They SHOULD do it several times with a timeout. The Seed Node which is not responding MUST mark, eventually, as DEAD if it is not responding.

#### Node Group Member <-> Node Group Member communication

###### Node Group Member Gossip

Every Node Group Member MUST request the Node Group Member Gossip on a regular base. A Node which requests the Node Group Member Gossip MUST select a Node Group Neighbor randomly. This Node Group Neighbor SHOULD respond to this request. If it doesn't respond, it SHOULD be marked as DEAD. The Node Group Member Gossip MUST contains the following information:
- Latest known transactions since the last milestone.
- Latest known milestone.
- Its own status

If the Member detected an issue on its side, it MUST response with a BROKEN status within the Node Group Member Gossip.

The Node Group Members SHOULD retry contacting the not responding Node Member. They SHOULD do it several times with a timeout. The Node Group Member which is not responding MUST mark, eventually, as DEAD if it is not responding.

#### Worker Node <-> Seed Node communication

###### Worker Node Gossip

Every Worker Node MUST request a Seed Node on a regular base. The Seed Node MUST selected randomly. The Seed Node SHOULD respond. If the Seed Node does not respond, the Worker Node MUST select a new Seed Node randomly and request the new selected Seed Node. The response of a Seed Node MUST contain the following information:
- New Worker Nodes (a new Worker Node SHOULD be deleted from the list after x hours.)


#### Node Group Member Registration

###### Add a WorkerNode to an existing Node Group

If a new WorkerNode gets added to an existing Node Group, it MUST request a list of all existing Seed Nodes by a Gateway. It MUST communicate to this given Seed Nodes and MUST gives them all necassary information to operate within the Node Group. At least one Seed Node MUST response with all necassary information for the new added Node Group WorkerNode. At least one Seed Node MUST also response with a list of all existing WorkerNode of the Node Group.


###### Creation of a new Node Group

A new Node Group MUST be added to a Gateway. A new Node Group MUST NOT be created by a Node itself. The first Member of a new created Node Group MUST be a Seed Node. The Member of the created Node Group MUST request a Gateway for initial configuration.

###### Add a new Seed Node

Like the creation of a new Node Group, a Seed Node MUST be added to a Node Group through a Gateway. After adding a new Seed Node to a Gateway, the new added Seed Node MUST get the initial configuration by a Gatway. It MUST also register itself to all of its Node Group Neighbor Seed Nodes. It MUST provide all necessary information to operate within the Node Group. The already active Seed Nodes MUST receiving the request of the new Seed Node. They MUST request a Gatway for the new Node Group NodeSeed list. They MUST check if the new Seed Node is in this list. If the list doesn't contain the new Seed Node, they MUST retry requesting the seed list. They SHOULD do it x times. They SHOULD do it with a timeout. This time SHOULD be x seconds. If the Seed Node is still not in the list, they SHOULD ignore all of the new Seed Nodes requests.

#### Seed Keys

A Seed Key MUST generated by the node itself. It SHALL NOT share this Seed with other nodes. It MUST keep this Seed Key secure in its storage. It MUST also keep track of the current used IOTA Address' index.


#### Receiving an IOTA Address' transactions

The Node Group Member which is interested in an incoming transaction MUST request an addresses transactions since a specific Address Milestone. The response MUST contains all new available transactions since the given Address' Milestone. The latest transaction SHALL be marked with the latest Milestone. To get the IOTA Addresses' transactions, the node SHOULD request a Gateway. It is OPTIONAL to request also the Node Group Neighbors. The Member which is intersted in an incoming transaction SHOULD use an timeout and SHOULD request the addresses transactions again, if it is still waiting for it. A Node Group Member SHOULD also be able to force its Node Group Neighbors to forword its request to any Gateway. The Gateway or Node Group Neighbors response MUST contain the latest Address Milestone and all transactions between the provided Milestone and the latest. If the Node Group Member wasn't the initator of the request, it MUST forwords the information to the Node Group Member which requested the transactions and MUST updates its own transaction database.


#### Receiving new addresses


### Access Data, Services or Goods

#### Receiving Payments as Payment Node



