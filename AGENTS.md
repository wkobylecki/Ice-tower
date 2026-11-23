# AGENTS.md — Lodowa Wieża

Gra zręcznościowa w stylu Icy Tower. Cały projekt to **jeden samowystarczalny plik** `index.html`
(HTML + CSS + canvas 2D + WebAudio). Otwiera się bezpośrednio z dysku — bez serwera, bez builda.

## Zasady

- **Język:** cały tekst w grze i w UI po polsku.
- **Zero zależności:** brak CDN, bibliotek, fontów zewnętrznych, obrazów. Grafika jest rysowana w canvasie,
  efekty i muzyka generowane proceduralnie w WebAudio (`tone`, `sfx`, `SONGS`, `scheduleStep`).
- **`music/*.mp3`** — podkłady użytkownika (`TRACKS`), folder jest w `.gitignore` (prawa autorskie — nie wrzucamy
  ich do publicznego repo). Gracz wybiera utwór w panelu 🎵 (przycisk w menu albo klawisz `T`), wybór siedzi w
  `localStorage: ice-track`. Brak pliku → gra wraca do muzyki proceduralnej. Muzyka milknie po przegranej i startuje
  od początku przy nowej grze.
- **Jeden plik:** nie rozbijaj gry na moduły ani pliki obok, chyba że użytkownik o to poprosi.
- **Nie ruszaj** `../index.html` (Tetris) i `../astro-blaster/index.html` — to osobne projekty użytkownika.
- **Git:** repo to `icy-tower/`, remote `git@github.com:wkobylecki/Ice-tower.git`, gałąź `main`.
  Push idzie przez SSH (klucz w `~/.ssh`), bez pytania o hasło.
- Zmiany rób małe i celowe: dopracowanie sterowania/tempa/muzyki, a nie przebudowa działających mechanik.

## Mapa kodu (`index.html`)

- `CFG` — fizyka: grawitacje (`gravHold/gravUp/gravDown`), `jump`, `padJump`, `maxVX`, przyspieszenia,
  `wallBounce`, `wallJumpVX/VY`.
- `ZONES` + `buildPalette()` — 5 biomów co 1000 m (LODOWE GÓRY → PUSTYNIA → MORZE → GÓRY → KOSMOS),
  kolory interpolowane przy wejściu w kolejny biom, a `drawBiome()` rysuje krajobraz (`ridge`, `drawSun`, `drawClouds`).
- Sklep: `SKIN_DATA`, `diamonds` (waluta), `openShop()`, `pickSkin()`; tabela wyników w `saveScore()`
  (`localStorage: ice-diamonds / ice-skins / ice-skin / ice-scores`).
- Pułapki: `spawnHazard()` / `updateHazards()` / `drawHazards()` — spadające sople od 50 m.
- `resize()` / `SCALE` / `VIEW_W` / `WORLD_W` / `OFF_X` / `VH` — skalowanie świata do okna; świat jest
  wyśrodkowany, marginesy wypełnia tło (`drawSideShade`).
- `makeFloor()` / `addFloorUp()` / `generateWorld()` — proceduralne piętra (jednostronne kolizje),
  sople, sprężyny, kryształy.
- `updatePlayer()` — ruch, coyote time, bufor skoku, wall jump i ślizg po ścianie, kolizje.
- Płynność: `step()` zapisuje `prevCamY` / `prevX` / `prevY`, `frame()` liczy `game.alpha`, a `draw()`
  interpoluje kamerę (przesunięcie kontekstu) i gracza (`player.rx/ry`) — brak szarpania na 120/144 Hz.
- `updateCamera()` + `scrollSpeed()` — wieża przyśpiesza liniowo z wysokością (0.10 → limit 3.4).
- `step()` (logika 60 Hz) i `frame()` (pętla z akumulatorem) + `draw()`.
- Audio: `TRACKS` / `song` (`loadTrack()`, `selectTrack()`, `songPlay()` / `songPause()`), `tone()` (SFX), `SONGS` / `musicPump()` /
  `scheduleStep()` (zapasowa muzyka per strefa, tempo rośnie z wysokością), `toggleMute()` (M),
  `toggleFullscreen()` (F). Pingwin: hitbox 31×41, rysunek skaluje się przez `p.w / 26`.

## Sprawdzanie zmian

Nie ma testów w repo. Do weryfikacji bez przeglądarki użyj tymczasowego harnessu Bun
(`$FREEBUFF_BUN`, `node` nie jest zainstalowany): wyciągnij `<script>` z `index.html`,
sprawdź składnię przez `new Function(code)`, a potem wystartuj grę na atrapach `document`,
canvasu (`ctx` jako Proxy no-op z gradientami) i WebAudio (licznik oscylatorów), wywołując
`requestAnimationFrame` ręcznie co 1000/60. Harness usuwaj po testach.

Muzyki nie da się ocenić automatem — głośność balansuj uszami (`musicFade(0.45, …)` i wolumeny
w `scheduleStep`).
