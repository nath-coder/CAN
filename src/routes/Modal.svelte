<script lang="ts">
    export let open = false;
    export let onClose: () => void;
    export let tablaSimbolos;
    export let tokens;
    export let opcion="";
  </script>
  
  {#if open}
    <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div class="bg-white rounded-2xl shadow-lg p-6 w-[90%] max-w-md">
        <h2 class="text-xl font-bold mb-4">Ventana emergente</h2>
        {#if opcion === "tabla de simbolos"}
          <h3 class="text-lg font-semibold">Tabla de Símbolos</h3>
          <pre>{JSON.stringify(tablaSimbolos, null, 2)}</pre>
        {:else if opcion === "tokens"}
          <h3 class="text-lg font-semibold">Tokens</h3>
          <pre>{JSON.stringify(tokens, null, 2)}</pre>
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
  