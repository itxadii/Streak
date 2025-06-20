# Streak Maintainer: Automated Daily GitHub Contributions

[![Build Status](https://img.shields.io/github/actions/workflow/status/itxadii/Streak/main.yml?label=Build\&style=flat-square)](https://github.com/itxadii/Streak/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](https://opensource.org/licenses/MIT)

Automate your GitHub contribution streak with verified GPG-signed commits. This solution ensures you never miss a day, even if you're busy or on vacation.

## Features

* ✅ Daily automated commits to maintain your streak
* 🔒 Verified GPG-signed commits for authenticity
* ⏰ Triple-redundant scheduling to prevent misses
* ⚙️ Smart commit detection - only commits once per day
* 🌐 Time zone optimized for global reliability

## Why Use This?

* Maintain your GitHub contribution graph (green dots)
* Never lose your streak due to travel, illness, or busy periods
* Learn GitHub Actions and GPG commit signing
* Customizable for any repository
* 100% free and open source

## Prerequisites

* GitHub Account with a public repository
* GitHub Personal Access Token (PAT)

  * Required permissions: `repo` (full control of private repositories)
  * Fine-grained permissions: `Contents: Read and write`
* GPG Key Pair (public key added to GitHub account)

---

## Setup Guide

### Step 1: Generate GPG Key

```bash
# Install GPG if needed
brew install gnupg        # macOS
sudo apt install gnupg    # Ubuntu

# Generate key
gpg --full-generate-key

# Follow prompts:
# 1. Key type: RSA and RSA (default)
# 2. Key size: 4096
# 3. Expiration: 0 = no expiration
# 4. Name: Your Name
# 5. Email: Your GitHub email
# 6. Password: Don't create just skip, otherwise script will not work. 

# Export keys
gpg --armor --export YOUR_KEY_ID > public-gpg-key.asc
gpg --armor --export-secret-keys YOUR_KEY_ID > private-gpg-key.asc
```

Add your **public key** here:
👉 [https://github.com/settings/keys](https://github.com/settings/keys)

---

### Step 2: Create GitHub Secrets

Go to your repository → `Settings > Secrets > Actions`

Create these secrets:

* `GH_PAT`: Your GitHub Personal Access Token
* `GPG_PRIVATE_KEY`: Contents of `private-gpg-key.asc`
* 
---

### Step 3: Add Workflow File

Create `.github/workflows/main.yml`:

```yaml
name: Auto Contribution Commit

on:
  schedule:
    - cron: '0 21 * * *'
    - cron: '30 22 * * *'
    - cron: '50 23 * * *'
  workflow_dispatch:

jobs:
  commit:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GH_PAT }}
          fetch-depth: 0

      - name: Setup Git Identity
        run: |
          git config user.name "${{ github.repository_owner }}"
          git config user.email "YOUR_GITHUB_EMAIL"

      - name: Check for existing commit today
        id: commit_check
        run: |
          LAST_COMMIT_UTC=$(git log -1 --format=%cd --date=format:'%Y-%m-%d')
          TODAY_UTC=$(date -u '+%Y-%m-%d')
          if [ "$LAST_COMMIT_UTC" = "$TODAY_UTC" ]; then
            echo "already_committed=true" >> $GITHUB_OUTPUT
          else
            echo "already_committed=false" >> $GITHUB_OUTPUT
          fi

      - name: Setup GPG Environment
        if: steps.commit_check.outputs.already_committed == 'false'
        run: |
          mkdir -p ~/.gnupg
          chmod 700 ~/.gnupg
          echo "use-agent" >> ~/.gnupg/gpg.conf
          echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf
          echo "allow-loopback-pinentry" >> ~/.gnupg/gpg-agent.conf
          gpgconf --kill gpg-agent

      - name: Import GPG Key
        if: steps.commit_check.outputs.already_committed == 'false'
        env:
          GPG_PRIVATE_KEY: ${{ secrets.GPG_PRIVATE_KEY }}
          GPG_PASSPHRASE: ${{ secrets.GPG_PASSPHRASE }}
        run: |
          echo "$GPG_PRIVATE_KEY" > private.key
          gpg --batch --passphrase "$GPG_PASSPHRASE" --import private.key
          rm private.key
          KEY_ID=$(gpg --list-secret-keys --with-colons | awk -F: '$1 == "sec" {print $5}' | head -n1)
          git config --global user.signingkey "$KEY_ID"
          git config --global commit.gpgsign true

      - name: Update contribution file
        if: steps.commit_check.outputs.already_committed == 'false'
        run: |
          echo "🟢 $(date -u)" >> AUTOMATED_CONTRIBUTIONS.md
          echo "Verified commit: $(date -u '+%Y-%m-%d %H:%M:%S %Z')" >> AUTOMATED_CONTRIBUTIONS.md

      - name: Commit and Push
        if: steps.commit_check.outputs.already_committed == 'false'
        env:
          GPG_PASSPHRASE: ${{ secrets.GPG_PASSPHRASE }}
        run: |
          git add AUTOMATED_CONTRIBUTIONS.md
          git commit -S -m "chore: auto commit at $(date -u)" || exit 0
          git push
```

---

### Step 4: Create Contribution File

Create `AUTOMATED_CONTRIBUTIONS.md`:

```markdown
# Automated Daily Contributions

This file maintains my GitHub contribution streak with verified commits.

- Project: https://github.com/itxadii/Streak
- Last update: $(date -u)
```

---

## Customization

### Change Schedule Times

```yaml
on:
  schedule:
    - cron: '0 21 * * *'  # 9 PM UTC
    - cron: '30 22 * * *' # 10:30 PM UTC
    - cron: '50 23 * * *' # 11:50 PM UTC
```

### Use Different File

```yaml
- name: Update contribution file
  run: |
    echo "🟢 $(date -u)" >> YOUR_FILENAME.md
```

---

## Safety Features

* Triple Execution Window: Three attempts per day
* Smart Detection: Only commits once per day
* UTC Time: Avoids daylight saving issues
* Verified Commits: GPG-signed for authenticity
* Secure Secrets: Encrypted storage of credentials

---

## Troubleshooting

| Issue                | Solution                                   |
| -------------------- | ------------------------------------------ |
| Commits not verified | Verify GPG key is added to GitHub account  |
| Workflow fails       | Check Actions tab for error details        |
| No commits made      | Verify PAT has write permissions           |
| Multiple commits     | Check timezone conversion in cron schedule |

### Contribution Graph Impact

![Graph Example](https://user-images.githubusercontent.com/51070104/174474720-3f2a1e0a-3b70-4c34-8d95-0e8d0d5d0c8a.png)

*Note: GitHub may take 24–48 hours to update contribution graphs.*

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Disclaimer

This project automates GitHub contributions for streak maintenance. Use it responsibly and in compliance with GitHub's Terms of Service.

---

⭐ Give this repo a star if you find it helpful 💖
