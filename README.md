[![npm](https://img.shields.io/npm/v/raam.client.js.svg)](https://www.npmjs.com/package/raam.client.js)

# RAAM - Random Access Authenticated Messaging
RAAM is a second layer data communication protocol for IOTA
enableing protected datastream access and publishing, organized in so called channels.

RAAM uses the same quantum proof signing scheme and hash function used in IOTA to sign transactions. 
These techniques enable the construction of secure data channels providing data integrity and 
authorship authentication. Furthermore the data is encrypted and, hence it is stored on the tangle, 
immutable. By using optional passwords for a channel or for specific messages reading access can be limited
to a specific private audience. RAAM can be used without any changes to IOTA nodes. Each message strenthens
the IOTA network, because RAAM messages at their core are a set of zero-value transactions, confirming other 
transactions on the tangle.

The messages in a channel don't have to be accessed from first to last, but can be accessed in any random order in O(1).
For that only the channel id and the index of the message are needed.

**Features**
- [x] indexed messages
- [x] access of arbitrary messages in O(1)
- [x] authentication (proof of authorship) and spam protection of channels
- [x] 4 different security levels
- [x] private mode with channel password
- [x] public mode with finding messages by address
- [x] encrypting different messages with different passwords
- [x] subscribing to new messages in channel
- [x] constructing messages and publishing them later
- [x] channel branching

RAAM enables messaging for a variety of use cases which need privacy and integrity for data communication. This includes
M2M communication for the IoT in consumer electronics as well as in machines in industrial contexts, such as
autonomous data marketplaces, supply chains, mobility and smart cities.

This javascript library acts as a reference implementation showcasing the specified abilities of the protocol. 

## Basic usage
After downloading and importing the library into your project it will provide access to all functions for reading and 
writing from/to RAAM channels.

**Generating a new channel and publishing a message**  
```js
const RAAM = require('raam.client.js')
const iota = require('@iota/core').composeAPI({
    provider: 'https://nodes.devnet.iota.org'
})
const seed = "DONTGIVEYOURSEEDTOANYBODYELSEDONTGIVEYOURSEEDTOANYBODYELSEDONTGIVEYOURSEEDTOANYBODYELSE"
const raam = await RAAM.fromSeed(seed, {height: 4, iota})

await raam.publish("HELLOIOTA")
```

**Reading from a channel**
```js
const { RAAMReader } = require('raam.client.js')
const channelId = "TIGXUEKKCGTOPNXEIGUYQCJUCODSVAXVHZRARCWRAOVCZKN9WDILGKRIDAXBJSACGDWTTVBEOIZHQTSYX"
const raam = new RAAMReader(channelId, {iota})
let response = await raam.fetch({index: 3})
console.log(response.messages[0])

response = await raam.fetch({start: 0, end: 2})
console.log(response.messages)
```

Take a look at the [API Reference](docs/api.md) to learn more.

## How it works
Since the winternitz signing scheme used in IOTA creates one time signatures, you need multiple signing keys for 
multiple messages. A reader can verify the integrity of a message by using the verifying key included in the message.

A reader can also verify, that a message has the same author than all other messages in the channel, which
is called authentication. For that, RAAM uses a merkle tree signing scheme, where the verifying keys of all messages in 
a channel are the leafs of the tree. From all verifying keys the root of the tree, the merkle root, is constructed, which 
acts as the id for a RAAM channel. 

Therefore, someone who publishes a message in a RAAM channel must not only possess the key that signed this message,
but all other signing keys for the channel aswell. To authenticate the authorship of a message the merkle root is 
reconstructed by using the verifying key of the message and other parts of the tree, which are provided aswell. 
Since the merkle root is generated by hashing, it is impossible to reconstruct certain leafs (verifying keys) from 
a given merkle root, the same way it is impossible to forge a signing key from a given verifying key. This way it is easy
to ensure that two different messages in the same channel belong to the same author.

Because of that the maximum amount of messages that can be published in a channel depends on the size of a merkle tree, 
which has to be created in advance.

* * *

&copy; 2018 Robin Lamberti \<lamberti.robin@gmail.com\>.