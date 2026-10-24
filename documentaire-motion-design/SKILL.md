---
name: documentaire-motion-design
description: Créer des vidéos documentaires en motion design codé (Remotion, React/SVG) rendues en MP4, avec voix off, sous-titres, styles au choix et miniature YouTube.
---

# Documentaire en motion design codé

Fabrique des vidéos documentaires et explicatives où **chaque image est dessinée par du code** (React + SVG avec Remotion), comme une scène de jeu vidéo ou une diapo PowerPoint animée, puis rendues en MP4. Pas d'images générées par IA : des formes, des personnages, des graphiques et du texte animés.

Livrables habituels : le MP4 1080p avec voix off et sous-titres, le texte de la voix off, la miniature YouTube (1280x720), le titre et la description YouTube avec chapitres.

## Styles disponibles

| Style | Rendu | Police |
|---|---|---|
| `papier` (défaut) | collage de papier découpé : bords irréguliers, grain, ombres douces, couleurs pastel | Fredoka |
| `craie` | tableau noir vert ardoise, décors à la craie, notes en papier | Caveat |
| `infographie` | aplats nets et clairs, ombres légères, style data-journalisme | Poppins |
| `nuit` | fond bleu nuit, éléments lumineux avec halo | Space Grotesk |

On choisit au rendu : `--props='{"style":"craie"}'`. Si la personne ne précise rien, utiliser `papier` et proposer les autres. Pour un nouveau style : ajouter une entrée dans `STYLES` (`src/theme.ts`) et, si besoin, un filtre dans `DefsPapier` (`src/papier.tsx`).

## Déroulé

### 1. Monter le projet
- Si un projet existe déjà (chercher `src/moteur/film.tsx`), le réutiliser.
- Sinon, créer les fichiers du « Kit » ci-dessous, puis `npm install`.
- Dans un conteneur sans Chrome système, ajouter `--browser-executable=<chemin de chromium>` aux commandes Remotion (ex. `/opt/pw-browsers/chromium-*/chrome-linux64/chrome`).

### 2. Recherche et plan
- Vérifier chaque chiffre sur des sources officielles et récentes (instituts statistiques, lois, rapports). Noter la source de chaque chiffre : elle s'affiche à l'écran (composant `Source`) et va dans la description YouTube.
- Plan type de 8 à 12 scènes : titre → contexte/origine → fonctionnement → chiffres clés (graphiques) → problème/tension → avenir/choix → conclusion. **Une idée par scène**, 25 à 45 mots de narration.
- Rester factuel et neutre sur les sujets politiques : présenter les options, pas une opinion.

### 3. Écrire la narration
`src/<sujet>/narration.json` : `[{"id": "01-titre", "texte": "…"}, …]`. Les ids donnent les noms des fichiers audio. Écrire pour l'oral : phrases courtes, nombres arrondis (« plus de 930 milliards »). Donner aussi ce texte à la personne, scène par scène, prêt à copier. **Ne pas mettre les titres de scène dans le texte à lire** : les synthèses vocales les lisent à voix haute.

### 4. Coder les scènes
`src/<sujet>/scenes.tsx` : une fonction React par scène, qui reçoit `{texte, duree}`.
- Toujours `<Toile>` puis `<Fond />` en premier. Zone utile : y 260 à 880 (le titre `Etiquette` est en haut vers y=150, les sous-titres en bas).
- Faire apparaître chaque élément **quand la voix en parle** : `delai={instantDuMot(texte, 'mot', duree)}`.
- Pièces disponibles (`src/papier.tsx`) : `Pop` (apparition avec rebond), `Glisse` (entrée par translation), `Etiquette` (titre + sous-titre), `Texte` (`surFond` pour un texte posé sur le décor plutôt que sur une carte), `Personnage` (`vieux` pour un senior), `Piece` (euro), `Soleil`, `Nuage`, `Mer`, `Goutte`, `Collines`, `Fond`, `interp`, `useRessort`.
- Graphiques : barres (rect dont la hauteur suit `interp`), anneau (fonction `arc`), compteur (nombre interpolé), pictogrammes (rangées de `Personnage`). Voir l'exemple.
- Pièces propres à un sujet (icônes, objets) : les mettre dans `src/<sujet>/pieces.tsx` et les réutiliser d'un film à l'autre (ex. pour la cybersécurité : `Serveur`, `Navigateur`, `Cadenas`, `Robot`, `Requete`, `Alerte`, `Bouclier`, `Tampon`, `Carte`).
- Couleurs : `C.*` (`src/theme.ts`) pour que tous les styles fonctionnent. Texte sur carte claire : couleur par défaut. Texte posé sur le décor : `surFond`.
- Puis `src/<sujet>/Film.tsx` : `creerFilm(narration, [Scene1, …], 'voix/<sujet>', marge)`, et ajouter la `<Composition>` avec `calculateMetadata` dans `src/Root.tsx` (même modèle que dans le Kit). `marge` (1,2 s par défaut) est le silence laissé après chaque piste ; le baisser (ex. 0,6) donne un rythme plus soutenu.

### 5. Vérifier avant de rendre
Rendre des images fixes vers 80 % de chaque scène, les assembler en planche (PIL) et **les regarder** : chevauchements, texte qui dépasse, éléments hors cadre, lisibilité dans le style choisi.
`npx remotion still src/index.ts <Id> out/test.png --frame=<n> --props='{"style":"papier"}'`

