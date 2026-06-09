Architecture — High Level Design (HLD)
Sommaire

Vue d'ensemble
Schéma réseau
Plan d'adressage
Description des zones
Flux réseau autorisés
Pare-feu FW01


1. Vue d'ensemble
L'infrastructure BillU repose sur une architecture 3 zones séparées par un pare-feu pfSense :

WAN — connexion Internet via VMnet8 (NAT)
LAN — réseau interne hébergeant tous les serveurs et clients
DMZ — zone exposée pour les services accessibles depuis Internet (Phase 2)

Tout le trafic transite obligatoirement par FW01, qui applique une politique Deny All — seul ce qui est explicitement autorisé passe.
L'environnement est entièrement virtualisé sur VMware Workstation, avec le domaine Active Directory tssr.lan.

2. Schéma réseau
Afficher l'image

3. Plan d'adressage
Réseaux
ZoneRéseauPasserelleUsageWAN192.168.50.0/24192.168.50.1 (box FAI)Accès Internet — VMnet8 NATLAN10.0.1.0/2410.0.1.254 (FW01)Réseau interneDMZ10.0.2.0/2410.0.2.254 (FW01)Zone exposée — Phase 2
Machines — IPs statiques
MachineRôleOSIPInterfaceFW01Pare-feu pfSensepfSense 2.7.xWAN: 192.168.50.2 / LAN: 10.0.1.254 / DMZ: 10.0.2.254VMnet8 / VMnet2 / VMnet3SRVWIN01DC principal, DNS, DHCPWindows Server 2022 Datacenter10.0.1.1VMnet2SRVWIN04WSUSWindows Server 2022 Datacenter10.0.1.2VMnet2SRVLX01GLPI + MessagerieDebian 1210.0.1.3VMnet2IPBX01VoIP FreePBXLinux10.0.1.4VMnet2SRVWEB01 (Phase 2)Serveur web externeLinux10.0.2.1VMnet3
Clients — DHCP
MachineRôleOSIPCLIWIN01ClientWindows 11DHCPCLIWIN02ClientWindows 11DHCP
Plage DHCP : 10.0.1.50 → 10.0.1.240
Serveur DHCP : SRVWIN01 (10.0.1.1)

4. Description des zones
Zone WAN — 192.168.50.0/24
Connexion vers Internet via VMware VMnet8 en mode NAT. FW01 présente l'interface 192.168.50.2 côté WAN. La gateway est 192.168.50.1 (routeur VMware).
Politique : tout le trafic entrant est bloqué sauf ce qui est explicitement autorisé (règles pfSense).
Zone LAN — 10.0.1.0/24
Réseau interne hébergeant l'ensemble des serveurs d'infrastructure et des postes clients. Tous les équipements du LAN communiquent via le switch virtuel VMnet2.
Le trafic sortant vers Internet passe par FW01 (10.0.1.254).
Services disponibles sur le LAN :

Domaine Active Directory tssr.lan
DNS, DHCP
GLPI (gestion de parc, ticketing)
WSUS (mises à jour centralisées)
VoIP FreePBX

Zone DMZ — 10.0.2.0/24 (Phase 2)
Zone isolée destinée à héberger le serveur web externe SRVWEB01. Le trafic entre la DMZ et le LAN est bloqué par défaut — un attaquant qui compromet la DMZ ne peut pas atteindre le LAN.
Accessible depuis Internet sur les ports 80 et 443 via une règle pfSense.

5. Flux réseau autorisés
SourceDestinationPortRègleLANInternet80, 443✅ AutoriséLANDMZ80, 443✅ AutoriséLANDMZAutres❌ BloquéDMZInternet80, 443✅ AutoriséDMZLANTous❌ BloquéWANDMZ80, 443✅ Autorisé (Phase 2)WANLANTous❌ Bloqué

6. Pare-feu FW01
ParamètreValeurNomFW01OSpfSense 2.7.x Community EditionInterface WAN192.168.50.2/24 — VMnet8 (NAT)Interface LAN10.0.1.254/24 — VMnet2Interface DMZ10.0.2.254/24 — VMnet3Politique généraleDeny All + exceptions explicitesAccès WebGUIhttp://10.0.1.254 (depuis le LAN)Loginadmin
Règles WAN

Block private networks (RFC 1918) ✅
Block bogon networks ✅
Deny All implicite ✅

Règles LAN

Anti-Lockout Rule (port 80 → WebGUI) ✅
Allow LAN to any (IPv4) ✅
Allow LAN → DMZ port 80/443 ✅
Block LAN → DMZ (tout le reste) ✅

Règles DMZ

Allow DMZ → Internet port 80/443 ✅
Block DMZ → LAN ✅
