<script lang="ts">
    interface Instruction {
      name: string;
      syntax: string;
      description: string;
    }
  
    const instrucciones: Instruction[] = [
      { name: "avanza", syntax: "avanza();", description: "Mueve al robot una celda hacia adelante." },
      { name: "giraIzquierda", syntax: "giraIzquierda();", description: "Gira al robot 90° a la izquierda." },
      { name: "giraDerecha", syntax: "giraDerecha();", description: "Gira al robot 90° a la derecha." },
      { name: "recogeZumbador", syntax: "recogeZumbador();", description: "Recoge un zumbador de la celda actual." },
      { name: "dejaZumbador", syntax: "dejaZumbador();", description: "Deja un zumbador en la celda actual." },
      { name: "apagate", syntax: "apagate();", description: "Finaliza la ejecución del robot." },
      { name: "repetir", syntax: "repetir (n) { ... }", description: "Repite las instrucciones n veces." },
      { name: "mientras", syntax: "mientras (condición) { ... }", description: "Ejecuta el bloque mientras la condición sea verdadera." },
      { name: "si", syntax: "si (condición) { ... }", description: "Ejecuta el bloque solo si la condición se cumple." },
      { name: "sino", syntax: "sino { ... }", description: "Ejecuta el bloque alternativo si la condición no se cumple." },
      { name: "función", syntax: "funcion nombre() { ... }", description: "Declara una nueva función personalizada que luego puede ser llamada." },
      { name: "llamada a función", syntax: "nombre();", description: "Ejecuta la función previamente definida con ese nombre." },
    ];
  
    const condiciones: Instruction[] = [
      { name: "frenteLibre", syntax: "frenteLibre", description: "Verdadero si no hay una pared frente al robot." },
      { name: "frenteBloqueado", syntax: "frenteBloqueado", description: "Verdadero si hay una pared frente al robot." },
      { name: "juntoAZumbador", syntax: "juntoAZumbador", description: "Verdadero si hay un zumbador en la celda actual." },
      { name: "noJuntoAZumbador", syntax: "noJuntoAZumbador", description: "Verdadero si no hay un zumbador en la celda actual." },
      { name: "hayZumbadoresEnLaMochila", syntax: "hayZumbadoresEnLaMochila", description: "Verdadero si el robot tiene al menos un zumbador en su mochila." },
      { name: "noHayZumbadoresEnLaMochila", syntax: "noHayZumbadoresEnLaMochila", description: "Verdadero si el robot no tiene zumbadores en su mochila." },
    ];
  
    const operadores: Instruction[] = [
      { name: "AND lógico", syntax: "cond1 && cond2", description: "Ambas condiciones deben ser verdaderas para que el resultado sea verdadero." },
      { name: "OR lógico", syntax: "cond1 || cond2", description: "Al menos una condición debe ser verdadera para que el resultado sea verdadero." },
    ];
  </script>
  
  <section class="min-h-screen bg-gray-950 text-gray-100 px-6 py-12">
    <div class="max-w-6xl mx-auto">
      <h1 class="text-4xl font-bold text-purple-400 mb-10">📘 Instrucciones del Lenguaje CAN</h1>
  
      <h2 class="text-2xl text-purple-300 font-semibold mb-4">🧠 Instrucciones</h2>
      <div class="grid gap-6 md:grid-cols-2">
        {#each instrucciones as instr}
          <div class="bg-gray-800 border border-gray-700 rounded-2xl p-5">
            <h3 class="text-xl font-semibold text-purple-200">{instr.name}</h3>
            <p class="text-green-300 font-mono text-sm mt-1 mb-2">{instr.syntax}</p>
            <p class="text-gray-300 text-sm">{instr.description}</p>
          </div>
        {/each}
      </div>
  
      <h2 class="text-2xl text-purple-300 font-semibold mt-12 mb-4">⚙️ Condiciones</h2>
      <div class="grid gap-6 md:grid-cols-2">
        {#each condiciones as cond}
          <div class="bg-gray-800 border border-gray-700 rounded-2xl p-5">
            <h3 class="text-xl font-semibold text-purple-200">{cond.name}</h3>
            <p class="text-yellow-300 font-mono text-sm mt-1 mb-2">{cond.syntax}</p>
            <p class="text-gray-300 text-sm">{cond.description}</p>
          </div>
        {/each}
      </div>
  
      <h2 class="text-2xl text-purple-300 font-semibold mt-12 mb-4">🔗 Operadores Lógicos</h2>
      <div class="grid gap-6 md:grid-cols-2">
        {#each operadores as op}
          <div class="bg-gray-800 border border-gray-700 rounded-2xl p-5">
            <h3 class="text-xl font-semibold text-purple-200">{op.name}</h3>
            <p class="text-pink-300 font-mono text-sm mt-1 mb-2">{op.syntax}</p>
            <p class="text-gray-300 text-sm">{op.description}</p>
          </div>
        {/each}
      </div>
  
      <footer class="mt-16 text-center text-sm text-gray-500">
        Sistema CAN — Documentación para usuarios y programadores
      </footer>
    </div>
  </section>
  