### 6. Voix off
Les pistes vont dans `public/voix/<sujet>/<id>.mp3`. Chaque scène prend la durée de sa piste + `marge` (1,2 s par défaut). Sans pistes, la vidéo sort quand même, sous-titrée, avec des durées estimées.
- Voix naturelle de la personne (ElevenLabs, etc.) : `python decouper_voix.py --preparer --narration …` crée `texte_elevenlabs.txt`, avec `<break time="2.0s" />` entre les scènes. La personne le colle tel quel et renvoie un seul MP3. Ensuite, `python decouper_voix.py voix.mp3 --narration … --sortie public/voix/<sujet>` le découpe sur les longues pauses et vérifie le nombre de scènes.
- ElevenLabs **ignore parfois les balises `<break>`** : le script trouve alors trop peu de pauses et s'arrête. Dans ce cas, aligner sur une voix de référence Google (voir ci-dessous), puis caler chaque coupe sur la pause la plus proche (silencedetect ffmpeg, seuil plus bas, ex. 0,25 s à -38 dB) et vérifier avec la matrice de coûts.
- Si la personne a collé les titres et qu'ils ont été lus : retrouver les coupes à la main, en alignant sur une voix de référence Google par DTW de MFCC (librosa), puis vérifier par une matrice de coûts DTW (scène i contre référence j : la diagonale doit être nettement plus basse).
- **Accélérer la voix** si la personne trouve le rythme lent : `ffmpeg -i in.mp3 -af atempo=1.12 out.mp3` sur chaque piste (garde la hauteur de la voix). Proposer 10 à 15 % (au-delà, ça sonne pressé) et baisser `marge` en même temps (ex. 0,6). Garder les pistes d'origine dans un autre dossier pour pouvoir revenir en arrière, et recalculer les chapitres YouTube après.
- Gratuit, sur l'ordinateur de la personne : `python generer_voix.py` (edge-tts, voix Microsoft `fr-FR-HenriNeural` / `fr-FR-DeniseNeural`).
- Depuis un conteneur au réseau filtré : `python generer_voix_google.py` (voix Google Traduction, correcte mais robotique ; à présenter comme une version provisoire).

### 7. Rendu
`npx remotion render src/index.ts <Id> out/<nom>.mp4 --props='{"style":"…"}' --concurrency=4` (compter environ 2 minutes de rendu par minute de film). Contrôler avec ffprobe que le fichier a bien une piste audio et la bonne durée.

### 8. Pour YouTube
- **Miniature** : un `<Still>` 1280x720 dans le même style. Deux à quatre mots énormes, une question ou un chiffre choc, un ou deux personnages, un élément graphique fort (flèche, chiffre). Pas plus de 2 Mo.
- **Titre** : moins de 70 caractères, sujet et promesse (« … expliqué en 2 minutes »), avec 2 ou 3 variantes.
- **Description** : 2-3 phrases d'accroche, **chapitres** calculés à partir des durées réelles : début de la scène n = somme des `ceil((durée piste + marge) × 30) / 30` des scènes précédentes, arrondi à la seconde inférieure (première ligne `0:00`, au moins 3 chapitres de 10 s ou plus), sources, 3 à 5 hashtags.

## Kit : fichiers du projet

#### `package.json`

```json
{
  "name": "documentaire-code",
  "private": true,
  "scripts": {
    "studio": "remotion studio"
  },
  "dependencies": {
    "@remotion/cli": "4.0.538",
    "@remotion/google-fonts": "4.0.538",
    "@remotion/media-utils": "4.0.538",
    "react": "19.1.0",
    "react-dom": "19.1.0",
    "remotion": "4.0.538"
  },
  "devDependencies": {
    "@types/react": "19.1.8",
    "typescript": "5.8.3"
  }
}
```

#### `tsconfig.json`

```json
{"compilerOptions":{"resolveJsonModule":true,"target":"ES2022","module":"ESNext","moduleResolution":"bundler","jsx":"react-jsx","strict":true,"skipLibCheck":true,"noEmit":true},"include":["src"]}
```

#### `src/index.ts`

```ts
import {registerRoot} from 'remotion';
import {RemotionRoot} from './Root';

registerRoot(RemotionRoot);
```

#### `src/theme.ts`

```ts
import {getInputProps} from 'remotion';
import {loadFont as fredoka} from '@remotion/google-fonts/Fredoka';
import {loadFont as caveat} from '@remotion/google-fonts/Caveat';
import {loadFont as poppins} from '@remotion/google-fonts/Poppins';
import {loadFont as spaceGrotesk} from '@remotion/google-fonts/SpaceGrotesk';

// Les styles visuels disponibles. On choisit au rendu :
//   npx remotion render ... --props='{"style":"craie"}'
// Le texte posé sur des cartes/étiquettes reste toujours foncé ; `texteFond` sert
// au texte écrit directement sur le décor.
export type NomStyle = 'papier' | 'craie' | 'infographie' | 'nuit';

type Style = {
  couleurs: typeof PAPIER;
  police: () => string;
  filtre: 'papier' | 'craie' | 'plat' | 'lueur';
};

const PAPIER = {
  creme: '#fbf3e4',
  ciel: '#cfe8ef',
  ciel2: '#c3dde8',
  ciel3: '#b9cbd6',
  soleil: '#ffc94d',
  soleilBord: '#f5a623',
  colline1: '#9fd18b',
  colline2: '#6fb36b',
  colline3: '#4f9a5c',
  mer: '#6cb7d9',
  merFonce: '#3f8fbf',
  goutte: '#4fa3e0',
  nuage: '#ffffff',
  nuageGris: '#c9d3dc',
  montagne: '#b7a28c',
  neige: '#ffffff',
  texte: '#4a3b2c',
  texteFond: '#4a3b2c',
  etiquette: '#fff8ea',
  accent: '#e8735a',
};

const STYLES: Record<NomStyle, Style> = {
  // Collage de papier découpé : bords irréguliers, grain, ombres douces.
  papier: {couleurs: PAPIER, filtre: 'papier', police: () => fredoka('normal', {weights: ['500', '600'], subsets: ['latin', 'latin-ext']}).fontFamily},
  // Tableau noir : fond vert ardoise, décors tracés à la craie, notes en papier.
  craie: {
    couleurs: {...PAPIER, creme: '#2a2420', ciel: '#2f4a3c', ciel2: '#2c4538', ciel3: '#293f34', colline1: '#4d6b55', colline2: '#425e4a',
      colline3: '#38513f', mer: '#4f7f8f', merFonce: '#3d6876', montagne: '#6b6358', nuageGris: '#9aa5a0', texteFond: '#f4f1e8',
      etiquette: '#f6f0dc'},
    filtre: 'craie',
    police: () => caveat('normal', {weights: ['500', '700'], subsets: ['latin', 'latin-ext']}).fontFamily,
  },
  // Infographie moderne : aplats nets, ombres légères, typographie géométrique.
  infographie: {
    couleurs: {...PAPIER, creme: '#e9edf5', ciel: '#f7f9fc', ciel2: '#eef3f9', ciel3: '#e6ecf3', colline1: '#b8e0c8', colline2: '#8fcfa9',
      colline3: '#66bb8a', texte: '#1f2a44', texteFond: '#1f2a44', etiquette: '#ffffff'},
    filtre: 'plat',
    police: () => poppins('normal', {weights: ['500', '600'], subsets: ['latin', 'latin-ext']}).fontFamily,
  },
  // Nuit : fond bleu nuit, éléments lumineux avec un halo.
  nuit: {
    couleurs: {...PAPIER, creme: '#070914', ciel: '#121734', ciel2: '#10142d', ciel3: '#0e1228', colline1: '#24406a', colline2: '#1c3458',
      colline3: '#152846', mer: '#2c6fb0', merFonce: '#1f5590', montagne: '#3a3f63', nuage: '#dfe6ff', nuageGris: '#8a93b8',
      texte: '#141a33', texteFond: '#eef1ff', etiquette: '#e9edff'},
    filtre: 'lueur',
    police: () => spaceGrotesk('normal', {weights: ['500', '600'], subsets: ['latin', 'latin-ext']}).fontFamily,
  },
};

const demande = (getInputProps() as {style?: NomStyle}).style;
export const NOM_STYLE: NomStyle = demande && demande in STYLES ? demande : 'papier';
const style = STYLES[NOM_STYLE];

export const C = style.couleurs;
export const FILTRE = style.filtre;
export const POLICE = style.police();
```

