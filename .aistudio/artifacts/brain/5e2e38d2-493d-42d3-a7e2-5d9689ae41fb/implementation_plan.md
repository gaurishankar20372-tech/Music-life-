# AI Music Studio — Phase 1 Architecture & Implementation Plan

AI Music Studio is a premium, mobile-first, and responsive music production platform empowering creators to generate song concepts, record vocal tracks, craft lyric structures, configure synthetic AI singers, and arrange instrumentals with an integrated audio player and project manager.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> The following choices were confirmed with the user during Phase 1 clarification:
> - **Playable Studio Demo Tracks & Stems**: High-fidelity playable studio demo tracks (Cyberpunk Synthwave, Lofi Chillhop, Neo-Soul Pop) with real audio waveforms, playback, scrubbing, and volume control are pre-loaded into the studio audio engine.
> - **AI Backend Integration**: Pre-configured server-side Gemini 3.8 Flash pipeline for intelligent lyric structuring, prompt harmonization, and synthetic voice style prompt descriptors, paired with modular connector status indicators.
> - **Vocal Recordings Handling**: Audio captured through the "Sing & Add Music" microphone recording interface is automatically processed, given an audio wave preview, and committed directly into the "My Songs" project library with immediate playback capability.

---

### 1. Overview & Core Concept

- **What It Does**: AI Music Studio provides music creators, producers, vocalists, and songwriters with a dark-themed digital audio workstation (DAW) frontend. Users can describe desired songs to generate rich arrangement prompts, capture live vocals or upload audio stems, compose and structure genre-tailored lyrics, audition synthetic AI singers across vocal registers, customize instrumental textures, and inspect their active discography in "My Songs".
- **Target Audience & Persona**: Modern music producers, content creators, lyricists, indie artists, and mobile creators who need rapid music ideation with professional audio feedback.
- **Key Value**: Delivers an authentic music production interface with real Web Audio recording, canvas-based live waveform visualization, multi-track audio playback, and a modular AI service connector for Phase 1.

---

### 2. User Experience & Visual Design

#### A. Key User Flows
1. **Home & Dashboard Discovery**:
   - Hero banner with headline *"CREATE YOUR MUSIC"* and studio status badge.
   - 5 large interactive feature cards:
     1. **Generate Song** (Prompt generator, music styles, tempo/BPM, key signature, mood sliders).
     2. **Sing & Add Music** (Microphone permission, active audio recorder, real-time audio visualizer meter, stop, playback, retake, upload, and auto-save to My Songs).
     3. **AI Lyrics** (Genre, language, structure selector: Verse/Chorus/Bridge, intelligent lyric formatting, rhyming scheme helper).
     4. **AI Singer** (Synthetic voice gallery: pitch range, timbre, vocal character, preview audio demos, voice cloning prompt generator).
     5. **Create Instrumental** (Genre selector, mood tags, acoustic vs electronic blend, drum groove selector, BPM dial).
2. **Sing & Add Music Studio Flow**:
   - Touch-optimized studio booth with `Record`, `Stop`, `Playback`, `Upload`, and `Retake` buttons.
   - Native `navigator.mediaDevices.getUserMedia` handling with clean permission states and microphone error resilience.
   - Live visual audio level meter using Web Audio API `AudioContext` and `AnalyserNode`.
   - Audio waveform playback and direct "Auto-Save to My Songs" confirmation.
3. **Global Persistent Audio Player**:
   - Docked at the bottom of the viewport (mobile drawer / desktop dock).
   - Play/Pause toggle, previous/next track, interactive scrubbing timeline, duration timer, volume slider with mute, and synchronized animated waveform canvas.
4. **My Songs & Projects**:
   - Grid & list view of created tracks, recorded takes, and generated arrangements.
   - Track status tags (Ready, Mastered, Vocal Take, Instrumental Draft), duration, BPM, key, and quick Play trigger that feeds directly into the global player.
5. **Settings & AI Connector**:
   - Status indicators for Gemini API, ElevenLabs Voice Connector, and Stem Synthesis Engine.
   - Studio audio sample rate (44.1kHz / 48kHz), output device routing, and dark studio aesthetic accent customization.

#### B. Visual Identity & Theme
- **Theme & Colors**:
  - Deep studio obsidian background (`#08070e` / `#0d0b14`).
  - Dark-purple atmosphere (`#170f2a`, `#23153c`) with subtle violet and electric indigo gradients.
  - Accent colors: Studio Cyan/Electric Violet (`#8b5cf6`, `#6366f1`, `#06b6d4`, `#ec4899`).
  - Strict 60-30-10 color allocation ensuring high contrast and readability.
- **Typography**:
  - Display Font: `Plus Jakarta Sans` / `Outfit` for modern, bold studio headlines (`text-2xl` to `text-4xl`, tracking-tight).
  - Body Prose: Clean sans-serif with high contrast (`#f1f5f9` on dark canvas).
  - Time & Telemetry: Tabular figures (`tabular-nums font-mono`) for BPM, time elapsed, and track durations.
