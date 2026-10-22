# Bisate Victory – Report System (Node.js + MySQL, works offline)

Login with two roles. **Admin**: dashboard, reports, accounts, subject assignment, learners. **Teacher**: sees only the classes and subjects assigned to them; sets the test maximum and report maximum (automatic conversion, e.g. 100 → 45) and enters marks.

## Offline and sync
- Sign in once with internet. After that the app keeps the school data and the app itself on the device.
- With no internet, teachers and admin keep entering marks, changing test maximums, and admin can view the dashboard and print reports. A bar at the top shows "Offline" and how many changes are waiting.
- When the internet returns, waiting changes are sent to the database automatically (also every 20 seconds while waiting). Then the latest data is loaded.
- Needs internet: signing in the first time, adding or deleting accounts and learners, assigning subjects.
- If two people change the same mark, the last one to sync wins.
- Offline mode needs the site served over HTTPS (or http://localhost). Open it once online, then use "Add to Home screen" / "Install".

## Database: Aiven MySQL
The browser never talks to MySQL. The Node server does, with SSL. All tables start with `rs_` so they cannot clash with other apps in the same database.
1. In the Aiven service page click Show next to "CA certificate", save it as `ca.pem` in this folder.
2. `cp .env.example .env`, paste the database password into `DB_PASSWORD`, and set a long random `JWT_SECRET`.
3. `npm install`
4. `npm run seed` – creates the `rs_` tables, loads classes, learners, current P1–P5 marks and starter accounts. **Passwords are printed once. Copy them and change them after first login.**
5. `npm start` and open http://localhost:3000

## Put it online (AWS EC2)
Run with PM2 (`pm2 start server.js --name bisate`), put nginx with HTTPS in front, set `NODE_ENV=production` in `.env`, and keep `.env` and `ca.pem` private (never commit them).

## Notes
- Mid-term (Term I) only. Terms II and III stay blank on the report.
- Teachers can only change marks for subjects assigned to them.
- Aiven keeps backups; also take your own (`mysqldump`) before big changes.

## Android app (APK)
The Android app is the same web app packaged with Capacitor. The screens are inside the app, so it opens with no internet; data comes from your server when online.
1. Put this whole folder in a GitHub repository (do not commit `.env` or `ca.pem`).
2. On GitHub open Actions, choose "Build Android APK", click Run workflow.
3. When it finishes, download the `bisate-reports-apk` file from the run, unzip it, and install `app-debug.apk` on the phone (allow installs from unknown sources).
4. On first start type your server address (it must be https://...), then sign in once with internet.
Notes: the server must be reachable from the phones. In the app, the Reports tab has a button that builds a PDF of the whole class (or one learner) on the phone, then opens the Android share sheet so you can print it, save it, or send it by WhatsApp. For a Play Store release you need a signed release build.

## School logos
Admin tab, "School details and logos": upload the Ministry logo (left) and the school logo (right), set the school name, academic year and phone line. They appear on every report card and are kept offline.

## No website needed (free connector)
Teachers only ever use the app. The server is just a small hidden connector between the app and the database, so the database password is never stored inside the APK. Free options such as Render usually sleep when idle (the first sync after a pause is slow); the app keeps working offline meanwhile.
1. Run `npm run seed` once on your own computer (with `.env` filled in) to create the tables and accounts.
2. On Render create a new Blueprint from your GitHub repository (it reads `render.yaml`). Enter the Aiven values; for `DB_CA_PEM` paste the full certificate text.
3. Render gives you an address like https://bisate-reports.onrender.com. In GitHub open Settings, Secrets and variables, Variables, add `SERVER_URL` with that address, then run "Build Android APK" again. The app then needs no server address typed in.
