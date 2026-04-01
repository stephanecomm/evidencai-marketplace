# Mon Greffier

Assistant du juge consulaire au Tribunal de Commerce.
Plugin Cowork par EvidencAI.

## Ce que fait le plugin

Mon Greffier aide les juges consulaires a rediger des projets de jugements commerciaux.
Il travaille en 5 phases : Cadrage, Decisions, Redaction, Robustesse, Redaction definitive.

Le juge interagit avec Claude dans Cowork. Les resultats s'affichent sur le dashboard web
(mongreffier.evidencai.com) ou le juge relit, annote et valide chaque phase.

## Composants

| Type | Fichier | Description |
|------|---------|-------------|
| Skill | skills/mon-greffier/SKILL.md | Workflow complet de redaction |
| Command | commands/install.md | Onboarding premiere connexion |
| MCP | .mcp.json | Serveur Supabase (CRUD + PISTE) |

## Prerequis

- Un compte Mon Greffier (cree lors du /install)
- Des credits (1 credit = 1 dossier complet)
- Acces internet (pour Supabase + PISTE)

## Outils MCP disponibles (17)

### Authentification
- mongreffier_login : connexion
- mongreffier_signup : creation de compte

### Credits
- mongreffier_get_credits : consulter le solde
- mongreffier_apply_promo : appliquer un code promo

### Dossiers (CRUD Supabase)
- mongreffier_create_dossier : nouveau dossier
- mongreffier_write_phase : ecrire une phase (JSON)
- mongreffier_read_responses : lire les reponses du juge
- mongreffier_update_dossier : mettre a jour un dossier
- mongreffier_read_dossier : lire un dossier complet

### PISTE (Legifrance + Judilibre)
- rechercher_code : article dans un code
- rechercher_dans_texte_legal : loi ou decret
- rechercher_jurisprudence_judiciaire : jurisprudences
- recherche_journal_officiel : Journal Officiel
- lister_codes_juridiques : liste des codes
- consulter_article : texte complet d'un article
- consulter_decision : texte complet d'une decision

## Configuration

Le plugin se connecte automatiquement au serveur MCP Supabase via le .mcp.json.
Aucune variable d'environnement a configurer cote utilisateur.

## Support

Contact : contact@evidencai.com
Site : https://www.evidencai.com
Dashboard : https://mongreffier.evidencai.com
