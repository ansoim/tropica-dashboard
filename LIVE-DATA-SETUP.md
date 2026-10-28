# Free scheduled Excel to GitHub Pages refresh

This project uses GitHub Actions instead of Power Automate's Premium HTTP action. A public GitHub repository's standard hosted runners are free. The workflow checks the public workbook every five minutes, refreshes `data/sales.json` only when the summary values change, and deploys the dashboard.

The workflow reads `Summary!A4:F10` from the public workbook link. Anyone with access to this public repository can also read the dashboard data and the workbook sharing URL. Do not use this setup for confidential workbook information.

## Install in the existing `ansoim/tropica-dashboard` repository

1. Extract the ZIP.
2. Upload the new `scripts/fetch_excel.py` file to a `scripts` folder in the repository.
3. Upload `.github/workflows/refresh-dashboard.yml` to that exact path. GitHub's **Add file > Create new file** also works: enter `.github/workflows/refresh-dashboard.yml` as the filename, then paste its contents.
4. Commit both files to the `main` branch.
5. In GitHub, open **Settings > Pages** and set **Source** to **GitHub Actions**. Save if asked.
6. Open the **Actions** tab, choose **Refresh Tropica dashboard from Excel**, click **Run workflow**, then confirm **Run workflow**.
7. Wait for the run to finish. A green check means the workbook downloaded and the dashboard deployed. If the run fails, open it and inspect the failed step. The fetch script reports when SharePoint returns a sign-in page instead of the workbook.

The workflow uses GitHub's built-in `GITHUB_TOKEN`; no personal access token, Power Automate HTTP action, or paid connector is needed. The workflow itself may take a few minutes to publish. Later, scheduled runs check for changes every five minutes. GitHub may delay scheduled runs during busy periods.

## If the download fails

The owner must keep the workbook link set to **Anyone with the link can view**, and downloading must be allowed. If the organization blocks anonymous file downloads, this free automatic route cannot read the workbook. Do not change the link to **Can edit** just to make the workflow work.
