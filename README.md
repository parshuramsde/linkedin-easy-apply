# LinkedIn Easy Apply Helper

A Chrome extension that makes LinkedIn **Easy Apply** fast. You browse LinkedIn normally. The helper:

- opens the next 5 un-applied jobs from a search page in new tabs,
- starts Easy Apply in each tab and fills every step from your answers bank (`profile.json`),
- asks you, inside a small panel, only the questions it has never seen, and remembers your answer,
- clicks **Next** until the review page.

**It never clicks Submit. Submit is always your own click.** It refuses to press any button whose label
contains the word "submit".

Repository: `github.com/parshuramsde/linkedin-easy-apply` (private). The repository contains **no personal
data**. Your answers live only in `profile.json` on your laptop, which git ignores.

---

## 1. Setting it up on a new laptop (from GitHub)

Follow these in order. Total time about 15 minutes.

### 1.1 Install the three tools

| Tool | Why | How |
|------|-----|-----|
| Google Chrome | runs the extension | google.com/chrome |
| Git | downloads and updates the code | macOS: run `git --version` in Terminal and accept the install prompt. Windows: git-scm.com |
| Node.js 18 or newer | runs the small helper scripts | nodejs.org, LTS version. Check with `node -v` |

`npm install` is **not** needed to use the extension. It is only needed for the tests (see section 7).

### 1.2 Connect the laptop to your GitHub account

Pick one.

**Option A: HTTPS (simplest).** Nothing to configure. The first `git clone` of a private repo opens a browser
window asking you to log in to GitHub. Use the account that owns the repo.

**Option B: SSH key (what the work laptop uses).** Needed only if the laptop already has another GitHub
account's key, so the two must be kept apart.

```bash
ssh-keygen -t ed25519 -C "parshuramkumarsde@gmail.com" -f ~/.ssh/id_ed25519_parshuramsde
cat ~/.ssh/id_ed25519_parshuramsde.pub      # copy the output
```
GitHub → Settings → SSH and GPG keys → New SSH key → paste. Then add this block to `~/.ssh/config`
(create the file if missing):
```
Host github-parshuramsde
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_parshuramsde
    IdentitiesOnly yes
```
Test it: `ssh -T git@github-parshuramsde` should print "Hi parshuramsde!".

### 1.3 Download the code

```bash
cd ~                                   # or wherever you keep projects
# Option A (HTTPS)
git clone https://github.com/parshuramsde/linkedin-easy-apply.git
# Option B (SSH alias)
git clone git@github-parshuramsde:parshuramsde/linkedin-easy-apply.git

cd linkedin-easy-apply
```

### 1.4 Put your answers file in place

The code ships without `profile.json`. You need one of these:

- **You have the file from your old laptop** (best): copy `profile.json` from the old laptop's project folder
  into this folder. Use AirDrop, a USB stick, or a private drive. Do not email it or commit it.
- **You do not have it**: create it from the template and fill in every `FILL_ME`:
  ```bash
  cp profile.example.json profile.json
  ```
  Fastest way to fill it: give `profile.json` and your resume to Claude and ask it to fill the personal block
  and the answer rules from the resume. Section 4 explains the format.

Then generate the copy the extension reads:
```bash
node sync-extension.js
```
Expected output: `extension/profile.json and extension/config.json updated`.
If it says `profile.json not found`, step 1.4 was skipped.

### 1.5 Load the extension in Chrome

1. Open Chrome and go to `chrome://extensions`.
2. Turn on **Developer mode** (switch in the top-right corner).
3. Click **Load unpacked** and choose the `extension/` folder inside `linkedin-easy-apply`.
4. "Easy Apply Helper" appears in the list. Leave it enabled.

Optional: click the puzzle-piece icon in Chrome's toolbar and pin the helper so its options are one
click away.

### 1.6 Log in and verify

1. Log in to LinkedIn in this Chrome. The helper never sees or stores your password; it only works on pages
   you are already logged in to.
2. Open any job search, for example `linkedin.com/jobs/search/?keywords=java%20developer&f_AL=true`
   (`f_AL=true` is the Easy Apply filter).
3. A panel titled **⚡ Easy Apply Helper** appears at the bottom-right with
   `Found: n | Remaining: n | Opened: 0`. You are set up.

If the panel's small grey hint line starts with **⚠ profile.json missing**, run `node sync-extension.js`
and click the panel's **↻** button.

