# Build log: ChillChain

Team Rocket (Rahul, Anmol, Anirudh, Parth) · Prompt-ae-thon 2026, CodeChef BVCOE Chapter × Horizon

Live: https://chillchain.chillchain.workers.dev

---

## How it started

In the first round of this hackathon we built **FreshGrid**: an app for staple-crop farmers (wheat, pulses, vegetables) to grade produce, book cold storage and sell to buyers with escrow.

**The problem with FreshGrid (judges' feedback):** mandis don't work that way for staple crops. Farmers sell through commission agents who also lend them money, prices come from MSP or the agent, and at ₹15–25/kg there is no margin to pay for grading, cold rooms or escrow. They liked the tech but said it was aimed at the wrong crops, and suggested exotic and short-life fruit.

**What we changed:** we kept the engine (scan → store → move → sell, escrow, ratings) and rebuilt it as **ChillChain** for produce that is expensive, spoils in days and sells on grade: strawberries, litchi, dragon fruit, flowers, mushrooms. Many of these have no proper mandi channel at all. For them the real problem isn't the price, it's the clock, so every lot now gets a sell-by clock and everything in the app is about beating it.

## Log

**Oct 19:** Copied the FreshGrid code into a new project. Set up Neon Postgres, Clerk login and the base tables (`neon/schema.sql`). Kept the 4-role idea: grower, transporter, buyer, cold-room owner.

**Oct 20:** Truck bookings and fare quotes (`002`). Cold-room owner side: register a room, check farmers in with a gate pass, release crates (`003a/b`). Wrote demo drivers for testing.

**Oct 21:** Driver accounts: see nearby loads, accept trips, mark delivered and paid (`004`, `005`). Added "verify later" so drivers can look around before uploading RC, licence and insurance.
ChillScore ratings (`006`). Ratings stay hidden for 48 hours, or until both sides rate, so nobody rates in revenge. New users start near an average score, so one bad rating doesn't sink them.
Saved farm profile for the crop advisor (`007`) and the settings page (`008`).

**Oct 22:** The big pivot day (`009_chillchain.sql`).

- New crop list: exotic and short-life fruit, flowers, mushrooms, polyhouse greens.
- Shelf-life rules and price steps (100 → 90 → 75 → 60%, then processor only). These live in Postgres as well as `src/lib/shelf.ts` and have to match exactly.
- Pre-coolers and ripening rooms rented by the hour, not the month.
- Shared reefer trucks: lots going the same route on the same day share one truck.
- Buyer pre-orders with escrow.

Then `010`: who graded a scan (AI or the grower), the air temperature the clock used, partial acceptance at delivery with refunds for the rejected part.

Grading: Gemini Vision for any crop, plus small on-phone models (Teachable Machine) for apple and cherry tomato so it works without signal. When Gemini runs out of free quota the grower can grade it themselves, and buyers see "self-graded".

Redesigned the UI to look like an iPhone app (grouped lists, large titles, the bottom tab bar).

**Oct 23:**

- **15 languages.** Every string goes through `t("English", "हिन्दी")`. A script collects them all and each language has its own dictionary. Urdu needed right-to-left.
- **Razorpay in test mode** (`011`). The server works out the amount, so the browser can't change what gets charged. Failed orders refund automatically.
- **Offline mode.** Growers are often in fields with no signal: the app opens with the last data, and scans made offline wait on the phone and upload later.
- **Live mandi prices** from Agmarknet (`012`), saved every 30 minutes. Two bugs we hit: flowers are reported per bunch, not per kg (tuberose came out at ₹0.60/kg), and comparing different markets on different days showed fake +400% jumps. Fixed by comparing each market only with itself.
- Deployed on Cloudflare Workers.

## Things that went wrong / what we learned

- Free AI quotas run out at the worst time. Always have a manual fallback.
- "Online" on a phone doesn't mean the internet works. We treat a failed request as offline too.
- Money rules (price steps, escrow amounts) belong in the database, not just the app.
- Government price data is messy; check units before trusting it.

## AI use (being upfront)

- **Inside the app:** Gemini Vision grades photos and runs the chat helper. TensorFlow.js runs the on-phone grading models.
- **While building:** we used an AI coding assistant (Claude) a lot: for writing and fixing a good share of the code, the translations into the 13 extra languages, and first drafts of the slides.
- **What was ours:** the idea and the pivot from FreshGrid, which features to build and in what order, the rules (shelf life, price steps, ratings), design choices, testing, and the deploy.

## What's still fake

No real money moves (test payments only). Storage facilities on the map are sample data. WhatsApp/SMS alerts and reefer temperature logs aren't built yet. See the README for the full list.
