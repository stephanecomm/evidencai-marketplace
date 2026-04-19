# Phase 0 — Extraction des conclusions

## Contexte

Cette phase se fait dans un fil Cowork dédié. Son seul but : lire les PDF fournis par le juge, en extraire les conclusions en markdown propre, et créer le dossier dans Mon Greffier avec ces conclusions déjà stockées. Le cadrage se fera dans un fil séparé pour économiser le quota (éco de contexte).

## Prérequis

Le juge doit être authentifié (mongreffier_login exécuté). Si pas encore fait, demander email + mot de passe.

## Workflow

### 1. Recevoir les conclusions

Demander au juge de fournir les conclusions : fichiers PDF glissés dans la conversation OU texte collé.

Lire les PDF joints (Claude lit nativement les PDF texte et les scans via Vision).

### 2. Extraire le texte en markdown

Produire un markdown propre des conclusions :
- Un en-tête clair par partie (demandeur, défendeur)
- Conserver la numérotation des pièces
- Conserver les visas et références d'articles tels quels
- Retirer les numéros de page, répétitions d'en-tête/pied de page, mentions de cabinet

### 3. Identifier les métadonnées disponibles dans les conclusions

Repérer sans inventer :
- **Titre du dossier** : typiquement "X c/ Y" (à défaut, demander au juge)
- **Numéro RG** : souvent dans l'en-tête des conclusions (à défaut, demander au juge)
- **Juridiction** : "Tribunal de Commerce de ..."
- **Date d'audience** : si mentionnée (plaidoirie)

Ne **rien inventer** pour président, assesseurs, greffier, date de mise à disposition, note de délibéré : ces informations ne figurent jamais dans les conclusions. Le juge les saisira lui-même dans le dashboard.

### 4. Créer le dossier en un seul appel

Appeler `mongreffier_create_dossier` avec les MD et les métadonnées extraites :

```
mongreffier_create_dossier(
  user_id: <user_id>,
  titre: "X c/ Y",
  numero_rg: "2024/...",
  conclusions_md: "<markdown complet>",
  date_audience: "2024-03-15"  # si trouvée, sinon omettre
)
```

Si `conclusions_md` est fourni, le dossier est créé directement en phase `extraction_done`. Un crédit est décompté.

### 5. Message au juge

Afficher :

"Le dossier **<titre>** a été créé (RG <numero_rg>). Les conclusions sont stockées.

**Avant de lancer le cadrage**, ouvrez le dashboard Mon Greffier (https://mongreffier.evidencai.com) et cliquez sur **Formation** :
- sur la carte du dossier dans la liste, ou
- sur le badge **Formation** à côté du titre une fois le dossier ouvert

Renseignez :
- Président, assesseurs, greffier
- Date de mise à disposition (date du délibéré)
- **Note de délibéré** : ce que la formation a décidé collégialement à l'audience

La note de délibéré est utilisée dès le cadrage pour orienter l'analyse. Sans elle, le LLM analyse à partir des seules conclusions.

Une fois la formation remplie, ouvrez un **nouveau fil Cowork** et collez cette commande :"

**Lance le cadrage du dossier `<dossier_id>`.**

Expliquer : "Ouvrir un nouveau fil permet de réduire la consommation de votre quota d'abonnement (éco de contexte). Chaque phase fonctionne de manière indépendante."
