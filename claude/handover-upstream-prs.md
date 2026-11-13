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

## Stand der PRs (13.11.2026, Sitzungsende)

**Gemergt am 13.11.2026:** #210 (imageInfo-Validierung), #211 (Takeout-Duplikate), #212 (EXIF-Beschreibung), #213 (sharp bei Bedarf laden), #214 (sharp 0.35.5), #216 (Notifications von/zu anderen Modulen: URL/URLS/UPDATE_IMAGE_LIST in `notificationReceived`, IMAGE_UPDATED/FILELIST per `sendNotification`; Fixes #170, #180, #126), #217 (Symlinks beim Scannen folgen, `statSync` + `realpath`-Set gegen Schleifen; Related to #139), dazu Dependabot #197, #200, #202, #205, #206, #215. `upstream/master` = `0b66aed`; sharp `^0.35.5`, Lockfile mit `libc`-Feldern. `MMM-BackgroundSlideshow.js` ist auf master ESLint-sauber; `node_helper.js` hat weiterhin 42 alte ESLint-Fehler. #201 (Dependabot sharp 0.35.4) ist geschlossen.

**Offen:** keine PRs von Marvin. Marvins Fork-`master` war vor dem Merge von #217 synchronisiert; ggf. erneut „Sync fork“.

Frühere, bereits gemergte PRs: #208 (`avoidRepeatsAcrossCycles`), #209 (decode before display), beide am 05.11.2026. Der Maintainer merged kleine, saubere PRs zügig, mit Merge-Commits. Deshalb schließt ein `Fixes #n` in einer Commit-Message das Issue ebenfalls.

## Gepostete Issue-Kommentare (13.11.2026, von Marvin)

- **#207** (jn3va, „imageInfo not displaying“) – **Update:** jn3va hat geantwortet, die Beschreibung („Chihuly Museum Seattle WA“) steht in allen drei Feldern (EXIF `ImageDescription`, IPTC `Caption-Abstract`, XMP `Description`). #210 und #212 sind gemergt, damit sollte es gehen; er testet am Wochenende. Issue noch offen. Ursprünglicher Kommentar: zwei Ursachen erklärt (Validierung → #210; Beschreibung kam nur aus Takeout-JSON → #212). Gefragt, wo die Beschreibung steht: `exiftool -EXIF:ImageDescription -IPTC:Caption-Abstract -XMP:Description WildGlass_JN3_4408.jpg`. Sein Bild liegt in `/home/dpf/Pictures/`, kein Takeout.
- **#186:** Hinweis auf `imageInfoDateFormat` (kam mit #189), Beispiel `'MMMM D, YYYY'`, Schließen vorgeschlagen.
- **#177:** Seit #203 wird die ganze Info-Box samt Überschrift ausgeblendet, wenn nichts anzuzeigen ist. Schließen vorgeschlagen.
- **#168:** PREVIOUS funktioniert seit #182. NEXT überspringt mehrere Bilder vermutlich, weil mehrere Clients verbunden sind: Jedes Frontend schickt `BACKGROUNDSLIDESHOW_NEXT_IMAGE`, und der gemeinsame node_helper ruft jedes Mal `getNextImage()` auf (`node_helper.js` ~Z. 725). Als Frage formuliert.
- **#148:** Follow-up zu Marvins Kommentar vom 11.12.2024: #209 dekodiert vor dem Überblenden, dazu Hinweis auf `resizeImages`. Hintergrund für Rückfragen: Auf Marvins Pi lag das Ruckeln an Electrons Software-Rendering ohne GPU; geholfen haben #209, die Überblendung und 4 Raster-Threads (zuschneiden per `cropImagesToScreen` dagegen nicht, s. u.).
- **#198:** sharp-Binaries für 32-Bit-ARM brauchen glibc ≥ 2.36 (Bookworm); empfohlen: Bookworm/64-Bit, der Workaround des Reporters, #213.
- **#188:** Die Installation war laut Ausgabe erfolgreich; nachgefragt, was genau nicht geht. `brace-expansion` kommt nur über ESLint (dev), sharp-Lücke → #214, kein `npm audit fix --force`.
- **#201** (Dependabot): Hinweis, dass 0.35.4 noch betroffen ist → #214.
- **Zu #216 (entworfen, Marvin postet):** #127 (Status: NEXT/PREVIOUS/PAUSE/PLAY gehen seit #182, Rest → #216), #175 (`UPDATE_IMAGE_LIST` + MMM-ModuleScheduler funktioniert erst mit #216).
- **Runde 3 (entworfen, Marvin postet):** #139 (Symlink-Vermutung → #217, Rückfrage nach Config/Log), #26 (`excludePaths` mit Default `['@eaDir']` seit `593db46`, Schließen vorgeschlagen), #110 (Maintainer will Übersetzer-Modul; PAUSE/PLAY gehen von anderen Modulen, Schließen vorgeschlagen), #173 (HEIC: prebuilt sharp liest nur AVIF, Chromium kein HEIC → selbst kompilieren oder vorher nach JPEG konvertieren / iPhone „Maximale Kompatibilität“).
- **Bereits gelöste Issues, Schließen vorgeschlagen (entworfen, Marvin postet):** #80 und #25 (`horizontalGradient`, behoben 2021 in `823f83b`), #131 (`translations/fr.json` seit `a636e37`, `language: 'fr'`), #130 (README-Snippet existiert nicht mehr). **#164** ist Marvins eigenes Issue, behoben durch #165 → Marvin schließt es selbst.

