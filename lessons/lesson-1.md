# Lesson 1 — Accessible Modal in Nuxt 4

## Summary of Learnings

- Focus moves automatically into modal when opened.
- Focus is trapped inside modal; Tab cycles only within.
- Close mechanism: button + Escape key.
- Background content made inert (aria-hidden + inert).
- Semantic HTML elements matter for keyboard activation.
- ARIA roles and labels provide screen reader context.
- Observing behavior with keyboard + screen reader is critical.

## Notes

- v-if vs v-show: v-show easier for focus experiments; v-if requires nextTick or child component.
- VoiceOver announcements order is structural-first.
- Next steps: transitions, dynamic content, reusable modal component.
