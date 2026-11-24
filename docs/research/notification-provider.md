# Which notification provider fits Cadence?

Prices and limits checked: 2026-11-24

## TL;DR

- **Top pick: Knock (Free plan).** It is the only hosted free tier that covers every functional need: a React feed with bell and unread count, 10,000 messages/month, unlimited workflows, multi-day delays, batch/digest windows up to 90 days, recipient-timezone schedules, cancellation, preferences (workflow x category x channel), a hosted preference center and automatic unsubscribe headers. The catch is hosting. It is **US-only today**, and EU residency is only "expected by the end of 2026". A DPA with EU SCCs is published, so it meets the "US acceptable with a GDPR DPA" bar. Web push is not first-party; it runs through FCM.
- **Runner-up: Novu Cloud (Free), with Novu self-hosted (MIT) as the exit route.** It is the only free tier with **EU hosting (Frankfurt)**. The free tier is tight, though: 5 workflows, a **1-day maximum delay/digest**, a hard stop at 10,000 runs (HTTP 402), 24 h activity log, no hosted preference page and no List-Unsubscribe header support.
- **Not a fit for the free launch.** SuprSend Free has no preferences, digest or batching. Courier lists digest only under Enterprise and EU only on paid plans. MagicBell Free has 1,000 deliveries a month and no digest. OneSignal has no in-app inbox or feed, and its DPA appears to need a paid plan.
- **Unmet constraint.** No provider offers EU hosting and multi-day digests/delays and one-click List-Unsubscribe together on a free tier. No hosted provider in this list offers first-party VAPID web push together with digests on a free tier either.

Terminology: Cadence's audiences are **Students** and **Instructors**. Every provider below models recipients as generic users with custom properties. None of them hard-codes roles, so "Instructor" or "Student" goes in a custom property, list, topic or tenant/object that Cadence controls.

---

## 1. Knock