#### `src/papier.tsx`

```tsx
import React from 'react';
import {interpolate, spring, useCurrentFrame, useVideoConfig} from 'remotion';
import {C, FILTRE, POLICE} from './theme';

// Filtre SVG qui donne l'effet « papier découpé » : bords légèrement irréguliers,
// grain de papier et ombre portée. À placer une fois dans chaque <svg> de scène.
export const DefsPapier = () => (
  <defs>
    {FILTRE === 'papier' && (
      <filter id="papier" x="-10%" y="-10%" width="120%" height="135%">
        <feTurbulence type="fractalNoise" baseFrequency="0.035" numOctaves="2" seed="4" result="bruit" />
        <feDisplacementMap in="SourceGraphic" in2="bruit" scale="7" xChannelSelector="R" yChannelSelector="G" result="bord" />
        <feTurbulence type="fractalNoise" baseFrequency="0.8" numOctaves="2" seed="9" result="grain" />
        <feColorMatrix in="grain" type="matrix"
          values="0 0 0 0 0.35  0 0 0 0 0.28  0 0 0 0 0.2  0 0 0 0.35 -0.05" result="grainBrun" />
        <feComposite in="grainBrun" in2="bord" operator="in" result="grainMasque" />
        <feMerge result="texture">
          <feMergeNode in="bord" />
          <feMergeNode in="grainMasque" />
        </feMerge>
        <feDropShadow dx="0" dy="7" stdDeviation="6" floodColor="#5a4630" floodOpacity="0.32" />
      </filter>
    )}
    {FILTRE === 'craie' && (
      // Bords tremblés et texture granuleuse de craie (des « trous » dans la matière).
      <filter id="papier" x="-10%" y="-10%" width="120%" height="130%">
        <feTurbulence type="fractalNoise" baseFrequency="0.05" numOctaves="2" seed="2" result="bruit" />
        <feDisplacementMap in="SourceGraphic" in2="bruit" scale="5" xChannelSelector="R" yChannelSelector="G" result="bord" />
        <feTurbulence type="fractalNoise" baseFrequency="1.2" numOctaves="1" seed="7" result="grain" />
        <feColorMatrix in="grain" type="matrix" values="0 0 0 0 0  0 0 0 0 0  0 0 0 0 0  0 0 0 -1.3 1.55" result="trous" />
        <feComposite in="bord" in2="trous" operator="in" result="crayeux" />
        <feDropShadow dx="0" dy="4" stdDeviation="3" floodColor="#000" floodOpacity="0.35" />
      </filter>
    )}
    {FILTRE === 'plat' && (
      <filter id="papier" x="-10%" y="-10%" width="120%" height="135%">
        <feDropShadow dx="0" dy="8" stdDeviation="10" floodColor="#1f2a44" floodOpacity="0.12" />
      </filter>
    )}
    {FILTRE === 'lueur' && (
      <filter id="papier" x="-20%" y="-20%" width="140%" height="150%">
        <feGaussianBlur in="SourceGraphic" stdDeviation="9" result="flou" />
        <feColorMatrix in="flou" type="matrix" values="1 0 0 0 0.1  0 1 0 0 0.15  0 0 1 0 0.35  0 0 0 0.7 0" result="halo" />
        <feMerge>
          <feMergeNode in="halo" />
          <feMergeNode in="SourceGraphic" />
        </feMerge>
      </filter>
    )}
  </defs>
);

// Ressort avec un petit rebond : la pièce « saute » en place comme dans un collage.
export const useRessort = (delai: number, rebond = true) => {
  const f = useCurrentFrame();
  const {fps} = useVideoConfig();
  return spring({frame: f - delai, fps, config: rebond ? {damping: 11, stiffness: 140} : {damping: 200}});
};

// Une pièce de papier qui apparaît en grandissant depuis son point d'ancrage (cx, cy).
export const Pop: React.FC<{cx: number; cy: number; delai?: number; children: React.ReactNode}> = ({
  cx, cy, delai = 0, children,
}) => {
  const s = useRessort(delai);
  return (
    <g filter="url(#papier)" transform={`translate(${cx} ${cy}) scale(${s}) translate(${-cx} ${-cy})`}>
      {children}
    </g>
  );
};

// Une pièce qui glisse depuis un décalage (dx, dy) jusqu'à sa place.
export const Glisse: React.FC<{dx?: number; dy?: number; delai?: number; children: React.ReactNode}> = ({
  dx = 0, dy = 0, delai = 0, children,
}) => {
  const p = useRessort(delai, false);
  return (
    <g filter="url(#papier)" transform={`translate(${dx * (1 - p)} ${dy * (1 - p)})`}>
      {children}
    </g>
  );
};

// Étiquette en papier avec un titre et une phrase courte (une idée par scène).
export const Etiquette: React.FC<{titre: string; texte?: string; x?: number; y?: number; delai?: number; largeur?: number}> = ({
  titre, texte, x = 960, y = 150, delai = 10, largeur = 1000,
}) => {
  const h = texte ? 190 : 130;
  const s = useRessort(delai);
  const pTexte = useRessort(delai + 12, false);
  return (
    <g transform={`translate(${x} ${y}) scale(${s}) rotate(${-1.5 * s})`}>
      <g filter="url(#papier)">
        <rect x={-largeur / 2} y={-h / 2} width={largeur} height={h} rx={26} fill={C.etiquette} />
      </g>
      <text y={texte ? -18 : 22} textAnchor="middle" fontFamily={POLICE} fontWeight={600} fontSize={64} fill={C.texte}>
        {titre}
      </text>
      {texte && (
        <text y={52} textAnchor="middle" fontFamily={POLICE} fontWeight={500} fontSize={38} fill={C.texte} opacity={pTexte}>
          {texte}
        </text>
      )}
    </g>
  );
};

// Goutte d'eau avec deux petits yeux : notre personnage principal.
export const Goutte: React.FC<{x: number; y: number; taille?: number; yeux?: boolean}> = ({x, y, taille = 1, yeux = true}) => (
  <g transform={`translate(${x} ${y}) scale(${taille})`}>
    <path d="M0 -40 C 18 -14, 30 4, 30 18 A 30 30 0 0 1 -30 18 C -30 4, -18 -14, 0 -40 Z" fill={C.goutte} />
    <ellipse cx={-11} cy={10} rx={6} ry={10} fill="white" opacity={0.55} />
    {yeux && (
      <>
        <circle cx={-9} cy={20} r={3.6} fill="#3a2c20" />
        <circle cx={9} cy={20} r={3.6} fill="#3a2c20" />
      </>
    )}
  </g>
);

export const Nuage: React.FC<{x: number; y: number; taille?: number; couleur?: string}> = ({x, y, taille = 1, couleur = C.nuage}) => (
  <g transform={`translate(${x} ${y}) scale(${taille})`} fill={couleur}>
    <circle cx={-70} cy={10} r={50} />
    <circle cx={0} cy={-20} r={70} />
    <circle cx={75} cy={5} r={55} />
    <rect x={-120} y={10} width={250} height={55} rx={27} />
  </g>
);

export const Soleil: React.FC<{x: number; y: number; r?: number}> = ({x, y, r = 90}) => {
  const f = useCurrentFrame();
  return (
    <g transform={`translate(${x} ${y})`}>
      <g transform={`rotate(${f * 0.4})`}>
        {Array.from({length: 12}).map((_, i) => (
          <rect key={i} x={-9} y={-r - 48} width={18} height={36} rx={9} fill={C.soleilBord} transform={`rotate(${i * 30})`} />
        ))}
      </g>
      <circle r={r} fill={C.soleil} />
    </g>
  );
};

// Vagues de mer qui ondulent doucement.
export const Mer: React.FC<{y?: number}> = ({y = 860}) => {
  const f = useCurrentFrame();
  const vague = (decal: number, amp: number) => {
    let d = `M -50 ${y}`;
    for (let x = -50; x <= 1970; x += 40) {
      d += ` L ${x} ${y + Math.sin(x / 90 + f / 12 + decal) * amp}`;
    }
    return d + ' L 1970 1100 L -50 1100 Z';
  };
  return (
    <g filter="url(#papier)">
      <path d={vague(0, 14)} fill={C.mer} />
      <path d={vague(2, 10)} transform="translate(0 70)" fill={C.merFonce} />
    </g>
  );
};

// Fond crème + ciel, commun à toutes les scènes.
export const Fond: React.FC<{ciel?: string}> = ({ciel = C.ciel}) => (
  <>
    <rect width={1920} height={1080} fill={C.creme} />
    <rect x={30} y={30} width={1860} height={1020} rx={40} fill={ciel} filter="url(#papier)" />
  </>
);

export const interp = (f: number, a: number[], b: number[]) =>
  interpolate(f, a, b, {extrapolateLeft: 'clamp', extrapolateRight: 'clamp'});

// Petit personnage en papier. `vieux` ajoute des cheveux gris et une canne.
export const Personnage: React.FC<{x: number; y: number; couleur: string; taille?: number; vieux?: boolean; phase?: number}> = ({
  x, y, couleur, taille = 1, vieux = false, phase = 0,
}) => {
  const f = useCurrentFrame();
  const saut = Math.abs(Math.sin(f / 10 + phase)) * 6;
  return (
    <g transform={`translate(${x} ${y - saut}) scale(${taille})`}>
      {vieux && <rect x={34} y={-60} width={7} height={70} rx={3} fill="#8a6a4a" />}
      <rect x={-34} y={-70} width={68} height={80} rx={30} fill={couleur} />
      <circle cx={0} cy={-100} r={28} fill="#f1c9a0" />
      {vieux && <path d="M -28 -104 A 28 28 0 0 1 28 -104 L 22 -112 A 22 22 0 0 0 -22 -112 Z" fill="#e6e6e6" />}
      {!vieux && <path d="M -28 -102 A 28 28 0 0 1 28 -102 Q 0 -118 -28 -102 Z" fill="#5b3b25" />}
      <circle cx={-9} cy={-98} r={3.2} fill="#3a2c20" />
      <circle cx={9} cy={-98} r={3.2} fill="#3a2c20" />
    </g>
  );
};

// Pièce de monnaie en euros.
export const Piece: React.FC<{x: number; y: number; taille?: number}> = ({x, y, taille = 1}) => (
  <g transform={`translate(${x} ${y}) scale(${taille})`}>
    <circle r={22} fill="#f5c542" />
    <circle r={16} fill="none" stroke="#d9a520" strokeWidth={3} />
    <text y={8} textAnchor="middle" fontFamily={POLICE} fontWeight={600} fontSize={24} fill="#a87a10">€</text>
  </g>
);

// Texte SVG avec la police du film.
export const Texte: React.FC<{x: number; y: number; taille?: number; gras?: boolean; couleur?: string; ancre?: 'start' | 'middle' | 'end'; opacite?: number; surFond?: boolean; children: React.ReactNode}> = ({
  x, y, taille = 40, gras = false, couleur, ancre = 'middle', opacite = 1, surFond = false, children,
}) => (
  <text x={x} y={y} textAnchor={ancre} fontFamily={POLICE} fontWeight={gras ? 600 : 500} fontSize={taille} fill={couleur ?? (surFond ? C.texteFond : C.texte)} opacity={opacite}>
    {children}
  </text>
);

// Toile SVG d'une scène (1920x1080) : filtres de style + cadre arrondi qui coupe ce qui dépasse.
export const Toile: React.FC<{children: React.ReactNode}> = ({children}) => (
  <svg viewBox="0 0 1920 1080" width="100%" height="100%">
    <DefsPapier />
    <clipPath id="cadre"><rect x={30} y={30} width={1860} height={1020} rx={40} /></clipPath>
    <g clipPath="url(#cadre)">{children}</g>
  </svg>
);

const colline = (yBase: number, amp: number, decal = 0) =>
  `M 30 1050 L 30 ${yBase} Q ${480 + decal} ${yBase - amp} ${960 + decal} ${yBase} T 1890 ${yBase} L 1890 1050 Z`;

export const Collines = ({delai = 0}: {delai?: number}) => (
  <>
    <Glisse dy={500} delai={delai}><path d={colline(760, 160, 120)} fill={C.colline1} /></Glisse>
    <Glisse dy={500} delai={delai + 6}><path d={colline(850, 120, -200)} fill={C.colline2} /></Glisse>
    <Glisse dy={500} delai={delai + 12}><path d={colline(950, 90, 60)} fill={C.colline3} /></Glisse>
  </>
);
```

