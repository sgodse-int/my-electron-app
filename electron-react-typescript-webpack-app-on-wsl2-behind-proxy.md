# ⚡ Clean Electron + React + TypeScript + Webpack App in WSL2 Behind a Proxy (with WSLg GUI)

This guide walks through **setting up a modern Electron app** with React and TypeScript on **WSL2**, **behind a corporate proxy**, and **with GUI support via WSLg**.

---

**How Electron, React, Webpack, and TypeScript all work together in the app**
=============================================================================

## 🔄 Overview Diagram

```
User
 ↓
Electron Main Process (main.ts)
 └── Creates BrowserWindow
       ↓
    Loads index.html
       ↓
    <script src="renderer.js"> from Webpack
       ↓
Webpack Renderer Bundle (renderer.tsx → React + App.tsx)
 └── Bootstraps React UI in #root div
       ↓
    React Components Rendered
```

---

## 🧱 How All Pieces Fit Together

### ✅ 1. **TypeScript**

* You write your code in `.ts` and `.tsx` files.
* TypeScript provides types and checks for errors **before** you compile.

Example:

```tsx
const App: React.FC = () => <h1>Hello</h1>;
```

---

### ✅ 2. **React**

* React handles rendering dynamic UI.
* The entry point is `renderer.tsx`, which mounts the `<App />` component.

---

### ✅ 3. **Webpack**

You have **two Webpack configs**:

#### ➤ `webpack.main.config.js`

* **Entry**: `src/main.ts`
* **Output**: `dist/main.js`
* **Role**: Bundles your Electron backend code (responsible for window creation).

#### ➤ `webpack.renderer.config.js`

* **Entry**: `src/renderer.tsx`
* **Output**: `dist/renderer.js`
* **Role**: Bundles all frontend React code (your UI) into one JavaScript file.

Webpack also uses **`ts-loader`** to convert TypeScript to JavaScript.

---

### ✅ 4. **Electron**

Electron has **two processes**:

#### 🔹 Main Process (backend):

* Runs `main.js` (compiled from `main.ts`)
* Creates windows using Electron’s API
* Manages system-level things like tray icons, file access, IPC, etc.

#### 🔹 Renderer Process (frontend):

* Electron loads `index.html`
* The `<script>` inside loads `renderer.js`, the React/Webpack bundle
* Renders UI in a Chromium browser window

---

## 🔁 Data Flow (Start to Finish)

1. **Electron boots up** and runs `main.js`
2. **`BrowserWindow` is created** and it loads `index.html`
3. The `<script src="renderer.js">` loads your **React App**
4. React finds `<div id="root">` and injects the UI (`App.tsx`)
5. The user interacts with your UI rendered by React

---

## 📦 Build Output

After `npx webpack`, you get:

```
dist/
├── main.js        ← Electron backend
└── renderer.js    ← React frontend bundle
```

These are the files Electron needs to run your app.

---

Would you like a **real graphical diagram** of this setup? I can generate an image showing the interaction between these components.

==================================================================================================
---

## ✅ Prerequisites

* **Windows 11** with WSL2 installed
* **WSLg** (for GUI apps in WSL2)
* Ubuntu (20.04 or later) installed via WSL2
* Corporate proxy (e.g., `http://proxy-dmz.intel.com:912`)

---

## 1. 🔥 Cleanup Any Previous Node/npm

```bash
sudo apt remove -y nodejs npm
sudo rm -rf ~/.npm ~/.nvm ~/.node-gyp /usr/local/lib/node_modules
```

---

## 2. 📦 Install Node.js LTS (via NodeSource)

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify:

```bash
node -v
npm -v
npm config list
```

---

## 3. 🌐 Configure Proxy for npm and Electron

```bash
npm config set proxy http://proxy-dmz.intel.com:912
npm config set https-proxy http://proxy-dmz.intel.com:912
npm config set strict-ssl true
```

```bash 
export http_proxy=http://proxy-dmz.intel.com:912
export https_proxy=http://proxy-dmz.intel.com:912
```

---

## 4. 🏗️ Create Project

```bash
mkdir my-electron-app && cd my-electron-app
npm init -y
```

---

## 5. 🚀 Install Electron (with proxy settings)

```bash
ELECTRON_GET_USE_PROXY=true \
ELECTRON_GET_PROXY=http://proxy-dmz.intel.com:912 \
npm install --save-dev electron
```
If fails, try `npm install --save-dev electron --no-audit`  


