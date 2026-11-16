# Minigame Mex – Netzwerk (1.21.11)

## Starten / Stoppen
```bash
./start-all.sh        # Datenbank + Lobby + Proxy (tmux-Session "mex") – Spielmodi starten automatisch
./stop-all.sh         # alles sauber herunterfahren (DB läuft weiter)
tmux attach -t mex    # Konsolen ansehen: Strg+B 0/1/2 wechseln, Strg+B D lösen
LOBBY_MEM=1G FASTSOFT_MEM=2G PROXY_MEM=512M ./start-all.sh   # RAM überschreiben
```
Verbinden: `localhost:25565` (von außen nur Port 25565 freigeben).

## Server
| Server | Software | Port | Plugins |
|---|---|---|---|
| proxy | Velocity 4.3.0 | 25565 | LuckPerms, TAB, VelocityScoreboardAPI, AdvancedServerList, LibertyBans 1.2.0-M1, Maintenance, ViaVersion, ViaBackwards, SignedVelocity, velocity-hub, **MexAutoStart** |
| lobby | Paper 1.21.11 | 127.0.0.1:30066 | LuckPerms, EssentialsX (+Chat, +Spawn), VaultUnlocked, PlaceholderAPI, FAWE, WorldGuard, GrimAC, TAB-Bridge, SignedVelocity |
| karaoke | Paper 1.21.11 | 127.0.0.1:30069 | Basis-Plugins + Simple Voice Chat + **Karaoke** – schläft bei Leere |
| minelayer | Paper 1.21.11 | 127.0.0.1:30068 | Basis-Plugins + **MineLayer** (eigenes 2-Spieler-Minenspiel) – schläft bei Leere |
| splatoon | Paper 1.21.11 | 127.0.0.1:30071 | Basis-Plugins + **Splatoon** (Revierkampf, 2–8 Spieler) – schläft bei Leere |
| ledfloor | Paper 1.21.11 | 127.0.0.1:30070 | Basis-Plugins + **LedFloor** (Endlos-Spiel, 1–3 Spieler) – schläft bei Leere |
| fastsoft | Paper 1.21.11 | 127.0.0.1:30067 | wie Lobby ohne WorldGuard + **FastSoft** (eigenes Airsoft-Plugin) |
| database | MariaDB 11.4 (Docker) | 127.0.0.1:3306 | Datenbanken `luckperms`, `libertybans`, `fastsoft`, `minelayer`, `karaoke`, `ledfloor` – Passwörter in `database/*.env` |

## Ränge
Definiert in `tools/ranks.py` (Farben, Gewicht, Vererbung, Rechte). Nach Änderungen:
```bash
python3 tools/ranks.py --apply   # sendet alle LuckPerms-Befehle an die Lobby-Konsole
```
Owner › Admin › Dev › Mod › Supporter › Builder › Content › Premium+ › Premium › Spieler

Rang vergeben: `lp user <name> parent set <rang>` (Lobby/fastsoft) bzw. `lpv user ...` (Proxy).

## Wichtige Befehle
- `/hub`, `/lobby` – zurück zur Lobby
- `/ban`, `/mute`, `/warn`, `/kick`, `/history` – LibertyBans (netzwerkweit)
- `/maintenance on|off` – Wartungsmodus (Staff ab Supporter + Builder kommen rein)
- `/tab reload`, `/asl reload` – TAB / Serverliste neu laden (Proxy)

## Lobby
Flache Welt, Adventure, Peaceful, WorldGuard `__global__` schützt alles (Builder+ haben Bypass).
Spawn setzen: ingame `/setspawn` an der gewünschten Stelle.
Gamerules heißen ab 1.21.11 snake_case, z. B. `gamerule advance_time false`.

## Fast Soft (Airsoft-FFA)
Quellcode: `plugins-src/FastSoft/` – bauen + testen + deployen: `plugins-src/FastSoft/build.sh`, dann `./restart-server.sh fastsoft`.
Configs: `fastsoft/plugins/FastSoft/` (`config.yml`, `weapons.yml`, `cosmetics.yml`, `gadgets.yml`, `maps.yml`) – nach Änderungen `./restart-server.sh fastsoft`.

