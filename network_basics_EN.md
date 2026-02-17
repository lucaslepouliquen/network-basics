# Networking Fundamentals for DevOps Interview

## Table of Contents
1. [IP Addressing](#ip-addressing)
2. [DNS (Domain Name System)](#dns-domain-name-system)
3. [FQDN (Fully Qualified Domain Name)](#fqdn-fully-qualified-domain-name)
4. [Subnets and Subnet Masks](#subnets-and-subnet-masks)
5. [Gateway and Routing](#gateway-and-routing)
6. [NAT (Network Address Translation)](#nat-network-address-translation)
7. [Ports and Protocols](#ports-and-protocols)
8. [OSI Model](#osi-model)
9. [Load Balancing](#load-balancing)
10. [VPN and Tunneling](#vpn-and-tunneling)

---

## IP Addressing
An IP address (Internet Protocol) is a unique numerical identifier assigned to every device connected to a network. 
### IPv4
- **Format:** 4 octets separated by dots
- **Example:** `192.168.1.10`
- **Size:** 32 bits (2^32 = ~4.3 billion addresses)

### Address Classes

#### Private Addresses (RFC 1918)
- **Class A:** `10.0.0.0` - `10.255.255.255` (10.0.0.0/8)
- **Class B:** `172.16.0.0` - `172.31.255.255` (172.16.0.0/12)
- **Class C:** `192.168.0.0` - `192.168.255.255` (192.168.0.0/16)

**Usage:** Internal networks, not routable on the Internet

#### Public Addresses
All other addresses (routable on the Internet)

#### Special Addresses
- **Loopback:** `127.0.0.1` (localhost)
- **APIPA:** `169.254.0.0/16` (auto-configuration if no DHCP)
- **Broadcast:** `255.255.255.255`

### IPv6
- **Format:** 8 groups of 4 hexadecimal digits (16 octets)
- **Example:** `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- **Size:** 128 bits (2^128 addresses)
- **Short notation:** `2001:db8:85a3::8a2e:370:7334` (consecutive zeros omitted)

---

## DNS (Domain Name System)

### Principle
Resolves domain names to IP addresses

**Example:** `google.com` → `142.250.185.46`

### DNS Record Types

| Type | Description | Example |
|------|-------------|---------|
| **A** | Domain name → IPv4 | `example.com` → `93.184.216.34` |
| **AAAA** | Domain name → IPv6 | `example.com` → `2606:2800:220:1:...` |
| **CNAME** | Alias (name → name) | `www.example.com` → `example.com` |
| **MX** | Mail server | `example.com` → `mail.example.com` |
| **TXT** | Arbitrary text | SPF, DKIM, domain verification |
| **NS** | Name servers | `example.com` → `ns1.provider.com` |
| **PTR** | Reverse DNS (IP → name) | `93.184.216.34` → `example.com` |

### DNS Resolution Process

```
1. Client requests google.com
2. Check local cache
3. If absent, query DNS server (e.g., 8.8.8.8)
4. DNS responds with IP
5. Client connects to IP
```

### Common Public DNS Servers
- **Google:** `8.8.8.8`, `8.8.4.4`
- **Cloudflare:** `1.1.1.1`, `1.0.0.1`
- **OpenDNS:** `208.67.222.222`, `208.67.220.220`

By default, your machine uses the DNS servers provided by your ISP (Internet Service Provider) like Orange or Free in France. But you can configure alternative public DNS like Google's 8.8.8.8 for better performance.

---

## FQDN (Fully Qualified Domain Name)

### Definition
Complete domain name including hostname + domain

### Structure
```
web01.production.company.com
  |        |          |     |
  |        |          |     └─ TLD (Top Level Domain)
  |        |          └─────── Domain
  |        └────────────────── Subdomain
  └─────────────────────────── Hostname
```

### Examples
- **FQDN:** `api.prod.company.com`
- **Hostname only:** `api`
- **Domain:** `company.com`

### Usage
- Unique identification of a machine on the network
- DNS configuration
- SSL/TLS certificates
- Kubernetes Ingress

---

## Subnets and Subnet Masks

### Principle
Dividing an IP network into smaller subnetworks

### CIDR Notation (Classless Inter-Domain Routing)

**Format:** `IP_Address/Number_of_mask_bits`

**Examples:**
- `192.168.1.0/24` = 256 addresses (254 usable)
- `10.0.0.0/16` = 65,536 addresses
- `172.16.0.0/12` = 1,048,576 addresses

### Mask Calculation

| CIDR | Decimal Mask | Number of Hosts |
|------|--------------|-----------------|
| /32 | 255.255.255.255 | 1 (single host) |
| /31 | 255.255.255.254 | 2 (point-to-point link) |
| /30 | 255.255.255.252 | 4 (2 usable) |
| /29 | 255.255.255.248 | 8 (6 usable) |
| /28 | 255.255.255.240 | 16 (14 usable) |
| /24 | 255.255.255.0 | 256 (254 usable) |
| /16 | 255.255.0.0 | 65,536 |
| /8 | 255.0.0.0 | 16,777,216 |

### Practical Example

**Network:** `192.168.1.0/24`

- **Network address:** `192.168.1.0`
- **First usable IP:** `192.168.1.1`
- **Last usable IP:** `192.168.1.254`
- **Broadcast:** `192.168.1.255`
- **Gateway (typical):** `192.168.1.1`

### Subnetting - Example

**Base network:** `10.0.0.0/16` (65,536 IPs)

**Divide into 4 /18 subnets:**
1. `10.0.0.0/18` (10.0.0.0 - 10.0.63.255)
2. `10.0.64.0/18` (10.0.64.0 - 10.0.127.255)
3. `10.0.128.0/18` (10.0.128.0 - 10.0.191.255)
4. `10.0.192.0/18` (10.0.192.0 - 10.0.255.255)

---

## Gateway and Routing

### Gateway

**Definition:** Exit point of a local network to another network

**Function:**
- Route traffic between networks
- Usually the first IP of the subnet

**Example:**
```
Network: 192.168.1.0/24
Gateway: 192.168.1.1

Client 192.168.1.100 wants to reach 8.8.8.8
→ Sends packet to gateway 192.168.1.1
→ Gateway routes to Internet
```

### Default Gateway

**Definition:** Gateway used when destination is not on the local network

**Typical configuration:**
- **Machine IP:** `192.168.1.50`
- **Subnet mask:** `255.255.255.0`
- **Default gateway:** `192.168.1.1`
- **DNS:** `8.8.8.8`

### Routing Table

A routing table is a data structure stored on a router (or any network device) that lists the rules for deciding where to forward incoming packets.

**Example routing table:**

```
Destination       Gateway         Interface
0.0.0.0/0        192.168.1.1     eth0      (default route)
192.168.1.0/24   0.0.0.0         eth0      (local network)
10.0.0.0/8       192.168.1.254   eth0      (internal network)
```

### Useful Commands

**Linux/Mac:**
```bash
# View routing table
ip route show
route -n

# Add a route
ip route add 10.0.0.0/8 via 192.168.1.254

# Default route
ip route add default via 192.168.1.1
```

**Windows:**
```cmd
# View routing table
route print

# Add a route
route add 10.0.0.0 mask 255.0.0.0 192.168.1.254
```

---

## NAT (Network Address Translation)

### Principle
Translating private IP addresses to public IP address(es)

### How it Works

```
Internal network (private)      Internet (public)
192.168.1.10 ──┐
192.168.1.20 ──┼──> [NAT Router] ──> 203.0.113.5
192.168.1.30 ──┘    (51.15.45.78)
```

### NAT Types

#### 1. SNAT (Source NAT)
Changes the source IP of outbound packets

**Example:**
```
Internal: 192.168.1.10:54321 → 8.8.8.8:53
External: 203.0.113.5:12345 → 8.8.8.8:53
```

#### 2. DNAT (Destination NAT)
Changes the destination IP of inbound packets

**Example:** Port forwarding
```
Internet: Client → 203.0.113.5:80
Internal: Client → 192.168.1.100:80 (web server)
```

#### 3. PAT (Port Address Translation)
Multiple private IPs share 1 public IP via different ports

**This is the most common NAT in home routers**

### NAT in Kubernetes

**Service Type LoadBalancer:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80        # External port
    targetPort: 8080  # Pod port
```

The Load Balancer performs NAT: External port 80 → Pod port 8080

---

## Ports and Protocols

### Network Ports

**Definition:** Numerical identifier (0-65535) for services/applications

### Port Ranges

- **Well-known ports:** 0-1023 (require root privileges)
- **Registered ports:** 1024-49151
- **Dynamic/Private ports:** 49152-65535

### Common Ports to Know

| Port | Protocol | Service |
|------|----------|---------|
| 20-21 | FTP | File Transfer |
| 22 | SSH | Secure Shell |
| 23 | Telnet | Remote login (insecure) |
| 25 | SMTP | Email (send) |
| 53 | DNS | Domain Name System |
| 80 | HTTP | Unsecured web |
| 110 | POP3 | Email (receive) |
| 143 | IMAP | Email (receive) |
| 443 | HTTPS | Secured web |
| 3306 | MySQL | Database |
| 3389 | RDP | Remote Desktop (Windows) |
| 5432 | PostgreSQL | Database |
| 6379 | Redis | Cache/DB |
| 8080 | HTTP-alt | Alternative web |
| 27017 | MongoDB | Database |

### Transport Protocols

#### TCP (Transmission Control Protocol)
- **Connection:** Connection-oriented (handshake)
- **Reliability:** Guarantees delivery and order
- **Usage:** HTTP, HTTPS, SSH, FTP, databases

**3-way handshake:**
```
Client → SYN → Server
Client ← SYN-ACK ← Server
Client → ACK → Server
[Connection established]
```

#### UDP (User Datagram Protocol)
- **Connection:** Connectionless
- **Reliability:** No delivery guarantee
- **Usage:** DNS, streaming, gaming, VoIP
- **Advantage:** Faster than TCP

---

## OSI Model

### The 7 Layers

| Layer | Name | Function | Examples |
|-------|------|----------|----------|
| **7** | Application | User interface | HTTP, FTP, SMTP, DNS |
| **6** | Presentation | Data formatting | SSL/TLS, JPEG, ASCII |
| **5** | Session | Session management | NetBIOS, RPC |
| **4** | Transport | End-to-end communication | TCP, UDP |
| **3** | Network | Packet routing | IP, ICMP, OSPF |
| **2** | Data Link | Local communication | Ethernet, WiFi, MAC |
| **1** | Physical | Bit transmission | Cables, signals |

### Mnemonic (top to bottom)
**"All People Seem To Need Data Processing"**

### Practical Example: HTTP Request

```
7. Application:  GET /index.html HTTP/1.1
6. Presentation: SSL/TLS Encryption (if HTTPS)
5. Session:      Establish TCP session
4. Transport:    TCP segment (port 443)
3. Network:      IP packet (src/dst addresses)
2. Data Link:    Ethernet frame (MAC addresses)
1. Physical:     Bits on cable/WiFi
```

---

## Load Balancing

### Principle
Distributing traffic across multiple servers

### Distribution Algorithms

#### 1. Round Robin
Sequential distribution

```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A
...
```

#### 2. Least Connections
Send to server with fewest active connections

#### 3. IP Hash
Hash of client IP determines server
→ Same client = same server (session persistence)

#### 4. Weighted Round Robin
Round robin with weighting based on server capacity

### Load Balancing Layers

#### Layer 4 (L4) - Transport
- Based on IP/Port
- No content visibility
- Very fast
- Example: TCP/UDP load balancing

#### Layer 7 (L7) - Application
- Based on HTTP content (URL, headers, cookies)
- Intelligent routing
- Slower than L4
- Example: Nginx, HAProxy, Azure Application Gateway

### Health Checks

**Verify servers are available:**

```yaml
# Kubernetes example
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

### Cloud Examples

#### Azure Load Balancer (L4)
```
Internet → Azure LB → VM1, VM2, VM3
```

#### Azure Application Gateway (L7)
```
Internet → App Gateway → AKS pods
           ├─ /api/* → API pods
           └─ /web/* → Web pods
```

---

## VPN and Tunneling

### VPN (Virtual Private Network)

**Principle:** Create a secure private network over public Internet

### VPN Types

#### 1. Site-to-Site VPN
Connects two remote networks

```
Network A (Paris) ←─[VPN Tunnel]─→ Network B (London)
10.0.0.0/16                         172.16.0.0/16
```

#### 2. Point-to-Site VPN (Remote Access)
Remote user connects to corporate network

```
Employee laptop ←─[VPN]─→ Corporate Network
  (anywhere)                  (10.0.0.0/16)
```

### VPN Protocols

- **IPSec:** Standard, secure, complex
- **OpenVPN:** Open source, flexible
- **WireGuard:** Modern, fast, simple
- **L2TP/IPSec:** Protocol combination

### Tunneling

**Principle:** Encapsulate one protocol within another

**Example:** VPN tunnel
```
[Private data] → [Encryption] → [Public IP header] → Internet
```

### VPN in Azure

#### Azure VPN Gateway
```
On-premises Network ←─[Site-to-Site VPN]─→ Azure VNet
```

#### Azure Bastion
Secure access to VMs without public IP
```
User → Azure Bastion → Private VM (no public IP)
```

---

## Kubernetes Network Concepts

### Pod Networking

**Principle:** Each pod has its own IP

```
Node 1:
  Pod A: 10.244.0.5
  Pod B: 10.244.0.6

Node 2:
  Pod C: 10.244.1.5
  Pod D: 10.244.1.6
```

### Service Types

#### 1. ClusterIP (default)
Internal cluster IP only

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

#### 2. NodePort
Expose on a port of each node

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Accessible on <NodeIP>:30080
```

#### 3. LoadBalancer
Create a cloud load balancer (Azure LB, AWS ELB...)

```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

### Network Policies

**Control traffic between pods:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 3306
```

### Ingress

**External HTTP/HTTPS routing:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

---

## Network Troubleshooting

### Essential Commands

#### ping
Test IP connectivity

```bash
ping 8.8.8.8
ping google.com
```

#### traceroute / tracert
Trace packet path

```bash
traceroute google.com      # Linux/Mac
tracert google.com         # Windows
```

#### nslookup / dig
DNS queries

```bash
nslookup google.com
dig google.com
```

#### netstat / ss
Active network connections

```bash
netstat -tuln    # All listening ports
ss -tuln         # Modern version of netstat
```

#### curl / wget
HTTP testing

```bash
curl https://api.example.com/health
wget https://example.com/file.txt
```

#### telnet / nc (netcat)
Test connection on a port

```bash
telnet example.com 80
nc -zv example.com 443
```

### Kubernetes Troubleshooting

```bash
# Test connectivity from a pod
kubectl exec -it <pod-name> -- curl http://service-name

# View service endpoints
kubectl get endpoints <service-name>

# Pod logs
kubectl logs <pod-name>

# Describe service (see selectors)
kubectl describe service <service-name>

# Test DNS in cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
```

---

## Interview Scenarios - Example Responses

### Scenario 1: "A client cannot access the application"

**Methodical approach:**

1. **Check application layer:**
   - Is the service up? `kubectl get pods`
   - Do logs show errors? `kubectl logs`

2. **Check network layer:**
   - Does Kubernetes Service exist? `kubectl get svc`
   - Are endpoints correct? `kubectl get endpoints`
   - Does Load Balancer have public IP? `kubectl get svc -o wide`

3. **Check DNS:**
   - Does DNS resolve correctly? `nslookup app.example.com`
   - Is SSL certificate valid?

4. **Check connectivity:**
   - Ping public IP
   - Curl the endpoint
   - Check Network Policies
   - Check Security Groups/NSG

### Scenario 2: "High latency between two services"

**Debugging:**

1. **Determine if network or application:**
   ```bash
   # From pod A to service B
   kubectl exec -it pod-a -- time curl http://service-b
   ```

2. **Check locality:**
   - Are pods on same node?
   - Availability zone issue?

3. **Monitoring:**
   - Check Grafana/Prometheus metrics
   - Network latency between nodes
   - CPU/memory load of pods

4. **Solutions:**
   - Anti-affinity rules if need to separate
   - Affinity rules if need to bring closer
   - Scaling if overloaded

### Scenario 3: "How to secure traffic between pods?"

**Responses:**

1. **Network Policies:**
   - Restrict ingress/egress traffic
   - Principle of least privilege

2. **Service Mesh (Istio, Linkerd):**
   - Automatic mTLS between pods
   - Advanced traffic policies

3. **Private Endpoints:**
   - No public exposure
   - Traffic stays in VNet

4. **Encryption in transit:**
   - TLS between services
   - Managed certificates (cert-manager)

---

## Quick Glossary

| Term | Short Definition |
|------|------------------|
| **ACL** | Access Control List - filtering rules |
| **BGP** | Border Gateway Protocol - Internet routing |
| **DHCP** | Dynamic Host Configuration Protocol - auto IP assignment |
| **DMZ** | Demilitarized Zone - semi-secure network |
| **Firewall** | Filters network traffic |
| **ICMP** | Internet Control Message Protocol - used by ping |
| **MAC Address** | Unique physical address of network interface |
| **MTU** | Maximum Transmission Unit - max packet size |
| **QoS** | Quality of Service - traffic prioritization |
| **VLAN** | Virtual LAN - virtual network segmentation |
| **VPC** | Virtual Private Cloud - isolated network in cloud |

---

## Interview Checklist

**You should be able to explain:**

- Difference between public and private IP
- How DNS works
- What is an FQDN and give an example
- Calculate how many IPs in a /24
- What is a gateway and its role
- NAT principle and why we use it
- Difference TCP vs UDP
- Common ports (22, 80, 443, 3306...)
- Load balancing L4 vs L7
- Kubernetes Service types
- How to troubleshoot a network issue

**Commands to know:**
```bash
ping, traceroute, nslookup, curl, netstat
kubectl get svc/pods/endpoints, kubectl logs, kubectl describe
```
