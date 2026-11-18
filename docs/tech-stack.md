# Storypod Tech Stack

**Status:** Recommended stack for the first release  
**Scope:** Responsive web app with a Python API and paginated illustrated stories, initially using local story generation and designed for AWS deployment.

## Stack Summary

| Area | Technology | Why |
|---|---|---|
| Frontend language | TypeScript | Catches input and response-shape mistakes before runtime and gives the web UI a clear API contract. |
| UI | React | Component-based UI fits the form, loading/error states, reading view, and saved-story library. Its ecosystem is mature and broadly supported. |
| Build and development server | Vite | Fast local startup and builds, with a small configuration footprint for a client-side app. |
| Styling | CSS Modules or scoped plain CSS with CSS custom properties | Supports a distinctive responsive design without committing the first release to a large UI framework. Tokens can be shared through custom properties. |
| Icons | lucide-react | Provides consistent, accessible icon components for familiar controls without hand-drawn interface icons. |
| Backend language | Python | Matches the team's experience and has a strong ecosystem for AI provider integrations and text processing. |
| API | FastAPI | Provides typed request/response validation and a lightweight HTTP API between the web UI and story service. |
| Story generation, first release | Local Python generator behind a `generate_story(input)` service interface | Returns ordered story pages without requiring paid AI credentials, while keeping future AI providers isolated. |
| Page visuals, first release | Still illustrations bundled with or served alongside the app, referenced by each page | Guarantees each page has a visual without depending on a paid media API. The page contract also allows video as an optional media type. |
| Saved stories, first release | Browser `localStorage` behind a `StoryRepository` interface | Enables save/reopen/delete with no account or backend. The interface leaves room to add cloud storage later. |
| AWS API hosting, production | API Gateway and AWS Lambda for the FastAPI service | Runs the API without maintaining servers and scales with requests. A container-based AWS ECS Fargate deployment is an alternative if container operations are preferred. |
| AWS frontend hosting, production | S3 and CloudFront | Serves the built static web app from AWS with CDN delivery. |
| Cloud database, when needed | Amazon RDS for PostgreSQL | Uses a familiar relational database when account-based or cross-device story persistence is required. Not needed for the local-storage MVP. |
| Infrastructure as code | Terraform | Makes AWS resources reviewable, repeatable, and consistent across environments. |
| Source control and delivery | GitHub and Jenkins | GitHub holds the code; Jenkins can run tests and build/deploy steps if it is the team's standard CI/CD system. |
| Containers | Docker, optional | Useful for a consistent local API environment and for an ECS deployment; not required for Lambda. |
| Logging | AWS CloudWatch, with Splunk integration if required | Provides AWS-native application and runtime logs. Avoid recording child-specific story inputs; forward only operational data to Splunk if needed. |
| Unit and component tests | Vitest and React Testing Library | Fits the Vite toolchain and tests behavior at the service and component level without tying tests to implementation details. |
| Backend tests | pytest | Familiar Python test runner for API, validation, and generation-service behavior. |
| End-to-end tests | Playwright | Verifies the main create, read, and saved-story flows in a real browser at mobile and desktop sizes. |

## Why This Is the First-Release Stack

React, TypeScript, and Vite provide a small, well-supported web foundation. Python and FastAPI put story-generation logic behind a clear API boundary and align with the team's existing skills. A local Python generator, bundled illustrations, and browser storage make the paginated story experience testable without AI credentials, recurring model costs, or account setup. AWS is the recommended deployment target, but cloud deployment can wait until the local end-to-end flow is working.

The generator and storage interfaces are intentional boundaries. The local generator can later call a server-side AI provider, and browser storage can later be replaced by a persistent repository, while keeping the main UI and story data contract stable.

## Application Shape

```text
React UI
  -> typed story request
  -> Python FastAPI endpoint
       -> generate_story(input)
            -> local page generator + illustrations (first release)
            -> server-side text and optional media providers (later)
  -> typed page-based story result
  -> paginated reading view (text + image/video per page)
  -> StoryRepository
       -> localStorage (first release)
       -> PostgreSQL on Amazon RDS (optional later)

Production hosting:
  React build -> Amazon S3 -> CloudFront
  FastAPI -> API Gateway -> AWS Lambda
  AWS resources -> Terraform
```

Keep provider SDKs, secrets, and network-specific error handling out of React components. The browser must never contain a production AI API key. Keep secrets in AWS-managed secret storage or secure deployment configuration, never in Git.

The MVP runs on the developer's machine: Vite serves the frontend and FastAPI runs locally. The local story generator, bundled images, and browser storage do not require AWS, Docker, an AWS account, or an AWS emulator. AWS is only needed later if the team chooses to deploy the app or integrate AWS services.

## Fit for the Listed Tools

- **Use for this app:** Python, AWS, PostgreSQL if cloud persistence is added, Terraform, GitHub, Docker where useful, Jenkins if it is the team's CI standard, Postman for API checks, and Splunk integration if organizational logging requires it.
- **Use only if requirements call for them:** Oracle, MSSQL, Teradata, Ansible, Control-M, Apache Airflow, Qlik Sense, SAP BO, Tableau, and Amazon QuickSight. Story generation is an interactive request/response flow, so it does not need batch orchestration or a BI platform in the MVP. These tools may fit a future enterprise integration or reporting requirement.
- **Operating systems:** Develop locally on the team's supported OS. AWS Lambda manages its runtime; if the service moves to ECS, use a supported Linux container image. RHEL or Windows Server administration is not required for the proposed serverless deployment.

## Production Choices Deferred

- **AI provider and model:** Select after evaluating age-appropriate output quality, safety controls, cost, latency, and data handling.
- **AWS compute option:** The recommendation is API Gateway plus Lambda for a small request/response API. Choose ECS Fargate with Docker instead if the service needs long-running processes, custom runtime dependencies, or container-based operations.
- **Cloud database and accounts:** Defer RDS PostgreSQL until cross-device sync, account ownership, retention, and deletion requirements are confirmed. Browser storage is device-local and is not a backup.
- **Audio:** Defer until read-aloud requirements are defined; generated audio introduces additional accessibility, cost, and privacy decisions.
- **AI-generated visuals and video:** Keep page visuals behind a separate provider boundary. Use still illustrations first; select an image/video provider only after reviewing quality, child-safety controls, cost, latency, and data handling. Video generation is not assumed to be available in a free tier.

## Dependencies to Keep Small

Use dependencies when they solve a real need, but avoid adding a component framework, global state library, or routing library until the product requires their complexity. The initial experience can be a single-page app with local component state and a few focused services.

## Initial Project Commands

The project should provide standard commands for frontend and backend development, tests, and production builds. Exact command names should follow the generated project configuration; the expected capabilities are:

- Start the Vite frontend development server.
- Start the FastAPI service locally.
- Run Vitest and React Testing Library frontend checks.
- Run pytest backend checks.
- Run Playwright end-to-end tests.
- Create and verify the frontend production build and backend deployment artifact.

## Decision Record

- **Use now:** React, TypeScript, Vite, CSS, lucide-react, Python, FastAPI, a local page-based generator, bundled still illustrations, browser-local story storage, and focused frontend/backend tests.
- **Use for AWS deployment:** API Gateway, Lambda, S3, CloudFront, and Terraform. Add Jenkins, Docker, and Splunk integration where they match the team's delivery and operations standards.
- **Decide later:** Text/image/video AI providers, Lambda versus ECS Fargate, PostgreSQL/RDS, authentication, generated video, and audio.

See [architecture.md](architecture.md) for product architecture and [implementation-phases.md](implementation-phases.md) for the step-by-step delivery plan.
