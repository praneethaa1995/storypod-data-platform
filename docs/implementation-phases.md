# Storypod Implementation Phases

This document turns the proposal in [architecture.md](architecture.md) into an ordered, step-by-step implementation process. Complete and verify each phase before moving to the next. The first release uses a local story generator, so the app is usable without AI credentials.

## Phase 1: Project Foundation

### Steps

1. Confirm the first release is a responsive web app and choose the frontend stack.
2. Initialize the app and add the development, build, and test commands.
3. Create the app shell and a minimal route or page structure.
4. Add global styles, responsive breakpoints, typography, and color tokens.
5. Define shared TypeScript types for story inputs and generated stories.
6. Add a basic automated check for the production build.

### Done when

- The app starts locally and builds successfully.
- The initial page renders at mobile and desktop widths without horizontal overflow.
- Shared story types are defined and used by the app.
- The frontend and backend run locally without AWS credentials, Docker, or cloud services.

## Phase 2: Story Builder

### Steps

1. Add fields for main character and story theme.
2. Add optional sidekick and lesson fields.
3. Add age range, story length, and tone controls using finite, labeled options.
4. Validate required fields, trim whitespace, and enforce sensible input limits.
5. Show validation feedback beside the relevant fields.
6. Add submit, pending, disabled, and failure states.
7. Prevent duplicate submissions while a story is being generated.

### Done when

- A caregiver can complete the form using a keyboard or touch screen.
- Invalid input is explained clearly and does not submit.
- Pending and error states are visible and accessible.

## Phase 3: Local Story Generation

### Steps

1. Define a provider-independent `generateStory(input)` service interface.
2. Implement a local generator that creates a complete story from the submitted inputs without network calls or secrets.
3. Split the story into ordered pages and normalize the output into the shared page-based response type.
4. Attach a bundled still illustration to every page, with a fallback if an asset is unavailable.
5. Handle unexpected or empty output with a recoverable error message.
6. Connect the story builder to the service.

### Done when

- The complete create-to-story flow works without an API key or network connection.
- Generated output includes a title and ordered pages, each with story text and a still illustration.
- The generator can be replaced without coupling provider-specific code to UI components.

## Phase 4: Reading Experience

### Steps

1. Add a picture-book reader that displays one story page at a time, pairing its text with an illustration.
2. Add previous/next controls, a page counter, and clear first/last-page states.
3. Support optional video page visuals with a poster and accessible playback controls; do not autoplay sound.
4. Use readable font sizing, line length, and media sizing across screen sizes.
5. Add actions to create another story and return to edit the inputs.
6. Announce page changes and generation results for assistive technology.
7. Respect reduced-motion settings for page transitions and video behavior.

### Done when

- Stories display as navigable pages, each with text and a visual, on a phone and desktop.
- The caregiver can regenerate or revise the inputs without getting stuck.
- Page navigation and media controls work with keyboard and assistive technology.

## Phase 5: Saved Stories

### Steps

1. Define a `StoryRepository` interface for saving, listing, retrieving, and deleting stories.
2. Implement browser-local storage behind that interface.
3. Add save and delete actions to the reading view and library.
4. Handle missing, invalid, or full storage without blocking story generation.
5. Clearly indicate that saved stories are local to the current browser/device.

### Done when

- Stories can be saved, reopened, and deleted.
- Storage failures do not break story creation or reading.
- Saved stories survive a page reload in the same browser.

## Phase 6: Production AI Provider (Optional, After Provider Selection)

### Steps

1. Choose an AI provider and confirm expected cost, latency, and data handling.
2. Add a server-side generation endpoint; never place provider credentials in client code.
3. Validate request shape and input lengths on the server as well as in the client.
4. Add age-appropriate generation instructions and a response validation step.
5. Normalize provider errors and malformed responses into the app's generation error model.
6. Add rate limiting, timeouts, and appropriate operational logging without logging sensitive story inputs.
7. Configure deployment secrets outside the repository.

### Done when

- Provider credentials are only available to server-side code.
- Requests, responses, errors, and timeouts are handled and tested.
- Privacy and retention behavior is documented before storing server-side story data.

## Phase 7: Verification and Release

### Steps

1. Run unit tests for validation, generation normalization, and storage behavior.
2. Run integration tests for create, paginated read, page navigation, visual fallback, retry, save, reopen, and delete flows.
3. Run the production build and address relevant errors.
4. Check keyboard navigation, focus visibility, labels, contrast, and status announcements.
5. Manually inspect small phone, tablet, and desktop layouts.
6. Review privacy behavior and ensure the app does not request unnecessary child information.
7. Document local setup, environment configuration, and deployment steps.

### Done when

- Automated checks pass and the production build succeeds.
- Core flows work at supported viewport sizes.
- Accessibility, privacy, and deployment checks are complete.

## Future Phase: Monetization (Optional)

Do not start this phase as part of the MVP. Revisit it after launch only if user feedback and product usage establish a need for paid plans.

### Steps

1. Define whether the product needs subscriptions, one-time purchases, or no paid access.
2. Decide which features, if any, belong in free and paid tiers.
3. Select a payment provider and review its fees, supported regions, and data/privacy requirements.
4. Design subscription purchase, renewal, cancellation, refund, and failed-payment flows.
5. Implement server-side payment event handling and entitlement checks; do not trust client-side payment state.
6. Add tests for payment events, subscription state changes, and access rules before enabling billing.

### Done when

- Paid access has a validated product rationale and clear user-facing terms.
- Payment secrets and entitlement decisions are handled server-side.
- Purchase, renewal, cancellation, refund, and failure cases are tested.

## Suggested Working Loop

For each step:

1. Make the smallest change that completes the step.
2. Run the closest relevant test, build, or type check.
3. Fix failures in that step before continuing.
4. Update this document if scope or decisions change.
5. Mark the phase complete only after its done criteria are met.
