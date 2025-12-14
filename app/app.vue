<script lang="ts" setup>
const isOpen = ref(false);
const activeElement = ref({
	tag: "",
	text: "",
});

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

onMounted(() => {
	updateActiveElement();
	window.addEventListener("focusin", updateActiveElement);
});

onBeforeUnmount(() => {
	window.removeEventListener("focusin", updateActiveElement);
});

const modalRef = ref<HTMLElement | null>(null);
const mainContent = ref<HTMLElement | null>(null);

watch(isOpen, async (newVal) => {
	if (newVal && modalRef.value) {
		await nextTick();
		modalRef.value.focus();
		window.addEventListener("keydown", handleKeydown);

		if (!mainContent.value) return;
		mainContent.value.setAttribute("aria-hidden", "true");
		mainContent.value.inert = true;
	} else {
		window.removeEventListener("keydown", handleKeydown);
		if (!mainContent.value) return;
		mainContent.value.removeAttribute("aria-hidden");
		mainContent.value.inert = false;
	}
});

const handleKeydown = (event: KeyboardEvent) => {
	if (event.key === "Escape" && isOpen.value) {
		isOpen.value = false;
	}

	trapFocus(event);
};

const trapFocus = (event: KeyboardEvent) => {
	if (event.key !== "Tab" || !modalRef.value) return;

	const focusableEls = Array.from(
		modalRef.value.querySelectorAll<HTMLElement>(
			'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
		)
	);

	if (focusableEls.length === 0) return;

	const firstEl = focusableEls[0];
	const lastEl = focusableEls[focusableEls.length - 1];

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
		<NuxtRouteAnnouncer />
		<div ref="mainContent">
			<button @click="isOpen = !isOpen">Open Modal</button>
			<div style="margin-top: 2rem; font-family: monospace">
				<strong>Active element:</strong>
				<div>Tag: {{ activeElement.tag }}</div>
				<div>Text: {{ activeElement.text }}</div>
			</div>
		</div>
		<div
			v-show="isOpen"
			ref="modalRef"
			tabindex="-1"
			role="dialog"
			aria-modal="true"
			aria-labelledby="modalTitle"
		>
			<h2 id="modalTitle">Modal Dialog</h2>
			<button @click="isOpen = false">Close</button>
			<button>Option 1</button>
			<button>Option 2</button>
		</div>
	</div>
	<div aria-live="assertive" class="sr-only">
		Modal opened: {{ isOpen ? "Yes" : "No" }}
	</div>
</template>
