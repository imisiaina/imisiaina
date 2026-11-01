# Upswing for iOS

Upswing is a one-tap arcade climber. You swing from glowing rings, tap to let go, and grab the next ring as you fly past it, while a tide rises from below.

This folder contains the iPhone app. The game itself is a single web page (`www/index.html`), and [Capacitor](https://capacitorjs.com) runs it inside a native iOS app. That gives the game real haptics, saves that survive app updates, and room to add Android later from the same code.

## What you need

- A Mac with the latest **Xcode** from the Mac App Store
- **Node.js 20 or newer** (`brew install node`, or the installer from nodejs.org)
- To run it on your own iPhone: a free Apple ID. To publish on the App Store: a paid [Apple Developer Program](https://developer.apple.com/programs/) membership.

## Run it

```bash
cd upswing
npm install          # gets Capacitor and the iOS plugins
npm run ios          # copies the game into the Xcode project and opens Xcode
```

In Xcode:

1. Select the **App** target, open **Signing & Capabilities**, and choose your **Team**. If Xcode says the bundle ID `com.imisiaina.upswing` is taken, change it to something unique to you.
2. Choose an iPhone simulator, or your iPhone connected by cable, in the toolbar.
3. Press **Run** (⌘R).

The first build takes a minute while Xcode downloads Capacitor through Swift Package Manager.

> On a real iPhone, the first launch may show "Untrusted Developer". Open **Settings › General › VPN & Device Management**, trust your developer certificate, and launch it again. You may also need to turn on **Developer Mode** (Settings › Privacy & Security).

## Change the game

1. Edit `www/index.html`. You can test it in any browser by opening the file directly.
2. Run `npm run sync` to copy your changes into the iOS project.
3. Press **Run** in Xcode again.

The values that control difficulty are near the top of the script:

| Constant | What it controls |
| --- | --- |
| `minWindow(i)` | The shortest tap window (in ms) the level generator allows for ring *i*. Raise it to make the game easier. |
| `leapChance(i)` | How often a ring is a real leap instead of a breather (a ring whose catch zone brushes your swing, so you can tap as you pass) |
| `grabR(i)` | Catch radius around each ring. It shrinks slowly as you climb. |
| `12 + score*0.7` in `update()` | How fast the tide rises, after a 4-second head start |
| `guide` in `draw()` | The guide path and ring highlight, shown fully until ring 6 and faded out by ring 12 |
| `BOOST`, `AMP` | Jump strength and swing height. The generator measures every ring against these, so changing them stays fair. |
| `SKINS` | Ember colours and how many gems unlock each one |

Before placing each ring, the generator measures its tap window with `releaseWindow()`. That's the stretch of each swing during which letting go actually lands you on the ring. No ring is ever placed with a window below `minWindow`.

## What's iOS-specific

- **Haptics:** light taps when you let go and grab, a stronger tap on a perfect grab, a heavy tap when a cracked ring snaps, and an error buzz when you die. This uses `@capacitor/haptics`.
- **Saves:** your best score, gems and chosen colour are stored with `@capacitor/preferences`, so app updates don't erase them.
- **Full screen:** portrait only, no status bar, and the home indicator fades out. Swipes near the top and bottom edges go to the game first, so a tap there doesn't leave the app.
- **Sound:** game sounds mix with whatever music the player is listening to and follow the silent switch. If a phone call interrupts the game, sound comes back on the next tap.
- **Works offline:** the fonts are bundled in the app (both use the SIL Open Font License).
- **iPhone only:** on iPad, the app runs in iPhone compatibility mode.

## Put it on the App Store

1. In [App Store Connect](https://appstoreconnect.apple.com), create a new app with the same bundle ID.
2. In Xcode, choose **Any iOS Device (arm64)**, then **Product › Archive**, then **Distribute App › App Store Connect**.
3. Test the build with friends through **TestFlight**, then submit it for review.

The app icon (`ios/App/App/Assets.xcassets/AppIcon.appiconset`) and the launch screen are already set up. The app declares that it uses no non-exempt encryption, so App Store Connect won't ask about export compliance.

## Project layout

```
upswing/
├── www/                 the game: index.html, fonts, Capacitor runtime
├── ios/App/             Xcode project (open App.xcodeproj via `npm run open`)
│   └── App/
│       ├── SceneDelegate.swift   full-screen game view controller
│       ├── AppDelegate.swift     audio session setup
│       └── Info.plist            portrait-only, hidden status bar
├── capacitor.config.json
└── package.json
```
