# Play Store release kit

Everything needed to put Farm Village on Google Play as an Android app. The app is a
Trusted Web Activity (TWA): a thin Android wrapper that opens the live game from
https://caspxau.github.io/farm-village/ full screen, with no browser bar. Updates pushed
to GitHub reach players without a new Play release.

## Files here

| File | Play Console field |
|---|---|
| `icon-512.png` | App icon (512 x 512) |
| `feature-graphic-1024x500.png` | Feature graphic |
| `screenshots/*.png` | Phone screenshots (1080 x 1920, upload 2 to 8) |
| `../privacy.html` | Privacy policy URL: https://caspxau.github.io/farm-village/privacy.html |

## Store listing text

**App name:** Farm Village

**Short description (80 max):**
Grow crops, raise animals and build the coziest farm in the valley.

**Full description:**
Welcome to Farm Village, a cozy farming game you can play anywhere, even offline.

Plant wheat, strawberries, pumpkins and 24 other crops, each with its own season.
Raise chickens, cows, sheep, pigs, horses and llamas, and give every animal a home.
Pick fruit from apple, cherry and peach trees.

Turn your harvest into jam, bread, cheese, pies and cider in the Farm Kitchen, Cider
Press and more. Load Dale's truck and send deliveries to town, fill your neighbors'
orders, and trade with them for the materials to upgrade your barn and buildings.

Decorate with fences, flower beds, gnomes, a windmill and a hot air balloon, furnish
your farmhouse, and dress your farmer just the way you like.

- Painted, hand-drawn style with a Cartoon option
- Seasons and weather that change how your crops grow
- Tractor, seeder and harvester to farm whole fields at once
- Ribbons, collections, daily goals and quests
- Plays in portrait or landscape, works offline
- No ads, no accounts, no data collected
- Optional Farm Cash packs for premium outfits, furniture and decorations

**Category:** Games > Simulation. **Tags:** Farming, Casual, Offline.
**Content rating:** fill the IARC questionnaire: no violence, no user interaction or
chat, no ads, **in-app purchases: yes** (Farm Cash, digital goods). Expect "Everyone" / PEGI 3 with the
"In-app purchases" notice.
**Data safety:** "No data collected" and "No data shared" (the game only stores the save
on the device).

## Build the Android app (one time)

1. Install the tool: `npm i -g @bubblewrap/cli` (it downloads a JDK and the Android SDK
   on first run).
2. In an empty folder: `bubblewrap init --manifest https://caspxau.github.io/farm-village/manifest.webmanifest`
   - Application ID: something like `io.github.caspxau.farmvillage` (permanent, cannot change later)
   - Display mode: fullscreen. Orientation: default.
   - Let it create a new signing key and **back up the keystore file and its passwords**.
     Losing them means you can never update the app.
3. `bubblewrap build` produces `app-release-bundle.aab` (upload this to Play) and an
   `.apk` for testing on your own phone.
4. Get the key fingerprint: `bubblewrap fingerprint list` (or `keytool -list -v -keystore android.keystore`).

## Verify the site so the browser bar disappears

Android only hides the URL bar when the website vouches for the app through
`/.well-known/assetlinks.json` **at the domain root**. For GitHub Pages that is
`https://caspxau.github.io/.well-known/assetlinks.json`, which lives in a separate repo
named exactly `caspxau.github.io` (this repo only serves `/farm-village/`).

1. Create the public repo `caspxau.github.io` with GitHub Pages turned on.
2. Add `.well-known/assetlinks.json` (Bubblewrap prints it, or `bubblewrap fingerprint generateAssetLinks`):
   ```json
   [{"relation":["delegate_permission/common.handle_all_urls"],
     "target":{"namespace":"android_app","package_name":"io.github.caspxau.farmvillage",
       "sha256_cert_fingerprints":["<UPLOAD KEY SHA-256>","<PLAY APP SIGNING KEY SHA-256>"]}}]
   ```
3. Add an empty `.nojekyll` file so GitHub serves the `.well-known` folder.
4. After the first upload, Play Console > Setup > App signing shows a second SHA-256
   (Google's app signing key). Add it to the list above, or the installed Play version
   will show a URL bar.

## In-app purchases (Farm Cash packs)

The game sells Farm Cash through Google Play Billing, which a TWA reaches through the web
Digital Goods API. Outside the Play app (the plain website) the Cash Shop still shows, but
the buy buttons only explain that purchases need the app.

1. Bubblewrap: in `twa-manifest.json` set `"features": {"playBilling": {"enabled": true}}`,
   then `bubblewrap update` and `bubblewrap build`. This adds the billing library and the
   `com.android.vending.BILLING` permission to the Android wrapper.
2. Play Console > Monetize > Payments profile: set up a merchant account (needed before
   you can sell anything).
3. Play Console > Monetize > Products > In-app products: create five **consumable**
   products with exactly these ids (the game matches them in `CASH_PACKS` in `index.html`):

   | Product id | Farm Cash | Suggested price |
   |---|---|---|
   | `cash_10` | 10 | $0.99 |
   | `cash_55` | 55 | $4.99 |
   | `cash_120` | 120 | $9.99 |
   | `cash_260` | 260 | $19.99 |
   | `cash_700` | 700 | $49.99 |

   The game shows Play's local price for each pack once it is available; the dollar prices
   in `CASH_PACKS` are only a fallback label.
4. Add your Google account as a **licence tester** (Setup > Licence testing) to buy packs
   for free while testing the closed track.
5. Data safety: purchases are processed by Google Play; the game itself only keeps the
   purchase token on the device (never sent anywhere), so "No data collected" still holds.
   Update the store listing content rating answers to say the app has in-app purchases.

How it works in the code: `showCashShop()` lists the packs, `buyPack()` opens the Play
payment sheet with `PaymentRequest`, and `grantPurchase()` adds the Farm Cash once per
purchase token (kept in `S.iap`) and then consumes the purchase so it can be bought again.
On every launch `claimPending()` delivers any purchase that was paid for but not yet
granted (for example if the app closed mid-purchase).

## Play Console steps

1. Create the app (free game), fill Store listing with the text and images above.
2. App content: privacy policy URL, ads = no, content rating (with in-app purchases), target audience
   (13+ is simplest; choosing under 13 adds the Families policy review), data safety.
3. New personal developer accounts must run a **closed test with at least 12 testers
   for 14 days** before production is unlocked. Start this early.
4. Upload the `.aab` to the closed test track, then promote to production.

## Before each Play update

Only needed when the Android wrapper itself changes (icon, name, package). Game changes
just go to GitHub as usual; bump `VERSION` in `sw.js` so players get the new version.
