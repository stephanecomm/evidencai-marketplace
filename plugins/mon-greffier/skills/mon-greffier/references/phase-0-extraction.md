# Phase 0 — Extraction des conclusions

## Contexte

Cette phase se fait dans un fil Cowork dédié. Son seul but : extraire le contenu des PDF fournis par le juge et le stocker en base. Le cadrage se fera dans un fil séparé pour économiser le quota.

## Prérequis

Le juge doit être authentifié (mongreffier_login exécuté). Si pas encore fait, demander email + mot de passe.

## Workflow

### 1. Créer ou identifier le dossier

**Nouveau dossier :**
1. Demander le titre (ex: "BP ARA c/ LICINI") et le numéro RG (optionnel)
2. Appeler `mongreffier_create_dossier(user_id, titre, numero_rg)`
3. Dire au juge : "Ouvrez le dashboard Mon Greffier : https://mongreffier.evidencai.com"

**Dossier existant :**
1. Appeler `mongreffier_read_dossier(dossier_id)`

### 2. Recevoir les conclusions

Demander au juge de fournir les conclusions : fichiers PDF glissés dans la conversation OU texte collé.
Lire les PDF joints (Claude lit nativement les PDF texte et les scans via Vision).

### 3. Extraire et stocker

Extraire le texte intégral des conclusions en markdown propre.
Stocker via `mongreffier_update_dossier(dossier_id, conclusions_md: "...")`.

### 4. Mettre à jour la phase

Appeler `mongreffier_update_dossier(dossier_id, current_phase: "extraction_done")`.

### 5. Message au juge

Afficher :

"Les conclusions ont été extraites et stockées. Pour lancer le cadrage, ouvrez un nouveau fil Cowork et collez cette commande :"

**Extraction terminée. Lance le cadrage du dossier [ID_DU_DOSSIER].**

Expliquer au juge : "Utiliser un nouveau fil permet de réduire la consommation de votre quota d'abonnement. Chaque phase fonctionne de manière indépendante."
