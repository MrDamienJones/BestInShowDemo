# Requirements Document

## Introduction

This feature integrates a third-party booking service into an existing React 19 + Vite single-page application so that external visitors can schedule 1-on-1 meetings with the site owner. The document first evaluates the build-vs-buy decision (custom AWS platform vs. existing booking service), then compares Cal.com, Calendly, and Cronofy to recommend the best fit. The chosen service must embed into the React app, sync with Google Calendar, and be covered by unit tests.

## Glossary

- **Booking_Widget**: A UI component that renders the embedded scheduling interface from the chosen third-party booking service inside the React application.
- **Booking_Service**: The external scheduling platform (Cal.com, Calendly, or Cronofy) that provides calendar availability, appointment creation, and Google Calendar synchronization.
- **App**: The existing React 19 + Vite single-page application (`cal-booking-poc`).
- **Visitor**: An external, unauthenticated user who accesses the App to schedule a 1-on-1 meeting.
- **Host**: The site owner or team member whose calendar availability is exposed through the Booking_Widget.
- **Google_Calendar_Sync**: The bidirectional synchronization between the Booking_Service and the Host's Google Calendar so that new bookings appear on the calendar and existing events block availability.
- **Test_Suite**: The collection of Vitest + React Testing Library unit tests that verify the App's booking integration behavior.

## Requirements

### Requirement 1: Platform Decision — Build vs. Buy

**User Story:** As a site owner, I want a clear recommendation on whether to build a custom booking platform with AWS services or use an existing booking service, so that I can make an informed cost-effective decision.

#### Acceptance Criteria

1. THE App SHALL integrate with an existing third-party Booking_Service rather than a custom-built AWS booking platform.

> **Rationale (not a requirement — context for the decision):**
> Building a custom booking platform on AWS (e.g., DynamoDB for slot storage, Lambda for availability logic, API Gateway, SES/SNS for notifications, Cognito for auth) would require significant development effort (estimated 4–8 weeks for a minimal viable version), ongoing maintenance, and operational cost that is disproportionate to the current low-volume, 1-on-1 scheduling use case. An existing service provides calendar sync, conflict detection, timezone handling, email notifications, and embeddable widgets out of the box. At low volumes, the free tiers of existing services cover the need at zero or near-zero cost. A custom build only becomes justified at high scale with complex custom logic (multi-resource scheduling, payment workflows, deep domain-specific rules).

### Requirement 2: Service Selection — Cal.com vs. Calendly vs. Cronofy

**User Story:** As a site owner, I want a comparison of Cal.com, Calendly, and Cronofy, so that I can choose the service that best fits my needs for cost, embeddability, and Google Calendar sync.

#### Acceptance Criteria

1. THE App SHALL use Cal.com as the Booking_Service.

> **Rationale (not a requirement — context for the selection):**
>
> | Criterion | Cal.com | Calendly | Cronofy |
> |---|---|---|---|
> | **Type** | Scheduling platform (open-source, self-hostable, or cloud) | Scheduling platform (SaaS only) | Scheduling API/infrastructure (developer toolkit) |
> | **Free tier** | Generous free plan for individuals; unlimited event types | Free plan limited to 1 event type | No free plan; 14-day trial only |
> | **React embed** | Official `@calcom/embed-react` package with inline, popup, and floating button modes | JavaScript embed snippet (no official React package); requires manual script injection | JavaScript SDK; no pre-built React component; requires building custom UI on top of their API |
> | **Google Calendar sync** | Included on free plan | Included on free plan | Included (API-level integration) |
> | **Self-hosting option** | Yes (Docker, full control over data) | No | No |
> | **Customization** | Full source access if self-hosted; embed theming on cloud | Limited on free plan; more on paid | Full API control but requires building all UI |
> | **Cost at low volume** | $0 (free plan or self-hosted) | $0 (free plan, but limited) | ~$49/mo minimum after trial |
>
> **Recommendation:** Cal.com is the best fit because it offers a dedicated React embed package, a generous free tier, Google Calendar sync at no cost, and the option to self-host later if data residency or deeper customization becomes important. Calendly is a reasonable alternative but lacks an official React component and limits the free plan to one event type. Cronofy is a powerful API-first toolkit but is overkill and expensive for a low-volume 1-on-1 scheduling use case — it shines when you need to build a fully custom scheduling experience across multiple calendar providers at scale.

### Requirement 3: Embed the Booking Widget

**User Story:** As a visitor, I want to see an inline booking calendar on the page, so that I can select an available time slot and schedule a 1-on-1 meeting without leaving the site.

#### Acceptance Criteria

1. WHEN the App loads, THE Booking_Widget SHALL render an inline Cal.com scheduling interface inside the main content area.
2. THE Booking_Widget SHALL accept a configurable Cal.com username (or event-type URL slug) as a prop so that different Hosts can be supported without code changes.
3. WHEN the Visitor selects a time slot and confirms the booking, THE Booking_Widget SHALL complete the booking flow within the embedded interface without navigating the Visitor away from the App.
4. IF the Cal.com embed script fails to load, THEN THE Booking_Widget SHALL display a fallback message with a direct link to the Host's Cal.com scheduling page.

### Requirement 4: Google Calendar Synchronization

**User Story:** As a host, I want bookings to automatically appear on my Google Calendar and my existing events to block availability, so that I avoid double-bookings.

#### Acceptance Criteria

1. WHEN a Visitor completes a booking through the Booking_Widget, THE Booking_Service SHALL create a corresponding event on the Host's Google Calendar.
2. WHILE the Host has an existing event on Google Calendar during a time slot, THE Booking_Service SHALL mark that time slot as unavailable in the Booking_Widget.
3. WHEN a booking is cancelled through the Booking_Service, THE Booking_Service SHALL remove the corresponding event from the Host's Google Calendar.

> **Note:** Google Calendar sync is configured within Cal.com's admin settings (Connected Calendars). No custom code is required in the App for this requirement — it is fulfilled by the Booking_Service's built-in integration.

### Requirement 5: Unit Test Coverage

**User Story:** As a developer, I want unit tests for the booking integration components, so that I can verify correct rendering and behavior without depending on the live Cal.com service.

#### Acceptance Criteria

1. THE Test_Suite SHALL use Vitest as the test runner and React Testing Library as the component testing utility.
2. THE Test_Suite SHALL verify that the Booking_Widget renders the Cal.com inline embed element when the component mounts.
3. THE Test_Suite SHALL verify that the Booking_Widget passes the configured Cal.com username prop to the embed component.
4. THE Test_Suite SHALL verify that the Booking_Widget renders the fallback message with a direct link when the embed fails to load.
5. THE Test_Suite SHALL mock the `@calcom/embed-react` package so that tests execute without network calls to Cal.com.
6. WHEN a developer runs the test command, THE Test_Suite SHALL complete without requiring any external service credentials or network connectivity.

### Requirement 6: Loading State

**User Story:** As a visitor, I want to see a loading indicator while the booking calendar is initializing, so that I know the page is working and not broken.

#### Acceptance Criteria

1. WHILE the Booking_Widget is loading the Cal.com embed, THE App SHALL display a loading indicator in the Booking_Widget's container.
2. WHEN the Cal.com embed finishes loading, THE App SHALL replace the loading indicator with the Booking_Widget.
