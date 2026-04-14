

#  AAA (Authentication, Authorization, Accounting)

````markdown
# AAA - Authentication, Authorization, Accounting

## 📌 Objectif
Mettre en place un contrôle d'accès centralisé pour :
- les administrateurs réseau (switch, équipements)
- les équipements réseau via RADIUS
- la traçabilité (accounting)

---

## 🧱 Architecture

- Serveur AAA : FreeRADIUS
- Backend : LDAP
- Équipements : switchs Cisco
- Protocoles : RADIUS

---

## 🔐 Authentification admin (VTY / console)

### Configuration switch

```bash
aaa group server radius AAA-GROUP
 server name radius-admin

aaa authentication login default group AAA-GROUP local
aaa authentication login ADMIN-VTY group AAA-GROUP local
aaa authorization exec default group AAA-GROUP if-authenticated
aaa accounting exec default start-stop group AAA-GROUP

line vty 0 15
 login authentication ADMIN-VTY
````

---

## 👥 Gestion des privilèges via LDAP

### Exemple de règles FreeRADIUS (`users`)

```bash
DEFAULT Huntgroup-Name == "switch-admin", Ldap-Group == "****"
    Service-Type = NAS-Prompt-User,
    Cisco-AVPair += "shell:priv-lvl=15"

DEFAULT Huntgroup-Name == "switch-admin", Ldap-Group == "****"
    Cisco-AVPair += "shell:priv-lvl=10"
```

---

## 🔎 Attribution dynamique du contexte (Huntgroup)

```bash
update request {
    Huntgroup-Name := "%{ldap:...}"
}
```

➡️ Permet d'adapter les droits selon :

* le switch
* le type d'accès

---

## 🗂️ Mapping LDAP ↔ RADIUS

Extrait (`attrmap`) :

```bash
control:NT-Password := 'sambaNtPassword'
reply:Service-Type  := 'radiusServiceType'
reply:Filter-Id     := 'radiusFilterId'
reply:Tunnel-Private-Group-Id := 'radiusTunnelPrivateGroupId'
```

---

## 📊 Accounting (journalisation)

```bash
aaa accounting exec default start-stop group AAA-GROUP
```

Permet :

* traçabilité des connexions
* audit sécurité



---

## ✅ Résultat

* Authentification centralisée
* Gestion des rôles via LDAP
* Traçabilité complète

````


```
