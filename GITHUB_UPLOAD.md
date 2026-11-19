# Publish on GitHub

Recommended repository name: `xiao-smart-ir-mate-dual-mode`

## Using the GitHub website

1. Sign in and select **New repository**.
2. Enter `xiao-smart-ir-mate-dual-mode`; describe it as “ESPHome Seeed IR learning + Home Assistant HAIR native IR proxy for XIAO Smart IR Mate”.
3. Select **Public** or **Private**, and do **not** auto-create README/license if you will upload this folder's files.
4. Use **Add file → Upload files** to add everything in this folder, including `.gitignore`. (GitHub may hide dotfiles in a file-picker; verify it appears.)
5. Commit changes. Review the file list before publishing: **never commit `secrets.yaml`, real encryption keys, passwords, or local addresses**.

## Using Git CLI

```bash
cd xiao-smart-ir-mate-dual-mode
git init
git add README.md xiao_smart_ir_mate.yaml secrets.example.yaml .gitignore LICENSE ATTRIBUTION.md CHANGELOG.md GITHUB_UPLOAD.md
git commit -m "Initial dual-mode XIAO IR Mate release"
git branch -M main
# Create the empty repo via GitHub's UI first, then replace YOUR_USERNAME below.
git remote add origin https://github.com/YOUR_USERNAME/xiao-smart-ir-mate-dual-mode.git
git push -u origin main
```

Note: GitHub authentication may require a personal access token or SSH key. This bundle does not publish to GitHub automatically.
