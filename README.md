# Documentaire en motion design codé — skill pour Claude

Un *skill* (Claude Code / Claude Agent SDK) qui fabrique des vidéos documentaires et explicatives où **chaque image est dessinée par du code** (React + SVG avec [Remotion](https://www.remotion.dev/)), puis rendues en MP4. Pas d'images générées par IA : des formes, des personnages, des graphiques et du texte animés, synchronisés sur la voix off.

*English: a Claude skill that produces explainer/documentary videos in coded motion design (Remotion, React/SVG), rendered to MP4 with voice-over, subtitles, selectable visual styles and a YouTube thumbnail. The skill content is in French; contributions in English or French are welcome.*

## Ce que le skill produit

- Le MP4 1080p avec voix off et sous-titres
- Le texte de la voix off, scène par scène
- La miniature YouTube (1280×720)
- Le titre et la description YouTube avec chapitres et sources

## Styles

| Style | Rendu | Police |
|---|---|---|
| `papier` (défaut) | collage de papier découpé, grain, ombres douces | Fredoka |
| `craie` | tableau noir, décors à la craie | Caveat |
| `infographie` | aplats nets, style data-journalisme | Poppins |
| `nuit` | fond bleu nuit, éléments lumineux | Space Grotesk |

## Installation

Copiez le dossier `documentaire-motion-design/` dans votre dossier de skills :

```bash
git clone https://github.com/Sami-Fd/documentaire-motion-design.git
# skill personnel
cp -r documentaire-motion-design/documentaire-motion-design ~/.claude/skills/
# ou skill de projet
cp -r documentaire-motion-design/documentaire-motion-design .claude/skills/
```

Puis demandez par exemple à Claude : « Fais-moi une vidéo documentaire de 3 minutes sur le cycle de l'eau, style craie. »

## Prérequis

- Node.js 18+ (Remotion 4)
- ffmpeg (découpe et contrôle des pistes audio)
- Python 3.9+ pour les scripts de voix (`pip install edge-tts` pour la voix gratuite)

## Contribuer

Les corrections et suggestions sont les bienvenues : voir [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

[MIT](LICENSE)
