<script lang="ts">
	import Header from './Header.svelte';
	import Footer from './Footer.svelte';
	import '../app.css';
	
	let { children } = $props();
</script>

<div class="app">
	<Header />

	<main class="flex-grow">
		{@render children()}
	</main>

	<Footer />

</div>

<style>
	.app {
	display: flex;
	flex-direction: column;
	min-height: 100vh;
	padding-top: 0%;
}

main {
	flex: 1;
	display: flex;
	flex-direction: column;
	margin: 0 auto;
	box-sizing: border-box;
}

footer {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	padding: 12px;
}

footer a {
	font-weight: bold;
}

@media (min-width: 480px) {
	footer {
		padding: 12px 0;
	}
}
</style>
