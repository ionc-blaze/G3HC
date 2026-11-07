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
| `NOTIFY_EMAIL` | Email address to alert on each new request. Blank = no email (keep it blank while testing). |
| `MAX_DAYS_AHEAD` | How far ahead people can book (default 540). |
| `BUFFER_MINUTES` | Time allowed before the set-up time (default 60). |
| `PICKUP_ALLOWANCE_MINUTES` | Time allowed after the pick-up time for collecting (default 60). A booking holds the calendar from the buffer before set-up until this long after pick-up. A request that overlaps another booking is saved as a `Time review` for you to check. Touching exactly is fine. |
| `MIN_DAYS_AHEAD` | Earliest date customers can pick, in days from today (default 1, so tomorrow onwards, never today). |
| `SEND_CLIENT_EMAILS` | `yes` or `no`. Emails the customer when a request arrives and when you confirm it. |
| `REPLY_TO_EMAIL` | Where customer replies go. Blank while testing, so replies go to the Google account running the script. Add the business email when going live. |
| `ADMIN_KEY` | Secret for the private bookings page (see below). Clear it to switch the page off. |
| `WHATSAPP_TO` | Your WhatsApp number, like `+447700900123`, for alerts (see below). Blank = off. |
| `DELIVERY_BASE` | Delivery charge in pounds (default 80). Shown to the customer as information; not charged online. |
| `DELIVERY_STEP_MILES` / `DELIVERY_STEP_PRICE` | Outside the M25, an estimated extra of this many pounds (default 10) for each started block of this many miles (default 5) beyond the M25. Only an estimate: it is shown to the customer and written in the Bookings sheet, but not charged online. |

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

## Updating after a change to Code.gs

1. Apps Script > replace everything in Code.gs with the new version > Save.
2. **Deploy > Manage deployments > pencil icon > Version: New version > Deploy.** The web address stays the same.
3. Upload the new `index.html` to GitHub, with your web address pasted back into `SCRIPT_URL`.

## Delivery pricing and postcodes

Customers enter the venue postcode. The script looks it up on postcodes.io (free, no account) and works out where it is relative to the M25.

- **Base delivery (default £80)** is shown to the customer and written in the Bookings sheet. Nothing is charged online; you invoice as before.
- **Beyond the M25:** the page shows an estimate of £10 for each started 5 miles beyond the M25 (measured in a straight line to the nearest part of the M25). The Bookings sheet shows it in the "Delivery estimate" column, and the calendar event shows it too, for you to confirm with the customer.
- **If the postcode cannot be placed** (or the lookup service is down) the booking still goes through and the Delivery estimate column says to confirm any extra distance.
- **Approximate boundary:** the M25 outline in the script is hand-drawn from the junctions and is accurate to about a mile or two. Treat the estimate as a guide near the M25 itself.
- Bookings now has two extra columns, **Postcode** and **Delivery estimate** (added automatically at the end).

## One-off setup after pasting this version

1. Paste the new `Code.gs`, then in Apps Script choose **authorise** and **Run** (it asks for one more permission), then choose **setupSheet** and **Run**.
2. Deploy > Manage deployments > edit > New version.

`setupSheet` is safe to run again. It adds the Status drop-down and colours, the **In stock** column on Games, the **Includes** column on Packages, the new Settings rows, the trigger that reacts when you change a Status, a daily refresh, and the **This week** tab. The first time it fills In stock with placeholder numbers (2 for games, 1 for extras). Replace them with the real numbers. Its log shows the private bookings link.

## Set-up time, pick-up time and the calendar

Customers choose a **set-up time** (every half hour, 7:00 am to 6:00 pm) and a **pick-up time** (8:00 am to 8:00 pm, at least an hour after set-up). The earliest date is tomorrow. The pick-up time is saved in the Bookings sheet (column O, "Pick-up time").

- Each request creates a **timed calendar event** from an hour before set-up until an hour after pick-up (`BUFFER_MINUTES`, `PICKUP_ALLOWANCE_MINUTES`). The title reads like `PROVISIONAL set-up 11:00, pick-up 17:00: Package Two - Alex Smith`.
- Only bookings with Status `Provisional` or `Confirmed` hold time and stock. Enquiries, `Declined` and `Cancelled` hold nothing.
- A request whose hold overlaps a held booking is saved as `Time review`, with an event titled `ENQUIRY (time clash) ...`. The page tells the customer it will be checked.
- Older rows with no pick-up time are assumed to last two hours (plus the allowance).
- The public side only shares time ranges and which items are out, never who booked them.
- This is a simple stand-in for real travel time between jobs.

