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
| `TELEGRAM_CHAT_ID` | Free Telegram alerts (see below). Blank = off. |
| `TELEGRAM_BUTTONS` | `no` (default): the alert links to the booking in the Sheet. `yes`: it also has Confirm and Decline buttons (see below). |
| `WHATSAPP_TO` | Your WhatsApp number, like `+447700900123`, for alerts (see below). Blank = off. |
| `DELIVERY_BASE` | Delivery fee in pounds within the M25 (default 80). Shown on the page and quoted in the confirmation email. Nothing is taken online. |
| `DELIVERY_STEP_MILES` / `DELIVERY_STEP_PRICE` | Outside the M25, an estimated extra of this many pounds (default 10) for each started block of this many miles (default 5) beyond the M25. Only an estimate: it is added to the delivery fee that is proposed when you confirm, and you can change that fee first. |
| `HOLDING_DEPOSIT` | Non-refundable holding deposit in pounds (default 20). Paid up front to secure the date, only asked for once you are sure you can do the booking, and taken off the total. `0` means none. |
| `REFUNDABLE_DEPOSIT_PERCENT` | Refundable deposit quoted when you confirm, as a percentage of the package price only (not extras or delivery). Default 50. `0` means no deposit. You refund it by hand after collection. |
| `PAYMENT_NOTE` | A line under the amounts in the confirmation email, for example how to pay the holding deposit. Type `-` to leave it out. |

Packages tab: edit names, descriptions, prices; changes appear on the page straight away. Keep each Package ID unique.

## What you can edit (the colours)

`setupSheet` colours the Sheet so it is clear what is yours to change:

- **Green tab or green cells: yours to edit.** The Packages and Games tabs, the Value column of Settings, the Status column on Bookings and the three money columns on the right of Bookings (Hire price, Delivery fee, Refundable deposit).
- **Grey tab or grey cells: filled in by the script or by formulas, please do not edit.** The Dashboard and This week tabs, the Name column of Settings and its explanations, and every other Bookings column (customer details, event ID, pick-up time, postcode, delivery estimate, client emails and stock items).
- The Bookings tab has an amber tab colour because it is half and half.
- Typing in a grey area shows a warning ("Filled in by the script") but is not blocked, so nothing stops you if you really mean it. Hover over cell A1 on Bookings or Settings for the colour key.

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

Customers choose a **set-up time** (every half hour, 6:00 am to 6:00 pm) and a **pick-up time** (8:00 am to 8:00 pm, at least an hour after set-up). The earliest date is tomorrow. The pick-up time is saved in the Bookings sheet ("Pick-up time" column).

- Each request creates a **timed calendar event** from an hour before set-up until an hour after pick-up (`BUFFER_MINUTES`, `PICKUP_ALLOWANCE_MINUTES`). The title reads like `PROVISIONAL set-up 11:00, pick-up 17:00: Package Two - Alex Smith`.
- Only bookings with Status `Provisional` or `Confirmed` hold time and stock. Enquiries, `Declined` and `Cancelled` hold nothing.
- A request whose hold overlaps a held booking is saved as `Time review`, with an event titled `ENQUIRY (time clash) ...`. The page tells the customer it will be checked.
- Older rows with no pick-up time are assumed to last two hours (plus the allowance).
- The public side only shares time ranges and which items are out, never who booked them.
- This is a simple stand-in for real travel time between jobs.

## Confirming a booking

The order is deliberate: look along the row, check or change the three money columns (S to U), and only then choose the Status in the last column. Choosing **Confirmed** is what sends the customer their email with those amounts.

In the **Bookings** tab the **Status** column (the last one, column V, after the prices) is a drop-down: Provisional, Time review, Busy day enquiry, Stock review, Confirmed, Declined, Cancelled (coloured to match).

- Choose **Confirmed**: the calendar event is renamed `CONFIRMED set-up ...` and turns green, and the customer gets a "your booking is confirmed" email with the amounts (once only, even if you change the Status again).
- **Declined**: the calendar entry is deleted and the Event ID cell says so, the booking stops holding time and stock, and the customer gets one polite decline email. Changing it back to Provisional or Confirmed puts the calendar entry back.
- **Cancelled**: the calendar entry is deleted, the Event ID cell says so, and the booking stops holding time and stock. No email is sent (ring or email the customer yourself). If you change a cancelled booking back to Provisional or Confirmed, its calendar entry is put back.
- **Speed:** when a request comes in, the customer is answered as soon as the booking row is saved. The calendar entry, the customer's email, your alerts (email, Telegram, WhatsApp) and the dashboard refresh follow a second or two later from a one-off background trigger, so the Event ID cell fills in just after the row appears. Settings, Packages and Games are kept for a minute to save re-reading them, and any edit you make in the Sheet clears that straight away. If the page says an email is on its way and it later fails, the Client emails cell says so.
- **Deleting a row** in the Bookings tab (right-click the row number, Delete row) also deletes that booking's calendar entry. Only entries the script made are ever removed; anything you added to the calendar by hand is left alone. Bookings made before this feature are covered once you have run `setupSheet` again. Deleting a row frees its time and stock, and nothing is emailed.
- If your Sheet is from an earlier version with Status in column M, the script moves it to the end by itself the first time it runs (the Status drop-down, colours and Dashboard formulas move with it).
- Changing Status by editing the cell in the Sheet is what triggers this. Changes made on a phone through the Sheets app work too, but can take a few seconds.
- You can do the same from Telegram with one tap (see below).