---

## 6. 🧰 Install Dependencies

```bash
npm install --save react react-dom
npm install --save-dev typescript ts-loader webpack webpack-cli html-webpack-plugin @types/react @types/react-dom
```

---


## 🔧 Step 7: Create Project Structure

This step sets up your project's directory layout and basic code files. Here's what we created:

```
my-electron-app/
├── src/
│   ├── App.tsx
│   ├── main.ts
│   └── renderer.tsx
├── public/
│   └── index.html
├── tsconfig.json
├── webpack.main.config.js
├── webpack.renderer.config.js
├── package.json (auto-generated earlier)
```

Let’s go through each file in detail:

---

### 🧠 `src/main.ts`

```ts
const { app, BrowserWindow } = require('electron');

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      contextIsolation: true,
      nodeIntegration: false,
    },
  });

  win.loadFile('public/index.html');
};

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

**Explanation:**

* `require('electron')`: Loads Electron’s core features.
* `BrowserWindow`: This creates a native window.
* `createWindow()`: A function that:

  * Creates an 800x600 window.
  * Loads your HTML file that will show the React app.
* `app.whenReady()`: Runs when Electron is fully initialized.
* `app.on('window-all-closed')`: Exits the app when all windows are closed, except on macOS (where apps usually stay open).

---

### ⚛️ `src/renderer.tsx`

```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root') as HTMLElement);
root.render(<App />);
```

**Explanation:**

* `import React`: Needed for JSX (HTML-like syntax inside JavaScript).
* `ReactDOM.createRoot(...)`: New React 18 method to create a root.
* `document.getElementById('root')`: Looks for a `<div id="root"></div>` in your HTML file.
* `<App />`: Renders the React component you define in `App.tsx`.

---

### 📦 `src/App.tsx`

```tsx
import React from 'react';

const App: React.FC = () => {
  return <h1>Hello from React in Electron!</h1>;
};

export default App;
```

**Explanation:**

* `React.FC`: Stands for "React Functional Component".
* `return <h1>...</h1>`: What gets rendered on the page.
* `export default App`: Allows other files (like `renderer.tsx`) to import this component.

---

### 🌐 `public/index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Electron React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="../dist/renderer.js"></script>
  </body>
</html>
```

**Explanation:**

* `<div id="root"></div>`: This is where React injects your app UI.
* `<script src="../dist/renderer.js">`: This loads the compiled JavaScript bundle from Webpack.

---

### 🧰 `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "jsx": "react-jsx",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```

**Explanation:**

* Tells TypeScript how to compile the code.
* `target: "ESNext"`: Use modern JavaScript.
* `jsx: "react-jsx"`: Supports JSX for React.
* `strict: true`: Enables strict type-checking.

---

### ⚙️ `webpack.renderer.config.js`

```js
const path = require('path');

module.exports = {
  entry: './src/renderer.tsx',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'renderer.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

**Explanation:**

* `entry`: Your app’s entry point (renderer process).
* `ts-loader`: Converts TypeScript to JavaScript.
* `output`: Saves the final JavaScript bundle to `dist/renderer.js`.

---

### ⚙️ `webpack.main.config.js`

```js
const path = require('path');

module.exports = {
  entry: './src/main.ts',
  target: 'electron-main',
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.ts', '.js'],
  },
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

**Explanation:**

* Similar to the renderer config, but for Electron’s **main process**.
* `target: 'electron-main'`: Tells Webpack this is the Electron backend process.

---

Would you like me to also walk you through **how Webpack, TypeScript, Electron, and React work together** as a diagram or visual concept?


---

## 11. 🎮 Run the App

```bash
npm run build
npm start
```

If you see Electron GUI with React: ✅ success!

---

## 🧑‍💻 GUI on WSL: Using WSLg

### Verify WSLg is Working

```bash
echo $DISPLAY
```

Should return `:0` or similar.

### Reset in Case of Errors:

```bash
unset DISPLAY
wsl --shutdown  # in PowerShell
# Then relaunch Ubuntu and try again
```

Test with:

```bash
sudo apt install x11-apps
xeyes
```

---

## ✅ You're Done!

You now have a **fully working Electron + React + TypeScript** development environment in **WSL2** behind a **corporate proxy**, with **WSLg GUI** support.

