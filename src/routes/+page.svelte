<script lang="ts">
import { onMount } from 'svelte';

let canvas: HTMLCanvasElement;
const gridSize = 100;
const cellSize = 25;
const canvasSize = gridSize * cellSize;

let wallsV: { [key: string]: boolean} = {};
let wallsH: { [key: string]: boolean} = {};
let zumbadores: { [key: string]: number } = {};
let current_st = 0;
let ban = true;
let code = '';

let highlightedCode = '';
let textArea: HTMLTextAreaElement;
let highlightedDiv: HTMLDivElement;
let scrollContainer: HTMLDivElement;
let contadorFunciones = 0;
let contadorNumeros = 0;	
let banderaError=false;
let errores: string[] = [];
let funciones: Map<string, number> = new Map();
let numeros: Map<number, number> = new Map();
let cadenaDeTokens = "";

let matriz: number[][] = [];
let matrizCondicionales: number[][] = [];
let zumbadoresMoc=0;
let can= {
  x: 0,
  y: 99,
  dir: 0, // 0=arriba, 1=derecha, 2=abajo, 3=izquierda
  mochila: zumbadoresMoc
};

type ASTNode =
  | { tipo: 'programa', cuerpo: ASTNode[] }
  | { tipo: 'define', nombre: number, cuerpo: ASTNode[] }
  | { tipo: 'llamada_funcion', nombre: number }
  | { tipo: 'avanza' }
  | { tipo: 'gira_izquierda' }
  | { tipo: 'apagate' }
  | { tipo: 'repite', veces: number, cuerpo: ASTNode[] }
  | { tipo: 'mientras', condicion: Condicion, cuerpo: ASTNode[] }
  | { tipo: 'si', condicion: Condicion, entonces: ASTNode[], sino?: ASTNode[] };

type Condicion =
  | { tipo: 'condicion_base', nombre: number }
  | { tipo: 'not', valor: Condicion }
  | { tipo: 'and', izquierda: Condicion, derecha: Condicion }
  | { tipo: 'or', izquierda: Condicion, derecha: Condicion };

let tokens: (number)[] = [];
let tablaFunciones: { [key: number]: ASTNode } = {};
let contSin=0;
function parse(inputTokens: (number )[]): ASTNode {
  tokens = inputTokens;
  contSin = 0;
  const cuerpo = parseInstrucciones();
  return { tipo: 'programa', cuerpo };
}


function parseInstrucciones(): ASTNode[] {
  const instrucciones: ASTNode[] = [];
  while (contSin < tokens.length && tokens[contSin] !== 5005) {
    instrucciones.push(parseInstruccion());
  }
  return instrucciones;
}
/*
function parseInstruccion(): ASTNode {
  const token = tokens[contSin];

  if (token === TokenType.AVANZA) {
    i++; expect(TokenType.PUNTO_COMA);
    return { tipo: 'avanza' };
  }
  if (token === TokenType.GIRA) {
    i++; expect(TokenType.PUNTO_COMA);
    return { tipo: 'gira_izquierda' };
  }
  if (token === TokenType.APAGATE) {
    i++; expect(TokenType.PUNTO_COMA);
    return { tipo: 'apagate' };
  }
  if (token === TokenType.REPETIR) {
    i++; expect(TokenType.PARENTESIS_IZQ);
    const veces = tokens[i++] as number;
    expect(TokenType.PARENTESIS_DER);
    expect(TokenType.LLAVE_IZQ);
    const cuerpo = parseInstrucciones();
    expect(TokenType.LLAVE_DER);
    return { tipo: 'repite', veces, cuerpo };
  }
  if (token === TokenType.MIENTRAS) {
    i++; expect(TokenType.PARENTESIS_IZQ);
    const condicion = parseCondicion();
    expect(TokenType.PARENTESIS_DER);
    expect(TokenType.LLAVE_IZQ);
    const cuerpo = parseInstrucciones();
    expect(TokenType.LLAVE_DER);
    return { tipo: 'mientras', condicion, cuerpo };
  }
  if (token === TokenType.SI) {
    i++; expect(TokenType.PARENTESIS_IZQ);
    const condicion = parseCondicion();
    expect(TokenType.PARENTESIS_DER);
    expect(TokenType.LLAVE_IZQ);
    const entonces = parseInstrucciones();
    expect(TokenType.LLAVE_DER);
    let sino: ASTNode[] | undefined;
    if (tokens[i] === TokenType.SINO) {
      i++; expect(TokenType.LLAVE_IZQ);
      sino = parseInstrucciones();
      expect(TokenType.LLAVE_DER);
    }
    return { tipo: 'si', condicion, entonces, sino };
  }
  if (token === TokenType.DEFINIR) {
    i++;
    const nombre = tokens[i++] as string;
    expect(TokenType.PARENTESIS_IZQ);
    expect(TokenType.PARENTESIS_DER);
    expect(TokenType.LLAVE_IZQ);
    const cuerpo = parseInstrucciones();
    expect(TokenType.LLAVE_DER);
    tablaFunciones[nombre] = cuerpo;
    return { tipo: 'define', nombre, cuerpo };
  }
  if (typeof token === 'string') {
    i++; expect(TokenType.PUNTO_COMA);
    return { tipo: 'llamada_funcion', nombre: token };
  }
  throw new Error(`Instrucción inválida: ${token}`);
}*/

