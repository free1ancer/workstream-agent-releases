# Workstream Agent — releases

Downloads for the Workstream desktop agent. The agent is built from
[free1ancer/workstream-app](https://github.com/free1ancer/workstream-app); this
repository exists only to serve release assets, which is why it is public while
the source is not.

Every payload is signed with minisign. The agent's own auto-updater verifies
that signature before installing anything, and so does the installer below.

## macOS (Apple Silicon) — one line

```sh
curl -fsSL https://github.com/free1ancer/workstream-agent-releases/releases/latest/download/install.sh | sh
```

It downloads the latest release, **verifies its signature, and refuses to
install if the check fails**, then puts the app in `/Applications` and starts
it. It needs `minisign`:

```sh
brew install minisign
```

The installer stops with instructions rather than proceeding if `minisign` is
missing. That is deliberate — installing a binary you cannot verify is the thing
signing exists to prevent.

To install a specific version rather than the latest:

```sh
curl -fsSL https://github.com/free1ancer/workstream-agent-releases/releases/latest/download/install.sh | WORKSTREAM_VERSION=1.0.1 sh
```

## macOS — by hand

1. From the
   [latest release](https://github.com/free1ancer/workstream-agent-releases/releases/latest),
   download `workstream-agent_<version>_darwin-aarch64.dmg`.
2. Open it and drag **Workstream Agent.app** to `/Applications`.

That is the whole procedure. The app is signed with an Apple Developer ID
certificate, notarized by Apple and stapled, so it opens on a double-click:
there is no quarantine flag to clear, no right-click-Open, and no "damaged"
dialog. If you ever see one, something is wrong with the download — check it
rather than working around it:

```sh
spctl --assess -vv "/Applications/Workstream Agent.app"
```

should say `source=Notarized Developer ID`.

You can also verify the update payload independently of Apple, the way the
agent's own updater does:

```sh
minisign -Vm workstream-agent_<version>_darwin-aarch64.app.tar.gz \
  -x workstream-agent_<version>_darwin-aarch64.app.tar.gz.sig \
  -P <the public key from the agent's tauri.conf.json>
```

### 1.0.3: re-grant once

1.0.3 is the first signed release, and macOS keys privacy grants and keychain
items to an app's signature. Upgrading from an unsigned build therefore looks
like a different app to the system exactly once:

- **Screen Recording** has to be granted again — the agent notices and walks you
  through it. If Workstream is still listed in System Settings from the old
  build, remove it with the **−** button and add it back.
- You may be **asked to sign in again**, because the stored token belonged to
  the old signature.

Both are one-time. From 1.0.3 onwards the signature is stable, so grants and
sign-in survive every update.

## Windows

Download and run `workstream-agent_<version>_windows-x86_64.msi` from the
[latest release](https://github.com/free1ancer/workstream-agent-releases/releases/latest).
The MSI is not yet Authenticode-signed, so SmartScreen will warn on first run.
(The macOS build is signed and notarized; the Windows certificate has not landed
yet.)

## Intel Macs

Not published yet. Only Apple Silicon (`aarch64`) macOS builds are built.

## Managed fleets

Do not use any of the above. Deploy through Jamf or Intune as described in the
agent's MDM guide: the enrollment token goes out through managed preferences,
the agent enrols itself on first launch, and the first-run wizard never appears.

## After installing

The agent lives in the menu bar. First launch walks through screen access and
sign-in; after that, clicking the icon shows the day at a glance. It updates
itself silently in the background — there is nothing to re-download here, and
updates keep your Screen Recording grant.
