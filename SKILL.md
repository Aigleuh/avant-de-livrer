---
name: avant-de-livrer
description: Prépare un projet logiciel avant de le livrer à un client ou de le mettre en production. Audite puis corrige 5 points : journaux d'erreurs, clés API et mots de passe, alertes, plafonds de dépenses sur les API payantes, document de passation (README). À utiliser quand l'utilisateur dit "avant de livrer", "mise en production", "check avant livraison", "sécuriser mon projet", "mon projet est prêt à livrer", "audit de mon code", ou demande de vérifier un projet avant de le donner à un client.
---

# Avant de livrer : les 5 réglages

Ce skill transforme un projet qui "marche sur mon ordinateur" en projet que l'on peut livrer sans mauvaise surprise. Il fait un audit, propose les corrections, puis les applique avec l'accord de l'utilisateur.

Parle en français simple. L'utilisateur n'est pas forcément développeur. Explique chaque changement en une phrase claire.

## Règles absolues

1. Ne jamais afficher la valeur d'une clé, d'un mot de passe ou d'un jeton dans la conversation. Indique seulement le fichier, la ligne et le type probable (par exemple "clé API probable, fichier src/api.js ligne 12").
2. Ne jamais écrire une vraie clé dans un fichier versionné. Le fichier `.env.example` contient uniquement des noms de variables avec des valeurs vides.
3. Ne jamais faire de commit, de push ni de déploiement sans que l'utilisateur le demande.
4. Ne jamais supprimer de fichier sans l'accord de l'utilisateur.
5. Quand une action doit être faite par l'utilisateur lui-même (régénérer une clé chez un fournisseur, régler une limite dans un tableau de bord), l'écrire clairement dans le rapport final. Claude ne peut pas la faire à sa place.
6. Si un point n'est pas applicable (pas d'API payante, pas de base de données), le dire et passer au suivant. Ne rien inventer.

## Étape 0 : cartographier le projet

Avant toute modification, lis la structure du projet et note :
- le langage et le framework utilisés,
- les endroits où le code appelle une API externe ou une base de données,
- les fichiers de configuration et les variables d'environnement déjà présentes,
- la présence d'un dépôt git, d'un `.gitignore`, d'un `README`,
- où le projet est hébergé ou doit l'être.

Résume cela en quelques lignes à l'utilisateur, puis annonce que tu vas traiter les 5 points dans l'ordre.

## Point 1 : les journaux d'erreurs

Objectif : quand un client signale un bug, on sait exactement ce qui s'est cassé et à quel moment.

À faire :
- Créer un petit module de journalisation unique (une seule fonction `logError` et une seule fonction `logInfo`), au lieu de disperser des `console.log`.
- Dans chaque fonction qui appelle une API externe ou la base de données, entourer l'appel d'un bloc de capture d'erreur qui journalise : la date et l'heure, le nom de la fonction, ce qu'elle essayait de faire, le message d'erreur, et un identifiant de contexte utile (identifiant de commande, de requête, etc.).
- Ne jamais journaliser de secret (clé, mot de passe, jeton), ni de donnée personnelle sensible (numéro de carte, pièce d'identité). Masquer ou omettre.
- Ne pas avaler l'erreur en silence : après journalisation, relancer l'erreur ou renvoyer une réponse d'erreur claire.
- Indiquer à l'utilisateur où les journaux sont écrits (console de l'hébergeur, fichier, service externe) et comment les consulter.

Vérification : provoquer volontairement une erreur (mauvaise URL, mauvais identifiant) et montrer la ligne de journal produite.

## Point 2 : les clés et mots de passe

Objectif : plus aucun secret dans le code.

À faire :
1. Chercher dans tout le projet les secrets en dur : motifs comme `api_key`, `apikey`, `secret`, `token`, `password`, `passwd`, `Bearer `, `sk-`, `pk_live`, `postgres://`, `mongodb+srv://`, et les longues chaînes aléatoires. Inclure les fichiers de configuration, les notebooks, les scripts et les anciens fichiers de test.
2. Déplacer chaque secret dans un fichier `.env` et le lire dans le code via les variables d'environnement.
3. Ajouter `.env` (et `.env.*` sauf `.env.example`) au `.gitignore`.
4. Créer un fichier `.env.example` qui liste chaque variable avec une valeur vide et une courte description.
5. Vérifier l'historique git : `git log --all --oneline -- .env` et, si `gitleaks` ou `trufflehog` est installé, lancer un scan de l'historique. Si un secret a déjà été commité, supprimer la ligne dans le code ne suffit pas, il reste visible dans l'historique.
6. Pour le code qui tourne dans le navigateur, rappeler que tout ce qui est envoyé au navigateur est public. Les clés secrètes doivent rester côté serveur.
7. En production, les variables se configurent dans l'hébergeur (tableau de bord des variables d'environnement), on ne téléverse pas le fichier `.env`.

