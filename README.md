# DEDDIE Outage Watch — Koutsounari / Ferma (Ierapetra)

Checks DEDDIE's planned power outage list for the Municipality of Ierapetra
(Prefecture of Lasithi) and emails + WhatsApps you when a new outage
mentioning **Koutsounari** or **Ferma** shows up. Runs automatically every
2 hours via GitHub Actions — no server needed.

## 1. Create the repo

1. Go to github.com → **New repository** (can be private).
2. Upload all the files from this folder, keeping the folder structure
   (the `.github/workflows/check_outages.yml` file must stay in that exact
   path).

## 2. Set up email sending (Gmail)

1. Use a Gmail account (a new free one is fine — don't reuse a sensitive
   account if you'd rather not).
2. Turn on 2-Step Verification: myaccount.google.com/security
3. Create an **App Password**: myaccount.google.com/apppasswords
   → choose "Mail" → generate → copy the 16-character password.

## 3. Set up WhatsApp alerts (CallMeBot — free)

1. Save this number in your phone contacts: **+34 644 84 71 66**
2. Send it this WhatsApp message: `I allow callmebot to send me messages`
3. You'll get a reply with your personal **API key** (a number).
4. Your "phone" value is your WhatsApp number in international format,
   digits only, no `+` or spaces (e.g. `306912345678` for a Greek number).

## 4. Add GitHub Secrets

In your repo: **Settings → Secrets and variables → Actions → New repository
secret**. Add each of these:

| Secret name          | Value                                      |
|-----------------------|---------------------------------------------|
| `EMAIL_FROM`          | the Gmail address you're sending from       |
| `EMAIL_APP_PASSWORD`  | the 16-character app password from step 2   |
| `EMAIL_TO`            | the email address you want alerts sent to   |
| `CALLMEBOT_PHONE`     | your WhatsApp number (digits only)          |
| `CALLMEBOT_APIKEY`    | the API key CallMeBot sent you              |

## 5. Test it

Go to the **Actions** tab → "Check DEDDIE Outages" → **Run workflow** to
trigger it manually. Check the run's logs — with `--debug` on, it prints
every outage row it found for Ierapetra and which ones matched your
keywords, so you can confirm it's working even before a real outage is
posted for your area.

## Adjusting things

- **Change the area keywords**: add a repo secret/variable named
  `AREA_KEYWORDS` with a comma-separated list, e.g. `Κουτσουνάρι,Φέρμα,Γρα
  Λυγιά` if you want to widen or narrow the match. The script strips Greek
  accents automatically, so accented/unaccented spellings both match.
- **Change how often it checks**: edit the `cron` line in
  `.github/workflows/check_outages.yml`. `0 */2 * * *` = every 2 hours.
  `0 6,18 * * *` = twice a day (6am and 6pm UTC).
- **First run**: it will only notify about outages that appear *after*
  you start it, since it treats everything present at first run as
  "already seen" once it runs once. If you want it to alert on anything
  already posted right now, delete the contents of `seen_outages.json`
  (leave it as `[]`) before the very first run — it's already set up that
  way by default.

## A note on reliability

DEDDIE's site (https://siteapps.deddie.gr/outages2public) is not a public
API — this script parses their HTML page, using the internal IDs for
Λασιθίου (36) and Δήμος Ιεράπετρας (735). If DEDDIE redesigns their site,
the script may need small updates to keep working. The debug output in the
Actions log ("Fetched N total rows...") is the first thing to check if
notifications stop coming — if that number drops to 0 unexpectedly, the
page structure likely changed and the table-parsing logic needs a look.
