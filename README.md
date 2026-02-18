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

---

# 🔹 PARTIE 5 — Configuration IP des PC

Configurer IP + passerelle selon le plan d’adressage.

---

# 🧪 TESTS

## Test 1 — Intra-VLAN
PC1 → PC2  
👉 Doit fonctionner

<img width="1332" height="1024" alt="image" src="https://github.com/user-attachments/assets/c0315058-fe5d-415c-9bae-694c6963c03c" />


---

## Test 2 — Inter-VLAN
PC1 → PC3  
👉 Fonctionne uniquement grâce au routeur

<img width="1332" height="1024" alt="image" src="https://github.com/user-attachments/assets/43cb60e8-19bb-44eb-b3d4-8e32e5aa4a67" />
  
---

# ❓ Questions de réflexion

1. Pourquoi PC1 ne voit-il pas PC3 sans routeur ? -> Répondez directement sur ce Readme.md
PC1 ne voit pas PC3 sans routeur car isl font partie de VLANs différents. Ils sont isolés l'un de l'autre.
   
3. Quel rôle joue le masque /24 ? -> Répondez directement sur ce Readme.md
Le masque /24 attribue les 3 premiers octets de l'adresse ip au réseau, laissant le dernier octet à l'identifiant matériel au sein du même réseau.
PC1 a une ip de 192.168.10.10
PC3 a une ip de 192.168.20.10
Le troisième octet est différent, donc le réseau est différent ce qui explique qu'ils ne puissent pas se voir sans routeur.
   
5. Que se passe-t-il si VLAN 10 et VLAN 20 ont le même réseau IP ? -> Répondez directement sur ce Readme.md
Si les deux VLAN utilisent le même adressage (ex: 192.168.10.0/24), le conflit d'adressage empêchera le routage. Le routeur (R1) ne pourra pas avoir deux sous-interfaces (g0/0.10 et g0/0.20)    dans le même réseau, car il ne saurait pas vers quelle interface renvoyer les paquets.

7. Pourquoi un trunk est-il nécessaire ? -> Répondez directement sur ce Readme.md
Le trunk est nécessaite car il permet de faire transiter les flux de plusieurs VLANs sur un seul câble physique. Sans le taggage, le switch et le routeur ne peuvent pas savoir si un paquet appartient au VLAN 10 ou au VLAN 20 lors du passage par le port f0/24.

---

# ⭐ Travail sur les Masques

Changer VLAN 10 en :

```
192.168.10.0/25
```

Questions :
- Combien d’hôtes max ?
Il y a 126 hotes au maximum avec un CIDR de 25 (128-2).

- Quelle plage IP valide ?
Réseau : 192.168.10.0
Première IP : 192.168.10.1 (ici la Gateway)
Dernière IP : 192.168.10.126
Broadcast : 192.168.10.127

- Peut-on encore communiquer avec VLAN 20 ?
Oui, à condition de modifier l'adresse et le masque de la sous-interface du routeur (g0/0.10).
---

# 🚀 Extensions

- Ajouter VLAN 30
J'ai ajouté une VLAN 30 que j'ai appelée GUEST.
<img width="852" height="434" alt="image" src="https://github.com/user-attachments/assets/6c9e296b-11b4-4bf6-a050-66184b30ac27" />

- Mettre un DHCP par VLAN  
On fait une capture du résultat show dhcp binding
<img width="737" height="127" alt="image" src="https://github.com/user-attachments/assets/8cdf4475-5e4f-4ad5-90c8-034fbc36e153" />

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