### The amounts

Every request gets three money columns on the right of the Bookings tab: **Hire price** (package plus any extras), **Delivery fee** (the base fee, plus the estimated distance charge outside the M25) and **Refundable deposit** (`REFUNDABLE_DEPOSIT_PERCENT` of the package price). The confirmation email quotes them with the total hire cost, the `HOLDING_DEPOSIT` (taken off the total) and the balance after it. The 50% refundable deposit is still to be confirmed with Troy. A "from" priced extra such as the candy cart is counted at its "from" price, and the Telegram confirm screen warns you to check the hire price. Type over any of them in the Sheet (or use the Telegram buttons) before you confirm and the email quotes your figure. Nothing is taken online.

## Dates and time zone

Every date people see is in UK order (dd/mm/yyyy, or "Thursday 19 November 2026" in emails and on the page). `setupSheet` sets the Sheet's locale to United Kingdom and its time zone to London, which also makes the date pickers and the way you type dates UK style. It rewrites any `yyyy-mm-dd` dates saved by earlier versions in the Received, Event date and Client emails columns, and updates the month rows on the Dashboard to match. The calendar events and the 6am daily refresh use London time whatever time zone the Apps Script project is set to.

## Days when you are not taking bookings

The **Days off** tab (created by `setupSheet`, green tab) lists dates you are not available. Click a cell in **From date** (double-click for the calendar picker, or type it like 25/12/2026); for a run of days also pick the last day in **To date**; leave To date blank for a single day. The third column is a private reason for your own notes, and customers never see it. Blocked dates are greyed out on the booking page and the script refuses them too. Past dates are ignored. Bookings already made on those dates are not changed, so check the calendar yourself. Delete a row to open the dates again. This blocks whole days only, not part of a day.

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
- **Private bookings page:** a summary only: counts, what is waiting, the next 7 days, newest requests and most booked games, each row with an **Open in Sheet** link that jumps to that booking's row. It shows no names, phone numbers or emails, so the detail stays behind your Google login. The link is the Web app URL followed by `?page=admin&key=` and the `ADMIN_KEY` from Settings (`setupSheet` logs it, and the owner alert email includes it). Clear `ADMIN_KEY` to switch the page off. It is read-only: confirm bookings in the Sheet. The **This week** tab inside the Sheet does show names and phone numbers, because the Sheet is already private.

## Alerts when a booking comes in

- **Email:** put your address in `NOTIFY_EMAIL`. Leave it blank while testing.
- **Telegram (free, recommended):** no cost and no business account.
  1. In Telegram, message **@BotFather**, send `/newbot` and follow the prompts. It gives you a token.
  2. In Apps Script > Project Settings > Script properties, add `TELEGRAM_BOT_TOKEN` with that token.
  3. Open your new bot in Telegram and send it any message (for Troy, he does this from his own Telegram).
  4. In Apps Script choose **telegramFindChatId** and **Run**. The Logs show your chat id.
  5. Put that number in `TELEGRAM_CHAT_ID` in the Settings tab. New requests now arrive as a Telegram message. To alert two people, make a Telegram group with both of them and the bot, send a message in it, run `telegramFindChatId` again and use the group's id (it starts with a minus).
  This has been tested against a mock of Telegram, not a real bot yet.
- **Link to the booking:** each Telegram alert (and the owner email) includes a link that opens the Sheet at that booking's row, where you change the Status to Confirmed or Declined.
- **Confirm and decline buttons (optional, off by default):** set `TELEGRAM_BUTTONS` to `yes` in the Settings tab, and after the steps above, choose **setupTelegramBot** in Apps Script and **Run** (once, after the web app has been deployed as a New version). Each new request then arrives with **Confirm** and **Decline** buttons.
  - **Confirm** first shows the hire price, delivery fee, refundable deposit and total. Tap **Send confirmation** to confirm (the Sheet, calendar and customer email all update), or **Change hire price**, **Change delivery fee** or **Change refundable deposit**, reply with a number in pounds, and check the new total.
  - **Decline** asks "are you sure" first. **Yes, decline** marks the Sheet and calendar and emails the customer.
  - If `setupTelegramBot` says the web app address does not end in `/exec` (the editor reports the test address), add a Script property called `WEB_APP_URL` with the **Web app URL** from Deploy > Manage deployments, then run it again.
  - Send `/waiting` to the bot to list everything still waiting for a decision, each with the same buttons.
  - Only your chat is obeyed. While the buttons are on, `telegramFindChatId` will not work (Telegram only allows one way of listening). Run **stopTelegramBot** first if you ever need it again.
- **WhatsApp (optional, not free):** WhatsApp has no free route for a script. The script can send through Twilio's WhatsApp service. You need a Twilio account, then add `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` and `TWILIO_WHATSAPP_FROM` in Script properties, and your number in `WHATSAPP_TO`. Not tested against a real Twilio account. Twilio and WhatsApp charge per message, and a live number needs WhatsApp's approval of the message wording.
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
