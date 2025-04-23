
<script lang="ts">
	import { page } from '$app/state'; // ← store correcto para `page`
	import logo from '$lib/images/svelte-logo.svg';
	import github from '$lib/images/github.svg';

	let isOpen = false;

	const toggleMenu = () => {
		isOpen = !isOpen;
	};
</script>

<header class="fixed w-full top-0 left-0 z-50 bg-zinc-900 text-white shadow-lg">
	<div class="max-w-7xl mx-auto px-6 py-4 flex justify-between items-center">
		<!-- Logo -->
		<a href="/" class="flex items-center gap-3">
			<img src={logo} alt="SvelteKit" class="w-8 h-8" />
			<span class="text-xl font-extrabold tracking-wide text-orange-400">CAN</span>
		</a>

		<!-- Botón Hamburguesa -->
		<button class="md:hidden focus:outline-none" on:click={toggleMenu}>
			<svg class="h-6 w-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
					d="M4 6h16M4 12h16M4 18h16" />
			</svg>
		</button>

		<!-- Menú de Escritorio -->
		<nav class="hidden md:flex items-center space-x-8 text-sm font-semibold">
			<a href="/" class="hover:text-orange-400 transition" aria-current={page.url.pathname === '/' ? 'page' : undefined}>Inicio</a>
			<a href="/about" class="hover:text-orange-400 transition" aria-current={page.url.pathname === '/about' ? 'page' : undefined}>Documentacion</a>
			<a href="/sverdle" class="hover:text-orange-400 transition" aria-current={page.url.pathname.startsWith('/sverdle') ? 'page' : undefined}>Nosotros</a>
			<a href="https://github.com/sveltejs/kit" target="_blank">
				<img src={github} alt="GitHub" class="w-5 h-5 hover:opacity-80 transition" />
			</a>
		</nav>
	</div>

	<!-- Menú móvil desplegable -->
	{#if isOpen}
		<div class="md:hidden bg-zinc-900 px-6 pb-4 space-y-3 text-sm font-semibold">
			<a href="/" class="block hover:text-orange-400" on:click={() => (isOpen = false)}>Inicio</a>
			<a href="/about" class="block hover:text-orange-400" on:click={() => (isOpen = false)}>Acerca</a>
			<a href="/sverdle" class="block hover:text-orange-400" on:click={() => (isOpen = false)}>Documentacion</a>
			<a href="https://github.com/sveltejs/kit" class="block" target="_blank">
				<img src={github} alt="GitHub" class="w-5 h-5 inline-block hover:opacity-80 transition" />
			</a>
		</div>
	{/if}
</header>

<style>
	:global(body) {
		padding-top: 4.5rem;
		background-color: #18181b; /* Fondo oscuro */
		font-family: 'Inter', sans-serif;
	}
</style>