---

## 2. Daily flow

1. **Search page.** Open a LinkedIn job search with the Easy Apply filter on. The panel shows
   **Found | Remaining | Opened** and a line "Skipped here: X applied, Y external, Z by title".
   Click **Open next 5 jobs** or press **Alt+O**. The next five unopened jobs open in background tabs, in
   page order from the top. Each opened card gets a green **✓ opened** badge.
   - Batch size is editable in the panel.
   - **Skip senior / lead / manager titles** checkbox applies the title list from `config.json`.
   - **Reset history** forgets which jobs were opened and applied.
   - Scroll down or go to the next results page and press Alt+O again when Remaining reaches 0.
2. **Each job tab.** The helper starts Easy Apply by itself and fills step after step. Watch the panel:
   - Nothing listed: it is clicking Next for you. Wait.
   - **Questions listed**: it does not know these. Type the answers and click **Save & Next**. It will never
     ask them again (saved in this Chrome, see section 5 to move them into `profile.json`).
   - "Review page: check it, then click LinkedIn's Submit yourself." Read the review, then click LinkedIn's
     own **Submit application** button.
3. Press **Alt+W** to close the tab, and move to the next tab.

Shortcuts on a job page: **Alt+A** Easy Apply + fill · **Alt+F** fill this step only · **Alt+N** next ·
**Alt+W** close tab. The **–** button minimises the panel, **↻** reloads the extension from disk.

Naukri search pages get the same batch opener. Naukri application forms are not filled yet.

Keep volumes modest. Opening 5 at a time and reviewing each one is fine; hundreds a day will get the
account flagged by LinkedIn.

---

## 3. Keeping it up to date

The code is in git, your answers are not. The two are updated separately.

**Code: after changing anything on one laptop**
```bash
npm run push          # syncs the extension folder, commits everything, pushes to GitHub
```
**Code: on any other laptop**
```bash
npm run pull          # git pull + node sync-extension.js
```
then click **↻** in the panel. No need to open `chrome://extensions`.

**Answers: after editing `profile.json`**
```bash
node sync-extension.js
```
then click **↻** in the panel. The hint line shows "Answers loaded <time>" so you can confirm.

**Answers: moving them to another laptop** is a manual copy of `profile.json` (section 1.4). Git never
carries it.

---

## 4. The answers bank: `profile.json`

```jsonc
{
  "personal": { "firstName": "...", "phone": "...", "phoneCountryCode": "India (+91)", ... },
  "resumePath": "./resume.pdf",            // used by the old runner only; LinkedIn keeps your resume
  "defaults": { "yesNoDefault": "Yes", "numericDefault": "1", "textDefault": "N/A" },
  "answers": [
    { "match": ["^country$"],               "answer": "India" },
    { "match": ["notice period"],           "answer": "0" },
    { "match": ["current ctc", "current salary"], "answer": "400000" },
    { "match": ["^middle name$"],           "answer": "SKIP" },
    { "match": ["^work from month$"],       "answer": "December" }
  ]
}
```

Rules:

- `match` holds regular expressions, matched case-insensitively against the question label LinkedIn shows.
  **The first matching rule wins**, so keep specific rules above generic ones.
- `@personal.phone` as an answer copies the value from the `personal` block.
- `SKIP` leaves the field empty on purpose.
- `FILL_ME` means "ask me in the panel". That is how the template ships.
- Numeric fields: LinkedIn rejects "4 LPA" or "0 days" with "Invalid input". Give digits only. The helper
  retries with digits if a value is rejected, but a clean number is better.
- Radio and dropdown questions: the answer is matched against the option texts. "Yes"/"No" works for all
  yes/no questions. For ranges such as "1-3 years", a number picks the range that contains it.
- Date fields in the work-experience and education steps are asked as `work from month`,
  `work from year`, `work to month`, `education from year`, and so on. "I currently work here" is ticked
  by the `currently work` rule.

`config.json` has two things the extension uses: `behaviour.skipIfTitleMatches` (the title filter list)
and `behaviour.unfollowCompany` (untick "Follow company" on the last step). Everything else in it is for
the old runner. After editing it run `node sync-extension.js` and click ↻.

---

## 5. Answers typed in the panel

They are stored in this Chrome's extension storage, not in `profile.json`. To see or edit them:
right-click the helper icon → **Options** (or `chrome://extensions` → Easy Apply Helper → Details →
Extension options). The options page lets you:

