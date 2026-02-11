CJP2P - Connectionless JSON P2P Protocol

A connectionless, simple, interoperable, expansible, p2p protocol, inspired by https://farcaster.xyz/vitalik.eth/0xd6b8e141 and https://medium.com/@webseanhickey/the-evolution-of-a-software-engineer-db854689243

# as seen in the wild 

Implement or extend as much or as little as you like, or something new!   

## protocol 
JSON array of message

## message types 
```JSON
{"PleaseSendPeers":{}}
{"Peers":{
      "peers":[
          "148.71.89.128:43344",
          "148.71.89.128:50352"] } }
{"PleaseSendContent":{
      "id":"8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4",
      "length":4096,
      "offset":0 }}
{ "Content": { 
      "id":"8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4",
      "base64": "aGk=",
      "eof": 2,           // this is the file length, not a boolean, so for files will always have the same value.  For streams it may change.
      "offset":0 } }
{"PleaseReturnThisMessage":{...}}   
{"ReturnedMessage":{...}}  // probably for timestamping, maybe other things, like an HTTP cookie.   Send it all back with your reply, if there is any reply to send.
{"MaybeTheyHaveSome":{"id":"foo",
      "peers":[ "148.71.89.128:43344", "148.71.89.128:50352"] } } // suggest where else to look for foo, this is likely in reply to PLeaseSendContent
}}   
```

## implementations
https://github.com/kermit4/cjp2p-rust (implements everything listed above and by most the most deveoped)
https://github.com/kermit4/cjp2p-bash (most features, slow transfers, but good for ideas and easier to read than Rust for many)
https://github.com/kermit4/cjp2p-ruby (most feature very fast development cycle, but not very intelligent) 
https://github.com/kermit4/cjp2p-haskell (very few features)

## likely to be running nodes
148.71.89.128:24254
159.69.54.127:24254

# development hints:
  echo -n '[{"PleaseSendPeers":{}}]' |nc -u localhost 24254

  tcpdump -As 9999 -i any port 24254

The file sharing is more of a primitave than a main purpose, providing applications a way to reliably receive data of arbitrary size from many peers.

The protocol should sound more like people than computers.   Simple requests, share a lot, expect little, be tolerant -- you're talking to strangers using automation, not computers.  Prefer to leave decisions up to implementations.  It's a language for ordinary people using automation.  Everyone starts somewhere, keep it accessable to any programming skill level, with more advanced features optional (or not, it's up to you on your node and implementation).

pay attention to unhandled messages and try to handle them, or make your own -- you don't have to wait for some official protocol update or new messages or new fields.

Telegram group: https://t.me/cjp2p

## test files available (under both their sha256 hash and name)
- c3514bf0056180d09376462a7a1b4f213c1d6e8ea67fae5c25099c6fd3d8274b ubuntu-24.04.3-live-server-amd64.iso
- c74833a55e525b1e99e1541509c566bb3e32bdb53bf27ea3347174364a57f47c ubuntu-24.04.3-wsl-amd64.wsl
- 1M d8b778285d0006ac17839bcded0fb9bd5dc9cbc8e869adb7b9bbea31efa8070e
- 2M 39d0e0e08bda0113b570b2486127fcfaaa18c7c47d389b9ecb27b2b863750671
- 4M e0f0b3c745acbf7631d1e98153e406045bacea2f3dc2ea310c1b82ab0c23e471
- 8M 5b6656f16181bc0689b583d02b8b8272a02049af3ba07715c4a6c08beef814c2
- 16M 7caacb04f205faf47a8d55ea7c3c6b642377b850d970f7df5233f213415829d2
- 32M 24349fedc2836f75e58b199c748e6fb1808136bb8ab9f618a264c64ce735fa5b
- 64M 35fd7b1f88666d3156d32fa89b0bb0930b3a8eb86dd711d0fe277f45b465791f
- 128M e1c4691d6cc8f2638250127beaadeb1b3d041c6ba877cfb5e551bb9da2f63303
- 256M cb407d7355bb63929d7f4b282684f5a2884a0c3fb73d56642455600569a6888b
- 512M 6f5a06b0a8b83d66583a319bfa104393f5e52d2c017437a1b425e9275576500c
- 1024M c7dce40a2af023d2ab7d4bc26fac78cba7f7cb7854f67f9fb5bf72b14d9931d8
- 

# future possibilities

## protocol ideas:
- checksums of the message array
- "hi" messages just to keep peers active in peer lists?   3 way with timestamps?
- timestamp requests to learn most responsive service from your location (and some protocol that replies return these timestamps)
- some way to prevent being used as a DDOS by replying to a spoofed IP (some sort of cookie/handshake..calculated by a local random, to not need to remember them all)
- need checksums of blocks before complete, confirmed data before sending it on, otherwise a bad bit may copy around forever, breaking a file
- public keys in "Receive peers."
- streams, i.e. files that keep growing..maybe just make eof an optional field
- channels, like a stream but multiple senders
- encryption
- economics to incentivize resource sharing
- chat
- chat message white listing to avoid spam

## test files

  
