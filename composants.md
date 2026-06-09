# Components — Low Level Design (LLD)

Ce dossier contient la configuration détaillée de chaque composant de l'infrastructure BillU.

## Sommaire

- [FW01 — Pare-feu pfSense](#fw01--pare-feu-pfsense)
- [SRVWIN01 — DC principal, DNS, DHCP](#srvwin01--dc-principal-dns-dhcp)
- [SRVWIN04 — WSUS](#srvwin04--wsus)
- [SRVLX01 — GLPI + Messagerie](#srvlx01--glpi--messagerie)
- [IPBX01 — VoIP FreePBX](#ipbx01--voip-freepbx)
- [CLIWIN01 / CLIWIN02 — Clients Windows](#cliwin01--cliwin02--clients-windows)

---

## FW01 — Pare-feu pfSense

### Configuration matérielle (VM)

| Paramètre | Valeur |
|---|---|
| Hyperviseur | VMware Workstation |
| `RAM` | 1 GB |
| Disque | 8 GB |
| Carte réseau 1 | VMnet8 (NAT) — WAN |
| Carte réseau 2 | VMnet2 (LAN) |
| Carte réseau 3 | VMnet3 (DMZ) — Phase 2 |

### Configuration réseau

| Interface | IP | Réseau |
|---|---|---|
| WAN (em0) | `192.168.50.2/24` | VMnet8 |
| LAN (em1) | `10.0.1.254/24` | VMnet2 |
| DMZ / OPT1 (em2) | `10.0.2.254/24` | VMnet3 |

### Credentials

| Paramètre | Valeur |
|---|---|
| URL WebGUI | `http://10.0.1.254` |
| Login | `admin` |
| Mot de passe | `pfsense` (à changer en production) |

### Règles pare-feu

**WAN**
- Block private networks (RFC 1918) ✅
- Block bogon networks ✅
- Deny All implicite ✅

**LAN**
- Anti-Lockout Rule (port 80 → WebGUI) ✅
- Allow LAN → Internet (IPv4) ✅
- Allow LAN → DMZ port 80/443 ✅
- Block LAN → DMZ (tout le reste) ✅

**DMZ**
- Allow DMZ → Internet port 80/443 ✅
- Block DMZ → LAN ✅

---

## SRVWIN01 — DC principal, DNS, DHCP

### Configuration matérielle (VM)

| Paramètre | Valeur |
|---|---|
| Hyperviseur | VMware Workstation |
| OS | Windows Server 2022 Datacenter (Desktop Experience) |
| `RAM` | 3 GB |
| Disque | 35 GB |
| Carte réseau | VMnet2 (LAN) |

### Configuration réseau

| Paramètre | Valeur |
|---|---|
| IP | `10.0.1.1` |
| Masque réseau | `255.255.255.0` |
| Passerelle | `10.0.1.254` |
| DNS | `127.0.0.1` (lui-même) |

### Rôles installés

- Active Directory Domain Services (AD DS)
- DNS Server
- DHCP Server

### Active Directory

| Paramètre | Valeur |
|---|---|
| Domaine | `tssr.lan` |
| Niveau fonctionnel forêt | Windows Server 2022 |
| Niveau fonctionnel domaine | Windows Server 2022 |
| Mot de passe DSRM | *(confidentiel)* |

### Structure OU

```
tssr.lan
└── BillU
    ├── Direction
    ├── DSI
    ├── Developpement-Logiciel
    ├── Finance-Comptabilite
    ├── Juridique
    ├── Communication
    ├── Commercial
    ├── QHSE
    ├── Recrutement
    └── Ordinateurs
        ├── Serveurs
        └── Clients
```

### Groupes de sécurité

| Groupe | OU | Type | Portée |
|---|---|---|---|
| GRP-DIRECTION | Direction | Sécurité | Globale |
| GRP-DSI | DSI | Sécurité | Globale |
| GRP-DEV | Developpement-Logiciel | Sécurité | Globale |
| GRP-FINANCE | Finance-Comptabilite | Sécurité | Globale |
| GRP-JURIDIQUE | Juridique | Sécurité | Globale |
| GRP-COMMUNICATION | Communication | Sécurité | Globale |
| GRP-COMMERCIAL | Commercial | Sécurité | Globale |
| GRP-QHSE | QHSE | Sécurité | Globale |
| GRP-RH | Recrutement | Sécurité | Globale |

### GPO

| Nom GPO | Liée à | Description |
|---|---|---|
| USR-SEC-MDP-Complexite | OU=BillU | Complexité mot de passe |
| USR-SEC-Verrouillage-Compte | OU=BillU | Verrouillage après X erreurs |
| USR-CFG-Panneau-Config | OU=BillU | Blocage panneau de configuration |
| PC-CFG-Admin-Local | OU=Ordinateurs | Compte admin local des machines |
| USR-SEC-PowerShell | OU=BillU | Restriction PowerShell |
| USR-CFG-Blocage-ReseauxSociaux | OU=BillU | Blocage réseaux sociaux (DNS) |
| USR-CFG-Fond-Ecran | OU=BillU | Fond d'écran BillU imposé |
| PC-CFG-WSUS | OU=Ordinateurs | Redirection MAJ vers SRVWIN04 |

### DHCP

| Paramètre | Valeur |
|---|---|
| Plage | `10.0.1.50` → `10.0.1.240` |
| Masque réseau | `255.255.255.0` |
| Passerelle distribuée | `10.0.1.254` |
| DNS distribué | `10.0.1.1` |

### DNS

| Paramètre | Valeur |
|---|---|
| Zone directe | `tssr.lan` |
| Forwarders | `8.8.8.8`, `8.8.4.4` |

### NTP

| Paramètre | Valeur |
|---|---|
| Source | `pool.ntp.org` |
| Commande | `w32tm /config /manualpeerlist:"pool.ntp.org" /syncfromflags:manual /reliable:YES /update` |

### Credentials

| Compte | Rôle | Mot de passe |
|---|---|---|
| `Administrator` | Admin domaine | *(confidentiel)* |
| `prenom.nom` | Utilisateurs BillU | `BillU2026!` (changement au 1er login) |

---

## SRVWIN04 — WSUS

### Configuration matérielle (VM)

| Paramètre | Valeur |
|---|---|
| Hyperviseur | VMware Workstation |
| OS | Windows Server 2022 Datacenter |
| `RAM` | 4 GB |
| Disque système | 35 GB |
| Disque données | 100 GB (E:\WSUS) |
| Carte réseau | VMnet2 (LAN) |

### Configuration réseau

| Paramètre | Valeur |
|---|---|
| IP | `10.0.1.2` |
| Masque réseau | `255.255.255.0` |
| Passerelle | `10.0.1.254` |
| DNS | `10.0.1.1` |

### Configuration WSUS

| Paramètre | Valeur |
|---|---|
| Port | `8530` |
| Dossier stockage | `E:\WSUS` |
| Source synchronisation | Microsoft Update |
| Produits ciblés | Windows 11, Windows Server 2022 |
| Classifications | Mises à jour critiques, Mises à jour de sécurité |
| URL clients | `http://10.0.1.2:8530` |

### Groupes WSUS

| Groupe | Machines |
|---|---|
| TEST | Machines de test (2-3 postes) |
| PRODUCTION | Tous les autres clients |

### GPO associée

`PC-CFG-WSUS` — pointe tous les clients vers `http://10.0.1.2:8530`

---

## SRVLX01 — GLPI + Messagerie

### Configuration matérielle (VM)

| Paramètre | Valeur |
|---|---|
| Hyperviseur | VMware Workstation |
| OS | Debian 12 (CLI) |
| `RAM` | 2 GB |
| Disque | 20 GB |
| Carte réseau | VMnet2 (LAN) |

### Configuration réseau

| Paramètre | Valeur |
|---|---|
| IP | `10.0.1.3` |
| Masque réseau | `255.255.255.0` |
| Passerelle | `10.0.1.254` |
| DNS | `10.0.1.1` |

### GLPI

| Paramètre | Valeur |
|---|---|
| Version | GLPI 10.0.x |
| URL accès | `http://10.0.1.3/glpi` |
| Base de données | MariaDB |
| Login admin | `glpi` |
| Mot de passe admin | *(changé après installation)* |

### Stack technique GLPI

| Composant | Version |
|---|---|
| Apache2 | Dernière stable Debian 12 |
| PHP | 8.x |
| MariaDB | Dernière stable Debian 12 |

### Credentials MariaDB

| Compte | Mot de passe |
|---|---|
| root | `Azerty1*` |
| glpi | *(confidentiel)* |

---

## IPBX01 — VoIP FreePBX

### Configuration matérielle (VM)

| Paramètre | Valeur |
|---|---|
| Hyperviseur | VMware Workstation |
| OS | Linux (FreePBX ISO) |
| `RAM` | 2 GB |
| Disque | 20 GB |
| Carte réseau | VMnet2 (LAN) |

### Configuration réseau

| Paramètre | Valeur |
|---|---|
| IP | `10.0.1.4` |
| Masque réseau | `255.255.255.0` |
| Passerelle | `10.0.1.254` |
| DNS | `10.0.1.1` |

### État

> ⚠️ **En cours de déploiement** — installation et configuration des lignes VoIP à réaliser.

---

## CLIWIN01 / CLIWIN02 — Clients Windows

### Configuration matérielle (VM)

| Paramètre | CLIWIN01 | CLIWIN02 |
|---|---|---|
| OS | Windows 11 | Windows 11 |
| `RAM` | 4 GB | 4 GB |
| Disque | 64 GB (thin) | 64 GB (thin) |
| Carte réseau | VMnet2 (LAN) | VMnet2 (LAN) |

### Configuration réseau

| Paramètre | Valeur |
|---|---|
| Mode | DHCP |
| Plage | `10.0.1.50` → `10.0.1.240` |
| DNS | `10.0.1.1` (via DHCP) |
| Passerelle | `10.0.1.254` (via DHCP) |

### Domaine

| Paramètre | Valeur |
|---|---|
| Domaine | `tssr.lan` |
| Jonction | ✅ Effectuée |

### GPO appliquées

- Fond d'écran BillU ✅
- Panneau de configuration bloqué ✅
- PowerShell restreint ✅
- Réseaux sociaux bloqués ✅
- Mises à jour via WSUS (SRVWIN04) ✅
