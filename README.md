# Web files for HiProxy Universal Links

Static files to publish on the Universal Link domain (spec §5). Nothing here is part of the app build.

| Path on the domain | File | Notes |
| --- | --- | --- |
| `/.well-known/apple-app-site-association` | `.well-known/apple-app-site-association` | No extension. Serve as `Content-Type: application/json`, over valid HTTPS, **without any redirect**. |
| `/add` | `add/index.html` | Fallback page shown when the app is not installed. |
| `/import` | `import/index.html` | Same page (it detects `/import` from the path). |
| `/privacy` | `privacy/index.html` | Public privacy policy (vi + en). Fill in `[TÊN NHÀ PHÁT TRIỂN]` / `[DEVELOPER NAME]` and `[EMAIL LIÊN HỆ]` / `[CONTACT EMAIL]` first. |
| — | `.nojekyll` | Required on GitHub Pages, otherwise folders starting with a dot (`.well-known`) are not published. |

`appIDs` already contains the real team and bundle ID: `D76958N2BK.xteam.HiProxy`.

## Server configuration

- Links may carry `user`/`pass` in the query string (when a user shares with "include password").
  **Disable query-string logging** for `/add` and `/import` on the web server / CDN, and do not add
  analytics to these pages. The pages set `referrer: no-referrer` and never send the query anywhere.
- Apple fetches the AASA file through its CDN; after changing it, allow up to a day or test with
  `swcutil` / Developer Mode "Associated Domains Development".

## Enabling Universal Links in the app (once the domain is live)

1. In `Packages/ProxyKit/Sources/ProxyKit/Model/AppIdentifiers.swift` set
   `universalLinkHosts = ["your.domain"]`.
2. Add to `HiProxy/HiProxy.entitlements`:
   ```xml
   <key>com.apple.developer.associated-domains</key>
   <array><string>applinks:your.domain</string></array>
   ```
   (or Xcode › target HiProxy › Signing & Capabilities › + Associated Domains).
3. Build, install, open `https://your.domain/add?type=socks5&host=…&port=…` from Notes.

## Hosting on GitHub Pages

1. Create a public repository (e.g. `hiproxy-site`) and push the **contents** of this `Web/` folder to its root
   (including the hidden `.nojekyll` and `.well-known`).
2. Repository › Settings › Pages › *Deploy from a branch* › `main` / `(root)` › Save.
3. After about a minute the policy is at `https://<username>.github.io/hiproxy-site/privacy/` — use this URL as the
   Privacy Policy URL in App Store Connect.

Universal Links need the AASA file at the **root of a domain** (`https://<domain>/.well-known/…`). A project page
(`<username>.github.io/<repo>/`) is not a domain root: use a custom domain on the Pages site, or a
`<username>.github.io` user site, before enabling Universal Links.
