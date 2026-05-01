---
title: "Injecting Browser Extension Without the Store - Introducing stomp.py"
date: 2026-05-02 00:33:00 +0200
categories: [MalDev, Research]
tags: [offensive-security, browser, bypass]
---

Browser extensions are a massively underestimated attack surface. They run natively in the browser, have access to cookies, network requests, and every page the user visits, and most EDRs don't even look at them. Browser extensions can be leveraged as a real threat and act as a malicious agent.

In this post, I want to introduce **stomp**, a tool I developed during my red team research that automates the injection of a browser extension into Chromium-based browsers, bypassing GPO restrictions in the process. This post covers the technique, the tooling, and the underlying mechanics.

---

## Prior Work - Standing on Synacktiv's Shoulders

This work builds on the research published by Synacktiv in their article [*L'extension fantôme*](https://www.synacktiv.com/publications/lextension-fantome-infiltrer-chrome-par-des-voies-inexplorees), which first explored the concept of injecting a browser extension by manipulating the `Secure Preferences` file. Their tool, **extloader**, demonstrated that it was possible to craft a valid `Secure Preferences` with a forged HMAC, bypassing Chrome's integrity check.

stomp.py takes this foundation and extends it with automation, GPO bypass, and a complete deployment workflow designed for red team operations.

---

## The Secure Preferences File

Every Chromium-based browser maintains a `Secure Preferences` file in the user's profile directory. This JSON file stores the browser's configuration, including the list of installed extensions, their permissions, and their paths on disk.

```
# Microsoft Edge
C:\Users\<user>\AppData\Local\Microsoft\Edge\User Data\Default\Secure Preferences

# Google Chrome
C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Secure Preferences
```

The file is signed with an HMAC computed over its content. On every launch, the browser verifies this signature. If it doesn't match, the browser rejects the modifications or reinitializes the file, meaning you can't just edit the JSON directly.

This is where the injection technique comes in: instead of modifying the file manually, we generate a new valid `Secure Preferences` that includes our extension, with a correctly computed HMAC.

The HMAC computation depends on two inputs: the file content (Secure Preferences), and the user's SID (required to compute HMAC). This is why stomp requires the SID as a parameter, without it, the generated file will be rejected by the browser.

---

## What stomp.py Does

stomp.py takes your extension folder, the current `Secure Preferences` file from the target, and the user's SID, and produces a deployment-ready package.

```bash
python3 stomp.py EXTENSION_A/ \
  --prefs-file SecurePreferences \
  --sid "S-1-5-21-...-...-...-..." \
  --target-dir "C:\\Users\\<user>\\AppData\\Local" \
  --debug
```

The output is a ZIP archive containing everything needed for deployment:

```
output.zip
├── extension/               # The extension folder, ready to copy
├── info.json                # Extension metadata and deployment info
├── inject.bat               # Automated deployment script
├── preferences/             # Generated Secure Preferences for Chrome, Edge, Brave
└── SecurePreferencesClean   # Backup of the original Secure Preferences
```

The `inject.bat` script handles the full deployment sequence:

1. Kill the browser process
2. Restore the clean `Secure Preferences` to avoid conflicts with previous injections
3. Open and close the browser to reset its internal state
4. Copy the extension folder to `%LOCALAPPDATA%`
5. Overwrite `Secure Preferences` with the generated file

On next launch, the browser reads the `Secure Preferences`, finds the extension ID, resolves the path, and loads the extension; no user interaction, no store, no warning dialog.

---

## Bypassing GPO Whitelists - The `--spoof` Option

Enterprise environments often go further than just blocking manual installs. Some GPO configurations restrict the browser to a whitelist of allowed extension IDs (`ExtensionInstallAllowlist`). These IDs are visible at `edge://policy` or `chrome://policy`.

An extension's ID is derived from its public key. stomp's `--spoof` option exploits this: it fetches the public key of a whitelisted extension from the Microsoft Store or Chorme Store and injects it into the manifest of the extension being deployed. The injected extension ends up with the same ID as the legitimate whitelisted one.

```bash
python3 stomp.py EXTENSION_A/ \
  --spoof nmhdhpibnnopknkmonacoephklnflpho \
  --prefs-file SecurePreferences \
  --sid "S-1-5-21-...-...-...-..." \
  --target-dir "C:\\Users\\<user>\\AppData\\Local" \
  --debug
```

The browser loads the malicious extension thinking it's loading the whitelisted one. GPO restrictions never trigger because the ID matches.

---

## Limitations

stomp is not a magic bullet. A few things to keep in mind:

- **The extension folder is visible on disk.** After injection, the extension folder sits in `%LOCALAPPDATA%`. This is an artifact that can be detected by a SOC analyst or a forensic tool. Addressing this is a separate research track.
- **The SID is required.** You need the target user's SID to generate a valid HMAC. In a red team context, this is typically trivial to obtain, but it is a prerequisite.
- **Tested on Chrome, Edge, Brave, Vivaldi.** Opera and other Chromium forks use a similar structure but may differ in their HMAC implementation.
- **MV3 constraints apply.** The injected extension is still subject to Manifest V3 restrictions; no persistent background page, no `webRequestBlocking`. Plan your extension's architecture accordingly.

---

## What's Next

stomp.py will be released on my GitHub this weekend. The repository will include the script, usage documentation, and example extension structures for testing.

This post covers the injection primitive. Further research on persistence and stealth is ongoing, more on that soon.

---

*Follow me on [GitHub](https://github.com/Fir3n0x) and [LinkedIn](https://www.linkedin.com/in/corentin-mahieu/) for updates.*
