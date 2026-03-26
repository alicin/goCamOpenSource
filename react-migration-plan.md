# React Migration Plan

## 1. File Structure Layout

We should restructure the frontend logic to separate the **UI Views** (React) from the **Business Logic** (Existing `Avs` namespace, gradually verified).

Proposed `source/frontend/react` structure:

```text
source/frontend/react/
├── main.tsx                # Entry point
├── App.tsx                 # Main Layout & Router
├── context/
│   └── AvsContext.tsx      # Global State for Verification Flow
├── hooks/
│   └── useLegacyAvs.ts     # Bridge hook to interact with window.Avs
├── components/
│   ├── layout/
│   │   ├── Header.tsx      # Maps to base.twig & home header
│   │   └── Footer.tsx
│   ├── ui/                 # Reusable atomic components
│   │   ├── Button.tsx
│   │   ├── Checkbox.tsx
│   │   ├── Loader.tsx
│   │   └── Modal.tsx
│   └── steps/              # The "Pages" or "Steps" of the Verification Flow
│       ├── StartScreen.tsx                 # (startPage)
│       ├── ScanIdIntroScreen.tsx           # (scanIdAgeVerificationIntro)
│       ├── ScanIdVerificationScreen.tsx    # (scanIdAgeVerificationPage)
│       ├── SelfieIntroScreen.tsx           # (selfieAgeDetectionIntro)
│       ├── SelfieDetectionScreen.tsx       # (selfieAgeDetectionPage)
│       ├── ResultSuccessScreen.tsx         # (resultPageSuccess)
│       └── ResultFailScreen.tsx            # (resultPageFail)
└── pages/
    ├── HomePage.tsx        # (home/index.twig)
    ├── TokenPage.tsx       # (token/index.twig) - The main Single Page App
    └── IframeCheckPage.tsx # (token/embedCheck.twig)
```

## 2. Dependencies to Add

*   `react`, `react-dom`
*   `vite` (for building the React bundle)
*   `sass` (to reuse existing `.scss` files)

## 3. Migration Stages

### Stage 1: Hybrid Integration (The Bridge)
*   **Goal**: Render React components into the existing Twig pages without changing the backend logic.
*   **Action**:
    1.  Setup Vite to build `main.tsx` into `app/frontend/static/js/react-bundle.js`.
    2.  Include this bundle in `base.twig`.
    3.  Create a "Root" component that reads the `script#app-data` (JSON) to initialize state.
    4.  Keep `Avs.js` active. Use React purely for rendering the *view* (replacing Twig HTML).
    5.  Use `useLegacyAvs` hook to listen to `Avs` events (e.g., `Avs.Event.Listener`) and update React state.
    6.  **Milestone**: The page renders via React, but the clicking of buttons still triggers the underlying global `Avs` logic via event delegation or direct calls.

### Stage 2: Component Replacement
*   **Goal**: Replace `AvsFactory` (UI Logic) with React Components.
*   **Action**:
    1.  Translate `avsFactory.js/**/10-ui.ts` (jQuery selectors) into React Components with props.
    2.  Translate `20-binding.ts` (Event listeners) into React `onClick`/`onChange` handlers.
    3.  Move the "Page Switching" logic (hiding/showing divs) into a React State (e.g., `currentStep` enum).

### Stage 3: Logic Extraction (Full Rewrite)
*   **Goal**: Remove dependency on the global `Avs` namespace for UI control.
*   **Action**:
    1.  Move logic from `avs.js` (business logic) into pure TypeScript hooks/utils imported by React components.
    2.  Refactor the heavy lifting (WebRTC, Tesseract, FaceAPI) into services:
        *   `services/CameraService.ts`
        *   `services/OcrService.ts`
    3.  Stop loading the legacy `avs.js` bundle.

## 4. Immediate Next Steps (No code changes yet)

1.  Review `source/frontend/js/app/avsFactory.js` to map every "Page" folder to a React Component.
2.  Ensure `tsconfig.json` can handle JSX.
3.  Plan the Vite configuration to output assets to `app/frontend/static`.
