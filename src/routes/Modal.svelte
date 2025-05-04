<script lang="ts">
    export let open = false;
    export let onClose: () => void;
    export let tablaSimbolos;
    export let tokens;
    export let opcion="";
  </script>
  
  {#if open}
    <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 scroll-auto">
      <div class="bg-slate-800 rounded-2xl shadow-lg p-6 w-[90%] max-w-md">
        {#if opcion === "tabla de simbolos"}
          <h3 class="text-lg font-semibold">Tabla de Símbolos</h3>

          {#if tablaSimbolos.length > 0}
            <table class="table-auto w-full border-collapse border border-gray-300 mt-4 text-sm">
              <thead class="bg-purple-700 text-white">
                <tr>
                  <th class="border px-3 py-2">Nombre</th>
                  <th class="border px-3 py-2">Tipo</th>
                  <th class="border px-3 py-2">Línea</th>
                  <th class="border px-3 py-2">Ámbito</th>
                  <th class="border px-3 py-2">Valor</th>
                </tr>
              </thead>
              <tbody>
                {#each tablaSimbolos as simbolo}
                  <tr class="hover:bg-purple-100">
                    <td class="border px-2 py-1">{simbolo.nombre}</td>
                    <td class="border px-2 py-1">{simbolo.tipo}</td>
                    <td class="border px-2 py-1">{simbolo.linea}</td>
                    <td class="border px-2 py-1">{simbolo.ambito}</td>
                    <td class="border px-2 py-1">
                      {#if simbolo.valor !== undefined}
                        {typeof simbolo.valor === 'object' ? JSON.stringify(simbolo.valor) : simbolo.valor}
                      {:else}
                        -
                      {/if}
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          {:else}
            <p class="text-gray-500 mt-2">No hay símbolos registrados.</p>
          {/if}
        {:else if opcion === "tokens"}
        <h3 class="text-lg font-semibold">Tokens</h3>
          <div class="flex flex-wrap gap-2 mt-2">
            {#each tokens as token}
              <span class="bg-purple-200 text-purple-800 text-sm font-medium px-3 py-1 rounded-full shadow">
                {token}
              </span>
            {/each}
          </div>
        {:else}
          <p>Opción no válida.</p>
        {/if}
        <button 
          class="mt-4 bg-purple-600 text-white px-4 py-2 rounded hover:bg-purple-700"
          on:click={onClose}
        >
          Cerrar
        </button>
      </div>
    </div>
  {/if}
  