function mover() {
  const { x, y, dir } = can;
  const checkWall = () => {
    if (dir === 0) return wallsH[`${x},${y}`];
    if (dir === 1) return wallsV[`${x+1},${y}`];
    if (dir === 2) return wallsH[`${x},${y+1}`];
    if (dir === 3) return wallsV[`${x},${y}`];
  };

  if (checkWall()) {
    alert("¡No se puede mover por una pared!");
    return;
  }

  if (dir === 0 && y > 0) can.y--;
  else if (dir === 1 && x < gridSize - 1) can.x++;
  else if (dir === 2 && y < gridSize - 1) can.y++;
  else if (dir === 3 && x > 0) can.x--;
  else alert("¡No se puede mover fuera del mundo!");

  drawWorld();
}

function turnLeft() {
  can.dir = (can.dir + 3) % 4;
  drawWorld();
}

function pickBeeper() {
  const key = `${can.x},${can.y}`;
  if (zumbadores[key]) {
    zumbadores[key]--;
    if (zumbadores[key] === 0) delete zumbadores[key];
    can.mochila++;
  } else {
   alert("¡No hay zumbadores en esta posición!");
  }
  drawWorld();
}

function putBeeper() {
  if (can.mochila > 0) {
    const key = `${can.x},${can.y}`;
    zumbadores[key] = (zumbadores[key] || 0) + 1;
    can.mochila--;
  } else {
    alert("¡No tienes zumbadores en la mochila!");
  }
  drawWorld();
}

function compilar() {
  const code = textArea.value;
  const resultado = analizar(code);
  highlightedCode = resultado;
  console.log("cadenaDeTokens :",cadenaDeTokens);
  let btnEjecutar = document.getElementById("btnEjecutar");
  let txtParrafo= document.getElementById("txtError");
if (txtParrafo) {
  if (errores.length > 0) {
    txtParrafo.innerHTML = errores.join("<br>");
    txtParrafo.classList.remove("text-green-500");
    txtParrafo.classList.add("text-red-500");
  } else {
    btnEjecutar?.removeAttribute("disabled");
    txtParrafo.innerHTML = "Sin errores";
    txtParrafo.classList.remove("text-red-500");
    txtParrafo.classList.add("text-green-500");
  }
}
}
function ejecutar() {
  const acciones = [
    mover,
    mover,
    mover,
    pickBeeper,
    turnLeft,
    turnLeft,
    turnLeft,
    mover,
    putBeeper
  ];

  acciones.reduce((prom, accion) => {
    return prom.then(() => new Promise(res => {
      setTimeout(() => {
        accion();
        res();
      }, 500);
    }));
  }, Promise.resolve());
}


//Definición de funciones para leer los documento de la matriz
async function cargarMatriz() {
	const res = await fetch('src/lib/matriz.txt');
	console.log(res);
	const texto = await res.text();
	console.log(texto);
	matriz = texto.trim().split('\n').map(linea =>
		linea.split('\t').map(Number)
	);

}

async function cargarMatrizCondicionales() {
	const res = await fetch('src/lib/matriz_condicionales.txt');
	console.log(res);
	const texto = await res.text();
	console.log(texto);
	matrizCondicionales = texto.trim().split('\n').map(linea =>
		linea.split('\t').map(Number)
	);

}

