# Sécurité — Impera (canal de publication Windows)

Ce fichier est commun à tous les dépôts (Foody, Impera, Patrimo,
HealthChecks, Lineup, Fleetly, BeeFluid, Kikiladi et le site Lorderis) et tenu
à l'identique ; seule la section « Propre à ce dépôt » change.

## Signaler une faille

Ne pas ouvrir d'issue publique. Passer par **Security → Report a
vulnerability** sur ce dépôt (signalement privé GitHub). Le signalement doit
dire ce qui est atteint, comment le reproduire, et ce qu'un attaquant y gagne.

## Les règles communes

- **Aucun secret dans le dépôt.** Les clés publiques (clé `anon` /
  `publishable` de Supabase, identifiants de projet Appwrite, ID clients OAuth)
  ne sont pas des secrets : elles voyagent dans chaque requête du navigateur.
  Tout le reste — clé `service_role`, mots de passe SFTP, clés de signature,
  jetons de publication — vit dans les secrets GitHub Actions ou dans ceux des
  Edge Functions. Le bloc « Secrets et clés » du `.gitignore` est le même
  partout. Un secret committé une fois se **change** : le retirer du dépôt ne
  le retire pas de l'historique.
- **Les workflows lisent, sauf exception écrite.** `permissions: contents:
  read` par défaut ; une tâche qui pousse un commit ou une étiquette le dit en
  tête de fichier. Les actions sont épinglées par SHA, avec la version en
  commentaire, et Dependabot les tient à jour.
- **Une valeur saisie ou un secret passe par `env:`**, jamais par `${{ }}` dans
  le corps d'un script : l'interpolation y colle le texte tel quel.
- **SFTP : la clé d'hôte est vérifiée** (`FTP_KNOWN_HOSTS`), jamais acceptée
  d'office. Sans elle, le déploiement échoue au lieu d'envoyer les
  identifiants à un serveur non vérifié.
- **Les droits se tiennent côté serveur.** RLS et fonctions `SECURITY
  DEFINER` (avec `search_path` fixé) pour Supabase, règles de document pour
  Appwrite. Ce que l'interface cache n'est pas protégé pour autant. Le SQL du
  projet Supabase commun vit dans Account-LordEris ; une copie ailleurs porte
  une garde qui refuse de s'exécuter sur le projet commun.
- **Un échec ne se tait pas, une donnée absente n'est pas favorable** : ce
  sont les règles de chaque dépôt, et elles valent pour la sécurité aussi.

## Propre à ce dépôt

- Dépôt **public**, sans code ni secret : il ne reçoit que des binaires déjà
  construits par Windows-App-Impera. Rien d'interne ne s'y écrit — ni valeur
  de jeton, ni adresse d'outil privé, ni détail du dépôt source au-delà de ce
  que le README en dit.
- Les Releases sont écrites par un seul jeton à portée fine limitée à ce dépôt
  (`RELEASES_TOKEN`), tenu dans les secrets du dépôt source. Aucun jeton n'est
  compilé dans l'app : `electron-updater` lit ce canal sans authentification,
  et c'est pour cela qu'il est public.
- L'intégrité d'une mise à jour tient au `sha512` de `latest.yml`, que
  `electron-updater` compare au binaire téléchargé. Un installeur remplacé à la
  main sans son `latest.yml` est refusé par les apps installées ; une Release
  se refait par le workflow du dépôt source, pas à la main.
- L'installeur n'est signé que si un certificat est fourni à la construction
  (`CSC_LINK`) : sans lui, SmartScreen l'annonce comme éditeur inconnu.
