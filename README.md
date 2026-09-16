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
   download `workstream-agent_<version>_darwin-aarch64.app.tar.gz`.
2. Unpack it and drag **Workstream Agent.app** to `/Applications`.
3. Clear the download flag, or macOS will refuse to open it:

   ```sh
   xattr -dr com.apple.quarantine "/Applications/Workstream Agent.app"
   ```

Step 3 is needed because the app is not yet signed with an Apple Developer ID
certificate. Gatekeeper reports an unsigned downloaded app as "damaged", which
is misleading — nothing is wrong with the file, and its minisign signature can
be checked independently:

```sh
minisign -Vm workstream-agent_<version>_darwin-aarch64.app.tar.gz \
  -x workstream-agent_<version>_darwin-aarch64.app.tar.gz.sig \
  -P <the public key from the agent's tauri.conf.json>
```

This step disappears once the certificate lands.

## Windows

Download and run `workstream-agent_<version>_windows-x86_64.msi` from the
[latest release](https://github.com/free1ancer/workstream-agent-releases/releases/latest).
The MSI is not yet Authenticode-signed, so SmartScreen will warn on first run.

## Intel Macs

Not published yet. Only Apple Silicon (`aarch64`) macOS builds are built.

## Managed fleets

Do not use any of the above. Deploy through Jamf or Intune as described in the
agent's MDM guide: the enrollment token goes out through managed preferences,
the agent enrols itself on first launch, and the first-run wizard never appears.

## After installing

The agent lives in the menu bar. First launch walks through screen access and
sign-in; after that, clicking the icon shows the day at a glance. It updates
itself silently in the background — there is nothing to re-download here.
