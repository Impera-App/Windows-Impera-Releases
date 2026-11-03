# Windows-Impera-Releases — canal public de mise à jour de l'app Windows d'Impera

Le dépôt public dont les *GitHub Releases* hébergent les installeurs de l'app Windows d'Impera, et où l'app installée vient chercher ses mises à jour.

| Famille | Plateforme | Publié sur | État |
|---|---|---|---|
| Impera | Windows (installeur NSIS, `electron-updater`) | GitHub Releases de ce dépôt | Actif : lu par chaque app Windows installée |

## Sommaire

- [En bref](#en-bref)
- [Fonctionnement](#fonctionnement)
- [Démarrer](#démarrer)
- [Vérifier](#vérifier)
- [Publier](#publier)
- [Configuration](#configuration)
- [Organisation du dépôt](#organisation-du-dépôt)
- [Sécurité](#sécurité)
- [Dépôts liés](#dépôts-liés)
- [Licence](#licence)

## En bref

Dépôt dédié à l'**hébergement des binaires de release** de l'app de bureau
Impera ([Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera)).
Il sert à deux publics : l'app installée, qui y trouve ses mises à jour, et les
personnes qui téléchargent l'installeur depuis le site.

Pas de code source ici. Ce dépôt sert uniquement de point de publication pour
[`electron-updater`](https://www.electron.build/auto-update), qui télécharge les
mises à jour directement depuis les *GitHub Releases* de ce dépôt.

## Fonctionnement

### Pourquoi un dépôt séparé

L'app de bureau est dans
[Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera) (source
privée). Les binaires sont publiés ici (dépôt public) pour trois raisons :

1. **Téléchargement public** — `electron-updater` doit pouvoir lire les
   artefacts sans jeton. Les Releases publiques le permettent sans exposer le
   code source.
2. **Bande passante** — GitHub Releases sert les binaires gratuitement, derrière
   un CDN correct. Pas besoin de payer un stockage S3.
3. **Audit et transparence** — les empreintes des binaires sont publiques :
   n'importe qui peut vérifier qu'il télécharge bien la version annoncée.

### Format des releases

Chaque release suit la convention `electron-updater` :

```
Impera.Setup.1.2.3.exe          ← installeur NSIS
Impera.Setup.1.2.3.exe.blockmap ← carte de blocs (mise à jour incrémentale)
latest.yml                      ← métadonnées (version, sha512, taille)
```

Des **points**, pas des tirets : le nom vient de
`package.json#build.nsis.artifactName` dans le dépôt source, qui vaut
`${productName}.Setup.${version}.exe`. Ce README a un temps annoncé des
tirets, ce qu'aucune release n'a jamais porté.

- Étiquette : `v<version>` (ex. : `v1.2.3`)
- Titre : `Impera v1.2.3 — <ligne d'accroche>`
- Description : journal des modifications en Markdown (nouveautés /
  corrections / ruptures), repris du message de commit ou du CHANGELOG du dépôt
  source.

### Qui lit ce canal

- **L'app Windows installée** vérifie `latest.yml` toutes les heures,
  télécharge la mise à jour en arrière-plan si une nouvelle version existe, et
  propose de relancer pour l'appliquer.
- **Le site** ([Website-Impera](https://github.com/Impera-App/Website-Impera))
  porte la page de téléchargement, qui pointe vers la dernière release de ce
  dépôt.
- **L'app mobile** ([Mobile-App-Impera](https://github.com/Impera-App/Mobile-App-Impera))
  a son propre canal, géré par EAS : elle ne passe pas par ici.
- **ImperaGuard**, le service du mode strict (dossier `guard/` du dépôt
  source), est embarqué dans l'installeur : il n'a pas de release à lui.

### Version bêta et version stable

Pour publier une bêta :

- étiquette `v1.2.3-beta.1` ;
- release marquée **Pre-release** sur GitHub.

Pour publier une version stable :

- étiquette `v1.2.3` ;
- « Pre-release » décoché.

`electron-updater` sait suivre un canal (`stable` ou `beta`), mais **rien
n'est configuré en ce sens aujourd'hui** : l'app de bureau n'a ni `app.json`
ni réglage de canal. Une pre-release publiée ici n'est donc pas proposée
automatiquement — elle se télécharge à la main.

### Vérification d'intégrité

- **Installeur NSIS** : signé avec le certificat EV de signature de code
  d'Impera (vérifiable par `signtool verify`) **lorsqu'un certificat est fourni
  à la construction**. Un certificat EV vit le plus souvent sur un jeton
  matériel, qu'un exécuteur GitHub ne peut pas présenter : une release produite
  par la CI sans `CSC_LINK` n'est pas signée, et SmartScreen l'annonce comme
  éditeur inconnu.
- **`latest.yml`** : contient le `sha512` de l'installeur ; `electron-updater`
  rejette tout binaire qui ne correspond pas.

## Démarrer

Sans objet : ce dépôt ne contient aucun code, rien ne s'y installe ni ne s'y
lance. Le développement de l'app se fait dans
[Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera).

## Vérifier

Sans objet : aucun code, donc aucun test ni construction. Ce qui se vérifie,
c'est que le canal répond **sans authentification**, comme le lit l'app
installée — par exemple depuis une fenêtre de navigation privée, ou :

```bash
curl -sSfL https://github.com/Impera-App/Windows-Impera-Releases/releases/latest/download/latest.yml
```

Si le YAML s'affiche, avec la version attendue, le canal fonctionne.

## Publier

Tout est piloté depuis
[Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera) :

```bash
# Dans Windows-App-Impera
npm version patch          # ou minor/major — monte package.json et pose l'étiquette
git push --follow-tags     # l'étiquette v* déclenche .github/workflows/release.yml
```

Le workflow tourne sur un exécuteur `windows-latest` — obligatoire, la cible
NSIS et ImperaGuard (`net8.0-windows`) ne se construisent pas ailleurs — et
publie ici. Il se lance aussi à la main : Actions → **Release Windows** → *Run
workflow*, qui republie la version courante de `package.json`.

Un `npm run dist` en local reste possible sur une machine Windows équipée du
SDK .NET 8 ; c'était le seul chemin avant le workflow.

Le `publish` est configuré dans `package.json#build.publish` du dépôt source :

```json
{
  "provider": "github",
  "owner": "Impera-App",
  "repo": "Windows-Impera-Releases"
}
```

Avec un `GH_TOKEN` valide — côté CI le secret `RELEASES_TOKEN`, car le
`GITHUB_TOKEN` d'Actions n'a de droits que sur le dépôt source —
`electron-builder` :

1. crée une nouvelle GitHub Release ici ;
2. téléverse les artefacts (installeur, blockmap, `latest.yml`) ;
3. marque la release comme « latest ».

## Configuration

Rien ne se configure dans ce dépôt : tout vit dans le dépôt source.

| Nom | Où le poser | Public ? | Rôle |
|---|---|---|---|
| `build.publish` | `package.json` du dépôt source | Oui | Désigne ce dépôt comme destination des Releases |
| `build.nsis.artifactName` | `package.json` du dépôt source | Oui | Nom de l'installeur, `${productName}.Setup.${version}.exe` |
| `RELEASES_TOKEN` | Secrets GitHub Actions du dépôt source | Non | Jeton à portée fine qui écrit les Releases ici ; passé à `electron-builder` sous le nom `GH_TOKEN` |
| `CSC_LINK`, `CSC_KEY_PASSWORD` | Secrets GitHub Actions du dépôt source | Non | Certificat de signature de code et son mot de passe ; absents, l'installeur n'est pas signé |

## Organisation du dépôt

```
.
├── README.md, SECURITY.md, AGENTS.md, CLAUDE.md
├── .editorconfig, .gitattributes
└── .github/                       gabarits de PR et d'issues
```

Les binaires ne sont pas dans l'arbre git : ils sont attachés aux *GitHub
Releases*, une par version.

## Sécurité

Règles communes et signalement d'une faille : [SECURITY.md](SECURITY.md).
Propre à ce dépôt : il est **public** et ne porte aucun code ni secret ; il ne
reçoit que des binaires déjà construits par le dépôt source, et l'intégrité de
chaque mise à jour tient au `sha512` de `latest.yml` (voir
[Vérification d'intégrité](#vérification-dintégrité)).

## Dépôts liés

Le même tableau figure dans chaque dépôt. Un dépôt qui s'ajoute s'ajoute partout.

| Famille | Dépôt | Rôle |
|---|---|---|
| Foody | [Mobile-App-Foody](https://github.com/App-Foody/Mobile-App-Foody) | Application de suivi alimentaire (Expo), schéma Supabase `foody` |
| Foody | [Website-Foody](https://github.com/App-Foody/Website-Foody) | Site public, compte, réinitialisation, suppression de compte |
| Foody | [Website-FoodySupport](https://github.com/App-Foody/Website-FoodySupport) | Console des agents du support |
| Impera | [Mobile-App-Impera](https://github.com/Impera-App/Mobile-App-Impera) | Application de concentration, Android (Expo) |
| Impera | [Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera) | Application Windows (Electron) et service ImperaGuard |
| Impera | [MacOS-App-Impera](https://github.com/Impera-App/MacOS-App-Impera) | Application macOS (Swift) et filtre réseau |
| Impera | [Website-Impera](https://github.com/Impera-App/Website-Impera) | Site public, compte, paiement ; Edge Functions Stripe |
| Impera | [Website-ImperaSupport](https://github.com/Impera-App/Website-ImperaSupport) | Console des agents du support |
| Impera | [Windows-Impera-Releases](https://github.com/Impera-App/Windows-Impera-Releases) | Canal public de mise à jour de l'app Windows |
| BeeFluid | [Windows-Beefluid](https://github.com/BeeFluid/Windows-Beefluid) | Application terrain (Expo) et poste de bureau d'études (Tauri), schéma Supabase `beefluid` |
| BeeFluid | [Windows-Beefluid-releases](https://github.com/BeeFluid/Windows-Beefluid-releases) | Canal public de publication du poste et de l'APK |
| BeeFluid | [Mobile-Beefluid](https://github.com/BeeFluid/Mobile-Beefluid) | Vide : l'application mobile vit dans Windows-Beefluid |
| Lorderis | [Account-LordEris](https://github.com/LordEris/Account-LordEris) | Compte Lorderis unique : SQL du projet Supabase commun, account.lorderis.dev, code client de référence |
| Lorderis | [Patrimo](https://github.com/LordEris/capsa) | Gestion de patrimoine (ex-Capsa) : web, schéma Supabase `patrimo` |
| Lorderis | [HealthChecks](https://github.com/LordEris/HealthChecks) | Santé et suivi personnel (ex-Alfred), données chiffrées sur l'appareil : web, Electron, Android |
| Lorderis | [Lineup](https://github.com/LordEris/Lineup) | Playlists de groupe (Next.js), schéma Supabase `lineup` |
| Lorderis | [Fleetly](https://github.com/LordEris/Fleetly) | Gestion de garage et de flotte (Expo), schéma Supabase `fleetly` |
| Lorderis | [Website-Kikiladi](https://github.com/LordEris/Website-Kikiladi) | Jeu « Kikiladi ? » (statique + PHP) |
| Lorderis | [Website-LordEris](https://github.com/LordEris/Website-LordEris) | Site vitrine Lorderis et pages des projets |

## Licence

Aucune licence publiée : tous droits réservés.
