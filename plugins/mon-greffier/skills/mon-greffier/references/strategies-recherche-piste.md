# Stratégies de Recherche — API PISTE (Legifrance + Judilibre)

> Ce fichier documente les stratégies de recherche pour les 7 outils MCP `mongreffier-piste`.
> Les exemples utilisent la syntaxe des appels d'outils MCP.

---

## 1. Articles de Codes — `rechercher_code`

### Paramètres disponibles
| Param | Type | Obligatoire | Valeurs possibles |
|-------|------|:-----------:|-------------------|
| code_name | string | ✓ | Nom exact du code (ex: "Code civil") |
| search | string | ✓ | Numéro d'article ou mots-clés |
| champ | string | | ALL, TITLE, TABLE, NUM_ARTICLE, ARTICLE |
| type_recherche | string | | TOUS_LES_MOTS_DANS_UN_CHAMP, EXACTE, UN_DES_MOTS |
| max_results | number | | Défaut: 10, max: 100 |

### Stratégie 1 : Recherche exacte par numéro

```
rechercher_code(
    code_name="Code civil",
    search="1240",
    champ="NUM_ARTICLE"
)
```

### Stratégie 2 : Recherche par contenu

Si numéro introuvable, rechercher par mots-clés du contenu :

```
rechercher_code(
    code_name="Code civil",
    search="responsabilité fait personnel dommage",
    champ="ARTICLE",
    type_recherche="TOUS_LES_MOTS_DANS_UN_CHAMP"
)
```

### Stratégie 3 : Recherche table des matières

Pour localiser un article par son sujet :

```
rechercher_code(
    code_name="Code civil",
    search="responsabilité délictuelle",
    champ="TABLE"
)
```

### Stratégie 4 : Consulter le texte intégral — `consulter_article`

Si un article est trouvé, récupérer son contenu complet pour vérifier les citations :

```
consulter_article(article_id="LEGIARTI000006436298")
```

> L'`article_id` est retourné dans les résultats de `rechercher_code`.

### Cas particuliers — Numérotation

| Préfixe | Signification | Exemple |
|---------|---------------|---------|
| L. | Partie législative | L. 121-1 |
| R. | Réglementaire (décrets en CE) | R. 121-1 |
| D. | Réglementaire (décrets simples) | D. 121-1 |
| A. | Arrêtés | A. 121-1 |

**Important** : Toujours inclure le préfixe exact dans la recherche.
Utiliser `lister_codes_juridiques()` pour obtenir les noms exacts des codes.

### Renumérotations fréquentes (réforme 2016)

| Ancien | Nouveau | Objet |
|--------|---------|-------|
| 1382 | 1240 | Responsabilité délictuelle |
| 1383 | 1241 | Négligence / imprudence |
| 1134 | 1103 | Force obligatoire des contrats |
| 1147 | 1231-1 | Responsabilité contractuelle |
| 1184 | 1224 | Résolution pour inexécution |

---

## 2. Lois et Décrets — `rechercher_dans_texte_legal`

### Paramètres disponibles
| Param | Type | Obligatoire | Valeurs possibles |
|-------|------|:-----------:|-------------------|
| search | string | ✓ | Mots-clés ou numéro d'article |
| text_id | string | | Identifiant du texte (format AAAA-NUMERO) |
| champ | string | | ALL, TITLE, TABLE, NUM_ARTICLE, ARTICLE |
| type_recherche | string | | TOUS_LES_MOTS_DANS_UN_CHAMP, EXACTE, UN_DES_MOTS |
| max_results | number | | Défaut: 10 |

### Stratégie 1 : Par identifiant exact

```
rechercher_dans_texte_legal(
    search="",
    text_id="78-17"
)
```

### Stratégie 2 : Article spécifique dans une loi

```
rechercher_dans_texte_legal(
    search="7",
    text_id="78-17",
    champ="NUM_ARTICLE"
)
```

### Stratégie 3 : Recherche textuelle (identifiant inconnu)

```
rechercher_dans_texte_legal(
    search="informatique libertés données personnelles",
    type_recherche="TOUS_LES_MOTS_DANS_UN_CHAMP",
    max_results=15
)
```

### Formats d'identifiants

| Format | Exemple | Notes |
|--------|---------|-------|
| AAAA-NNN | 78-17 | Standard |
| AAAA-NNNN | 2018-1125 | Numéros > 999 |
| Sans tiret | Recherche textuelle | Fallback si format inconnu |

---

## 3. Jurisprudence — `rechercher_jurisprudence_judiciaire`

### Paramètres disponibles
| Param | Type | Obligatoire | Valeurs possibles |
|-------|------|:-----------:|-------------------|
| search | string | ✓ | N° pourvoi ou mots-clés |
| source | string | | **judilibre** (défaut) ou **legifrance** |
| champ | string | | ALL, TITLE, ABSTRATS, TEXTE, RESUMES, NUM_AFFAIRE (Legifrance uniquement) |
| type_recherche | string | | TOUS_LES_MOTS_DANS_UN_CHAMP, EXACTE, UN_DES_MOTS (Legifrance uniquement) |
| juridiction | array | | Judilibre: cc, ca, etc. / Legifrance: noms complets |
| max_results | number | | Défaut: 10 |

### Double source : Judilibre vs Legifrance

| | Judilibre (défaut) | Legifrance (fond JURI) |
|---|---|---|
| Couverture | Cour de cassation principalement | Cass. + CA + 1ère instance |
| Fraîcheur | Très récent (open data) | Plus d'historique |
| Recherche | Texte libre (query) | Structurée (champ, type_recherche) |
| Filtres spécifiques | juridiction, chambre | juridiction |
| Vitesse | Rapide (GET) | Plus lent (POST) |

