# REACHABLE

*Know before you go.*

A campus accessibility map. It reroutes wheelchair, crutch and low-vision users when a barrier is reported or cleared,
and it never draws a route it cannot vouch for. This is the **frontend** (Vite, React, TypeScript). It is a working
prototype for one fictional campus (Ashgrove University).

- **Website:** https://reachable-app.vercel.app (this code, running with no server)
- **Full app with a shared-reports server:** https://reachable-app.onrender.com (the free plan sleeps when idle, so it
  can take about a minute to wake)
- **Full project with the Express and SQLite backend:** https://github.com/vjxeyy/reachable

## What it does

- **Routes by profile, and by combinations of them.** Wheelchair, Crutches or Low vision, each with its own hard rules and
  penalties (steps, width, slope, path length). Tick more than one and the route follows the strictest rule of each.
- **Photo reports.** Tap a road on the map, or open **Report obstacle** and pick the road from a list. Add a photo and the app
  suggests what the obstacle is. You confirm or correct it.
- **Verification-aware routing.** New reports are *unverified*: routes avoid them whenever another way exists. A report
  blocks a path outright only after 2 independent confirmations or a Dashboard verify.
- **Fail-safe.** If every path is blocked, the app says **"No verified step-free route"** instead of drawing a risky one.
  The best unverified option is available on request, labelled *"Unverified, use with caution"*.
- **Freshness.** Reports go *outdated* if nobody re-confirms them (parked vehicle 3 h, flooding 12 h, blocked ramp
  3 days, broken pavement 30 days, missing kerb cut 90 days). Outdated reports warn but no longer block.
- **Still there / Cleared.** Anyone can confirm a report. With a shared server only the signed-in admin can mark one cleared;
  with no server (this repo on its own) the reports are the visitor's own, so Cleared stays open.
- **Dashboard** (`#/admin`). Anyone can look at it: filter reports and see how many paths are affected. Verifying or resolving reports needs the admin login when a shared server is connected (see below).

## Run it

You need **Node.js 22.12 or newer** and npm.

```bash
npm install
npm run dev
```

Open the address Vite prints. The header shows **This browser only**: there is no server in this repo, so reports are saved in
the browser (localStorage). Other commands:

```bash
npm test
npm run build
npm run lint
```

To point the app at an API instead, build with `VITE_API_URL` set to the API address (the full project repo has one).

## The 60-second demo

Start from a fresh load, or press **Reset demo** under **Demo tools** at the bottom of the page. Reset restores the 4
example reports (each tagged **Example**) and clears the route picker. The photo buttons are labelled **Sample drawing**:
they are bundled illustrations, not photos.

1. **Route.** From **Library**, To **Hostel**, with **Wheelchair** ticked. A step-free route appears (460 m).
2. **Report.** Tap the **Canteen to Hostel ramp** on the map, tap **Sample drawing: blocked ramp**, then **Looks right**
   and **Submit report**.
3. **Reroute.** The route redraws through Garden Court, 105 m longer, with the reason shown.
4. **Fail-safe.** Report a flood on **Library to Garden Court** the same way. The app says **"No verified step-free
   route"** and draws nothing.
5. **Clear.** Tap the hostel ramp again and press **Cleared**. The original route returns.

## Admin login

The **Admin** button at the top right opens the login page. This repo holds the screen and the calls to the login API; the
password is checked by the server in the full project (https://github.com/vjxeyy/reachable), never in the browser.

- With a shared server: anyone can view the Dashboard, but verifying or resolving needs the one admin's email and password. There is no sign-up and no other kind of
  account, so nobody else can get in. Everyone else can still read and report without an account.
- Without a server (this repo on its own, or the Vercel site): a password cannot be checked for real, so none is faked. The
  Dashboard opens with a note that changes only affect reports saved in this browser.

## How it is built

```
data/campus.json         the campus (15 places, 26 paths), pure data
data/seed-reports.json   the example reports
src/core/                routing, profiles, trust and seed builder, with tests (no React, no network)
src/classifier/          the photo classifier interface and the demo classifier
src/components/          map, route panel, report form, report list, trust badge, icons
src/pages/               Home (map), Report (report obstacle), Login, Admin (the Dashboard)
src/state/               the store (localStorage, or an API when one is set) and the API client
tests/                   the style-rules test
```

`cost = lengthM + profilePenalty + obstaclePenalty`, found with Dijkstra in `src/core/route.ts`. A strict pass skips paths that
break the profile's hard rules or carry any blocking report; only a route found there is shown as the route. Nothing in the
logic names a place: the campus is data.

## Tests and design rules

`npm test` runs the Vitest suite. It covers the routing acceptance checks, combined profiles, trust and freshness, the seed
data, the demo classifier, the map decoration and a **style-rules test** (`tests/style-rules.test.ts`). That last test fails with
the file and line if the interface gains purple or indigo colours, gradients, pill-shaped controls, emoji or glyph icons,
em or en dashes, cursor effects, scroll-driven animation or testimonial copy.

## What is simulated

- The **demo classifier** does not look at the picture. It uses the file name, the shape of the path and a hash of the
  image, so the demo is repeatable. It is always labelled **"AI suggestion (demo)"**. The full project repo can use real
  vision models behind the same interface.
- Example reports are tagged **Example** and the sample pictures are drawings, not photos.
- The campus is **fictional** and the map is an illustration, not to scale. There is no GPS or indoor map.
- Visitors are identified by an anonymous id in the browser (no accounts). Login is only checked by a server, so without one there is none.
