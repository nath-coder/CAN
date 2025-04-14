<script lang="ts">
	export let creators = [
		{
			name: 'Diana Nathasha Macías Sevilla',
			role: 'Desarrolladora de la interfaz del autómata, encargada de la parte visual',
			image: '/images/imagen.jpg',
			description: 'Estudiante de la carrera de Ingenieria en Sistemas Computacionales, le apasiona la programacion y el desarrollo de software.'
		},
		{
			name: 'Crisitian Eduardo ',
			role: 'Encargado del análisis estático, diseño y programación del autómata',
			image: '/images/imagen.jpg',
			description: 'Diseñó y programó la parte visual interactiva del autómata.'
		},
		{
			name: 'Abel Aguilar Flores',
			role: 'Encargado del backend y compilador',
			image: '/images/imagen.jpg',
			description: 'Implementó el analizador léxico y sintáctico.'
		}
	];
</script>
<div>
<section class="articles">
	{#each creators as creator (creator.name)}
	<article>
	  <div class="article-wrapper">
		<figure>
		  <img src={creator.image} alt={creator.name} />
		</figure>
		<div class="article-body">
		  <h2>{creator.name}</h2>
		  <p>{creator.role}</p>
		  <p class="description">{creator.description}</p>
		  <a href="#" class="read-more">
			Contactar <span class="sr-only">sobre {creator.name}</span>
			<svg xmlns="http://www.w3.org/2000/svg" class="icon" viewBox="0 0 20 20" fill="currentColor">
			  <path fill-rule="evenodd" d="M12.293 5.293a1 1 0 011.414 0l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414-1.414L14.586 11H3a1 1 0 110-2h11.586l-2.293-2.293a1 1 0 010-1.414z" clip-rule="evenodd" />
			</svg>
		  </a>
		</div>
	  </div>
	</article>
	{/each}
  </section>
</div>

<style>
article {
  --img-scale: 1.001;
  --title-color: white;
  --link-icon-translate: -20px;
  --link-icon-opacity: 0;
  position: relative;
  border-radius: 16px;
  box-shadow: none;
  background: #111;
  color: white;
  transform-origin: center;
  transition: all 0.4s ease-in-out;
  overflow: hidden;
}

article a::after {
  position: absolute;
  inset-block: 0;
  inset-inline: 0;
  cursor: pointer;
  content: "";
}

article h2 {
  margin: 0 0 18px 0;
  font-family: "Bebas Neue", cursive;
  font-size: 1.9rem;
  letter-spacing: 0.06em;
  color: var(--title-color);
  transition: color 0.3s ease-out;
}

figure {
  margin: 0;
  padding: 0;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}

article img {
  max-width: 100%;
  transform-origin: center;
  transform: scale(var(--img-scale));
  transition: transform 0.4s ease-in-out;
}

.article-body {
  padding: 24px;
}

article a {
  display: inline-flex;
  align-items: center;
  text-decoration: none;
  color: #ccc;
}

article a:focus {
  outline: 1px dotted #ccc;
}

article a .icon {
  min-width: 24px;
  width: 24px;
  height: 24px;
  margin-left: 5px;
  transform: translateX(var(--link-icon-translate));
  opacity: var(--link-icon-opacity);
  transition: all 0.3s;
}

article:has(:hover, :focus) {
  --img-scale: 1.1;
  --title-color: #bbb;
  --link-icon-translate: 0;
  --link-icon-opacity: 1;
  box-shadow: rgba(255, 255, 255, 0.1) 0px 10px 36px 0px, rgba(255, 255, 255, 0.04) 0px 0px 0px 1px;
}

body {
  background-color: #000;
  color: #fff;
  font-family: "Figtree", sans-serif;
  font-size: 1.2rem;
  line-height: 1.6rem;
  margin: 0;
  padding: 48px 0;
}

.articles {
  display: grid;
  max-width: 1200px;
  margin-inline: auto;
  padding-inline: 24px;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 24px;
}

@media screen and (max-width: 960px) {
  article {
    container: card/inline-size;
  }
  .article-body p.description {
    display: none;
  }
}

@container card (min-width: 380px) {
  .article-wrapper {
    display: grid;
    grid-template-columns: 100px 1fr;
    gap: 16px;
  }
  .article-body {
    padding-left: 0;
  }
  figure {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
  figure img {
    height: 100%;
    aspect-ratio: 1;
    object-fit: cover;
  }
}

.sr-only:not(:focus):not(:active) {
  clip: rect(0 0 0 0); 
  clip-path: inset(50%);
  height: 1px;
  overflow: hidden;
  position: absolute;
  white-space: nowrap; 
  width: 1px;
}
</style>