- **Mobile Ergonomics**:
  - Touch targets $\ge 44\text{px}$ for all playback, recording, and navigation controls.
  - Sticky bottom tab bar for mobile viewports (`Home`, `Create`, `Sing`, `Singer`, `My Songs`), auto-hiding or docking cleanly above the audio player.
  - Desktop responsive expansion to full multi-column studio dashboard (no narrow artificial constraints).

---

### 3. Key Product Decisions & Trade-Offs

- **Web Audio API Native Recording vs External Stubs**:
  - *Chosen*: Real browser `MediaRecorder` + `AudioContext` analyzer node for live vocal capture and visualization.
  - *Why*: Gives users immediate tactile validation that microphone permissions, audio levels, and playback work cleanly on mobile and desktop without requiring server audio encoding.
- **Persistent Global Audio State**:
  - *Chosen*: Unified React Context / state store managing current track, playback time, duration, volume, and active waveform buffer.
  - *Why*: Allows creators to browse lyrics, explore singer presets, or navigate project lists while a track continues uninterrupted in the player.
- **Pre-Loaded Audio Demos & Stems**:
  - *Chosen*: Bundled royalty-free Web Audio synthesizer / audio stem data for immediate testing.
  - *Why*: Guarantees instant audio feedback and waveform visualization without waiting on external network dependencies.
- **Real Server-Side Gemini API Proxy**:
  - *Chosen*: Express backend proxy endpoint (`/api/gemini/lyrics` and `/api/gemini/prompt`) using `@google/genai` and `gemini-3.8-flash`.
  - *Why*: Keeps the Gemini API key secure on the server side and powers real lyric composition and track prompt generation.

---

### 4. Technical Architecture & Data Strategy

```
┌────────────────────────────────────────────────────────────────────────┐
│                          AI Music Studio Frontend                      │
├────────────────────────────────────────────────────────────────────────┤
│  Top Navigation (Brand Wordmark, 5 Quick Nav Links, New Project CTA)   │
├────────────────────────────────────────────────────────────────────────┤
│                             View Router                                │
│   ┌───────────────┬─────────────────┬──────────────┬────────────────┐  │
│   │ DashboardView │ GenerateSongView│ SingRecordView│ AI LyricsView  │  │
│   ├───────────────┼─────────────────┼──────────────┼────────────────┤  │
│   │ AISingerView  │ InstrumentalView│ MySongsView  │ SettingsView   │  │
│   └───────────────┴─────────────────┴──────────────┴────────────────┘  │
├────────────────────────────────────────────────────────────────────────┤
│                          Audio Player Dock                             │
│    Track Art · Title/Artist · Waveform · Play/Pause · Time · Volume    │
├────────────────────────────────────────────────────────────────────────┤
│         Mobile Bottom Navigation Bar (Home · Create · Sing · Songs)     │
└────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                 Studio State & Web Audio Engine                        │
│   • AudioContext (Live Mic Analyser, Waveform Canvas, Gain Node)       │
│   • MediaRecorder API (Vocal Capture, WebM/WAV Blob, Auto-Library)     │
│   • Track Store (Local storage persistence + pre-loaded stems)         │
│   • Connector Status (Gemini API, Voice Synthesizer, Stems API)        │
└────────────────────────────────────────────────────────────────────────┘
                                    │ (fetch /api/ai/...)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        Express Full-Stack Server                       │
│   • /api/gemini/lyrics    -> Generates structured song lyrics          │
│   • /api/gemini/describe  -> Expands musical prompt & arrangement tags │
│   • /api/health           -> Service health and API connectivity check │
└────────────────────────────────────────────────────────────────────────┘
```

#### Core Data Entities
```typescript
interface SongTrack {
  id: string;
  title: string;
  artist: string;
  genre: string;
  mood: string;
  bpm: number;
  key: string;
  duration: number; // in seconds
  audioUrl: string;
  coverGradient: string;
  status: 'Ready' | 'Processing' | 'Vocal Take' | 'Draft';
  createdAt: string;
  lyrics?: string;
  waveformSample: number[];
}

interface VoiceModel {
  id: string;
  name: string;
  gender: 'Female' | 'Male' | 'Androgynous';
  style: string;
  vocalRange: string;
  recommendedGenres: string[];
  sampleAudioUrl?: string;
}
```

---

### Verification & Testing Plan
1. **Compilation & Linting**: Run `compile_applet` and `lint_applet` to guarantee clean TypeScript types and zero missing imports.
2. **Audio Player Test**: Verify Play, Pause, seek slider, volume mute/unmute, and waveform animation respond instantly.
3. **Microphone Recording Test**: Verify `navigator.mediaDevices.getUserMedia` prompt, active recording timer, live audio level visualizer, Stop, Retake, and automatic addition into `My Songs`.
4. **Feature Navigation**: Test all 8 navigation destinations (`Home`, `Create`, `Sing & Add Music`, `AI Singer`, `Instrumental`, `My Songs`, `Projects`, `Settings`) across both mobile and desktop screen sizes.
5. **AI Generation Handlers**: Verify prompt generation and lyric editing with responsive error handling and API connection status.
