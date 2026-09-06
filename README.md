# TxTEmpire Shop — Website

Next.js Frontend + FastAPI Backend für den TxTEmpire Minecraft Shop.

## Struktur

| Ordner | Beschreibung |
|--------|--------------|
| `frontend/` | Next.js Website (:3000) |
| `backend/` | FastAPI REST API (:8000) |
| `scripts/preview.sh` | Lokale Vorschau starten |

## Schnellstart

```bash
# Backend
cd backend
cp .env.example .env
pip install -r requirements.txt
python3 -m uvicorn app.main:app --host 0.0.0.0 --port 8000

# Frontend (neues Terminal)
cd frontend
npm install
npm run dev
```

Website: http://localhost:3000  
API: http://localhost:8000

Details: [PREVIEW.md](./PREVIEW.md)

## Bot-Anbindung

Der Discord-Bot ([KingF3rdi/txtempire-shop-bot](https://github.com/KingF3rdi/txtempire-shop-bot)) läuft als eigenes, separates Deployment — er ist nicht Teil dieses Repos. Er ist die einzige Quelle, die einen Kauf final abschließt (Ticket im Discord-Server); diese Website bestätigt nie selbst einen Verkauf.

Setze in `backend/.env`:

```
BOT_API_KEY=dein-geheimer-key
SHOP_BOT_IGN=TxtEmpire
FRONTEND_URL=http://localhost:3000
DISCORD_BOT_TOKEN=...
DISCORD_GUILD_ID=...
```

Im `.env` des Discord-Bots dieselben Werte spiegeln: `SHOP_API_URL` = öffentliche URL dieses Backends, `BOT_API_KEY`, `DISCORD_TOKEN` = `DISCORD_BOT_TOKEN`, `GUILD_ID` = `DISCORD_GUILD_ID`, `MC_LINK_IGN` = `SHOP_BOT_IGN`.

Ablauf: Spieler verknüpfen ihren Account auf der Website (`/account` → Verknüpfungscode) und lösen ihn ingame per Chat-Befehl ein. Das Fabric-Client-Mod ([`mods/txtempire-mc-watcher-1.0.8.jar`](./mods/txtempire-mc-watcher-1.0.8.jar)) beobachtet den Chat auf diesen Code und auf Zahlungen und meldet beides an die **HTTP-API des Discord-Bots** (nicht an dieses Backend). Der Bot leitet Verknüpfung und Zahlungsbestätigung über `/api/bot/*` (mit `BOT_API_KEY`) an dieses Backend weiter und schließt das zugehörige Ticket in Discord ab.

Das Mod wird nicht über die Website verteilt — der Serverbetreiber gibt es den Spielern direkt (z.B. per Discord/Modpack) und konfiguriert dessen `apiUrl` auf den Host, auf dem der Discord-Bot läuft (Standard-Port `8765`).

## API-Überblick

| Pfad | Beschreibung |
|------|--------------|
| `/api/shop/*` | Produkte, Warenkorb, Checkout |
| `/api/bot/*` | Bot-Integration (API-Key) |
| `/api/client/*` | Direkter Fallback für Server ohne Discord-Bot (vom mitgelieferten Mod nicht genutzt) |
| `/api/user/*` | Profil, Vouches, Wunschliste |
