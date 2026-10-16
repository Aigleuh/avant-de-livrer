# avant-de-livrer

Un skill pour Claude Code qui prépare un projet avant de le livrer à un client : journaux d'erreurs, clés et mots de passe, alertes, plafonds de dépenses, document de passation.

Par Clive AI.

## Installation en une consigne

Ouvre Claude Code, puis colle :

```
Installe le skill avant-de-livrer. Clone https://github.com/Aigleuh/avant-de-livrer dans le dossier personnel des skills de Claude Code (le dossier .claude/skills de mon dossier utilisateur), de façon à obtenir le fichier .claude/skills/avant-de-livrer/SKILL.md. Vérifie que ce fichier existe bien, puis dis-moi comment lancer le skill.
```

Il faut que `git` soit installé.

## Installation à la main

Mac, Linux ou WSL :

```
git clone --depth 1 https://github.com/Aigleuh/avant-de-livrer.git ~/.claude/skills/avant-de-livrer
```

Windows (PowerShell) :

```
git clone --depth 1 https://github.com/Aigleuh/avant-de-livrer.git "$HOME\.claude\skills\avant-de-livrer"
```

Le fichier doit se trouver à `.claude/skills/avant-de-livrer/SKILL.md`.

## Utilisation

Dans Claude Code, à la racine de ton projet : `/avant-de-livrer`

Fais une sauvegarde de ton projet avant (commit git ou copie du dossier).

## Mise à jour

```
git -C ~/.claude/skills/avant-de-livrer pull
```

## Limites

Ce skill est une base solide, pas un audit de sécurité professionnel. Pour un projet qui manipule des paiements ou des données sensibles, fais-le relire par un spécialiste.
