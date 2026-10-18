---
name: personal-blog-writer
description: Turn unstructured personal material (notes, scratch pads, discussions, chat logs, transcripts, pasted text, or any folder the user points to) into a clean first-person blog post in the author's own voice. Add Mermaid diagrams, generated images (uploaded to ImageKit), or credited online images whenever a visual genuinely helps the reader, and skip decorative ones. Use when the user gives raw material and wants it organized into a blog post, or asks to polish a blog draft, fix its grammar, or make it less AI-sounding. Do not use for rewriting someone else's writing or for corporate, marketing, or LinkedIn-style content.
---

# Personal Blog Writer

Turn raw material into a publishable blog post that still sounds like the author. The job is to unclutter and organize, not to rewrite. The author's voice, opinions, phrasing, and personality stay intact. The output should look like the author on a tidy day, not like a different writer.

## Inputs

The input is whatever the user points to: a folder (e.g., `raw/k8s/`), uploaded files, pasted text, a chat export, a transcript, a discussion thread, or a mix. Read everything in the given location. If no material is given, ask for it and stop.

## Workspace layout

Paths are relative to the workspace root (e.g., `~/my-blogs/`).

- `raw/` holds input material. Never write output here.
- `blogs/<blog-slug>/<blog-slug>.md` holds the finished blog body.
- `generated/images/<blog-slug>/` holds images the skill creates.

`<blog-slug>` is the post title in lowercase with hyphens, e.g., `kubernetes-intro`. Create the folders if they don't exist.

## Outputs

Every run produces:

1. **Metadata in the chat reply**, as plain labeled lines:
   - **Post Title:** one title
   - **Excerpt / SEO Description:** one to two sentences, under 160 characters
   - **Tags:** comma-separated, lowercase, 3 to 6 tags
2. **The blog body** at `blogs/<blog-slug>/<blog-slug>.md`. It contains only the post content. The title, excerpt, and tags never go inside the file. A top-level heading is fine if the post uses one.
3. **Any visuals** the post needs, following Step 4b.

If the material is too thin for a real post, say so in the reply and do not create a file.

## Pipeline

Run these steps in order.

### Step 1: Read everything

Read all of the material. Do not skip fragments, half-sentences, stray lines, tangents, or the back-and-forth of a chat. Note the core problem, the main point or the path to the solution, and any code, commands, file paths, error messages, or numbers that must stay exact. Filter out conversational rabbit holes, side debates, and off-topic tangents that naturally occur in chat dumps to keep the post tight and focused.

### Step 2: Propose an outline and wait for approval

Let the content decide the order. Show the user:

- A proposed title and slug
- The sections in order, each with a one-line description
- A short note on why the order works, if it is not obvious
- Anything dropped as duplicate or off-topic, with the reason

Wait for the user to approve or edit the outline before drafting. Skip this stop only if the user has said to write straight through, for example "just write it."

### Step 3: Fill knowledge gaps (only where needed)

The material may reference a fact, a version number, a command's behavior, or a term without enough detail to be clear. Fill it using general knowledge or a web search.

- Only fill gaps the post actually needs. Do not add background the author never wanted.
- Stay scoped. A post is often about one small piece of a bigger topic. Fill only what that piece needs. Do not write a general overview of the whole subject or search for everything related to it.
- Keep any filled-in detail short and factual. Do not turn the author's point into a lecture.
- Never change the author's opinions, conclusions, experiences, or claims to match what you found. If the author's claim conflicts with a fact, keep the author's claim and flag the conflict in the reply.
- Do not invent personal experiences, results, credentials, or feelings. The author's experiences come only from the material.

### Step 4: Draft, keeping the author's own words

Follow the approved outline. Apply these rules:

- **Layout and breathing room:** Avoid machine-like walls of text and dense, back-to-back paragraph dumps. Keep paragraphs short and punchy (1 to 3 sentences maximum), separated by clear blank lines so the post can breathe.
- **Visual anchors and structural variety:** Break up text logically using:
  - Horizontal rules (`---`) between major conceptual shifts.
  - Blockquotes (`> **Core insight**`) to anchor central realizations, principles, or mental models.
  - Markdown tables for direct side-by-side technical trade-offs (e.g. speed, ratio, memory, windows) instead of dense explanatory paragraphs.
  - Concise bullet items with bold lead-ins for structural lists.
- Keep the author's sentences wherever they work. Fix grammar, spelling, and punctuation. Reorder paragraphs and join fragments so the flow makes sense.
- Only reword a sentence when it is broken, unclear, or self-contradictory. Otherwise leave it alone.
- Keep the author's personality: jokes, asides, bluntness, the way they end a thought. Remove only clutter: repetition, dead ends, filler words from transcripts, duplicated notes, and chat chatter that does not belong in a post.
- Write in first person as the author. Frame the narrative through personal journey and discovery: ground it in how I encountered the problem, got confused or overwhelmed by it ("For the longest time I wondered...", "I used to think X until I dug in and realized Y..."), and how I finally connected the dots. Don't lecture or recite facts like a textbook.
- **Short, purposeful analogies:** When a technical concept is counter-intuitive or abstract, use a short, grounded real-world analogy to make it instantly click. Don't go overboard or stack metaphors—limit it to one or two crisp analogies in the entire post where they genuinely clarify mechanics, never decorative ones.
- Do not copy, quote, or cite external sources in the post text. Facts from gap-filling are restated in the author's voice. Only images get credits.
- Keep code, commands, file paths, error messages, and numbers exactly as written.
- Use headers only where the content has distinct sections. Use bullet lists only where the material is already a list.
- If a part of the material is genuinely unclear and a safe gap-fill isn't possible, keep it as close to the original as possible and flag it in the reply. Do not guess.

### Step 4b: Visuals (only when they earn their place)

Add a diagram or image only when it makes the point clearer than text. Skip anything decorative. Most posts need zero to two.

- **Mermaid diagram** for flows, sequences, and architecture. Write it inline in a ```mermaid code block. No upload needed. Keep diagrams well-proportioned: do not cram two distinct systems or models into a single wide diagram that gets stretched or squished horizontally. Split them into separate, focused diagrams (favoring `flowchart TD` or compact `flowchart LR`).
- **Generated image** for a concept a diagram can't show. Save the file at `generated/images/<blog-slug>/<descriptive-name>.png`. Upload it with:

  ```
  img up generated/images/<blog-slug>/<descriptive-name>.png -f blogs/<blog-slug> --no-copy
  ```

  The output includes a line like `CDN URL  https://ik.imagekit.io/...`. Take that URL and use it in the Markdown as `![alt text](https://ik.imagekit.io/...)`. Never put local paths in the post.
- **Resizing or other transforms** use ImageKit query parameters on the CDN URL, e.g., `https://ik.imagekit.io/.../request-flow.png?tr=w-800`. No new file is needed.
- **Online image.** Use its source URL and add a credit line directly under it (e.g., "Image: Wikimedia Commons").

Use a sub-agent for image generation and upload when one is available, so the writing isn't held up. Otherwise do it directly.

Alt text describes what the image shows. Upload each image once and reuse its URL if it appears again.

### Step 5: Voice pass

Match this profile:

- Short, crisp sentences by default. Go longer only when a technical point needs it.
- A direct, catchy opening. No throat-clearing, no "In today's world."
- Casual peer-to-peer tone, written for another developer. Not a beginner tutorial, not a guru lecture.
- Confident without posturing. Not an expert showing off, not a student apologizing. The target is a plain "I got overwhelmed by this, dug into it, finally figured out how it connects, and wrote it down" register.
- Contractions always: don't, it's, can't.
- Mild profanity only if the material already uses it or the mood clearly calls for it. Never more than the author would use.
- Light wit is fine. Self-deprecating jokes and dry humor are fine. Humor should come from the content.
- Ending: a short, wry line that fits the post. No calls to action, no "Let me know below," no "Thoughts?"