#### `src/moteur/film.tsx`

```tsx
import React from 'react';
import {AbsoluteFill, CalculateMetadataFunction, Html5Audio, interpolate, Series, staticFile, useCurrentFrame} from 'remotion';
import {getAudioDurationInSeconds} from '@remotion/media-utils';
import {C, POLICE} from '../theme';

// Moteur commun à tous les films : enchaîne les scènes, pose la voix off et les
// sous-titres, et cale la durée de chaque scène sur sa piste audio.

export const FPS = 30;
export type Narration = {id: string; texte: string}[];
export type PropsScene = {texte: string; duree: number};
export type PropsFilm = {durees: number[]; avecVoix: boolean[]; sousTitres: boolean; style?: string};

// Instant (en images) où un mot est prononcé, estimé d'après sa position dans le texte.
// Sert à faire apparaître chaque élément au moment où la voix en parle.
export const instantDuMot = (texte: string, mot: string, duree: number) => {
  const i = texte.indexOf(mot);
  if (i < 0) return 0;
  return Math.round((i / texte.length) * (duree - 15) * 0.85);
};

const Fondu: React.FC<{duree: number; children: React.ReactNode}> = ({duree, children}) => {
  const f = useCurrentFrame();
  const opacity = interpolate(f, [0, 10, duree - 10, duree], [0, 1, 1, 0], {extrapolateLeft: 'clamp', extrapolateRight: 'clamp'});
  return <AbsoluteFill style={{opacity}}>{children}</AbsoluteFill>;
};

// Sous-titres : le texte est découpé en phrases, chacune affichée pendant une durée
// proportionnelle à sa longueur.
const SousTitres: React.FC<{texte: string; duree: number}> = ({texte, duree}) => {
  const f = useCurrentFrame();
  const phrases = texte.split(/(?<=[.?!])\s+/);
  const total = phrases.reduce((a, p) => a + p.length, 0);
  const utile = duree - 20;
  let debut = 5;
  let actuelle = phrases[phrases.length - 1];
  for (const p of phrases) {
    const d = (p.length / total) * utile;
    if (f < debut + d) {
      actuelle = p;
      break;
    }
    debut += d;
  }
  return (
    <AbsoluteFill style={{justifyContent: 'flex-end', alignItems: 'center', paddingBottom: 62}}>
      <div style={{fontFamily: POLICE, fontWeight: 500, fontSize: 38, lineHeight: 1.3, color: '#fff', background: 'rgba(30,25,20,0.82)',
        padding: '12px 30px', borderRadius: 14, maxWidth: 1500, textAlign: 'center'}}>
        {actuelle}
      </div>
    </AbsoluteFill>
  );
};

// `dossierVoix` : sous-dossier de public/ qui contient <id>.mp3 pour chaque scène.
// `marge` : secondes ajoutées après chaque piste avant la scène suivante (plus petit = rythme plus soutenu).
export const creerFilm = (narration: Narration, scenes: React.FC<PropsScene>[], dossierVoix: string, marge = 1.2) => {
  if (scenes.length !== narration.length) {
    throw new Error(`${narration.length} textes de narration mais ${scenes.length} scènes`);
  }

  // Durée de chaque scène : celle de sa piste de voix si elle existe,
  // sinon une estimation d'après le nombre de mots.
  const calculerDurees: CalculateMetadataFunction<PropsFilm> = async ({props}) => {
    const res = await Promise.all(
      narration.map(async (s) => {
        const src = staticFile(`${dossierVoix}/${s.id}.mp3`);
        try {
          const r = await fetch(src, {method: 'HEAD'});
          if (!r.ok) throw new Error('absent');
          const sec = await getAudioDurationInSeconds(src);
          return {duree: Math.ceil((sec + marge) * FPS), voix: true};
        } catch {
          const mots = s.texte.split(/\s+/).length;
          return {duree: Math.ceil((mots / 2.4 + 1.5) * FPS), voix: false};
        }
      }),
    );
    const durees = res.map((r) => r.duree);
    return {durationInFrames: durees.reduce((a, b) => a + b, 0), props: {...props, durees, avecVoix: res.map((r) => r.voix)}};
  };

  const Film: React.FC<PropsFilm> = ({durees, avecVoix, sousTitres}) => (
    <AbsoluteFill style={{backgroundColor: C.creme}}>
      <Series>
        {narration.map((s, i) => {
          const Scene = scenes[i];
          const duree = durees[i];
          return (
            <Series.Sequence key={s.id} durationInFrames={duree}>
              <Fondu duree={duree}>
                <Scene texte={s.texte} duree={duree} />
              </Fondu>
              {sousTitres && <SousTitres texte={s.texte} duree={duree} />}
              {avecVoix[i] && <Html5Audio src={staticFile(`${dossierVoix}/${s.id}.mp3`)} />}
            </Series.Sequence>
          );
        })}
      </Series>
    </AbsoluteFill>
  );

  return {Film, calculerDurees};
};
```