//analizador lexico
function analizar(code: string): string {
	contadorFunciones = 0;
	contadorNumeros = 0;
	funciones = new Map();
	errores = [];
	//declaracion de variables
	let current_st = 0;
	let current_st_cond = 0;
	let cadena = "";
	let cadenaCond = "";
	cadenaDeTokens = "";
	let resultado = "";
	let banderaError = true;


	//declaraciones dela constante que retorna el valor del caracter
	const valorCarac = (carac: string): number => {
		const mapa: { [key: string]: number } = {
			a: 1, b: 2, c: 3, d: 4, e: 5, f: 6, g: 7, h: 8, i: 9,
			j: 10, k: 11, l: 12, m: 13, n: 14, o: 15, p: 16, q: 17, r: 18,
			s: 19, t: 20, u: 21, v: 22, w: 23, x: 24, y: 25, z: 26,
			'0': 27, '1': 28, '2': 29, '3': 30, '4': 31, '5': 32,
			'6': 33, '7': 34, '8': 35, '9': 36, '_': 37, '&': 38, '|': 39, '!': 40,
			 '{': 41, '}': 42, '(': 43, ')': 44,
			'\n': 45, ' ': 46, ';': 47
		};
		return mapa[carac] ?? -5;
	};
	
	for (let i = 0; i < code.length; i++) {
		const carac = code[i];
		const valor = valorCarac(carac);
		console.log("valor caracter:",valor);

		if (valor === -5) {
			cadenaDeTokens += "Caracter no definido en "+ carac + " ";
			errores.push("Caracter no definido en "+ carac + " ");
			resultado += `<span class="text-red-500">${carac}</span>`;
			banderaError = false;
			continue; // Ignorar caracteres no válidos
		}
		console.log("caracter:",valor);
		// Acumular la cadena
		if (current_st === 0) 
			cadena = "";
		if (current_st_cond === 0)
			cadenaCond = "";

		
		cadena += carac;
		cadenaCond += carac;

		// Cambio de estado
		current_st = matriz[current_st + 1][valor];
		console.log("condicional currentst:",current_st_cond+1);
		current_st_cond = matrizCondicionales[current_st_cond + 1][valor];
		
		console.log("normal:",current_st);
		console.log("condicional:",current_st_cond);
		// Error
		if ((current_st === -10 || current_st === -1 || current_st === 90)&&current_st_cond=== -1) {
			resultado += `<span class="text-red-500">${cadena}</span>`;
			if(!(cadena===" " || cadenaCond===" " || cadenaCond==="\n" || cadena==="\n")){
				banderaError = false;
				cadenaDeTokens += "Hay un error en "+ cadena + " ";
				errores.push("Hay un error en "+ cadena + " ");
			}
			current_st = 0;
			current_st_cond = 0;
			cadena = "";
			cadenaCond = "";
			continue;
		}else if(current_st === -10 || current_st === -1 || current_st === 90){
			current_st = 0;
			cadena = "";
		}else if(current_st_cond === -1){
			current_st_cond = 0;
			cadenaCond = "";
		}
		
		
		// Estado final
		if (current_st > 999 || current_st_cond > 999) {
			let clase = "text-white";
			if (current_st >= 1000 && current_st < 2000){ clase = "text-yellow-400"; // sentencia de control
			}
			else if (current_st_cond >= 2000 && current_st_cond < 3000){ clase = "text-blue-400"; // condiciones
				console.log(cadenaCond.substring(0, cadena.length - 1));
				resultado += `<span class="${clase}">${cadenaCond.substring(0,cadenaCond.length-1)}</span>`;
			 	console.log("cadenaCond",cadenaCond);
			}
			else if (current_st >= 3000 && current_st < 4000) clase = "text-green-400"; // acciones
			else if (current_st >= 4000 && current_st < 5000){
				let number=parseInt(cadena);
				clase = "text-cyan-400"; // numero enteros
				if(!numeros.hasOwnProperty(number)){
				 contadorNumeros++;
				 numeros.set(number, contadorNumeros+4000);
				 
			   }
			   if(banderaError)
			   		cadenaDeTokens +=numeros.get(cadena)+" ";
			} 
			else if (current_st >= 5000 && current_st < 6000) clase = "text-pink-400"; // puntuacion
			else if (current_st >= 6000 && current_st < 7000){
			   clase = "text-purple-400";
			   if(!funciones.hasOwnProperty(cadena)){
				 contadorFunciones++;
				 funciones.set(cadena, contadorFunciones+7000);
				 
			   }
			   if(banderaError)
			   		cadenaDeTokens +=funciones.get(cadena)+" ";
			} // definicion de funcioenes
			if(current_st_cond>=2000 && current_st_cond<3000)
				i--;
			else if (!(current_st>=5000&&current_st<=5005)&&current_st!=5007 && !(current_st_cond>=2000 && current_st_cond<3000)){
				console.log(cadena.substring(0, cadena.length - 1));
				resultado += `<span class="${clase}">${cadena.substring(0,cadena.length-1)}</span>`;
				i--;
			}else if(current_st==5007 ||  (current_st>=5000&&current_st<=5005)) {
				console.log(cadena);
				resultado += `<span class="${clase}">${cadena}</span>`;
			}

			console.log("current_st",current_st);
			console.log("current_st_cond",current_st_cond);
			console.log("banderaError",banderaError);
	        if(current_st>999 && current_st!==7000 && current_st!==4000)
				cadenaDeTokens += current_st+" ";
			else if (current_st_cond!==-1 && banderaError)
				cadenaDeTokens += current_st_cond+" ";
			console.log("-----cadenaDeTokens:",cadenaDeTokens);
			current_st = 0;
			cadena = "";
			current_st_cond = 0;
			cadenaCond = "";
			
			
		}
		
	}
	

	return resultado;
}


  $: highlightedCode = analizar(code);

  function syncScroll() {
    highlightedDiv.scrollTop = textArea.scrollTop;
    highlightedDiv.scrollLeft = textArea.scrollLeft;
  }
