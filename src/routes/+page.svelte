<script lang="ts">
import { onMount } from 'svelte';

let canvas: HTMLCanvasElement;
const gridSize = 100;
const cellSize = 25;
const canvasSize = gridSize * cellSize;

let wallsV: { [key: string]: boolean} = {};
let wallsH: { [key: string]: boolean} = {};
let zumbadores: { [key: string]: number } = {};


let code='';
function tokenize(input: string) {
    const tokens = input.split(/(\s+|\b)/).filter(t => t.length > 0);

    return tokens.map(token => {
      if (['if', 'while', 'move', 'turnLeft'].includes(token)) {
        return `<span class="text-blue-500">${token}</span>`;
      } else if (/^\d+$/.test(token)) {
        return `<span class="text-green-500">${token}</span>`;
      } else if (/^[a-zA-Z_]\w*$/.test(token)) {
        return `<span class="text-yellow-500">${token}</span>`;
      } else {
        return `<span class="text-white">${token}</span>`;
      }
    }).join('');
}

$: highlightedCode = tokenize(code);

function drawWorld() {
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  ctx.clearRect(0, 0, canvasSize, canvasSize);

  // Dibujar la cuadrícula
  ctx.strokeStyle = '#ccc';
  for (let i = 0; i <= gridSize; i++) {
	const pos = i * cellSize;
	ctx.beginPath();
	ctx.moveTo(pos, 0);
	ctx.lineTo(pos, canvasSize);
	ctx.stroke();

	ctx.beginPath();
	ctx.moveTo(0, pos);
	ctx.lineTo(canvasSize, pos);
	ctx.stroke();
  }

  // Dibujar paredes
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 2;
  for (const key in wallsH) {
	const [x, y] = key.split(',').map(Number);
	const type = wallsH[key];

	if(wallsH[key]){
	  ctx.beginPath();
	  ctx.moveTo(x * cellSize, y * cellSize);
	  ctx.lineTo((x + 1) * cellSize, y * cellSize);
	  ctx.stroke();
	}
  }

  for (const key in wallsV) {
	const [x, y] = key.split(',').map(Number);
	const type = wallsV[key];

	if(wallsV[key]){
	  ctx.beginPath();
	  ctx.moveTo(x * cellSize, y * cellSize);
	  ctx.lineTo(x * cellSize, (y + 1) * cellSize);
	  ctx.stroke();
	}
	
  }
  // Dibujar zumbadores

  for (const key in zumbadores) {
	const [x, y] = key.split(',').map(Number);
	const cx = x * cellSize + cellSize / 2;
	const cy = y * cellSize + cellSize / 2;

	ctx.fillStyle = 'blue';
	ctx.beginPath();
	ctx.arc(cx, cy, cellSize / 4, 0, 2 * Math.PI);
	ctx.fill();

	ctx.fillStyle = 'white'; // O el color que se vea bien sobre el círculo
	ctx.font = `${cellSize * 0.4}px sans-serif`;
	ctx.textAlign = 'center';
	ctx.textBaseline = 'middle';
	ctx.fillText(zumbadores[key].toString(), cx, cy);
  }
}

function handleClick(event: MouseEvent) {
  const rect = canvas.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;

  const gridX = Math.floor(x / cellSize);
  const gridY = Math.floor(y / cellSize);
  const dx = x % cellSize;
  const dy = y % cellSize;

  const border = 2;


	// Clic izquierdo
	if (dy < border) {
	  // borde superior -> pared horizontal
	  const key = `${gridX},${gridY}`;
	  wallsH[key] = !wallsH[key] 
	} else {
		if (dx < border) {
		// borde izquierdo -> pared vertical
		const key = `${gridX},${gridY}`;
		wallsV[key] = !wallsV[key] 
		} else if(event.button === 0){
			const key = `${gridX},${gridY}`;
			zumbadores[key] = (zumbadores[key] || 0) + 1;
		}else{
			const key = `${gridX},${gridY}`;
			if(zumbadores[key]){
				zumbadores[key] = zumbadores[key] - 1;
				if(zumbadores[key] <= 0){
					delete zumbadores[key];
				}
			}
		}
	}
  

  drawWorld();
}

onMount(() => {
  drawWorld();
});
</script>

<svelte:head>
	<title>CAN</title>
	<meta name="description" content="if you imagite, you can do" />
</svelte:head>

<div class="flex flex-col h-screen">
	<!-- Barra superior -->
	<div class="bg-gray-800 text-white flex items-center justify-between px-4 py-2 shadow-md">
	  <div class="flex gap-2">
		<button class="bg-purple-600 hover:bg-purple-700 px-4 py-2 rounded-xl text-sm shadow">
		  Compilar
		</button>
		<button class="bg-indigo-600 hover:bg-indigo-700 px-4 py-2 rounded-xl text-sm shadow">
		  Ejecutar
		</button>
	  </div>
	  <div>
		<input
		  type="number"
		  min="0"
		  placeholder="Zumbadores"
		  class="rounded-xl px-3 py-1 text-black text-sm w-32"
		/>
	  </div>
	</div>
  
	<!-- Cuerpo -->
	<div class="flex flex-1 overflow-hidden">
	  <!-- Barra lateral izquierda -->

	  

	  <div class="w-1/4 bg-gray-900 text-white flex flex-col border-r border-gray-700">
		<div class="relative w-full h-60 bg-gray-900 rounded-md font-mono text-sm">
			<!-- Div de resaltado léxico -->
			<div class="absolute top-0 left-0 right-0 bottom-0 p-2 whitespace-pre-wrap pointer-events-none text-wrap"> {@html highlightedCode}</div>
		  
			<!-- Entrada editable -->
			<textarea
			  class="absolute top-0 left-0 right-0 bottom-0 p-2 bg-transparent text-transparent caret-white selection:bg-indigo-500 w-full h-full resize-none overflow-auto"
			  bind:value={code}
			></textarea>
		</div>
  
		<!-- Mensajes de error -->
		<div class="h-32 overflow-y-auto p-2 text-red-400 text-xs border-t border-gray-700 bg-gray-950">
		  <p class="italic text-gray-500">Sin errores</p>
		</div>
	  </div>
  
	  <!-- Mundo de Karel -->
	  <div class="flex-1 bg-gray-100 p-4 overflow-auto">
		<canvas
		  bind:this={canvas}
		  width={canvasSize}
		  height={canvasSize}
		  on:contextmenu|preventDefault
		  on:click={handleClick}
		  on:auxclick={handleClick}
		  class="border border-gray-400 rounded-lg shadow-lg"
		></canvas>
	  </div>
	</div>
  </div>
<style>

</style>
