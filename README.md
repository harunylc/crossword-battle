# `web/site/` — what the domain becomes once the game is on Play

One page that says the game is on Google Play, and nothing else. It replaces the WebGL build and the
APK download page on production launch day.

## Why the game leaves the web

The WebGL build and `/indir/` both existed for one reason: before the store listing there was no other
way to let anyone play or install. Once the app is on Play they stop being useful and start being
harmful — **a sideloaded APK cannot be updated by Play**, so every person who installs one is frozen on
that build and out of reach of every fix afterwards. Two install routes also split the audience across
two versions for no gain.

## What must NOT be removed with it

Two files on this domain are load-bearing and neither has anything to do with the game being playable:

- **`app-ads.txt`** at the domain root (source: `web/app-ads-root/`). AdMob fetches it from the root of
  the website named in the Play listing to verify who is allowed to sell this app's inventory. Losing it
  is a direct hit to ad revenue, not a cosmetic problem.
- **`gizlilik/`** — the privacy policy. Play *requires* a reachable policy URL on the store listing, and
  the listing points here. A dead link is a policy violation, not a broken page.

`deploy-web.ps1` already excludes both from its mirror (`/XD "indir" "gizlilik"`), and the privacy page
is copied from `web/gizlilik/` on every deploy.

## Deploying it, on the day

`-BuildPath` is a parameter, so this needs no change to the script — point it here instead of at the
WebGL build and `/MIR` deletes the game as a side effect of mirroring:

```powershell
powershell -ExecutionPolicy Bypass -File tools/deploy-web.ps1 `
  -RepoPath D:\CrosswordBattleWeb -BuildPath web\site
```

Then remove the download page by hand, because the mirror is told to leave it alone:

```powershell
Remove-Item D:\CrosswordBattleWeb\indir -Recurse -Force
cd D:\CrosswordBattleWeb; git add -A; git commit -m "Retire the APK download page"; git push
```

**Check afterwards, in this order** — the first two are the ones that cost money or break compliance:

1. `https://harunylc.github.io/crossword-battle/app-ads.txt` still returns the file
2. `https://harunylc.github.io/crossword-battle/gizlilik/` still returns 200
3. the root shows this page and the Play button opens the listing
4. `/indir/` returns 404

## The store link

`https://play.google.com/store/apps/details?id=com.harunylc.crosswordbattle` — derived from the package
name, so it is correct before the listing exists and needs no editing on the day. It 404s until the app
is published, which is exactly why this page is deployed *after* the production release goes live and
not before.
