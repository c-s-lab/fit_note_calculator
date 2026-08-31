<script lang="ts">
	type FitNoteStatus = 'NO_FIT_NOTE' | 'FIT_NOTE';

	interface FitNoteResult {
		days: number;
		status: FitNoteStatus;
	}

	let startDate = $state('');
	let endDate = $state('');
	let submitted = $state(false);

	// Parse a yyyy-mm-dd date input as UTC midnight so DST transitions
	// in the local timezone can never shift the calendar-day count.
	function parseDateUTC(value: string): Date {
		const [year, month, day] = value.split('-').map(Number);
		return new Date(Date.UTC(year, month - 1, day));
	}

	function inclusiveCalendarDayDifference(start: string, end: string): number {
		const startUTC = parseDateUTC(start);
		const endUTC = parseDateUTC(end);
		const msPerDay = 24 * 60 * 60 * 1000;
		const diffDays = Math.round((endUTC.getTime() - startUTC.getTime()) / msPerDay);
		return diffDays + 1;
	}

	const validationError = $derived.by((): string | null => {
		if (!submitted) return null;
		if (!startDate) return 'The first day of illness is required.';
		if (!endDate) return 'The last day of illness is required.';
		if (endDate < startDate) {
			return 'The last day of illness cannot be before the first day of illness.';
		}
		return null;
	});

	const result = $derived.by((): FitNoteResult | null => {
		if (!submitted || validationError || !startDate || !endDate) return null;

		const days = inclusiveCalendarDayDifference(startDate, endDate);
		const status: FitNoteStatus = days <= 7 ? 'NO_FIT_NOTE' : 'FIT_NOTE';

		return { days, status };
	});

	function handleSubmit(event: SubmitEvent) {
		event.preventDefault();
		submitted = true;
	}

	function reset() {
		startDate = '';
		endDate = '';
		submitted = false;
	}
</script>

<svelte:head>
	<title>Do I need a Fit Note? (UK)</title>
</svelte:head>

