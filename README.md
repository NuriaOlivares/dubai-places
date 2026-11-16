# Dubai, saved

A personal web app that gathers the places I want to visit in Dubai and the UAE (restaurants, activities, getaways, shopping...) in one place. They used to be scattered across Google Maps, TikTok and Instagram. It works as a phone app (added to the home screen) and stores its data in Supabase.

> Personal use. A single HTML file, with no build step and nothing to install.

## What it does

- **Places by category:** Restaurants, Cafés, Activities, Getaways, Beaches, Shopping and Nightlife, with a second level of types (cuisine, "Cheap", "Day trip"...).
- **Several locations per place:** for franchises with the same name in different areas. Each location has its own area, Google Maps link, note and a "Not recommended" flag.
- **Configurable areas:** filter by area, plus a screen to add, rename (merges if the name already exists) and delete areas.
- **Creator's tip and link to the post** saved with each place, so I remember why I saved it.
- **Drafts:** anything added as a draft waits in its own tab until I approve it, and stays out of the rest of the app.
- **Rating when marking "Been":** 👍 loved it, 😐 it was ok, 👎 not for me.
- **Guests tab:** a shortlist of things to do when family or friends visit.
- **Calendar:** plan a saved place or a free-form plan on a date.
- **Near me:** sorts by distance using the phone's location (needs long Maps links that contain coordinates).
- **Closed:** "Mark as closed" moves a place out of the main list into a Closed section, where it can be restored or deleted.
- **Backup:** an "Export backup" button that gives all the data as JSON to copy.
- **Login** with email and password.

## How it's built

| Piece | What it is |
|---|---|
| `index.html` | The whole app: HTML, CSS and JavaScript, no framework |
| `icon.png` | Home screen icon |
| `setup.sql` | Supabase tables and security rules |
| Supabase | Postgres database, login and API |
| GitHub Pages | Free hosting |

The app uses `@supabase/supabase-js`, loaded from a CDN with a pinned version.

### Data

Three tables, all with a `user_id` and Row Level Security rules (each user only sees their own rows):

- `places`: name, category, type, `locs` (a JSON list of locations), price, note, source, guest, done, tip, link, rating, draft, closed.
- `events`: date, place (optional) and title, for the calendar.
- `areas`: area names.

Each item in `locs` has the shape `{ area, maps, lat, lng, note, avoid }`.

The old `area`, `maps`, `lat` and `lng` columns on `places` still exist for compatibility. `setup.sql` migrates them into `locs`.

## Setup

### 1. Supabase

1. Create a free project at [supabase.com](https://supabase.com). When creating it, I recommend leaving *Enable Data API* and *Automatically expose new tables* checked, and checking *Enable automatic RLS*.
2. In the **SQL Editor**, paste and run `setup.sql`. It's safe to run more than once.
3. Under **Authentication → Users → Add user**, create your user (email and password) and tick **Auto Confirm User**.
4. Under **Authentication**, turn off **Allow new users to sign up** so nobody else can register.
5. Under **Project Settings → API**, copy the **Project URL** and the **anon / publishable** key.

### 2. Configure the app

In `index.html`, near the top of the script, put in your values:

```js
var SB_URL = "https://YOUR-PROJECT.supabase.co",
    SB_KEY = "sb_publishable_...";
```

### 3. Publish with GitHub Pages

1. Create a public repository.
2. Upload `index.html`, `icon.png` and this README.
3. Go to **Settings → Pages**, choose **Deploy from a branch**, branch `main`, folder `/ (root)`.
4. Wait a minute. The app is then at `https://YOUR-USERNAME.github.io/REPO-NAME/`.

### 4. Install on iPhone

Open the link in Safari, sign in, then **Share → Add to Home Screen**. The home screen app doesn't share Safari's session, so you need to sign in once more there.

## Security

- The `sb_publishable_...` key is **public by design** and can live in the code. On its own it gives no access to the data.
- The data is protected by the **login** and the **RLS** policies in `setup.sql`.
- **Never** put Supabase's `secret` / `service_role` key in the repository, or keys for other services (Anthropic, OpenAI, Google).
- With a public repository, anyone can see the code and the project URL, but not the places.
- Now and then, check that all three tables show *RLS enabled* and that new-user sign-ups are still off.

## Updating the app

Edit `index.html` on GitHub (pencil icon), paste the new version and commit. On the phone, fully close the app and reopen it if the change doesn't show. If a new version needs database changes, run the matching SQL in Supabase first.

## Known limitations

- **Supabase free plan:** the project pauses after a week without activity and has to be reactivated from the dashboard.
- **Short Maps links** (`maps.app.goo.gl`) don't contain coordinates, so those locations are left out of the "Near me" ordering. Long links with `@lat,lng` work.
- **No automatic closed-place or opening-hours check:** the "Closed" status is manual.
- **No import from TikTok or Instagram** in this version.

## Roadmap

1. Import places from a reel: a Supabase function that reads the text and proposes the places with the creator's tip, landing as drafts.
2. An iOS Shortcut to share a reel from TikTok or Instagram straight to the app.
3. Automatic coordinates, Place ID and opening hours with Google Places, plus a closed-place check.
4. A "What should we do now?" button based on location, time and season (indoor/outdoor, best season).
5. A list shared with my husband (a household list, not separate users).
6. Measure how many saved places I actually visit.
