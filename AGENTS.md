# Notes pour les agents

Les deux premières sections sont **communes à tous les dépôts** et identiques
partout : les modifier, c'est les modifier dans chaque dépôt. Ce qui est propre à
ce dépôt commence à « Ce dépôt ».

## Organisation commune

**Chaque dépôt a la même forme.** On doit pouvoir passer de l'un à l'autre sans
chercher où sont les choses.

| Fichier | Rôle | Identique partout ? |
|---|---|---|
| `README.md` | Présentation, toujours au même plan (ci-dessous) | Le plan, oui ; le contenu, non |
| `SECURITY.md` | Signalement d'une faille, règles communes, points propres | Oui, sauf « Propre à ce dépôt » |
| `AGENTS.md` | Ce fichier : règles pour qui modifie le code | Oui, jusqu'à « Ce dépôt » |
| `CLAUDE.md` | Une seule ligne, `@AGENTS.md` | Oui |
| `.editorconfig`, `.gitattributes` | Mise en forme et fins de ligne | Oui |
| `.gitignore` | Propre au dépôt, puis le bloc « Secrets et clés » commun en fin de fichier | Le bloc, oui |
| `.github/dependabot.yml` | Actions mensuelles, npm en correctifs de sécurité seuls | Oui, aux écosystèmes près |
| `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/` | Gabarits de PR et d'issues | Oui |
| `.github/workflows/` | CI et publication, au même gabarit (voir Sécurité) | La structure, oui |
| `docs/` | Toute documentation longue : conception, migrations, audits, procédures | — |

À la racine il n'y a **que** ces fichiers, le code et la configuration des
outils. Un document de conception, une migration, un audit vont dans `docs/` —
une racine qui en accumule n'est plus lue.

**Le README suit toujours ce plan, dans cet ordre**, et aucune section n'est
omise : une section sans objet le dit (« Sans objet : … »), parce qu'une section
absente se lit comme un oubli.

1. `# Nom — rôle`, une phrase, puis le tableau Famille · Plateforme · Publié sur · État ;
2. `## En bref` — ce que fait le produit, pour qui ;
3. `## Fonctionnement` — ce qui est propre au produit ; le long va dans `docs/` ;
4. `## Démarrer` — prérequis, installation, lancement en local ;
5. `## Vérifier` — les commandes à lancer avant de livrer ;
6. `## Publier` — déploiement ou publication ;
7. `## Configuration` — tableau Nom · Où le poser · Public ? · Rôle ;
8. `## Organisation du dépôt` — l'arbre, commenté ;
9. `## Sécurité` — renvoi à `SECURITY.md` et points propres ;
10. `## Dépôts liés` — le tableau commun, identique partout ;
11. `## Licence`.

**Langue** : les **identifiants sont en anglais** — fonctions, variables,
fichiers, composants, classes CSS, clés i18n, objets SQL, adresses techniques —
et le **texte visible est en français**, posé par-dessus (i18n). Les noms de
pages des sites restent en français (`confidentialite.html`). Commentaires,
documentation et messages de commit sont en français ; un commentaire dit
**pourquoi**, pas ce que fait la ligne. Le code existant écrit en français se
convertit au fil des modifications, jamais d'un bloc : un renommage massif
casse plus qu'il ne range.

**Nommage : tout ce qui appartient à un projet porte son nom**, toujours de la
même façon.

| Quoi | Règle | Exemple |
|---|---|---|
| Adresse | `<projet>.lorderis.dev` | `foody.lorderis.dev`, `account.lorderis.dev` |
| Schéma Supabase | `<projet>` | `impera`, `healthchecks` |
| Edge Function | `<projet>-<action>` | `impera-stripe-webhook` |
| Secret | `<PROJET>_…` | `IMPERA_STRIPE_SECRET_KEY` |
| Dossier distant | `website-<projet>/` | `website-foody/` |

