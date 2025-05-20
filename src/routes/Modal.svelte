<script lang="ts">
import { onMount } from 'svelte';

export let open = false;
export let onClose: () => void;
export let tablaSimbolos;
export let tokens;
export let opcion = "";

export let ast: ASTNode;
  type ASTNode =
    | { tipo: 'programa', cuerpo: ASTNode[]; funciones: ASTNode[] }
    | { tipo: 'define', nombre: number, cuerpo: ASTNode[] }
    | { tipo: 'llamada_funcion', nombre: number }
    | { tipo: 'avanza' }
    | { tipo: 'gira' }
    | { tipo: 'toma' }
    | { tipo: 'deja' }
    | { tipo: 'apagate' }
    | { tipo: 'repite', veces: number, cuerpo: ASTNode[] }
    | { tipo: 'mientras', condicion: Condicion, cuerpo: ASTNode[] }
    | { tipo: 'si', condicion: Condicion, entonces: ASTNode[], sino?: ASTNode[] };

  type Condicion =
    | { tipo: 'condicion_base', nombre: number }
    | { tipo: 'not', valor: Condicion }
    | { tipo: 'and', izquierda: Condicion, derecha: Condicion }
    | { tipo: 'or', izquierda: Condicion, derecha: Condicion };

  let canvas: HTMLCanvasElement;
  let idCounter = 0;

  // Parámetros de layout
  const nodeWidth = 120;
  const nodeHeight = 40;
  const horizontalSpacing = 30;
  const verticalSpacing = 80;

  function sintactico() {
    const ctx = canvas.getContext('2d');
    if (!ctx) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    idCounter = 0;

    ctx.font = '14px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';

    // Calculamos posiciones y dibujamos
    const startX = canvas.width / 2;
    const startY = 20;

    dibujarNodo(ast, startX, startY, ctx);
  }

  // Función que dibuja un nodo y devuelve el ancho total ocupado por el nodo y sus hijos
  function dibujarNodo(nodo: ASTNode | Condicion, x: number, y: number, ctx: CanvasRenderingContext2D): number {
    const hijos: (ASTNode | Condicion)[] = [];

    if ('cuerpo' in nodo && Array.isArray(nodo.cuerpo)) hijos.push(...nodo.cuerpo);
    if ('entonces' in nodo && Array.isArray(nodo.entonces)) hijos.push(...nodo.entonces);
    if ('sino' in nodo && Array.isArray(nodo.sino)) hijos.push(...nodo.sino);
    if ('funciones' in nodo && Array.isArray(nodo.funciones)) hijos.push(...nodo.funciones);
    if ('condicion' in nodo && nodo.condicion) hijos.unshift(nodo.condicion);

    // Si no tiene hijos, dibuja solo el nodo
    if (hijos.length === 0) {
      dibujarRectangulo(ctx, x, y, nodo);
      return nodeWidth;
    }

    // Primero calculamos el ancho total que ocuparán todos los hijos juntos
    const hijosAnchos = hijos.map(h => dibujarNodoObtenerAncho(h));
    const totalAncho = hijosAnchos.reduce((a, b) => a + b, 0) + (hijos.length - 1) * horizontalSpacing;

    // Posición inicial para el primer hijo (centrado respecto al padre)
    let currentX = x - totalAncho / 2;

    // Dibuja líneas y hijos
    hijos.forEach((hijo, i) => {
      const anchoHijo = hijosAnchos[i];
      const hijoX = currentX + anchoHijo / 2;
      const hijoY = y + verticalSpacing;

      // Línea desde centro abajo del nodo padre al centro arriba del hijo
      ctx.beginPath();
      ctx.moveTo(x, y + nodeHeight);
      ctx.lineTo(hijoX, hijoY);
      ctx.stroke();

      // Dibuja recursivamente el hijo
      dibujarNodo(hijo, hijoX, hijoY, ctx);

      currentX += anchoHijo + horizontalSpacing;
    });

    // Dibuja el nodo actual encima de sus hijos
    dibujarRectangulo(ctx, x, y, nodo);

    return totalAncho > nodeWidth ? totalAncho : nodeWidth;
  }

  // Función auxiliar para estimar el ancho que ocupará un nodo (sin dibujar)
  function dibujarNodoObtenerAncho(nodo: ASTNode | Condicion): number {
    let hijos: (ASTNode | Condicion)[] = [];
    if ('cuerpo' in nodo && Array.isArray(nodo.cuerpo)) hijos.push(...nodo.cuerpo);
    if ('entonces' in nodo && Array.isArray(nodo.entonces)) hijos.push(...nodo.entonces);
    if ('sino' in nodo && Array.isArray(nodo.sino)) hijos.push(...nodo.sino);
    if ('funciones' in nodo && Array.isArray(nodo.funciones)) hijos.push(...nodo.funciones);
    if ('condicion' in nodo && nodo.condicion) hijos.unshift(nodo.condicion);

    if (hijos.length === 0) return nodeWidth;

    const hijosAnchos = hijos.map(dibujarNodoObtenerAncho);
    const totalAncho = hijosAnchos.reduce((a, b) => a + b, 0) + (hijos.length - 1) * horizontalSpacing;
    return totalAncho > nodeWidth ? totalAncho : nodeWidth;
  }

  function dibujarRectangulo(ctx: CanvasRenderingContext2D, x: number, y: number, nodo: ASTNode | Condicion) {
    const colorFondo = esCondicion(nodo) ? '#9b59b6' : '#8e44ad';

    ctx.fillStyle = colorFondo;
    ctx.fillRect(x - nodeWidth / 2, y, nodeWidth, nodeHeight);
    ctx.strokeStyle = 'black';
    ctx.strokeRect(x - nodeWidth / 2, y, nodeWidth, nodeHeight);
    ctx.fillStyle = 'white';

    // Texto multilinea para mayor claridad si es muy largo
    const texto = formatearTexto(nodo);
    const lineas = texto.split('\n');
    lineas.forEach((linea, i) => {
      ctx.fillText(linea, x, y + nodeHeight / 2 + (i - (lineas.length - 1) / 2) * 16);
    });
  }

  function esCondicion(obj: any): obj is Condicion {
    return obj && typeof obj === 'object' && ['condicion_base', 'not', 'and', 'or'].includes(obj.tipo);
  }

  function formatearTexto(nodo: ASTNode | Condicion): string {
    switch (nodo.tipo) {
      case 'define':
      case 'llamada_funcion':
      case 'condicion_base':
        return `${nodo.tipo} (${(nodo as any).nombre})`;
      case 'repite':
        return `repite ${(nodo as any).veces}`;
      case 'and':
      case 'or':
        return `${nodo.tipo}\n${formatearTexto(nodo.izquierda)}\n${formatearTexto(nodo.derecha)}`;
      case 'not':
        return `not\n${formatearTexto(nodo.valor)}`;
      default:
        return nodo.tipo;
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
