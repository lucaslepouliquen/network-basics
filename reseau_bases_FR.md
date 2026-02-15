# Bases Réseau pour Entretien DevOps

## Table des matières
1. [Adressage IP](#adressage-ip)
2. [DNS (Domain Name System)](#dns-domain-name-system)
3. [FQDN (Fully Qualified Domain Name)](#fqdn-fully-qualified-domain-name)
4. [Subnets et Masques de sous-réseau](#subnets-et-masques-de-sous-réseau)
5. [Gateway et Routing](#gateway-et-routing)
6. [NAT (Network Address Translation)](#nat-network-address-translation)
7. [Ports et Protocoles](#ports-et-protocoles)
8. [Modèle OSI](#modèle-osi)
9. [Load Balancing](#load-balancing)
10. [VPN et Tunneling](#vpn-et-tunneling)

---

## Adressage IP

### IPv4
- **Format:** 4 octets séparés par des points
- **Exemple:** `192.168.1.10`
- **Taille:** 32 bits (2^32 = ~4.3 milliards d'adresses)

### Classes d'adresses

#### Adresses Privées (RFC 1918)
- **Classe A:** `10.0.0.0` - `10.255.255.255` (10.0.0.0/8)
- **Classe B:** `172.16.0.0` - `172.31.255.255` (172.16.0.0/12)
- **Classe C:** `192.168.0.0` - `192.168.255.255` (192.168.0.0/16)

**Usage:** Réseaux internes, non routables sur Internet

#### Adresses Publiques
Toutes les autres adresses (routables sur Internet)

#### Adresses Spéciales
- **Loopback:** `127.0.0.1` (localhost)
- **APIPA:** `169.254.0.0/16` (auto-configuration si pas de DHCP)
- **Broadcast:** `255.255.255.255`

### IPv6
- **Format:** 8 groupes de 4 chiffres hexadécimaux
- **Exemple:** `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- **Taille:** 128 bits (2^128 adresses)
- **Notation courte:** `2001:db8:85a3::8a2e:370:7334` (zéros consécutifs omis)

---

## DNS (Domain Name System)

### Principe
Résolution de noms de domaine en adresses IP

**Exemple:** `google.com` → `142.250.185.46`

### Types d'enregistrements DNS

| Type | Description | Exemple |
|------|-------------|---------|
| **A** | Nom de domaine → IPv4 | `example.com` → `93.184.216.34` |
| **AAAA** | Nom de domaine → IPv6 | `example.com` → `2606:2800:220:1:...` |
| **CNAME** | Alias (nom → nom) | `www.example.com` → `example.com` |
| **MX** | Serveur mail | `example.com` → `mail.example.com` |
| **TXT** | Texte arbitraire | SPF, DKIM, vérifications domaine |
| **NS** | Serveurs de noms | `example.com` → `ns1.provider.com` |
| **PTR** | Reverse DNS (IP → nom) | `93.184.216.34` → `example.com` |

### Processus de résolution DNS

```
1. Client demande google.com
2. Check cache local
3. Si absent, requête au serveur DNS (ex: 8.8.8.8)
4. DNS répond avec l'IP
5. Client se connecte à l'IP
```

### Serveurs DNS publics courants
- **Google:** `8.8.8.8`, `8.8.4.4`
- **Cloudflare:** `1.1.1.1`, `1.0.0.1`
- **OpenDNS:** `208.67.222.222`, `208.67.220.220`

---

## FQDN (Fully Qualified Domain Name)

### Définition
Nom de domaine complet incluant hostname + domaine

### Structure
```
web01.production.company.com
  |        |          |     |
  |        |          |     └─ TLD (Top Level Domain)
  |        |          └─────── Domain
  |        └────────────────── Subdomain
  └─────────────────────────── Hostname
```

### Exemples
- **FQDN:** `api.prod.company.com`
- **Hostname seul:** `api`
- **Domain:** `company.com`

### Utilisation
- Identification unique d'une machine sur le réseau
- Configuration DNS
- Certificats SSL/TLS
- Kubernetes Ingress

---

## Subnets et Masques de sous-réseau

### Principe
Découpage d'un réseau IP en sous-réseaux plus petits

### Notation CIDR (Classless Inter-Domain Routing)

**Format:** `Adresse_IP/Nombre_de_bits_masque`

**Exemples:**
- `192.168.1.0/24` = 256 adresses (254 utilisables)
- `10.0.0.0/16` = 65,536 adresses
- `172.16.0.0/12` = 1,048,576 adresses

### Calcul du masque

| CIDR | Masque décimal | Nombre d'hôtes |
|------|----------------|----------------|
| /32 | 255.255.255.255 | 1 (hôte unique) |
| /31 | 255.255.255.254 | 2 (lien point-to-point) |
| /30 | 255.255.255.252 | 4 (2 utilisables) |
| /29 | 255.255.255.248 | 8 (6 utilisables) |
| /28 | 255.255.255.240 | 16 (14 utilisables) |
| /24 | 255.255.255.0 | 256 (254 utilisables) |
| /16 | 255.255.0.0 | 65,536 |
| /8 | 255.0.0.0 | 16,777,216 |

### Exemple pratique

**Réseau:** `192.168.1.0/24`

- **Adresse réseau:** `192.168.1.0`
- **Première IP utilisable:** `192.168.1.1`
- **Dernière IP utilisable:** `192.168.1.254`
- **Broadcast:** `192.168.1.255`
- **Gateway (typique):** `192.168.1.1`

### Subnetting - Exemple

**Réseau de base:** `10.0.0.0/16` (65,536 IPs)

**Découpage en 4 sous-réseaux /18:**
1. `10.0.0.0/18` (10.0.0.0 - 10.0.63.255)
2. `10.0.64.0/18` (10.0.64.0 - 10.0.127.255)
3. `10.0.128.0/18` (10.0.128.0 - 10.0.191.255)
4. `10.0.192.0/18` (10.0.192.0 - 10.0.255.255)

---

## Gateway et Routing

### Gateway (Passerelle)

**Définition:** Point de sortie d'un réseau local vers un autre réseau

**Fonction:**
- Router le trafic entre réseaux
- Généralement la première IP du subnet

**Exemple:**
```
Réseau: 192.168.1.0/24
Gateway: 192.168.1.1

Client 192.168.1.100 veut joindre 8.8.8.8
→ Envoie le paquet à la gateway 192.168.1.1
→ Gateway route vers Internet
```

### Default Gateway

**Définition:** Gateway utilisée quand la destination n'est pas sur le réseau local

**Configuration typique:**
- **IP machine:** `192.168.1.50`
- **Subnet mask:** `255.255.255.0`
- **Default gateway:** `192.168.1.1`
- **DNS:** `8.8.8.8`

### Routing Table

**Exemple de table de routage:**

```
Destination       Gateway         Interface
0.0.0.0/0        192.168.1.1     eth0      (route par défaut)
192.168.1.0/24   0.0.0.0         eth0      (réseau local)
10.0.0.0/8       192.168.1.254   eth0      (réseau interne)
```

### Commandes utiles

**Linux/Mac:**
```bash
# Voir la table de routage
ip route show
route -n

# Ajouter une route
ip route add 10.0.0.0/8 via 192.168.1.254

# Route par défaut
ip route add default via 192.168.1.1
```

**Windows:**
```cmd
# Voir la table de routage
route print

# Ajouter une route
route add 10.0.0.0 mask 255.0.0.0 192.168.1.254
```

---

## NAT (Network Address Translation)

### Principe
Traduction d'adresses IP privées en adresse(s) IP publique(s)

### Fonctionnement

```
Réseau interne (privé)          Internet (public)
192.168.1.10 ──┐
192.168.1.20 ──┼──> [NAT Router] ──> 203.0.113.5
192.168.1.30 ──┘    (51.15.45.78)
```

### Types de NAT

#### 1. SNAT (Source NAT)
Change l'IP source du paquet sortant

**Exemple:**
```
Interne: 192.168.1.10:54321 → 8.8.8.8:53
Externe: 203.0.113.5:12345 → 8.8.8.8:53
```

#### 2. DNAT (Destination NAT)
Change l'IP de destination du paquet entrant

**Exemple:** Port forwarding
```
Internet: Client → 203.0.113.5:80
Interne:  Client → 192.168.1.100:80 (serveur web)
```

#### 3. PAT (Port Address Translation)
Plusieurs IPs privées partagent 1 IP publique via différents ports

**C'est le NAT le plus courant dans les routeurs domestiques**

### NAT dans Kubernetes

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
  - port: 80        # Port externe
    targetPort: 8080  # Port du pod
```

Le Load Balancer fait du NAT: Port 80 externe → Port 8080 pod

---

## Ports et Protocoles

### Ports réseau

**Définition:** Identifiant numérique (0-65535) pour les services/applications

### Plages de ports

- **Well-known ports:** 0-1023 (nécessitent privilèges root)
- **Registered ports:** 1024-49151
- **Dynamic/Private ports:** 49152-65535

### Ports courants à connaître

| Port | Protocole | Service |
|------|-----------|---------|
| 20-21 | FTP | File Transfer |
| 22 | SSH | Secure Shell |
| 23 | Telnet | Remote login (non sécurisé) |
| 25 | SMTP | Email (envoi) |
| 53 | DNS | Domain Name System |
| 80 | HTTP | Web non sécurisé |
| 110 | POP3 | Email (réception) |
| 143 | IMAP | Email (réception) |
| 443 | HTTPS | Web sécurisé |
| 3306 | MySQL | Base de données |
| 3389 | RDP | Remote Desktop (Windows) |
| 5432 | PostgreSQL | Base de données |
| 6379 | Redis | Cache/DB |
| 8080 | HTTP-alt | Web alternatif |
| 27017 | MongoDB | Base de données |

### Protocoles de transport

#### TCP (Transmission Control Protocol)
- **Connexion:** Orienté connexion (handshake)
- **Fiabilité:** Garantit livraison et ordre
- **Usage:** HTTP, HTTPS, SSH, FTP, bases de données

**3-way handshake:**
```
Client → SYN → Serveur
Client ← SYN-ACK ← Serveur
Client → ACK → Serveur
[Connexion établie]
```

#### UDP (User Datagram Protocol)
- **Connexion:** Sans connexion
- **Fiabilité:** Pas de garantie de livraison
- **Usage:** DNS, streaming, gaming, VoIP
- **Avantage:** Plus rapide que TCP

---

## Modèle OSI

### Les 7 couches

| Couche | Nom | Fonction | Exemples |
|--------|-----|----------|----------|
| **7** | Application | Interface utilisateur | HTTP, FTP, SMTP, DNS |
| **6** | Présentation | Formatage des données | SSL/TLS, JPEG, ASCII |
| **5** | Session | Gestion des sessions | NetBIOS, RPC |
| **4** | Transport | Communication bout-en-bout | TCP, UDP |
| **3** | Réseau | Routage des paquets | IP, ICMP, OSPF |
| **2** | Liaison | Communication locale | Ethernet, WiFi, MAC |
| **1** | Physique | Transmission bits | Câbles, signaux |

### Mnémotechnique (de haut en bas)
**"All People Seem To Need Data Processing"**

### Exemple pratique: Requête HTTP

```
7. Application:  GET /index.html HTTP/1.1
6. Présentation: Encryption SSL/TLS (si HTTPS)
5. Session:      Établit session TCP
4. Transport:    TCP segment (port 443)
3. Réseau:       Paquet IP (adresses src/dst)
2. Liaison:      Frame Ethernet (adresses MAC)
1. Physique:     Bits sur le câble/WiFi
```

---

## Load Balancing

### Principe
Distribution du trafic entre plusieurs serveurs

### Algorithmes de répartition

#### 1. Round Robin
Distribution séquentielle

```
Requête 1 → Serveur A
Requête 2 → Serveur B
Requête 3 → Serveur C
Requête 4 → Serveur A
...
```

#### 2. Least Connections
Envoie vers le serveur avec le moins de connexions actives

#### 3. IP Hash
Hash de l'IP client détermine le serveur
→ Même client = même serveur (session persistence)

#### 4. Weighted Round Robin
Round robin avec pondération selon capacité serveur

### Couches de Load Balancing

#### Layer 4 (L4) - Transport
- Basé sur IP/Port
- Pas de visibilité contenu
- Très rapide
- Exemple: TCP/UDP load balancing

#### Layer 7 (L7) - Application
- Basé sur contenu HTTP (URL, headers, cookies)
- Routing intelligent
- Plus lent que L4
- Exemple: Nginx, HAProxy, Azure Application Gateway

### Health Checks

**Vérification que les serveurs sont disponibles:**

```yaml
# Exemple Kubernetes
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

### Exemples dans le cloud

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

## VPN et Tunneling

### VPN (Virtual Private Network)

**Principe:** Créer un réseau privé sécurisé sur Internet public

### Types de VPN

#### 1. Site-to-Site VPN
Connecte deux réseaux distants

```
Réseau A (Paris) ←─[VPN Tunnel]─→ Réseau B (Londres)
10.0.0.0/16                         172.16.0.0/16
```

#### 2. Point-to-Site VPN (Remote Access)
Utilisateur distant se connecte au réseau d'entreprise

```
Employee laptop ←─[VPN]─→ Corporate Network
  (anywhere)                  (10.0.0.0/16)
```

### Protocoles VPN

- **IPSec:** Standard, sécurisé, complexe
- **OpenVPN:** Open source, flexible
- **WireGuard:** Moderne, rapide, simple
- **L2TP/IPSec:** Combo de protocoles

### Tunneling

**Principe:** Encapsuler un protocole dans un autre

**Exemple:** VPN tunnel
```
[Données privées] → [Encryption] → [IP header public] → Internet
```

### VPN dans Azure

#### Azure VPN Gateway
```
On-premises Network ←─[Site-to-Site VPN]─→ Azure VNet
```

#### Azure Bastion
Accès sécurisé aux VMs sans IP publique
```
User → Azure Bastion → VM privée (pas d'IP publique)
```

---

## Concepts Kubernetes Réseau

### Pod Networking

**Principe:** Chaque pod a sa propre IP

```
Node 1:
  Pod A: 10.244.0.5
  Pod B: 10.244.0.6

Node 2:
  Pod C: 10.244.1.5
  Pod D: 10.244.1.6
```

### Service Types

#### 1. ClusterIP (défaut)
IP interne au cluster uniquement

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
Expose sur un port de chaque node

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Accessible sur <NodeIP>:30080
```

#### 3. LoadBalancer
Crée un load balancer cloud (Azure LB, AWS ELB...)

```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

### Network Policies

**Contrôle du trafic entre pods:**

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

**Routage HTTP/HTTPS externe:**

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

## Troubleshooting Réseau

### Commandes essentielles

#### ping
Teste la connectivité IP

```bash
ping 8.8.8.8
ping google.com
```

#### traceroute / tracert
Trace le chemin des paquets

```bash
traceroute google.com      # Linux/Mac
tracert google.com         # Windows
```

#### nslookup / dig
Requêtes DNS

```bash
nslookup google.com
dig google.com
```

#### netstat / ss
Connexions réseau actives

```bash
netstat -tuln    # Tous les ports en écoute
ss -tuln         # Version moderne de netstat
```

#### curl / wget
Test HTTP

```bash
curl https://api.example.com/health
wget https://example.com/file.txt
```

#### telnet / nc (netcat)
Test de connexion sur un port

```bash
telnet example.com 80
nc -zv example.com 443
```

### Troubleshooting Kubernetes

```bash
# Tester connectivité depuis un pod
kubectl exec -it <pod-name> -- curl http://service-name

# Voir les endpoints d'un service
kubectl get endpoints <service-name>

# Logs d'un pod
kubectl logs <pod-name>

# Décrire un service (voir les selectors)
kubectl describe service <service-name>

# Tester DNS dans le cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
```

---

## Scénarios d'Entretien - Exemples de Réponses

### Scénario 1: "Un client ne peut pas accéder à l'application"

**Approche méthodique:**

1. **Vérifier la couche applicative:**
   - Le service est-il up? `kubectl get pods`
   - Les logs montrent-ils des erreurs? `kubectl logs`

2. **Vérifier la couche réseau:**
   - Le Service Kubernetes existe? `kubectl get svc`
   - Les endpoints sont corrects? `kubectl get endpoints`
   - Le Load Balancer a une IP publique? `kubectl get svc -o wide`

3. **Vérifier DNS:**
   - Le DNS résout-il correctement? `nslookup app.example.com`
   - Le certificat SSL est-il valide?

4. **Vérifier connectivité:**
   - Ping l'IP publique
   - Curl l'endpoint
   - Vérifier les Network Policies
   - Vérifier les Security Groups/NSG

### Scénario 2: "Latence élevée entre deux services"

**Debugging:**

1. **Vérifier si c'est réseau ou application:**
   ```bash
   # Depuis pod A vers service B
   kubectl exec -it pod-a -- time curl http://service-b
   ```

2. **Vérifier la localité:**
   - Les pods sont-ils sur le même node?
   - Problème de zone de disponibilité?

3. **Monitoring:**
   - Check Grafana/Prometheus metrics
   - Network latency entre nodes
   - Charge CPU/mémoire des pods

4. **Solutions:**
   - Anti-affinity rules si besoin de séparer
   - Affinity rules si besoin de rapprocher
   - Scaling si surchargé

### Scénario 3: "Comment sécuriser le trafic entre pods?"

**Réponses:**

1. **Network Policies:**
   - Restreindre trafic ingress/egress
   - Principe du moindre privilège

2. **Service Mesh (Istio, Linkerd):**
   - mTLS automatique entre pods
   - Policies de trafic avancées

3. **Private Endpoints:**
   - Pas d'exposition publique
   - Trafic reste dans le VNet

4. **Encryption in transit:**
   - TLS entre services
   - Certificats gérés (cert-manager)

---

## Glossaire Rapide

| Terme | Définition courte |
|-------|-------------------|
| **ACL** | Access Control List - règles de filtrage |
| **BGP** | Border Gateway Protocol - routage Internet |
| **DHCP** | Dynamic Host Configuration Protocol - attribution IP auto |
| **DMZ** | Zone démilitarisée - réseau semi-sécurisé |
| **Firewall** | Pare-feu - filtre le trafic réseau |
| **ICMP** | Internet Control Message Protocol - utilisé par ping |
| **MAC Address** | Adresse physique unique d'une interface réseau |
| **MTU** | Maximum Transmission Unit - taille max d'un paquet |
| **QoS** | Quality of Service - priorisation du trafic |
| **VLAN** | Virtual LAN - segmentation réseau virtuelle |
| **VPC** | Virtual Private Cloud - réseau isolé dans le cloud |

---

## Checklist pour l'Entretien

**Tu dois être capable d'expliquer:**

- Différence entre IP publique et privée
- Comment fonctionne DNS
- C'est quoi un FQDN et donner un exemple
- Calculer combien d'IPs dans un /24
- C'est quoi une gateway et son rôle
- Principe du NAT et pourquoi on l'utilise
- Différence TCP vs UDP
- Ports courants (22, 80, 443, 3306...)
- Load balancing L4 vs L7
- Types de Services Kubernetes
- Comment troubleshoot un problème réseau

**Commandes à connaître:**
```bash
ping, traceroute, nslookup, curl, netstat
kubectl get svc/pods/endpoints, kubectl logs, kubectl describe
```

Gateway (Passerelle):
Fonction:
Point de sortie du réseau local vers un autre réseau (typiquement Internet)
Quand c'est utilisé:
Scénario: Machine 192.168.1.50 veut atteindre 8.8.8.8 (Google DNS)
1. Machine regarde: "8.8.8.8 est dans mon réseau local (192.168.1.x)?"
   → NON

2. Machine envoie le paquet à la GATEWAY (192.168.1.1)

3. Gateway (routeur) route le paquet vers Internet
Analogie:
La gateway = la porte de sortie de ta maison vers la rue

Broadcast (Diffusion):
Fonction:
Envoyer un message à TOUTES les machines du réseau local en même temps
Quand c'est utilisé:
Exemples:
1. ARP (Address Resolution Protocol):
Machine A: "Qui a l'IP 192.168.1.100?"
   ↓
Envoie à l'adresse broadcast 192.168.1.255
   ↓
TOUTES les machines du réseau reçoivent la question
   ↓
Machine avec IP 192.168.1.100 répond: "C'est moi! Mon MAC est XX:XX:XX:XX:XX:XX"
2. DHCP Discovery:
Nouvelle machine arrive sur le réseau
   ↓
"Y a-t-il un serveur DHCP ici?"
   ↓
Envoie à 255.255.255.255 (broadcast)
   ↓
Serveur DHCP répond et attribue une IP
3. Wake-on-LAN:
Envoyer "magic packet" à broadcast
   ↓
Machine cible se réveille
Analogie:
Broadcast = crier dans une pièce pour que TOUT LE MONDE entende