<main class="mx-auto max-w-xl px-5 py-12 sm:py-16">
	<h1 class="text-3xl font-semibold tracking-tight text-slate-900 dark:text-slate-100">
		Do I need a fit note?
	</h1>
	<p class="mt-3 text-slate-600 dark:text-slate-400">
		Answer two questions about your sickness absence to find out whether you should get a fit note,
		based on current
		<a
			href="https://www.gov.uk/government/publications/the-fit-note-a-guide-for-patients-and-employees/the-fit-note-guidance-for-patients-and-employees"
			target="_blank"
			rel="noopener noreferrer"
			class="text-emerald-700 underline hover:text-emerald-800 dark:text-emerald-400 dark:hover:text-emerald-300"
		>
			GOV.UK
		</a>,
		<a
			href="https://www.nhs.uk/nhs-services/gps/getting-a-fit-note/"
			target="_blank"
			rel="noopener noreferrer"
			class="text-emerald-700 underline hover:text-emerald-800 dark:text-emerald-400 dark:hover:text-emerald-300"
		>
			NHS
		</a>
		and
		<a
			href="https://www.acas.org.uk/fit-notes-and-proof-of-sickness"
			target="_blank"
			rel="noopener noreferrer"
			class="text-emerald-700 underline hover:text-emerald-800 dark:text-emerald-400 dark:hover:text-emerald-300"
		>
			Acas
		</a>
		guidance.
	</p>

	<form onsubmit={handleSubmit} class="mt-8 space-y-6" novalidate>
		<div>
			<label for="startDate" class="block text-sm font-medium text-slate-900 dark:text-slate-100">
				What was the first day you were unable to work because you were ill?
			</label>
			<input
				id="startDate"
				type="date"
				bind:value={startDate}
				class="mt-2 block w-full max-w-xs rounded-md border-slate-300 shadow-sm focus:border-emerald-600 focus:ring-emerald-600 sm:text-sm dark:border-slate-600 dark:bg-slate-800 dark:text-slate-100 dark:scheme-dark"
			/>
		</div>

		<div>
			<label for="endDate" class="block text-sm font-medium text-slate-900 dark:text-slate-100">
				What was the last day you were unable to work because you were ill?
			</label>
			<input
				id="endDate"
				type="date"
				bind:value={endDate}
				class="mt-2 block w-full max-w-xs rounded-md border-slate-300 shadow-sm focus:border-emerald-600 focus:ring-emerald-600 sm:text-sm dark:border-slate-600 dark:bg-slate-800 dark:text-slate-100 dark:scheme-dark"
			/>
			<p class="mt-2 text-sm text-slate-500 dark:text-slate-400">
				This is the last day you were ill, not the day you returned to work.
			</p>
		</div>

		<div class="flex items-center gap-3 pt-2">
			<button
				type="submit"
				class="rounded-md border-2 border-emerald-700 bg-emerald-700 px-5 py-2 text-sm font-semibold text-white hover:border-emerald-800 hover:bg-emerald-800 dark:border-emerald-600 dark:bg-emerald-600 dark:hover:border-emerald-500 dark:hover:bg-emerald-500"
			>
				Check
			</button>
			{#if submitted}
				<button
					type="button"
					onclick={reset}
					class="rounded-md border-2 border-slate-300 px-5 py-2 text-sm font-semibold text-slate-700 hover:border-slate-400 dark:border-slate-600 dark:text-slate-300 dark:hover:border-slate-500"
				>
					Start again
				</button>
			{/if}
		</div>
	</form>

	{#if validationError}
		<div
			class="mt-8 rounded-md border-l-4 border-red-600 bg-red-50 px-5 py-4 dark:border-red-500 dark:bg-red-950"
			role="alert"
		>
			<p class="font-medium text-red-700 dark:text-red-400">{validationError}</p>
		</div>
	{:else if result}
		{#if result.status === 'NO_FIT_NOTE'}
			<div
				class="mt-8 rounded-md border-l-4 border-emerald-700 bg-emerald-50 px-5 py-5 dark:border-emerald-500 dark:bg-emerald-950"
			>
				<h2 class="text-lg font-semibold text-slate-900 dark:text-slate-100">
					You do not need a fit note based on the length of your sickness absence
				</h2>
				<p class="mt-2 text-slate-700 dark:text-slate-300">
					Your sickness absence lasted
					<strong>{result.days} calendar day{result.days === 1 ? '' : 's'}</strong>.
				</p>
				<p class="mt-2 text-slate-700 dark:text-slate-300">
					If you are off sick for 7 calendar days or fewer, you can normally self-certify your
					sickness.
				</p>
				<p class="mt-3 text-sm text-slate-600 dark:text-slate-400">
					Calendar days include weekends, bank holidays and days you do not normally work.
				</p>
			</div>
		{:else}
			<div
				class="mt-8 rounded-md border-l-4 border-red-600 bg-red-50 px-5 py-5 dark:border-red-500 dark:bg-red-950"
			>
				<h2 class="text-lg font-semibold text-slate-900 dark:text-slate-100">
					You should get a fit note
				</h2>
				<p class="mt-2 text-slate-700 dark:text-slate-300">
					Your sickness absence lasted <strong>{result.days} calendar days</strong>, which is more
					than 7 calendar days.
				</p>
				<p class="mt-2 text-slate-700 dark:text-slate-300">
					Based on the 7-calendar-day rule, you should ask a healthcare professional for a fit note.
				</p>
			</div>
		{/if}

		<p class="mt-6 text-sm text-slate-500 dark:text-slate-400">
			This calculator provides general information based on current GOV.UK, NHS and Acas guidance.
			It does not take account of your employer's individual sickness policy or every possible
			employment situation. Check with your employer if you are unsure.
		</p>

		<div class="mt-6 border-t border-slate-200 pt-6 dark:border-slate-700">
			<h3 class="text-sm font-medium text-slate-900 dark:text-slate-100">Sources</h3>
			<ul class="mt-2 space-y-1 text-sm text-slate-600 dark:text-slate-400">
				<li>
					<a
						href="https://www.gov.uk/government/publications/the-fit-note-a-guide-for-patients-and-employees/the-fit-note-guidance-for-patients-and-employees"
						target="_blank"
						rel="noopener noreferrer"
						class="text-emerald-700 underline hover:text-emerald-800 dark:text-emerald-400 dark:hover:text-emerald-300"
					>
						GOV.UK — The fit note: guidance for patients and employees
					</a>
				</li>
				<li>
					<a
						href="https://www.nhs.uk/nhs-services/gps/getting-a-fit-note/"
						target="_blank"
						rel="noopener noreferrer"
						class="text-emerald-700 underline hover:text-emerald-800 dark:text-emerald-400 dark:hover:text-emerald-300"
					>
						NHS — Getting a fit note
					</a>
				</li>
				<li>
					<a
						href="https://www.acas.org.uk/fit-notes-and-proof-of-sickness"
						target="_blank"
						rel="noopener noreferrer"
						class="text-emerald-700 underline hover:text-emerald-800 dark:text-emerald-400 dark:hover:text-emerald-300"
					>
						Acas — Fit notes and proof of sickness
					</a>
				</li>
			</ul>
		</div>
	{/if}
</main>
