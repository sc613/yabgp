
# Final Project - BGP
## Spring 2023 Topics in Internet Security

[Presentation slides](https://github.com/sc613/yabgp/blob/master/slides.pdf)

[yabgp README](https://github.com/sc613/yabgp/blob/master/slides.pdf)

### Additional installations required besides yabgp
- (apt) `netcat curl`
- (pip) `cryptography==35.0.0`

### Demonstration environment
- BGP client implementation: yabgp ([github](https://github.com/smartbgp/yabgp))
  - Not usable as a BGP server
  - But runs REST API server at port 8801 to send UPDATE requests to peer
- 3 docker containers running Ubuntu 20.04
  - docker run -it --privileged ubuntu:20.04
- 1 container (#3) for relaying TCP traffic (port 179)
  - ASN assumed to be 65003
  - Runs only nc -l 179 < fifo | nc -l 179 > fifo
  - The two containers establish connection to each one of netcat processes
- 2 containers for BGP peers
  - (Container #1) `python yabgp --bgp-local_addr=<container #1 IP> --bgp-local_as=65001 --bgp-remote_addr=<container #3 IP> --bgp-remote_as=65002`
  - (Container #2) `python yabgp --bgp-local_addr=<container #2 IP> --bgp-local_as=65002 --bgp-remote_addr=<container #3 IP> --bgp-remote_as=65001`
- The three containers are connected to the default docker bridge network

### Additional directories
- [key](https://github.com/sc613/yabgp/tree/master/key): stores relevant RSA private/public keys and signatures
  - `*_certificate`: byte array of (ASN, prefix, signature on (ASN, prefix))
  - `*_origin_signature`: signature on (origin ASN, origin NLRI)
  - `root_pubkey.pub`: public key of "root CA"
- [prefixes](https://github.com/sc613/yabgp/tree/master/prefixes): stores reachable prefixes for each AS
  - One prefix for each line

### Command to send UPDATE message to container #2 via POST request
```
curl -u admin:admin -X POST -H "Content-Type: application/json" -d '{"attr":{"1":1,"2":[[2,[65003,65001]]],"3":"172.17.0.2"},"nlri": ["1.1.1.0/24"],"origin_msg":{"asn":65003,"nlri":["1.1.1.0/24"]}}' http://<container #1 IP>:8801/v1/peer/<container #3 IP>/send/update
```
