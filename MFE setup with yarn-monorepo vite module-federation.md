# Micro Frontend (MFE) Setup with Vite & Module Federation

This guide will help you set up a **Micro Frontend (MFE)** architecture using **Vite and Module Federation**. Follow the steps sequentially to create a **Host** application and multiple **Remote** applications that share components.

---

## 🚀 Project Structure
```
my-new-mfe/
├── host/         # Main container (Host App)
├── remotes/      # Directory for Micro Frontend Remotes
│   ├── header/   # Example Remote: Header Component
│   ├── button/   # Example Remote: Button Component
```

## 📌 Step 1: Initialize the Project
Create a new project directory and navigate into it:
```sh
mkdir my-new-mfe && cd my-new-mfe
```

Initialize a Yarn workspace:
```sh
yarn init -y
```

---

## 📌 Step 2: Create Host & Remote Directories
Create directories for the **Host** and **Remote Apps**:
```sh
mkdir -p host remotes/header remotes/button
```

Modify `package.json` to define the workspace structure:
```json
{
  "private": true,
  "workspaces": ["host", "remotes/*"]
}
```

---

## 📌 Step 3: Create Vite Apps for Host & Remotes
Inside each **Host** and **Remote** directory, run:
```sh
yarn create vite . --template react
```

Install **Module Federation Plugin**:
```sh
yarn add @originjs/vite-plugin-federation
```

---

## 📌 Step 4: Create a Shared Component in Remote Apps
Each remote should have a **component** inside `src/components/`. Example for `button` remote:

📁 `remotes/button/src/components/Button.jsx`
```jsx
const Button = () => {
  return <button>Click Me</button>;
};
export default Button;
```

---

## 📌 Step 5: Configure Vite for Remote Apps
Modify the `vite.config.js` file inside each **Remote App** (e.g., `button` remote):

📄 `remotes/button/vite.config.js`
```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: "remoteButton",
      filename: "remoteEntry.js",
      exposes: {
        "./Button": "./src/components/Button.jsx"
      },
      shared: {
        react: { singleton: true, requiredVersion: "auto" },
        "react-dom": { singleton: true, requiredVersion: "auto" }
      },
    }),
  ],
  server: { port: 5001 },
  build: {
    target: "esnext",
    outDir: "dist",
    rollupOptions: {
      output: {
        publicPath: "auto",
      },
    },
  },
});
```

---

## 📌 Step 6: Configure Vite for Host App
Modify the `vite.config.js` file in the **Host App**:

📄 `host/vite.config.js`
```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";

export default defineConfig({
  plugins: [
    react(),
    federation({
      remotes: {
        remoteButton: process.env.VITE_REMOTE_BUTTON_URL,
      },
      shared: {
        react: { singleton: true, requiredVersion: "auto" },
        "react-dom": { singleton: true, requiredVersion: "auto" }
      },
    }),
  ],
  server: { port: 5000 },
  build: { target: "esnext" },
});
```

Create a `.env` file in the **Host App** directory:
```sh
VITE_REMOTE_BUTTON_URL=http://localhost:5001/dist/assets/remoteEntry.js
```

---

## 📌 Step 7: Import Remote Components in Host
Use **React.lazy** and **Suspense** to load remote components dynamically:

📄 `host/src/App.jsx`
```jsx
import { lazy, Suspense } from "react";

const RemoteButton = lazy(() => import("remoteButton/Button"));

const App = () => {
  return (
    <>
      <h1>Host Application</h1>
      <Suspense fallback={<p>Loading...</p>}>
        <RemoteButton />
      </Suspense>
    </>
  );
};

export default App;
```

---

## 📌 Step 8: Run the Project
Follow these steps sequentially:

```sh
# Navigate to root directory
cd my-new-mfe

# Build the Remote App to generate remoteEntry.js
yarn workspace button build

# Start the Remote App
yarn workspace button dev

# Start the Host App
yarn workspace host dev
```

Your **Host App** should now be able to consume the **Remote Button Component** dynamically. 🎉

---

## 🎯 Summary
- **Micro Frontend (MFE)** setup using Vite + Module Federation.
- **Host App** dynamically imports components from **Remote Apps**.
- **Workspaces** used to manage multiple MFE applications.
- **React.lazy & Suspense** used for seamless async loading.
- **Environment variables** are used for flexible remote configurations.

🚀 Now you have a fully functional Micro Frontend architecture! Happy coding! 🎉

