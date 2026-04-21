# Implementation Plan: Booking Integration

## Overview

Integrate Cal.com's inline scheduling embed into the existing React 19 + Vite app by installing dependencies, creating the `BookingWidget` and `BookingWidgetErrorBoundary` components, wiring them into `App.jsx`, and covering the integration with property-based and unit tests.

## Tasks

- [x] 1. Install dependencies and configure Vitest
  - [x] 1.1 Install runtime and dev dependencies
    - Run `npm install @calcom/embed-react`
    - Run `npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom fast-check`
    - _Requirements: 2.1, 5.1_

  - [x] 1.2 Add Vitest configuration and test script
    - Add `"test": "vitest --run"` to `package.json` scripts
    - Update `vite.config.js` to include Vitest `test` config with `environment: "jsdom"` and `setupFiles` pointing to a test setup file
    - Create `src/setupTests.js` that imports `@testing-library/jest-dom`
    - _Requirements: 5.1_

- [ ] 2. Implement BookingWidgetErrorBoundary component
  - [ ] 2.1 Create `src/components/BookingWidgetErrorBoundary.jsx`
    - Implement a React class component error boundary
    - Accept `calLink` and `children` props
    - On error: render a user-friendly message with an anchor linking to `https://cal.com/{calLink}`
    - On success: render children transparently
    - _Requirements: 3.4, 5.4_

  - [ ]* 2.2 Write property test: Error fallback link correctness (Property 2)
    - **Property 2: Error fallback link correctness**
    - Generate arbitrary non-empty strings with `fast-check`, render `BookingWidgetErrorBoundary` wrapping a component that throws, assert the fallback anchor `href` equals `https://cal.com/{calLink}`
    - Minimum 100 iterations
    - **Validates: Requirements 3.4, 5.4**

- [ ] 3. Implement BookingWidget component
  - [ ] 3.1 Create `src/components/BookingWidget.jsx`
    - Import `Cal` from `@calcom/embed-react`
    - Accept `calLink` (required) and `config` (optional, default `{}`) props
    - Render a CSS-only loading spinner that displays while the embed initializes
    - Listen for the Cal.com `__iframeReady` message to hide the spinner
    - Wrap the `Cal` component in `BookingWidgetErrorBoundary`, forwarding `calLink`
    - Pass `calLink` and spread `config` to the `Cal` component
    - _Requirements: 3.1, 3.2, 3.3, 6.1, 6.2_

  - [ ]* 3.2 Write property test: calLink prop forwarding (Property 1)
    - **Property 1: calLink prop forwarding**
    - Mock `@calcom/embed-react` to expose `calLink` as a data attribute
    - Generate arbitrary non-empty strings with `fast-check`, render `BookingWidget` with each, assert the mock Cal element's `data-callink` matches the input
    - Minimum 100 iterations
    - **Validates: Requirements 3.2, 5.3**

  - [ ]* 3.3 Write unit tests for BookingWidget
    - Test: renders Cal embed element on mount (`calLink="jane/30min"`)
    - Test: shows loading indicator before embed is ready
    - Test: hides loading indicator after embed ready event
    - Test: renders fallback message with direct link when embed throws
    - All tests use mocked `@calcom/embed-react` — no network calls
    - _Requirements: 3.1, 3.4, 5.2, 5.3, 5.4, 5.5, 5.6, 6.1, 6.2_

- [ ] 4. Checkpoint
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Wire BookingWidget into the App
  - [ ] 5.1 Update `src/App.jsx` to use BookingWidget
    - Replace the placeholder `div` with `<BookingWidget calLink="your-username/30min" />`
    - Import `BookingWidget` from `./components/BookingWidget.jsx`
    - _Requirements: 3.1, 3.2, 3.3_

- [ ] 6. Final checkpoint
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests validate universal correctness properties from the design document
- Unit tests validate specific examples and edge cases
- All tests mock `@calcom/embed-react` so no network or Cal.com credentials are needed
