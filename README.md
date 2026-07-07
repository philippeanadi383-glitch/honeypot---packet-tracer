<div align="center">
<img src="https://raw.githubusercontent.com/philippeanadi383-glitch/honeypot--packet-tracer/main/banner.png" width="800"/>
  #  Honeypot Smart City Security Lab

  ![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue?logo=cisco)
  ![Security](https://img.shields.io/badge/Cybersecurity-Honeypot-red)
  ![Status](https://img.shields.io/badge/Status-Complete-green)
  ![ACL](https://img.shields.io/badge/ACL-Firewall-orange)
  ![Telnet](https://img.shields.io/badge/Protocol-Telnet-darkred)
  ![SmartCity](https://img.shields.io/badge/Smart-City-purple)
</div>

#  Honeypot Smart City — Cisco Packet Tracer

## Objectif
Simuler un piège réseau (Honeypot) dans une Smart City pour attirer un attaquant extérieur vers un faux serveur vulnérable, tout en protégeant l'infrastructure réelle de la ville.

---

## 📋 Table des matières
- [Scénario](#-scénario)
- [Topologie](#-topologie)
- [Adressage](#-table-dadressage)
- [Configuration](#-configuration)
- [ACL](#-politique-de-sécurité)
- [Tests](#-tests)
- [Concepts](#-concepts-démontrés)

---

##  Scénario
```
[Attaquant] ──Internet simulé── [Routeur_Ville] ──┬── Honeypot_IoT (piège)
                                                   └── Serveur_Central (protégé)
```
L'attaquant depuis Internet tente de pénétrer le réseau de la ville.
Il tombe sur le Honeypot — pendant ce temps le vrai serveur reste invisible.
L'admin détecte l'intrusion et isole l'attaquant.

---

##  Topologie
![Topologie](topologie.png)

---

##  Table d'adressage

| Appareil | Interface | IP | Masque | Gateway |
|---|---|---|---|---|
| PC0 Attaquant | Fa0 | 203.0.113.5 | 255.255.255.0 | 203.0.113.1 |
| Routeur_Internet | G0/0/0 | 203.0.113.1 | 255.255.255.0 | — |
| Routeur_Internet | G0/0/1 | 10.10.10.1 | 255.255.255.0 | — |
| Routeur_Ville | G0/0/0 | 10.10.10.2 | 255.255.255.0 | — |
| Routeur_Ville | G0/0/1 | 192.168.1.1 | 255.255.255.0 | — |
| Honeypot_IoT | Fa0 | 192.168.1.99 | 255.255.255.0 | 192.168.1.1 |
| Serveur_Central | Fa0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |

---

##  Configuration

### Routes statiques
```bash
# Routeur_Internet
ip route 192.168.1.0 255.255.255.0 10.10.10.2

# Routeur_Ville
ip route 203.0.113.0 255.255.255.0 10.10.10.1
```

### Telnet sur Routeur_Ville (piège volontaire)
```bash
line vty 0 4
password cisco
login
transport input telnet
exit
```

---

##  Politique de sécurité

### Phase 1 — Attaquant redirigé vers le Honeypot
```bash
access-list 150 permit ip host 203.0.113.5 host 192.168.1.99
access-list 150 deny ip host 203.0.113.5 host 192.168.1.10
access-list 150 permit ip any any

interface g0/0/0
 ip access-group 150 in
```

### Phase 2 — Isolation complète après détection
```bash
no access-list 150
access-list 150 deny ip host 203.0.113.5 any
access-list 150 permit ip any any
```

---

##  Tests

| Test | Résultat |
|---|---|
| Attaquant → ping Honeypot |  Accessible |
| Attaquant → ping Serveur_Central |  Bloqué |
| Attaquant → telnet 192.168.1.1 |  Entre dans le piège |
| Admin isole l'attaquant |  Session coupée |

---

##  Concepts démontrés
- Honeypot — piège réseau pour attirer les attaquants
- ACL dynamique — modification en temps réel pour isoler une menace
- Telnet volontairement vulnérable vs SSH sécurisé
- Simulation Internet avec double routeur
- IDS/IPS — détection et blocage automatique (concept)
- Ping sweep — technique de reconnaissance réseau
- Adresses RFC 5737 (203.0.113.x) pour simulation Internet
