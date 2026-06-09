# TSSR-0226-P3-G5 — Infrastructure réseau BillU

## Sommaire

1. [Présentation du projet](#1-présentation-du-projet)
2. [Contexte de l'entreprise](#2-contexte-de-lentreprise)
3. [Objectifs](#3-objectifs)
4. [Infrastructure déployée](#4-infrastructure-déployée)
5. [Plan d'adressage réseau](#5-plan-dadressage-réseau)
6. [Organisation de la documentation](#6-organisation-de-la-documentation)
7. [Accès à la documentation](#7-accès-à-la-documentation)

---

## 1. Présentation du projet

Ce projet est réalisé dans le cadre de la formation **TSSR — Technicien Supérieur Systèmes et Réseaux** à la Wild Code School (promotion 0226).

L'objectif est la conception, le déploiement et la documentation complète d'une nouvelle infrastructure réseau pour la société fictive **BillU**, dans un contexte professionnel simulé. Le prestataire (l'étudiant) est mandaté par le DSI de BillU pour bâtir l'infrastructure de zéro.

---

## 2. Contexte de l'entreprise

**BillU** est une filiale du groupe international **RemindMe** (2000+ collaborateurs, présence mondiale), spécialisée dans le développement de logiciels de facturation.

| Caractéristique | Détail |
|---|---|
| Localisation | Paris, 20ᵉ arrondissement |
| Effectif | 196 collaborateurs |
| Départements | 9 (Direction, DSI, Développement Logiciel, Finance-Comptabilité, Juridique, Communication, Commercial, QHSE, Recrutement) |
| Matériel client | 100% PC portables |

### Situation initiale (avant projet)

- Postes en workgroup, aucun domaine AD
- Réseau basé sur une box FAI et des répéteurs Wi-Fi (`172.16.10.0/24`)
- Aucune sécurité d'identité — mots de passe locaux partagés
- Stockage sur NAS grand public sans redondance
- Messagerie cloud (`prenom.nom@billu.com`)
- Aucun serveur interne

---

## 3. Objectifs

### 3.1 Objectifs principaux

- [x] Pare-feu pfSense (WAN / LAN / DMZ)
- [x] Domaine Active Directory `tssr.lan` (AD DS, DNS, DHCP)
- [x] GPO (politique MDP, verrouillage, panneau de config, PowerShell, fond d'écran, réseaux sociaux, admin local)
- [x] Clients Windows joints au domaine (CLIWIN01, CLIWIN02)
- [x] GLPI — gestion de parc et ticketing
- [X] WSUS — gestion centralisée des mises à jour
- [ ] VoIP — FreePBX sur IPBX01
- [ ] Messagerie — Zimbra ou iRedMail

### 3.2 Objectifs secondaires

- [x] Restrictions horaires utilisateurs (7h30–20h, lun–sam)
- [x] NTP — synchronisation de l'heure sur SRVWIN01
- [ ] Dossiers partagés (lecteur I: par utilisateur)
- [ ] DC secondaires SRVWIN02 et SRVWIN03 (Windows Server Core)
- [ ] Serveur web interne (LAN)
- [ ] Serveur web externe (DMZ)
- [ ] Synchronisation GLPI ↔ AD
- [ ] Switch virtuel + routeur VyOS

---

## 4. Infrastructure déployée

| Machine | Rôle | OS | IP |
|---|---|---|---|
| FW01 | Pare-feu pfSense | pfSense 2.x | WAN: `192.168.50.2` / LAN: `10.0.1.254` / DMZ: `10.0.2.254` |
| SRVWIN01 | DC principal, DNS, DHCP | Windows Server 2022 Datacenter | `10.0.1.1` |
| SRVWIN04 | WSUS | Windows Server 2022 Datacenter | `10.0.1.2` |
| SRVLX01 | GLPI + Messagerie | Debian 12 | `10.0.1.3` |
| IPBX01 | VoIP FreePBX | — | `10.0.1.4` |
| CLIWIN01 | Client Windows 10 | Windows 10 | DHCP (`10.0.1.50–240`) |
| CLIWIN02 | Client Windows 11 | Windows 11 | DHCP (`10.0.1.50–240`) |

---

## 5. Plan d'adressage réseau

| Réseau | Plage | Rôle |
|---|---|---|
| WAN | `192.168.50.0/24` | Accès Internet (VMnet8) |
| LAN | `10.0.1.0/24` | Réseau interne |
| DMZ | `10.0.2.0/24` | Zone démilitarisée |

| Plage | Usage |
|---|---|
| `10.0.1.1 – 10.0.1.49` | IPs statiques serveurs |
| `10.0.1.50 – 10.0.1.240` | Pool DHCP clients |
| `10.0.1.254` | Passerelle LAN (FW01) |

---

## 6. Organisation de la documentation

La documentation suit la structure standard TSSR :

| Dossier | Type | Contenu |
|---|---|---|
| `architecture/` | HLD — High Level Design | Vue globale, schéma réseau, plan d'adressage |
| `components/` | LLD — Low Level Design | Configuration détaillée de chaque serveur/service |
| `operations/` | DEX — Documentation d'exploitation | Procédures d'administration, guides utilisateur |
| `naming.md` | Nomenclature | Convention de nommage des machines, utilisateurs, groupes, GPO |

---

## 7. Accès à la documentation

- **Nomenclature** : [naming.md](./naming.md)
- **Architecture (HLD)** : [architecture/](./architecture/)
- **Composants (LLD)** : [components/](./components/)
- **Exploitation (DEX)** : [operations/](./operations/)
