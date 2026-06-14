# instantpay-demo
Using the API of a major provider of Instant Pay solutions, this serves as an interactive walkthrough of how to build a simple iFrame integration to the provider's platform.
# GigShifts Instant Pay: Branch Integration Walkthrough

An interactive, single-file walkthrough that maps a real business goal (adding instant payouts to a gig staffing platform) to the concrete technical work required to ship it on the Branch API.

Open the live version here: **[insert GitHub Pages URL]**

---

## The Scenario

GigShifts is an app-based gig staffing platform that connects businesses with 1099 contractors. Today it pays workers via ACH on a weekly cadence, running everything through its own backend. The company wants to catch up to platforms like Uber by offering instant pay at the end of each shift, using payout speed as a lever for contractor acquisition and retention.

The open question is the one every platform asks before committing to a payments integration: *how much does this actually cost us to build, and what do we own versus what does the vendor own?*

This walkthrough answers that question by stepping through the entire Branch integration in the order an engineering team would encounter it, from API authentication through final funds settlement.

## Who This Is For

The walkthrough is built to be read by three people in the same room, because they each judge a payments integration on different terms:

- **Product** wants a clear integration roadmap that drops cleanly into the backlog, including the end-to-end worker onboarding experience.
- **Engineering** wants the security model, the exact API surface area, and confidence the work fits inside a sprint.
- **Finance** wants proof that instant pay does not tie up working capital, and that tax compliance is handled.

Each screen is annotated so a reader from any of those three seats can find the answer that matters to them.

## What It Covers

The demo moves through eight steps, each one a real integration touchpoint:

- **Executive Overview** frames the business case, the contractor benefit, and the technical reality in plain terms.
- **Getting Started** shows API authentication against the sandbox, plus the developer tooling and key-handling practices that come with onboarding.
- **Create Employee** is the single backend call that registers a worker, carrying no sensitive PII.
- **Embed Branch Direct** is the one piece of front end GigShifts builds: a single iframe or WebView that hosts every KYC, identity, and payout-setup screen.
- **Listen for Webhooks** walks the onboarding lifecycle events, including which single event gates the ability to pay a worker.
- **Create a Disbursement** is the one call that pays a worker instantly, with built-in duplicate protection and a full failure-handling reference.
- **Funds Flow** explains how GigShifts settles with Branch afterward via ACH pull, and why no pre-funding is required.
- **Technical Lift** sums the entire footprint: a handful of endpoints, zero PII liability, and a clear line between what the platform owns and what Branch owns.

## Product and Scoping Decisions

The most important decision here is what to *leave out*. A payments integration can look enormous on a whiteboard, so the walkthrough is deliberately structured to show how small the real footprint is:

- **Lead with the lift, not the feature list.** The closing screen reduces the entire integration to a short, countable set of touchpoints. For a team scoping a sprint, "three server calls and one embed" is a more useful answer than a feature tour.
- **Separate ownership explicitly.** Every screen distinguishes what GigShifts builds from what Branch hosts. This is the difference between a project that looks like a quarter of work and one that fits in a sprint, and it is the single biggest driver of the build-versus-buy decision.
- **Make the PII story unmissable.** Because Branch collects SSN, date of birth, and address directly from the worker inside the embedded portal, sensitive data never touches GigShifts servers. That is called out repeatedly because it removes an entire category of compliance scope.
- **Show the failure path, not just the happy path.** The disbursement step includes the full set of reason codes and the correct platform response to each. A demo that only shows success is not credible to an engineer who has shipped a payments integration before.

## Architecture and Build Notes

- **Single-file HTML.** All CSS and JavaScript are embedded in one file with no build step, no package manager, and no framework. It runs anywhere a browser does and deploys straight to GitHub Pages.
- **Self-contained navigation.** A sidebar and footer step through the eight screens. State is held in plain JavaScript, so there is nothing to install and nothing to configure.
- **Mocked, not live.** API calls, payloads, and responses are illustrative and modeled on the real Branch API shapes. The goal is to communicate the integration accurately, not to hit a live endpoint. The request and response bodies reflect the actual field names, status codes, and lifecycle events documented by Branch.
- **No external dependencies.** Syntax highlighting and all interactivity are hand-rolled in vanilla JavaScript to keep the file portable and dependency-free.

## Domain Context

A few details in the walkthrough that reflect how this kind of integration works in practice:

- **The payability gate is a single webhook event.** A worker cannot be paid until `PAYMENT_PROFILE_ACTIVATED` arrives. Building the listener around that one gate is the difference between a clean integration and a stream of failed disbursements.
- **Idempotency is enforced through a caller-supplied `external_id`.** GigShifts generates a unique receipt per shift, so a network retry or a double-click can never pay the same shift twice.
- **Settlement is a priority ACH pull, not pre-funding.** Branch pays the worker first and invoices the platform afterward, which is what keeps the feature off the working-capital line.
- **Webhook payloads are partially encrypted.** Outer routing fields arrive in plaintext while the sensitive `data` block is encrypted, so a listener can route and deduplicate before it ever decrypts anything.

## Running It Locally

No build tools required. Either:

1. Open the HTML file directly in any modern browser, or
2. Serve the folder with any static server, for example `python3 -m http.server`, then visit the local address it prints.

To publish on GitHub Pages, rename the demo file to `index.html` (or set it as the Pages entry point) and enable Pages on the repository.

---

Built by John Simpson, fintech and payments product and pre-sales professional with 20 years in enterprise payments.
