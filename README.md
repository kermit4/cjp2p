CJP2P - Connectionless JSON P2P Protocol

A connectionless, simple, interoperable, expansible, p2p protocol, inspired by https://farcaster.xyz/vitalik.eth/0xd6b8e141 and https://medium.com/@webseanhickey/the-evolution-of-a-software-engineer-db854689243

# as seen in the wild 


## protocol 
JSON array of messages.   Please make a PR into here if you spot any new fields or messages.

## message types 
### MUST implement
#### Like an HTTP Cookie, send it back with any message to the node that provided it.  
```JSON
{"PleaseAlwaysReturnThisMessage":["any",{"valid":"JSON"}]}
{"AlwaysReturned":               ["any",{"valid":"JSON"}]}  
```
### SHOULD implement
#### Peer discovery
```JSON
{"PleaseSendPeers":{}}
{"Peers":{
      "peers":[
          "148.71.89.128:43344",
          "148.71.89.128:50352"] } }

{"PleaseReturnThisMessage":["any",{"valid":"JSON"}]}
{"ReturnedMessage":        ["any",{"valid":"JSON"}]}  
```

### MAY implement
#### a basic use. EOF refers to the full completed length of the content.  
```JSON
{"PleaseSendContent":{
      "id":"8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4",
      "length":4096,
      "offset":0 }}
{ "Content": { 
      "id":"8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4",
      "base64": "aGk=",
      "eof": 2,           
      "offset":0 } }
{"MaybeTheyHaveSome":{
       "id":"foo",
      "peers":[ "148.71.89.128:43344", "148.71.89.128:50352"] } }

```

## implementations
- https://github.com/kermit4/cjp2p-rust (implements everything listed above and far the most developed and intelligent)
- https://github.com/kermit4/cjp2p-ruby (most protocol features, but not intelligent)
- https://github.com/kermit4/cjp2p-bash (most protocol features, but not intelligent, slow transfers)
- https://github.com/kermit4/cjp2p-haskell (very few features)

## likely to be running nodes
148.71.89.128:24254
159.69.54.127:24254

# development hints:
  echo -n '[{"PleaseSendPeers":{}}]' |nc -u localhost -p 12321 24254

  tcpdump -As 9999 -i any port 24254

The file sharing is more of a primitive than a main purpose, providing applications a way to reliably receive data of arbitrary size from many peers.

The protocol should sound more like people than computers.   Simple requests, share a lot, expect little, be tolerant -- you're talking to strangers using automation, not computers.  Prefer to leave decisions up to implementations.  It's a language for ordinary people using automation.  Everyone starts somewhere, keep it accessable to any programming skill level, with more advanced features optional (or not, it's up to you on your node and implementation).

pay attention to unhandled messages and try to handle them, or make your own -- you don't have to wait for some official protocol update to add messages or fields, just don't crash if you receive some.

Telegram group: https://t.me/cjp2p

## test files available (under both their SHA256 hash and name, though the Rust implementation expects it to be a SHA256)
- c3514bf0056180d09376462a7a1b4f213c1d6e8ea67fae5c25099c6fd3d8274b ubuntu-24.04.3-live-server-amd64.iso
- c74833a55e525b1e99e1541509c566bb3e32bdb53bf27ea3347174364a57f47c ubuntu-24.04.3-wsl-amd64.wsl
- d8b778285d0006ac17839bcded0fb9bd5dc9cbc8e869adb7b9bbea31efa8070e 1M
- 39d0e0e08bda0113b570b2486127fcfaaa18c7c47d389b9ecb27b2b863750671 2M
- e0f0b3c745acbf7631d1e98153e406045bacea2f3dc2ea310c1b82ab0c23e471 4M
- 5b6656f16181bc0689b583d02b8b8272a02049af3ba07715c4a6c08beef814c2 8M
- 7caacb04f205faf47a8d55ea7c3c6b642377b850d970f7df5233f213415829d2 16M
- 24349fedc2836f75e58b199c748e6fb1808136bb8ab9f618a264c64ce735fa5b 32M
- 35fd7b1f88666d3156d32fa89b0bb0930b3a8eb86dd711d0fe277f45b465791f 64M
- e1c4691d6cc8f2638250127beaadeb1b3d041c6ba877cfb5e551bb9da2f63303 128M
- cb407d7355bb63929d7f4b282684f5a2884a0c3fb73d56642455600569a6888b 256M
- 6f5a06b0a8b83d66583a319bfa104393f5e52d2c017437a1b425e9275576500c 512M
- c7dce40a2af023d2ab7d4bc26fac78cba7f7cb7854f67f9fb5bf72b14d9931d8 1024M
- 8e008973582673665a326cc44c681c11d9d39ec61dd529f3c1aa26695f4880e7 0x10001 bytes (one byte more than 64k)

- 80dfa275db78819616ae2cf361c39323d3b2d69b565dfea5706eed6a29aad352  1024 sha256sums of each 1M block of 1024M, which are separately downloadable -- this is fun: timeout 4 ./target/debug/cjp2p-rust                                         80dfa275db78819616ae2cf361c39323d3b2d69b565dfea5706eed6a29aad352 ; ./target/debug/cjp2p-rust $(cat                                                shared/80dfa275db78819616ae2cf361c39323d3b2d69b565dfea5706eed6a29aad352 )
- a40e24319477590fdcad751a76dca92e542f0134f6dd93582decd1557d2676ad  1024 sha256sums of each 256k block of 256M, which are separately downloadable


# future possibilities

## protocol ideas:
- public keys in "PeerInfo"
- large files as a series of sha256sums of 256K blocks. so parts can be shared before its all done, and errors dont cause a complete retransfer.
- streams, i.e. a running series of sha256sums which are fetched with PleaseSendContent
- channels, like a stream but multiple senders, a concensus on it (like a blockchain)
- channels, like a stream but multiple senders, just open to anyone (could get noisy)
- encryption
- economics to incentivize resource sharing
- chat
- chat message white or black listing to avoid spam, and sharing the lists

