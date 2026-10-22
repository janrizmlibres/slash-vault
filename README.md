# Slash Vault

A Chrome extension that autofills [Slash](https://joinslash.com) virtual cards and billing addresses into checkout forms. Cards come from the Slash API. Addresses, field mappings and users live in Supabase. A small Express server on your machine sits between the two, so the Slash API key never reaches the browser.

## What it does

- **Cards.** The popup lists the cards in your Slash card group, with search and a CVV you reveal on demand. "Generate New Card" creates one through the Slash API.
- **Addresses.** An admin bulk-imports addresses from CSV or TSV. Each card in the list is paired with an address by round robin, and the filter box narrows the pool by city or state.
- **Autofill.** Fill a specific card from the popup, or press `Ctrl+Shift+F` to fill the next card that isn't cooling down. A used card and its address are held back for a cooldown period that admins set (30 minutes by default).
- **Field mapping.** Right-click an input on a checkout page and pick what it is, such as Card Number, CVV or ZIP. Mappings are saved per domain as selector profiles, and admins can edit them as CSS or XPath selectors.
- **Name detection.** Admins can add network profiles that read the shopper's name from a site's own API responses, by URL pattern and JSON path.

## Roles

Users sign up with email and password through Supabase Auth. Each new user gets their own Slash card group and sees only that group's cards. Admins see every card. They also get the admin panel, which holds the imports, the cooldown and both kinds of profile.

## Setup

You need Node.js and Chrome. You also need a Supabase project and a Slash API key.

1. Run `schema.sql` in the Supabase SQL editor. `schema_reset.sql` drops everything if you need to start over.
2. Copy `.env.example` to `.env` and fill it in. The server also reads `SLASH_ACCOUNT_ID`, which card creation needs.

   ```env
   SUPABASE_URL=...
   SUPABASE_KEY=...            # service role key, server only
   VITE_SUPABASE_URL=...
   VITE_SUPABASE_ANON_KEY=...  # anon key, bundled into the extension
   SLASH_API_KEY=...
   SLASH_ACCOUNT_ID=...
   ```

3. Install, build and start the server:

   ```bash
   npm install
   npm run build    # or npm run dev to rebuild on change
   npm run server   # Express on http://localhost:3000
   ```

4. In Chrome, open `chrome://extensions`, turn on Developer mode, click "Load unpacked" and pick `dist/`.

To make someone an admin, set their `role` to `admin` in the `users` table.

## Layout

```
src/popup/       React popup (400x600)
src/background/  service worker that talks to the server, context menus, the shortcut
src/content/     content script that maps fields and fills them
server/          Express API over Supabase and Slash
schema.sql       tables and row-level security policies
```

## Status

Built for one client and run from localhost. The server trusts the user ID and role the extension sends, so don't expose it beyond your own machine. There are no automated tests.

## License

ISC, as declared in `package.json`.
