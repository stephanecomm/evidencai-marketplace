# Phase 3 — Rédaction

## Prérequis

1. Identité du juge confirmée via OAuth (Cowork a déjà géré l'auth au premier appel MCP). Au besoin : `mongreffier_whoami`.
2. **Lire les préférences du juge** : `mongreffier_get_profile()` (l'identité est déduite du token) → récupérer `preferences.juridiction`, `preferences.ville`, `preferences.formule_pcm_override`, `preferences.vocabulaire_local`, `preferences.signature`
3. Appeler `mongreffier_read_dossier(dossier_id, for_phase: "redaction")` — charge les phases pertinentes **et les métadonnées du dossier** : `president`, `assesseurs`, `greffier`, `date_audience`, `date_delibere`, `note_delibere`
4. Lire les réponses du juge : `mongreffier_read_responses(dossier_id, phase_type: "decisions")`
5. Les choix du juge sur chaque point de décision déterminent la rédaction
6. Si relance_context existe → relance, lire la note du juge et l'historique
7. Si `note_delibere` est présente, l'intégrer comme consigne de haute priorité pour la rédaction (elle reflète la décision collégiale prise en délibéré)

## Règles de rédaction

- Chaque montant accordé DOIT être motivé individuellement
- Chaque demande rejetée DOIT être motivée
- Formulation impersonnelle du tribunal ("Le Tribunal", pas "nous")
- Visa des textes applicables en tête de chaque point
- Le dispositif reprend EXACTEMENT les décisions prises, sans ajout ni omission
- Exécution provisoire de droit (décret 2019-1333, art. 514 CPC, 1er janvier 2020) : NE RIEN ÉCRIRE pour la motiver. Elle est de droit, zéro motivation systématique.

## Moyens des parties

Les champs `moyens_des_parties.demandeur` et `moyens_des_parties.defendeur` contiennent les prétentions des parties telles qu'elles figurent au **PAR CES MOTIFS** de leurs dernières conclusions, reprises **mot à mot en liste à puces Markdown**.

**Règles strictes** :

- Aucune reformulation, aucune synthèse, aucun ajout de contenu.
- Chaque chef de prétention sur une ligne distincte, précédée d'un tiret Markdown (`- `).
- Respecter la casse des verbes (JUGER, CONDAMNER, DÉBOUTER, ORDONNER, DIRE, etc.) telle qu'elle figure aux conclusions.
- Conserver les montants, les formules « à titre principal / subsidiaire », les visas et les renvois aux articles, tels qu'écrits par l'avocat.
- Conserver les sous-titres éventuels (ex : « À titre principal : », « En toutes hypothèses : ») sous forme de puces mères, les chefs subordonnés indentés en puces filles.
- Si une partie n'a pas déposé de conclusions, indiquer : `- Pas de conclusions déposées.`

**Justification** : le jugement doit refléter fidèlement ce que les parties ont demandé. C'est la base du contrôle de l'ultra petita et de l'infra petita : toute reformulation à ce stade brouille la traçabilité de la décision et expose au risque d'appel pour dénaturation des prétentions.

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

## En-tête du jugement

Le champ `jugement.en_tete` doit être construit à partir des métadonnées du dossier et des préférences du juge. Structure type :

```
{{juridiction}}
{{ville}}

JUGEMENT DU {{date_delibere formatée : "JOUR MOIS ANNÉE"}}

RG : {{numero_rg}}

COMPOSITION DU TRIBUNAL :
- Président : {{president}}
- Assesseurs : {{assesseurs joints par ", "}}
- Greffier : {{greffier}}

DÉBATS : à l'audience publique du {{date_audience formatée}}
DÉLIBÉRÉ : mis en délibéré au {{date_delibere formatée}}

ENTRE :
{{demandeur, extrait des conclusions}}
ET :
{{défendeur, extrait des conclusions}}
```

**Règles** :
- Toute métadonnée absente : **omettre la ligne**, ne rien inventer.
- Si `president`, `assesseurs` ou `greffier` sont vides, ne pas écrire la section "COMPOSITION" incomplète : écrire uniquement les lignes présentes, ou signaler au juge dans un commentaire de robustesse si tout manque.
- Formater les dates en toutes lettres (ex : "13 avril 2026").

## Format du dispositif

### Formule PCM introductive (obligatoire)

**Ordre de priorité** :

1. **Si `preferences.formule_pcm_override` est défini** : l'utiliser tel quel, en interpolant les placeholders `{{ville}}`, `{{type_jugement}}`, `{{ressort}}`.
2. **Sinon, utiliser cette formule par défaut** (article 450 CPC) :

```
{{juridiction}}, après en avoir délibéré conformément à la loi, statuant par jugement {{type_jugement}} et en {{ressort}} ressort, prononcé publiquement par mise à disposition au greffe, les parties ayant été préalablement avisées dans les conditions prévues au deuxième alinéa de l'article 450 du Code de procédure civile ;
```

**Interpolation des placeholders** :

- `{{juridiction}}` : `preferences.juridiction` si défini, sinon demander au juge (ex : "Tribunal de Commerce de Romans-sur-Isère").
- `{{ville}}` : `preferences.ville` si défini.
- `{{type_jugement}}` : déduire des conclusions des avocats et des présences à l'audience :
  - `contradictoire` si les deux parties ont conclu
  - `réputé contradictoire` si le défendeur a été assigné à personne mais n'a pas conclu
  - `par défaut` si le défendeur n'a pas été assigné à personne et n'a pas conclu
- `{{ressort}}` : déduire du montant total du litige :
  - `premier` ressort si montant > 5 000 € (appel possible)
  - `dernier` ressort si montant ≤ 5 000 € (jugement non susceptible d'appel)

**Ne jamais** écrire la formule classique simplifiée "Le Tribunal, statuant publiquement, contradictoirement et en premier ressort..." : elle est juridiquement incomplète, ne vise pas l'article 450 CPC, et ne mentionne pas la mise à disposition au greffe.

### Règles de forme du dispositif

- Vu les articles en *italique* (markdown : `*Vu*`)
- Verbes en MAJUSCULES suivis d'un espace (CONDAMNE, REJETTE, DIT, ORDONNE, DÉBOUTE, DÉCLARE)
- Montants en lettres et en chiffres (ex : "la somme de dix mille euros (10 000 €)")
- Point-virgule entre chaque chef de dispositif
- Point final uniquement sur le dernier chef
- Formule finale sur les dépens et l'article 700 du CPC
- Exécution provisoire : **SILENCE** (de droit depuis 2020). Mentionner uniquement si demande d'écartement.

### Vocabulaire local

Si `preferences.vocabulaire_local` est défini, l'appliquer systématiquement (ex : "utiliser 'Le Tribunal' plutôt que 'Nous'", "toujours viser 1231-5 avant 1343-5").

### Signature finale

Si `preferences.signature` est défini, l'ajouter en fin de dispositif (ex : "Le Président, [Nom Prénom]").

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
