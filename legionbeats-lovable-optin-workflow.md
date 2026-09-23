# legionbeats.com (Lovable) → GHL Legion Beats: Free Beats Opt-in Workflow

Sub-account: **Legion Beats** (`1cYXqvdqTNQgRpK3NC6q`)
Lovable project: `legionbeats.com-lovable-remake` (`7d8c3c3c-eed1-4714-bc70-b1669b848e73`)

## Where things stand (checked 2026-09-23)

- **The form code works.** `src/lib/ghl.functions.ts` POSTs JSON to whatever URL is in the
  `GHL_WEBHOOK_URL` secret. There are 3 forms on the page, and each sends a different `source`:
  `free-beats-hero`, `free-beats`, `free-beats-exit-intent`.
- **No leads have come through.** No contacts were added to the sub-account in the last 3 days from
  the form. The tag `lb (optin) lb.com2026` exists, but no contact has it.
- **The pipeline already exists:** `legionbeats.com free beats optin 2026`, with first stage
  `New Lead - Downloaded Free Beats`.
- **Existing workflows** (both published and edited today): `parts? legionbeats.com Free Beats Optin | WebHook Received`
  and `Free Beats Download Workflow`. The public API returns only names and status, not steps, so
  I couldn't see what's inside either one.
- **The GHL API can't create or edit workflows.** You have to build it in the builder, using the spec below.

## Payload Lovable sends

```json
{
  "first_name": "Gabe",
  "last_name": "Schillinger",
  "full_name": "Gabe Schillinger",
  "name": "Gabe Schillinger",
  "email": "someone@example.com",
  "phone": "+15555555555",
  "source": "free-beats-hero",
  "submittedAt": "2026-09-23T12:00:00.000Z",
  "page": "legionbeats.com"
}
```

`phone` is optional. It's left out of the payload when the field is blank.

## Workflow: `LB (Optin) legionbeats.com Free Beats | Inbound Webhook`

Build it **from scratch**: Automation → Workflows → Create Workflow → Start from Scratch.
After it's live and tested, set `parts? legionbeats.com Free Beats Optin | WebHook Received` to **Draft**
so you have only one webhook workflow and no double sends.

1. **Trigger: Inbound Webhook** (premium, about $0.01 per run)
   - Copy the webhook URL.
   - Send one test request so the fields become mappable. Paste the URL to Claude and it'll send the
     sample payload above, or submit the live form once.
2. **Create Contact** (or **Create/Update Contact**, whichever your builder shows)
   - First Name: `{{inboundWebhookRequest.first_name}}`
   - Last Name: `{{inboundWebhookRequest.last_name}}`
   - Email: `{{inboundWebhookRequest.email}}`
   - Phone: `{{inboundWebhookRequest.phone}}`
   - Source: `{{inboundWebhookRequest.source}}` (tells you which of the 3 forms converts)
3. **Add Tag:** `lb (optin) lb.com2026`
4. **Create/Update Opportunity**
   - Pipeline: `legionbeats.com free beats optin 2026`
   - Stage: `New Lead - Downloaded Free Beats`
   - Name: `{{contact.name}} - Free Beats`
   - Allow duplicate opportunities: **off**
5. **Send Email** (delivery)
   - From: the Legion Beats sending domain you're warming (`* LB Email Warmup 2026`)
   - Subject: `Your 5 free beats (download inside)`
   - Body: see copy below. **Needs the real download link.**
6. **If/Else:** Phone *is not empty*
   - Yes → **Send SMS** (copy below)
   - No → end
7. **Internal notification** (optional for the first week): email or SMS yourself `New LB.com optin: {{contact.name}} / {{contact.source}}`

Publish, then update `GHL_WEBHOOK_URL` in Lovable (Cloud → Secrets) to the new URL and republish the site.

## Copy

**Email**

> Subject: Your 5 free beats (download inside)
>
> Yo {{contact.first_name | default: "fam"}},
>
> Here are your 5 free beats, as promised:
>
> 👉 **[DOWNLOAD LINK]**
>
> HQ files, ready to write to. No strings.
>
> When you record something on one, reply and send it to me. I actually listen.
>
> Gabe / Legion Beats

**SMS** (phone given)

> Legion Beats: your 5 free beats just hit your inbox ({{contact.email}}). Didn't see it? Check promos/spam. Reply STOP to opt out.

## Compliance flag on SMS

The form's phone field has **no SMS consent language**. Before sending that SMS, add a line under the
phone field like: *"By entering your phone you agree to receive texts from Legion Beats. Msg & data
rates may apply. Reply STOP to opt out."* Without it, you're texting without documented consent,
which is an A2P/TCPA risk.

## Test checklist

1. Submit the live form with `gabe.gs+lbtest1@gmail.com`, name, and phone.
2. Workflow → Execution Logs: 1 run, with every step green.
3. Contacts: the contact exists with the tag and the Source filled in.
4. Opportunities: the contact is in `New Lead - Downloaded Free Beats`.
5. The email lands in the inbox (not promos/spam), and the download link works.
6. The SMS arrives.
7. Submit the same email again. It should not create a second opportunity.

## Hosting (switched 2026-09-23)

- Code: GitHub `LegionBeats/framer-freedom-project` (Lovable pushes here automatically).
- Host: Cloudflare Worker `tanstack-start-app` (account thisisthelegion@gmail.com). Workers Builds
  runs `npm run build` then `npx wrangler deploy` on every push.
- Secret: `GHL_WEBHOOK_URL` is set on the Worker. If the GHL webhook changes, update it there. Lovable's secret no longer affects the live site.
- DNS: `legionbeats.com` is a Worker custom domain. `www` is a CNAME to the root plus Page Rule (301 to root).
  Bulk Redirects `prettylinks_from_legionbeats` (194 links) are unchanged.
- The old WordPress host has been retired. To roll back, remove the Worker custom domain and re-add
  `A @ 199.16.172.5` and `A @ 199.16.173.151` (both Proxied).
- Lovable's "Publish" button no longer controls the live site.
