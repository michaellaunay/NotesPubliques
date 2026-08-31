---
schema_version: 1
uid: "01M02JG1WCFPJQ1Z4NYK4RP3SZ"
titre: "Algorithme de compression basé sur PI"
type: idee
statut: a-relire
para: ressource
domaines:
  - communication
themes:
  - informatique
  - algorithmique
  - compression
  - mathematiques
resume: "Exercice pour le cours de Python : un « compresseur » qui remplace les données par des index dans les décimales de π — énoncé, corrigé testé (mpmath, recherche gloutonne, bilan en bits) et leçon : l'index coûte autant que la donnée, l'entropie interdit la compression moyenne."
maturite: validee
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-01-04
date_modification: 2026-08-31
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---

> [!info] Idée devenue exercice
> Idée notée en janvier 2024 ; énoncé, corrigé et leçon écrits et testés le 31 août 2026 (audit du dossier réflexions, point 26). L'observation d'origine — huit nombres de cinq chiffres absents du premier million de décimales — a été revérifiée : la liste est exacte.

# L'idée d'origine (janvier 2024)

Dans la série des exercices pour mon cours de [[Python]] 
Je propose de faire un algorithme de compression qui se base sur la suite de décimale de PI.
Ainsi nous pourrions chercher les morceaux de données à compresser dans une table des décimales de PI.
La sortie aurait alors le format suivant :
Longueur total de la donnée décompressée
Nombre de tuples
Nombre de décimales en base 256 (octet) requis (le nombre de décimales de pi à calculer si on n'a pas une table).
Liste de tuples (index, longueur)

Pour décompresser générer les décimales de pi en base 256 (octet),
Puis pour chaque tuple copier la sous chaîne des décimales d'index et de longueur donnés par le tuple.

Pour les 10⁶ premières décimales, il existe 8 nombres de 5 digits n'ayant pas de représentation ['14523', '17125', '22801', '33394', '36173', '39648', '40527', '96710'].

Voir [Une formule ÉTRANGE | Axel Arno](https://youtu.be/MxcRztpXKFQ?si=rSVm8nF9l8mhjVQI) et [Une formule STUPÉFIANTE | Axel Arno](https://youtu.be/nZRKNth6OSA?si=Wo_n8xfmoHB0c3XL)

# Énoncé

On dispose d'une table des `N` premières décimales de π sous forme d'une chaîne de chiffres. L'idée est de « compresser » une donnée en la remplaçant par des positions dans cette table.

1. Générer la table avec `mpmath` (un million de décimales suffit ; conserver le résultat dans un fichier, le calcul prend une dizaine de secondes).
2. Écrire `octets_vers_chiffres(donnees: bytes) -> str`, qui représente chaque octet par trois chiffres décimaux, et sa réciproque `chiffres_vers_octets`.
3. Écrire `compresser(chiffres, table, lmax) -> list[tuple[int, int]]` : en partant du début, chercher le plus long préfixe (au plus `lmax` chiffres) présent dans la table avec `str.find`, émettre le tuple `(index, longueur)`, avancer, recommencer.
4. Écrire `decompresser(tuples, table) -> str` et vérifier que la composition des deux redonne les octets d'origine.
5. Mesurer : un tuple coûte `ceil(log2(N))` bits pour l'index et `ceil(log2(lmax + 1))` bits pour la longueur. Comparer le total au nombre de bits de la donnée d'origine, sur un texte court, puis sur les 256 valeurs d'octet. Que constate-t-on ? Comment la longueur moyenne d'un motif trouvé dépend-elle de `N` ?
6. Expliquer pourquoi le résultat de la question 5 n'est pas une malchance mais une nécessité. Quelle hypothèse sur π faudrait-il pour garantir que toute donnée est représentable ?

Variante : refaire l'exercice avec des décimales en base 256 (un octet par « chiffre »), comme le proposait l'idée d'origine — le bilan ne change pas, seuls les logarithmes changent de base.

# Corrigé

```python
import math
import mpmath

def table_pi(n: int) -> str:
    """Les n premiers chiffres de pi (3 compris), sous forme de chaîne."""
    mpmath.mp.dps = n + 10
    return mpmath.nstr(mpmath.mp.pi, n + 5, strip_zeros=False).replace(".", "")[:n]

def octets_vers_chiffres(donnees: bytes) -> str:
    return "".join(f"{o:03d}" for o in donnees)

def chiffres_vers_octets(chiffres: str) -> bytes:
    return bytes(int(chiffres[i:i + 3]) for i in range(0, len(chiffres), 3))

def compresser(chiffres: str, table: str, lmax: int = 15) -> list[tuple[int, int]]:
    tuples, i = [], 0
    while i < len(chiffres):
        longueur = min(lmax, len(chiffres) - i)
        while longueur > 0:                       # plus long préfixe présent dans la table
            index = table.find(chiffres[i:i + longueur])
            if index != -1:
                tuples.append((index, longueur))
                i += longueur
                break
            longueur -= 1
        else:
            raise ValueError("un chiffre manque dans la table")   # impossible dès que N >= 33
    return tuples

def decompresser(tuples: list[tuple[int, int]], table: str) -> str:
    return "".join(table[i:i + n] for i, n in tuples)

def bilan(donnees: bytes, table: str, lmax: int = 15) -> None:
    chiffres = octets_vers_chiffres(donnees)
    tuples = compresser(chiffres, table, lmax)
    assert chiffres_vers_octets(decompresser(tuples, table)) == donnees
    bits_index = math.ceil(math.log2(len(table)))
    bits_longueur = math.ceil(math.log2(lmax + 1))
    brut, comprime = len(donnees) * 8, len(tuples) * (bits_index + bits_longueur)
    print(f"{len(donnees)} octets = {brut} bits -> {len(tuples)} tuples = {comprime} bits "
          f"(x{comprime / brut:.2f}), motif moyen : {len(chiffres) / len(tuples):.1f} chiffres")

table = table_pi(1_000_000)
bilan(b"Compression par pi", table)
bilan(b"Le petit chat est mort.", table)
bilan(bytes(range(256)), table)
```

Résultats obtenus avec un million de décimales (`lmax = 15`) :

| Donnée | Brut | Tuples | « Compressé » | Rapport | Motif moyen |
|---|---:|---:|---:|---:|---:|
| `Compression par pi` (18 octets) | 144 bits | 10 | 240 bits | × 1,67 | 5,4 chiffres |
| `Le petit chat est mort.` (23 octets) | 184 bits | 12 | 288 bits | × 1,57 | 5,8 chiffres |
| les 256 valeurs d'octet | 2 048 bits | 133 | 3 192 bits | × 1,56 | 5,8 chiffres |

Le « compresseur » multiplie la taille par 1,6, et le motif moyen trouvé fait environ six chiffres — c'est-à-dire log₁₀ N.

# La leçon

Dans une table de `N` chiffres qui se comportent comme des chiffres au hasard, une chaîne de `k` chiffres donnée a une chance sur 10ᵏ d'apparaître à une position donnée ; il faut donc en moyenne 10ᵏ positions pour la rencontrer, et le plus long motif que l'on peut espérer trouver dans la table est de l'ordre de log₁₀ N chiffres. Or coder une position dans la table coûte log₂ N ≈ 3,32 × log₁₀ N bits : l'index coûte exactement ce que valent les chiffres qu'il désigne, 3,32 bits par chiffre décimal, qui est l'entropie d'un chiffre uniforme — et la longueur du motif s'ajoute par-dessus. L'expansion de 1,6 mesurée est ce surcoût.

Le résultat est général, et ne tient pas à π : aucun procédé sans perte ne peut raccourcir toutes les données, car il n'y a que 2ⁿ − 1 messages de moins de n bits pour 2ⁿ messages de n bits (argument des tiroirs). Un vrai compresseur ne gagne que sur les données redondantes, en apprenant leur structure ; les décimales de π ne connaissent pas les nôtres.

Deux remarques pour finir. Rien ne garantit même que toute chaîne finie figure dans π : ce serait le cas si π était un nombre normal, ce qui est conjecturé et non démontré — d'où les huit nombres de cinq chiffres absents du premier million de décimales, là où une suite aléatoire en laisserait environ 4,5 (10⁵ × e⁻¹⁰). Et l'idée a déjà été implémentée pour rire : πfs, un système de fichiers qui stocke chaque fichier comme un index dans π (Philip Langdale, 2012, https://github.com/philipl/pifs), dont le README annonce fièrement un taux de compression de 100 %.
