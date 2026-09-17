# Virtual Try-On

## Project overview

A full-stack application that generates a clothing preview from a photo of the user and an image of a clothing item. A React/TypeScript client handles uploads and displays results, while a Node.js/Express backend sends the images to Gemini.

The Gemini prompt attempts to preserve the person, pose, facial features, and background while changing the clothing. These details are not guaranteed to remain unchanged in the generated image.

## Demo

<table>
  <tr>
    <th>Input images</th>
    <th>Generated try-on</th>
  </tr>
  <tr>
    <td valign="top"><img src="docs/images/virtual-try-on-input.png" alt="Person and clothing image previews on the upload screen" width="600"></td>
    <td valign="top"><img src="docs/images/virtual-try-result.png" alt="Generated try-on result" width="400"></td>
  </tr>
</table>

**Try-on gallery**

![Try-on gallery showing completed results and input thumbnails](docs/images/virtual-try-on-gallery.png)

## How it works

1. The user selects a person photo and a clothing image, with previews before submission.
2. The client sends two multipart image uploads to `POST /api/upload/user` and `POST /api/upload/clothing`.
3. The client submits the returned image paths to `POST /api/try-on`.
4. The backend reads both files and sends a base64 multimodal request to Gemini through the Google Gen AI SDK, using `gemini-3-pro-image-preview`.
5. The backend saves the generated image and session metadata. The client displays the result and includes completed try-ons in the gallery.

## Key features

- Separate person and clothing uploads with previews and removal controls.
- Gemini-generated clothing previews from both image inputs.
- Uploading and processing indicators.
- A gallery showing completed results alongside their original inputs.
- Reuse of a person photo or generated result for another try-on.
- Local image and JSON session persistence across server restarts.

## Tech stack

- **Client:** React 19, TypeScript, Vite 7, Tailwind CSS 4, DaisyUI, and Heroicons.
- **Backend:** Node.js, TypeScript, Express 4, Multer, dotenv, and nanoid.
- **AI integration:** Google Gen AI SDK (`@google/genai`) and Gemini.
- **Development:** concurrently, nodemon, and tsx.

## Architecture

```text
React client → Express API → Gemini
                   ↓
          Local images and session JSON
```

- `client/src/components/`: upload interface, processing modal, navigation, and gallery.
- `client/src/services/api.ts`: backend HTTP requests.
- `server/src/routes/` and `server/src/controllers/`: upload and try-on endpoints.
- `server/src/services/gemini.ts`: multimodal request construction and response handling.
- `server/uploads/`: uploaded inputs and generated images, served through `/uploads`.
- `server/data/sessions.json`: session metadata managed by `server/src/data/tryOnSessions.ts`.

## Local setup

Use Node.js 22.12 or later and npm. You need a Gemini API key with access to the configured image model.

From the repository root, install root, client, and server dependencies:

```bash
npm run install:all
```

Copy the environment template if you do not already have `server/.env`:

```bash
cp server/.env.example server/.env
```

Set your Gemini API key in `server/.env`, then start both development servers from the repository root:

```bash
npm run dev
```

Open the URL printed by Vite (normally `http://localhost:5173`). The backend runs at `http://localhost:3000`.

## Environment variables

Set these values in `server/.env`, replacing the API key placeholder locally:

```dotenv
GEMINI_API_KEY=your_gemini_api_key_here
PORT=3000
```

`GEMINI_API_KEY` is required for generation. `PORT` is optional and defaults to `3000`; use this port with the current client. Keep the key on the server and never commit it. The root `.gitignore` excludes environment files, uploads, and session data, with an exception for `server/.env.example`.

## Notes

Results depend on the Gemini model. This is a visual preview, not a physical sizing tool. Images and sessions are stored locally for demonstration purposes.
