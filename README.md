# 🧩 TP VLAN & Masques — Comprendre la segmentation réseau

> Atelier pratique pour comprendre le rôle des VLAN et des masques IP dans la segmentation d’un réseau.

---

# 🎯 Objectifs pédagogiques

À la fin de ce TP, vous devrez savoir :

✅ Expliquer le rôle d’un VLAN  
✅ Comprendre le lien VLAN ↔ sous-réseau IP  
✅ Calculer et utiliser des masques IP  
✅ Mettre en place un trunk 802.1Q  
✅ Tester la communication intra/inter-VLAN

---

# 🧠 Rappel théorique simple

## 📌 VLAN
Un VLAN est un **réseau logique** sur un même switch physique.

👉 Chaque VLAN = un domaine de broadcast distinct  
👉 Chaque VLAN = un sous-réseau IP différent

---

## 📌 Masque de sous-réseau
Le masque définit :
- La partie réseau  
- La partie hôte  

| Masque | Nb hôtes |
|------|---------|
   /24 | 254 |
   /25 | 126 |
   /26 | 62 |

---

# 🗺️ Topologie du TP

```
         VLAN 10         VLAN 20
        192.168.10.0/24 192.168.20.0/24

PC1 -------- SW1 -------- R1 -------- PC3
              |  (trunk)
              |
             PC2
```
<img width="718" height="484" alt="image" src="https://github.com/user-attachments/assets/9c99168e-997f-4812-8d08-c38156f1aec3" />

---

# 🧩 Matériel Packet Tracer

- 1 routeur (2911)  
- 1 switch (2960)  
- 3 PC  

---

# 🌐 Plan d’adressage

## VLAN 10

| Équipement | IP |
|----------|------|
PC1 | 192.168.10.10/24 |
PC2 | 192.168.10.20/24 |
GW | 192.168.10.1 |

---

## VLAN 20

| Équipement | IP |
|----------|------|
PC3 | 192.168.20.10/24 |
GW | 192.168.20.1 |

---

# 🔹 PARTIE 1 — Création des VLAN

Sur le switch :

```
enable
conf t
vlan 10
name ADMIN
vlan 20
name USERS
```

---

# 🔹 PARTIE 2 — Affectation des ports

```
interface f0/1
switchport mode access
switchport access vlan 10

interface f0/2
switchport mode access
switchport access vlan 10

interface f0/3
switchport mode access
switchport access vlan 20
```
<img width="874" height="378" alt="image" src="https://github.com/user-attachments/assets/244c6ac6-8088-4fc1-af46-f76137e1ae92" />

---

# 🔹 PARTIE 3 — Trunk vers le routeur

```
interface f0/24
switchport mode trunk
```

---

# 🔹 PARTIE 4 — Router-on-a-stick

Sur R1 :

```
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface g0/0
no shutdown
```
<img width="876" height="186" alt="image" src="https://github.com/user-attachments/assets/de45faf2-1669-4674-bcdd-1b6e2b1d8e04" />

---

# 🔹 PARTIE 5 — Configuration IP des PC

Configurer IP + passerelle selon le plan d’adressage.

---

# 🧪 TESTS

## Test 1 — Intra-VLAN
PC1 → PC2  
👉 Doit fonctionner

<img width="720" height="614" alt="image" src="https://github.com/user-attachments/assets/88800e7f-ee11-48fe-b8e8-48ec43993147" />


---

## Test 2 — Inter-VLAN
PC1 → PC3  
👉 Fonctionne uniquement grâce au routeur

<img width="676" height="317" alt="image" src="https://github.com/user-attachments/assets/6e66a61b-8dbe-4fea-9b78-7c31437e80ae" />

  
---

# ❓ Questions de réflexion

1. Pourquoi PC1 ne voit-il pas PC3 sans routeur ? -> Répondez directement sur ce Readme.md
PC1 ne voit-il pas PC3 sans routeur parce que PC1 est dans le VLAN 10 et PC3 dans le VLAN 20, ce sont deux domaines de broadcast séparés. Un switch ne route pas entre VLAN (couche 2). Il faut un équipement de couche 3 (routeur ou switch) pour faire l’inter-VLAN routing.