| Befehl | Wer |
|---|---|
| `/fs stats [spieler]`, `/fs shop`, `/fs loadout` | alle |
| `/fs setwait [x y z [welt]]` – Warte-Spawn | Dev+ |
| `/fs map create <name>` · `addspawn <name>` · `setvoid <name> <y>` · `delete` · `list` | Dev+ |
| `/fs start` / `/fs stop` | Dev+ |
| `/fs coins <spieler> add\|set\|take <betrag>` | Dev+ |

Eigene Map: im Creative bauen (Builder+ dürfen auf fastsoft bauen), dann `/fs map create <name>`,
an mind. 2 Stellen `/fs map addspawn <name>`, `/fs map setvoid <name> <y>` (unter dieser Höhe ist man raus).
Test-Map `test`: schwebende Arena über dem Spawn (Y=150).
Resource-Pack für eigene Modelle: siehe `plugins-src/FastSoft/resourcepack/README.md`.

## Server schlafen lassen (MexAutoStart)
Fast Soft, MineLayer, Karaoke und LED Floor laufen nur, wenn jemand spielt. Wer einen schlafenden Modus im Kompass wählt, bleibt kurz
in der Lobby ("startet… ~20 s") und wird automatisch verbunden. Nach **5 Minuten ohne Spieler** fährt der Proxy
den Server wieder herunter.
- Status: `/autostart` (Dev+) bzw. in der Proxy-Konsole `autostart`
- Config: `proxy/plugins/mexautostart/config.yml` (Startbefehl, RAM, `idle-minutes`) – danach Proxy neu starten
- Manuell starten/neu starten: `./restart-server.sh fastsoft` (wird trotzdem nach Leerlauf gestoppt)

## MineLayer (2 Spieler)
Join über den Kompass → automatisch in der Warteschlange → bei 2 Spielern startet ein Duell in einer freien Arena (4 Arenen).
1. **Minen legen (60 s):** Rechtsklick auf 3 Blöcke im Feld unter dir (das Feld des Gegners). Nochmal klicken = entfernen.
2. **Graben:** abwechselnd 1 Block im eigenen Feld (Linksklick, 20 s pro Zug). Mine = −1 ❤. Wer 0 ❤ hat, verliert.

| Befehl | Wer |
|---|---|
| `/ml stats [spieler]` | alle |
| `/ml arenas` · `/ml stop <nr>` · `/ml setlobby [x y z]` | Dev+ |

Quellcode: `plugins-src/MineLayer/` und `plugins-src/MexAutoStart/` (je `build.sh`).

## Admin-Werkzeuge
| Befehl | Wo | Was |
|---|---|---|
| `/move_game <Spieler\|all\|here> <server>` | überall (Proxy) | Spieler auf einen anderen Server schicken; schlafende Server werden geweckt. `here` = alle auf deinem Server. Recht `mex.move` (Dev+) |
| `/bot spawn [Name]` · `/bot remove <Name\|all>` · `/bot list` · `/bot tp <Name>` | jeder Server | AFK-Testbot (Fake-Spieler ohne Verbindung) an deiner Position – zählt für die Spiele als Spieler, tut aber nichts. Nur auf dem Server, wo er gespawnt wurde; verschwindet beim Server-Stopp. Recht `mexbot.admin` (Dev+) |

Test-/Debug-Befehle für die Ende-Shows (Admin): `/sp end` (Splatoon-Match sofort regulär beenden), `/sp special fill|use <Spieler> [TYP]`,
`/ml boom <arena>` (MineLayer: Spieler A verliert wie durch die letzte Mine → Explosion), `/fs debugkill <Schütze> <Opfer>` (zählt als echter Kill!),
`/fs killcam <Spieler>` (letzten Kill-Clip zeigen).

Bots haben Offline-UUIDs (Version 3) und erscheinen deshalb nicht in den Lobby-Leaderboards.