**Règle** : Toujours commencer par Judilibre. Basculer sur Legifrance si échec ou si recherche historique.

### Stratégie 1 : Par n° de pourvoi (Judilibre)

```
rechercher_jurisprudence_judiciaire(
    search="21-12.345",
    source="judilibre"
)
```

### Stratégie 2 : Par n° de pourvoi (Legifrance, fallback)

```
rechercher_jurisprudence_judiciaire(
    search="21-12.345",
    source="legifrance",
    champ="NUM_AFFAIRE",
    type_recherche="EXACTE"
)
```

### Stratégie 3 : Par thème (Judilibre)

```
rechercher_jurisprudence_judiciaire(
    search="responsabilité produits défectueux",
    source="judilibre",
    juridiction=["cc"],
    max_results=10
)
```

### Stratégie 4 : Recherche élargie (Legifrance, tous les mots)

```
rechercher_jurisprudence_judiciaire(
    search="accident péage autoroute indemnisation",
    source="legifrance",
    champ="TEXTE",
    type_recherche="UN_DES_MOTS",
    max_results=20
)
```

### Stratégie 5 : Consulter le texte intégral — `consulter_decision`

```
consulter_decision(decision_id="6543abc...")
```

> Le `decision_id` est retourné dans les résultats de `rechercher_jurisprudence_judiciaire`.
> Zones structurées : introduction, exposé, moyens, motivations, dispositif.

### Formats de pourvoi

| Format | Juridiction | Exemple |
|--------|-------------|---------|
| NN-NN.NNN | Cass. civiles | 21-12.345 |
| NN-NNNNN | Cass. criminelle | 21-84567 |
| N/NNNNN | Cours d'appel | 1/12345 |

> Tester les variantes avec/sans point : 21-12345 et 21-12.345

---

## 4. Journal Officiel — `recherche_journal_officiel`

### Paramètres disponibles
| Param | Type | Obligatoire | Valeurs possibles |
|-------|------|:-----------:|-------------------|
| search | string | ✓ | Mots-clés |
| champ | string | | ALL, TITLE, TABLE, NOR, NUM, NUM_ARTICLE, ARTICLE, VISA, NOTICE, VISA_NOTICE, TRAVAUX_PREP, SIGNATURE, NOTA |
| type_recherche | string | | TOUS_LES_MOTS_DANS_UN_CHAMP, EXACTE, UN_DES_MOTS |
| date_publication | array | | Période [date_debut, date_fin] au format YYYY-MM-DD |
| max_results | number | | Défaut: 5 |

### Stratégie 1 : Par date et mots-clés

```
recherche_journal_officiel(
    search="nomination directeur",
    date_publication=["2024-01-01", "2024-12-31"],
    max_results=20
)
```

### Stratégie 2 : Par NOR (identifiant unique)

```
recherche_journal_officiel(
    search="ECOE2400123A",
    champ="NOR"
)
```

---

## 5. Liste des codes — `lister_codes_juridiques`

Aucun paramètre. Retourne la liste complète des codes avec leurs noms exacts.

```
lister_codes_juridiques()
```

> Utiliser cette liste pour valider le `code_name` avant un `rechercher_code`.

---

## Arbres de décision

### Article de code non trouvé

```
1. rechercher_code(champ="NUM_ARTICLE", exacte)
   └─ Échec → 2. lister_codes_juridiques() — vérifier le nom exact du code
              └─ Échec → 3. rechercher_code(autre code probable)
                         └─ Échec → 4. rechercher_code(champ="ARTICLE", mots-clés)
                                    └─ Échec → 5. rechercher_code(champ="TABLE")
                                               └─ Échec → CONCLUSION: REF_INEXISTANTE
   └─ Trouvé → consulter_article(article_id) — vérifier contenu
```

### Jurisprudence non trouvée

```
1. Judilibre : search par n° pourvoi
   └─ Échec → 2. Legifrance : champ=NUM_AFFAIRE, EXACTE
              └─ Échec → 3. Variations format (avec/sans point)
                         └─ Échec → 4. Judilibre : mots-clés thématiques
                                    └─ Échec → 5. Legifrance : champ=TEXTE, UN_DES_MOTS
                                               └─ Échec → CONCLUSION: non_verifiable
                                                          (peut être non publiée)
   └─ Trouvé → consulter_decision(decision_id) — vérifier contenu
```

### Loi non trouvée

```
1. rechercher_dans_texte_legal(text_id exact)
   └─ Échec → 2. Vérifier format (AAAA-NNN vs AAAA-NNNN)
              └─ Échec → 3. Recherche par intitulé (search textuel)
                         └─ Échec → 4. recherche_journal_officiel par date promulgation
                                    └─ Échec → CONCLUSION: REF_INEXISTANTE
```

---

## Pièges courants

| Piège | Solution |
|-------|----------|
| Article avec tiret (L. 121-1) | Tester avec et sans espaces |
| Ancienne numérotation (pré-2016) | Consulter la table de renumérotation ci-dessus |
| Code abrégé ("C. civ.") | Utiliser le nom complet ("Code civil") via lister_codes_juridiques |
| Jurisprudence très récente | Judilibre indexe vite, mais délai possible de quelques jours |
| Décision non publiée au bulletin | Statut "non_verifiable", pas "HALLUCINATION" |
| Judilibre renvoie 0 résultat | Tester Legifrance (fond JURI, couverture plus large) |
| Legifrance 403 sur /consult/code/article/num | Utiliser POST /search avec fond=CODE_DATE à la place |
