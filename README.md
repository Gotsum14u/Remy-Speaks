#!/data/data/com.termux/files/usr/bin/bash
# Remy's Thoughts - build the site from setup.sh and push to GitHub Pages
set -e

SETUP="${SETUP:-$HOME/setup.sh}"
REPO_URL="${1:-https://github.com/otsum14u/otsum14u.github.io.git}"
SRC="$HOME/remys-deploy"

[ -f "$SETUP" ] || { echo "Can't find $SETUP"; exit 1; }
pkg install -y git >/dev/null

rm -rf "$SRC" && mkdir -p "$SRC/.github/workflows" && cd "$SRC"

# 1. Extract the app files from the heredocs in setup.sh
awk '
/^cat > [^ ]+ << .EOF.$/ { f = $3; next }
/^EOF$/                  { f = ""; next }
f != ""                  { print > f }
' "$SETUP"

# 2. GitHub Pages extras
touch .nojekyll

cat > README.md << 'EOF'
# Remy's Thoughts

A 3D black pug who says what's on his mind and answers your questions.

**Live site:** https://otsum14u.github.io

## Features
- Three.js 3D scene with soft shadows and tone mapping
- Synthesized barks and text-to-speech answers
- Voice input (Chrome) and text input
- Installable PWA that works offline after the first visit

## Run locally
Open `index.html` through any web server, or use `python -m http.server 8080`.
EOF

cat > .gitignore << 'EOF'
.DS_Store
node_modules/
EOF

cat > .github/workflows/pages.yml << 'EOF'
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: .
      - id: deployment
        uses: actions/deploy-pages@v4
EOF

# 3. Commit and push
git init -b main >/dev/null
git add -A
git -c user.name="Remy Deploy" -c user.email="remy@example.com" \
  commit -m "Deploy Remy's Thoughts" >/dev/null
git remote add origin "$REPO_URL"
echo ""
echo "Pushing to $REPO_URL"
echo "When asked for a password, paste a GitHub personal access token (repo scope)."
git push -f origin main

echo ""
echo "✅ Pushed. Next: enable Pages in the repo (see Step 2)."
