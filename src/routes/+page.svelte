<script lang="ts">
import { onMount } from 'svelte';
import Modal from './Modal.svelte';
let showModal = false;

let canvas: HTMLCanvasElement;
const gridSize = 100;
const cellSize = 25;
const canvasSize = gridSize * cellSize;

let wallsV: { [key: string]: boolean} = {};
let wallsH: { [key: string]: boolean} = {};
let zumbadores: { [key: string]: number } = {};
// Variables globales adicionales
let tablaSimbolos: Array<{
  nombre: string;
  tipo: string;
  linea: number;
  ambito: string;
  valor?: any;
}> = [];

let pilaErrores: Array<{
  codigo: number;
  linea: number;
  caracter?: string;
}> = [];
let code = '';

let highlightedCode = '';
let textArea: HTMLTextAreaElement;
let highlightedDiv: HTMLDivElement;
let scrollContainer: HTMLDivElement;
let contadorFunciones = 0;
let contadorNumeros = 0;	
let funciones: Map<string, number> = new Map();
let numeros: Map<number, number> = new Map();
let errores: Map<number, string> = new Map();
errores.set(-1, "No es un palabra reservada");
errores.set(-5, "Caracter no válido");
errores.set(90, "No es un número entero");
errores.set(-10, "Nombre de función no válido");

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

// Definición de tipos de tokens
type Token = number;

type ASTNode =
  | { tipo: 'programa', cuerpo: ASTNode[]; funciones: ASTNode[] }
  | { tipo: 'define', nombre: number, cuerpo: ASTNode[] }
  | { tipo: 'llamada_funcion', nombre: number }
  | { tipo: 'avanza' }
  | { tipo: 'gira' }
  | {tipo: 'toma'}
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

let tokens: (number)[] = [];
let tablaFunciones: { [key: number]: ASTNode[] } = {};
let contSin=0;
function expect(token: number) {
  if (tokens[contSin] !== token) {
    throw new Error(`Se esperaba '${token}' pero se encontró '${tokens[contSin]}'`);
  }
  contSin++;
}
function parse(inputTokens: (number )[]): ASTNode {
	contSin = 0;
  const funciones: ASTNode[] = [];
  let inicioEncontrado = false;
  let cuerpoInicio: ASTNode[] = [];

  while (contSin < tokens.length) {
    const tokenActual = tokens[contSin];

    if (tokenActual === 1000) { // token de 'inicio'
      if (inicioEncontrado) {
        throw new Error("Error: ya se definió 'inicio()'. Solo se permite una.");
      }
      contSin++;
      expect(5000); // consumir '('
	  console.log("token actual (");
	  expect(5001); // consumir ')'
	  console.log("token actual )");
	  expect(5004); // consumir '{'
	  console.log("token actual {");

      const instrucciones: ASTNode[] = [];
      while (tokens[contSin] !== 5005) { // token de '}'
        instrucciones.push(parseInstruccion());
      }
      expect(5005); // consumir '}'
      cuerpoInicio = instrucciones;
      inicioEncontrado = true;

    } else if (tokenActual === 1009) { // token de 'define'
      funciones.push(parseDefine());

    } else {
      throw new Error("Error: solo se permiten definiciones de funciones o 'inicio()' a nivel superior.");
    }
  }

  if (!inicioEncontrado) {
    throw new Error("Error: no se encontró la función 'inicio()'.");
  }

  return {
    tipo: 'programa',
    cuerpo: cuerpoInicio,
    funciones
  };
}
function parseDefine(): ASTNode {
  contSin++;
	let nombre=0;
    if ( tokens[contSin] >=6000 && tokens[contSin] < 7000) {
		nombre = tokens[contSin];
	}else{
		throw new Error(`Se esperaba un nombre de funcion pero se encontró '${tokens[contSin]}'`);
	}
	contSin++;
    expect(5000);
    expect(5001);
    expect(5004);
    const cuerpo = parseInstrucciones();
    expect(5005);
    tablaFunciones[nombre] = cuerpo;
    return { tipo: 'define', nombre, cuerpo };
}
function parseInstrucciones(): ASTNode[] {
  const instrucciones: ASTNode[] = [];
  while (contSin < tokens.length && tokens[contSin] !== 5005) {
    instrucciones.push(parseInstruccion());
  }
  return instrucciones;
}

