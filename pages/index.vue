<script setup>
// @ts-check

// create an interface for the pokemon object
// interface Pokemon {
//   id: string;
//   name: string;
//   age: number;
//   address: string;
//   image: {
//     url: string;
//   };
// }

// pokemon list logic
// pokemon lists query
const query = gql`
  query Pokemons {
    pokemons(orderBy: id_ASC) {
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
    attaques {
      nom
      description
      degats
      image {
        url(
          transformation: {image: {resize: {fit: crop, height: 64, width: 64}}, document: {output: {format: webp}}}
        )
      }
      typeDePokemon {
        id
        nom
      }
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

// selected pokemon logic
// create a reactive reference called "selectedPokemon" and initialize it to null
const selectedPokemon = ref();

/**
* @param {string} pokemonSlug
*/
function selectPokemon(pokemonSlug) {
  // loop through the pokemons array and find the pokemon object that matches the pokemonSlug parameter
  for (const pokemon of pokemons.value) {
    console.log(pokemon.slug);
    if (pokemon.slug === pokemonSlug) {
      // assign the pokemon object to the selectedPokemon reference
      selectedPokemon.value = pokemon;
      // break the loop
      break;
    };
  }
};

// second query to fetch the larger image for the selected pokemon, using the same transformation argument
// We need to query the other fields once again and not just the image, because the POKEMON query expects a Pokemon object as a return type, and a Pokemon object has more fields than just the image. If we only query the image, we will get an error from the GraphQL server saying that we are missing some fields.
const largeImageQuery = gql`
query Pokemon($slug: String!) {
  pokemon(where: { slug: $slug }) {
    # other fields
    image {
      url(
        transformation: {
          document: { output: { format: webp } }
          image: { resize: { fit: crop, height: 1024, width: 1024 } }
        }
      )
    }
  }
}
`;

// create a reactive reference called "pokemonImage" and initialize it to null
/* const pokemonImage = ref(null);

// import the useQuery function and the POKEMON query
import { useQuery } from "@vue/apollo-composable";
import { POKEMON } from "../graphql-operations";

// use the useQuery function to create a query object, passing the POKEMON query and the pokemonSlug parameter as arguments
const { data: pokemonData } = useQuery(POKEMON, () => ({
  slug: selectedPokemon.value.slug,
}));

// use a watchEffect hook to assign the pokemonImage reference to the image.url property of the pokemonData.value.pokemon object
watchEffect(() => {
  if (pokemonData.value) {
    pokemonImage.value = pokemonData.value.pokemon.image.url;
  }
}); */
</script>

<template>
  <div class="flex sm:flex-row flex-col">
    <ul v-if="pokemons" class="sm:w-1/2 grid gap-8 grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-6">
      <!-- TODO Search bar + dropdown with types of pokemon-->
      <li v-for="pokemon in pokemons" :key="pokemon?.id">
        <button @click="selectPokemon(pokemon?.slug);">
          <NuxtImg :src="pokemon?.image.url" :alt="pokemon?.nom" />
          <h2 class="text-3xl text-center">{{ pokemon?.nom }}</h2>
        </button>
      </li>
    </ul>
    <ul v-else>
      <li>Loading...</li>
    </ul>
    <!-- selected pokemon div -->
    <!-- this div is hidden until a pokemon is selected -->
    <!-- when a pokemon is selected this div shows its data -->
    <div v-if="selectedPokemon" class="sm:w-1/2 flex flex-col items-center space-y-4 mx-auto sm:order-none order-first">
      <h2 class="uppercase text-justify text-red-950 font-extrabold text-2xl">{{
        selectedPokemon?.nom }}
      </h2>
      <p class="text-justify text-red-950">{{ selectedPokemon?.description }}</p>
      <NuxtImg :src="selectedPokemon?.image.url" :alt="selectedPokemon?.nom" />
      <!-- use the pokemonImage reference to display the larger image for the selected pokemon from the query that gets a larger image -->
      <!-- <NuxtImg :src="pokemonImage" :alt="selectedPokemon?.nom" /> -->
      <!-- TODO -->
    </div>
    <div v-else>
      <p class="text-justify text-red-950">S'il vous plait sélectionnez un pokémon de la liste.</p>
      <!-- <p>Please select a pokemon from the list.</p> -->
    </div>
  </div>
</template>