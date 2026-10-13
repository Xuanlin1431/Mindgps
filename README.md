# Putting MindGPS online (Netlify)

This folder is the member version of the app. The free parts are in `public/index.html`.
The members-only readings sit in `netlify/functions/premium.json` and are only sent out
by the `content` function when someone enters a valid member code.

## One-time setup (about 15 minutes)
1. Create a free account at netlify.com (you can use the same account as shaneychen.com).
2. Install the Netlify command line once on your computer: `npm install -g netlify-cli`
3. Open a terminal in this `deploy` folder and run:
   - `netlify login`
   - `netlify deploy --prod`  (choose "Create & configure a new site", publish folder = `public`)
   Note: dragging the folder onto the Netlify website does NOT install the member function, so use the command above
   (or connect a GitHub repository containing this folder).
4. In Netlify: Site configuration → Environment variables → Add variable
   - Key: `MEMBER_CODES`
   - Value: the codes members may use, separated by commas, e.g. `LGPS-Z8XW-K6TT,LGPS-QSYC-YEZ9`
   Then redeploy (Deploys → Trigger deploy) so the function picks it up.
5. Optional: Domain management → add your own address, e.g. app.shaneychen.com.

## Payments (Stripe) – credits
Credits never expire and there are no subscriptions. Open a person's full reading = 3 credits (once per person, kept for good) ·
PDF report = 1 credit · daily card free. Everyone who signs in with their email gets 5 free credits every month
(reset on the 1st, Singapore time; not carried over; used before bought credits). Bought credits never expire.
Packs: 10 credits USD 3.10 · 35 credits USD 9.31 · 400 credits USD 94.31. Team member codes still open everything for free.

1. Sign up at stripe.com (Singapore business). Finish "Activate payments" (business details, bank account for payouts).
2. Developers → API keys → copy the **Secret key** (starts `sk_test_` while testing, `sk_live_` when live).
3. Netlify → Site configuration → Environment variables, add:
   - `STRIPE_SECRET_KEY` = your secret key
   - `TOKEN_SECRET` = any long random text (e.g. 40 random letters and numbers). Never change it once customers have paid,
     or their access codes stop working.
   Then Deploys → Trigger deploy.
4. Test with the `sk_test_` key and Stripe's test card 4242 4242 4242 4242 (any future date, any CVC). When happy, swap to `sk_live_`.
5. Optional: Stripe → Settings → Emails → turn on "Successful payments" receipts.

Sign-in email (one-time code; 5 free credits a month per account)
6. Sign up at resend.com (free for up to 3,000 emails a month). Domains → Add domain → shaneychen.com, then add the
   DNS records it shows (in Netlify: Domains → shaneychen.com → DNS settings). Wait until it says Verified.
7. API Keys → Create API key (sending access). In Netlify add:
   - `RESEND_API_KEY` = that key
   - `FROM_EMAIL` = `MindGPS <hello@shaneychen.com>` (any address on your verified domain)
   Then Deploys → Trigger deploy.
- Everyone who signs in appears in Stripe → Customers (metadata `joined`). Those who ticked the
  news box also have `marketing = yes`, so you can export them as leads for your courses.
- Gmail dots and "+" tags count as the same person (amy.tan+x@gmail.com = amytan@gmail.com).

How it works
- Packs and credit costs are set in `netlify/functions/account.js` (PACKS, COST_OPEN, COST_PDF, FREE_MONTHLY) and shown in the app's Credits window. You do not need to create products in Stripe.
- After paying, the customer gets an access code (LG-…). The app remembers it; on another phone or computer they tap
  Plans → "Already paid on another device?" and paste the code. If someone loses it, find them by email in Stripe → Customers;
  their code is shown under Metadata as `access_code`.
- Credits and the list of opened people are stored on the customer in Stripe (metadata), so there is no database to run.
- Pro tools (success path, partner match, candidate fit, team map) unlock with the 400-credit Pro pack, team member codes,
  or by hand: Stripe → Customers → the customer → Metadata → add `pro` = `yes`.
- To give someone credits by hand: Stripe → Customers → the customer → Metadata → change `credits`.
- Stripe fees (about 3.4% + SGD 0.50 per card payment in Singapore) are why the smallest pack is 10 credits.

## Member codes
- `member-codes.txt` has 50 ready-made random codes. Give one code to each member.
- To remove a member, delete their code from `MEMBER_CODES` and redeploy.
- Members enter the code once on each device; the app remembers it.
- Keep `member-codes.txt` private. Do not upload it to the website.

## Updates
- After any change, run `python3 build.py` in the app folder, then `netlify deploy --prod` here.
- `public/_headers` tells phones to always fetch the newest version, so old copies don't linger.

## On phones
Members open the web address in Safari (iPhone) or Chrome (Android) and use
"Add to Home Screen" – the app then opens full screen with the MindGPS icon.
