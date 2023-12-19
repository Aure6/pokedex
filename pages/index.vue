<script setup>
// pokemon lists query
const query = gql`
  query Pokemons {
    pokemons(orderBy: id_ASC) {
      createdAt
      description
      id
      nom
      publishedAt
      slug
      updatedAt
      image {
        url(
          transformation: {
            document: { output: { format: webp } }
            image: { resize: { fit: crop, height: 256, width: 256 } }
          }
        )
      }
    }
  }
`;
const pokemons = ref();
const { data } = await useAsyncQuery(query);
console.log(data.value);
pokemons.value = data.value.pokemons;

// selected pokemon query
const selectedPokemonQuery = gql`
  query Pokemon($slug: String!) {
  pokemon(where: {slug: $slug}) {
    id
    nom
    slug
    description
    createdAt
    publishedAt
    updatedAt
    stage
    image {
      url(
        transformation: {image: {resize: {fit: crop, height: 1024, width: 1024}}, document: {output: {format: webp}}}
      )
    }
    pointDeVie
    mass
    height
    color {
      hex
    }
    typesDePokemon {
      ... on TypeDePokemon {
        id
        nom
      }
    }
  }
}
`;
// const pokemon = ref();
// const route = useRoute();
// const { selectedPokemonData } = await useAsyncQuery(query, {
//   slug: route.params.slug,
// });
// console.log(selectedPokemonData.value);
// pokemon.value = selectedPokemonData.value.pokemon;
</script>

<template>
  <div class="flex">
    <ul v-if="pokemons" class="w-1/2 grid gap-8 grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-6">
      <li v-for="pokemon in pokemons" :key="pokemon?.id">
        <NuxtLink :to="`/pokemon/${pokemon?.slug}`">
          <NuxtImg :src="pokemon?.image.url" :alt="pokemon?.nom" />
          <h2 class="text-3xl text-center">{{ pokemon?.nom }}</h2>
        </NuxtLink>
      </li>
    </ul>
    <ul v-else>
      <li>Loading...</li>
    </ul>
    <div v-if="selectedPokemonQuery" class="w-1/2 space-y-4 mx-auto">
      <aside class="lg:float-right lg:clear-right space-y-4 mb-4 lg:mx-4 mx-auto">
        <h2 class="text-3xl text-center">{{ pokemon?.nom }}</h2>
        <figure>
          <NuxtImg class="shadow-2xl rounded-lg mx-auto" :src="pokemon?.image.url" :alt="pokemon?.nom" />
        </figure>
      </aside>
      <p class="text-justify text-red-950">{{ pokemon?.description }}</p>
      <p class="text-justify text-red-950">Taille: {{ pokemon?.height }} mètres</p>
      <p class="text-justify text-red-950">Masse: {{ pokemon?.mass }} kg</p>
      <p class="text-justify text-red-950">Couleur (hex): {{ pokemon?.color.hex }}</p>
      <!-- ajouter la couleur dans tsconfig.json pour pouvoir la charger dynamiquement, sinon se servir d'une balise html "style" inline remplie de CSS pure -->
      <div class="h-6 w-6" :class="`bg-[${pokemon?.color.hex}]`"></div>
      <p class="text-justify text-red-950">{{ pokemon?.pointDeVie }} points de vie</p>
      <p class="text-justify text-red-950">Type de pokemon: {{ pokemon?.typesDepokemon?.nom }}</p>
    </div>
  </div>
</template>