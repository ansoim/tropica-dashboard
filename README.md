# Tropica Sales Command Centre

This GitHub Pages dashboard reads its sales figures from `data/sales.json` on page load. The file in this package is a starter snapshot; follow [LIVE-DATA-SETUP.md](LIVE-DATA-SETUP.md) to connect the existing `Sales_Follow_Up_Tracker.xlsx` workbook to Power Automate and have it update the GitHub data file automatically.

The dashboard keeps the existing design, rotating 3D gold/silver/bronze trophies, animation controls, and logo. The bundled Three.js library is MIT licensed; see `THREE-LICENSE.txt`.

The repository and `data/sales.json` must be public for a public GitHub Pages site. The data feed contains only the Summary range used by the dashboard. Never put GitHub tokens, Microsoft passwords, or app secrets in the repository.

If WebGL setup fails, the browser console reports the trophy rank, setup stage, and underlying exception before the emergency 2D fallback is used. All asset paths remain relative for GitHub Pages project sites.