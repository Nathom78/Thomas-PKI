
# 📘 Thomas‑PKI — Public Certificate Repository

Ce dépôt contient les artefacts publics de la PKI personnelle **CA‑Thoom**, utilisés pour :

- la validation de certificats PIV/SSH  
- la vérification de la chaîne de confiance  
- la vérification de révocation via CRL  
- la publication statique via GitHub Pages  

Ce dépôt **ne contient aucune clé privée**.  
Il expose uniquement les éléments nécessaires à la validation publique.

---
## 🔐 PKI Status

![PKI Status](https://img.shields.io/badge/PKI%20Status-Operational-brightgreen?style=for-the-badge&logo=lock)
![CRL Updated](https://img.shields.io/badge/CRL-Updated-blue?style=for-the-badge&logo=shield)

![Root CA](https://img.shields.io/badge/Root_CA-OK-success?style=flat&logo=key)
![Intermediate CA](https://img.shields.io/badge/Intermediate_CA-OK-success?style=flat&logo=key)
![Leaf Certificates](https://img.shields.io/badge/Leaf_Certs-OK-success?style=flat&logo=key)

![S/MIME](https://img.shields.io/badge/S%2FMIME-Available-blue?style=for-the-badge&logo=maildotru)

![OpenSSL](https://img.shields.io/badge/OpenSSL-4.0.0%20Ready-purple?style=for-the-badge&logo=openssl)
![YubiKey](https://img.shields.io/badge/YubiKey-Compatible-yellow?style=for-the-badge&logo=yubico)

### CRL Last Update (Dynamic)
![CRL Last Update](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/Nathom78/Thomas-PKI/main/status.json)


## 📁 Structure du dépôt

```
Thomas-PKI/
│
├── root-ecc/
│   ├── certs/
│   │   └── root-cert.pem
│   └── crl/
│       └── root-crl.pem
│
├── intermediate-ecc/
│   ├── certs/
│   │   └── intermediate-cert.pem
│   └── crl/
│       └── intermediate-crl.pem
│
└── leafs/
    ├── 9A/
    │   └── 9A-cert.pem
    ├── 9C/
    │   └── 9C-cert.pem
    ├── 9D/
    │   └── 9D-cert.pem
    └── 9E/
        └── 9E-cert.pem
```

---

## 🏛️ PKI Overview

La PKI CA‑Thoom suit une architecture classique :

```
Root CA (offline)
   ↓ signs
Intermediate CA (online)
   ↓ signs
Leaf Certificates (PIV/SSH)
```

- La **Root CA** est utilisée uniquement pour signer l’Intermediate.  
- L’**Intermediate CA** signe les certificats utilisateurs (leafs).  
- Les leafs sont utilisés pour PIV, SSH, authentification, signature, etc.

---

## 🔐 Certificats publiés

### Root CA
- `root-ecc/certs/root-cert.pem`  
  → Certificat public de la Root CA  
- `root-ecc/crl/root-crl.pem`  
  → CRL Root (rarement utilisée)

### Intermediate CA
- `intermediate-ecc/certs/intermediate-cert.pem`  
  → Certificat public de l’Intermediate  
- `intermediate-ecc/crl/intermediate-crl.pem`  
  → CRL utilisée pour la révocation des leafs

### Leaf Certificates
- `leafs/<slot>/<slot>-cert.pem`  
  → Certificats PIV (9A, 9C, 9D, 9E)

---

## 🔗 CDP & AIA (Distribution Points)

Les certificats leafs contiennent :

### **AIA (Authority Information Access)**  
→ Permet de télécharger le certificat de l’Intermediate  
```
https://nathom78.github.io/Thomas-PKI/intermediate-ecc/certs/intermediate-cert.pem
```

### **CDP (CRL Distribution Point)**  
→ Permet de télécharger la CRL Intermediate  
```
https://nathom78.github.io/Thomas-PKI/intermediate-ecc/crl/intermediate-crl.pem
```

Ces URLs sont intégrées automatiquement lors de la génération des certificats.

---

## 🧪 Vérification d’un certificat

### Sous Windows
Double‑cliquer sur un certificat → onglet **Détails** → **Afficher les propriétés du certificat**.

Windows :

1. télécharge automatiquement la CRL  
2. vérifie la signature  
3. vérifie si le certificat est révoqué  
4. reconstruit la chaîne Root → Intermediate → Leaf

### Sous OpenSSL

```
openssl verify -crl_check -CAfile intermediate-cert.pem leaf-cert.pem
```

---

## 🔄 Mise à jour du dépôt

Les fichiers sont publiés automatiquement via :

- `Publish-CAThoomCRL.ps1` (Root)
- `Publish-CAThoomCRLIntermediate.ps1` (Intermediate)
- `Publish-CAThoomLeaf.ps1` (Leafs)
- `Update-PKIGitHub.ps1` (git add/commit/push)

---

## ⚠️ Sécurité

Ce dépôt **ne contient aucune clé privée** :

- pas de `root-key.pem`  
- pas de `intermediate-key.pem`  
- pas de clés PIV  

Seuls les artefacts publics nécessaires à la validation sont publiés.

---

## 📄 Licence

Usage personnel, pédagogique et expérimental.

## 🖼️ Diagramme Mermaid — Architecture CA‑THOOM‑PIV‑SSH v2

```mermaid
flowchart TD

    subgraph ROOT["Root CA (ECC P‑384)"]
        RCRT["root-cert.pem"]
        RCKEY["root-key.pem (offline)"]
        RCRL["root-crl.pem"]
    end

    subgraph INTER["Intermediate CA (slot 82 - YubiKey)"]
        ICRT["intermediate-cert.pem"]
        ICKEY["slot 82 key (non-exportable)"]
        ICRL["intermediate-crl.pem"]
    end

    subgraph LEAFS["Leaf Certificates (PIV Slots)"]
        A9A["9A - SmartCard / BitLocker"]
        A9C["9C - Signature"]
        A9D["9D - S/MIME"]
        A9E["9E - SSH PKCS#11"]
    end

    subgraph WIN["Windows Integration"]
        WSTORE["Windows Cert Store"]
        BITLOCKER["BitLocker Protectors"]
        SCLOGON["SmartCard Logon"]
    end

    subgraph SSH["SSH Integration"]
        PKCS11["libykcs11.dll"]
        WCSSH["WinCryptSSHAgent"]
    end

    subgraph GIT["GitHub Pages"]
        GROOT["root-ecc/crl"]
        GINTER["intermediate-ecc/crl"]
        GLEAFS["leafs/<slot>"]
    end

    RCRT --> ICRT
    RCRL --> GROOT

    ICRT --> A9A
    ICRT --> A9C
    ICRT --> A9D
    ICRT --> A9E

    ICRL --> GINTER

    A9A --> WSTORE
    A9C --> WSTORE
    A9D --> WSTORE
    A9E --> WSTORE

    A9A --> SCLOGON
    A9A --> BITLOCKER

    A9E --> PKCS11
    A9E --> WCSSH

    A9A --> GLEAFS
    A9C --> GLEAFS
    A9D --> GLEAFS
    A9E --> GLEAFS
```

```mermaid
flowchart TD

A[Root CA] --> B[Intermediate CA slot 82]
B --> C{New-CAThoomLeafRequest}
C --> D[CSR Leaf]
D --> E[Invoke-CAThoomLeafSigning]
E --> F[Cert Leaf]

F --> G[Initialize-PIVSlot]
G --> H[Import dans slot]
G --> I[Install-PIVCertificate]
G --> J[Approve-PIVSlotCoherence]

G -->|PublishAfter| K[Publish-CAThoomLeaf]

subgraph Orchestrateur
    L[Initialize-PIVAllSlots]
end

L --> G
```