**Un seul compte : le compte Lorderis.** Toute app qui a des comptes utilise
celui du projet Supabase commun, dont le dépôt de référence est
[Account-LordEris](https://github.com/LordEris/Account-LordEris) : même
e-mail, même mot de passe, même compte Google partout, gérés sur
`account.lorderis.dev`. Une app ne crée jamais son propre système de comptes.
Les textes de connexion, d'inscription et de suppression, et la ligne « Un
projet Lorderis » en bas de page sont communs, au mot près.

**Livrer** : une branche, une pull request, jamais de poussée directe sur la
branche par défaut. Avant d'ajouter à l'index, relire `git status` : un
`git add -A` ramasse aussi ce qui traîne.

## Sécurité commune

- **Aucun secret dans le dépôt.** Les clés publiques (clé `anon` /
  `publishable` de Supabase, identifiants Appwrite, ID clients OAuth) n'en sont
  pas ; tout le reste vit dans les secrets d'Actions ou des Edge Functions. Un
  secret committé une fois se **change** : le retirer ne le retire pas de
  l'historique.
- **Workflows** : `permissions: contents: read` par défaut, une tâche qui écrit
  le dit en tête de fichier ; actions épinglées par **SHA** avec la version en
  commentaire ; toute valeur saisie ou tout secret passe par `env:`, jamais par
  `${{ }}` dans le corps d'un script ; `persist-credentials: false` quand rien
  ne pousse.
- **Déploiement SFTP** : un seul gabarit, avec contrôle préalable des secrets,
  clé d'hôte vérifiée (`FTP_KNOWN_HOSTS`, jamais `sftp:auto-confirm yes`),
  envoi sérialisé (`concurrency`) et vérification du fichier publié. Les
  exclusions du miroir sont doublées par un refus côté serveur (`.htaccess`) :
  un fichier interne envoyé par erreur ne doit pas être servi.
- **Sites** : même socle d'en-têtes partout — HSTS, CSP sans script en ligne,
  `frame-ancestors 'none'`, `nosniff`, Referrer-Policy, Permissions-Policy,
  redirection HTTPS, fichiers cachés et internes refusés. Un écart se justifie
  en commentaire.
- **Données de santé** : elles ne quittent jamais l'appareil, chiffrées dans
  un coffre (Keystore, Keychain, `safeStorage`). Les autres apps peuvent en
  envoyer vers HealthChecks ; HealthChecks n'envoie rien ailleurs, à
  personne — seul un rapport de plantage anonyme (type d'erreur et pile
  d'appels, sans aucune donnée) part chez Sentry, région UE — et n'appelle
  aucun service d'IA.
- **Projet Supabase commun** : une base pour toutes les apps, **un schéma par
  app**. RLS sur chaque table ; toute fonction `SECURITY DEFINER` fixe son
  `search_path`, vérifie `auth.uid()` et retire `execute` à `public`/`anon`
  quand elle n'est pas publique. La clé `service_role` ne quitte jamais une
  Edge Function. Le SQL du projet commun vit dans Account-LordEris ; une
  copie ailleurs porte une garde qui refuse de s'exécuter sur le projet commun.
- **Applications** : sessions dans le stockage sécurisé du système (Keychain,
  Keystore, SecureStore), jamais en clair ; une entrée venant d'un fichier, d'un
  lien ou d'un autre processus est hostile ; un processus élevé ne lit rien
  qu'un processus non élevé puisse écrire sans le valider.
- **Dépendances** : Dependabot pour les alertes, `npm audit --omit=dev` avant
  une publication ; une montée majeure se fait dans sa propre PR, testée.
- **Un échec ne se tait pas, une donnée absente n'est pas favorable** : un
  contrôle qui ne peut pas juger le dit, il ne conclut pas « conforme ».

## Ce dépôt

Le canal public de mise à jour de l'app Windows d'Impera. Il ne contient que sa
documentation : les installeurs vivent dans ses *GitHub Releases*, une par
version, avec la carte de blocs et le `latest.yml` que lit `electron-updater`.
Tout ce qui y est publié est produit par le workflow **Release Windows** de
Windows-App-Impera ; ce dépôt n'a ni code, ni workflow, ni secret.

### Un dépôt public : rien d'interne

**Tout ce qui s'écrit ici est lu par n'importe qui**, README compris. Les noms
de secrets peuvent y figurer, jamais leurs valeurs ; aucune adresse d'outil
privé, aucun détail du dépôt source au-delà de ce qui sert à comprendre le
canal. Le dépôt est public parce que l'app installée lit le canal **sans
authentification** — un jeton compilé dans l'exécutable s'en extrairait — et
non par goût de la transparence : c'est une contrainte, pas une vitrine.

### Les Releases se font par le workflow, pas à la main

**Une Release se crée et se refait par le workflow du dépôt source.**
`electron-builder` y écrit ensemble l'installeur, sa carte de blocs et
`latest.yml`, dont le `sha512` doit correspondre au binaire : un installeur
remplacé à la main sans son `latest.yml` est rejeté par toutes les apps
installées, et la mise à jour s'arrête sans que l'utilisateur sache pourquoi.

**Les binaires ne vont jamais dans l'arbre git.** Ils sont attachés aux
Releases ; versionnés, ils alourdiraient chaque clone pour toujours.

Les conventions que l'app attend, et qu'il ne faut pas « corriger » ici :

- **le nom de l'installeur porte des points**, `Impera.Setup.1.2.3.exe` : il
  vient de `build.nsis.artifactName` dans le dépôt source, et ce README a déjà
  annoncé des tirets qu'aucune release n'a portés ;
- **l'étiquette est `v<version>`**, le titre `Impera v1.2.3 — <accroche>` ;
- **une pre-release n'est proposée à personne** : aucun canal `beta` n'est
  configuré dans l'app, elle se télécharge à la main. L'annoncer comme une
  bêta distribuée serait annoncer ce qu'on n'a pas.

### La signature n'est pas acquise

**Un installeur n'est signé que si un certificat est fourni à la
construction.** Un certificat EV vit le plus souvent sur un jeton matériel,
qu'un exécuteur GitHub ne peut pas présenter : une release produite par la CI
sans `CSC_LINK` part non signée. Le README le dit, et ne doit pas promettre
une signature qu'une release donnée ne porte pas.

## Vérifications avant de livrer

Sans objet pour le code : il n'y en a pas, et rien ne s'exécute dans ce dépôt.
Une modification de la documentation se relit comme un texte public. Le canal,
lui, se vérifie comme le lit l'app installée — sans authentification :

```bash
curl -sSfL https://github.com/Impera-App/Windows-Impera-Releases/releases/latest/download/latest.yml
```

Si le YAML s'affiche avec la version attendue, le canal fonctionne.
