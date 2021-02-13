<script lang="ts" context="module">
	/** Takes a time in ms and converts to XX:XX.XXX format */
	export const formatTime = (mili: number) => {
		/** The number of minutes, floored. 60000ms = 1000ms * 60s = 1m */
		let minutes = Math.floor(mili / 60000);
		/** The number of seconds, floored. 1000ms = 1s */
		let seconds = Math.floor(mili / 1000) - minutes * 60;
		/** Subtract the minutes and seconds, then divide by 1000 to get it to a decimal. Then round it to 3 digits, and remove the leading 0*/
		let ms = ((mili - minutes * 60000 - Number(seconds) * 1000) / 1000)
			.toFixed(3)
			.substring(1);

		// If time is over 1 minute, pad seconds with appropriate 0s, else return seconds+ms
		if (minutes) {
			return minutes + ":" + String(seconds).padStart(2, "0") + ms;
		} else {
			return seconds + ms;
		}
	};
</script>

<script lang="ts">
	import { differenceInMilliseconds } from "date-fns";
	import { XCircle } from "svelte-hero-icons";
	import party from "party-js";
	import goodSound from "./good.mp3";
	import badSound from "./bad.mp3";
	import HighScore from "./HighScore.svelte";

	let text: any = "Hold Space To Begin";
	let initialDelay: any;
	let inspectionCountdown: any;
	let inspectionDelay: any;
	let dnfDelay: any;
	let dnfCountdown: any;
	let timer: any;
	let startTime: any;
	let endTime: any;
	let previousTimes: number[] = JSON.parse(
		localStorage.getItem("CubeData") || "[]"
	);
	let textColour: string;
	let textSize = "text-3xl";
	/** The average solve time */
	let averageTime = 0;
	/** The minimum solve time */
	let minTime = 0;
	/** The maximum solve time */
	let maxTime = 0;

	// Interface for presed key
	interface Key {
		code: string;
		repeat?: boolean;
	}

	// Set min,max,avg when previoudTimes is modified. If there are no entries, they default to 0
	$: {
		if (previousTimes.length) {
			minTime = Math.min(...previousTimes);
			maxTime = Math.max(...previousTimes);
			// Little sorting algo from stack
			averageTime =
				previousTimes.reduce((p, c) => (c += p)) / previousTimes.length;
		} else {
			minTime = 0;
			maxTime = 0;
			averageTime = 0;
		}
	}

	// Update the localStorage whenever previousTimes changes
	$: localStorage.setItem("CubeData", JSON.stringify(previousTimes));

	/** Receive key presses from the main div */
	const keyPressed = ({ code, repeat }: Key) => {
		if (code === "Space") {
			// Prevent repeat presses
			if (repeat) return;

			// Make sure text is black
			textColour = "bg-white";

			// If timer is active, stop the timer
			if (timer) {
				clearInterval(timer);
				timer = undefined;
				endTimer();
			} else {
				// Else, start the preCountdown
				preCountdown();
			}
		}
	};

	/** Receive key releases from the main div */
	const keyReleased = ({ code }: Key) => {
		if (code === "Space") {
			if (initialDelay) {
				clearTimeout(initialDelay);
				initialDelay = undefined;
				text = "Hold Space To Begin";
				textSize = "text-3xl";
			} else if ((inspectionDelay || dnfDelay) && text !== "DNF") {
				clearInterval(inspectionCountdown);
				clearTimeout(inspectionDelay);
				clearInterval(dnfCountdown);
				clearTimeout(dnfDelay);
				inspectionDelay = undefined;
				dnfDelay = undefined;
				beginTimer();
			}
		}
	};

	const preCountdown = () => {
		// Begin countdown to ensure user is holding down space
		initialDelay = setTimeout(beginCountdown, 1000);
		textSize = "text-6xl md:text-9xl";
		text = ".";
		let dots = setInterval(() => {
			if (inspectionDelay || text === "Hold Space To Begin") {
				clearInterval(dots);
				return;
			}
			text += ".";
		}, 200);
	};

	const beginCountdown = () => {
		initialDelay = undefined;
		text = 15;
		inspectionCountdown = setInterval(() => {
			--text;
			if (text <= 3) textColour = "bg-red-500";
			else if (text <= 7) textColour = "bg-yellow-500";
		}, 1000);
		inspectionDelay = setTimeout(() => {
			clearInterval(inspectionCountdown);
			overTime();
		}, 15000);
	};

	const overTime = () => {
		text = 0;
		dnfCountdown = setInterval(() => {
			text = "+2";
		}, 1000);
		dnfDelay = setTimeout(() => {
			clearInterval(dnfCountdown);
			text = "DNF";
			clearTimeout(initialDelay);
			inspectionDelay = undefined;
			initialDelay = undefined;
			timer = undefined;
		}, 3000);
	};

	const beginTimer = () => {
		textColour = "bg-white";
		text = "Go!";
		startTime = new Date();
		timer = setInterval(() => {
			if (text === "Go!") text = 1;
			else text++;
		}, 1000);
	};

	const endTimer = () => {
		previousTimes = [
			differenceInMilliseconds(new Date(), startTime),
			...previousTimes,
		];
		text = formatTime(previousTimes[0]) + "s";
		updateRecords();
	};

	const removeTime = (index: number) => {
		previousTimes.splice(index, 1);
		previousTimes = previousTimes;
	};

	const updateRecords = () => {
		if (previousTimes.length > 1)
			if (previousTimes[0] < averageTime) {
				party.screen({
					color: [
						"#009B48",
						"#B90000",
						"#0045AD",
						"#FF5900",
						"#FFFFFF",
						"#FFD500",
					],
					size: party.minmax(6, 12),
					count: party.variation(
						300 * (window.innerWidth / 1980),
						0.4
					),
					angle: -180,
					spread: 80,
					angularVelocity: party.minmax(6, 9),
				});
				new Audio(goodSound).play();
			} else if (previousTimes[0] > averageTime * 1.25) {
				new Audio(badSound).play();
			}
	};
