# Changelog

## 1.1.0
- `creerFilm` takes a 4th parameter `marge` (gap after each voice track, default 1.2 s) for tighter pacing
- Voice speed-up guide (`atempo`, 10 to 15 %, with a smaller `marge`)
- Fallback when ElevenLabs ignores `<break>` tags: align on a reference voice (MFCC DTW), snap cuts to nearest pause, verify with a cost matrix
- Exact formula for YouTube chapter timestamps
- Convention for topic-specific reusable pieces (`src/<sujet>/pieces.tsx`)
- Voice script docstrings no longer point to a project-specific npm script

## 1.0.0
- First release: Remotion engine, 4 styles, voice-over scripts, subtitles, thumbnail and YouTube chapters
