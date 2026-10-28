# Live Excel to GitHub Pages setup

The dashboard reads `data/sales.json` on every page load. `data/sales.json` in this ZIP is only a starter snapshot; the flow below replaces it when the workbook changes. The workbook name and sheet names stay as they are.

## A. Create the GitHub repository

1. On GitHub, choose **New repository**. Give it a repository name such as `tropica-sales-dashboard` and set it to **Public** (needed for a public GitHub Pages site).
2. Extract this ZIP on your computer.
3. In the new repository, choose **Add file > Upload files**. Upload the contents of the extracted project folder, keeping `index.html`, `.nojekyll`, `assets/`, and `data/` at the repository root. Commit to `main`.
4. Open **Settings > Pages**. Under build and deployment, select **Deploy from a branch**, branch `main`, folder `/(root)`, then Save.
5. Wait for GitHub Pages to show the site address. It will look like `https://GITHUB-USER.github.io/REPOSITORY/`.

## B. Confirm the Office Script

The script shown in Excel is named `Script` and has already run successfully. It can stay named `Script`; renaming the workbook is not needed. Its code should read `Summary!A4:F10` and return the result as JSON:

```typescript
function main(workbook: ExcelScript.Workbook): string {
  const values = workbook.getWorksheet("Summary").getRange("A4:F10").getValues();
  return JSON.stringify({ updatedAt: new Date().toISOString(), values });
}
```

If the script is still open in the Office Scripts pane, the green success indicator confirms it is saved in your Office Scripts list. You can leave the title as `Script` and select that title in Power Automate.

## C. Create the automatic flow

1. Go to [Power Automate](https://make.powerautomate.com/) and choose **Create > Automated cloud flow**.
2. Choose the OneDrive for Business trigger **When a file is modified (properties only)**. Select the folder containing `Sales_Follow_Up_Tracker.xlsx`.
3. Add a condition so the flow continues only when **File name with extension** equals `Sales_Follow_Up_Tracker.xlsx`.
4. In the Yes branch, add Excel Online (Business) **Run script**. Select the same workbook and script `Script`.
5. Add an HTTP action to update the JSON file in GitHub. The HTTP action may require a Power Automate Premium license.

### GitHub authorization for the flow

Create a **fine-grained personal access token** in GitHub for this one repository, with **Contents: Read and write** permission. Set an expiry date. Enter the token only into the Power Automate HTTP action's Authorization header as `Bearer YOUR_TOKEN`; never paste it into this dashboard or share it in chat. Enable **Secure inputs** and **Secure outputs** on the HTTP actions so the token and payload do not appear in run history.

### HTTP action 1: read the existing file SHA

- Method: `GET`
- URI: `https://api.github.com/repos/GITHUB-USER/REPOSITORY/contents/data/sales.json?ref=main`
- Headers:
  - `Accept`: `application/vnd.github+json`
  - `Authorization`: `Bearer YOUR_TOKEN`
  - `X-GitHub-Api-Version`: `2022-11-28`

Replace `GITHUB-USER` and `REPOSITORY` with the new repository owner and name. In this response, GitHub returns a `sha` for the existing file.

### HTTP action 2: write the new data

- Method: `PUT`
- URI: `https://api.github.com/repos/GITHUB-USER/REPOSITORY/contents/data/sales.json`
- Use the same headers as the GET action.
- JSON body:

```json
{
  "message": "Refresh dashboard data from Excel",
  "content": "BASE64_SCRIPT_RESULT",
  "sha": "CURRENT_FILE_SHA",
  "branch": "main"
}
```

For `content`, use the Power Automate expression `base64(outputs('Run_script')?['body/result'])` (replace `Run_script` with the actual action name if Power Automate named it differently). For `sha`, insert the `sha` value from HTTP action 1 using dynamic content or the expression `body('HTTP_GET')?['sha']` (replace `HTTP_GET` with that action's actual name).

The GitHub file contents API requires the current `sha` when updating an existing file. Each successful PUT creates a commit; that causes Pages to deploy the updated JSON with the site.

## D. Verify

1. In Power Automate, save the flow and run a test.
2. Open `data/sales.json` in GitHub and confirm its `updatedAt` and values changed.
3. Wait for the Pages deployment to complete, then open the dashboard and reload it. The status beside the summary controls shows when the feed was last generated.
4. Change a harmless test value in the workbook, wait for the flow and Pages deployment, and verify the dashboard updates.

The OneDrive modification trigger is best effort, so changes can arrive with a delay and a file edit can sometimes start more than one flow run. The dashboard uses the most recently committed JSON data. Since the repository and workbook are public, the dashboard data file is public too.