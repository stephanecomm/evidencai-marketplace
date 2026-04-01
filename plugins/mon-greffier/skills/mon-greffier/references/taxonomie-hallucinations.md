# Taxonomie des Hallucinations Juridiques

## Classification par Type d'Erreur

### Gravité CRITIQUE

#### REF_INEXISTANTE
- **Définition** : Article, loi, décret ou code cité n'existe pas
- **Exemples** : "article 2847 du Code civil", "Code du numérique"
- **Score** : 0-10
- **Détection** : Aucun résultat après recherches multiples

#### CONTENU_DEFORME
- **Définition** : Citation verbatim ne correspondant pas au texte officiel
- **Exemples** : Omission de la notion de "faute" dans art. 1240
- **Score** : 20-40
- **Détection** : Comparaison textuelle avec source officielle

#### JURISPRUDENCE_FICTIVE
- **Définition** : Décision de justice inventée
- **Indices** : Numéro de pourvoi inexistant, date/formation incohérentes
- **Score** : 0-15
- **Détection** : Recherche par numéro d'affaire sans résultat

### Gravité HAUTE

#### NUM_ERRONE
- **Définition** : Numéro d'article ou de texte incorrect
- **Exemples** : "article 1241" pour la responsabilité du fait des choses (= 1242)
- **Score** : 30-50
- **Détection** : Article existe mais contenu différent

#### CODE_ERRONE
- **Définition** : Attribution à un mauvais code juridique
- **Exemples** : "article L.442-1 du Code civil" (= Code de commerce)
- **Score** : 25-40
- **Détection** : Article existe dans un autre code

#### ABROGATION_IGNOREE
- **Définition** : Texte abrogé ou renuméroté cité comme en vigueur
- **Exemples** : "article 1382 du Code civil" (= 1240 depuis 2016)
- **Score** : 35-50
- **Détection** : Vérifier tables de concordance

#### CITATION_PARTIELLE
- **Définition** : Extrait correct mais contexte omis changeant le sens
- **Exemples** : Principe cité sans ses exceptions
- **Score** : 50-70
- **Détection** : Analyse du contexte complet de l'article

### Gravité MOYENNE

#### DATE_ERRONEE
- **Définition** : Date de promulgation ou publication incorrecte
- **Exemples** : "loi du 6 mars 1978" (= 6 janvier 1978)
- **Score** : 60-80
- **Détection** : Vérification des métadonnées officielles

#### JURIDICTION_ERRONEE
- **Définition** : Mauvaise juridiction ou formation citée
- **Exemples** : "Chambre commerciale" pour un arrêt de la 1re civile
- **Score** : 55-75
- **Détection** : Vérification formation dans décision

#### CONFUSION_TERME
- **Définition** : Mélange entre notions juridiques distinctes
- **Exemples** : "vendeur" au lieu de "acquéreur" pour garantie vices cachés
- **Score** : 50-65
- **Détection** : Analyse sémantique du contenu

## Matrice de Décision

```
Référence trouvée ?
├─ NON → Recherches alternatives épuisées ?
│        ├─ OUI → REF_INEXISTANTE (score 0-10)
│        └─ NON → Continuer recherches
│
└─ OUI → Contenu correspond ?
         ├─ NON → Type d'écart ?
         │        ├─ Numéro différent → NUM_ERRONE
         │        ├─ Code différent → CODE_ERRONE
         │        ├─ Texte abrogé → ABROGATION_IGNOREE
         │        ├─ Citation inexacte → CONTENU_DEFORME
         │        └─ Contexte manquant → CITATION_PARTIELLE
         │
         └─ OUI → En vigueur ?
                  ├─ OUI → VERIFIE_EXACT (score 100)
                  └─ NON → ABROGATION_IGNOREE
```

## Calcul du Score

### Formule de Base

```
Score = 100 - (Pénalité_Type × Coefficient_Gravité)
```

### Coefficients

| Gravité | Coefficient | Pénalité Base |
|---------|-------------|---------------|
| CRITIQUE | 1.0 | 80-100 |
| HAUTE | 0.7 | 50-70 |
| MOYENNE | 0.4 | 20-40 |
| MINEURE | 0.2 | 5-15 |

### Ajustements

- **Texte fondamental** (art. 1240, 9 C.civ) : +10% pénalité
- **Citation verbatim présente** : vérification stricte requise
- **Absence de date** : -5% pénalité (référence incomplète mais pas fausse)
- **Contexte professionnel** (conclusion, mémoire) : +15% pénalité
