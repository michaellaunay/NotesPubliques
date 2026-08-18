---
schema_version: 1
uid: 01M02JG1WES094EWYC7TP139K0
titre: Comparaison de l'assembleur RISC-V avec celui du Montorola 68000
aliases:
- RISC-V vs 68000
type: fiche
statut: actif
para: ressource
domaines:
- communication
themes:
- informatique
- architecture-materielle
- assembleur
- risc-v
resume: 'Comparaison technique entre l''ISA modulaire et ouverte RISC-V et le jeu d''instructions du Motorola 68000 : modularité, pipeline, multitâche, compression, multi-cœur.'
auteurs:
- Michaël Launay
langue: fr
date_creation: 2025-07-18
date_modification: 2026-08-18
confidentialite: publique
publication:
- notes-publiques
rag: true
metadata_verifiees: false
---
Le **RISC-V** est une architecture **moderne**, conçue **40 ans après** le **68000**, avec une approche très différente : celle d’un **ISA minimaliste, modulaire, ouverte et extensible**.

Voici donc **ce que RISC-V propose que le 68000 (et même le 68060) n’a pas**, en termes de **concepts, design et fonctionnalités modernes**.

---

# 🧠 1. **Une architecture modulaire et extensible**

|Caractéristique|**RISC-V**|**68000**|
|---|---|---|
|Modularité ISA|✅ Instructions de base + extensions (M, A, F, D, V…)|❌ Monolithique|
|ISA ouverte|✅ Spécifications libres, sans royalties|❌ Propriétaire Motorola|
|Extensions personnalisées|✅ On peut créer ses propres instructions|❌ Impossible|

👉 En RISC-V, le jeu d’instruction est défini par des extensions :

- `I` = Integer (base)
    
- `M` = Multiply/Divide
    
- `A` = Atomic
    
- `F/D` = Floating point
    
- `V` = Vector
    
- `Z` = Optionnelles (compressed, bitmanip, etc.)
    

---

# 📚 2. **Une spécification claire, moderne, et standardisée**

- **Spécifications formelles en machine-readable (Sail)**
    
- Support direct pour **formal verification**
    
- Très facile à implémenter en FPGA, ASIC, ou émulateur
    

👉 Contrairement au 68k dont l'implémentation dépend de documents historiques ou reverse engineering partiels.

---

# 🚀 3. **Une performance pipeline-friendly**

|Fonction|RISC-V|68060|
|---|---|---|
|Pipeline propre|✅ 5 à 13 étages, prévisible|⚠️ Super-scalaire, mais pas RISC|
|Instruction taille fixe|✅ 32 bits (compactable en 16 bits)|❌ Variable, décodeur complexe|
|Pas de microcode|✅ Direct hardware control|❌ Oui (68000 → microcode CISC)|

👉 Cela permet à RISC-V d’être **extrêmement performant**, même sur microcontrôleurs ou CPU haut de gamme.

---

# 🛠️ 4. **Meilleure prise en charge du multitâche et des OS modernes**

|Fonction|RISC-V|68k|
|---|---|---|
|Mode utilisateur / superviseur / machine|✅ 3 niveaux clairs (M/S/U)|❌ Pas sur le 68000 (apparaît avec 68020 + MMU)|
|CSR (Control & Status Registers)|✅ Oui|❌ Non|
|Instructions atomiques|✅ Avec extension `A`|❌ Gestion manuelle + pas d’instruction atomique|
|Virtualisation / hyperviseur|✅ Extensions `H`|❌ Jamais prévu|

---

# 📉 5. **Compression et efficacité mémoire**

- **RISC-V "C"** : une extension d'instructions **compressées 16 bits**, pour économiser :
    
    - **taille mémoire**
        
    - **bande passante**
        
    - **temps d’exécution**
        

👉 Aucun équivalent dans 68k. Les instructions sont souvent longues et varient de 2 à 6 octets.

---

# 🧩 6. **Support natif du multi-cœur, des accès atomiques, de la vectorisation**

|Fonction|RISC-V|68060|
|---|---|---|
|Multi-cœur|✅ Très bien supporté|❌ Pas prévu|
|Instructions atomiques|✅ `lr/sc`, `amo*`|❌|
|Instructions vectorielles|✅ Extension `V`|❌ (pas même en FPU)|

---

# 🌐 7. **Écosystème open-source massif**

