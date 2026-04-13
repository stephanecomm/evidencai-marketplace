# Phase 1 — Cadrage

## Prérequis

1. Authentifier le juge si pas encore fait (mongreffier_login)
2. Appeler `mongreffier_read_dossier(dossier_id, include_conclusions: true)` pour charger les conclusions
3. Si relance_context existe sur la phase cadrage → c'est une relance, lire la note du juge et l'historique

## Travail du LLM

### Analyse critique des conclusions

**Règle 1 : Ne pas se noyer dans la "soupe" des avocats.**
Les conclusions mélangent fait, droit, rhétorique et émotion. Pour CHAQUE demande, extraire :
- La prétention juridique précise (ce qui est demandé, sur quel fondement)
- Le syllogisme réel : règle de droit → faits allégués → conclusion
- Si le syllogisme est incomplet ou incohérent, le signaler dans alertes

**Règle 2 : Confronter chaque argument au droit positif.**
Quand un avocat cite un article, ne pas se contenter de vérifier que l'article existe. Vérifier que :
- L'article est applicable au cas d'espèce (bon code, bon contexte, en vigueur)
- L'interprétation qu'en fait l'avocat est fidèle au texte
- La jurisprudence citée confirme bien cette interprétation (et pas l'inverse)

**Règle 3 : Analyser les prétentions NON contestées et les arguments sans réponse.**
Le silence d'une partie sur un argument adverse est un SIGNAL, pas un aveu.

**3a. Demandes non contestées :**
1. Identifier chaque demande non contestée
2. Le fondement sera vérifié par mongreffier_verifier_references (batch)
3. Signaler au juge : "Demande non contestée, fondement vérifié : [résultat]" ou "Demande non contestée MAIS fondement fragile : [raison]"
Le juge doit pouvoir motiver même les demandes acquises.

**3b. Arguments sans réponse :**
1. Identifier chaque argument soulevé par une partie auquel l'adversaire ne répond pas
2. Trois hypothèses à distinguer :
   - Argument juridiquement inopérant (JP constante contraire) → silence logique
   - Argument fondé et non contesté → signal fort
   - Argument discutable → laisser l'appréciation au juge
3. Ne JAMAIS présumer qu'un silence cache quelque chose sans avoir vérifié le droit

**Règle 4 : Détecter les arguments "écran de fumée".**
Signaler dans alertes : arguments circulaires, moyens sans fondement textuel, confusions de fondement, jurisprudences hors contexte.

### Vérification juridique — Batch PISTE

Collecter TOUTES les références citées par les parties (articles + jurisprudences) avec leur contexte d'usage, puis appeler `mongreffier_verifier_references` en un seul appel :

```json
{
  "references": [
    {
      "type": "article",
      "code": "Code civil",
      "numero": "1240",
      "partie": "demandeur",
      "contexte_usage": "Responsabilité délictuelle pour faute dans l'exécution du contrat"
    },
    {
      "type": "jurisprudence",
      "numero_pourvoi": "21-12.345",
      "partie": "defendeur",
      "contexte_usage": "Absence de mise en demeure préalable"
    }
  ]
}
```

Le batch renvoie pour chaque référence : statut d'existence, score, texte résumé (~500 chars), pré-évaluation de pertinence par Haiku, lien Legifrance.

**Le LLM principal reste maître du verdict final.** Si la pré-évaluation Haiku est douteuse (score entre 40 et 70), ou si le contexte du dossier rend la référence suspecte, utiliser les outils unitaires (consulter_article, consulter_decision) pour trancher.

Si PISTE est indisponible : signaler au juge "L'API PISTE n'est pas disponible. Les références n'ont pas pu être vérifiées automatiquement. Vérifiez-les manuellement." Continuer le cadrage avec statut "a_verifier" sur toutes les références.

## Schéma JSON de sortie

Écrire via `mongreffier_write_phase(dossier_id, phase_type: "cadrage", llm_output: {...})`

```json
{
  "parties": {
    "demandeur": { "nom": "", "forme": "", "rcs_siret": "", "siege": "", "representation": "" },
    "defendeur": { "nom": "", "forme": "", "rcs_siret": "", "siege": "", "representation": "" }
  },
  "qualification": {
    "type": "contradictoire|repute_contradictoire|defaut",
    "ressort": "premier_ressort|dernier_ressort",
    "motivation": ""
  },
  "fins_non_recevoir": [
    {
      "type": "prescription|forclusion|chose_jugee|defaut_qualite|defaut_interet|clause_conciliation",
      "soulevee_par": "demandeur|defendeur|ia_detection",
      "fondement": "",
      "analyse": "",
      "calcul_delai": { "fait_generateur": "", "date_assignation": "", "delai_ecoule": "", "delai_applicable": "" }
    }
  ],
  "demandes": [
    {
      "partie": "demandeur|defendeur",
      "objet": "",
      "montant": "",
      "fondement": "",
      "pieces": []
    }
  ],
  "articles_cites": [
    {
      "reference": "",
      "partie": "demandeur|defendeur",
      "objet": "",
      "lien_legifrance": "",
      "statut": "VERIFIE_EXACT|VERIFIE_MINEUR|PARTIELLEMENT_EXACT|SUBSTANTIELLEMENT_ERRONE|HALLUCINATION|a_verifier",
      "score_verification": 0,
      "type_erreur": null,
      "correction": null,
      "alerte": ""
    }
  ],
  "jurisprudences": [
    {
      "reference": "",
      "partie": "demandeur|defendeur",
      "principe": "",
      "lien_legifrance": "",
      "statut": "VERIFIE_EXACT|VERIFIE_MINEUR|PARTIELLEMENT_EXACT|SUBSTANTIELLEMENT_ERRONE|HALLUCINATION|non_verifiable",
      "score_verification": 0,
      "type_erreur": null,
      "correction": null
    }
  ],
  "points_a_verifier": [
    {
      "id": 1,
      "priorite": "haute|moyenne|basse",
      "question": "",
      "piece": "",
      "enjeu": ""
    }
  ],
  "ordre_examen": [],
  "alertes": [
    { "severite": "critique|important|mineur", "message": "" }
  ],
  "verification_synthese": {
    "total_references": 0,
    "exactes": 0,
    "erreurs_mineures": 0,
    "erreurs_majeures": 0,
    "hallucinations": 0,
    "non_verifiables": 0,
    "score_global": 0
  }
}
```

## Enums fixes (contrat avec le dashboard)

- statut article : VERIFIE_EXACT | VERIFIE_MINEUR | PARTIELLEMENT_EXACT | SUBSTANTIELLEMENT_ERRONE | HALLUCINATION | a_verifier
- statut jurisprudence : VERIFIE_EXACT | VERIFIE_MINEUR | PARTIELLEMENT_EXACT | SUBSTANTIELLEMENT_ERRONE | HALLUCINATION | non_verifiable
- severite : critique | important | mineur
- priorite : haute | moyenne | basse
- qualification type : contradictoire | repute_contradictoire | defaut
- score_verification : entier de 0 à 100

## Après écriture

1. Dire au juge : "Le cadrage est prêt dans votre dashboard. Vérifiez les points, cochez et annotez. Revenez ici quand vous avez validé."
2. Mettre à jour : `mongreffier_update_dossier(dossier_id, current_phase: "cadrage")`
3. **NE PAS passer à la suite tant que le juge n'a pas explicitement validé**