Action obligatoire pour l'utilisateur : toute clé qui a été écrite dans le code, dans un commit, dans une capture d'écran ou dans une conversation doit être considérée comme compromise. Il doit la régénérer dans le tableau de bord du fournisseur, révoquer l'ancienne, puis mettre la nouvelle dans `.env`. Liste dans le rapport chaque fournisseur concerné.

Vérification : relancer la recherche de secrets et montrer qu'il ne reste aucun résultat. Lancer l'application pour confirmer qu'elle lit bien le `.env`.

## Point 3 : les alertes

Objectif : être prévenu dès qu'une erreur grave arrive, même à 2 h du matin.

Précision importante à expliquer à l'utilisateur : c'est le code qui envoie l'alerte, ce n'est pas Claude qui surveille le projet. Il faut donc écrire ce code.

À faire :
- Demander quel canal l'utilisateur préfère : e-mail (le plus simple), messagerie (Telegram, Slack) ou WhatsApp. WhatsApp passe par l'API WhatsApp Business, plus longue à mettre en place : proposer l'e-mail en premier.
- Créer une fonction `alertAdmin(niveau, message, contexte)` appelée par le module de journalisation pour les erreurs graves (paiement qui échoue, base de données injoignable, API qui refuse la clé, plafond de dépense atteint).
- Limiter l'envoi : ne pas envoyer plus d'une alerte identique par tranche de 15 minutes, sinon une boucle d'erreurs remplit la boîte mail.
- Ne jamais mettre de secret ni de donnée personnelle dans le texte de l'alerte.
- Ajouter une commande ou une route de test qui envoie une alerte factice.
- Rappeler la limite : si le serveur entier tombe, le code ne peut plus envoyer d'alerte. Recommander en plus un service externe de surveillance de disponibilité qui interroge le site à intervalle régulier.

Vérification : déclencher l'alerte de test et confirmer avec l'utilisateur qu'il l'a bien reçue.

## Point 4 : le plafond de dépenses

Objectif : une boucle cassée ne doit pas pouvoir vider le compte.

À faire dans le code, pour chaque API payante :
- Une limite de débit (nombre maximum d'appels par minute).
- Un plafond par jour (en nombre d'appels, en jetons ou en euros estimés), stocké de façon persistante pour survivre à un redémarrage. Quand le plafond est atteint : arrêt immédiat des appels, erreur claire, alerte au point 3.
- Un nombre maximum de nouvelles tentatives avec délai croissant, un délai d'attente maximum par appel, et une limite sur la taille des réponses demandées.
- Un garde-fou dans les boucles et les traitements en tâche de fond (nombre maximum d'itérations).
- Des plafonds configurables par variables d'environnement, avec une valeur par défaut prudente.

À faire par l'utilisateur chez les fournisseurs : régler aussi une limite de dépense dans le tableau de bord de chaque fournisseur payant. Pour l'API d'Anthropic, la console permet de fixer une limite mensuelle pour l'organisation et par espace de travail (sauf l'espace par défaut). Ces limites sont mensuelles : le plafond journalier doit donc rester dans le code. Pour les autres fournisseurs, l'utilisateur doit consulter les réglages de facturation et vérifier ce qui existe.

Précision à donner : les compteurs du code sont des estimations, pas une facture. Les limites du fournisseur restent le dernier filet de sécurité.

Vérification : simuler un dépassement (plafond mis à une valeur très basse) et montrer que les appels s'arrêtent et que l'alerte part.

## Point 5 : le document de passation

Objectif : n'importe qui peut reprendre le projet sans l'auteur.

Écrire ou mettre à jour `README.md` en français simple, sans jargon inutile, avec ces sections :
1. **Ce que fait le logiciel** : en 3 à 5 phrases, pour quelqu'un qui ne l'a jamais vu.
2. **Ce qu'il faut avoir avant de commencer** : outils à installer, comptes nécessaires.
3. **Comment le lancer** : commandes exactes, dans l'ordre, sur un ordinateur propre.
4. **Les variables d'environnement** : la liste des noms (jamais les valeurs), avec à quoi chacune sert et où obtenir la valeur.
5. **Comment le déployer**.
6. **Où regarder quand ça casse** : où sont les journaux, comment sont envoyées les alertes, les erreurs fréquentes et leur solution.
7. **Les services externes utilisés** : liste avec lien vers chaque tableau de bord et ce qui est facturé.
8. **Qui contacter** : à remplir par l'utilisateur.

Vérification : relire le README comme si on découvrait le projet, et tester les commandes de lancement quand c'est possible.

## Rapport final

Termine toujours par un rapport court, dans ce format :

| Point | Statut | Ce qui a été fait |
|---|---|---|
| 1. Journaux | fait / partiel / non applicable | ... |
| 2. Clés | ... | ... |
| 3. Alertes | ... | ... |
| 4. Plafond | ... | ... |
| 5. Passation | ... | ... |

Puis une section **À faire par toi** avec la liste précise des actions manuelles (clés à régénérer chez quel fournisseur, limites à régler, canal d'alerte à confirmer, informations à ajouter au README), et une section **Risques restants** si quelque chose n'a pas pu être vérifié.
