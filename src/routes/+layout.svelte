<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	import { browser } from '$app/environment';

	let { children } = $props();

	type Theme = 'light' | 'dark';

	function getInitialTheme(): Theme {
		if (!browser) return 'light';
		const stored = localStorage.getItem('theme');
		if (stored === 'light' || stored === 'dark') return stored;
		return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
	}

	let theme = $state<Theme>(getInitialTheme());

	$effect(() => {
		if (!browser) return;
		document.documentElement.classList.toggle('dark', theme === 'dark');
		localStorage.setItem('theme', theme);
	});

	function toggleTheme() {
		theme = theme === 'dark' ? 'light' : 'dark';
	}
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>

<button
	type="button"
	onclick={toggleTheme}
	aria-label={theme === 'dark' ? 'Switch to light mode' : 'Switch to dark mode'}
	aria-pressed={theme === 'dark'}
	class="fixed top-4 right-4 z-50 inline-flex h-8 w-16 shrink-0 items-center rounded-full bg-slate-200 transition-colors duration-300 dark:bg-slate-700"
>
	<span
		class="absolute left-1 flex h-6 w-6 items-center justify-center rounded-full bg-white text-slate-700 shadow-md transition-transform duration-300 ease-in-out dark:translate-x-8 dark:bg-slate-900 dark:text-slate-100"
	>
		{#if theme === 'dark'}
			<svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24">
				<path
					fill="none"
					stroke="currentColor"
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M12 21a9 9 0 0 1-.5-17.986V3c-.354.966-.5 1.911-.5 3a9 9 0 0 0 9 9c.239 0 .254.018.488 0A9 9 0 0 1 12 21"
				/>
			</svg>
		{:else}
			<svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24">
				<path
					fill="none"
					stroke="currentColor"
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M12 5V3m0 18v-2M7.05 7.05L5.636 5.636m12.728 12.728L16.95 16.95M5 12H3m18 0h-2M7.05 16.95l-1.414 1.414M18.364 5.636L16.95 7.05M16 12a4 4 0 1 1-8 0a4 4 0 0 1 8 0"
				/>
			</svg>
		{/if}
	</span>
</button>

{@render children()}
