# Phase 2 — Décisions

## Prérequis

1. Authentifier le juge si pas encore fait (mongreffier_login)
2. Appeler `mongreffier_read_dossier(dossier_id, for_phase: "decisions")` — ne charge que la phase cadrage (pas les conclusions brutes)
3. Lire les réponses du juge : `mongreffier_read_responses(dossier_id, phase_type: "cadrage")`
4. Intégrer les annotations et observations du juge dans l'analyse
5. Si relance_context existe → relance, lire la note du juge et l'historique

## Méthode d'analyse pour chaque point de décision

Le skill ne résume pas les positions. Pour chaque point il DOIT :

1. **Reformuler la question juridique précise** (pas "le demandeur demande X", mais "la question est de savoir si les conditions de l'article Y sont réunies")
2. **Évaluer la solidité de chaque position** :
   - L'argument repose-t-il sur un texte applicable ?
   - La jurisprudence invoquée est-elle transposable aux faits ?
   - Les conditions d'application sont-elles toutes réunies ?
3. **Identifier ce qui manque** :
   - Condition légale oubliée par l'avocat ?
   - Charge de la preuve remplie ? Par quelle pièce ?
   - Jurisprudence contraire non citée ?
4. **Si le juge a ajouté une référence en annotation**, la vérifier via `mongreffier_verifier_references` (un seul appel batch pour toutes les nouvelles références)
5. **Pour les demandes non contestées** : indiquer le fondement vérifié et la formulation de motivation standard
6. **Dans avis_mongreffier** : être direct. "L'argument du demandeur est faible parce que..." La balance traduit la direction, la confidence la solidité des sources.

## Schéma JSON de sortie

Écrire via `mongreffier_write_phase(dossier_id, phase_type: "decisions", llm_output: {...})`

```json
{
  "points_decision": [
    {
      "id": 1,
      "intitule": "",
      "contexte": "",
      "contestation": "contestee|non_contestee|partiellement_contestee",
      "options": [
        {
          "label": "",
          "fondement_juridique": "",
          "consequence": "",
          "analyse_ia": ""
        }
      ],
      "avis_mongreffier": {
        "balance": 0,
        "confidence": "haute|moyenne|faible",
        "labels": ["Position partie 1", "Position partie 2"],
        "raisonnement": "",
        "fondement_verifie": "",
        "recommandation_prudence": ""
      },
      "fondements": [
        {
          "partie": "demandeur|defendeur",
          "motivation_detaillee": "",
          "articles": [],
          "jurisprudences": [],
          "pieces": []
        }
      ],
      "cascade_rules": []
    }
  ]
}
```

## Enums fixes (contrat avec le dashboard)

- contestation : contestee | non_contestee | partiellement_contestee
- confidence : haute | moyenne | faible
- balance : entier de -100 (favorable défendeur) à +100 (favorable demandeur)
- labels : tableau de 2 strings [position gauche, position droite]

## Champs avis_mongreffier

- `balance` : positif = favorable au demandeur. 0 = équilibré
- `confidence` : "haute" (texte clair + JP constante), "moyenne" (texte applicable, JP variable), "faible" (zone grise)
- `raisonnement` : texte structuré incluant l'analyse de contestation
- `fondement_verifie` : la référence PISTE vérifiée qui fonde la position
- `recommandation_prudence` : mise en garde éventuelle (risque d'appel, JP divisée)

## Champ contestation au niveau du point

- `"non_contestee"` : le raisonnement DOIT indiquer le fondement vérifié et la formulation de motivation standard
- `"partiellement_contestee"` : préciser ce qui est contesté et ce qui ne l'est pas
- `"contestee"` : confronter les deux positions

## Après écriture

1. Dire au juge : "Les points de décision sont dans votre dashboard. Choisissez vos options, annotez, puis validez."
2. Mettre à jour : `mongreffier_update_dossier(dossier_id, current_phase: "decisions")`
3. **NE PAS passer à la suite sans validation explicite du juge**
