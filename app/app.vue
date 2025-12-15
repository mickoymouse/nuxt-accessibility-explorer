<script lang="ts" setup>
// Modal state control
const isOpen = ref(false);

// Track the currently focused element for debugging/visualization purposes
const activeElement = ref({
	tag: "",
	text: "",
});
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
	previouslyFocusedElement.value = null;
};

// Updates the active element display whenever focus changes in the document
const updateActiveElement = () => {
	const el = document.activeElement as HTMLElement;
	if (el) {
		activeElement.value = {
			tag: el.tagName,
			text: el.innerText || el.getAttribute("aria-label") || "",
		};
	} else {
		activeElement.value = {
			tag: "",
			text: "",
		};
	}
};

// Set up focus tracking on mount
onMounted(() => {
	updateActiveElement();
	window.addEventListener("focusin", updateActiveElement);
});

// Clean up event listener on unmount
onBeforeUnmount(() => {
	window.removeEventListener("focusin", updateActiveElement);
});

// Refs for managing focus and inert behavior
const modalRef = ref<HTMLElement | null>(null);
const mainContent = ref<HTMLElement | null>(null);

// Watch modal state to manage focus and background content accessibility
watch(isOpen, async (newVal) => {
	if (newVal && modalRef.value) {
		// Wait for DOM update before manipulating focus
		// await nextTick();
		// Move focus into the modal automatically when opened
		// modalRef.value.focus();
		// Enable keyboard event handling (Escape key and focus trap)
		window.addEventListener("keydown", handleKeydown);

		// Make background content inert: hidden from screen readers and non-interactive
		if (!mainContent.value) return;
		mainContent.value.setAttribute("aria-hidden", "true");
		mainContent.value.inert = true;
	} else {
		// Clean up keyboard event listener when modal closes
		window.removeEventListener("keydown", handleKeydown);
		// Restore background content accessibility
		if (!mainContent.value) return;
		mainContent.value.removeAttribute("aria-hidden");
		mainContent.value.inert = false;
	}
});

const focusModal = () => {
	if (modalRef.value) {
		modalRef.value.focus();
	}
};

// Handle keyboard interactions while modal is open
const handleKeydown = (event: KeyboardEvent) => {
	// Close modal with Escape key
	if (event.key === "Escape" && isOpen.value) {
		closeModal();
	}

	// Keep focus trapped within modal
	trapFocus(event);
};

// Trap focus within the modal: Tab cycles only through modal's focusable elements
const trapFocus = (event: KeyboardEvent) => {
	if (event.key !== "Tab" || !modalRef.value) return;

	// Find all focusable elements within the modal
	const focusableEls = Array.from(
		modalRef.value.querySelectorAll<HTMLElement>(
			'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
		)
	);

	if (focusableEls.length === 0) return;

	const firstEl = focusableEls[0];
	const lastEl = focusableEls[focusableEls.length - 1];

	// Shift+Tab on first element: cycle to last element
	if (event.shiftKey && document.activeElement === firstEl) {
		event.preventDefault();
		lastEl?.focus();
	} else if (!event.shiftKey && document.activeElement === lastEl) {
		event.preventDefault();
		firstEl?.focus();
	}
};
</script>

<template>
	<div>
		<!-- Nuxt's built-in route change announcer for screen readers -->
		<NuxtRouteAnnouncer />
		<div ref="mainContent">
			<button @click="openModal">Open Modal</button>

			<!-- Debug display showing currently focused element -->
			<div style="margin-top: 2rem; font-family: monospace">
				<strong>Active element:</strong>
				<div>Tag: {{ activeElement.tag }}</div>
				<div>Text: {{ activeElement.text }}</div>
			</div>
		</div>

		<!-- 
			Accessible Modal Dialog:
			- v-show used instead of v-if for easier focus management
			- tabindex="-1" allows programmatic focus but removes from tab order
			- role="dialog" identifies this as a dialog for assistive tech
			- aria-modal="true" indicates this is a modal interaction
			- aria-labelledby connects the modal to its title for screen readers
		-->
		<Transition
			name="modal-fade"
			@after-enter="focusModal"
			@after-leave="restoreFocus"
		>
			<div
				v-show="isOpen"
				ref="modalRef"
				tabindex="-1"
				role="dialog"
				aria-modal="true"
				aria-labelledby="modalTitle"
			>
				<h2 id="modalTitle">Modal Dialog</h2>
				<!-- Close button provides visible close mechanism (Escape also works) -->
				<button @click="closeModal">Close</button>
				<button>Option 1</button>
				<button>Option 2</button>
			</div>
		</Transition>
	</div>
	<div aria-live="assertive" class="sr-only">
		Modal opened: {{ isOpen ? "Yes" : "No" }}
	</div>
</template>
<style scoped>
.modal-fade-enter-active,
.modal-fade-leave-active {
	transition: opacity 0.2s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
	opacity: 0;
}
</style>
