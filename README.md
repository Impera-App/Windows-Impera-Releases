# Impera Releases — Channel auto-update

Repo dédié à l'**hébergement des binaires de release** de l'app desktop Impera ([Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera)).

Pas de code source ici. Ce repo sert uniquement de point de publication pour [`electron-updater`](https://www.electron.build/auto-update), qui télécharge les MAJ directement depuis les *GitHub Releases* de ce dépôt.

## Pourquoi un repo séparé ?

L'app desktop est dans [Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera) (source privée). Les binaires de release sont publiés ici (repo public) pour 3 raisons :

1. **Téléchargement public** — `electron-updater` doit pouvoir GET les artefacts sans token. Les releases publiques satisfont ça sans exposer le code source.
2. **Bande passante** — GitHub Releases sert les binaires gratuitement et avec un CDN décent. Pas besoin de payer un S3.
3. **Audit & transparence** — les hash des binaires sont publics, n'importe qui peut vérifier qu'il télécharge bien la version annoncée.

## Format des releases

Chaque release suit la convention `electron-updater` :

```
Impera-Setup-1.2.3.exe         ← Installer NSIS signé
Impera-Setup-1.2.3.exe.blockmap ← Delta block-map (MAJ incrémentale)
latest.yml                      ← Métadonnée (version, sha512, taille)
```

Tag : `v<version>` (ex : `v1.2.3`)

Title : `Impera v1.2.3 — <ligne tagline>`

Description : changelog Markdown (features / fixes / breaking) — repris depuis le commit message ou le CHANGELOG du repo source.

## Workflow de release

Tout est piloté depuis [Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera) :

```bash
# Dans Windows-App-Impera
npm version patch    # ou minor/major — bump package.json + tag git
npm run dist         # build + electron-builder + publish vers Impera-Releases
```

Le `publish` est configuré dans `package.json#build.publish` :

```json
{
  "provider": "github",
  "owner": "Impera-App",
  "repo": "Windows-Impera-Releases",
  "releaseType": "release"
}
```

Avec un `GH_TOKEN` valide, `electron-builder` :
1. Crée une nouvelle GitHub Release ici
2. Upload les artefacts (installer + blockmap + latest.yml)
3. Marque la release comme "latest"

L'app desktop installée chez les users vérifie `latest.yml` toutes les heures, télécharge la MAJ en background si une nouvelle version existe, et propose à l'user de relancer pour appliquer.

## Promotion vs draft

Pour publier une beta :
- Tag : `v1.2.3-beta.1`
- Marquer la release comme **Pre-release** sur GitHub

Pour publier une release stable :
- Tag : `v1.2.3`
- Décocher "Pre-release"

`electron-updater` peut être configuré pour suivre un canal (`stable` vs `beta`) — voir `app.json#updater.channel` côté app desktop.

## Vérification d'intégrité

Chaque release est signée :
- **Installer NSIS** : signé avec le certificat EV Code Signing d'Impera (vérifiable via `signtool verify`)
- **`latest.yml`** : contient le `sha512` de l'installer ; `electron-updater` rejette tout binaire qui ne match pas

## Repos liés

| Repo | Rôle |
|------|------|
| [Windows-App-Impera](https://github.com/Impera-App/Windows-App-Impera) | Code source de l'app desktop publiée ici |
| [Windows-App-ImperaGuard](https://github.com/Impera-App/Windows-App-ImperaGuard) | Service strict mode (embarqué dans l'installer) |
| [Website-Impera](https://github.com/Impera-App/Website-Impera) | Page de téléchargement ; pointe vers la latest release de ce repo |
| [Mobile-App-Impera](https://github.com/Impera-App/Mobile-App-Impera) | App mobile Android + iOS (Expo / React Native) (channel séparé, géré par EAS) |
