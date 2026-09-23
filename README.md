# HANDSON SCRAPER — AUTOMATION SETUP
### Three ways to run automatically. Pick one.

---

## ⚡ FASTEST: GitHub Actions (free, 15 min setup)

No server needed. GitHub runs the scraper every 2 hours on their computers.

**Step 1 — Create a GitHub repo:**
1. Go to github.com → New repository
2. Name it: `handson-ventures`
3. Set to Private
4. Click Create

**Step 2 — Upload your files:**
```bash
# On your computer:
cd ~/Desktop/handson
git init
git add ventures-scraper.js
git add .github/workflows/scraper.yml
git commit -m "HandsOn scraper"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/handson-ventures.git
git push -u origin main
```

**Step 3 — Done. It runs automatically.**

GitHub Actions runs the scraper every 2 hours. Go to:
`github.com/YOUR_USERNAME/handson-ventures/actions`

You'll see each run with lead counts. Download `leads.json` from the artifacts.

**Add SMS alerts (optional):**
Go to repo Settings → Secrets → New secret:
- `TWILIO_SID` — from twilio.com
- `TWILIO_TOKEN` — from twilio.com  
- `TWILIO_FROM` — your Twilio number
- `TWILIO_TO` — `+17208990383` (your cell)

Now you get a text every time CRITICAL leads come in.

**Trigger a run manually:**
Actions tab → HandsOn Lead Scraper → Run workflow → pick a source → Run

---

## 🚀 BEST: DigitalOcean ($6/mo, 20 min setup)

Live dashboard, real URL, runs forever even if your computer is off.

**Step 1 — Create a $6 droplet:**
1. Go to digitalocean.com
2. Create → Droplets
3. Choose: Ubuntu 24.04, Basic, $6/mo (1GB RAM)
4. Pick any region (Denver/SF closest)
5. Add your SSH key (or use password)
6. Click Create Droplet

**Step 2 — Upload the scraper:**
```bash
# Your droplet IP is shown in the DigitalOcean dashboard
scp ventures-scraper.js root@YOUR_DROPLET_IP:/root/handson/
```

**Step 3 — SSH in and run the deploy script:**
```bash
ssh root@YOUR_DROPLET_IP

# Create the handson folder
mkdir -p /root/handson
cd /root/handson

# Upload deploy script (or paste contents)
# Then run it:
bash deploy_digitalocean.sh
```

**That's it.** The script:
- Installs Node.js 20
- Installs PM2 (process manager)
- Configures Nginx (clean port 80 URL)
- Starts the scraper
- Sets up auto-restart on crash
- Sets up auto-start on reboot
- Opens your firewall

**After setup:**
```
Dashboard:  http://YOUR_DROPLET_IP
API:        http://YOUR_DROPLET_IP/leads
Health:     http://YOUR_DROPLET_IP/health
```

**Management commands (SSH in to run these):**
```bash
cd /root/handson

./status.sh          # see current lead count
./force_scrape.sh    # trigger a scrape right now
./logs.sh            # tail the logs

pm2 status           # is it running?
pm2 logs handson-scraper   # live log stream
pm2 restart handson-scraper  # restart
pm2 stop handson-scraper     # stop
```

**Add your API keys** (for SMS alerts, skip tracing):
```bash
nano /root/handson/.env
# Uncomment the TWILIO_ lines and add your keys
# Save: Ctrl+X → Y → Enter
# Restart: pm2 restart handson-scraper
```

---

## 🍎 EASY: Mac Background Service (free)

Runs on your Mac every 2 hours. Starts automatically when you log in.

```bash
# Download ventures-scraper.js and deploy_mac.sh
# Put both in the same folder, then:

chmod +x deploy_mac.sh
./deploy_mac.sh
```

**After setup:**
```
Dashboard:  http://localhost:3007
Leads:      ~/handson/leads.json
Logs:       ~/handson/logs/
```

**Commands:**
```bash
~/handson/status.sh    # see lead count
~/handson/run_now.sh   # force scrape now

# To stop the automation:
launchctl unload ~/Library/LaunchAgents/com.handson.scraper.plist
```

---

## CONNECT THE REACT APP TO LIVE DATA

Once the scraper is running somewhere with a public URL:

**HandsOn.jsx** — find this line near the top and update it:
```javascript
const API_BASE = 'http://YOUR_DROPLET_IP';
// or
const API_BASE = 'https://abc123.localhost.run'; // tunnel URL
```

**ContractorApp.jsx** and **AdminDashboard.jsx** — same thing.

The apps will then pull live leads from your running scraper.

---

## QUICK TUNNEL (no server needed, temporary)

Want to test right now from your laptop?

```bash
# Terminal 1 — start scraper
node ventures-scraper.js

# Terminal 2 — open tunnel to internet
ssh -R 80:localhost:3007 nokey@localhost.run
# Copy the URL it gives you (e.g. https://abc123.localhost.run)
# Paste it into API_BASE in your React apps
```

The tunnel stays open as long as the terminal is open.

---

## WHAT RUNS ON EACH SCHEDULE

```
Every 2 hours:
  ▶ Redfin SOLD last 7 days (75 markets × 2 pages = up to 52,500 listings)
  ▶ Redfin PENDING under contract (75 markets)
  ▶ Redfin ACTIVE for-sale status=1 (75 markets × 2 pages)
  ▶ Redfin COMING SOON status=131 (tier-1 markets)
  ▶ Zillow active for-sale + price drops (12 markets)
  ▶ Apartments.com new listings (rotating batch of 11 cities)
  ▶ Zillow rentals new (rotating batch)
  ▶ StorageAuctions.com lien calendar (18 markets)
  ▶ Craigslist (9 categories × rotating cities)
  ▶ Reddit (50+ subreddits)
  ▶ NOAA storm reports
  ▶ City permits (Denver + Austin open data)
  ▶ Eviction court filings
  ▶ Divorce filings (CourtListener)
  ▶ WARN Act (rotating 7 states per run)
  ▶ County deed recordings
  ▶ HUD foreclosures
  ▶ Code/HOA violations
  ▶ University housing cycles

24-hour coverage:
  All 75 Redfin markets covered every 24 hours
  All 100 apartment cities covered every 24 hours
  All 40 WARN states covered every 24 hours
```

---

*Tiger | HandsOn Ventures LLC | Denver CO*
*(720) 899-0383 | handsonmovers303@gmail.com*# handsonleads
The scraper is the foundation but here's what makes it compound: every lead that comes in and gets claimed by a contractor should automatically trigger a follow-up chain for the other trades at the same address. Someone books a mover → 30 days later, auto-generate a painting lead for the same address → 60 days later, a landscaping lead. 
