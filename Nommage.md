# Nomenclature — Projet BillU

## 1. Machines

| Type | Format | Exemples |
|---|---|---|
| Serveurs Windows | `SRVWIN` + numéro | `SRVWIN01`, `SRVWIN04` |
| Serveurs Linux | `SRVLX` + numéro | `SRVLX01` |
| Clients Windows | `CLIWIN` + numéro | `CLIWIN01`, `CLIWIN02` |
| Pare-feu | `FW` + numéro | `FW01` |
| IPBX / VoIP | `IPBX` + numéro | `IPBX01` |

## 2. Utilisateurs

| Élément | Format | Exemple |
|---|---|---|
| Login (SamAccountName) | `prenom.nom` | `corentin.arnaud` |
| UPN | `prenom.nom@tssr.lan` | `corentin.arnaud@tssr.lan` |
| Email BillU | `prenom.nom@billu.com` | `corentin.arnaud@billu.com` |
| Mot de passe initial | `BillU2026!` | Changement obligatoire au 1er login |

**Gestion des homonymes** : ajouter un chiffre en suffixe → `prenom.nom2`

**Règles de formatage** :
- Tout en minuscules
- Accents supprimés (é → e, ç → c, etc.)
- Espaces remplacés par des points

## 3. Groupes de sécurité

| Format | Exemple |
|---|---|
| `GRP-` + Département | `GRP-DSI`, `GRP-JURIDIQUE` |

| Groupe | Département |
|---|---|
| `GRP-DIRECTION` | Direction |
| `GRP-DSI` | DSI |
| `GRP-DEV` | Développement Logiciel |
| `GRP-FINANCE` | Finance et Comptabilité |
| `GRP-JURIDIQUE` | Juridique |
| `GRP-COMMUNICATION` | Communication |
| `GRP-COMMERCIAL` | Commercial |
| `GRP-QHSE` | QHSE |
| `GRP-RH` | Recrutement |

**Type** : Sécurité — **Portée** : Globale

## 4. Unités d'Organisation (OU)

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

**Règle** : Les utilisateurs sont placés dans l'OU correspondant à leur département. Les machines sont dans `Ordinateurs > Serveurs` ou `Ordinateurs > Clients`.

## 5. GPO

| Format | Signification |
|---|---|
| `USR-` | Cible les utilisateurs |
| `PC-` | Cible les machines |
| `-SEC-` | Politique de sécurité |
| `-CFG-` | Configuration |

| Nom GPO | Cible | Description |
|---|---|---|
| `USR-SEC-MDP-Complexite` | Utilisateurs | Complexité et longueur des mots de passe |
| `USR-SEC-Verrouillage-Compte` | Utilisateurs | Blocage après X erreurs de mot de passe |
| `USR-CFG-Panneau-Config` | Utilisateurs | Blocage du panneau de configuration |
| `PC-CFG-Admin-Local` | Machines | Gestion du compte admin local |
| `USR-SEC-PowerShell` | Utilisateurs | Restriction d'utilisation de PowerShell |
| `USR-CFG-Blocage-ReseauxSociaux` | Utilisateurs | Blocage des réseaux sociaux |
| `USR-CFG-Fond-Ecran` | Utilisateurs | Fond d'écran imposé BillU |
| `PC-CFG-WSUS` | Machines | Redirection des MAJ vers SRVWIN04 |

## 6. Domaine

| Élément | Valeur |
|---|---|
| Nom de domaine AD | `tssr.lan` |
| Niveau fonctionnel | Windows Server 2022 |
| DC principal | `SRVWIN01.tssr.lan` |