#### `generer_voix.py`

```python
"""Génère la voix off de la vidéo, une piste MP3 par scène.

Utilise edge-tts (voix neuronales gratuites de Microsoft, pas de clé API).
Installation :  pip install edge-tts
Lancement   :  python generer_voix.py              (voix masculine par défaut)
               python generer_voix.py --voix fr-FR-DeniseNeural
Liste des voix françaises :  edge-tts --list-voices | grep fr-

Le texte est lu depuis --narration (par défaut src/social/narration.json) : modifiez-le là, relancez ce
script, puis relancez le rendu (`npx remotion render …`). La vidéo se cale toute seule sur la
durée de chaque piste.
"""
import argparse
import asyncio
import json
from pathlib import Path

import edge_tts



async def main(voix: str, debit: str, texte: Path, sortie: Path) -> None:
    sortie.mkdir(parents=True, exist_ok=True)
    for scene in json.loads(texte.read_text(encoding="utf-8")):
        fichier = sortie / f"{scene['id']}.mp3"
        await edge_tts.Communicate(scene["texte"], voix, rate=debit).save(str(fichier))
        print(f"OK  {fichier}")


if __name__ == "__main__":
    p = argparse.ArgumentParser()
    p.add_argument("--voix", default="fr-FR-HenriNeural")
    p.add_argument("--debit", default="-5%", help="vitesse de lecture, ex. -10%% ou +5%%")
    p.add_argument("--narration", default="src/social/narration.json")
    p.add_argument("--sortie", default="public/voix")
    a = p.parse_args()
    asyncio.run(main(a.voix, a.debit, Path(a.narration), Path(a.sortie)))
```

