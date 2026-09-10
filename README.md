# AANYA AI — Personal Intelligence

A mobile-first, dimensional, voice-first Aanya interface built with React, Vite, TypeScript, and Tailwind CSS. The cyan-glass orb is lightweight artwork layered with CSS holographic rings, particles, atmospheric light, and state-specific motion. No WebGL engine or heavy human model is loaded.

## Project tree

- `index.html` — app title, responsive metadata, and entry document
- `src/App.tsx` — assistant workspace, conversation, voice dock, settings, actions, history, help
- `src/index.css` — responsive glass visual system and accessibility rules
- `src/main.tsx` — React entry point
- `src/components/AanyaOrb.tsx` — dimensional five-state orb
- `src/lib/assistant.ts` — streaming transport, sentence segmentation, audio queue, storage, preview replies
- `src/lib/useAssistant.ts` — conversation lifecycle, recognition, wake word, cancellation
- `src/lib/actions.ts` — allowlisted, user-triggered browser actions
- `public/images/aanya-core.png` — optimized dimensional orb artwork
- `public/favicon.svg` — Aanya identity mark
- `.env.local.example` — server-only variable names, no credentials
- `.gitignore` — excludes secrets and generated files

## Scope and existing backend

This workspace originally contained a Vite starter, not the linked Next.js repository. This implementation does not overwrite, rebuild, or deploy that repository's backend. Its existing route contracts were inspected and retained in the client adapter:

- `POST /api/chat`: JSON body with `messages`, containing `role` and `content`; response `text/event-stream` with `data: {"delta":"text"}`, `data: {"done":true}`, or `data: {"error":"message"}`.
- `POST /api/tts`: JSON body with `text` and `voice: "Leda"`; response `audio/wav`.

The provider connection is configured in the existing server, not in the browser:

- NVIDIA endpoint: `https://integrate.api.nvidia.com/v1/chat/completions`
- NVIDIA model: `deepseek-ai/deepseek-v4-pro-0813`
- Gemini model: `gemini-3.1-flash-tts-preview`
- Gemini voice: `Leda`

The server's master prompt and conversation rules remain server-side and unchanged. No new prompt file or provider route is necessary for this frontend.

## Install and run

Use Node.js 22.12 or later, or a compatible Node.js 20 release supported by Vite 7.

1. Run `npm install`.
2. Run `npm run dev` for local development.
3. Run `npm run build` for a production build.
4. Serve the generated `dist` directory over HTTPS.

Dependencies include React, Lucide icons, Tailwind CSS, and Vite. Fonts load from Google Fonts with system-font fallbacks.

## Preview versus live intelligence

The app starts in clearly labeled **Interface preview** mode. Preview responses are deterministic samples, progressively displayed; they are not NVIDIA-generated answers. Preview speech uses browser speech synthesis, not Gemini or Leda. Browser actions work independently of AI connection.

To use live intelligence:

1. Keep the existing Next.js `/api/chat` and `/api/tts` server routes deployed.
2. Open **Settings → AI connection**.
3. Enter the public server URL, without an API path, credentials, query parameters, or fragments. Leave blank if the frontend and API share an origin.
4. Select **Save & use live AI**.
5. Send a message. The status becomes connected only when a live text delta is received.

For separate origins, configure your backend or gateway to allow only your frontend's origin through CORS, including POST and Content-Type in the preflight response. Alternatively, serve the frontend and existing API through a same-origin reverse proxy. Do not forward requests to arbitrary user-supplied hosts on the server. No live provider request has been verified from this static workspace because server credentials and a deployed API are not included.

Turning off preview alone does not create a backend. If the API is missing or returns HTML, the UI reports the connection problem rather than silently presenting sample text as live intelligence.

## API key setup

Copy `.env.local.example` into `.env.local` in your **existing Next.js server project**, then set fresh credentials locally or in your host's encrypted server environment:

- `NVIDIA_API_KEY`: your NVIDIA provider credential
- `NVIDIA_MODEL`: the existing configured model
- `AI_API_KEY`: your Gemini API credential, matching the existing TTS route

Restart or redeploy the backend after updating the environment. Do not add secrets to this Vite client. Never prefix secrets with `VITE_` or `NEXT_PUBLIC_`. Never put keys in Settings, localStorage, URLs, source files, screenshots, issue reports, or chat. Keep provider error details server-side and sanitize responses before returning them to users. Apply authentication, rate limits, request limits, and provider quotas on the server before making the API public.

## Voice and streaming behavior

- Manual microphone input supports `en-IN` and `hi-IN`, interim text, final results, permission errors, start, stop, abort, and cleanup.
- Voice recognition uses the browser implementation and may rely on the browser vendor's online recognition service.
- Wake recognition accepts common forms of “Hey Aanya,” “Hey Anya,” and “Hey Ania.” A spoken name alone activates command listening. The name is removed when a command follows it.
- Wake listening is opt-in and works on the active page only. It stops when the page becomes hidden. Re-enable it after returning. Closed or suspended pages cannot provide reliable browser wake-word support.
- Live SSE deltas update the conversation immediately. Completed sentences are added to the audio queue without waiting for the full answer.
- The first queued sentence requests Gemini WAV audio immediately. Text keeps streaming during synthesis and playback. Subsequent sentence requests and playback run sequentially to keep mobile resource use bounded.
- Sentence detection supports common Latin and Hindi punctuation and a small abbreviation guard. It is a lightweight heuristic, not a complete linguistic parser.
- AudioContext is unlocked on a user gesture. Mobile autoplay restrictions can still require tapping the sound control.
- Tap the microphone while Aanya is speaking to stop audio, clear the queue, cancel pending TTS, abort the chat request, and accept a new command.
- The square stop button cancels generation and playback without opening the mic.
- On page hiding or unmount, active recognition and requests are stopped; audio resources are stopped or disposed as appropriate.

