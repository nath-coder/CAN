<script lang="ts">
import { onMount } from 'svelte';

export let open = false;
export let onClose: () => void;
export let tablaSimbolos;
export let tokens;
export let opcion = "";

export let ast: ASTNode;

function contarHojas(node: ASTNode): number {
  if (!node.hijos || node.hijos.length === 0) return 1;
  return node.hijos.reduce((acc, hijo) => acc + contarHojas(hijo), 0);
}

type ASTNode = {
  token: number;
  hijos?: ASTNode[];
};

let canvas: HTMLCanvasElement;
let idCounter = 0;

function sintactico() {
  if (canvas) {
    const ctx = canvas.getContext('2d');
    if (ctx) {
      const hojas = contarHojas(ast);
      const nuevoAncho = Math.max(hojas * 100, 800);
      canvas.width = nuevoAncho;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = 'white';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.strokeStyle = 'black';
      ctx.lineWidth = 2;
      idCounter = 0;
      drawNode(ctx, ast, canvas.width / 2, 50, canvas.width / 2);
    }
  }
}
function drawNode(
  ctx: CanvasRenderingContext2D,
  node: ASTNode,
  x: number,
  y: number,
  offset: number
) {
  const nodeId = idCounter++;
  const radius = 20;

  // Dibuja el nodo actual
  ctx.fillStyle = 'lightblue';
  ctx.beginPath();
  ctx.arc(x, y, radius, 0, Math.PI * 2);
  ctx.fill();
  ctx.stroke();

  // Texto centrado
  ctx.fillStyle = 'black';
  ctx.font = '12px Arial';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(`T${node.token}`, x, y);

  if (node.hijos && node.hijos.length > 0) {
    const spacing = Math.max(offset / node.hijos.length, 60); // mínimo 60 px entre hijos
    const startX = x - spacing * (node.hijos.length - 1) / 2;

    node.hijos.forEach((child, index) => {
      const childX = startX + index * spacing;
      const childY = y + 100;

      // Línea al hijo
      ctx.beginPath();
      ctx.moveTo(x, y + radius);
      ctx.lineTo(childX, childY - radius);
      ctx.stroke();

      // Dibujo recursivo
      drawNode(ctx, child, childX, childY, offset * 0.6);
    });
  }
}

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
      {:else if opcion === "arbol sintactico"}
        <h3 class="text-lg font-semibold text-white">Árbol Sintáctico</h3>
         <button class="bg-purple-600 hover:bg-purple-700 transition px-4 py-2 rounded-xl text-sm shadow flex items-center gap-2"
          on:click={sintactico}>Generar</button>
          <br/>
         <div class="flex-1 bg-gray-100 p-4 overflow-auto">
         <div class="w-max overflow-x-auto">
              <canvas
                  id="miCanvas"
                  width={800}
                  height={800}
                  bind:this={canvas}
                  class="border border-gray-400 rounded-lg shadow-lg"
                />
          </div>
	      </div>
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