## Lobby-Stats (MexStats)
In der Lobby: **Buch „Statistik“** (Hotbar-Slot 7) bzw. `/stats [spieler]` öffnet ein Menü mit Rang, Wert, Platz #n von m,
Fortschritt zum nächsten Rang und weiteren Werten pro Modus. Klick auf einen Modus = Top 10 im Chat.
Vier **Leaderboard-Tafeln** (Top 10) stehen an den Wegen zu den Toren bzw. am Südweg (LED Floor), Aktualisierung alle 60 s.

| Modus | Rang nach | Bronze | Silber | Gold | Diamant | Meister |
|---|---|---|---|---|---|---|
| Fast Soft | Kills | 10 | 50 | 150 | 400 | 1000 |
| MineLayer | Siege | 3 | 10 | 25 | 60 | 150 |
| Karaoke | Siege | 1 | 5 | 15 | 40 | 100 |
| LED Floor | bestes Level | 3 | 8 | 15 | 25 | 40 |

| Befehl | Wer |
|---|---|
| `/stats [spieler]` | alle |
| `/mexstats board set <modus>` (Tafel an deine Position) · `board remove <modus>` · `reload` · `status` | Dev+ |

Platzhalter (PlaceholderAPI, Lobby): `%mexstats_rank_<modus>%` (farbig), `%mexstats_rankplain_<modus>%`,
`%mexstats_value_<modus>%`, `%mexstats_place_<modus>%` – Modi: `fastsoft`, `minelayer`, `karaoke`, `ledfloor`.
Schwellen/Positionen: `lobby/plugins/MexStats/config.yml`. Quellcode: `plugins-src/MexStats/`.

## Splatoon (Revierkampf, 2–8 Spieler)
Kompass → Warteraum → Waffe wählen (Hotbar 1–3) → ab 2 Spielern 30 s Countdown (bei 8 Spielern 5 s). Teams Pink gegen Türkis
werden ausgelost (kleineres Team: schneller nachfüllen und respawnen). 3 Minuten: Wer am Ende mehr **Boden** gefärbt hat, gewinnt.
- **Hauptwaffen** (Hotbar 1–6 im Warteraum): **Kleckser** (halten = Dauerfeuer), **Roller** (halten = Bahn, tippen = Schleuder), **Konzentrator** (laden, loslassen), **Pinsel** (halten = schnell malen, tippen = Kleckse), **Schwapper** (Schwall im Bogen), **Splatling** (laden, dann Dauerfeuer).
- **Nebenwaffe** (Slot 8 im Warteraum umschalten, im Match Slot 2, alle 8 s): je Hauptwaffe zwei zur Wahl aus **Klecksbombe**, **Sprinkler** (färbt 15 s von selbst), **Tintenmine** (für Gegner unsichtbar), **Spritzwand** (Glaswand 6 s).
- **Map-Objekte:** Tintenfässer (vollschießen → Explosion), Kupfer-Deckung (zerbricht), Kupfergitter (Tintenfische/Farbe fallen durch), Schienen (als Tintenfisch entlangrasen), Schleimblock-Federn, Schwämme (wachsen in eigener Farbe). Eigene Objekte im Creative als diese Blöcke setzen.
- **Sprinten auf eigener Farbe** = Tintenfisch: unsichtbar, sehr schnell, springt höher; gegen eigenfarbige Wände sprinten = hochklettern.
- **Schleichen auf eigener Farbe** = Tinte schnell aufladen (30 %/s, du stehst sichtbar still). Unter 20 % blinkt die Tintenleiste.
- Gegnerfarbe bremst und kostet Leben. 100 HP (1 Herz = 10), Heilen nach 2 s ohne Treffer. Tinte = Erfahrungsleiste.
- Maps: Hafen (x=1000) und Skatepark (x=2000), abwechselnd. Nach jedem Match (und nach einem Absturz) wird die Farbe zurückgesetzt.