#### `generer_voix_google.py`

```python
"""Variante de generer_voix.py qui utilise la voix de Google Traduction (sans clé).
Moins naturelle qu'edge-tts, mais sans dépendance : il faut seulement ffmpeg.
Lancement : python generer_voix_google.py
"""
import json
import re
import subprocess
import tempfile
import urllib.parse
import urllib.request
from pathlib import Path

URL = "https://translate.googleapis.com/translate_tts?client=gtx&ie=UTF-8&tl=fr&q="


def morceaux(texte: str, max_len: int = 180) -> list[str]:
    """Découpe le texte en phrases (puis aux virgules si besoin), car le service limite la longueur."""
    res = []
    for phrase in re.split(r"(?<=[.?!])\s+", texte):
        while len(phrase) > max_len:
            coupe = phrase.rfind(",", 0, max_len)
            coupe = coupe if coupe > 0 else phrase.rfind(" ", 0, max_len)
            res.append(phrase[: coupe + 1].strip())
            phrase = phrase[coupe + 1 :].strip()
        res.append(phrase)
    return [m for m in res if m]


def main() -> None:
    import argparse
    p = argparse.ArgumentParser()
    p.add_argument("--narration", default="src/social/narration.json")
    p.add_argument("--sortie", default="public/voix")
    a = p.parse_args()
    SORTIE = Path(a.sortie)
    SORTIE.mkdir(parents=True, exist_ok=True)
    for scene in json.loads(Path(a.narration).read_text(encoding="utf-8")):
        with tempfile.TemporaryDirectory() as tmp:
            liste = Path(tmp) / "liste.txt"
            lignes = []
            for i, m in enumerate(morceaux(scene["texte"])):
                mp3 = Path(tmp) / f"{i:02d}.mp3"
                req = urllib.request.Request(URL + urllib.parse.quote(m), headers={"User-Agent": "Mozilla/5.0"})
                mp3.write_bytes(urllib.request.urlopen(req, timeout=30).read())
                lignes.append(f"file '{mp3}'")
            liste.write_text("\n".join(lignes))
            fichier = SORTIE / f"{scene['id']}.mp3"
            # Assemble les morceaux et accélère légèrement la voix (10 %).
            subprocess.run(["ffmpeg", "-loglevel", "error", "-y", "-f", "concat", "-safe", "0", "-i", str(liste),
                            "-af", "atempo=1.1", "-ar", "44100", str(fichier)], check=True)
        print(f"OK  {fichier}")


if __name__ == "__main__":
    main()
```

#### `decouper_voix.py`

```python
"""Découpe UNE piste de voix off (ex. un export ElevenLabs) en une piste par scène.

La piste doit contenir une pause d'au moins 1,2 s entre chaque scène, et seulement là.
Le plus simple : générer la voix à partir de `texte_elevenlabs.txt` (créé par
`python decouper_voix.py --preparer`), qui place une balise <break time="2.0s" />
entre les scènes. Ne collez pas les titres des scènes : ils seraient lus à voix haute.

Lancement : python decouper_voix.py --preparer            -> crée texte_elevenlabs.txt
            python decouper_voix.py ma_voix.mp3          -> crée public/voix/<id>.mp3
Options   : --narration src/social/narration.json  --pause 1.2
"""
import argparse
import json
import re
import subprocess
from pathlib import Path


def silences(fichier: str, pause: float) -> list[tuple[float, float]]:
    err = subprocess.run(["ffmpeg", "-i", fichier, "-af", f"silencedetect=n=-38dB:d={pause}", "-f", "null", "-"],
                         capture_output=True, text=True).stderr
    debuts = [float(x) for x in re.findall(r"silence_start: ([0-9.]+)", err)]
    fins = [float(x) for x in re.findall(r"silence_end: ([0-9.]+)", err)]
    duree = float(re.search(r"Duration: (\d+):(\d+):([0-9.]+)", err).group(3)) + 60 * int(
        re.search(r"Duration: (\d+):(\d+)", err).group(2)) + 3600 * int(re.search(r"Duration: (\d+)", err).group(1))
    # On ignore le silence du tout début et de la toute fin.
    return [(d, f) for d, f in zip(debuts, fins) if d > 0.05 and f < duree - 0.05], duree


def main() -> None:
    p = argparse.ArgumentParser()
    p.add_argument("voix", nargs="?")
    p.add_argument("--narration", default="src/social/narration.json")
    p.add_argument("--sortie", default="public/voix")
    p.add_argument("--pause", type=float, default=1.2)
    p.add_argument("--preparer", action="store_true")
    a = p.parse_args()
    scenes = json.loads(Path(a.narration).read_text(encoding="utf-8"))

    if a.preparer:
        Path("texte_elevenlabs.txt").write_text('\n<break time="2.0s" />\n'.join(s["texte"] for s in scenes), encoding="utf-8")
        print("OK  texte_elevenlabs.txt : collez-le tel quel dans ElevenLabs.")
        return

    pauses, duree = silences(a.voix, a.pause)
    if len(pauses) != len(scenes) - 1:
        raise SystemExit(
            f"{len(pauses)} longues pauses trouvées, il en faut {len(scenes) - 1} (une entre chaque scène). "
            "Régénérez la voix depuis texte_elevenlabs.txt, ou ajustez --pause.")
    bornes = [0.0] + [x for d, f in pauses for x in (d, f)] + [duree]
    sortie = Path(a.sortie)
    sortie.mkdir(parents=True, exist_ok=True)
    for k, s in enumerate(scenes):
        debut, fin = bornes[2 * k], bornes[2 * k + 1]
        subprocess.run(["ffmpeg", "-loglevel", "error", "-y", "-ss", f"{max(0, debut - 0.1):.3f}", "-to", f"{min(duree, fin + 0.2):.3f}",
                        "-i", a.voix, "-af", "afade=t=in:d=0.05,areverse,afade=t=in:d=0.08,areverse", "-ar", "44100", "-b:a", "192k",
                        str(sortie / f"{s['id']}.mp3")], check=True)
        print(f"{s['id']:<22} {debut:7.2f} -> {fin:7.2f}  ({fin - debut:5.2f} s)")


if __name__ == "__main__":
    main()
```