## Confirming a booking

In the **Bookings** tab the **Status** column is a drop-down: Provisional, Time review, Busy day enquiry, Stock review, Confirmed, Declined, Cancelled (coloured to match).

- Choose **Confirmed**: the calendar event is renamed `CONFIRMED set-up ...` and turns green, and the customer gets a "your booking is confirmed" email (once only, even if you change the Status again).
- **Declined** or **Cancelled**: the event is renamed and turns grey, it stops holding time and stock, and no email is sent (ring or email the customer yourself).
- Changing Status by editing the cell in the Sheet is what triggers this. Changes made on a phone through the Sheets app work too, but can take a few seconds.

## Emails to the customer

When a request arrives the customer is emailed a copy of what they asked for, saying nothing is final until you confirm. When you confirm, they get a confirmation. The Bookings sheet's **Client emails** column logs what was sent. Emails come from the Google account running the script, under the business name. Set `REPLY_TO_EMAIL` so replies reach the business. Set `SEND_CLIENT_EMAILS` to `no` to switch them off.

## Stock

The Games tab has an **In stock** column. Blank means the item is not counted. A number means that many copies exist.

- Stock is not subtracted by hand. For a chosen date and times, the script counts how many held bookings use the item at the same moment. When the copies run out, the item is greyed out ("Booked out for your date and times") and the server refuses it too. Cancelling or declining a booking frees the stock again.
- Customers pick the date and times first, then the games, so the page knows what is free.
- The Packages tab has an **Includes (for stock)** column for items that come with a package (for example `Candy Cart`). If one of those is out, the request is not refused: it is saved as `Stock review` for you to check.
- If a game appears on two rows (for example Giant and Kids), the first number found is used. Put the same number on both.
- Items that come with a package but are not in the Games tab are not counted.

## Seeing what is coming up

- **This week** tab (made by `setupSheet`): what is waiting for your decision, the next 7 days, the newest requests and the most booked games. It refreshes when a booking arrives, when a Status changes, and every morning. The existing **Dashboard** tab (income and months) is untouched.
- **Private bookings page:** the same information as a web page. The link is the Web app URL followed by `?page=admin&key=` and the `ADMIN_KEY` from Settings. `setupSheet` logs the full link and the owner alert email includes it once `ADMIN_KEY` is set. Anyone with the link can see customer names and phone numbers, so keep it private, and clear `ADMIN_KEY` to switch it off. It is read-only: confirm bookings in the Sheet.

## Alerts when a booking comes in

- **Email:** put your address in `NOTIFY_EMAIL`. Leave it blank while testing.
- **WhatsApp (optional):** WhatsApp has no simple free route for a script. The script can send through Twilio's WhatsApp service. You need a Twilio account, then add `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` and `TWILIO_WHATSAPP_FROM` in Apps Script > Project Settings > Script properties, and your number in `WHATSAPP_TO`. This part has not been tested against a real Twilio account. Twilio's sandbox needs the recipient to opt in first, and a live business number needs WhatsApp's approval of the message template. Twilio and WhatsApp charge per message; check their current prices.
- **No extra setup alternative:** the calendar event can notify a phone if Troy's Google account is invited to it or subscribes to the calendar, with a phone reminder.

## Day to day

- Each request appears in **Bookings** with a Status and a calendar event whose title starts with its status.
- Change the Status as above. Rename or delete nothing by hand.

## Test it

Make a booking on the live page, using an email address you can read. Check: a new Bookings row, a timed event on the calendar from an hour before set-up to an hour after pick-up, and the "request received" email. Then change its Status to Confirmed and check the event turns green and the confirmation email arrives. Delete the test rows and events afterwards.

## Things to know

- **Whose account:** the script runs as whoever deployed it, so the Sheet, calendar and emails belong to that Google account. For the real business, set it up in (or transfer it to) the business's account.
- **Public address:** anyone with the Web app URL can call it. The public side only exposes packages, games, stock use and time ranges, never customer details. The private bookings page is separate and needs the `ADMIN_KEY`. Requests are validated, there is a hidden spam trap, and writes are locked so two simultaneous bookings cannot collide.
- **Don't type into the Bookings tab's header row or insert columns:** the script reads fixed column positions.
- **Limits:** Apps Script suits a small business (consumer accounts can send about 100 emails a day and create several hundred calendar events a day).
- **Placeholder look:** colours and wording are a stand-in until Troy's branding is known.
