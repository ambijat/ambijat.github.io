# ambijat.github.io

User site for `https://ambijat.github.io/`. Its only hard job is serving
`/.well-known/assetlinks.json`, which Android and Google Play fetch to verify
Trusted Web Activity deep links for hosts on this domain.

## Why this repo has to exist separately

Project repos (`gym-band`, `gym-log`, `gym-companion`) are published under
`https://ambijat.github.io/<repo>/`. Android digital asset link verification
only ever fetches the **domain root**:

    https://ambijat.github.io/.well-known/assetlinks.json

A file at `https://ambijat.github.io/gym-band/.well-known/assetlinks.json` is
never consulted. Root paths are served only by the repo named exactly
`ambijat.github.io`.

## `.nojekyll` is load-bearing

GitHub Pages runs Jekyll by default, and Jekyll excludes dot-directories. Without
`.nojekyll`, `.well-known/` is not published and the asset links file 404s.

## Fingerprints in `assetlinks.json`

| SHA-256 | Key | Where it comes from |
| --- | --- | --- |
| `22:88:68:5D:…:6A:61:16:CF` | Play App Signing key | Play Console → Setup → App integrity. The key Google re-signs the installed APK with — this is the one that must match on real devices. |
| `CB:AB:28:99:…:B2:4E:A3` | Upload key | `ANDROIDWORKS/gym-band/android.keystore`, alias `bandtrainingkey`. Used to sign the AAB before upload. |

Both are listed so the link verifies whether Play re-signs (current setup) or a
locally signed APK is installed. Android accepts the link if **any** listed
fingerprint matches.

## Adding another app

Append a second object to the JSON array with that app's `package_name` and its
Play App Signing fingerprint. One array entry per Android package.

## Verifying after a change

    curl -sS -D- -o- https://ambijat.github.io/.well-known/assetlinks.json

Expect HTTP 200 and `content-type: application/json`. Then in Play Console →
Deep links → the failing path → **Recheck verification**.

Play requires a new app version before users on the old build pick up the fix;
on-device re-verification normally happens on install or update.