#### `src/Root.tsx` (modèle)

```tsx
import {Composition, Still} from 'remotion';
import {calculerDurees, SystemeSocial} from './social/SystemeSocial';
import {Miniature} from './social/Miniature';

export const RemotionRoot = () => (
  <>
    <Composition
      id="SystemeSocial"
      component={SystemeSocial}
      calculateMetadata={calculerDurees}
      defaultProps={{durees: [], avecVoix: [], sousTitres: true}}
      durationInFrames={300}
      fps={30}
      width={1920}
      height={1080}
    />
    <Still id="Miniature" component={Miniature} width={1280} height={720} />
  </>
);
```

#### `src/social/SystemeSocial.tsx`

```tsx
import {creerFilm} from '../moteur/film';
import narration from './narration.json';
import {Branches, Deficit, Financement, Fin, Leviers, Montant, Naissance, Repartition, Titre, Vieillissement} from './scenes';

export const {Film: SystemeSocial, calculerDurees} = creerFilm(
  narration,
  [Titre, Naissance, Branches, Repartition, Financement, Montant, Deficit, Vieillissement, Leviers, Fin],
  'voix',
);
```

#### `src/social/narration.json` (extrait)

```json
[
  {
    "id": "01-titre",
    "texte": "La France et son système social. Comment ça marche ? Et surtout, où va-t-on ?"
  },
  {
    "id": "02-1945",
    "texte": "Tout commence en 1945. Au sortir de la guerre, la France crée la Sécurité sociale. L'idée est simple : chacun cotise selon ses moyens, et reçoit selon ses besoins."
  }
]
```

## Exemple de scènes (`src/social/scenes.tsx`, extrait)

Titre, graphique en anneau synchronisé sur la voix, pictogrammes comparés.

```tsx
import React from 'react';
import {useCurrentFrame} from 'remotion';
import {C} from '../theme';
import {Collines, Etiquette, Fond, Glisse, interp, Personnage, Piece, Pop, Texte, Toile, useRessort} from '../papier';
import {instantDuMot, PropsScene} from '../moteur/film';



const BLEU = '#3f6fb5';
const ROUGE = '#d9534f';
const COULEURS_GENS = ['#e8735a', '#4f9a5c', '#3f6fb5', '#f0a830', '#9b6fc2', '#4fa3e0', '#d9534f', '#6fb36b'];

const Source: React.FC<{children: React.ReactNode}> = ({children}) => (
  <Texte x={1850} y={880} taille={24} ancre="end" surFond opacite={0.75}>Source : {children}</Texte>
);

const Carte: React.FC<{x: number; y: number; l: number; h: number; couleur?: string; rot?: number}> = ({x, y, l, h, couleur = C.etiquette, rot = 0}) => (
  <rect x={x - l / 2} y={y - h / 2} width={l} height={h} rx={24} fill={couleur} transform={`rotate(${rot} ${x} ${y})`} />
);

// 1. Titre
export const Titre: React.FC<PropsScene> = () => (
  <Toile>
    <Fond />
    <Collines />
    <Pop cx={1640} cy={560} delai={30}>
      <rect x={1636} y={300} width={8} height={300} fill="#8a6a4a" />
      <rect x={1644} y={300} width={60} height={130} fill={BLEU} />
      <rect x={1704} y={300} width={60} height={130} fill="#ffffff" />
      <rect x={1764} y={300} width={60} height={130} fill={ROUGE} />
    </Pop>
    {[0, 1, 2, 3, 4].map((i) => (
      <Pop key={i} cx={720 + i * 120} cy={760} delai={35 + i * 5}>
        <Personnage x={720 + i * 120} y={760} couleur={COULEURS_GENS[i]} vieux={i === 4} phase={i} />
      </Pop>
    ))}
    <Etiquette titre="La France et son système social" texte="Comment ça marche… et où va-t-on ?" y={330} largeur={1250} />
  </Toile>
);

// Arc de couronne pour le graphique en anneau.
const arc = (cx: number, cy: number, R: number, r: number, a0: number, a1: number) => {
  const pt = (rad: number, a: number) => [cx + rad * Math.sin(a), cy - rad * Math.cos(a)];
  const grand = a1 - a0 > Math.PI ? 1 : 0;
  const [x0, y0] = pt(R, a0), [x1, y1] = pt(R, a1), [x2, y2] = pt(r, a1), [x3, y3] = pt(r, a0);
  return `M ${x0} ${y0} A ${R} ${R} 0 ${grand} 1 ${x1} ${y1} L ${x2} ${y2} A ${r} ${r} 0 ${grand} 0 ${x3} ${y3} Z`;
};

// 5. Le financement
export const Financement: React.FC<PropsScene> = ({texte, duree}) => {
  const f = useCurrentFrame();
  const parts = [
    {nom: 'Cotisations sociales', v: 49, c: BLEU, mot: 'cotisations'},
    {nom: 'CSG', v: 20, c: '#e8735a', mot: 'CSG'},
    {nom: 'Autres impôts et taxes', v: 13, c: '#f0a830', mot: 'autres impôts'},
    {nom: 'TVA', v: 8, c: '#4f9a5c', mot: 'TVA'},
    {nom: 'Autres recettes', v: 10, c: '#b7a28c', mot: 'autres impôts'},
  ];
  let cumul = 0;
  return (
    <Toile>
      <Fond />
      <g filter="url(#papier)">
        {parts.map((p) => {
          const debut = (cumul / 100) * 2 * Math.PI;
          cumul += p.v;
          const t0 = instantDuMot(texte, p.mot, duree) - 10;
          const fin = debut + interp(f, [t0, t0 + 20], [0, 1]) * (p.v / 100) * 2 * Math.PI;
          return fin > debut + 0.001 ? <path key={p.nom} d={arc(640, 560, 270, 135, debut, fin)} fill={p.c} /> : null;
        })}
      </g>
      {parts.map((p, i) => {
        const t0 = instantDuMot(texte, p.mot, duree) - 10;
        return (
          <Glisse key={p.nom} dx={300} delai={t0}>
            <g opacity={interp(f, [t0, t0 + 8], [0, 1])}>
              <rect x={1040} y={380 + i * 90} width={50} height={50} rx={10} fill={p.c} />
              <Texte x={1115} y={420 + i * 90} taille={42} ancre="start" surFond>{p.nom}</Texte>
              <Texte x={1720} y={420 + i * 90} taille={42} gras ancre="end" surFond>{p.v} %</Texte>
            </g>
          </Glisse>
        );
      })}
      <Source>Fipeco, recettes 2022</Source>
      <Etiquette titre="Qui paie la Sécu ?" y={150} largeur={800} />
    </Toile>
  );
};

// 8. Le vieillissement : actifs (20-64 ans) pour chaque senior (65 ans et plus).
const Panneau: React.FC<{x: number; titre: string; ratio: string; actifs: number; seniors: number; delai: number}> = ({x, titre, ratio, actifs, seniors, delai}) => {
  const ecart = 62;
  return (
    <Pop cx={x} cy={560} delai={delai}>
      <Carte x={x} y={560} l={780} h={560} />
      <Texte x={x} y={350} taille={44} gras>{titre}</Texte>
      {Array.from({length: actifs}).map((_, i) => (
        <Personnage key={`a${i}`} x={x - ((actifs - 1) * ecart) / 2 + i * ecart} y={510} taille={0.55} couleur={COULEURS_GENS[i % 6]} phase={i} />
      ))}
      {Array.from({length: seniors}).map((_, i) => (
        <Personnage key={`s${i}`} x={x - ((seniors - 1) * ecart) / 2 + i * ecart} y={640} taille={0.55} couleur="#b7a28c" vieux phase={i + 3} />
      ))}
      <Texte x={x} y={750} taille={70} gras couleur={BLEU}>{ratio}</Texte>
      <Texte x={x} y={800} taille={30}>actifs pour 1 senior</Texte>
    </Pop>
  );
};

export const Vieillissement: React.FC<PropsScene> = ({texte, duree}) => (
  <Toile>
    <Fond />
    <Panneau x={540} titre="Aujourd'hui" ratio="2,5" actifs={5} seniors={2} delai={instantDuMot(texte, "Aujourd'hui", duree)} />
    <Panneau x={1380} titre="En 2070" ratio="1,6" actifs={8} seniors={5} delai={instantDuMot(texte, '2070', duree) - 10} />
    <Source>COR, rapport annuel 2026 (20-64 ans / 65 ans et plus)</Source>
    <Etiquette titre="La population vieillit" y={150} largeur={850} />
  </Toile>
);
```

