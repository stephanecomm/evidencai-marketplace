# Phase 3 — Rédaction

## Prérequis

1. Authentifier le juge si pas encore fait (mongreffier_login)
2. Appeler `mongreffier_read_dossier(dossier_id, for_phase: "redaction")` — ne charge que la phase décisions
3. Lire les réponses du juge : `mongreffier_read_responses(dossier_id, phase_type: "decisions")`
4. Les choix du juge sur chaque point de décision déterminent la rédaction
5. Si relance_context existe → relance, lire la note du juge et l'historique

## Règles de rédaction

- Chaque montant accordé DOIT être motivé individuellement
- Chaque demande rejetée DOIT être motivée
- Formulation impersonnelle du tribunal ("Le Tribunal", pas "nous")
- Visa des textes applicables en tête de chaque point
- Le dispositif reprend EXACTEMENT les décisions prises, sans ajout ni omission
- Exécution provisoire de droit (décret 2019-1333, art. 514 CPC, 1er janvier 2020) : NE RIEN ÉCRIRE pour la motiver. Elle est de droit, zéro motivation systématique.

## Schéma JSON de sortie

Écrire via `mongreffier_write_phase(dossier_id, phase_type: "redaction", llm_output: {...})`

```json
{
  "jugement": {
    "en_tete": "",
    "faits_et_procedure": "",
    "moyens_des_parties": { "demandeur": "", "defendeur": "" },
    "motivation": [
      {
        "id": 0,
        "point": "",
        "analyse_droit": "",
        "analyse_fait": "",
        "fondement": "",
        "decision": ""
      }
    ],
    "dispositif": ""
  }
}
```

## Format du dispositif

Le dispositif doit suivre la formule PCM du skill assistant-tc :
- "Après en avoir délibéré conformément à la loi"
- Vu les articles en italique
- Verbes en MAJUSCULES suivis d'un espace (CONDAMNE, REJETTE, DIT, ORDONNE)
- Montants en lettres et en chiffres
- Point-virgule entre chefs de dispositif
- Formule finale sur les dépens et l'article 700

## Contraintes du champ motivation[]

- `id` : number, index séquentiel à partir de 0
- `point` : intitulé du point (sans préfixe "Sur")
- `analyse_droit` : 2 phrases maximum, cadre juridique applicable
- `analyse_fait` : application au cas d'espèce
- `fondement` : une ligne, références textuelles
- `decision` : la décision du tribunal sur ce point

## Après écriture

1. Dire au juge : "Le projet de jugement est dans votre dashboard. Relisez-le, annotez en marge si besoin, puis validez."
2. Mettre à jour : `mongreffier_update_dossier(dossier_id, current_phase: "redaction")`
3. **NE PAS passer à la suite sans validation explicite du juge**
