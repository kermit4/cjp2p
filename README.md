PEJOVU -  PEer-to-peer Json OVer Udp

A simple, interoperable, and expansible, p2p protocol, encouraged by https://farcaster.xyz/vitalik.eth/0xd6b8e141

# protocol 
JSON over UDP
## seen in the wild
- { message_type: "Who's around?"; }
- { message_type: "Who's around:"; 
    [
        { 
        IP: "1.2.3.4";
        port: 1234; },
    ]
}
- { message_type: "Please send content."; 
    content_sha256: "...";
    content_offset: 0;
    content_length: 1024; 
    }
- { message_type: "content:"; 
    content_sha256: "...";
    content_offset: 0;
    content: "<base64>"
}

## proposed:
- timestamp requests to learn most responsive hosts
- public keys in "whos around:"
- message_sha256: ".."; // calculate with this value empty
- [ JSON array of message support ] 
- { message_type: "try these others for the content"; content_sha256: "..."; peer_list: [ ip,port ... ] }
- chksums of the JSON in the JSON
- channels, like a stream but multiple senders
- encryption
- payments or accounting to insentivize resource sharing
- cookies so its not used for a DDOS, as people can spoof their source IPs
- chat
- chat message whitelisting to avoid spam