## Exemple de miniature

#### `src/social/Miniature.tsx`

```tsx
import React from 'react';
import {C, POLICE} from '../theme';
import {DefsPapier, Personnage, Piece} from '../papier';

// Miniature YouTube (1280x720). Les éléments sont figés : pas d'animation.
const ROUGE = '#d9534f';
const BLEU = '#3f6fb5';

export const Miniature = () => (
  <svg viewBox="0 0 1280 720" width="100%" height="100%">
    <DefsPapier />
    <rect width={1280} height={720} fill={C.creme} />
    <rect x={18} y={18} width={1244} height={684} rx={30} fill={C.ciel} filter="url(#papier)" />
    {/* Drapeau en coin */}
    <g filter="url(#papier)" transform="rotate(-8 120 110)">
      <rect x={60} y={70} width={50} height={90} fill={BLEU} />
      <rect x={110} y={70} width={50} height={90} fill="#fff" />
      <rect x={160} y={70} width={50} height={90} fill={ROUGE} />
    </g>
    {/* Titre principal */}
    <g filter="url(#papier)">
      <rect x={250} y={50} width={760} height={150} rx={22} fill={C.etiquette} transform="rotate(-2 630 125)" />
    </g>
    <text x={630} y={160} textAnchor="middle" fontFamily={POLICE} fontWeight={600} fontSize={118} fill={C.texte} transform="rotate(-2 630 125)">
      LA SÉCU
    </text>
    <g filter="url(#papier)">
      <rect x={300} y={215} width={680} height={110} rx={20} fill={ROUGE} transform="rotate(1.5 640 270)" />
    </g>
    <text x={640} y={295} textAnchor="middle" fontFamily={POLICE} fontWeight={600} fontSize={76} fill="#fff" transform="rotate(1.5 640 270)">
      VA-T-ELLE TENIR ?
    </text>
    {/* Courbe du déficit qui plonge */}
    <g filter="url(#papier)">
      <path d="M 760 420 L 880 450 L 980 520 L 1090 620" fill="none" stroke={ROUGE} strokeWidth={26} strokeLinecap="round" strokeLinejoin="round" />
      <path d="M 1050 560 L 1135 665 L 1010 650 Z" fill={ROUGE} />
    </g>
    <g filter="url(#papier)">
      <rect x={950} y={360} width={270} height={110} rx={18} fill={C.etiquette} transform="rotate(4 1085 415)" />
    </g>
    <text x={1085} y={410} textAnchor="middle" fontFamily={POLICE} fontWeight={600} fontSize={54} fill={ROUGE} transform="rotate(4 1085 415)">−23 Md€</text>
    <text x={1085} y={450} textAnchor="middle" fontFamily={POLICE} fontWeight={500} fontSize={28} fill={C.texte} transform="rotate(4 1085 415)">de déficit en 2025</text>
    {/* Colline et personnages */}
    <path d="M 18 702 L 18 590 Q 400 500 760 600 T 1262 640 L 1262 702 Z" fill={C.colline2} filter="url(#papier)" />
    <g filter="url(#papier)">
      <Personnage x={160} y={600} couleur="#4f9a5c" taille={1.5} />
      <Personnage x={300} y={590} couleur="#f0a830" taille={1.5} />
      <Personnage x={450} y={600} couleur="#b7a28c" taille={1.5} vieux />
      <Personnage x={590} y={615} couleur="#b7a28c" taille={1.5} vieux />
    </g>
    <g filter="url(#papier)">
      <Piece x={230} y={400} taille={1.8} />
      <Piece x={380} y={430} taille={1.5} />
      <Piece x={530} y={395} taille={1.7} />
    </g>
  </svg>
);
```
