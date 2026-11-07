# Giant Garden Games booking page

A booking page (GitHub Pages) that reads packages and busy dates from a Google Sheet and writes each request back to the Sheet and a Google Calendar, through a small Google Apps Script.

```
Visitor -> index.html (GitHub Pages) -> Apps Script web app -> Sheet + Google Calendar (+ email to Troy)
```

## What is in this folder

| File | Where it goes |
|---|---|
| `index.html` | The GitHub repo (this is the booking page) |
| `apps-script/Code.gs` | Pasted into the Sheet's Apps Script editor |
| `apps-script/appsscript.json` | Pasted into the same project (settings) |

## Part 1: the Apps Script (about 10 minutes)

1. Open the Google Sheet (it needs the tabs **Packages**, **Bookings**, **Settings**).
2. **Extensions > Apps Script**.
3. Delete the starter code, paste in all of `Code.gs`, press **Save**.
4. Left menu: **Project Settings** (cog) > tick **Show "appsscript.json" manifest file in editor**. Go back to **Editor**, open `appsscript.json`, replace its content with the file provided, **Save**.
5. In the function drop-down choose **authorise**, press **Run**. Google asks for permission (Sheets, Calendar, email). Click through "Advanced > Go to project (unsafe)": it is your own script. This does not change any data.
6. **Deploy > New deployment > type: Web app**.
   - Execute as: **Me**
   - Who has access: **Anyone**
   - **Deploy**, then copy the **Web app URL** (ends in `/exec`).
7. Copy the Web app URL from the Deploy dialog (or Deploy > Manage deployments). It starts `https://script.google.com/macros/s/` and ends `/exec`. Do not copy the address from the browser bar after opening it: that is a temporary `script.googleusercontent.com` copy and bookings sent to it are never saved. Paste the real URL into a new browser tab. You should see JSON starting `{"ok":true,"packages":[...`. If so, the backend works.

**Whenever you change Code.gs later:** Deploy > Manage deployments > pencil > Version: New version > Deploy. The URL stays the same.

## Part 2: the page on GitHub

1. Open `index.html`, find `SCRIPT_URL: ''` near the top and paste the Web app URL between the quotes.
2. Create a new GitHub repository, add `index.html` to it.
3. Repo **Settings > Pages > Deploy from a branch > main / (root)**. After a minute the page is live at `https://<your-username>.github.io/<repo-name>/`.
4. With `SCRIPT_URL` empty the page runs in demo mode (sample data, nothing saved), so you can also preview it before connecting anything.

## Part 3: settings in the Sheet

Settings tab, column A is the name, column B the value:

| Setting | Meaning |
|---|---|
| `BUSY_THRESHOLD` | Bookings on one date before it shows as busy (default 4). Busy days still accept an enquiry. |
| `CALENDAR_NAME` | Calendar created/used for events (default "Giant Garden Games Bookings"). |
| `NOTIFY_EMAIL` | Email address to alert on each new request. Blank = no email. |
| `MAX_DAYS_AHEAD` | How far ahead people can book (default 540). |
| `SITE_URL` | Online payment only. The public address of the booking page, where customers return after paying. |
| `BUFFER_MINUTES` | Gap needed between bookings on the same day (default 60). A request for a time closer than this to an existing booking is not booked straight away: it is saved as a `Time review` for you to check. Exactly this far apart is fine. |
| `DELIVERY_BASE` | Delivery charge in pounds (default 80), added to every online basket. |
| `DELIVERY_STEP_MILES` / `DELIVERY_STEP_PRICE` | Outside the M25, an estimated extra of this many pounds (default 10) for each started block of this many miles (default 5) beyond the M25. Only an estimate: it is shown to the customer and written in the Bookings sheet, but not charged online. |
| `REFUNDABLE_DEPOSIT_PERCENT` | A refundable deposit added to the online basket, as a percentage of the package price only (default 50; 0 turns it off). It is paid with the games and delivery and refunded by hand after collection. |
| `DEPOSIT_PERCENT` | Online payment only. 100 charges the whole basket; 30 charges a 30% deposit. |

Packages tab: edit names, descriptions, prices; changes appear on the page straight away. Keep each Package ID unique.

## The Sheet tabs

**Packages** (one row per package)

| Column | Meaning |
|---|---|
| Package ID | A unique short code, for example p1 |
| Name, Description, Price | Shown on the page exactly as typed |
| Games to choose | How many games the customer picks. 1 shows a single choice, 0 means nothing to pick (fixed package) |
| Choose from | Which game category to pick from: Giant, Kids or Choice. Leave blank for fixed packages |
| Offer extras | Yes or No. Yes shows the optional extras list |

**Games** (one row per game): Name, Category (Giant, Kids, Choice or Extra), Price (for extras), Note (size or contents, optional). The same name can appear in more than one category. Add, rename or delete rows and the page updates straight away.

## Online payment (optional, Stripe test mode)