| Befehl | Wer |
|---|---|
| `/sp stats [spieler]` | alle |
| `/sp start` · `/sp stop` · `/sp map <hafen\|skatepark>` · `/sp paintest` · `/sp resetpaint` | Dev+ |

Maps neu bauen: `mexbuild sp-wait`, `sp-hafen`, `sp-skatepark` (Splatoon-Konsole). Quellcode: `plugins-src/Splatoon/`.

## LED Floor (1–3 Spieler)
Join über den Kompass → Warteraum → Countdown 20 s (bei 3 Spielern 5 s). Jeder spielt auf seinem **eigenen** 12×12-Boden, alle bekommen dieselben Level.
- **Gelb** betreten → **schwarz**, schwarz betreten → wieder gelb. Alle Felder schwarz = nächstes Level (45 s Zeit, vorher 3-2-1).
- **Rote** Felder (wandern Feld für Feld über den Boden) betreten oder Zeit ablaufen = −1 ♥, Level startet neu. Springen und **grüne** Felder (im Boden) sind sicher.
- 3 Leben pro Run. 0 ♥ = Zuschauer. Wer das höchste Level schafft (bei Gleichstand: schneller), gewinnt.
- Level 1–10 sind fest, ab 11 erzeugt ein Generator immer schnellere Level mit mehr Roten und sicheren Feldern.

| Befehl | Wer |
|---|---|
| `/lf stats [spieler]` | alle |
| `/lf start [level]` · `/lf stop` · `/lf level <n> [boden]` · `/lf clear [boden]` | Dev+ |

Werte (Zeit, Leben, Countdown): `ledfloor/plugins/LedFloor/config.yml`. Level 1–10: `plugins-src/LedFloor/src/main/java/de/mex/ledfloor/game/Levels.java`.
Quellcode: `plugins-src/LedFloor/` (`build.sh`).

## Karaoke (Simple Voice Chat nötig)
**Neuen Song hinzufügen (nur Admin, im Terminal):** `./tools/karaoke-song.sh` → YouTube oder Spotify wählen, Link einfügen.
Das Skript lädt die MP3 nach `karaoke-songs/`, startet ggf. den Karaoke-Server und wandelt den Song um (`karaoke reload`).
Spotify-Links: Titel + Künstler werden auf YouTube gesucht (Bestätigung j/n). Braucht ein aktuelles `yt-dlp`.

Nur Spieler mit dem Mod **Simple Voice Chat** kommen rein (sonst nach 10 s zurück in die Lobby).
Voice läuft über den Proxy (UDP 25565), Spieler bekommen die Tunnel-Adresse `vivo-save.tun.ply.gg:24530`
(`proxy/plugins/voicechat/voicechat-proxy.properties`). Der Playit-Tunnel muss **UDP → localhost:25565** sein.

**Lieder hinzufügen:** MP3s (mit Gesang) nach `karaoke-songs/` legen → ingame/Konsole `/karaoke reload`.
Neue Lieder werden per KI (Demucs) entsungen und gecacht (ca. 20 s–2 min pro Lied, bis ~2 GB RAM).
Optional Liedtext: gleichnamige `.lrc`-Datei neben die MP3 (wirkt sofort, ohne Reload).

**Spielablauf:** Ab 2 Spielern mit Voice Chat startet ein 60-s-Countdown. Jeder Teilnehmer singt genau einmal
(zufällige Reihenfolge, max. 90 s). Das Publikum bewertet mit den Sternen in der Hotbar (1–5, Rechtsklick).
Punkte = Ø Sterne × 20. Am Ende: Siegerehrung auf dem Podest + Statistik (DB `karaoke`).
Musiklautstärke: `music-volume` in `karaoke/plugins/Karaoke/config.yml` (Standard 0.6).

| Befehl | Wer |
|---|---|
| `/karaoke voteskip` – Mehrheit des Publikums beendet den Auftritt | alle |
| `/karaoke stop` – eigenen Auftritt beenden · `/karaoke songs` | alle |
| `/karaoke stats [spieler]` · `/karaoke top` | alle |
| `/karaoke start` (Countdown überspringen) · `/karaoke skip` · `/karaoke reload` | Mod+ |

