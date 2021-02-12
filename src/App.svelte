<script lang="ts">
	import Countdown from "./Countdown.svelte";
	import Tailwind from "./Tailwind.svelte";
</script>

<Tailwind />
<main>
	<Countdown />
</main>

<style>
	main {
		@apply h-screen;
		@apply w-full;
	}
</style>
