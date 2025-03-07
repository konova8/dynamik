<script lang="ts">
	import type { PageData } from './$types';
	import { page } from '$app/stores';
	import { onMount } from 'svelte';
	import type { Degree, Teaching } from '$lib/teachings';
	import { getLoginUrl, getWhoAmI } from '$lib/upld';
	import ListTeaching from './ListTeaching.svelte';
	import type { TeachingsBatch } from './ListTeaching.svelte';

	export let data: PageData;
	let activeYears: string[] = [];

	let login:
		| Promise<{ error: string } | { username: string; name: string; avatarUrl: string }>
		| undefined;

	onMount(async () => {
		activeYears = (await data.streaming?.activeTeachings) ?? [];
		login = getWhoAmI(fetch);
	});

	function namesToTeachings(names: string[]): Teaching[] {
		return names.map(data.teachings.get, data.teachings).filter((x): x is Teaching => !!x);
	}

	function reorganizeTeachings(degree: Degree) {
		if (!degree.years) return { mandatory: [], electives: [] };
		const mandatory: TeachingsBatch[] = [];
		const electives: TeachingsBatch[] = [];

		for (const year of degree.years) {
			const m = year.teachings.mandatory;
			const e = year.teachings.electives;

			if (m) mandatory.push({ year: year.year, teachings: namesToTeachings(m) });
			if (e) electives.push({ year: year.year, teachings: namesToTeachings(e) });
		}

		return { mandatory, electives };
	}

	$: reorganizedTeachings = reorganizeTeachings(data.degree);
</script>

<svelte:head>
	<title>{data.degree?.name}</title>
	<!-- OG meta graph -->
	<meta property="og:title" content={data.degree?.name} />
	<meta
		name="url"
		property="og:url"
		content="https://risorse.students.cs.unibo.it/{data.degree?.name}"
	/>
	<meta name="description" property="og:description" content="Risorse di {data.degree?.name}" />
</svelte:head>

<div class="max-w-5xl p-4 mx-auto">
	<nav class="navbar flex bg-base-200 text-neutral-content rounded-box shadow-sm px-5 mb-5">
		<div class="navbar-start">
			{#if login}
				{#await login then login}
					{#if 'error' in login}
						<a class="btn btn-square btn-ghost" href={getLoginUrl($page.url)}> Login </a>
					{:else}
						<img src={login.avatarUrl} alt="User avatar" class="w-10 rounded-xl" />
					{/if}
				{/await}
			{/if}
		</div>
		<div class="navbar min-h-0 p-0 justify-center items-center">
			<h1 class="flex flex-wrap text-xl text-center font-semibold text-base-content">
				{data.degree.name}
			</h1>
		</div>
		<div class="navbar-end flex items-center">
			<a class="btn btn-square btn-ghost" title="Indietro" href="/">
				<span class="text-primary icon-[akar-icons--arrow-back-thick-fill]"></span>
			</a>
		</div>
	</nav>
	<ListTeaching
		years={reorganizedTeachings.mandatory}
		activeYears={namesToTeachings(activeYears)}
		title={''}
		from={data.degree.id}
	/>
	<ListTeaching
		title="facoltativi"
		years={reorganizedTeachings.electives}
		activeYears={namesToTeachings(activeYears)}
		from={data.degree.id}
	/>
</div>