Customers see a basket (package plus priced extras) on the review step. After they send a request for a free date they get a **Pay now** button that opens Stripe Checkout for that total. Busy-day enquiries are not asked to pay. The amount is worked out by the script from the prices in your Sheet, never from the page. Items priced like "from £140" are not charged online.

1. Create a free account at stripe.com and switch on **Test mode**.
2. In Stripe, open **Developers > API keys** and copy the **Secret key** (starts `sk_test_`). Never paste it into the Sheet or the page.
3. In Apps Script, click **Project Settings** (cog) > **Script properties** > **Add script property**. Name `STRIPE_SECRET_KEY`, value the key. Save.
4. In the Settings tab, set `SITE_URL` to your live booking page address (the GitHub Pages link) and, if you want, `DEPOSIT_PERCENT`.
5. In Apps Script, pick **authorise** and **Run** once more (it now also asks to contact Stripe), then redeploy a New version.
6. Make a test booking and pay with the test card **4242 4242 4242 4242**, any future expiry, any 3-digit code. You come back to the page with "Payment received", and the Bookings row shows `Paid £... (Stripe)` in the Payment column.

To go live later, Troy creates his own Stripe account and the live secret key (`sk_live_`) replaces the test one. The booking Status is still changed by hand. Payments show in the Stripe dashboard too.

## Updating after a change to Code.gs

1. Apps Script > replace everything in Code.gs with the new version > Save.
2. **Deploy > Manage deployments > pencil icon > Version: New version > Deploy.** The web address stays the same.
3. Upload the new `index.html` to GitHub, with your web address pasted back into `SCRIPT_URL`.

## Refundable deposit

The basket shows the games (package and extras), delivery and a refundable deposit (50% of the package price by default, set with `REFUNDABLE_DEPOSIT_PERCENT`). The customer pays all of it online. After the games are collected, refund the deposit in the Stripe dashboard (Payments > the payment > Refund, for the deposit amount). The script never refunds anything itself. Check the wording matches Troy's terms and conditions. `DEPOSIT_PERCENT` (part-payment of the whole basket) is a different setting; leave it at 100 so people pay in full.

## Delivery pricing and postcodes

Customers enter the venue postcode. The script looks it up on postcodes.io (free, no account) and works out where it is relative to the M25.

- **Everyone pays the base delivery (default £80)** as a line in the online basket (and a deposit, if you set one, is a percentage of the whole basket including delivery).
- **Beyond the M25:** the page shows an estimate of £10 for each started 5 miles beyond the M25 (measured in a straight line to the nearest part of the M25). It is not charged online. The Bookings sheet shows it in the "Delivery estimate" column, and the calendar event shows it too, for you to confirm with the customer.
- **If the postcode cannot be placed** (or the lookup service is down) the booking still goes through and the Delivery estimate column says to confirm any extra distance.
- **Approximate boundary:** the M25 outline in the script is hand-drawn from the junctions and is accurate to about a mile or two. Treat the estimate as a guide near the M25 itself.
- Bookings now has two extra columns, **Postcode** and **Delivery estimate** (added automatically at the end).

## Choosing a time, and the one-hour buffer

Customers choose the arrival or set-up time (every half hour, 7:00 am to 6:00 pm) on the same step as the date. The page shows times close to an existing booking as "(we will need to check)".

- Only bookings with Status `Provisional` or `Confirmed` count. Enquiries, `Declined` and `Cancelled` do not block anything, so change an enquiry to `Confirmed` once you accept it and its time is then protected.
- A request inside the buffer is saved with Status `Time review` and a calendar event titled `ENQUIRY (time clash) ...`. No payment is asked for.
- The public side only shares which times need review on each date, never who booked them.
- This is a simple stand-in for real travel time and distance between jobs. If Troy wants that, it can be added later.

## Day to day

- Each request appears in **Bookings** with Status `Provisional` (or `Busy day enquiry` / `Time review`) and a calendar event titled `PROVISIONAL 11:00: ...` / `ENQUIRY (busy day) 11:00: ...` / `ENQUIRY (time clash) 11:00: ...`.
- Change Status to `Confirmed`, `Declined` or `Cancelled`. Declined and Cancelled stop counting towards a busy day. Rename or delete the calendar event by hand to match.

## Test it

Make a booking on the live page. Check: a new Bookings row, an all-day event on the calendar with the time in its title, and (if set) the email. Delete the test row and event afterwards.

## Things to know

- **Whose account:** the script runs as whoever deployed it, so the Sheet, calendar and emails belong to that Google account. For the real business, set it up in (or transfer it to) the business's account.
- **Public address:** anyone with the Web app URL can call it. The public side only exposes packages and date counts, never customer details. Requests are validated, there is a hidden spam trap, and writes are locked so two simultaneous bookings cannot collide.
- **Don't type into the Bookings tab's header row or insert columns:** the script reads fixed column positions.
- **Limits:** Apps Script suits a small business (consumer accounts can send about 100 emails a day and create several hundred calendar events a day).
- **Placeholder look:** colours and wording are a stand-in until Troy's branding is known.
