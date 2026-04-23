# Mobile Development — Lesson 2: React Native Setup


## Recap

Before we dive in, let's revisit the key ideas from last session:

- Mobile development is different from web development — your code runs on a device, not a browser
- iOS and Android are different platforms with different toolchains
- Expo abstracts most of that complexity so you can focus on writing your app

Make sure you have the following installed before continuing:

- [Expo Go](https://expo.dev/client) on your phone (iOS or Android)
- Android Studio **or** Xcode

---

## Section 1: Project Setup

### What is Expo?

Expo is the officially recommended framework for React Native. When the React Native docs say "use a framework," they mean Expo.

Without a framework, you would need to manually wire up navigation, native API access, build configuration, and more. Expo gives you all of that out of the box, plus **Expo Go** — an app that lets you preview your project on a real device instantly without any build step.

### Do You Need an Apple Developer Account?

No — not for development. You can run your app on a real iPhone using Expo Go without any Apple account. An Apple Developer account is only required when you want to submit your app to the App Store.

### Useful References

- [Setting up the development environment — React Native](https://reactnative.dev/docs/set-up-your-environment)
- [Create a project — Expo Documentation](https://docs.expo.dev/get-started/create-a-project/)

### Creating a New Project

Run the following in your terminal:

```bash
npx create-expo-app@latest MyApp --template blank
```

Use the `--template blank` flag. The default template includes routing and extra boilerplate you don't need yet.

Navigate into your project:

```bash
cd MyApp
```

Start the development server:

```bash
npx expo start
```

You'll see a QR code in the terminal. Open **Expo Go** on your phone and scan it. Your app will load on the device.

**Emulator shortcuts** (while the dev server is running):

| Key | Action |
|-----|--------|
| `a` | Open Android emulator |
| `i` | Open iOS simulator (Mac only) |
| `w` | Open in browser |
| `r` | Reload the app |
| `m` | Toggle the in-app menu |

### Project File Structure

```
MyApp/
├── app/                    ← all your screens live here (file-based routing)
│   ├── (tabs)/             ← tab navigation group
│   │   ├── _layout.tsx     ← defines the tab bar
│   │   ├── index.tsx       ← first tab screen
│   │   └── explore.tsx     ← second tab screen
│   ├── _layout.tsx         ← root layout
│   ├── +html.tsx           ← web-only HTML shell (ignore for now)
│   └── +not-found.tsx      ← 404 screen
├── assets/                 ← images and fonts
│   ├── fonts/
│   └── images/
├── components/             ← reusable components
├── constants/              ← shared values (colours, sizes, etc.)
├── app.json                ← project config (name, icon, splash screen)
├── tsconfig.json
└── package.json
```

This project uses **Expo Router** — a file-based routing system where each file inside `app/` automatically becomes a screen. For this lesson you'll mostly be working inside the `app/` folder and `components/`.

`app.json` controls metadata about your app — its name, icon, and splash screen. When you're troubleshooting missing asset errors, this is the first place to check.

---

### Troubleshooting: Connection Issues on Windows

On Windows, Expo sometimes fails to auto-detect your machine's local IP. This happens because multiple network interfaces (WiFi, VPN, Docker) confuse the detection. You may see:

- `Could not connect to server`
- `exp://127.0.0.1:8081` in the QR code (127.0.0.1 is localhost — your phone can't reach that)

**Fix — set your IP manually:**

Step 1 — Find your local IP:

```powershell
ipconfig
```

Look for **IPv4 Address** under **Wireless LAN adapter Wi-Fi**. It will look something like `192.168.1.45`.

Step 2 — Set it as an environment variable (PowerShell):

```powershell
$env:REACT_NATIVE_PACKAGER_HOSTNAME='192.168.1.45'
```

Step 3 — Start with the LAN flag:

```bash
npx expo start --lan
```

> **Note:** Your phone and laptop must be on the same WiFi network for this to work. If you're on a school or office network that blocks device-to-device traffic, use an Android emulator instead.

> **Note for Docker users:** Docker adds virtual network interfaces that can interfere with Expo's IP detection. Try pausing Docker Desktop before running `npx expo start`.

---

## Section 2: React vs React Native

React Native uses the same component model as React — props, state, hooks, and the same JSX syntax. The difference is that there are no HTML elements. The browser doesn't exist on a phone, so React Native provides its own set of built-in components that map to native UI elements on iOS and Android.

If you want to try React Native components without a local setup, use one of these playgrounds:

- [snack.expo.dev](https://snack.expo.dev/) — React Native in the browser, includes a device preview
- [playcode.io/react](https://playcode.io/react) — React (web) playground for quick comparisons

### Side-by-Side Comparison

**A simple component in React (web):**

```jsx
export default function Greeting() {
  return (
    <div>
      <h1>Hello, world!</h1>
      <p>Welcome to my app.</p>
    </div>
  );
}
```

**The same component in React Native:**

```jsx
import { View, Text } from 'react-native';

export default function Greeting() {
  return (
    <View>
      <Text style={{ fontSize: 24, fontWeight: 'bold' }}>Hello, world!</Text>
      <Text>Welcome to my app.</Text>
    </View>
  );
}
```

### Key Differences

| Concept | React (Web) | React Native |
|---|---|---|
| Container | `<div>` | `<View>` |
| Text | `<p>`, `<h1>`, `<span>` | `<Text>` |
| Button | `<button>` | `<TouchableOpacity>` |
| Image | `<img>` | `<Image>` |
| Styling | CSS classes / `className` | `StyleSheet` object / `style` prop |
| Layout | CSS Flexbox | Flexbox (same concept, different defaults) |

Every piece of visible text in React Native must be wrapped in a `<Text>` component. You cannot put text directly inside a `<View>` — this is one of the most common mistakes when coming from web.

### Styling in React Native

There is no CSS in React Native. Instead, you create styles using `StyleSheet.create()` and pass them via the `style` prop:

```jsx
import { View, Text, StyleSheet } from 'react-native';

export default function Card() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>My Card</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#fff',
    borderRadius: 8,
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
  },
});
```

Style properties use camelCase (`backgroundColor`, not `background-color`). Values that would be in `px` in CSS are just plain numbers in React Native.

---

## Section 3: Building a One-Page App

Now let's put it together. We'll build a simple one-page app using three core components: `View`, `Text`, and `TouchableOpacity`.

### The Core Components

**`View`** — the basic building block for layout. Think of it as a `div`. It doesn't display anything on its own — it just groups and positions other components.

**`Text`** — renders text. Every string that appears on screen must be inside a `<Text>` component.

**`TouchableOpacity`** — a pressable container. When tapped, it fades slightly to give the user visual feedback. This is the standard way to make something tappable.

**`StyleSheet`** — not a component, but a utility for defining styles. It works like an object-based version of CSS.

Open `app/(tabs)/index.tsx` and replace its contents with the following:

```tsx
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useState } from 'react';

export default function HomeScreen() {
  const [count, setCount] = useState<number>(0);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>My First App</Text>
      <Text style={styles.subtitle}>You have pressed the button {count} times.</Text>
      <TouchableOpacity style={styles.button} onPress={() => setCount(count + 1)}>
        <Text style={styles.buttonText}>Press me</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginBottom: 32,
  },
  button: {
    backgroundColor: '#6200ee',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

### Testing Your App

Once your dev server is running, you can test on:

**Physical device** — scan the QR code with Expo Go. Any time you save a file, the app reloads automatically.

**iOS Simulator** (Mac only) — press `i` in the terminal. Requires Xcode to be installed.

**Android Emulator** — open Android Studio, start a virtual device from Device Manager, then press `a` in the terminal. Expo detects the running emulator automatically.

Try making a change to your `app/(tabs)/index.tsx` and saving — you'll see the app update on device without needing to restart anything. This is called **Fast Refresh** and is one of the big quality-of-life wins with Expo.

---

## Assignment — Travel Card App

Build a single-page React Native app with three components. Each component should be in its own file inside a `components/` folder and imported into `App.jsx`.

### Component 1 — DestinationCard

Accept `city` and `country` as props and display them on screen. Add a button that toggles between **"Save Destination"** and **"Saved ✓"** when tapped.

### Component 2 — PackingList

Display the following items on screen:

- 👕 T-Shirts
- 👟 Shoes
- 🪥 Toothbrush
- 📷 Camera
- 🗺️ Map

Each item should be tappable. When tapped, mark it as packed by showing a ✓ next to it. Tapping again should unmark it.

### Component 3 — WeatherMood

Display three weather options: ☀️ Sunny, 🌧️ Rainy, ❄️ Snowy. When the user taps one, show a message below that says which weather they selected.

### Requirements

- Each component is in its own file in `components/`
- All three components are visible on the same screen
- Each component uses `useState`
- The app runs without errors on a simulator or device

### Stretch Goal

Add basic styling to each component — a white card background, some padding, and readable font sizes.
