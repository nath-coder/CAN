<svelte:head>
	<title>Acerca de Nosotros | Equipo 4</title>
	<meta name="description" content="Conoce al equipo detrás del proyecto de Lenguajes y Autómatas II" />
</svelte:head>

<script>
	import { fade } from 'svelte/transition';
	import Creators from "../Creators.svelte";
	
	const team = [
		{
			name: "Cristian Eduardo Castañeda Perez",
			role: "Creación de matrices de transición",
			image: "/images/cris_foto.jpg",
			description: "Apasionado por la programación y creador del proyecto."
		},
		{
			name: "Diana Nathasha Macías Sevilla",
			role: "Desarrolladora de Backend",
			image: "/images/nathasha.png",
			description: "Especialista en estructuras de datos y compiladores."
		}
	];
	
	const tutor = {
		name: "Ricardo González González",
		role: "Tutor",
		image: "/images/imagen.jpg",
		description: "Especialista en lenguajes de programación y autómatas."
	}
</script>

<section class="min-h-screen bg-gradient-to-br from-purple-900 via-purple-800 to-indigo-900 py-12 px-4 sm:px-6 lg:px-8">
	<div class="max-w-7xl mx-auto">
		<!-- Encabezado -->
		<div class="text-center mb-16" in:fade>
			<h1 class="text-4xl md:text-5xl font-bold text-white mb-6">
				ACERCA DE <span class="text-purple-300">NOSOTROS</span>
			</h1>
			
			<div class="max-w-3xl mx-auto">
				<p class="text-lg text-purple-100 leading-relaxed mb-6">
					Somos el <span class="font-semibold text-white">Equipo 4</span> de la materia Lenguajes y Autómatas II, y estamos aquí para demostrar que la teoría no solo se aprende... ¡también se vive, se construye y se programa!
				</p>
				
				<p class="text-lg text-purple-100 leading-relaxed mb-8">
					💡 En este proyecto, combinamos lógica, creatividad y tecnología para dar vida a soluciones que nacen desde el corazón de la computación teórica: desde compiladores hasta simuladores de autómatas, todo hecho por nosotros, con pasión y dedicación.
				</p>
				
				<div class="inline-block bg-white/10 backdrop-blur-sm px-6 py-3 rounded-full border border-purple-300/30">
					<p class="text-purple-200 font-medium italic">
						"Donde el código se encuentra con la teoría... y juntos hacen magia."
					</p>
				</div>
			</div>
		</div>

		<!-- Tutor -->
		<div class="mb-20" in:fade>
			<h2 class="text-3xl font-bold text-center text-white mb-12">
				NUESTRO <span class="text-purple-300">TUTOR</span>
			</h2>
			
			<div class="grid md:grid-cols-2 gap-8 max-w-5xl mx-auto">
				<div class="bg-gradient-to-br from-purple-700/30 to-indigo-800/30 backdrop-blur-sm rounded-3xl shadow-xl p-8 hover:scale-[1.02] transform transition-all duration-300 border border-white/10">
					<div class="flex flex-col items-center text-center">
						<div class="relative mb-6">
							<img src={tutor.image} alt={tutor.name} 
								class="w-40 h-40 rounded-full object-cover border-4 border-purple-300/50 shadow-lg" />
							<div class="absolute -bottom-2 -right-2 bg-purple-500 text-white text-xs font-bold px-3 py-1 rounded-full">
								Tutor
							</div>
						</div>
						<h3 class="text-2xl font-bold text-white mb-2">{tutor.name}</h3>
						<p class="text-purple-300 font-medium mb-4">{tutor.role}</p>
						<p class="text-purple-200">{tutor.description}</p>
					</div>
				</div>
				
				<div class="flex items-center">
					<div class="bg-white/5 backdrop-blur-sm rounded-3xl p-8 border border-white/10 h-full">
						<h3 class="text-xl font-semibold text-white mb-4">Su contribución</h3>
						<p class="text-purple-200 mb-4">
							Nuestro tutor ha sido fundamental en guiarnos a través de los conceptos complejos de lenguajes formales y autómatas.
						</p>
						<ul class="space-y-3 text-purple-300">
							<li class="flex items-start">
								<svg class="w-5 h-5 mt-0.5 mr-2 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
								</svg>
								Experto en teoría de la computación
							</li>
							<li class="flex items-start">
								<svg class="w-5 h-5 mt-0.5 mr-2 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
								</svg>
								Guía en el desarrollo del proyecto
							</li>
							<li class="flex items-start">
								<svg class="w-5 h-5 mt-0.5 mr-2 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
								</svg>
								Apoyo en la resolución de problemas
							</li>
						</ul>
					</div>
				</div>
			</div>
		</div>

		<!-- Equipo -->
		<div in:fade>
			<h2 class="text-3xl font-bold text-center text-white mb-12">
				NUESTRO <span class="text-purple-300">EQUIPO</span>
			</h2>
			
			<div class="grid gap-8 md:grid-cols-2 lg:grid-cols-2">
				{#each team as member}
					<div class="bg-gradient-to-br from-purple-700/20 to-indigo-800/20 backdrop-blur-sm rounded-3xl shadow-lg p-8 hover:scale-[1.03] transform transition-all duration-300 border border-white/10">
						<div class="flex flex-col items-center text-center">
							<img src={member.image} alt={member.name} 
								class="w-32 h-32 rounded-full object-cover border-4 border-purple-300/50 shadow-md mb-6" />
							<h3 class="text-xl font-bold text-white mb-2">{member.name}</h3>
							<p class="text-purple-300 font-medium mb-4">{member.role}</p>
							<p class="text-purple-200 text-sm">{member.description}</p>
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>
</section>