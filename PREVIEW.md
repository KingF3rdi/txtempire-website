# Vorschau — alles verbunden

## Laufende Server

| Service | URL | Status |
|---------|-----|--------|
| **Website** | http://localhost:3000 | Next.js (Proxy → API) |
| **API** | http://localhost:8000 | FastAPI |

Start: `./scripts/preview.sh`

## Verbundene Komponenten

Der Discord-Bot ([KingF3rdi/txtempire-shop-bot](https://github.com/KingF3rdi/txtempire-shop-bot)) und das Fabric-Client-Mod (`mods/txtempire-mc-watcher-1.0.8.jar`) sind eigene, separat deployte/verteilte Komponenten — kein Teil dieses Repos. Das Mod läuft im Client der Spieler und redet nur mit der HTTP-API des Bots (Port `8765`), nie direkt mit dieser Website.

```
Website (3000) ──proxy──► API (8000) ◄──/api/bot/*── discord-bot (extern)
                                                            ▲
Minecraft-Client ──Chat-Watch──► Fabric-Mod ──HTTP:8765──►┘
                                                (Bot entscheidet, Ticket = final)
```

`/api/client/*` bleibt als direkter Fallback bestehen (Zahlungsbestätigung ohne Bot-API-Key), wird vom mitgelieferten Mod aber nicht genutzt — das Mod läuft ausschließlich über den Bot.

### Gemeinsame Konfiguration

| Variable | Wert (Vorschau) | Dateien |
|----------|-----------------|---------|
| `BOT_API_KEY` | `preview-bot-api-key` | `backend/.env` und im `.env` des Discord-Bots |
| `SHOP_API_URL` | `http://localhost:8000` | `.env` des Discord-Bots |
| `SHOP_BOT_IGN` / `MC_LINK_IGN` | `TxtEmpire` | `backend/.env` bzw. `.env` des Discord-Bots |
| `FRONTEND_URL` | `http://localhost:3000` | `backend/.env` |
| `apiUrl` (Mod-Config) | `http://<bot-host>:8765` | vom Mod selbst unter dessen Config-Verzeichnis angelegt |

## Zahlungs-Flow (ingame)

1. **Website** → Warenkorb → Checkout (ingame) → **Zahlungscode** (z.B. `AB12CD`)
2. Spieler zahlt ingame; das **Fabric-Mod** liest die Chat-Nachricht und meldet sie an die API des **Discord-Bots** (Port 8765)
3. **discord-bot** leitet die Zahlung an dieses Backend weiter: `POST /api/bot/payments/confirm`
4. Backend schaltet die Packs frei; der **Bot** schließt/bestätigt das zugehörige Ticket in Discord — das ist immer der finale Schritt

## Vouch abgeben (nach Kauf)

Einmalig pro bestätigter Bestellung — per **Website** oder **Discord-DM**:

| Weg | Aktion |
|-----|--------|
| **Website** | http://localhost:3000/account → „Vouch abgeben“ |
| **Discord-DM** | `/vouch rating:5 message:Dein Text` an den Bot |
| **Discord-Server** | `/vouch` (Ticket-Käufe) |

Nach Kauf sendet der Bot optional eine **Vouch-Anfrage per DM** (wenn `DISCORD_BOT_TOKEN` gesetzt).

API:
```bash
curl -H "X-Bot-Api-Key: preview-bot-api-key" "http://localhost:8000/api/bot/vouches/pending?discord_id=DEINE_DISCORD_ID"
```

Optional: `DISCORD_VOUCH_CHANNEL_ID` in `backend/.env` — Vouches erscheinen auch im Discord-Channel.

## Discord-Bot starten (optional, eigenes Repo)

```bash
git clone https://github.com/KingF3rdi/txtempire-shop-bot
cd txtempire-shop-bot && pip install -r requirements.txt
# .env: DISCORD_TOKEN, GUILD_ID, SHOP_API_URL=http://localhost:8000, BOT_API_KEY=preview-bot-api-key
python3 bot.py
```

Das Fabric-Mod (`mods/txtempire-mc-watcher-1.0.8.jar`) wird von Spielern lokal installiert, nicht hier gestartet — es verbindet sich mit der MC-API des laufenden Discord-Bots (`apiUrl`, Standard-Port `8765`).

## Client-Mod (`/api/client/*`, direkter Fallback ohne Bot)

Für Server, die ohne den Discord-Bot direkt gegen dieses Backend sprechen wollen — vom mitgelieferten Fabric-Mod nicht verwendet:

`config/txtshop.json`:
```json
{
  "shop-api-url": "http://localhost:8000",
  "payment-recipient": "TxtEmpire"
}
```

## API testen

```bash
curl http://localhost:8000/api/health
curl http://localhost:8000/api/config/payment
curl -H "X-Bot-Api-Key: preview-bot-api-key" http://localhost:8000/api/bot/catalog
curl -H "X-Bot-Api-Key: preview-bot-api-key" http://localhost:8000/api/bot/payments/pending
```