</script>

<div hidden id="dummy" />
<HighScore score={minTime} />
<section
	class={`text-gray-600 body-font w-full h-full flex md:items-center items-end ${textColour}`}
	tabindex="0"
	on:keydown={keyPressed}
	on:keyup={keyReleased}
>
	<div
		class="p-5 flex flex-wrap w-full h-full md:items-center items-end mx-auto"
	>
		<div class="md:w-1/2 text-center w-full">
			<h1 class={`title-font font-medium text-gray-900 ${textSize}`}>
				{text}
			</h1>
		</div>

		<div
			class="md:w-1/2 bg-gray-100 rounded-lg p-8 flex flex-col md:ml-auto w-full"
		>
			<div class="w-full flex flex-row space-x-4">
				<div
					class="w-1/3 text-white cursor-default text-center bg-purple-500 border-0 py-2 px-3 rounded text-lg"
				>
					<span class="text-2xl">Best</span>
					<div
						class=" text-white cursor-default text-center bg-purple-700 border-0 py-2 px-3 mt-2 rounded text-lg"
					>
						{formatTime(minTime)}s
					</div>
				</div>
				<div
					class="w-1/3 text-white cursor-default text-center bg-purple-500 border-0 py-2 px-3 rounded text-lg"
				>
					<span class="text-2xl">Worst</span>
					<div
						class=" text-white cursor-default text-center bg-purple-700 border-0 py-2 px-3 mt-2 rounded text-lg"
					>
						{formatTime(maxTime)}s
					</div>
				</div>
				<div
					class="w-1/3 text-white cursor-default text-center bg-purple-500 border-0 py-2 px-3 rounded text-lg"
				>
					<span class="text-2xl">Avg of {previousTimes.length}</span>
					<div
						class=" text-white cursor-default text-center bg-purple-700 border-0 py-2 px-3 mt-2 rounded text-lg"
					>
						{formatTime(averageTime)}s
					</div>
				</div>
			</div>

			<h2 class="text-gray-900 text-xl font-medium title-font mt-5 mb-3">
				Previous Times
				<button
					on:click={() => (previousTimes = [])}
					class="text-white text-center bg-purple-500 border-0 py-2 px-3 hover:bg-purple-600 rounded text-lg float-right"
				>
					Clear
				</button>
			</h2>
			<hr class="mb-5 border-gray-400" />
			<div class="space-y-2 overflow-y-scroll h-56">
				{#each previousTimes as time, index}
					{#if time < averageTime}
						<div
							class="text-white cursor-default text-center bg-green-500 border-0 py-2 px-3 hover:bg-green-600 rounded text-lg"
						>
							<span>{formatTime(time)}s</span>

							<div
								class="float-right cursor-pointer"
								on:click={() => removeTime(index)}
							>
								<XCircle size="1.6x" />
							</div>
						</div>
					{:else if time > averageTime}
						<div
							class="text-white cursor-default text-center bg-red-500 border-0 py-2 px-3 hover:bg-red-600 rounded text-lg"
						>
							<span>{formatTime(time)}s</span>

							<div
								class="float-right cursor-pointer"
								on:click={() => removeTime(index)}
							>
								<XCircle size="1.6x" />
							</div>
						</div>
					{:else}
						<div
							class="text-white cursor-default text-center bg-purple-500 border-0 py-2 px-3 hover:bg-purple-600 rounded text-lg"
						>
							<span>{formatTime(time)}s</span>

							<div
								class="float-right cursor-pointer"
								on:click={() => removeTime(index)}
							>
								<XCircle size="1.6x" />
							</div>
						</div>
					{/if}
				{/each}
			</div>
		</div>
	</div>
</section>

<style>
	#dummy {
		@apply text-3xl;
		@apply text-6xl;
		@apply text-9xl;
		@apply text-black;
		@apply bg-yellow-500;
		@apply bg-red-500;
		@apply bg-green-500;
		@apply bg-red-500;
	}

	/* Hide scrollbar for Chrome, Safari and Opera */
	.overflow-y-scroll::-webkit-scrollbar {
		display: none;
	}

	/* Hide scrollbar for IE, Edge and Firefox */
	:global(.overflow-y-scroll) {
		-ms-overflow-style: none; /* IE and Edge */
		scrollbar-width: none; /* Firefox */
	}
</style>
