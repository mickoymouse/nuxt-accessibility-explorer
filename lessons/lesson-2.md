# Lesson 2 — Temporal Accessibility: Transitions and Focus Lifecycle

## Goal

Learn how **time, transitions, and perception** affect accessibility, and manage focus correctly across the modal lifecycle.

This lesson focuses on **when** focus should move, not just **where**.

---

## Background

In Lesson 1, focus was moved immediately after state changes using `nextTick`.  
While this ensures DOM readiness, it does not guarantee **perceptual readiness** when transitions are present.

Transitions introduce a delay between:

- State change (`isOpen = true`)
- Visual readiness (modal fully visible)
- Meaningful interaction (user can orient themselves)

Lesson 2 addresses this gap.

---

## DOM Readiness vs Perceptual Readiness

- `nextTick` ensures Vue has updated the DOM.
- It does **not** ensure the UI is visually ready or stable.
- Moving focus too early can:
  - Focus invisible or partially rendered elements
  - Trigger premature screen reader announcements
  - Disorient keyboard users

> Accessibility should follow **perception**, not just implementation state.

---

## Focus Placement Using Transition Hooks

### Problem

Moving focus immediately after state change is too early when transitions exist.

### Solution

Move focus **after the enter transition completes** using Vue's `@after-enter` hook.

```vue
<Transition
	name="modal-fade"
	@after-enter="focusModal"
	@after-leave="restoreFocus"
>
  <div
    v-if="isOpen"
    ref="modalRef"
    tabindex="-1"
    role="dialog"
    aria-modal="true"
    aria-labelledby="modalTitle"
  >
    <h2 id="modalTitle">Modal title</h2>
    <button @click="closeModal">Close</button>
    <button>Action</button>
  </div>
</Transition>
```

### Bonus

- Use `@after-leave` to restore focus to the previously focused element when the modal closes.

```ts
const previouslyFocusedElement = ref<HTMLElement | null>(null);
const openModal = () => {
	previouslyFocusedElement.value = document.activeElement as HTMLElement;
	isOpen.value = true;
};
const closeModal = () => {
	isOpen.value = false;
};
const restoreFocus = () => {
	previouslyFocusedElement.value?.focus();
};
```
