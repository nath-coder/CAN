<script lang="ts">
    type Documento = {
        titulo: string;
        descripcion: string;
        url: string;
    };
  
    const documentos: Documento[] = [
        {
            titulo: 'Manual del Compilador Karel',
            descripcion: 'Guía completa sobre el compilador y el mundo de Karel.',
            url: '/pdfs/documentacion.pdf'
        },
        {
            titulo: 'Análisis Léxico',
            descripcion: 'Definiciones sintácticas y semánticas del lenguaje CAN.',
            url: '/pdfs/actividad4.pdf'
        },
        {
            titulo: 'Análisis Sintáctico',
            descripcion: 'Cómo utilizar la herramienta paso a paso.',
            url: '/pdfs/actividad5.pdf'
        },
        {
            titulo: 'Análisis Semántico',
            descripcion: 'Resumen de comandos disponibles para Karel.',
            url: '/pdfs/actividad6.pdf'
        }
    ];
</script>

<section class="min-h-screen bg-gradient-to-br from-gray-900 via-purple-900 to-indigo-900 py-12 px-4 sm:px-6 lg:px-8">
    <div class="max-w-7xl mx-auto">
        <!-- Encabezado -->
        <div class="text-center mb-12">
            <h1 class="text-4xl md:text-5xl font-bold text-white mb-4">
                <span class="text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-indigo-300">
                    Documentación
                </span> del Proyecto
            </h1>
            <p class="text-lg text-purple-200 max-w-3xl mx-auto">
                Explora nuestra colección de manuales y guías técnicas que documentan cada aspecto de nuestro trabajo.
            </p>
        </div>

        <!-- Grid de Documentos -->
        <div class="grid gap-8 md:grid-cols-2 lg:grid-cols-2 xl:grid-cols-2">
            {#each documentos as doc}
                <div 
                    class="bg-gradient-to-br from-gray-800/70 to-purple-900/50 backdrop-blur-sm rounded-3xl p-8 border border-purple-500/30 hover:border-purple-400/50 shadow-xl hover:shadow-purple-500/20 transition-all duration-300 hover:-translate-y-1 text-center"
                >
                    <div class="h-full flex flex-col">
                        <div class="flex-grow">
                            <h2 class="text-2xl font-bold text-white mb-3">
                                <span class="bg-gradient-to-r from-purple-400 to-indigo-300 bg-clip-text text-transparent">
                                    {doc.titulo}
                                </span>
                            </h2>
                            <p class="text-purple-200 mb-6">{doc.descripcion}</p>
                        </div>
                        
                        <div class="mt-auto">
                            <a
                                href={doc.url}
                                target="_blank"
                                class="inline-flex items-center justify-center gap-2 bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-500 hover:to-indigo-500 text-white px-6 py-3 rounded-lg transition-all duration-300 font-medium w-full sm:w-auto group"
                            >
                                <svg 
                                    xmlns="http://www.w3.org/2000/svg" 
                                    fill="none" 
                                    viewBox="0 0 24 24" 
                                    stroke-width="1.5" 
                                    stroke="currentColor" 
                                    class="w-5 h-5 group-hover:scale-110 transition-transform"
                                >
                                    <path 
                                        stroke-linecap="round" 
                                        stroke-linejoin="round" 
                                        d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m2.25 0H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z" 
                                    />
                                </svg>
                                Ver Documento
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 ml-1 group-hover:translate-x-1 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3" />
                                </svg>
                            </a>
                        </div>
                    </div>
                </div>
            {/each}
        </div>

        <!-- Nota adicional -->
        <div class="mt-12 text-center">
            <div class="inline-block bg-white/5 backdrop-blur-sm px-6 py-3 rounded-full border border-purple-300/20">
                <p class="text-purple-200 text-sm">
                    Todos los documentos están en formato PDF y se abrirán en una nueva ventana.
                </p>
            </div>
        </div>
    </div>
</section>