//funcion para dibujar el mundo
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

  // Dibujar a Karel
	const cx = can.x * cellSize + cellSize / 2;
	const cy = can.y * cellSize + cellSize / 2;

	ctx.fillStyle = 'red';
	ctx.beginPath();
	const size = cellSize * 0.4;
	if (can.dir === 0) {
	ctx.moveTo(cx, cy - size);
	ctx.lineTo(cx - size, cy + size);
	ctx.lineTo(cx + size, cy + size);
	} else if (can.dir === 1) {
	ctx.moveTo(cx + size, cy);
	ctx.lineTo(cx - size, cy - size);
	ctx.lineTo(cx - size, cy + size);
	} else if (can.dir === 2) {
	ctx.moveTo(cx, cy + size);
	ctx.lineTo(cx - size, cy - size);
	ctx.lineTo(cx + size, cy - size);
	} else if (can.dir === 3) {
	ctx.moveTo(cx - size, cy);
	ctx.lineTo(cx + size, cy - size);
	ctx.lineTo(cx + size, cy + size);
	}
	ctx.closePath();
	ctx.fill();
}
function inicio(){
	  // Reiniciar el estado de Karel
  can.x = 0;
  can.y = 99;
  can.dir = 0;
  can.mochila = zumbadoresMoc;
  drawWorld();
}
function limpiarMundo(){
	// Limpiar el mundo
	wallsV = {};
	wallsH = {};
	zumbadores = {};
	can.x = 0;
	can.y = 99;
	can.dir = 0;
	can.mochila = zumbadoresMoc;
	let btnEjecutar = document.getElementById("btnEjecutar");
	let txtParrafo= document.getElementById("txtError");
	if (txtParrafo) {
		txtParrafo.innerHTML = "Sin errores";
		txtParrafo.classList.remove("text-red-500");
		txtParrafo.classList.add("text-green-500");
	}
	btnEjecutar?.setAttribute("disabled", "true");

	// Limpiar el canvas
	drawWorld();
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
  cargarMatriz();
  cargarMatrizCondicionales();
  console.log("matriz",matriz);
  console.log("matrizCondicionales",matrizCondicionales);
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
		<button class="bg-purple-600 hover:bg-purple-700 px-4 py-2 rounded-xl text-sm shadow" on:click={compilar}>
		  Compilar
		</button>
		<button class="bg-indigo-600 hover:bg-indigo-700 px-4 py-2 rounded-xl text-sm shadow" disabled id="btnEjecutar" on:click={ejecutar}>
		  Ejecutar
		</button>
		<button class="bg-fuchsia-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={inicio}>
			Reiniciar
		</button>
		<button class="bg-yellow-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow">
		  Detener
		</button>
		<button class="bg-slate-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={limpiarMundo}>
			Limpiar mundo
		</button>
	  </div>
	  <div>
		<input
		  type="number"
		  min="0"
		  placeholder="Zumbadores"
		  class="rounded-xl px-3 py-1 text-black text-sm w-32"
		  bind:value={zumbadoresMoc}
		/>
	  </div>
	</div>
  
	<!-- Cuerpo -->
	<div class="flex flex-1 overflow-hidden">
	  <!-- Barra lateral izquierda -->

	  

	  <div class="w-1/4 bg-gray-900 text-white flex flex-col border-r border-gray-700">
		<div class="relative w-full h-3/4 bg-gray-900 rounded-md font-mono text-sm">
			<!-- Div resaltado -->
			<div
			class="absolute top-0 left-0 right-0 bottom-0 p-2 box-border pointer-events-none whitespace-pre font-mono text-sm leading-relaxed overflow-auto"
			bind:this={highlightedDiv}
			>
			<div>{@html highlightedCode}</div>
			</div>

			<!-- Textarea -->
			<textarea
			class="absolute top-0 left-0 right-0 bottom-0 p-2 box-border bg-transparent text-white caret-white selection:bg-indigo-500 w-full h-full resize-none overflow-auto font-mono text-sm leading-relaxed whitespace-pre"
			bind:value={code}
			bind:this={textArea}
			on:scroll={syncScroll}
			placeholder="Escribe tu código aquí..."
			/>
		</div>
  
		<!-- Mensajes de error -->
		<div class="h-32 overflow-y-auto p-2 text-red-400 text-xs border-t border-gray-700 bg-gray-950">
		  <p class="italic text-gray-500" id="txtError">Sin errores</p>
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
