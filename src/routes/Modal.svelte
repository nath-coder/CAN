<script lang="ts">
  export let open = false;
  export let onClose: () => void;
  export let tablaSimbolos;
  export let tokens;
  export let opcion = "";
</script>

{#if open}
  <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
    <div class="bg-slate-800 rounded-2xl shadow-lg p-6 w-[95%] max-w-3xl max-h-[90vh] overflow-y-auto">
      {#if opcion === "tabla de simbolos"}
        <h3 class="text-lg font-semibold text-white">Tabla de Símbolos</h3>

        {#if tablaSimbolos.length > 0}
          <div class="overflow-x-auto mt-4">
            <table class="table-auto w-full border-collapse border border-gray-300 text-sm">
              <thead class="bg-purple-700 text-white sticky top-0">
                <tr>
                  <th class="border px-3 py-2">Valor</th>
                  <th class="border px-3 py-2">Tokens</th>
                  <th class="border px-3 py-2">Tipo</th>
                  <th class="border px-3 py-2">Línea</th>
                  <th class="border px-3 py-2">Ámbito</th>
                </tr>
              </thead>
              <tbody>
                {#each tablaSimbolos as simbolo}
                  <tr class="hover:bg-purple-100">
                    <td class="border px-2 py-1">{simbolo.nombre}</td>
                    <td class="border px-2 py-1">
                      {#if simbolo.token !== undefined}
                        {typeof simbolo.token === 'object' ? JSON.stringify(simbolo.token) : simbolo.token}
                      {:else}
                        -
                      {/if}
                    </td>
                    <td class="border px-2 py-1">{simbolo.tipo}</td>
                    <td class="border px-2 py-1">{simbolo.linea}</td>
                    <td class="border px-2 py-1">{simbolo.ambito}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {:else}
          <p class="text-gray-400 mt-2">No hay símbolos registrados.</p>
        {/if}

      {:else if opcion === "tokens"}
        <h3 class="text-lg font-semibold text-white">Tokens</h3>
        <div class="flex flex-wrap gap-2 mt-2 max-h-[60vh] overflow-y-auto pr-1">
          {#each tokens as token}
            <span class="bg-purple-200 text-purple-800 text-sm font-medium px-3 py-1 rounded-full shadow">
              {token}
            </span>
          {/each}
        </div>

      {:else}
        <p class="text-white">Opción no válida.</p>
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
