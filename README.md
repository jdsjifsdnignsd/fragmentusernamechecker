# Fragment Username Checker
## Vibecoded so don't question some parts of the code (i abandoned using proxies but it PROBABLY works)

A small Node.js script that scans potential Fragment (fragment.com) usernames and notifies you via Telegram when a username is listed for sale/auction and its USD value (based on live TON price) is below a configured threshold.

This README explains how to configure, run, and organize the project.

---

## Features

- Fetches live TON -> USD rate from CoinGecko.
- Checks Fragment pages for sale/auction indicators.
- Optionally checks whether a Telegram account exists for the username.
- Posts notifications to a Telegram chat with inline buttons (Fragment link, and Telegram link if the account exists).
- Saves valid matches into `matching.txt`.

---

## Requirements

- Node.js >= 14
- npm (or yarn)
- Internet access (for CoinGecko, Fragment, Telegram API)
- (Optional) Proxies if you want to rotate requests

Dependencies:
- axios
- jsdom

---

## Quick install

1. Clone the repo (or add files to your project folder):
   git clone https://github.com/jdsjifsdnignsd/fragmentusernamechecker.git
2. Install dependencies:
   npm install axios jsdom

---

## Suggested folder structure

- fragmentusernamechecker/
  - README.md
  - package.json
  - index.js            <-- main script (your checker code)
  - input.txt           <-- newline-separated candidate usernames
  - matching.txt        <-- script appends matches here
  - .env                <-- (optional) environment file (DO NOT commit secrets)
  - /logs               <-- (optional) store logs or rotate output

Notes:
- Name the main script `index.js` (or keep your current filename); the README assumes `index.js`.
- Ensure `input.txt` is UTF-8 and contains one username per line.

---

## Configuration

Open the script and locate the configuration constants at the top. Key values to set:

- TELEGRAM_TOKEN — your bot token from BotFather (format: 123456:ABC-DEF...).
- TELEGRAM_CHAT_ID — the chat id (group or user) where alerts are sent.
- USD_FILTER_LIMIT — maximum USD value to accept (e.g., 10000).
- POST_DELAY — milliseconds to wait between requests (helps avoid rate limits). Default in the script: `4000`.
- MIN_USERNAME_LENGTH / MAX_USERNAME_LENGTH — filter by username length.
- PROXIES — array of proxy definitions if you want to route requests (script currently doesn't actively rotate proxies; you can extend it).
- USER_AGENTS — list of user agents (script currently uses the first entry).

Important: Do not commit TELEGRAM_TOKEN or other secrets into source control. Use environment variables or a `.env` file and a library like `dotenv` to keep them out of VCS.

Example: using environment variables (pseudo-steps)
1. Install dotenv (optional):
   npm install dotenv
2. Create `.env`:
   TELEGRAM_TOKEN=123456:ABC-DEF...
   TELEGRAM_CHAT_ID=-1001234567890
3. In your script, require dotenv at the top and use process.env.*:
   require('dotenv').config();
   const TELEGRAM_TOKEN = process.env.TELEGRAM_TOKEN;

---

## Input / Output

- input.txt — list of usernames (one per line). Example:
- matching.txt — appended lines of successful matches. Format used in the script:
username | {price} TON | ${usd value}

---

## How to run

1. Ensure dependencies are installed:
 npm install

2. Add candidate usernames to `input.txt`.

3. Make sure `TELEGRAM_TOKEN` and `TELEGRAM_CHAT_ID` are set in the script or environment.

4. Run:
 node index.js

The script will:
- fetch TON/USD rate,
- iterate usernames from `input.txt`,
- check fragment.com pages for sale/auction wording,
- if available and USD <= USD_FILTER_LIMIT, append result to `matching.txt` and send a Telegram alert.

---

## How it works (brief)

- The script uses axios to fetch the Fragment username page and jsdom to parse HTML.
- It looks for known availability keywords on the page to decide whether the username has sale/auction content.
- If a username is marked as available (based on those keywords), the script attempts to parse the displayed TON price from a `.table-cell-value.tm-value` element.
- The TON price is multiplied by a live TON->USD rate from CoinGecko. If the resulting USD value is under the configured USD_FILTER_LIMIT, the script logs, saves, and notifies via Telegram.
- Before adding the Telegram "Open Telegram" button to alerts, the script checks `https://t.me/{username}` to see if the account appears to be assigned.

---

## Tuning and best practices

- POST_DELAY: Increase to avoid being rate-limited or blocked by fragment.com.
- USER_AGENTS: Rotate multiple user agents and, optionally, configure proxy rotation to reduce request blocking.
- Proxies: If you add support for proxies, ensure you handle proxy failures and rotate indices safely.
- Parsing: The script conservatively searches page text for availability keywords — if Fragment changes its page structure, keyword list or selectors may need updates.
- Logging: Consider writing more detailed logs (timestamps, HTTP status codes, response snippets) into `/logs`.

---

## Troubleshooting

- No Telegram messages:
- Verify TELEGRAM_TOKEN and TELEGRAM_CHAT_ID are correct.
- Test the token with: https://api.telegram.org/bot<token>/getMe
- Check script logs for "Telegram Error" messages.

- Script returns no matches:
- Confirm `input.txt` contains candidate usernames (one-per-line).
- Confirm the AVAILABILITY_KEYWORDS in the script cover current Fragment phrasing.
- Check network access to fragment.com and CoinGecko.

- Script exits early / crashes:
- Ensure Node version is recent.
- Run `node index.js` and inspect console errors.
- Add try/catch or more verbose logging if needed.

---

## Security & ethical notes

- Do not scrape aggressively — respect site terms of use.
- Do not publish or commit tokens or credentials to public repositories.
- Use reasonable delays and consider obtaining permission if required for large scans.

---

## Example improvements you may want to add

- Use dotenv and environment variables for secrets.
- Add proxy support and rotation.
- Implement better HTML parsing and fallback selectors.
- Add concurrency control and exponential backoff on failures.
- Add retries for transient network errors.
- Add tests and CI checks.

---

## License

MIT
