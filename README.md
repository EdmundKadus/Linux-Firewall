

# Objectives

## 1. Basic Firewall Configuration

### Creating Custom Chains  

The `-N` option adds a new chain to the pre-existing chains (`INPUT`, `OUTPUT`, `FORWARD`).
  ~~~~
  iptables -N custom-chain
  ~~~~

  ![image](https://github.com/user-attachments/assets/c2522ea0-d339-423f-83e6-99f75dd582f8)
  

  We then append `-N` a rule to direct ICMP packets to the custom chain: 
  ~~~~
  iptables -A INPUT -p icmp -j custom-chain
  ~~~~
  To prevent packet hanging, we should append a return jump (-j RETURN) at the end of the chain, ensuring proper flow to netfilter.

  ![image](https://github.com/user-attachments/assets/9254dc95-7183-437e-8e02-e15a8fd3880f)

### Deleting custom chains

  To remove a custom chain, first, delete the rule referencing it. If CHAIN DEL fails, it means some related rules are still present and must be removed before the chain can be deleted.
  ~~~~
  iptables -D INPUT -p icmp -j custom-chain
  ~~~~

  Once all references are removed, delete the chain itself:
  ~~~~
  iptables -D INPUT -p icmp -j custom-chain
  ~~~~

    

## 2. Filtering traffic by IP Address and TCP Ports

### Rejecting all ICMP to 127.0.0.1
  Like most standard firewall we are able to state both source and destination IP addresses along with the ports that are allowed for the input chain.
  ~~~~
  iptables -D INPUT -p icmp -s 127.0.0.1 REJECT
  ~~~~
No pings are exchanged

![Screenshot 2025-02-02 005704](https://github.com/user-attachments/assets/42fbffc0-d19d-4f00-9eed-d883e4860d3b)

### Using the LOG chain for error logs

 ![Screenshot 2025-02-02 010145](https://github.com/user-attachments/assets/8fa1c36c-f4a2-4f6b-a74d-e12dc7ecaad3)

We can use grep `dmesg | grep "ICMP packet dropped: "` to search for the dropped packets which contains info such as source IP address & TTL(Time to live) along with the protcols used.

![Screenshot 2025-02-02 010206](https://github.com/user-attachments/assets/09bee28f-7561-4932-b44e-9a8808047044)

### Setting up a python web server for port testing

I played around with `nmap` and tried it on port 80 and it returns with a closed state.

![Screenshot 2025-02-02 013346](https://github.com/user-attachments/assets/7b86842b-d08d-4734-a530-8ec4170453cb)

Using `netstat` I can see that there is no active process listening on port 80, I will set up a python HTTP server on that port to test out the rules later.

![Screenshot 2025-02-02 013942](https://github.com/user-attachments/assets/43cae790-27b4-408e-86bb-e23237a62cfb)

Having done that, we can see that the port is up and running.

![Screenshot 2025-02-02 014052](https://github.com/user-attachments/assets/4c2e3671-0058-446d-8d2c-61a28dfaf6a7)

### Dropping TCP port 80 (HTTP) on 127.0.0.1

Here we specify 80 on the `--dport` option and pointing to DROP
  ~~~~
  iptables -A INPUT -p tcp --dport 80 -s 127.0.0.1 -j DROP
  ~~~~
Results:

![Screenshot 2025-02-02 014359](https://github.com/user-attachments/assets/a0cd00c7-8b8f-414a-97b0-8fcedcbb2dc1)

I observed that using either DROP or REJECT we will get a filtered state with `nmap`

Dropped logs:

![Screenshot 2025-02-02 014640](https://github.com/user-attachments/assets/4a483b35-025c-47d7-9c6b-6bcae618c792)