function parseInstruccion(): ASTNode {
  const token = tokens[contSin];


  //funciones de acccion
  if (token === 3000) {
    contSin++; expect(5000);
	expect(5001);
	expect(5002);
    return { tipo: 'avanza' };
  }
  if (token === 3001) {
    contSin++; expect(5000);
	expect(5001);
	expect(5002);
    return { tipo: 'gira' };
  }
  if (token === 3002) {
    contSin++; expect(5000);
	expect(5001);
	expect(5002);
    return { tipo: 'toma' };
  }
  if (token === 3003) {
    contSin++; expect(5000);
	expect(5001);
	expect(5002);
    return { tipo: 'deja' };
  }
  
  if (token === 3004) {
    contSin++; expect(5000);
	expect(5001);
	expect(5002);
    return { tipo: 'apagate' };
  }


  //funciones de control

  //token repetir
  if (token === 1007) {
    contSin++; expect(5000);
	let veces;
    if ( tokens[contSin] >=4000 && tokens[contSin] < 5000) {
		veces = tokens[contSin];
	}else{
		throw new Error(`Se esperaba an number pero se encontró '${tokens[contSin]}'`);
	}
	contSin++;
    expect(5001);
    expect(5004);
    const cuerpo = parseInstrucciones();
    expect(5005);
    return { tipo: 'repite', veces, cuerpo };
  }

  //token mientras
  if (token === 1005) {
    contSin++; expect(5000);
    const condicion = parseCondicion();
    expect(5001);
    expect(5004);
    const cuerpo = parseInstrucciones();
    expect(5005);
    return { tipo: 'mientras', condicion, cuerpo };
  }

  //token si
  if (token === 1002) {
    contSin++; expect(5000);
    const condicion = parseCondicion();
    expect(5001);
    expect(5004);
    const entonces = parseInstrucciones();
    expect(5005);
    let sino: ASTNode[] | undefined;
    if (tokens[contSin] === 1004) {
      contSin++; expect(5004);
      sino = parseInstrucciones();
      expect(5005);
    }
    return { tipo: 'si', condicion, entonces, sino };
  }

  //token define
  if (token === 1009) {
    throw new Error("Error: no se permite definir funciones dentro de otras funciones.");
  }
  if (token>=6000 && token < 7000) {
    contSin++; expect(5000);
	expect(5001);
	expect(5002);
    return { tipo: 'llamada_funcion', nombre: token };
  }
  throw new Error(`Instrucción inválida: ${token}`);

  function parseCondicion(): Condicion {
  return parseOr();
 }

function parseOr(): Condicion {
  let izquierda = parseAnd();
  while (tokens[contSin] === 5006) {
    contSin++;
    const derecha = parseAnd();
    izquierda = { tipo: 'or', izquierda, derecha };
  }
  return izquierda;
}

function parseAnd(): Condicion {
  let izquierda = parseNot();
  while (tokens[contSin] === 5008) {
    contSin++;
    const derecha = parseNot();
    izquierda = { tipo: 'and', izquierda, derecha };
  }
  return izquierda;
}

function parseNot(): Condicion {
  if (tokens[contSin] === 5007) {
    contSin++;
    const valor = parseNot();
    return { tipo: 'not', valor };
  }
  return parseCondicionBase();
}

function parseCondicionBase(): Condicion {
  if (tokens[contSin] === 5000) {
    contSin++;
    const cond = parseCondicion();
    expect(5001);
    return cond;
  }
  let nombre;
  if (tokens[contSin] >= 2000 && tokens[contSin] < 3000) {
	nombre = tokens[contSin];
  } else {
	throw new Error(`Se esperaba una condicion pero se encontró '${tokens[contSin]}'`);
  }
  return { tipo: 'condicion_base', nombre };
}


}