## Browser actions

All actions are user-triggered and explicitly allowlisted. No arbitrary AI-generated JavaScript is evaluated. Available actions include Google search, YouTube, Maps, validated `tel:` and `mailto:` links, Web Share, clipboard, vibration, fullscreen, notifications, and camera API capability detection.

Opening a dialer does not place a call. Opening a mail app does not send email. Camera detection never opens the camera. External services have their own privacy policies. Device capabilities, permissions, pop-up restrictions, installed apps, and browser support determine availability.

Mobile notifications may require a service worker; this app reports that limitation instead of claiming background notifications work. It does not install a background wake listener, silently control Android apps, access contacts, or provide native Android permissions.

## Local data and privacy

Preferences and the most recent 80 messages are stored in localStorage. They are not encrypted and are accessible to scripts on the same origin. Avoid shared devices for sensitive conversations. Use **Conversation → options → Clear conversation** to remove the active local history. Use Export to download a text copy.

Live mode sends the selected conversation history to your backend and its AI providers. Browser recognition and some device speech voices can use online services. Do not describe the app as entirely offline or end-to-end encrypted. The orb coordinate labels are decorative, not tracked location.

## GitHub security cleanup — urgent

The public repository listing includes a tracked `.env.local`. Treat every credential that may have been committed as compromised, even after deleting the file. No existing secret was opened, copied, or reproduced in this implementation.

1. **Immediately revoke and rotate** affected NVIDIA, Google/Gemini, and any other credentials in their provider consoles. Review usage logs, quotas, and billing. Removing a GitHub file does not revoke a secret.
2. Set fresh credentials in your deployment platform's private environment settings and your local server-only `.env.local`. Redeploy the backend. Restrict keys appropriately and monitor for abuse.
3. Ensure `.gitignore` contains `.env`, `.env.*`, and exceptions only for sanitized example files. Ignoring a file does not untrack an already committed file.
4. From your repository, run `git rm --cached -- .env.local`. This removes it from Git tracking while retaining your local copy. Commit the removal and `.gitignore`, then push the commit.
5. Remove the secret from Git history as well. Back up the repository, coordinate with collaborators, and use a fresh clone with `git-filter-repo`. The path-removal operation is `git filter-repo --path .env.local --invert-paths`. Include every historic path or renamed secret file if needed. Follow GitHub's sensitive-data-removal guidance for force-pushing rewritten refs and coordinating fresh clones.
6. History rewriting changes commit IDs and disrupts existing branches and pull requests. Do not blindly force-push over collaborators' changes. Forks, clones, PR refs, caches, build logs, and artifacts may retain old content. Contact GitHub Support for inaccessible cached references when applicable.
7. Enable GitHub secret scanning and push protection, audit workflow artifacts and deployment logs, and verify that tracked files contain no new credentials. `git ls-files .env.local` should produce no output. `git check-ignore .env.local` should confirm it is ignored.

Rotation is mandatory even if history cleanup is complete. This workspace cannot rotate your provider keys or remove files from the remote repository for you.

## Android testing checklist

Test on a physical Android device using current Chrome and an HTTPS deployment. Localhost on a laptop is not the same as a secure origin on a phone.

- [ ] Portrait layouts at 360, 390, and 412 CSS pixels: no horizontal scrolling; orb, cards, dock, and conversation sheet remain accessible.
- [ ] Landscape and desktop: transcript does not displace the orb; dialogs can scroll.
- [ ] Allow microphone permission and test English (India), then Hindi (India).
- [ ] Deny microphone permission and verify the actionable error and keyboard fallback.
- [ ] Check unsupported recognition browsers show a truthful compatibility message.
- [ ] Speak a wake word alone, then a command; also test a combined wake word and command.
- [ ] Switch tabs or lock the phone: recording and active requests stop; wake mode must be re-enabled on return.
- [ ] Connect the live server and confirm progressive text before the response completes.
- [ ] Inspect requests and verify one sentence begins TTS before the full chat reply finishes.
- [ ] Verify Gemini/Leda WAV speech, sequential playback, and visible speaking state.
- [ ] Interrupt during synthesis, playback, and text streaming. No old queued audio should restart.
- [ ] Mute/unmute, test browser autoplay restrictions, and test slow or disconnected networks.
- [ ] Try actions individually; confirm external destinations and unsupported-capability messages.
- [ ] Verify share, clipboard, vibration, fullscreen, dialer, and email behavior on the actual phone.
- [ ] Test reduced motion in OS settings and the in-app animation toggle.
- [ ] Test keyboard focus, Escape, dialog focus trapping, Ctrl/Command+K, and screen-reader labels.
- [ ] Export and clear conversation; reload to verify local persistence behavior.
- [ ] Inspect the built client and network panel: no API credentials are present.

## Feature checklist

Implemented: dimensional orb, idle/listening/thinking/speaking/error appearances, CSS holographic layers, responsive workspace, glass conversation UI, history search/export/clear, progressive SSE transport, abortable chat, sentence-aware speech queue, abortable TTS, interruption, browser speech input, active-page wake phrase handling, English/Hindi selection, safe actions, useful errors, local preferences, keyboard navigation, reduced-motion support, and server-only credential guidance.

Not claimed: a bundled or newly deployed backend, verified live credentials, native Android control, always-on closed-page listening, real-time weather/location data, a heavy 3D human model, or background notifications without an appropriate service worker.