### Step 6: Anti-AI pass

Remove these patterns from the output text. Leave any passage that already reads as human untouched.

- **Dashes:** replace em dashes with commas, periods, or parentheses. Keep hyphens inside compound words and en dashes in ranges. Do not place two dash-joined asides in one paragraph.
- **Contrast framing:** replace "not X, but Y" or "It's not about X, it's about Y" with a direct statement.
- **Rhetorical Q&A hooks:** cut self-answering rhetorical questions ("Why does this matter? Because...", "What does that mean? It means..."). State the point directly.
- **Formulaic colon setups:** cut artificial lead-in declarations ("The reason is simple:", "One thing is clear:").
- **Triplets:** do not group ideas in threes unless the material does.
- **Stock transitions:** cut "moreover," "furthermore," "additionally," "in addition," "notably," "crucially," "it is worth noting that," "at its core."
- **Overused words:** avoid "delve," "tapestry," "landscape," "realm," "testament," "beacon," "interplay," "foster," "harness," "streamline," "demystify," "multifaceted," "nuanced," "seamless," "robust," "leverage," "game-changer," "crucial," "vital."
- **Setup lines & grandiose endings:** no "In conclusion," "Let's dive in," "Let's break it down," "In this post, we'll explore," and no sweeping cinematic wrap-ups ("stands as a testament," "the journey has only just begun"). End on a grounded, wry line.
- **Hedging:** remove diplomat-style both-sides caveats ("While some may argue..."). Let the author's real stance stand firm.
- **Metaphor stacks:** cut clichéd metaphors. Allow only one or two short, functional analogies where genuinely needed to demystify complex mechanics, and cut decorative or excessive ones.
- **Formatting tells:** no bold-label bullets ("**Key takeaway:**"). No bold for emphasis unless the author used it. No emoji, no hashtags.
- **Burstiness & paragraph dumps:** break up clustered, monotonous blocks of text. Vary the rhythm by mixing short punchy lines with longer technical thoughts. Never allow dense, unbroken paragraph walls.
- **Quotes:** straight quotes as typed in the material.

### Step 7: Final check

Before saving, confirm:

- It is still first person and still sounds like the author.
- Most of the author's own sentences are still there.
- The layout has breathing room: short punchy paragraphs, clean dividers, blockquote anchors, and no dense text dumps.
- Mermaid diagrams are clean, well-proportioned, and not squished.
- No em dashes, stock transitions, or setup lines remain.
- Code, commands, and numbers are exact.
- No claim was added that the material doesn't support, except safe, flagged gap-fills.
- The file contains only the blog body, with no title, excerpt, or tags inside it.
- Every image link is a live CDN URL or a credited source URL. No local paths.
- The post ends on a wry line, not a call to action.

Fix anything that fails, then save the file and write the metadata in the reply.

## Reply format

Keep the reply short:

```
**Post Title:** ...
**Excerpt / SEO Description:** ...
**Tags:** tag-one, tag-two, tag-three
```

Then add one or two sentences on anything flagged, such as an unclear section, a conflicting claim, or a gap filled from outside the material. Then give the path of the blog file.

## What not to do

- Do not change the meaning, conclusions, or technical content.
- Do not flatten the author's personality into neutral prose. Keep the jokes, opinions, and bluntness.
- Do not restructure into a generic template ("Introduction, Body, Conclusion").
- Do not add a summary, TL;DR, or key takeaways section unless the author wrote one.
- Do not pad. Cut anything that adds only words.
- Do not write in a corporate, marketing, or LinkedIn register.
- Do not put the title, excerpt, or tags inside the Markdown file.
- Do not write output into `raw/`.
