# Phase 4 — Robustesse

## Prérequis

1. Authentifier le juge si pas encore fait (mongreffier_login)
2. Appeler `mongreffier_read_dossier(dossier_id, for_phase: "robustesse")` — ne charge que la phase rédaction
3. Lire les réponses du juge : `mongreffier_read_responses(dossier_id, phase_type: "redaction")`
4. Si le juge a ajouté des références en annotation, les vérifier via `mongreffier_verifier_references`
5. Si relance_context existe → relance, lire la note du juge et l'historique

## Travail du LLM

Relire le projet de jugement (llm_output de la phase redaction) et analyser :

1. **Angles morts** : moyens non examinés, arguments ignorés, demandes sur lesquelles le tribunal n'a pas statué
2. **Risques d'appel** : motivation insuffisante, erreur de droit, défaut de base légale, ultra petita, infra petita
3. **Vérification des références** : les articles et JP cités dans la rédaction sont-ils tous cohérents avec le cadrage vérifié ?
4. **Synthèse** : score global de robustesse (A/B/C), points forts, recommandations prioritaires

## Filet de sécurité exécution provisoire

Si le jugement contient une motivation de l'exécution provisoire alors qu'elle est de droit (art. 514 CPC, décisions rendues après le 1er janvier 2020), signaler dans risques_appel avec severite "important".

## Schéma JSON de sortie

Écrire via `mongreffier_write_phase(dossier_id, phase_type: "robustesse", llm_output: {...})`

```json
{
  "angles_morts": [
    {
      "categorie": "moyen_non_examine|argument_ignore|demande_non_statuee",
      "severite": "critique|important|mineur",
      "section": "",
      "probleme": "",
      "solution": ""
    }
  ],
  "risques_appel": [
    {
      "categorie": "motivation_insuffisante|erreur_droit|defaut_base_legale|ultra_petita|infra_petita",
      "severite": "critique|important|mineur",
      "section": "",
      "probleme": "",
      "solution": ""
    }
  ],
  "verification_references": {
    "articles_non_verifies": [],
    "jurisprudences_non_verifiees": [],
    "alertes_verification": []
  },
  "synthese": {
    "score": "A|B|C",
    "points_forts": [],
    "recommandations_prioritaires": []
  }
}
```

## Enums fixes

- categorie angles_morts : moyen_non_examine | argument_ignore | demande_non_statuee
- categorie risques_appel : motivation_insuffisante | erreur_droit | defaut_base_legale | ultra_petita | infra_petita
- severite : critique | important | mineur
- score : A (solide) | B (acceptable, corrections mineures) | C (fragile, corrections nécessaires)

## Après écriture

1. Dire au juge : "L'analyse de robustesse est dans votre dashboard. Vous pouvez cocher les corrections à intégrer, ou valider et exporter directement."
2. Mettre à jour : `mongreffier_update_dossier(dossier_id, current_phase: "robustesse")`
3. Le juge a deux options dans le dashboard :
   - **Valider et exporter** → fin du workflow, pas de phase 5
   - **Cocher des corrections** → le dashboard affiche la commande Cowork pour la phase 5