- edit the learned answers as JSON,
- **Export full bank (learned + profile)** to a download,
- **Reload profile.json from the extension folder**,
- **Clear opened/applied history**,
- toggle auto-Next, auto-start Easy Apply in opened tabs, the title filter and the batch size.

To fold learned answers into `profile.json` permanently (so they survive a Chrome reset and move with the
file):
```bash
node sync-extension.js --import ~/Downloads/profile.json     # the exported file
```
This adds only rules that are not already present.

---

## 6. Sharing it with a friend

The repository is safe to share: it has no personal data, only `profile.example.json`.

- **Collaborator**: GitHub → repo → Settings → Collaborators → add them. They clone the same URL.
- **Fork**: they press *Fork* on GitHub and clone their fork. To receive your updates they add your repo
  once with `git remote add upstream https://github.com/parshuramsde/linkedin-easy-apply.git`, then run
  `git pull upstream main && node sync-extension.js` whenever you push.

Their setup is section 1 with their own `profile.json` made from the template. Tell them:

- `config.json` → `skipIfTitleMatches` is tuned for a 1-year backend profile. Edit it, or untick the
  title-filter checkbox in the panel.
- The helper never clicks Submit. That is always their click.
- Their `profile.json` must stay out of git. The `.gitignore` already handles it.

---

## 7. Tests

```bash
npm install          # once, installs Playwright (about 300 MB)
npm test             # three offline tests, no LinkedIn login needed
```
They check: a mock Easy Apply form fills correctly, the work-experience step gets month/year/checkbox right,
and the batch opener dedupes LinkedIn's doubled job cards. Run them after changing anything in
`extension/filler.js` or `extension/content.js`. The tests use `test/fixture-profile.json`, a fake person,
never your real file.

---

## 8. Troubleshooting

| Symptom | Fix |
|---------|-----|
| No panel on LinkedIn | `chrome://extensions`: is the helper enabled? Reload the page. The panel only appears on `/jobs/...` pages. |
| Hint line says **⚠ profile.json missing** | `node sync-extension.js`, then ↻. |
| Fields filled with old answers after editing `profile.json` | You skipped `node sync-extension.js` or ↻. The hint line shows when answers were loaded. |
| "Invalid input" under a number field | Your answer has letters (e.g. "4 LPA"). Change the rule to digits only. |
| A question is asked every time | Its label changes slightly per job. Add a rule in `profile.json` with a looser regex, e.g. `years? of experience`. |
| Jobs left unopened on the search page | They are counted under "Skipped here": already applied, external apply, or title filter. Untick the title filter to include senior roles. |
| Filling stopped working after a LinkedIn redesign | LinkedIn changed its markup. Run `node test/inspect.js` to dump the current page into `debug/` and fix the selectors in `extension/filler.js`. |
| Chrome says the extension was "corrupted" or disabled it | Remove it and Load unpacked again from the same folder. Learned answers are kept unless you remove the extension entirely. |

---

## 9. Files

```
extension/              the Chrome extension (Load unpacked points here)
  manifest.json         permissions: linkedin.com and naukri.com only
  background.js         seeds storage from profile.json, opens/closes tabs, reload
  filler.js             reads a form step and fills it from the bank
  content.js            the panel, batch opener, auto-Next, learning
  options.html/.js      options page
  profile.json          GENERATED by sync-extension.js, gitignored
  config.json           GENERATED by sync-extension.js
profile.json            your master answers bank, gitignored
profile.example.json    template with FILL_ME, shared
config.json             title filter, unfollow, runner settings
sync-extension.js       copies profile/config into extension/, --import merges exports
test/                   offline tests and LinkedIn inspection helpers
apply.js, filler.js     old unattended Playwright runner, not used day to day (section 10)
```

Not in the repository on purpose: `profile.json`, `extension/profile.json`, `resume.pdf`,
`chrome-profile/`, `debug/`, `applied.jsonl`, `unanswered.jsonl`.

---

## 10. The old runner (`apply.js`)

Before the extension, `node apply.js` drove its own Chrome through Playwright, searched LinkedIn from
`config.json`, and applied unattended. It is kept for reference only. It needs `npm install`, a one-time
`node apply.js --login` to log in to its own Chrome profile (`chrome-profile/`), and it is more likely to be
flagged by LinkedIn because it behaves like a bot. **Use the extension.**
