p2p framework.

# objectives:

## connectionless messaging (UDP not TCP)
    . to not be limited by socket overhead
    . nor duplicating all the TCP features at OS level which you wind up doing in the application anyway (retransmission, checksumming, acknowledgement, timeouts) interacting to rather unpredictable behaviour (more lenghty rant about misuse of TCP at https://159.69.54.127/tcp_udp.md )
    . to be more latency aware at the application level to chose peers more accurately based on real results not out of band surveys.
    . to be default routed, no relaying needed when not limited to some number of "connections" -- leave routing to the network layer
    
## prefer expansability, compatibility, and readability, over premature optimization:
    . serialization format is JSON

# currently implimented
##    file distribution
### message types [ ]
{ message_type: "Please send this content."; 
    content_sha256: "...";
    content_offset: 0;
    content_length: 1024; // 4k is probably OK, may or may not be more efficient. Over 40k probably won't work.
    }
{ message_type: "The content you requested:"; 
    content_sha256: "...";
    content_offset: 0;
    content: "in base 64..no premature optimization here. if its reallly a bottleneck for you, this is easily extensible."
}
{ message_type: "Who's around?"; }
{ message_type: "Who's around:"; }

# FUTURE
message_sha256: ".."; // calculate with this value empty
[ JSON array of message support ] 
{ message_type: "try these others for the content"; content_sha256: "..."; peer_list: [ ip,port ... ] }
chksums of the JSON in the JSON
channels, like a stream but multiple senders
encryption
payments or accounting to insentivize resource sharing
cookies so its not used for a DDOS
chat
chat rate limit for unknown senders (optionally to 0)
