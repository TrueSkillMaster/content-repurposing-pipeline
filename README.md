# content-repurposing-pipeline

One YouTube video in, a week of social content out. A URL submitted through a form becomes a transcript, four platform-specific posts, matching AI images, a moderation queue — and, after approval, published posts.

**Status:** delivered (2026). Cuts manual content production time by ~90%.

> Architecture is documented here. Assistant instructions, API keys and the client's accounts are not included.

---

## The problem

A creator publishes long-form video but has no time to turn each one into posts for every platform. Copy-pasting the same text everywhere performs badly; writing four versions by hand every week doesn't happen.

## Pipeline

```
1. Airtable form            user submits a YouTube URL
2. Make.com                 → transcription API → transcript saved, status "Pending AI Analysis"
3. AI writers (×4)          OpenAI Assistants, one per platform, each with its own tone and format rules
                            LinkedIn · Facebook · Instagram · Telegram
4. Visual prompt agent      reads each post, writes an image prompt for it
5. Image generation         Replicate (Flux / Stable Diffusion), async with polling / sleep
6. Airtable "command center"  review · edit · regenerate — human in the loop
7. Status → "Approved"      triggers publishing scenario
8. Publish                  text + image per platform, status → "Published" (audit trail)
```

## Key decisions

**One specialised agent per platform.** A single prompt asked to "write for LinkedIn, Facebook, Instagram and a channel" averages them into one voice. Four assistants with platform-specific instructions produce four genuinely different posts.

**Image prompt written from the post, not from the video.** The visual matches what the reader actually sees next to it.

**Wait for assets before publishing.** Image generation is asynchronous. Publishing is blocked until both caption and image exist — no half-posts.

**Status-driven workflow.** The Airtable status field *is* the control panel. Changing "Review" → "Approved" is the only action a human needs to take; everything else is triggered by status changes.

**Asset library as a side effect.** Every transcript, post and image stays in the base — easy to reuse months later.

## Results

| | Before | After |
|---|---|---|
| Prep per video | ~30 min of manual transcript work | automatic |
| Content per video | written by hand, often skipped | 4 platform-specific posts + images |
| Total manual time | baseline | ~−90% (review and approval only) |

## Stack

`Make.com` `n8n` `OpenAI Assistants` `Supadata (transcription)` `Replicate — Flux / Stable Diffusion` `Airtable` `Social media APIs`
