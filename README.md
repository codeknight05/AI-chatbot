# AI Powered Chat Application

React + Vite chat app with Firebase authentication/storage and a user-provided Gemini API key.

## Local development

```bash
npm install
npm run dev
```

## Build

```bash
npm run build
```

The production files are generated in `dist`.

## Deploy

This is a static Vite app, so it can deploy to Vercel, Netlify, Firebase Hosting, or any static host.

Recommended Vercel settings:

- Framework preset: Vite
- Build command: `npm run build`
- Output directory: `dist`
- Install command: `npm install`

Recommended Netlify settings:

- Build command: `npm run build`
- Publish directory: `dist`

After deployment, add your production domain to Firebase Authentication authorized domains in the Firebase console. Users enter their own Gemini API key in the app, so no Gemini server environment variable is required for the current implementation.
