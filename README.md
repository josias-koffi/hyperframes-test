# Vidéos HyperFrames

Un seul repo pour toutes les vidéos réalisées avec [HyperFrames](https://github.com/heygen-com/hyperframes)
et Claude Code : **un dossier par vidéo** dans `videos/`, une seule configuration à la racine.

Une vidéo HyperFrames est une page HTML animée avec GSAP, que la CLI `hyperframes` rend en MP4.
La voix off et la musique sont générées par IA via OpenRouter.

## Installation

Prérequis : Node.js et `ffmpeg` (pour le rendu et la normalisation audio).

```bash
npm install
cp .env.example .env    # puis renseigner OPENROUTER_API_KEY
```

Aucun compte HeyGen n'est nécessaire : la création, la prévisualisation et le rendu
fonctionnent en local.

## Structure

```
.
├── .env.example          # modèle de configuration (versionné)
├── .env                  # clé OpenRouter + modèles (local, ignoré par git)
├── package.json          # version de HyperFrames figée + scripts
├── CLAUDE.md / AGENTS.md # règles pour les agents IA (Claude Code, Codex)
├── scripts/
│   └── new-video.sh      # création d'une vidéo (npm run new)
└── videos/
    └── <nom>/            # une vidéo = un projet HyperFrames autonome
        ├── index.html        # composition principale (timeline racine)
        ├── compositions/     # scènes (sous-compositions)
        ├── assets/           # voix off, musique, effets sonores, images
        ├── meta.json         # identifiant et nom de la vidéo
        ├── hyperframes.json  # configuration HyperFrames du projet
        └── renders/          # MP4 rendus (ignoré par git)
```

Les vidéos n'ont ni dépôt git, ni `package.json`, ni `.env` propres : tout est partagé à la
racine. Chaque commande HyperFrames prend le dossier de la vidéo en argument.

## Commandes

| Commande | Rôle |
|---|---|
| `npm run new -- <nom>` | Crée `videos/<nom>/` (format 1920×1080 par défaut) |
| `npm run new -- <nom> --resolution=portrait` | Format vertical 1080×1920 (`square` pour 1080×1080) |
| `npm run preview -- videos/<nom>` | Ouvre le Studio sur la vidéo (serveur en arrière-plan) |
| `npm run dev -- videos/<nom>` | Studio au premier plan (bloque le terminal jusqu'à Ctrl+C) |
| `npm run preview:stop` | Arrête tous les Studios lancés |
| `npm run check -- videos/<nom>` | Lint, mise en page, animation et contraste |
| `npm run render -- videos/<nom>` | Rend le MP4 dans `videos/<nom>/renders/` |
| `npm run publish -- videos/<nom>` | Publie la vidéo et donne un lien de partage |

## Créer une nouvelle vidéo

Avec Claude Code, ouvert à la racine du repo :

> Nouvelle vidéo sur [sujet], 30 s, format vertical, voix off en français.

Claude crée le dossier, écrit la composition, génère la voix off et la musique, vérifie avec
`check` puis lance le rendu.

À la main :

```bash
npm run new -- ma-video
npm run preview -- videos/ma-video
# … éditer videos/ma-video/index.html …
npm run check -- videos/ma-video
npm run render -- videos/ma-video
```

Une vidéo toute neuve échoue au `check` avec « Timeline did not advance under seek » : le
modèle vierge ne contient aucune animation. L'erreur disparaît dès qu'une animation est ajoutée.

## Voix off et musique

Les deux passent par **OpenRouter** avec une seule clé. Les modèles se règlent dans `.env` :

| Variable | Valeur par défaut | Rôle |
|---|---|---|
| `OPENROUTER_API_KEY` | — | Clé OpenRouter |
| `OPENROUTER_BASE_URL` | `https://openrouter.ai/api/v1` | Point d'accès de l'API |
| `TTS_MODEL` | `mistralai/voxtral-mini-tts-2603` | Voix off (Mistral Voxtral Mini TTS) |
| `TTS_VOICE` | `fr_marie_neutral` | Voix utilisée |
| `TTS_PEAK_DB` | `-1` | Niveau de crête visé après normalisation |
| `MUSIC_MODEL` | `google/lyria-3-clip-preview` | Musique de fond (Google Lyria 3 Clip) |

La sortie brute de Voxtral est faible (crête vers -12,5 dB). Elle est donc normalisée avant
d'être placée dans la vidéo, et la prise d'origine est gardée dans `assets/audio/voxtral/`.
Les réponses brutes des fournisseurs vont dans `.media/` (ignoré par git).

Ce choix remplace l'ordre par défaut de HyperFrames (HeyGen → ElevenLabs → Kokoro pour la voix,
HeyGen → Lyria → MusicGen pour la musique). Il est écrit dans `CLAUDE.md`, que les agents lisent
à chaque session.

## Studio

Le Studio local tourne dans le navigateur, sur une vidéo à la fois
(`http://localhost:3002/#project/<nom>`). Il sert à prévisualiser, parcourir la timeline et
faire des retouches. Pour ouvrir deux vidéos en même temps, lancer la seconde sur un autre port :

```bash
npm run preview -- videos/autre-video --port=3003
```

L'application de bureau HyperFrames Studio (avec Framey) demande un compte HeyGen et n'est pas
nécessaire ici : la création se fait avec Claude Code dans le terminal, le Studio web sert à
visualiser.

## Mettre à jour HyperFrames

La version de la CLI est figée dans `package.json` pour que les vidéos se rendent toujours
de la même façon. Pour passer à une version plus récente :

```bash
npx hyperframes@latest upgrade --project . --check   # affiche les changements
npx hyperframes@latest upgrade --project .           # met à jour les versions figées
```

Penser ensuite à mettre à jour la version dans `scripts/new-video.sh`.

## Vidéos

| Dossier | Contenu |
|---|---|
| `videos/claude-code-intro/` | « Et si Claude Code réalisait vos vidéos ? » — 16 s, 1080p, 60 fps |