2. Quel rôle joue le masque /24 ? -> Répondez directement sur ce Readme.md
Le masque /24 donc 255.255.255.0 définit la partie réseau et la partie hôte :
   - Réseau = 192.168.10.0 ou 192.168.20.0
   - Hôtes possibles de .1 à .254
Il permet à un PC de savoir si la destination est dans le même réseau (ARP direct) ou dans un autre réseau (envoi à la passerelle).

3. Que se passe-t-il si VLAN 10 et VLAN 20 ont le même réseau IP ? -> Répondez directement sur ce Readme.md
Si VLAN 10 et VLAN 20 ont le même réseau IP ça crée un conflit d’adressage, les deux VLAN seraient dans le même sous-réseau IP. Les conséquences sont :
   - Les PC croient que tout est “local” donc ils font ARP au lieu d’aller à la passerelle
   - Le routeur ne peut pas router correctement entre deux interfaces dans le même réseau
Ainsi la communication inter-VLAN est cassée ou incohérente.

4. Pourquoi un trunk est-il nécessaire ? -> Répondez directement sur ce Readme.md
Un trunk est nécessaire parce qu’un seul lien switch <--> routeur doit transporter plusieurs VLAN. Le trunk envoie les trames avec un tag 802.1Q (ici VLAN 10, VLAN 20).
Sans trunk, le lien ne transporterait qu’un seul VLAN (access), donc pas de router-on-a-stick possible.

---

# ⭐ Travail sur les Masques

Changer VLAN 10 en :

```
192.168.10.0/25
```

Questions :
- Combien d’hôtes max ?
/25 : il reste 7 bits hôte
Nombre d’hôtes = 2^7−2=128−2=126

- Quelle plage IP valide ?
Pour le réseau : 192.168.10.0/25 on a :
   - Adresse réseau : 192.168.10.0
   - Broadcast : 192.168.10.127
   - Hôtes valides : 192.168.10.1 à 192.168.10.126

- Peut-on encore communiquer avec VLAN 20 ?
Oui si la config routeur est mise à jour pour VLAN 10 :
   - Sur R1, l’interface g0/0.10 doit être en /25 (255.255.255.128)
   - VLAN 20 reste en /24 (255.255.255.0), ce sont deux réseaux différents donc le routeur route entre les deux.
 Sur le routeur on execute:
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.128

<img width="1012" height="547" alt="image" src="https://github.com/user-attachments/assets/ed40dcec-3bdc-460c-8586-e7d725cb0371" />
<img width="1015" height="549" alt="image" src="https://github.com/user-attachments/assets/a895ab9c-2fd5-43b7-a4a4-828cb3a9dc16" />

Teste de PC1 vers PC2 puis de PC1 vers PC3 :

<img width="664" height="622" alt="image" src="https://github.com/user-attachments/assets/65007ad0-f349-48c1-8ea7-0a1c25ad60c6" />


---

# 🚀 Extensions

- Ajouter VLAN 30:
  
  Sur le SW1 :

<img width="712" height="111" alt="image" src="https://github.com/user-attachments/assets/18727da5-ef30-464a-8866-24ce88a8d5a5" />
<img width="486" height="68" alt="image" src="https://github.com/user-attachments/assets/9797185d-29d9-4af9-ae40-6528c035489f" />
<img width="901" height="379" alt="image" src="https://github.com/user-attachments/assets/e8268c1c-397a-4692-bc1b-76f35b2b9af9" />

   Sur R1 :
   ```
<img width="885" height="375" alt="image" src="https://github.com/user-attachments/assets/928fad63-45b3-4bd7-9db3-6492fa71c219" />


- Mettre un DHCP par VLAN  
  
---

# 📝 Évaluation (/20)

| Critère | Points |
|--------|-------|
VLAN créés correctement | 4 |
Ports bien affectés | 2 |
Trunk opérationnel | 4 |
Inter-VLAN fonctionnel | 4 |
Travail sur les masques | 4 |  
Extention | 2 |  
  
# ✅ Fin du TP

Si vous savez expliquer :
> "Pourquoi deux VLAN ne communiquent pas sans routeur ?"

Alors vous avez compris la segmentation réseau 👍