## Nächste Schritte

1. **Antworten verfolgen** (per WebFetch auf die Issue-/PR-Seiten):
   - **#207:** jn3vas Testergebnis abwarten. Klappt es, kann er/der Maintainer schließen. Klappt es nicht: prüfen, ob er das Modul aktualisiert hat (`git pull` + `npm install`), dann Browser-Konsole/Log ansehen. Eine IPTC-/XMP-Erweiterung ist für ihn nicht nötig, weil `ImageDescription` gefüllt ist.
   - **#168, #188, #198:** Rückmeldungen der Reporter abwarten.
   - **Neue PRs:** bei Review-Kommentaren des Maintainers, auf denselben Branches im Fork (Force-Push nur nach Rücksprache mit Marvin).
2. **Branch `feature/crop-images-to-screen` im Fork: bewusst nicht eingereicht.** 3 Commits vom 05.11.2026 (gut 100 Zeilen, v. a. `node_helper.js`): Option `cropImagesToScreen` schneidet bei `resizeImages` die Bilder serverseitig per sharp auf genau `maxWidth × maxHeight` zu; der Ausschnitt richtet sich nach `backgroundPosition` (Umrechnung von CSS-Angaben wie `"center 30%"`), wie bei `backgroundSize: 'cover'`. Greift nur mit `resizeImages`, `backgroundSize: 'cover'` und ohne Hintergrund-Animation.
   **Ergebnis auf dem Pi: kein Effekt.** Der stückweise Bildaufbau lag nicht am Skalieren im Browser, sondern daran, dass Electron ohne GPU in Software zeichnet und Zwischenstände anzeigt. Geholfen haben `decode()` vor dem Anzeigen (#209), die Überblendung und 4 Raster-Threads; der Crop-Test wurde auf dem Pi zurückgenommen.
   → **Branch am 13.11.2026 gelöscht** (von Marvin über die GitHub-Oberfläche; der Git-Proxy der Cloud-Sitzung lehnt Branch-Löschen mit 403 ab). Letzter Commit, falls er doch noch gebraucht wird: `4c804e50a16741f1ca221abf4407d8e698a7963b`.
3. **Fork-`master`** ist deutlich hinter `upstream/master` (steht auf `0e2b0a7`). Nicht ohne Erlaubnis pushen; Marvin kann „Sync fork“ auf GitHub nutzen.
4. **Jetzt fällig (alle PRs bis #217 sind gemergt):** Auf dem Pi das Modul per `~/update_mm.sh` aktualisieren (siehe `claude/handover-raspi.md`). Mit #214 kommt sharp 0.35.5, also läuft auch ein `npm install`.

## Weitere Ideen (nicht begonnen, erst mit Marvin abstimmen)

- Feature-Issues geprüft, nicht gebaut: **#144** (nur Bilder der letzten x Tage) – machbar als Option nach Datei-`mtime` (node_helper hat `modified` schon), aber mtime ≠ Aufnahmedatum nach Takeout/Kopieren; EXIF-Datum hieße jede Datei beim Scan öffnen. Workaround `rclone --max-age` steht im Issue. Nutzen mittel. **#151** (leere Pause zwischen Bildern) – greift in Timer/Überblendung ein, Nutzen gering. **#171** (Sättigung/Kontrast) – Support-Frage mit Screenshots, ausgelassen.
- In `socketNotificationReceived` sind weitere Zweige toter Code (`BACKGROUNDSLIDESHOW_IMAGE_UPDATE`, `_NEXT`, `_PREVIOUS`, `_PAUSE`; der node_helper sendet sie nie). In #216 bewusst nicht angefasst, um den Diff klein zu halten.

- `readdirSync` des ganzen Ordners pro Bild in `getNextImage()` (für die Takeout-JSON) könnte gecacht werden, wenn große Ordner langsam sind (im PR-Text zu #211 erwähnt).
- Veralteter Code-Kommentar über `notificationReceived` (`MMM-BackgroundSlideshow.js` ~Z. 218) nennt noch `BACKGROUNDSLIDESHOW_PREV` statt `BACKGROUNDSLIDESHOW_PREVIOUS`; trivialer Doku-Fix.
- #138 (IPTC City/Country in imageInfo) würde zur IPTC-Erweiterung von #212 passen.
