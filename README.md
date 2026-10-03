# dockhide

Hide any macOS app's Dock icon, and put it back later.

Some apps (menu bar utilities, docking station helpers, sync agents) insist on
a Dock icon you never click. `dockhide` sets
[`LSUIElement`](https://developer.apple.com/documentation/bundleresources/information-property-list/lsuielement)
in the app's `Info.plist`, which tells macOS to treat it as an agent app (no Dock
icon, no app switcher entry), then ad-hoc re-signs the bundle so it still launches.

## Install

```sh
brew install zackwag/tap/dockhide
```

Or copy [`dockhide`](dockhide) anywhere on your `PATH`.

## Usage

```sh
dockhide hide "CalDigit_Docking_Station_Utility"   # by name (searched in /Applications, ~/Applications)
dockhide hide /Applications/Foo.app                # or by path
dockhide status Foo                                # hidden / visible, and whether dockhide did it
dockhide show Foo                                  # restore the original Dock behavior
dockhide list                                      # every app dockhide has hidden
```

`hide`, `show` and `status` take several apps at once. `sudo` is used only
when the app bundle isn't writable by you (e.g. apps installed by a `.pkg`).
Quit and reopen a running app for the change to take effect.

## Things to know

- **App updates undo it.** An update replaces `Info.plist`, so the Dock icon
  comes back. Run `dockhide hide` again; `dockhide list` won't show apps whose
  update wiped the change, so keep your own list (or a script) of what you hide.
- **The app's signature becomes ad hoc.** Editing `Info.plist` breaks the
  developer's signature, so `dockhide` re-signs with `codesign --sign -`. Apps
  that rely on entitlements tied to their developer signature (iCloud, some
  keychain access) may misbehave afterwards; `dockhide show` doesn't restore
  the original signature, but reinstalling the app does.
- **Refused apps:** Mac App Store apps (re-signing stops them launching) and
  system apps under `/System` (protected by SIP).
- `show` restores the exact original `LSUIElement` value, which `hide` records in
  a `DockhideOriginalLSUIElement` key in the app's `Info.plist`. Apps hidden
  some other way (by hand, or by the app itself) are left alone.

## License

[MIT](LICENSE)
