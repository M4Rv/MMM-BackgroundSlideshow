# Handover: Upstream-PRs für MMM-BackgroundSlideshow (Runde 2)

Stand: 13.11.2026, Ende der Sitzung. Für eine neue Claude-Code-Cloud-Sitzung mit Repo `M4Rv/MMM-BackgroundSlideshow` (Fork, Claude-GitHub-App mit Schreibrechten installiert).

## Rahmen (unverändert)

- Antworten auf Deutsch, Code/Commits/PR-Texte/Issue-Kommentare auf Englisch.
- Fork-Remote `origin` = `M4Rv/MMM-BackgroundSlideshow`; Upstream = `https://github.com/darickc/MMM-BackgroundSlideshow`, Branch `master`, Stand `916fd47` (Merge PR #209). Remote anlegen: `git remote add upstream https://github.com/darickc/MMM-BackgroundSlideshow && git fetch upstream master`.
- Die Claude-App ist im Upstream **nicht** installiert: Claude pusht nur Branches in den Fork, Marvin öffnet PRs und postet Kommentare selbst. PR-Texte und Kommentare im Chat liefern.
- **Links immer auf das Upstream-Repo**, nicht auf den Fork:
  - PR öffnen: `https://github.com/darickc/MMM-BackgroundSlideshow/compare/master...M4Rv:MMM-BackgroundSlideshow:<branch>?expand=1`
  - PRs/Issues: `https://github.com/darickc/MMM-BackgroundSlideshow/pull/<n>` bzw. `/issues/<n>`
- **Upstream lesen:** Die GitHub-API/MCP deckt darickc nicht ab, `curl github.com` gibt 403. **WebFetch funktioniert** für Issue- und PR-Seiten auf github.com (nur Zusammenfassungen, keine langen Zitate).
- **Commit-Autor:** `Marvin H. <3452671+M4Rv@users.noreply.github.com>` (aus Marvins gemergten Upstream-Commits verifiziert). Per `git -c user.name=... -c user.email=... commit` setzen, die globale git-Config steht auf Claude.
- **Lint:** `npm ci`, dann `npx prettier --check <dateien>` und `npx eslint <dateien>` nur für geänderte Dateien. Vorbestehende Fehler: `node_helper.js` hat auf master **50 ESLint-Fehler**, `MMM-BackgroundSlideshow.js` einen (`console.warn` Z. 145, fällt mit #210 weg). Nicht anfassen, nur sicherstellen, dass keine neuen dazukommen (Anzahl vorher/nachher vergleichen).
- **Lockfile:** mit `npx -y npm@11 install ...` erzeugen. Das vorinstallierte npm 10 entfernt die `libc`-Felder aus `package-lock.json`.
- Kein Test-Setup im Repo: Testskripte im Scratchpad halten, nicht committen, Ergebnisse im PR-Text beschreiben.
- Vor neuen PRs prüfen, ob sie sich mit den offenen PRs konfliktfrei mergen lassen (Probe-Merge aller Branches nacheinander auf `upstream/master`). #211 und #213 kollidierten zunächst, weil beide hinter derselben Zeile in `node_helper.js` einfügten.

## Offene PRs von Marvin (alle 13.11.2026, alle sechs konfliktfrei zusammen mergebar)

| PR | Branch | Inhalt | Issue |
|---|---|---|---|
| [#210](https://github.com/darickc/MMM-BackgroundSlideshow/pull/210) | `fix/imageinfo-validation` | imageInfo-Werte gegen Whitelist prüfen statt `name`/`date`-Regex; `desc_name` umgesetzt (`desc_or_name` als Alias) | „Part of #207“ (bewusst **kein** `Fixes`, weder im PR-Text noch in der Commit-Message) |
| [#211](https://github.com/darickc/MMM-BackgroundSlideshow/pull/211) | `fix/takeout-sidecar-duplicates` | `findTakeoutJsonFile()`: Takeout-JSON exakt zuordnen, auch bei `p2200111(1).jpg` → `p2200111.jpg.supplemental-metadata(1).json` | – |
| [#212](https://github.com/darickc/MMM-BackgroundSlideshow/pull/212) | `feature/exif-description-fallback` | Ohne Takeout-Beschreibung wird EXIF `ImageDescription` genommen; UTF-8-Dekodierung (exif-js liest byteweise), Kamera-Platzhalter wie „OLYMPUS DIGITAL CAMERA“ werden verworfen, `excludeDescriptionsRegexps` gilt auch hier | #207, sobald bestätigt |
| [#213](https://github.com/darickc/MMM-BackgroundSlideshow/pull/213) | `fix/load-sharp-on-demand` | sharp erst bei `resizeImages` laden (`loadSharp()`); schlägt das fehl, wird einmal ein Fehler geloggt und das Originalbild ausgeliefert | „Related to #198“ |
| [#214](https://github.com/darickc/MMM-BackgroundSlideshow/pull/214) | `chore/bump-sharp-0.35.5` | sharp `^0.35.5` wegen hoher Sicherheitslücke (GHSA-rgj7-g3m4-5g8c, GHSA-wq5f-xc86-pv6w); nur sharp-Pakete im Lockfile | ersetzt Dependabot-PR #201 (0.35.4 ist noch betroffen) |
| [#216](https://github.com/darickc/MMM-BackgroundSlideshow/pull/216) | `fix/notifications-from-other-modules` | `BACKGROUNDSLIDESHOW_URL`, `_URLS`, `_UPDATE_IMAGE_LIST` wurden nur in `socketNotificationReceived` behandelt (kommt nur vom eigenen node_helper) → nach `notificationReceived` verschoben; `IMAGE_UPDATED` und `FILELIST` gehen jetzt per `sendNotification` an andere Module (README dokumentiert beide). Bewusst nicht wie der geschlossene #135 alles umleiten. | Fixes #170, #180, #126; erwähnt in #127, #175 |

Frühere, bereits gemergte PRs: #208 (`avoidRepeatsAcrossCycles`), #209 (decode before display), beide am 05.11.2026. Der Maintainer merged kleine, saubere PRs zügig, mit Merge-Commits. Deshalb schließt ein `Fixes #n` in einer Commit-Message das Issue ebenfalls.

## Gepostete Issue-Kommentare (13.11.2026, von Marvin)

- **#207** (jn3va, „imageInfo not displaying“): zwei Ursachen erklärt (Validierung → #210; Beschreibung kam nur aus Takeout-JSON → #212). Gefragt, wo die Beschreibung steht: `exiftool -EXIF:ImageDescription -IPTC:Caption-Abstract -XMP:Description WildGlass_JN3_4408.jpg`. Sein Bild liegt in `/home/dpf/Pictures/`, kein Takeout.
- **#186:** Hinweis auf `imageInfoDateFormat` (kam mit #189), Beispiel `'MMMM D, YYYY'`, Schließen vorgeschlagen.
- **#177:** Seit #203 wird die ganze Info-Box samt Überschrift ausgeblendet, wenn nichts anzuzeigen ist. Schließen vorgeschlagen.
- **#168:** PREVIOUS funktioniert seit #182. NEXT überspringt mehrere Bilder vermutlich, weil mehrere Clients verbunden sind: Jedes Frontend schickt `BACKGROUNDSLIDESHOW_NEXT_IMAGE`, und der gemeinsame node_helper ruft jedes Mal `getNextImage()` auf (`node_helper.js` ~Z. 725). Als Frage formuliert.
- **#148:** Follow-up zu Marvins Kommentar vom 11.12.2024: #209 dekodiert vor dem Überblenden, dazu Hinweis auf `resizeImages`. Hintergrund für Rückfragen: Auf Marvins Pi lag das Ruckeln an Electrons Software-Rendering ohne GPU; geholfen haben #209, die Überblendung und 4 Raster-Threads (zuschneiden per `cropImagesToScreen` dagegen nicht, s. u.).
- **#198:** sharp-Binaries für 32-Bit-ARM brauchen glibc ≥ 2.36 (Bookworm); empfohlen: Bookworm/64-Bit, der Workaround des Reporters, #213.
- **#188:** Die Installation war laut Ausgabe erfolgreich; nachgefragt, was genau nicht geht. `brace-expansion` kommt nur über ESLint (dev), sharp-Lücke → #214, kein `npm audit fix --force`.
- **#201** (Dependabot): Hinweis, dass 0.35.4 noch betroffen ist → #214.
- **Zu #216 (entworfen, Marvin postet):** #127 (Status: NEXT/PREVIOUS/PAUSE/PLAY gehen seit #182, Rest → #216), #175 (`UPDATE_IMAGE_LIST` + MMM-ModuleScheduler funktioniert erst mit #216).
- **Bereits gelöste Issues, Schließen vorgeschlagen (entworfen, Marvin postet):** #80 und #25 (`horizontalGradient`, behoben 2021 in `823f83b`), #131 (`translations/fr.json` seit `a636e37`, `language: 'fr'`), #130 (README-Snippet existiert nicht mehr). **#164** ist Marvins eigenes Issue, behoben durch #165 → Marvin schließt es selbst.

## Nächste Schritte

1. **Antworten verfolgen** (per WebFetch auf die Issue-/PR-Seiten):
   - **#207:** Steht jn3vas Beschreibung in `ImageDescription` → Marvin trägt `Fixes #207` in den Text von #212 ein. Steht sie nur in **IPTC** (`Caption-Abstract`) → #212 erweitern; exif-js liest IPTC schon mit (`EXIF.getIptcData(img)` bzw. `img.iptcdata.caption`, ebenfalls byteweise, also dieselbe UTF-8-Dekodierung). **XMP** wäre aufwendiger (`EXIF.enableXmp()`, XML parsen) → erst mit Marvin abstimmen.
   - **#168, #188, #198:** Rückmeldungen der Reporter abwarten.
   - **Review-Kommentare des Maintainers** zu #210–#214 umsetzen, auf denselben Branches im Fork (Force-Push nur nach Rücksprache mit Marvin).
2. **Branch `feature/crop-images-to-screen` im Fork: bewusst nicht eingereicht.** 3 Commits vom 05.11.2026 (gut 100 Zeilen, v. a. `node_helper.js`): Option `cropImagesToScreen` schneidet bei `resizeImages` die Bilder serverseitig per sharp auf genau `maxWidth × maxHeight` zu; der Ausschnitt richtet sich nach `backgroundPosition` (Umrechnung von CSS-Angaben wie `"center 30%"`), wie bei `backgroundSize: 'cover'`. Greift nur mit `resizeImages`, `backgroundSize: 'cover'` und ohne Hintergrund-Animation.
   **Ergebnis auf dem Pi: kein Effekt.** Der stückweise Bildaufbau lag nicht am Skalieren im Browser, sondern daran, dass Electron ohne GPU in Software zeichnet und Zwischenstände anzeigt. Geholfen haben `decode()` vor dem Anzeigen (#209), die Überblendung und 4 Raster-Threads; der Crop-Test wurde auf dem Pi zurückgenommen.
   → **Branch am 13.11.2026 gelöscht** (von Marvin über die GitHub-Oberfläche; der Git-Proxy der Cloud-Sitzung lehnt Branch-Löschen mit 403 ab). Letzter Commit, falls er doch noch gebraucht wird: `4c804e50a16741f1ca221abf4407d8e698a7963b`.
3. **Fork-`master`** ist 6 Commits hinter `upstream/master` (steht auf `0e2b0a7`). Nicht ohne Erlaubnis pushen; Marvin kann „Sync fork“ auf GitHub nutzen.
4. **Nach dem Merge:** Auf dem Pi das Modul per `~/update_mm.sh` aktualisieren (siehe `claude/handover-raspi.md`). Mit #214 kommt sharp 0.35.5, also läuft auch ein `npm install`.

## Weitere Ideen (nicht begonnen, erst mit Marvin abstimmen)

- Noch nicht geprüfte Feature-Issues: #151 (Pause zwischen Bildern), #144 (nur Bilder der letzten x Tage), #110 (USER_PRESENCE), #26 (versteckte Ordner ausschließen).
- In `socketNotificationReceived` sind weitere Zweige toter Code (`BACKGROUNDSLIDESHOW_IMAGE_UPDATE`, `_NEXT`, `_PREVIOUS`, `_PAUSE`; der node_helper sendet sie nie). In #216 bewusst nicht angefasst, um den Diff klein zu halten.

- `readdirSync` des ganzen Ordners pro Bild in `getNextImage()` (für die Takeout-JSON) könnte gecacht werden, wenn große Ordner langsam sind (im PR-Text zu #211 erwähnt).
- Veralteter Code-Kommentar über `notificationReceived` (`MMM-BackgroundSlideshow.js` ~Z. 218) nennt noch `BACKGROUNDSLIDESHOW_PREV` statt `BACKGROUNDSLIDESHOW_PREVIOUS`; trivialer Doku-Fix.
- #138 (IPTC City/Country in imageInfo) würde zur IPTC-Erweiterung von #212 passen.
