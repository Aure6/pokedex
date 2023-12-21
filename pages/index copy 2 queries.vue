<script setup>
// @ts-check

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
// create a reactive reference called "pokemons"
const pokemons = ref();
// fetch data asynchronously using useAsyncQuery and destructure the data property from the result
const { data } = await useAsyncQuery(query);
console.log(data.value);
console.log(typeof data.value);
// assign the "pokemons" reference to the pokemons property of "data.value"
pokemons.value = data.value.pokemons;
console.log(typeof pokemons.value);

/**
* @param {string} pokemonSlug
*/
/* function selectPokemon(pokemonSlug) {
  for (const pokemon in data.value.pokemons) {
    console.log(data.value.pokemons[pokemon].slug);
    if (data.value.pokemons[pokemon].slug === pokemonSlug) {
      const selectedPokemon = ref();
      selectedPokemon.value = 
    };
  }
}; */
// console.log(data.value.pokemons);
// for (const pokemon in data.value.pokemons) {
//   console.log(data.value.pokemons[pokemon].slug);
//   if (data.value.pokemons[pokemon].slug === pokemonSlug) { };
// }

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
      id
      nom
    }
  }
}
`;

// console.log(selectedPokemonData.value);
// pokemon.value = selectedPokemonData.value.pokemon;
// console.log(selectedPokemonData)
// console.log(selectedPokemonData.value);
// console.log(selectedPokemonData.value[0]);

</script>

<template>
  <div class="flex">
    <ul v-if="pokemons" class="w-1/2 grid gap-8 grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-6">
      <!-- TODO Search bar + dropdown with types of pokemon-->
      <li v-for="pokemon in pokemons" :key="pokemon?.id">
        <button @click="selectPokemon(pokemon?.slug)">
          <NuxtImg :src="pokemon?.image.url" :alt="pokemon?.nom" />
          <h2 class="text-3xl text-center">{{ pokemon?.nom }}</h2>
        </button>
      </li>
    </ul>
    <ul v-else>
      <li>Loading...</li>
    </ul>
    <!-- selected pokemon query -->
    <div v-if="selectedPokemon" class="w-1/2 space-y-4 mx-auto">
      <aside class="lg:float-right lg:clear-right space-y-4 mb-4 lg:mx-4 mx-auto">
        <!-- TODO -->
      </aside>
      <h2>{{ selectedPokemonData?.nom }}</h2>
      <!-- TODO -->
    </div>
    <div v-else>
      <p>Loading...</p>
    </div>
  </div>
</template>