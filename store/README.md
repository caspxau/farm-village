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

**Category:** Games > Simulation. **Tags:** Farming, Casual, Offline.
**Content rating:** fill the IARC questionnaire: no violence, no user interaction or
chat, no purchases, no ads. Expect "Everyone" / PEGI 3.
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

## Play Console steps

1. Create the app (free game), fill Store listing with the text and images above.
2. App content: privacy policy URL, ads = no, content rating, target audience
   (13+ is simplest; choosing under 13 adds the Families policy review), data safety.
3. New personal developer accounts must run a **closed test with at least 12 testers
   for 14 days** before production is unlocked. Start this early.
4. Upload the `.aab` to the closed test track, then promote to production.

## Before each Play update

Only needed when the Android wrapper itself changes (icon, name, package). Game changes
just go to GitHub as usual; bump `VERSION` in `sw.js` so players get the new version.
