PEJOVU -  PEer-to-peer Json OVer Udp

A simple, interoperable, expansible, p2p protocol, inspired by https://farcaster.xyz/vitalik.eth/0xd6b8e141 and https://medium.com/@webseanhickey/the-evolution-of-a-software-engineer-db854689243

# protocol 
JSON array of messages, sent over UDP

Please return any cookies you receive in your responses. { .... cookie: ... };
## message types seen in the wild
- { message_type: "These are peers."; 
    [ "1.2.3.4:1234", ... ]
}
- { message_type: "Please send content."; 
    content_id: "...";
    content_offset: 0;
    content_length: 1024; 
    }
- { message_type: "Here is content."; 
    content_id: "...";
    content_offset: 0;
    content_b64: "<base64>"
}

## message type ideas:
- timestamp requests to learn most responsive service from your location
- public keys in "Receive peers."
- message_type hash of me  hash type sha2526 sha256: ".."; // checksum the json array string     so far before appending this object
- { message_type: "try these others for the content"; content_id: "..."; peer_list: [ ip,port ... ] }
- chksums of the JSON in the JSON
- channels, like a stream but multiple senders
- encryption
- economics to insentivize resource sharing
- chat
- chat message whitelisting to avoid spam

# currently available
## implementations
https://github.com/kermit4/pejovu-rust

## likely to be running nodes
148.71.89.128:24254
159.69.54.127:24254


## test files
faabcf33ae53976d2b8207a001ff32f4e5daae013505ac7188c9ea63988f8328 *ubuntu-24.04.3-desktop-amd64.iso
c3514bf0056180d09376462a7a1b4f213c1d6e8ea67fae5c25099c6fd3d8274b *ubuntu-24.04.3-live-server-amd64.iso
c74833a55e525b1e99e1541509c566bb3e32bdb53bf27ea3347174364a57f47c *ubuntu-24.04.3-wsl-amd64.wsl
1M d8b778285d0006ac17839bcded0fb9bd5dc9cbc8e869adb7b9bbea31efa8070e
2M 39d0e0e08bda0113b570b2486127fcfaaa18c7c47d389b9ecb27b2b863750671
4M e0f0b3c745acbf7631d1e98153e406045bacea2f3dc2ea310c1b82ab0c23e471
8M 5b6656f16181bc0689b583d02b8b8272a02049af3ba07715c4a6c08beef814c2
16M 7caacb04f205faf47a8d55ea7c3c6b642377b850d970f7df5233f213415829d2
32M 24349fedc2836f75e58b199c748e6fb1808136bb8ab9f618a264c64ce735fa5b
65M 35fd7b1f88666d3156d32fa89b0bb0930b3a8eb86dd711d0fe277f45b465791f



