PEJOVU -  PEer-to-peer Json OVer Udp

A connectionless, simple, interoperable, expansible, p2p protocol, inspired by https://farcaster.xyz/vitalik.eth/0xd6b8e141 and https://medium.com/@webseanhickey/the-evolution-of-a-software-engineer-db854689243

# current state as seen in the wild
## protocol 
JSON array of messages, sent over UDP

## message types 
- { "message_type": "Please send peers." }
- { "message_type":"These are peers.","peers":[
    "148.71.89.128:43344",
    "148.71.89.128:50352"] } 
- { "message_type": "Please send content.",
     "content_id":"8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4",
      "content_length":4096,
      "content_offset":0 }
- { "message_type": "Here is content.",
    "content_id":"8f434346648f6b96df89dda901c5176b10a6d83961dd3c1ac88b59b2dc327aa4",
    "content_b64": "aGk=",
    "content_eof": 2,
    "content_offset":0 }

## implementations
https://github.com/kermit4/pejovu-rust

## likely to be running nodes
148.71.89.128:24254
159.69.54.127:24254

# development hints:
  (echo -n '[{"message_type":"Please send peers."}]' ;read)|nc -u 148.71.89.128 24254 

  tcpdump -As 9999 -i any port 24254

pay attention to unhandled messages and try to handle them, or make your own, you don't have to wait for some official protocol update or new messages or new fields.

Telegram group: https://t.me/pejovu


## test files
- faabcf33ae53976d2b8207a001ff32f4e5daae013505ac7188c9ea63988f8328 *ubuntu-24.04.3-desktop-amd64.iso
- c3514bf0056180d09376462a7a1b4f213c1d6e8ea67fae5c25099c6fd3d8274b *ubuntu-24.04.3-live-server-amd64.iso
- c74833a55e525b1e99e1541509c566bb3e32bdb53bf27ea3347174364a57f47c *ubuntu-24.04.3-wsl-amd64.wsl
- 1M d8b778285d0006ac17839bcded0fb9bd5dc9cbc8e869adb7b9bbea31efa8070e
- 2M 39d0e0e08bda0113b570b2486127fcfaaa18c7c47d389b9ecb27b2b863750671
- 4M e0f0b3c745acbf7631d1e98153e406045bacea2f3dc2ea310c1b82ab0c23e471
- 8M 5b6656f16181bc0689b583d02b8b8272a02049af3ba07715c4a6c08beef814c2
- 16M 7caacb04f205faf47a8d55ea7c3c6b642377b850d970f7df5233f213415829d2
- 32M 24349fedc2836f75e58b199c748e6fb1808136bb8ab9f618a264c64ce735fa5b
- 65M 35fd7b1f88666d3156d32fa89b0bb0930b3a8eb86dd711d0fe277f45b465791f
- 

# future possibilities

## protocol ideas:
- checksums of the message array
- "hi" messages just to keep peers active in peer lists?   3 way with timestamps?
- timestamp requests to learn most responsive service from your location (and some protocol that replies return these timestamps)
- some way to prevent being used as a DDOS by replying to a spoofed IP (some sort of cookie/handshake..calculated by a local random, to not need to remember them all)
- need sub-hashes otherwise a bad bit may copy around forever, breaking a file
- public keys in "Receive peers."
- include suggestions as to where else to request content in replies to Please send content.
- streams, i.e. files that keep growing
- channels, like a stream but multiple senders
- encryption
- economics to insentivize resource sharing
- chat
- chat message whitelisting to avoid spam

## test files

  