Quellcode: `plugins-src/Karaoke/` · Demucs-Umgebung: `tools/karaoke-venv/`.

## Maps (MexBuilder)
Alle Maps werden per Code gebaut (`plugins-src/MexBuilder/`), reproduzierbar und jederzeit neu baubar.
Befehl in der jeweiligen Server-Konsole: `mexbuild <ziel>` (nur OP/Konsole).

| Ziel | Server | Was |
|---|---|---|
| `lobby` | lobby | Schwebende Insel (Y=100): Brunnen-Platz, 3 Modus-Tore mit Hologrammen, Kirschbäume, Aussichtspunkt. Setzt Welt- + Essentials-Spawn. |
| `fs-wait` | fastsoft | Warteraum-Insel (Y=180) mit Zelten, Schießstand; setzt `/fs setwait`. Entfernt die alte Test-Arena. |
| `fs-maps` | fastsoft | 5 Maps (Boden Y=200): `lagerhalle` (x=600), `dschungel` (1200), `wuestendorf` (1800), `frostbasis` (2400), `neonstadt` (3000) – registriert inkl. Spawns + Void-Höhe |
| `ml-wait` | minelayer | Warte-Insel mit Minenfeld-Deko (Y=100) |
| `lf-wait` | ledfloor | Dunkler Warteraum um 0/100/0 mit Regel-Tafel |
| `lf-arena` | ledfloor | Saal mit 3 Böden (x=1000/1016/1032, Y=100), Startstegen, Barrieren, Zuschauer-Steg (Y=106) |

Der Karaoke-Konzertsaal wird vom Karaoke-Plugin selbst gebaut (`HallBuilder`, Versionsmarke Lodestone unter 0/99/0 – entfernen = Neubau beim nächsten Start).
Lobby: wer von der Insel fällt, wird zum Spawn zurückgesetzt (MexLobby).


## Texturenpaket
Alle Waffen/Gadgets/Sub-Waffen/Mikrofon haben eigene 3D-Modelle (Pack ist Pflicht, Minecraft ≥ 1.21.4).
Neu bauen und ausliefern: `tools/pack.sh`. Proxy-Konsole: `mexpack info`, `mexpack reload`. Details: `resourcepack/README.md`.


## Zuschauen
`/spectate <Name>` – von jedem Server aus einem Spieler zuschauen (Ego-Sicht, Shift = frei fliegen, folgt bei Serverwechsel). `/spectate` – beenden (zurück in die Lobby). Der Beobachtete bekommt eine Nachricht.


## Karaoke-Mikrofon
Wer dran ist, bekommt automatisch das Mikro (Slot 1): **Rechtsklick** = Mikro an/aus (an = normale Stimme stumm, Gesang aus den 6 Lautsprechern), **Mausrad** = Stimme lauter/leiser (50–200 %), **Linksklick** = sich selbst hören. Musik kommt als Surround aus allen Lautsprechern.


## Freunde & Nachrichten
`/friends` (oder Kopf „Freunde“ in der Lobby, Slot 3) öffnet das Menü. Befehle: `/friend add|accept|deny|remove|jump <Name>`, `/friend list`, `/msg <Name> <Text>`, `/r <Text>`. Einstellungen im Menü: Anfragen (alle/niemand), Nachrichten (alle/Freunde/niemand), Online-Meldungen.


## Party
`/party invite <Name>` (Einladung 60 s, klickbar), `/party accept|deny <Name>`, `/party leave`, `/party kick|promote <Name>` und `/party disband` (Leiter), `/party list`, Party-Chat `/p <Text>`. Max. 8. Die Party folgt dem Leiter auf jeden Server; in Splatoon bleibt sie im selben Team (bis 4), in MineLayer spielen Mitglieder gegeneinander, in LED Floor kommt sie in dieselbe Runde. Menü: `/friends` → Party.
# Mingame-Mex