### Free tier
Source: https://knock.app/pricing
- $0, no card. **10,000 messages/month**. Bounced or failed messages are not counted.
- Unlimited stored profiles, unlimited team seats, unlimited workflows, broadcasts and channels.
- Logs are kept 30 days. **Feed retention is unlimited.**
- Included: preferences, batching/digests, scheduled workflows, multi-tenancy, Google SSO, SOC 2 Type 2 / GDPR.
- Excluded: **Knock branding removal**, translations, per-tenant branding and preferences, SAML, audit logs.
- Limit behaviour: "We don't hard cap your messages." Knock contacts you about billing the following month. The next tier is Starter at **$250/month** for 50k messages, then $0.005 each.
- **Metering** (https://docs.knock.app/manage-your-account/knock-plans.md): one message is one delivery "to a single user on a single channel". **In-app feed messages count.** One workflow that sends email plus in-app is 2 messages.
- **What breaks first:** the 10k message count. In-app plus email doubles usage per notification.

### Hosting
- **No EU region yet.** "We expect to support EU data residency by the end of 2026." (https://docs.knock.app/manage-your-account/knock-plans.md, https://docs.knock.app/developer-tools/security.md). As of 2026-11-24 the docs still describe it as future, so re-check before launch.
- No AWS region is stated on the security page.
- DPA: https://knock.app/legal/data-processing-addendum. It is part of the Terms of Service and includes EU SCCs (Modules 2 and 3, Decision 2021/914), Irish law and a UK Addendum. The page gives no last-updated date. Trust center: https://trust.knock.app.
- Self-host: none (proprietary SaaS).

### Channels
- **In-app:** `@knocklabs/react`, with `NotificationIconButton` (a bell with an unread/unseen count), `NotificationFeedPopover` and `NotificationFeed` (full page). The old `@knocklabs/react-notification-feed` is superseded. (https://docs.knock.app/in-app-ui/react/overview.md)
- **Email:** bring your own provider. Postmark, Resend, Amazon SES and SendGrid are documented (https://docs.knock.app/llms.txt, integrations/email).
- **Web push:** "Knock does not have a first-party web push channel." It goes through **FCM**, using the Firebase VAPID key and token (https://docs.knock.app/tutorials/sending-web-push-notifications-with-fcm.md).

### Preferences
- The PreferenceSet has `workflows`, `categories`, `channel_types` and `channels`. Defaults are opt-in, an environment-level default set is possible, and workflows can override preferences (https://docs.knock.app/preferences/overview.md).
- **Hosted preference center:** a signed URL at `p.knock.app`, configured in the dashboard, covering the default set only (https://docs.knock.app/preferences/hosted-preference-center.md). No plan restriction is stated. A React custom-center guide also exists.
- **One-click unsubscribe:** marking a workflow "Commercial" "adds required unsubscribe headers to all emails". The page does not literally name `List-Unsubscribe` or `List-Unsubscribe-Post`, so confirm with a test send (https://docs.knock.app/preferences/commercial-unsubscribe.md). The template variable is `vars.commercial_unsubscribe_url`.

### Workflows
- **Batch/digest** (https://docs.knock.app/designing-workflows/batch-function.md): fixed, sliding, dynamic, relative and recurring window rules (hourly, daily, weekly or monthly, in the recipient's timezone). The maximum window is 90 days and a batch key is supported. Only a render limit above 10 items needs Enterprise.
- **Delay:** a delay step exists (https://docs.knock.app/designing-workflows/delay-function.md). The page was not fetched in detail.
- **Schedules:** a "managed, recipient-timezone-aware cron job", either recurring or one-off, created through the API only with `client.schedules.create()`. You cannot auto-add all users (https://docs.knock.app/send-notifications/triggering-workflows/schedules.md).
- **Cancel:** pass a `cancellation_key` at trigger time and cancel during a delay, batch, wait-for-event or fetch-retry step (https://docs.knock.app/send-notifications/canceling-workflows.md).

### Integration
- Node: `@knocklabs/node` (Apache-2.0), `client.workflows.trigger(key, { recipients, data })` (https://github.com/knocklabs/knock-node).
- Users are identified with `PUT /v1/users/{id}` or inline at trigger time. Custom properties are allowed up to 256 kB. Objects and tenants exist (https://docs.knock.app/managing-recipients/identifying-recipients.md).
- Lock-in: workflow definitions live in Knock. A Management API SDK exists (https://github.com/knocklabs/knock-mgmt-node). Bulk data export was not verified.

---

## 2. Novu Cloud

### Free tier
Sources: https://novu.co/pricing, https://docs.novu.co/platform/developer/limits.md, https://docs.novu.co/platform/account/billing.md
- **10,000 workflow runs/month.** A run is one workflow for one subscriber. **Channels and steps do not add runs**, so in-app plus email plus push is 1 run.
- Unlimited subscribers. **3 team members.** **5 workflows.** 20 steps per workflow. 2 environments.
- **Delay max 1 day, digest max 1 day**, throttle max 1 h.
- Inbox feed retention is 30 days. Activity feed retention is 1 day (24 h).
- Novu branding stays on the free tier; removal starts at Pro.
- **Hard stop:** Free "stops new workflow runs at its included 10,000". Triggers return HTTP **402** until the period resets. Pro starts at **$30/month**.
- **What breaks first:** the 5-workflow cap, which limits the number of categories, and the 1-day delay/digest cap. A "nudge after 7 days of inactivity" or a weekly digest cannot be built inside Novu Free. Cadence would have to trigger these from its own scheduler.

### Hosting
- **EU region on all plans, including Free:** Frankfurt. Endpoints are `eu.api.novu.co` and `wss://eu.socket.novu.co`. The region is chosen at signup and keys are region-bound (https://docs.novu.co/platform/additional-resources/security.md).
- DPA: https://novu.co/dpa. It is an integral part of the Terms (automatic) and references the 2021/914 SCCs. It does not explicitly say whether it covers the free plan.

### Channels
- **In-app:** `@novu/react` (React 18/19), `<Inbox />`. The bell opens notifications plus a preferences panel (https://docs.novu.co/platform/inbox/setup-inbox.md). Production needs HMAC (`subscriberHash`). A standalone unread-badge API was not verified on that page.
- **Email:** bring your own provider. Resend, Postmark, SES, SendGrid, Brevo, Mailgun, SMTP and others are supported. There is a demo provider for testing (https://docs.novu.co/platform/integrations/email).
- **Web push:** **no native VAPID provider** is documented. It goes through **FCM**, OneSignal, Pusher Beams or Pushpad (a web-push service), with APNs and Expo for mobile (https://docs.novu.co/platform/integrations/push.md).

### Preferences
- Workflow defaults, then subscriber global per-channel preferences, then subscriber per-workflow per-channel preferences. Critical workflows hide the opt-out (https://docs.novu.co/platform/concepts/preferences.md).
- A preferences UI is embedded in `<Inbox />`. **No hosted preference page** is documented.
- **No List-Unsubscribe support:** "By default, Novu does not add any unsubscribe links". You build a token-secured link yourself (https://docs.novu.co/platform/integrations/email). The RFC 8058 header feature request is **open**: issue #12837, opened 2026-10-08 (https://github.com/novuhq/novu/issues/12837).

### Workflows
- **Digest:** regular or scheduled (minute, hour, day, week or month, in the subscriber's timezone), with a "group by" key (https://docs.novu.co/platform/workflow/add-and-configure-steps/configure-action-steps/digest.md). It is capped at 1 day on Free.
- **Delay:** fixed, scheduled or dynamic (ISO timestamp), capped at 1 day on Free (https://docs.novu.co/platform/workflow/add-and-configure-steps/configure-action-steps/delay.md).
- **Recurring triggers:** no cron-style trigger was found in the docs. Use an external scheduler.
- **Cancel:** `DELETE /v1/events/trigger/{transactionId}` cancels pending delay and digest jobs (https://docs.novu.co/api-reference/events/cancel-triggered-event).

### Integration
- Node: `@novu/api`, `novu.trigger({ to: { subscriberId }, workflowId, payload })`, with `serverURL: "https://eu.api.novu.co"` for the EU (https://docs.novu.co/platform/sdks/server/typescript.md).
- Subscribers are keyed by your own ID. The Inbox can create them just-in-time, and topics are supported.
- Lock-in is lowest of the hosted options, because the core is MIT-licensed and a self-host path exists.

## 3. Novu self-hosted (Community)

Sources: https://github.com/novuhq/novu, https://docs.novu.co/community/self-hosting-novu/overview.md, https://docs.novu.co/community/self-hosted-and-novu-cloud
- **License:** the core is MIT; the `enterprise/` folder is under a commercial license. The repo has about 40.4k stars.
- **What you run:** API, Worker, WS and Dashboard services, plus **MongoDB**, **Redis** (AOF on the queue) and **S3-compatible storage** (10 GB minimum).
  - Single-VM option: 4 vCPU / 8 GB for the services, plus MongoDB at 2 GB RAM / 20 GB disk, plus Redis at 2 GB.
  - Recommended multi-VM option: 3 VMs each for API, Worker and WS, 2 Redis clusters and Mongo M20+.
  - This is a large operational step up from Cadence's single Node + SQLite app.
- **Gaps versus Cloud:** 1 team member, no branding removal, no translations, no snooze, no RBAC/MFA/custom environments, no email activity tracking, no social login.
- **Kept:** digest, code-first and GUI workflows, Inbox components and the preferences component.
- Workflow, retention and subscriber limits "depend on your infrastructure". Whether the delay/digest caps are lifted when self-hosted is **not stated explicitly**, so treat it as unverified.
- The comparison page has **no date**, and Novu's Community v2.2.0 changelog says it moved toward Cloud parity, so the table may be stale.
- Hosting is wherever you deploy it, so EU is fully under Cadence's control.

---

## 4. Courier

### Free tier (Developer)
Source: https://www.courier.com/pricing
- **10,000 sends/month.** A send is "one template sent to one recipient". A multi-channel template still counts as **one** send. Test and custom-environment sends count the same.
- Included: journeys and broadcasts, all SDKs, Google/GitHub SSO, email and chat support.
- MAU, seats, log retention and branding are **not stated** on the pricing page.
- Over 10k you must move to Business at **$0.005 per send, billed from the first send** (10k sends = $50).
- **Digest & batching is listed only under Enterprise.** "MSA, DPA, BAA & legal review" is also listed under Enterprise; the standard DPA is still linked below.
- **What breaks first:** digests, which are not on Free if the pricing page is right, and the move to a paid plan once you pass 10k.

### Hosting
- EU datacenter (`app.eu.courier.com`, `api.eu.courier.com`) is available "on every paid plan: Business and Enterprise", so **not Free** (https://www.courier.com/docs/platform/workspaces/eu-datacenter.md).
- DPA: https://www.courier.com/data-processing-addendum. It includes EU SCCs (Module 2) and a UK Addendum, and states the data center infrastructure "is located in the United States". Last updated 2026-09-12. Whether it covers Free is not stated.
- Self-host: none.

### Channels
- **In-app:** `@trycourier/courier-react` (React 18+; `-react-17` variant), with `CourierInbox`, `CourierInboxPopupMenu`, `CourierToast` and `CourierPreferences`. Auth is a backend-minted JWT (https://www.courier.com/docs/sdk-libraries/courier-react-web.md). Whether the popup trigger shows an unread count is not stated on that page.
- **Email:** bring your own provider (SendGrid and others).
- **Web push:** through FCM, OneSignal, Pusher Beams, MagicBell or others. No native VAPID channel is documented (https://www.courier.com/docs/integrations/push/overview.md).

### Preferences
- A hosted preference page (`view.notificationcenter.app/p/<id>`) groups topics into sections, with per-section channel selection (https://www.courier.com/docs/guides/build-a-preference-center.md).
- **List-Unsubscribe + List-Unsubscribe-Post (one-click)** is supported per topic. It is automatic only on SendGrid; other providers need a provider override (same page). No plan requirement is stated.

### Workflows
- Journeys support delay nodes and `POST /journeys/cancel` with a `cancelation_token` (https://www.courier.com/docs/guides/build-an-onboarding-sequence.md).
- Digests are topic-based schedules the user can pick, plus a Batch node (https://www.courier.com/docs/guides/send-a-daily-digest.md).
- **Contradiction:** the digest guide states no plan requirement, but the pricing page lists "Digest & batching" under Enterprise only.
- A recurring or cron trigger was not found in the docs.

### Integration
- Node SDK: https://www.courier.com/docs/sdk-libraries/node.md (not fetched in detail). Users are profiles keyed by `user_id`.

---

## 5. MagicBell

### Free tier (Builder)
Source: https://www.magicbell.com/pricing
- **1,000 deliveries/month**, counted **per channel**: 100 users in-app plus 50 of them by email is 150. One project, unlimited users and team members, all integrations.
- Overage for Builder is not stated. Startup costs **$249/month** for 50k deliveries.
- Retention, branding, workflow limits and EU are **not stated**.
- **What breaks first:** 1,000 deliveries. With in-app and email both counting, that is roughly 500 multi-channel notifications.

### Hosting
- The security page names no region and no EU option. It says MagicBell "provides data processing agreements" but has **no DPA link** (https://www.magicbell.com/security). Unverified.
- Self-host: none.

### Channels
- **In-app:** `@magicbell/react`, with `floating-inbox` (an `InboxButton` bell), `inbox` (full feed), `UserPreferences` and `webpush-button` (https://www.magicbell.com/docs/libraries/magicbell-react). An unread badge on the bell is not stated.
- **Email:** SES, Mailgun, SendGrid, SMTP and Ping Email (https://www.magicbell.com/docs/channels/email). Resend and Postmark would need SMTP.
- **Web push:** **native VAPID**. You supply a key pair and register tokens via `PUT /v2/channels/web_push/tokens` (https://www.magicbell.com/docs/channels/web_push).

### Preferences
- Category x channel preferences and topic unsubscribes are supported, with a `UserPreferences` component.
- No hosted preference page and **no List-Unsubscribe documentation** were found.

### Workflows
- `wait` (seconds), `if` conditions, `pause` and `abort` are available (https://www.magicbell.com/workflows).
- **No digest or batching** and no calendar scheduling are documented.

### Integration
- Browser/Node client: https://www.magicbell.com/docs/libraries/magicbell-js. Users authenticate with a User JWT.

---

## 6. SuprSend

### Free tier
Source: https://www.suprsend.com/pricing
- **10,000 notifications/month**, counted per channel, but a digest or batch counts as 1.
- Unlimited seats. 30-day logs. SuprSend branding on the inbox.
- Processing continues over the limit and SuprSend contacts you.
- **Not on Free: user preferences (No), digest (No), batching (No), throttling, recipient-timezone send.** Preferences and digest only arrive at **Business ($250/month)**. Essentials ($100/month) still lacks them.
- **What breaks first:** missing features, not volume. It fails the preferences and digest requirements outright.

### Hosting
- The DPA says workspace data is stored in the **US or EU** hosting region, and that support staff may access it from India (https://www.suprsend.com/data-processing-addendum). It includes SCCs where the DPF does not apply.
- **Contradiction:** pricing lists "Custom data center region" as Enterprise-only. Whether a Free workspace can choose EU is **unverified**.
- Security page: https://docs.suprsend.com/docs/security.
- **Self-host: Enterprise licence only.** It runs on Kubernetes 1.29+ with PostgreSQL 17+, ClickHouse, two Redis instances, NATS and optionally OpenSearch (https://docs.suprsend.com/docs/self-hosted/introduction.md).

### Channels
- **In-app:** `@suprsend/react`, with `<Inbox />` and `NotificationFeed` (https://www.npmjs.com/package/@suprsend/react, https://docs.suprsend.com/docs/react-inbox-integration.md).
- **Email:** bring your own provider (Resend, Postmark, SES, SendGrid and others).
- **Web push:** native VAPID. You host your own `serviceworker.js` that imports `@suprsend/web-sdk` and call `webpush.registerPush()` (https://docs.suprsend.com/docs/react-webpush.md).

### Workflows
- Digest, batch and delay nodes, plus a cancel-workflow API, are documented (https://docs.suprsend.com/reference/cancel-workflow.md). Per pricing, digest and batch are not on Free.

---

## 7. OneSignal

### Free tier
Source: https://onesignal.com/pricing
- **10,000 web push subscribers**, with a 10k-subscriber cap per send.
- Mobile push and in-app messages are unlimited up to **1,000 MAU**.
- **10,000 email sends/month.** **3 active journeys.** Admin-only permissions.
- 30-day reporting history.
- Growth costs **$19/month** plus usage.
- **What breaks first:** functional gaps. There is no inbox and no digest, and the 3-journey cap is tight.

### Hosting
- "Our primary data centers are located in the European Union" (https://documentation.onesignal.com/docs/en/data-questions). The DPA PDF says Google Cloud in the Netherlands (https://media-cms.onesignal.com/cms/Files/OneSignal_EU_DPA.pdf, https://onesignal.com/dpa).
- **DPA appears to be paid-only.** The docs say to "see our Paid Plans" for a DPA or SCCs (https://documentation.onesignal.com/docs/en/handling-personal-data), and the pricing page marks Free as GDPR "Compliant" without a DPA.
- Self-host: none.

### Channels
- **No in-app inbox or feed.** OneSignal "does not currently store the history of messages sent to each individual user". You build the feed yourself (https://documentation.onesignal.com/docs/en/create-an-activity-feed.md). Its "in-app messages" are banners and modals.
- **Email:** bundled OneSignal Email, or bring SendGrid, Mailgun (US region only) or Mandrill (https://documentation.onesignal.com/docs/en/email-setup).
- **Web push:** OneSignal's own Web SDK plus a same-origin `OneSignalSDKWorker.js` (https://documentation.onesignal.com/docs/en/onesignal-service-worker.md).

### Preferences
- A preference center is **build-it-yourself** with tags and subscription toggles: "OneSignal does not provide any APIs for creating the Preference Center layout" (https://documentation.onesignal.com/docs/en/preference-center.md).
- List-Unsubscribe was not documented on the pages fetched. An unsubscribe-with-token API exists.

### Workflows
- Journeys support Wait Until and exit rules.
- **No digest** was found.
- `DELETE /notifications/{id}` cancels scheduled or not-yet-delivered messages (https://documentation.onesignal.com/reference/cancel-message). The scheduling parameters (`send_after` and similar) were not verified.

### Integration
- Server SDKs: https://documentation.onesignal.com/docs/en/server-sdk-reference.md. Users are identified by `external_id`.

---

## Email provider free tiers (for bring-your-own setups)

| Provider | Free tier | Notes |
|---|---|---|
| Resend | **3,000 emails/month, 100/day**, 3 domains, 30-day retention (https://resend.com/pricing) | An EU sending region (eu-west-1, Ireland) exists (https://resend.com/docs/dashboard/domains/regions), but Resend says stored data is not moved to the EU (https://resend.com/security/gdpr) |
| Postmark | **100 emails/month**, no expiry, no overage (https://postmarkapp.com/pricing) | The next tier is $15/month for 10k emails. 100/month is only enough for development |

Implication: with Knock or Novu plus Resend Free, **100 emails a day is the real email ceiling** at launch, well before the 10k notification cap.

---

## Comparison table

| | Knock Free | Novu Cloud Free | Novu self-host | Courier Free | MagicBell Free | SuprSend Free | OneSignal Free |
|---|---|---|---|---|---|---|---|
| Free volume | 10k msgs (per channel, in-app counts) | 10k runs (per subscriber, all channels) | Unlimited | 10k sends (per recipient, all channels) | **1k** deliveries (per channel) | 10k (per channel) | 10k web-push subs, 10k emails, 1k MAU |
| At limit | No hard cap, billed later | **Hard stop (402)** | n/a | Must go paid ($0.005 from first send) | Overage (rate unstated) | Continues, sales contact | n/a |
| Workflows / seats | Unlimited / unlimited | **5** / 3 | Infra-bound / **1** | Not stated | Not stated / unlimited | Not stated / unlimited | **3 journeys** |
| EU hosting on free | **No** (expected end of 2026) | **Yes (Frankfurt)** | Yes (you host) | No (paid only) | Not stated | Unclear (DPA says US/EU; pricing says Enterprise) | Yes (EU primary DC) |
| DPA | Yes, with SCCs | Yes (in Terms) | n/a | Yes, with SCCs (US DC) | Claimed, no link | Yes, with SCCs | **Paid plans** |
| React inbox | `@knocklabs/react` | `@novu/react` | `@novu/react` | `@trycourier/courier-react` | `@magicbell/react` | `@suprsend/react` | **None** |
| Web push | FCM only | FCM / OneSignal / Pushpad | same | FCM / OneSignal / Beams | **Native VAPID** | **Native VAPID** | **Native (own SDK)** |
| Prefs (cat x channel) | Yes | Yes (workflow x channel) | Yes | Yes (topic x channel) | Yes | **No on Free** | DIY tags |
| Hosted pref page | Yes | No (in Inbox) | No | Yes | No | No on Free | No |
| One-click unsubscribe header | Yes (commercial workflows) | **No** (issue open) | No | Yes (auto on SendGrid) | Not documented | Not verified | Not documented |
| Digest | Yes, up to 90 days | Yes, **max 1 day** | Yes | **Enterprise only** (per pricing) | **No** | **No on Free** | No |
| Delay | Yes | **Max 1 day** | Yes | Yes | Yes (`wait`) | Yes | Wait Until |
| Recurring / scheduled trigger | **Yes (schedules)** | No (external cron) | No | Not found | No | Broadcasts | Scheduled sends |
| Cancel pending | `cancellation_key` | `transactionId` | same | `cancelation_token` | `abort` | API | DELETE message |
| Branding on free | Knock branding | Novu branding | Novu branding | Not stated | Not stated | SuprSend branding | Not stated |

## Ranked recommendation

1. **Knock (Free).** It meets every functional requirement on the free tier:
   - feed component with bell and unread count
   - category x channel preferences, a hosted preference center and unsubscribe headers
   - batch up to 90 days, so weekly digests work
   - multi-day delays and recurring schedules for engagement nudges
   - cancellation
   - unlimited workflows and seats

   It does not hard-cap. Hosting is US, covered by an SCC-backed DPA, which meets "US acceptable with a GDPR DPA". Watch for the promised EU region, which is due by the end of 2026.

   Costs to plan for:
   - Web push needs a Firebase project (FCM).
   - Each in-app plus email notification uses 2 of the 10k messages.
   - The jump to paid is steep: $250/month.
2. **Novu Cloud Free (EU), with Novu self-hosted as the long-term exit.**
   - Strengths: the only EU-hosted free tier, cheaper upgrades (Pro from $30/month), channel-agnostic run counting, and an MIT core that reduces lock-in.
   - Why it ranks second: the free tier's 1-day delay/digest cap and 5-workflow cap force Cadence to run its own scheduler for nudges and weekly digests. Cadence also has to build its own unsubscribe link, header and landing page, and a 402 hard stop at 10k runs is a production risk.
   - Self-hosting removes the caps but adds MongoDB, Redis, S3 and 4+ services next to a SQLite app.

Not recommended for the free launch:
- **SuprSend:** no preferences or digest on Free.
- **Courier:** digest is Enterprise and EU is paid.
- **MagicBell:** 1k deliveries and no digest.
- **OneSignal:** no inbox and the DPA is paid. It is still a good push-only add-on, and Novu and Courier can route push through it.

## Constraints no provider fully meets (free tier)

- **EU hosting plus full workflows plus one-click unsubscribe.** Only Novu offers EU on Free, and it lacks multi-day digests and delays and List-Unsubscribe headers.
- **First-party VAPID web push plus digests.** MagicBell and SuprSend have native VAPID but no digest on Free. Knock, Novu and Courier rely on FCM or another third-party push service.
- **Email at volume.** Every option except OneSignal is bring-your-own. Resend Free caps at 100 emails/day and Postmark Free at 100/month.

## Contradictions, recent changes and unverified items

- **Knock EU residency** is "expected by the end of 2026" in the current docs. It is not live as of 2026-11-24 and could change any week.
- **Courier digest:** the pricing page lists "Digest & batching" under Enterprise; the digest guide states no plan restriction. Courier's DPA was last updated 2026-09-12.
- **SuprSend EU:** the DPA says the workspace region is US or EU; pricing says custom data center regions are Enterprise-only.
- **Novu:**
  - The pricing-page feature table was garbled when fetched, so the delay and digest limits come from the limits doc.
  - The self-hosted comparison page is undated and may predate Community v2.2.0.
  - The List-Unsubscribe feature request (#12837) was opened 2026-10-08 and is still open.
- **OneSignal:** pricing shows Free as GDPR "Compliant" without a DPA. The docs point DPA requests to paid plans, while also stating EU primary data centers.
- **MagicBell:** no DPA link, region, retention or List-Unsubscribe information was found. Treat as unverified.
- **Not fetched in detail:** Knock's delay-step page, Courier's Node SDK page and OneSignal's scheduling parameters.
- Only primary sources were used. Third-party comparison pages found by search (Railway templates, other GitHub forks' issues, competitor blogs) were deliberately ignored.
