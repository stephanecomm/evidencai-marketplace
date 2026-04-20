# Phase 5 — Rédaction définitive

## Déclenchement

La commande Cowork indique "RETOUR ROBUSTESSE — Le juge a sélectionné des corrections."

## Prérequis

1. Identité du juge confirmée via OAuth (Cowork a déjà géré l'auth au premier appel MCP). Au besoin : `mongreffier_whoami`.
2. **Lire les préférences du juge** : `mongreffier_get_profile()` (l'identité est déduite du token) → `preferences.juridiction`, `preferences.ville`, `preferences.formule_pcm_override`, `preferences.vocabulaire_local`, `preferences.signature`
3. Appeler `mongreffier_read_dossier(dossier_id, for_phase: "redaction_definitive")` — charge les phases rédaction + robustesse
4. Lire les réponses du juge : `mongreffier_read_responses(dossier_id, phase_type: "robustesse")`
4. Le champ juge_responses contiendra :
   - `action: "redaction_definitive"`
   - `corrections_selectionnees` : indices des angles morts et risques d'appel à intégrer
   - `annotations` du juge sur chaque point
   - `observations` générales
5. Relire la rédaction initiale (phase redaction, llm_output) depuis le read_dossier

## Travail du LLM

- Intégrer les corrections cochées : corriger les motivations insuffisantes, ajouter les moyens oubliés, renforcer les bases légales
- Respecter les annotations du juge sur chaque point
- La rédaction initiale (phase 3) reste intacte en base, la version corrigée est une NOUVELLE phase
- Si le juge a mentionné de nouvelles références dans ses annotations, les vérifier via `mongreffier_verifier_references`

## Schéma JSON de sortie

Écrire via `mongreffier_write_phase(dossier_id, phase_type: "redaction_definitive", llm_output: {...})`

Format IDENTIQUE à la phase rédaction :

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

Mêmes contraintes que la phase rédaction : point sans préfixe "Sur", analyse_droit 2 phrases max, fondement une ligne, dispositif complet avec formule PCM (voir `phase-3-redaction.md` § "Formule PCM introductive" pour les règles d'interpolation et la formule par défaut art. 450 CPC).

## Après écriture

1. Le juge voit le texte corrigé dans le dashboard, découpé en sections éditables
2. Pas d'intervention LLM à cette étape : le juge a le dernier mot sur la plume
3. Quand il est satisfait, il valide et exporte le projet de jugement (Word)
4. Mettre à jour : `mongreffier_update_dossier(dossier_id, current_phase: "redaction_definitive")`
