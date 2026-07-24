# Contribution #7665:Poor Web layout

**Contribution Number:**  1 

**Student:** Sanjna Tailor

**Issue:** (https://github.com/transmission/transmission/issues/7665)

**Status:** Phase 4 Complete

---

## Why I Chose This Issue

This issue interests me because it will test my understanding of C++ pipelines. I already have experience in C++, and I can extend it to other libraries within the language. This issue identifies UI problems within the web interface, leading to a decreased user experience. As I am interested in UI/UX, this issue is something I would have interest working on.

---

## Understanding the Issue

### Problem Description

Text overflows designated DOM containers on the Web GUI

### Expected Behavior

This text should be styled such that they stay within their visual parent components

### Current Behavior

The problem described is the current behavior

### Affected Components

The issue is in the web folder of the Transmission repository, within the transmission-app component

---

## Reproduction Process

### Environment Setup

I installed dependencies as listed in the ReadMe.md and Building-Transmission.md files. I had some issues with downloading dependencies, however resoloved them with claude

### Steps to Reproduce
1. Run a transmission-daemon to have a live RPC endpoint, and start the web dev server (cd web && npm install && npm run dev)
2. Open the Web client in chrome browser and change window size to 14" macOS
3. Narrow the torrent-list column, triggering the bug
4. Render long strings in the metric fields
   a. Add a few large torrents, actively downloading at high speed with many peers and a long ETA
   b. temporarily edit the row text to a long string to force the condition. 

### Reproduction Evidence

- **Commit showing reproduction:** [reproduction commit](https://github.com/SanjnaT41756/transmission/commit/ee9fd14aa28d6f623069562fb8273be57bf57af4)
- **Screenshots/logs:** [If applicable]
- **My findings:** I found that this issue is persistent across window/screen sizes

---

## Solution Approach

### Analysis

The overflow comes from two layers. The rendering layer produces variable-length text. web/src/formatter.js holds the humanizing functions (size, speedBps/speed, percentString, timeInterval, mem, ratioString) that return strings. Their lengths vary widely with no fixed-width or padding contract. web/src/torrent-row.js (the TorrentRendererFull/TorrentRendererCompact classes) writes these strings into the row's child elements via textContent on every refresh, so the metric line's width fluctuates with the data.
The layout layer is why that text escapes its box. In web/assets/css/transmission-app.scss, the metric containers (.torrent-peer-details, .torrent-progress-details, and the surrounding .torrent flex row) lack the standard "shrink-and-clip" styling.

### Proposed Solution

CSS: make the metric containers behave as properly constrained flex items. Add min-width: 0 to the flex children holding dynamic text, apply overflow: hidden; text-overflow: ellipsis; white-space: nowrap to the metric text so over-long values clip with an ellipsis, right-align the numeric block within its parent, add a small right-padding buffer, and set font-variant-numeric: tabular-nums (or a monospace family) on metric text so digit widths stay stable and the line doesn't expand as numbers update.
JS: normalize the formatter functions to a consistent width, so the produced strings stop varying in length.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:**  The web client's dynamic metric fields (size, speed, ETA, peer counts) overflow their element boundaries (text spills outside the row) most visibly on narrow laptop viewports in Chromium. The fields are plain text nodes whose length varies with the data, sitting in flex containers that neither shrink nor clip.

**Match:**  apply that same ellipsis/min-width: 0 treatment to the metric containers as is present in other components in transmission-app.scss

**Plan:** [Step-by-step implementation plan]
1. In web/assets/css/transmission-app.scss, add min-width: 0 to the flex children of .torrent that hold dynamic text, and overflow: hidden; text-overflow: ellipsis; white-space: nowrap to .torrent-peer-details / .torrent-progress-details; right-align the metric block, add a right-padding buffer, and apply font-variant-numeric: tabular-nums to metric text. Mirror the existing .torrent-name truncation pattern.
2. In web/src/formatter.js, normalize the humanizing functions to a consistent precision and unit-spacing contract so string lengths stabilize.
4. Rebuild the served artifacts (compile SCSS with rsass, bundle with esbuild to web/public_html/transmission-app.js) and confirm the change appears in the bundle

**Implement:** [[Link to fix branch]](https://github.com/SanjnaT41756/transmission/tree/7665-fix-text-layout)

**Review:** This fix follows the stylistic contribution guidelines: KISS, running ./code_style.sh to ensure uniform formatting, and using commonly-used tools

**Evaluate:** As there is no formal testing code for this visual change, I will be manually testing.

---

## Testing Strategy

### Unit Tests

Not applicable. The change modifies only CSS grid sizing (min-width, overflow, text-overflow). jsdom performs no layout, so unit tests cannot observe overflow or clipping. No JavaScript logic (formatter.js, torrent-row.js) was changed, so existing unit coverage remains valid and unaffected.

### Integration Tests

A browser-rendered test that reproduces the overflow condition and asserts it no longer occurs, exercising the fix directly. placed at web/tests/torrent-row-overflow.spec.js to **match existing tests**; build the CSS first (rsass assets/css/transmission-app.scss > assets/css/transmission-app.css) since the test loads the compiled output. **Existing test suite still passes**.

### Manual Testing

Environment: built via npm run dev (esbuild watch) against a running transmission-daemon, viewed at the served web UI in a Chromium browser on Windows; viewport width varied via window resize and DevTools device toolbar.

 - Reproduced the bug on the unpatched build
 - Confirmed the fix in full (non-compact) view with same width and torrents: metric text now stays within the row, wrapping inside its column instead of overflowing
 - Confirmed the fix in compact view; Checked long values in those fields using the DevTools console

---

## Implementation Notes

### Week [3] Progress

I worked on updating the CSS files of the web interface, getting rid of duplicates and styles that were being overwritten. Then I worked on the fix to the SCSS grid.

### Week [4] Progress

I continued on working on the visual fix while keeping in mind the dependencies and style guides for writing code. Only one file actually needed to be changed (rather than 2 that was outlined in the steps), where I had to modify an existing text styling rule

### Code Changes

- **Files modified:** transmission/web/assets/css/transmission-app.scss
- **Key commits:** [key commit](https://github.com/SanjnaT41756/transmission/commit/9ba9f7df499e0a6e673881f9c3f223a5508629c5)
- **Approach decisions:** This was the easiest way to fix this issue without touching other files unnecessarily. This follows the "KISS" consideration mentioned in the contribution guide for the repository: "Keep It Short and Simple".

---

## Pull Request

**PR Link:** [[GitHub PR](https://github.com/transmission/transmission/pull/8946)

**PR Description:** Notes: Fixes the overflow of text in the web client

What does this PR do?
fixes torrent-progress.scss to avoid text overflow.

The .torrent grid's peer/progress detail elements kept overflowing: visible, giving them a grid automatic-minimum-size equal to their min-content width. Long speed/size/ETA/peer strings therefore forced the middle 'auto' column (and the whole grid) wider than the row, so text spilled past the element boundary. Add min-width:0 + overflow: hidden + text-overflow: ellipsis, matching the existing .torrent-name and .torrent-labels rules, so the items shrink and clip instead.


**Maintainer Feedback: Awaiting Review**
- [June 25, 2026]: pr submitted, maintainer tagged, waiting for review
- [July 9, 2026]: Still waiting for review


**Status:** [**Awaiting review** / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

The main lesson I learned is that CSS Grid and Flex items don't shrink by default, so a long unbreakable string forces its track wider than the container and spills over. Setting overflow to anything other than visible resets that automatic minimum to zero, which is what lets an item shrink and then clip or ellipsize. Its good to understand this general layout principle that is applicable in any web development environment.
I also learned to read a bug against the code's own conventions. The fix was essentially copying the overflow: hidden; text-overflow: ellipsis code that .torrent-name and .torrent-labels rules already used. Beyond that, I got hands-on with the project's build pipeline — SCSS compiled with rsass, the app bundled with esbuild, and the daemon serving whatever TRANSMISSION_WEB_HOME points at.

### Challenges Overcome

The biggest challenge was finding the source of the bug within this big project. My first hypothesis assumed a flexbox layout and the standard flex min-width: 0 fix. Once I read the actual transmission-app.scss, it turned out to be a CSS Grid. The other challenge was with my development environment. On Windows PowerShell the npm run dev script failed because DEV=true node is Unix env-var syntax, which I worked around with $env:DEV="true".

### What I'd Do Differently Next Time

I would try to get more intuition from the code maintainers next time I work on an open source issue, because some code conventions are unique to the project. With this I can figure out more viable testing measures for checking the effectiveness of the code changes.

---

## Resources Used

Transmission web client build/run docs — https://github.com/transmission/transmission/blob/main/web/README.md (the rsass + esbuild build steps, npm run dev, and running at localhost:9000)