function imprimirAST(nodo: ASTNode, nivel: number = 0): void {
  const indent = '  '.repeat(nivel);

  switch (nodo.tipo) {
    case 'programa':
      console.log(`${indent}<programa>`);

      // Tratamos cuerpo como inicio
      console.log(`${indent}  <inicio>`);
      nodo.cuerpo.forEach(instr => imprimirAST(instr, nivel + 2));
      console.log(`${indent}  </inicio>`);

      // Luego imprimimos las funciones definidas
      nodo.funciones.forEach(fn => imprimirAST(fn, nivel + 1));

      console.log(`${indent}</programa>`);
      break;

    case 'define':
      console.log(`${indent}<define nombre="${nodo.nombre}">`);
      nodo.cuerpo.forEach(instr => imprimirAST(instr, nivel + 1));
      console.log(`${indent}</define>`);
      break;

    case 'llamada_funcion':
      console.log(`${indent}<llamada_funcion nombre="${nodo.nombre}" />`);
      break;

    case 'avanza':
    case 'gira':
    case 'toma':
    case 'deja':
    case 'apagate':
      console.log(`${indent}<${nodo.tipo} />`);
      break;

    case 'repite':
      console.log(`${indent}<repite veces="${nodo.veces}">`);
      nodo.cuerpo.forEach(instr => imprimirAST(instr, nivel + 1));
      console.log(`${indent}</repite>`);
      break;

    case 'mientras':
      console.log(`${indent}<mientras condicion="${stringCondicion(nodo.condicion)}">`);
      nodo.cuerpo.forEach(instr => imprimirAST(instr, nivel + 1));
      console.log(`${indent}</mientras>`);
      break;

    case 'si':
      console.log(`${indent}<si condicion="${stringCondicion(nodo.condicion)}">`);
      nodo.entonces.forEach(instr => imprimirAST(instr, nivel + 1));
      if (nodo.sino) {
        console.log(`${indent}  <sino>`);
        nodo.sino.forEach(instr => imprimirAST(instr, nivel + 2));
        console.log(`${indent}  </sino>`);
      }
      console.log(`${indent}</si>`);
      break;

    default:
      console.log(`${indent}<nodo tipo="${(nodo as any).tipo}" />`);
  }
}