- Outils libres :
    
    - [GCC](https://gcc.gnu.org/)
        
    - [LLVM](https://llvm.org/)
        
    - [QEMU](https://qemu.org/)
        
    - [Spike](https://github.com/riscv-software-src/riscv-isa-sim)
        
    - [Renode](https://renode.io/)
        
    - [Verilator](https://www.veripool.org/)
        
- OS supportés :
    
    - Linux
        
    - Zephyr, RTOS
        
    - FreeRTOS, NuttX
        
    - Plan 9, xv6, etc.
        

👉 Contrairement au 68k qui dépendait de l’écosystème Motorola ou de plateformes spécifiques (AmigaOS, MacOS classique…).

---

# 🛡️ 8. **Conçu pour la sécurité moderne**

- **Protection mémoire par design**
    
- Extensions cryptographiques (`Zk`)
    
- Extensions de contrôle d’accès mémoire (`PMP`)
    
- **Pas de microcode caché → moins de bugs type Spectre/Meltdown**
    

---

# 🏁 Résumé

|Capacité|RISC-V|Motorola 680x0|
|---|---|---|
|Modularité ISA|✅|❌|
|Open source & libre|✅|❌|
|Performance pipeline|✅|⚠️ (68060 seulement)|
|Compression d’instructions|✅ (`C`)|❌|
|Sécurité moderne|✅|❌|
|Multi-cœur natif|✅|❌|
|Extensions vectorielles|✅ (`V`)|❌|
|Écosystème open-source actif|✅|❌|

---

# ⚖️ Synthèse rapide

|Aspect|**Motorola 68k**|**RISC-V**|
|---|---|---|
|Type d’architecture|CISC (Complex)|RISC (Reduced)|
|Date de création|1979|2010s (UC Berkeley)|
|Registres|8 data (D0–D7), 8 adresse (A0–A7)|32 généralistes (`x0–x31`)|
|Taille instructions|Variable (16 à 96 bits)|Fixe (32 bits standard)|
|Lisibilité|Très lisible, proche du langage humain|Simple, brut, ultra prévisible|
|Complexité|Moyenne à élevée|Très faible|
|Accès mémoire|Multiples modes (pré-décrément, post-incrément, indirect, immédiat...)|Charge/Store uniquement (`lw`, `sw`, etc.)|
|Orthogonalité|Élevée|Totale|
|Syntaxe assembleur|Élégante et explicite|Minimaliste et normalisée|

---

# 📚 Exemple concret : Addition

## 🔹 Motorola 68k

```asm
MOVE.L  #5, D0       ; D0 = 5
MOVE.L  #10, D1      ; D1 = 10
ADD.L   D1, D0       ; D0 = D0 + D1
```

## 🔹 RISC-V

```asm
li x5, 5            # x5 = 5
li x6, 10           # x6 = 10
add x7, x5, x6      # x7 = x5 + x6
```

🧠 **Observations** :

- 68k est **plus proche de l’humain** ("MOVE", "ADD")
- RISC-V est **plus bas niveau**, mais plus systématique (`add dest, src1, src2`)

---

# 🧮 Exemple mémoire : lecture/sauvegarde

## 🔹 68k (complexe mais puissant)

```asm
MOVE.L 100(A0), D0      ; D0 = [A0 + 100]
MOVE.L D0, -(A7)        ; push D0 sur la pile (post-décrément)
```

## 🔹 RISC-V (Load/Store only)

```asm
lw x5, 100(x10)         # x5 = [x10 + 100]
addi x2, x2, -4         # décrémente stack pointer
sw x5, 0(x2)            # push x5 sur pile
```

👀 En 68k, un `MOVE` fait souvent **plusieurs choses à la fois**, tandis qu’en RISC-V, **chaque instruction fait une seule chose simple**.

---

# 🧩 Gestion de la pile

|Action|68k|RISC-V|
|---|---|---|
|Pousser une valeur|`MOVE.L D0, -(A7)`|`addi sp, sp, -4` + `sw xN, 0(sp)`|
|Retirer une valeur|`MOVE.L (A7)+, D0`|`lw xN, 0(sp)` + `addi sp, sp, 4`|

🔄 Le 68k gère **automatiquement** les décalages de pile ; RISC-V laisse tout à l’utilisateur.

---

# 🛠️ Fonctions et appels

## 🔹 68k

```asm
JSR Subroutine         ; Jump to subroutine
RTS                    ; Return from subroutine
```

## 🔹 RISC-V

```asm
jal ra, subroutine     # Jump and link
ret                    # Pseudo : jalr x0, 0(ra)
```

🧠 Le **JSR/RTS** du 68k est plus simple.  
Le **jal/jalr** du RISC-V est plus général et plus explicite.

---

# ✅ Avantages et inconvénients

## 🔷 Motorola 68k

**Avantages :**

- Très lisible et "humain"
    
- ISA élégante et orthogonale
    
- Beaucoup de modes d’adressage pratiques
    

**Inconvénients :**

- Instructions variables (difficiles à décoder)
    
- Complexe à implémenter en matériel
    
- Moins adapté à la vectorisation / modernisation
    

---

## 🔶 RISC-V

**Avantages :**

- ISA minimaliste, modulaire et extensible
    
- Facile à décoder et à implémenter (FPGA, éducation)
    
- Parfait pour les compilateurs et l’optimisation
    

**Inconvénients :**

- Verbeux (beaucoup d’instructions pour peu d'effet)
    
- Moins naturel à lire pour les humains
    
- Peu de modes d’adressage → tout est manuel
    

---

# 🏁 Conclusion

- Le **68k est une œuvre d'art d’architecture CISC** : expressif, compact, mais complexe    
- Le **RISC-V est une base pédagogique et industrielle** : simple, prévisible, mais spartiate

Base de développement RISC-V très performant [StarPro64](https://pine64.org/devices/starpro64)