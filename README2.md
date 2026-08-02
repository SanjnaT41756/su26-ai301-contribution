# Contribution [#2]: 🐛 [Bug]: QrCode size, iconSize, color 属性未响应式 #3078


**Contribution Number:** 2  
**Student:** Sanjna Tailor

**Issue:** [GitHub issue link ](https://github.com/opentiny/tiny-vue/issues/3078)

**Status:** [ Phase I / **Phase II** / Phase III / Phase IV] **[In Progress** / Complete]

---

## Why I Chose This Issue

I chose this issue because it lets me dive deeper in framework and component fundamentals. It's a Vue 3 reactivity bug, where the QrCode component's size, iconSize, and color props don't propagate to the rendered output after the initial mount. 
It's a problem with limited scope, which makes it approachable while still requiring me to genuinely understand how Vue tracks reactive props and re-runs rendering logic. 
It also builds naturally on my last contribution, a frontend fix in a component-driven project, but pushes me from CSS layout into the JavaScript/framework layer and into a large, real-world monorepo component library (OpenTiny/TinyVue).

What I hope to get out of it is a deeper grasp of the Vue 3 reactivity model. I'm also looking to practice the investigative side of open source: since the report has no reproduction or details, I'll need to build a minimal repro, trace the component's render path to find exactly where the props are consumed, and confirm the root cause before proposing a fix. Contributing to an established component library is a good way to learn a mature project's conventions for testing and reviewing UI components, which is a skill I want to carry into future work.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup
- Cloned `opentiny/tiny-vue`, checked out `dev` branch.
- Repo uses pnpm workspaces, so `pnpm i` is required 
- Ran `pnpm dev` to start the Vue 3 PC playground at http://127.0.0.1:7130/
- No special environment issues encountered beyond needing the correct package manager (pnpm) and Node version specified in `package.json`.

### Steps to Reproduce
1. Add a test page/component that renders `<tiny-qr-code>` bound to reactive `size`, `icon-size`, and `color` refs, e.g.:
```vue
   <template>
     <tiny-qr-code :value="value" :size="size" :icon-size="iconSize" :color="color" />
     <button @click="size += 20">Increase size</button>
     <button @click="color = '#ff0000'">Change color</button>
   </template>
   <script setup>
   import { ref } from 'vue'
   import { QrCode as TinyQrCode } from '@opentiny/vue'
   const value = ref('https://opentiny.design/tiny-vue')
   const size = ref(150)
   const iconSize = ref(30)
   const color = ref('#000000')
   </script>
```
2. Click the buttons to change `size`, `iconSize`, and `color` after the QR code has initially rendered.
3. Observed result: the QR code does not visually update — it keeps the size/icon size/color from the first render, even though the underlying refs have changed. Re-mounting the component (e.g. via `v-if` toggle or `:key` change) does pick up the new values, confirming the props aren't wired into a reactive watcher/computed inside the component and are likely only read once on initial canvas draw.

### Reproduction Evidence
- **Commit showing reproduction:** (https://github.com/opentiny/tiny-vue/commit/a543e78c3ecd3d73c1ddbc3620e02d02bf1bcd19)
- **Screenshots/logs:** N/A
- **My findings:** The props are likely consumed only inside a one-time draw/init function (e.g. an `onMounted` or initial `watch(..., { immediate: true })` without the prop in its dependency, or a canvas-draw function not re-invoked on prop change) rather than a reactive `watch`/`watchEffect` that re-triggers the canvas redraw whenever `size`, `iconSize`, or `color` change.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
