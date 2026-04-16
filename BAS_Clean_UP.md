commands:

run the below command and paste it in the copilot or any other ai with context.


df -h /home

****************************************************

echo "== Top level in HOME =="
du -xh --max-depth=1 "$HOME" 2>/dev/null | sort -h

echo "== Biggest under projects (2 levels) =="
du -xh --max-depth=2 "$HOME/projects" 2>/dev/null | sort -h | tail -n 40

echo "== Known cache suspects =="
du -sh \
  "$HOME/.vscode-server" \
  "$HOME/.vscode-server-insiders" \
  "$HOME/.vscode" \
  "$HOME/.local/share" \
  "$HOME/.hanatools" \
  "$HOME/.fioritools" \
  "$HOME/.asdf-inst" \
  "$HOME/.webide" \
  2>/dev/null || true



  *************************************************




2A) Clear cached VSIX + logs (Stable + Insiders + local)

****************************************************

rm -rf "$HOME/.vscode-server/data/CachedExtensionVSIXs/"* 2>/dev/null || true
rm -rf "$HOME/.vscode-server/data/logs/"* 2>/dev/null || true

rm -rf "$HOME/.vscode-server-insiders/data/CachedExtensionVSIXs/"* 2>/dev/null || true
rm -rf "$HOME/.vscode-server-insiders/data/logs/"* 2>/dev/null || true

rm -rf "$HOME/.vscode/data/logs/"* 2>/dev/null || true
rm -rf "$HOME/.vscode/data/CachedProfilesData/"* 2>/dev/null || true

********************************************************




2B) Remove old VS Code server versions (keep newest only)


*********************************************************


# list by newest first
ls -1dt "$HOME/.vscode-server/cli/servers/Stable-"* 2>/dev/null | head -n 5

# delete all but newest
ls -1dt "$HOME/.vscode-server/cli/servers/Stable-"* 2>/dev/null \
  | tail -n +2 | xargs -r rm -rf


  ls -1dt "$HOME/.vscode-server/code-"* 2>/dev/null | head -n 5
ls -1dt "$HOME/.vscode-server/code-"* 2>/dev/null \
  | tail -n +2 | xargs -r rm -rf


  ls -1dt "$HOME/.vscode-server-insiders/cli/servers/Insiders-"* 2>/dev/null | head -n 5
ls -1dt "$HOME/.vscode-server-insiders/cli/servers/Insiders-"* 2>/dev/null \
  | tail -n +2 | xargs -r rm -rf

ls -1dt "$HOME/.vscode-server-insiders/code-insiders-"* 2>/dev/null | head -n 5
ls -1dt "$HOME/.vscode-server-insiders/code-insiders-"* 2>/dev/null \
  | tail -n +2 | xargs -r rm -rf

  **********************************************************






  3) Big win #2: node_modules (likely the main culprit)

  3A) Measure node_modules total size

  ***********************************************

  echo "== node_modules directories =="
find "$HOME/projects" -type d -name node_modules -prune 2>/dev/null | wc -l

echo "== size per repo (top 20) =="
find "$HOME/projects" -maxdepth 3 -mindepth 2 -type d 2>/dev/null \
  | while read -r d; do
      [ -d "$d/node_modules" ] && du -sh "$d/node_modules"
    done | sort -h | tail -n 20

*************************************************


3B) Clean build caches (safe) without deleting node_modules
***********************************************************


find "$HOME/projects" -type d -path "*/node_modules/.cache" -prune -exec rm -rf {} + 2>/dev/null || true





[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[ This shoue be folder specefic  ]]]]]]]]]]]]]]]]]]]]]]]]]]]]]]]]

rm -rf "$HOME/projects/<externalServicesConnections>/app/deep-dev/node_modules" 2>/dev/null || true
rm -rf "$HOME/projects/<service_used_for_externalservice-through-destination>/app/react-student-demo/node_modules" 2>/dev/null || true
************************************************************




df -h /home
du -xh --max-depth=2 "$HOME/.hanatools" 2>/dev/null | sort -h | tail -n 30
du -sh "$HOME/.hanatools/aitoolkit/venv" 2>/dev/null || true
rm -rf "$HOME/.hanatools/aitoolkit/venv" 2>/dev/null || true


find "$HOME/.hanatools" -type d -name "__pycache__" -prune -exec rm -rf {} + 2>/dev/null || true
find "$HOME/.hanatools" -type f -name "*.pyc" -delete 2>/dev/null || true


du -xh --max-depth=2 "$HOME/projects/<externalServicesConnections>/app" 2>/dev/null | sort -h | tail -n 30
du -xh --max-depth=2 "$HOME/projects/<service_used_for_externalservice-through-destination>/app" 2>/dev/null | sort -h | tail -n 30


find "$HOME/projects" -type d -path "*/node_modules/.cache" -prune -exec rm -rf {} + 2>/dev/null || true
find "$HOME/projects" -type d -path "*/.cache" -prune -exec rm -rf {} + 2>/dev/null || true

find "$HOME/projects" -maxdepth 6 -type d \( -name "dist" -o -name "build" -o -name ".next" -o -name "target" \) \
  -prune -exec rm -rf {} + 2>/dev/null || true


  rm -rf "$HOME/projects/<externalServicesConnections>/mta_archives" 2>/dev/null || true
rm -rf "$HOME/projects/<service_used_for_externalservice-through-destination>/mta_archives" 2>/dev/null || true

rm -rf "$HOME/.fioritools/module-cache" 2>/dev/null || true
rm -rf "$HOME/.fioritools/search-cache" 2>/dev/null || true

rm -rf "$HOME/.local/share/vscode-sqltools/node_modules/cacache" 2>/dev/null || true

du -xh "$HOME" 2>/dev/null | sort -h | tail -n 50















rm -rf "$HOME/.vscode-server-insiders" 2>/dev/null || true
rm -rf "$HOME/.vscode-server/data/CachedExtensionVSIXs/"* 2>/dev/null || true
rm -rf "$HOME/.vscode-server/data/logs/"* 2>/dev/null || true

rm -rf "$HOME/.hanatools/aitoolkit/venv" 2>/dev/null || true

find "$HOME/projects" -type d -path "*/node_modules/.cache" -prune -exec rm -rf {} + 2>/dev/null || true

df -h /home



du -xh --max-depth=2 "$HOME/.hanatools" 2>/dev/null | sort -h | tail -n 20






