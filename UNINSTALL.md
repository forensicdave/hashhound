# Uninstalling HashHound

HashHound is a self‑contained app, so uninstalling is mostly "delete the app." Below is the
quick way, plus a complete removal if you want to clear everything it saved.

## Simple uninstall (the app + Finder extension)

1. **Quit HashHound** (⌘Q), and stop its Finder extension:
   ```sh
   osascript -e 'quit app "HashHound"'; pkill -f HashHoundFinder
   ```
2. **Move `/Applications/HashHound.app` to the Trash.**
   The Finder extension lives *inside* the app bundle, so deleting the app removes it too.
3. **Reload Finder** so the extension unregisters:
   ```sh
   killall Finder
   ```

The extension disappears from **System Settings → General → Login Items & Extensions** once the
app is gone.

## Complete uninstall (also removes saved data)

HashHound leaves a few small support files. To wipe everything:

```sh
# the app + its embedded Finder extension
rm -rf /Applications/HashHound.app

# saved lookup history, debug log, and preferences
rm -rf ~/Library/Application\ Support/HashHound
rm -rf ~/Library/Logs/HashHound
defaults delete com.forensicdave.HashHound 2>/dev/null

# reload Finder
killall Finder
```

## API keys (optional — only if you won't reinstall)

Your VirusTotal / MalwareBazaar / REDS keys are stored in the macOS **Keychain**. The steps above
leave them alone, so a future reinstall keeps working. To remove them as well:

```sh
for a in virustotal-api-key malwarebazaar-api-key reds-api-key; do
  security delete-generic-password -s com.forensicdave.HashHound -a "$a" 2>/dev/null
done
```

Or open **Keychain Access**, search for `HashHound`, and delete the matching entries.

## What HashHound creates (for reference)

| Item | Location |
|---|---|
| The app (incl. Finder extension) | `/Applications/HashHound.app` |
| Lookup history | `~/Library/Application Support/HashHound/` |
| Debug log (if enabled) | `~/Library/Logs/HashHound/` |
| Preferences | `~/Library/Preferences/com.forensicdave.HashHound.plist` |
| API keys | macOS Keychain (service `com.forensicdave.HashHound`) |

Nothing is installed outside your user account, and HashHound runs no background services or
daemons — so once the app and the items above are removed, it's completely gone.
