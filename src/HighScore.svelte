<script lang="ts">
	import { onMount } from "svelte";
	import { Fire, X } from "svelte-hero-icons";
	import { fade } from "svelte/transition";
	import { formatTime } from "./Countdown.svelte";

	export let score: number;

	let showModal = false;
	let showSubmit = false;
	let name: string;
	let scores: Highscore[];

	$: if (scores) scores.sort((a: Highscore, b: Highscore) => a.time - b.time);

	interface Highscore {
		user: string;
		time: number;
	}

	onMount(async () => {
		scores = await getScores();
		console.log(scores);
	});

	const getScores = async () => {
		let url = new URL(
			"https://api.jsonbin.io/b/60255820435c323ba1c4ddff/latest"
		);
		// url.search = new URLSearchParams({}).toString();

		const rawResponse = await fetch(url as any, {
			headers: {
				Accept: "application/json",
				"Content-Type": "application/json",
				"secret-key":
					"$2b$10$U09tYJeqr/PUBSpn/jp2ZeFdtvVvWf/ZiE0kiAjjl/HEn2Ze.nTqC",
			},
		});

		const content = await rawResponse.json();

		return content;
	};

	const submitScore = async () => {
		const url = "https://api.jsonbin.io/b/60255820435c323ba1c4ddff";

		const data = createScores();

		const rawResponse = await fetch(url, {
			method: "PUT",
			headers: {
				Accept: "application/json",
				"Content-Type": "application/json",
				"secret-key":
					"$2b$10$U09tYJeqr/PUBSpn/jp2ZeFdtvVvWf/ZiE0kiAjjl/HEn2Ze.nTqC",
			},
			body: data,
		});

		const content = await rawResponse.json();

		console.log(content);
	};

	const createScores = () => {
		let changed = false;

		scores.forEach((e) => {
			if (e.user === name) {
				e.time = score;
				changed = true;
			}
		});

		if (changed) scores = [...scores, { user: name, time: score }];
		else scores = scores;

		return JSON.stringify(scores);
	};
</script>

{#if showModal && scores}
	<div
		transition:fade={{ duration: 100 }}
		class="flex items-center justify-center fixed left-0 bottom-0 w-full h-full bg-gray-800"
	>
		<div class="bg-white rounded-lg w-3/4">
			<div class="flex flex-col items-start p-4 h-full">
				<div class="flex flex-initial items-center w-full mb-3">
					<div class="text-gray-900 font-medium text-lg">
						High Scores
					</div>
					<button
						class="ml-auto fill-current text-gray-700 w-6 h-6 cursor-pointer"
						on:click={() => (showModal = false)}
					>
						<X size="1.6x" />
					</button>
				</div>
				<hr />
				<div
					class="space-y-2 flex flex-col max-h-96 overflow-y-scroll w-full"
				>
					{#each scores as { user, time }}
						<div>
							<h3 class="text-xl">{user}</h3>
							<h1 class="text-3xl">{formatTime(time)}s</h1>
						</div>
					{/each}
				</div>
				<hr />
				<div
					class="flex flex-initial w-full space-x-2 mt-2 justify-end"
				>
					{#if showSubmit}
						<input
							type="text"
							class="rounded py-2 px-4 border border-purple-500 flex-grow focus:outline-none focus:ring focus:border-purple-300"
							placeholder="Name"
							bind:value={name}
						/>
						<button
							on:click={submitScore}
							class="bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded"
						>
							Submit
						</button>
						<button
							on:click={() => (showSubmit = false)}
							class="bg-transparent hover:bg-purple-200 text-purple-700 font-semibold hover:text-black py-2 px-4 border border-purple-500 hover:border-transparent rounded"
						>
							Close
						</button>
					{:else}
						<button
							on:click={() => (showSubmit = true)}
							class="bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded"
						>
							Add Score
						</button>
					{/if}
				</div>
			</div>
		</div>
	</div>
{:else}
	<div class="absolute right-2 bottom-2 hidden md:block">
		<button
			on:click={() => (showModal = true)}
			class="text-white px-4 w-auto h-10 bg-red-600 rounded-full hover:bg-red-700 active:shadow-lg mouse shadow transition ease-in duration-200 focus:outline-none"
		>
			<Fire
				class="w-6 h-6 inline-block mr-1 stroke-current text-white"
				size="1.5x"
			/>
			<span>High Scores</span>
		</button>
	</div>
{/if}
