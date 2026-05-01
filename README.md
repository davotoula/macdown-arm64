
# MacDown
This is a port of Macdown for Mac Silicon (arm64). The upstream repo is for x86 which Apple plans to sunset by 2026 or so.

MacDown is an open source Markdown editor for OS X, released under the MIT License. The author was inspired by [Chen Luo](https://twitter.com/chenluois)’s [Mou](http://mouapp.com).

For Mac Silicon (arm64), the [latest releases are here](https://github.com/tikkal/macdown-arm64/releases). 

For x86 builds, visit the [project site](http://macdown.uranusjr.com/) for more information.

## Changes in this fork

This fork (`davotoula/macdown-arm64`) is built on top of `tikkal/macdown-arm64` and adds the following security and hygiene fixes. See [PR #1](https://github.com/davotoula/macdown-arm64/pull/1) for the full diff.

### Sparkle auto-update disabled

The original repository ships with Sparkle 1.x configured to fetch updates from the upstream maintainer's server (`macdown.uranusjr.com`) and verify them against the upstream DSA public key. The `tikkal` fork did not change this configuration, which meant any binary built from that fork would silently auto-update from a server the fork has no control over — and would replace the arm64-only binary with upstream's x86_64 build.

This fork removes the entire Sparkle integration:

- `SUFeedURL`, `SUBetaFeedURL`, and `SUPublicDSAKeyFile` removed from `MacDown-Info.plist`.
- `Sparkle` pod removed from `Podfile` and `Podfile.lock`.
- `Sparkle.framework` no longer embedded; the `[CP] Embed Pods Frameworks` build phase has nothing to embed and is dropped by `pod install`.
- `<customObject customClass="SUUpdater">` and the **Check for Updates…** menu item removed from `MainMenu.xib`.
- Orphaned **Include pre-releases** checkbox removed from `MPGeneralPreferencesViewController.xib`.
- `#import <Sparkle/SUUpdater.h>` and the `feedURLStringForUpdater:` delegate method removed from `MPMainController.m`.

The `MPPreferences.updateIncludesPreReleases` `@dynamic` property is intentionally retained — it is harmless dead state in `NSUserDefaults`, and removing it would invalidate stored defaults for no security benefit.

**Updates are no longer automatic.** To get a newer build, re-download from the [Releases page](https://github.com/davotoula/macdown-arm64/releases).

### Other cleanup

- Deleted the empty `MacDown/Resources/Styles/GitHub-2020.css` placeholder.

### Build verification

The branch was verified with `pod install` (no Sparkle in the resolved graph), `xcodebuild … -arch arm64 build` (BUILD SUCCEEDED), and `xcodebuild … test` (20/20 tests pass). The produced `MacDown.app` contains no `Frameworks/` directory, no Sparkle symbols in the binary, and no `SU*` keys in the bundled `Info.plist`.

### Known follow-ups not in this fork

- If auto-update is ever re-enabled, migrate to **Sparkle 2.x** (EdDSA signatures, sandboxed updater XPC) and a fork-controlled appcast URL with a freshly generated key pair. Sparkle 1.x's DSA signing is deprecated.
- `MACOSX_DEPLOYMENT_TARGET = 14.6` (inherited from `tikkal`) excludes arm64 Macs running macOS 11–13. Lowering it to `11.0` would cover all arm64-capable hardware. This is a policy decision, not a security fix.
- Translation strings for the removed XIB IDs in `Localization/*/MainMenu.strings` and `Localization/*/MPGeneralPreferencesViewController.strings` are now dangling. Cosmetic only — `ibtool` may warn, but the build still succeeds.

## Install

[Download](http://macdown.uranusjr.com/download/latest/), unzip, and drag the app to Applications folder. MacDown is also available through [Homebrew Cask](https://caskroom.github.io/):

    brew install --cask macdown

## Screenshot

![screenshot](assets/screenshot.png)

## License

MacDown is released under the terms of MIT License. You may find the content of the license [here](http://opensource.org/licenses/MIT), or inside the `LICENSE` directory.

You may find full text of licenses about third-party components in the `LICENSE` directory, or the **About MacDown** panel in the application.

The following editor themes and CSS files are extracted from [Mou](http://mouapp.com), courtesy of Chen Luo:

* Mou Fresh Air
* Mou Fresh Air+
* Mou Night
* Mou Night+
* Mou Paper
* Mou Paper+
* Tomorrow
* Tomorrow Blue
* Tomorrow+
* Writer
* Writer+
* Clearness
* Clearness Dark
* GitHub
* GitHub2

## Development

### Requirements

If you wish to build MacDown yourself, you will need the following components/tools:

* OS X SDK (10.14 or later)
* Git
* [Bundler](http://bundler.io)

> Note: Old versions of CocoaPods are not supported. Please use Bundler to execute CocoaPods, or make sure your CocoaPods is later than shown in `Gemfile.lock`.

> Note: The Command Line Tools (CLT) should be unnecessary. If you failed to compile without it, please install CLT with
>
>     xcode-select --install
>
> and report back.

An appropriate SDK should be bundled with Xcode 5 or later versions.

### Environment Setup

After cloning the repository, run the following commands inside the repository root (directory containing this `README.md` file):

    git submodule update --init
    bundle install
    bundle exec pod install
    make -C Dependency/peg-markdown-highlight

and open `MacDown.xcworkspace` in Xcode. The first command initialises the dependency submodule(s) used in MacDown; the second one installs dependencies managed by CocoaPods.

Refer to the official guides of Git and CocoaPods if you need more instructions. If you run into build issues later on, try running the following commands to update dependencies:

    git submodule update
    bundle exec pod install

### Translation

Please help translation on [Transifex](https://www.transifex.com/macdown/macdown/).

![Transifex translation percentage](https://www.transifex.com/projects/p/macdown/resource/macdownxliff/chart/image_png/)

## Discussion

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/MacDownApp/macdown)

Join our [Gitter channel](https://gitter.im/MacDownApp/macdown) if you have any problems with MacDown. Any suggestions are welcomed, too!

You can also [file an issue directly](https://github.com/MacDownApp/macdown/issues/new) on GitHub if you prefer so. But please, **search first to make sure no-one has reported the same issue already** before opening one yourself. MacDown does not update in your computer immediately when we make changes, so something you experienced might be known, or even fixed in the development version.

MacDown depends a lot on other open source projects, such as [Hoedown](https://github.com/hoedown/hoedown) for Markdown-to-HTML rendering, [Prism](http://prismjs.com) for syntax highlighting (in code blocks), and [PEG Markdown Highlight](https://github.com/ali-rantakari/peg-markdown-highlight) for editor highlighting. If you find problems when using those particular features, you can also consider reporting them directly to upstream projects as well as to MacDown’s issue tracker. I will do what I can if you report it here, but sometimes it can be more beneficial to interact with them directly.

## Tipping

If you find MacDown suitable for your needs, please consider [giving me a tip through PayPal](http://macdown.uranusjr.com/faq/#donation). Or, if you prefer to buy me a drink *personally* instead, just [send me a tweet](https://twitter.com/uranusjr) when you visit [Taipei, Taiwan](http://en.wikipedia.org/wiki/Taipei), where I live. I look forward to meeting you!