function stringCondicion(cond: Condicion): string {
  switch (cond.tipo) {
    case 'condicion_base':
      return `cond(${cond.nombre})`;
    case 'not':
      return `not(${stringCondicion(cond.valor)})`;
    case 'and':
      return `(${stringCondicion(cond.izquierda)} and ${stringCondicion(cond.derecha)})`;
    case 'or':
      return `(${stringCondicion(cond.izquierda)} or ${stringCondicion(cond.derecha)})`;
    default:
      return 'condDesconocida';
  }
}


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
	let btnEjecutar = document.getElementById("btnEjecutar");
	let txtParrafo= document.getElementById("txtError");
	console.log("resultado:");
	for(let i=0; i<tablaSimbolos.length; i++){
		const simbolo=tablaSimbolos[i];
		if(simbolo.tipo==="constante"){
			console.log(simbolo.valor,`Constante ${simbolo.nombre} en la linea ${simbolo.linea}`);
		}else if(simbolo.tipo==="funcion"){
			console.log(simbolo.nombre,`Funcion ${simbolo.nombre} en la linea ${simbolo.linea}`);
		}
	}
	if (txtParrafo) {
		txtParrafo.innerHTML = "";
		if (pilaErrores.length > 0) {
			for (let i = 0; i < pilaErrores.length; i++) {
				const error = pilaErrores[i];
				txtParrafo.innerHTML += `Error ${error.codigo} en la línea ${error.linea}: ${error.caracter} descripcion: ${errores.get(error.codigo)}<br>`;
			}
			txtParrafo.classList.remove("text-green-500");
			txtParrafo.classList.add("text-red-500");
		} else {
			tokens = cadenaDeTokens.trim().split(' ').map(Number);
			console.log("tokens para realizar analices sintactico",tokens);
			/*const ast = parse(tokens);
			console.log("AST:", ast);
			imprimirAST(ast);*/
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
	//console.log(res);
	const texto = await res.text();
	//console.log(texto);
	matriz = texto.trim().split('\n').map(linea =>
		linea.split('\t').map(Number)
	);

}

async function cargarMatrizCondicionales() {
	const res = await fetch('src/lib/matriz_condicionales.txt');
	//console.log(res);
	const texto = await res.text();
	//console.log(texto);
	matrizCondicionales = texto.trim().split('\n').map(linea =>
		linea.split('\t').map(Number)
	);

}

//analizador lexico
function analizar(code: string): string {
	contadorFunciones = 0;
	contadorNumeros = 0;
	funciones = new Map();
	pilaErrores = [];
	tablaSimbolos = [];
	//declaracion de variables
	let current_st = 0;
	let current_st_cond = 0;
	let numeroLinea=1;
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
		//console.log("valor caracter:",valor);


		if(carac=='\n')
			numeroLinea++;
		if (valor === -5) {
			cadena += carac;
			resultado += `<span class="text-red-500">${cadena}</span>`;
			const error={
				codigo: -5,
				linea: numeroLinea,
				caracter:carac
			}
			pilaErrores.push(error);
			cadena = "";
			cadenaCond = "";
			continue; // Ignorar caracteres no válidos
		}
		//console.log("caracter:",valor);
		// Acumular la cadena
		if (current_st === 0) 
			cadena = "";
		if (current_st_cond === 0)
			cadenaCond = "";

		
		cadena += carac;
		cadenaCond += carac;

		// Cambio de estado
		current_st = matriz[current_st + 1][valor];
		//console.log("condicional currentst:",current_st_cond+1);
		current_st_cond = matrizCondicionales[current_st_cond + 1][valor];
		
		//console.log("normal:",current_st);
		//console.log("condicional:",current_st_cond);
		// Error
		if(current_st==0){
			resultado += `<span >${carac}</span>`;
		}
		if ((current_st === -10 || current_st === -1 || current_st === 90)&&current_st_cond<=0) {
			//console.log("current_st:",current_st);
	     	resultado += `<span class="text-red-500">${cadena}</span>`;
			
				//resultado += `<span class="text-red-500">${cadena}</span>`;
				const error={
					codigo: current_st,
					linea: numeroLinea,
					caracter:cadena
				}
				pilaErrores.push(error);
			
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
			else if (current_st_cond >= 2000 && current_st_cond < 3000){ 
				clase = "text-blue-400"; // condiciones
				//console.log(cadenaCond.substring(0, cadena.length - 1));
				resultado += `<span class="${clase}">${cadenaCond.substring(0,cadenaCond.length-1)}</span>`;
			 	//console.log("cadenaCond",cadenaCond);
			}
			else if (current_st >= 3000 && current_st < 4000) clase = "text-green-400"; // acciones
			else if (current_st >= 4000 && current_st < 5000){
				let number=parseInt(cadena);
				clase = "text-cyan-400"; // numero enteros
				if(numeros.get(number)===undefined){
				 contadorNumeros++;
				 
				 numeros.set(number, contadorNumeros+4000);	   
				}
				//console.log("cadena numeros: ",cadena,numeros);
				tablaSimbolos.push({
					nombre: `const_${number}`, // Ej: "const_3"
					tipo: "constante",
					valor: number,
					linea: numeroLinea,
					ambito: "local"
				});
			   	cadenaDeTokens +=numeros.get(number)+" ";
			} 
			else if (current_st >= 5000 && current_st < 6000) clase = "text-pink-400"; // puntuacion
			else if (current_st >= 6000 && current_st < 7000){
			   clase = "text-purple-400";
			   let funcion= cadena.substring(0, cadena.length - 1);
			   if(funciones.get(funcion)===undefined){
				 contadorFunciones++;
				 funciones.set(funcion, contadorFunciones+6000);
				 
			   }
			   tablaSimbolos.push({
					nombre: `${funcion}`, // Ej: "const_3"
					tipo: "funcion",
					linea: numeroLinea,
					ambito: "unkonown",
				});
			  cadenaDeTokens +=funciones.get(funcion)+" ";
			} // definicion de funcioenes
			if(current_st_cond>=2000 && current_st_cond<3000)
				i--;
			else if (!(current_st>=5000&&current_st<=5005)&&current_st!=5007 && !(current_st_cond>=2000 && current_st_cond<3000)){
				//console.log(cadena.substring(0, cadena.length - 1));
				resultado += `<span class="${clase}">${cadena.substring(0,cadena.length-1)}</span>`;
				i--;
			}else if(current_st==5007 ||  (current_st>=5000&&current_st<=5005)) {
				//console.log(cadena);
				resultado += `<span class="${clase}">${cadena}</span>`;
			}


			
	        if(current_st>999 && current_st!==7000 && current_st!==4000 && current_st!=6000) 
				cadenaDeTokens += current_st+" ";
			else if (current_st_cond>0 )
				cadenaDeTokens += current_st_cond+" ";
			//console.log("-----cadenaDeTokens:",cadenaDeTokens);
			current_st = 0;
			cadena = "";
			current_st_cond = 0;
			cadenaCond = "";
			
			
		}
		
	}
	if(cadena!==""){
		resultado += `<span class="text-red-500">${cadena}</span>`;
		const error={
			codigo: -1,
			linea: numeroLinea,
			caracter:cadena
		}
		pilaErrores.push(error);
		cadena = "";
		cadenaCond = "";
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

function subirCodigo() {
	const fileInput = document.createElement('input');
	fileInput.type = 'file';
	fileInput.accept = '.txt';

	fileInput.onchange = async (event) => {
		const file = (event.target as HTMLInputElement).files?.[0];
		if (!file) {
			alert("No se seleccionó ningún archivo.");
			return;
		}
		if (!file.name.endsWith('.txt')) {
			alert("Solo se permiten archivos con extensión .txt");
			return;
		}
		if (file.size === 0) {
			alert("El archivo está vacío.");
			return;
		}
		const reader = new FileReader();
		reader.onload = (e) => {
			const contenido = e.target?.result as string;
			if (!contenido.trim()) {
				alert("El archivo no contiene código válido.");
				return;
			}
			code = contenido;
		};
		reader.onerror = () => {
			alert("Ocurrió un error al leer el archivo.");
		};
		reader.readAsText(file);
	};
	fileInput.click();
}

function descargarCodigo() {
	if (!code.trim()) {
		alert("No hay código para descargar.");
		return;
	}
	const blob = new Blob([code], { type: 'text/plain' });
	const url = URL.createObjectURL(blob);
	const a = document.createElement('a');
	a.href = url;
	a.download = 'codigo.txt';
	a.click();
	URL.revokeObjectURL(url);
}

function ver(){
	showModal = true;
}
function cerrar(){
	showModal = false;
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
let selectedOption: string = "";

const analysisOptions = [
    { value: "", label: "Seleccione una opción", disabled: true },
	{ value: "tokens", label: "Cadena de tokens" },
    { value: "tabla de simbolos", label: "Tabla de símbolos" },
    { value: "árbol sintáctico", label: "Árbol de análisis sintáctico" },
    { value: "árbol semantico", label: "Árbol semántico" }
  ];
</script>

<svelte:head>
	<title>CAN</title>
	<meta name="description" content="if you imagite, you can do" />
</svelte:head>

<div class="flex flex-col h-115 w-screen

bg-gray-900" >
	<!-- Barra superior -->
	<div class="bg-gray-800 text-white flex items-center justify-between px-4 py-2 shadow-md">
	  <div class="flex gap-2">
		
			
		<div class="relative group inline-block">
			<button
			  class="bg-purple-600 hover:bg-purple-700 transition px-4 py-2 rounded-xl text-sm shadow flex items-center gap-2"
			  on:click={compilar}
			>
			<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
				<path stroke-linecap="round" stroke-linejoin="round" d="M10.343 3.94c.09-.542.56-.94 1.11-.94h1.093c.55 0 1.02.398 1.11.94l.149.894c.07.424.384.764.78.93.398.164.855.142 1.205-.108l.737-.527a1.125 1.125 0 0 1 1.45.12l.773.774c.39.389.44 1.002.12 1.45l-.527.737c-.25.35-.272.806-.107 1.204.165.397.505.71.93.78l.893.15c.543.09.94.559.94 1.109v1.094c0 .55-.397 1.02-.94 1.11l-.894.149c-.424.07-.764.383-.929.78-.165.398-.143.854.107 1.204l.527.738c.32.447.269 1.06-.12 1.45l-.774.773a1.125 1.125 0 0 1-1.449.12l-.738-.527c-.35-.25-.806-.272-1.203-.107-.398.165-.71.505-.781.929l-.149.894c-.09.542-.56.94-1.11.94h-1.094c-.55 0-1.019-.398-1.11-.94l-.148-.894c-.071-.424-.384-.764-.781-.93-.398-.164-.854-.142-1.204.108l-.738.527c-.447.32-1.06.269-1.45-.12l-.773-.774a1.125 1.125 0 0 1-.12-1.45l.527-.737c.25-.35.272-.806.108-1.204-.165-.397-.506-.71-.93-.78l-.894-.15c-.542-.09-.94-.56-.94-1.109v-1.094c0-.55.398-1.02.94-1.11l.894-.149c.424-.07.765-.383.93-.78.165-.398.143-.854-.108-1.204l-.526-.738a1.125 1.125 0 0 1 .12-1.45l.773-.773a1.125 1.125 0 0 1 1.45-.12l.737.527c.35.25.807.272 1.204.107.397-.165.71-.505.78-.929l.15-.894Z" />
				<path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z" />
			  </svg>			  
			</button>
		  
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Compilar"}
			</div>
		</div>
		<div class="relative group inline-block">
			<button class="bg-indigo-600 hover:bg-indigo-700 px-4 py-2 rounded-xl text-sm shadow flex items-center gap-2" disabled id="btnEjecutar" on:click={ejecutar}>
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="M5.25 5.653c0-.856.917-1.398 1.667-.986l11.54 6.347a1.125 1.125 0 0 1 0 1.972l-11.54 6.347a1.125 1.125 0 0 1-1.667-.986V5.653Z" />
				  </svg>
				  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Ejecutar"}
			</div>
		</div>
		<div class="relative group inline-block">
			<button class="bg-fuchsia-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={inicio}>
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
				  </svg>				  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Reiniciar"}
			</div>
		</div>
		<div class="relative group inline-block"> 
			<button class="bg-yellow-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow">
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="M15.75 5.25v13.5m-7.5-13.5v13.5" />
				  </svg>				  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Detener"}
			</div>
		</div>
		<div class="relative group inline-block">
			<button class="bg-slate-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={limpiarMundo}>
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0" />
				  </svg>				  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Limpiar mundo"}
			</div>
		</div>
		<div class="relative group inline-block">
			<button class="bg-slate-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={descargarCodigo}>
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3" />
				  </svg>				  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Descargar codigo"}
			</div>
		</div>
		<div class="relative group inline-block">
			<button class="bg-slate-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={subirCodigo}>
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5m-13.5-9L12 3m0 0 4.5 4.5M12 3v13.5" />
				</svg>			  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Subir codigo"}
			</div>
		</div>
		<!-- Select -->
		<select
			id="analysis-select"
			bind:value={selectedOption}
			class="block w-full rounded-md border text-gray-900 border-gray-300 py-2 pl-3 pr-10 text-base focus:border-indigo-500 focus:outline-none focus:ring-indigo-500"
		>
			{#each analysisOptions as option}
			<option 
				value={option.value} 
				disabled={option.disabled}
			>
				{option.label}
			</option>
			{/each}
		</select>
		<div class="relative group inline-block">
			<button class="bg-slate-600 hover:bg-red-700 px-4 py-2 rounded-xl text-sm shadow" on:click={ver}>
				<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
					<path stroke-linecap="round" stroke-linejoin="round" d="M2.036 12.322a1.012 1.012 0 0 1 0-.639C3.423 7.51 7.36 4.5 12 4.5c4.638 0 8.573 3.007 9.963 7.178.07.207.07.431 0 .639C20.577 16.49 16.64 19.5 12 19.5c-4.638 0-8.573-3.007-9.963-7.178Z" />
					<path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z" />
				  </svg>	  
			</button>
			<div
			  class="absolute up-full left-1/2 -translate-x-1/2 mb-2
					 hidden group-hover:flex px-3 py-1 bg-gray-800 text-white text-xs 
					 rounded shadow-lg z-10 whitespace-nowrap"
			>
			  {"Ver "+selectedOption}
			</div>	
		</div>
	  </div>
	  <Modal  tokens={tokens} tablaSimbolos={tablaSimbolos} opcion={selectedOption} open={showModal} onClose={cerrar} />
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
