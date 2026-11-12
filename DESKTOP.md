# Desktop integration

**Supported platform: Arch Linux + Hyprland only.** The standalone tool uses slurp/grim. The optional integration targets the inspected end-4 QuickShell ii / Lua Hyprland setup; it is not an installer for arbitrary desktop versions.

## A standalone Hyprland shortcut

Finish [setup](README.md#install) and test `./screen-ocr --select` first. Keep the repo in a stable location and optionally create the `~/.local/bin/screen-ocr` symlink described in the [guide](GUIDE.md).

Inspect existing shortcuts with `hyprctl binds` before choosing a free chord. On a conventional `hyprland.conf` configuration, a custom binding can look like:

```ini
bind = SUPER SHIFT, X, exec, ~/.local/bin/screen-ocr --select
```

On the Lua configuration used by this setup, use the equivalent custom binding with an absolute launcher path:

```lua
hl.bind("SUPER + SHIFT + X", hl.dsp.exec(os.getenv("HOME") .. "/.local/bin/screen-ocr --select"))
```

Choose the syntax your installed Hyprland actually uses. If the home path contains spaces, quote the command's executable path. Do not install a duplicate Super+Shift+X binding on end-4; use the next section instead. Keep personal bindings in your desktop's custom config, save a backup first, reload the config, and check `hyprctl configerrors`. To undo a manual binding, remove only the line you added and reload.

## Existing QuickShell ii / end-4 OCR action

This preserves the existing region selector and routes its shared screenshot OCR action to this helper. It also updates the existing Super+Shift+X fallback to call the same tool when QuickShell is unavailable.

The manifest includes exact supported versions of:

- `quickshell/ii/modules/common/utils/ScreenshotAction.qml`
- `hypr/hyprland/keybinds.lua`

Installed assets live under `~/.config/hypr/custom/ocr/` and `~/.config/hypr/custom/scripts/screen-ocr`. The installer reads and checks complete file contents, preserves other bindings in recognized versions, refuses unknown edits/concurrent changes, backs up privately, and rolls back partial writes. Do not copy manifest templates over your config manually.

From the cloned repo, while logged into the desktop:

```bash
python3 -B test_ocr.py
python3 customizations.py check --config-root "$HOME/.config"
```

`check` is read-only. If it refuses your current configuration, stop and adapt the integration against that version; do not whitelist unknown hashes just to bypass the guard. This manifest comes from a customized end-4 installation, not every fresh upstream install.

After setup, reviewing the check and saving unsent shell drafts:

```bash
python3 customizations.py install --config-root "$HOME/.config"
hyprctl reload config-only
hyprctl configerrors
```

Keep the printed backup path under `~/.local/state/hypr-ocr-backups/`. The installer does not restart the desktop or install a system service. The original desktop hot-reloaded the shared action; verify that behaviour on the target setup. Select a harmless text region with Super+Shift+X, inspect the copied output, and check that cancelling selection preserves the previous clipboard.

## Restore

Use the actual path printed by your installation:

```bash
python3 customizations.py restore --config-root "$HOME/.config" --backup /path/printed/by/install
hyprctl reload config-only
hyprctl configerrors
```

Restore refuses files edited since installation. It restores the previous shared action and fallback, including the original Tesseract route in the supported source snapshot. Keep Tesseract and ImageMagick installed if you rely on that fallback. The isolated models/venv can remain unused after rollback.

After deliberate end-4 updates/reinstallation, review the new files and run `check` before reapplying. Git pulls alone do not install this integration. Unknown upstream changes need a reviewed adaptation.
