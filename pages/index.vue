<script setup lang="ts">

/**
* pokemons list
*/
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
        transformation: {image: {resize: {fit: crop, height: 256, width: 256}}, document: {output: {format: webp}}}
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
          transformation: {image: {resize: {fit: crop, height: 32, width: 32}}, document: {output: {format: webp}}}
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
const pokemons = ref();
const { data }: any = await useAsyncQuery(query);
pokemons.value = data.value.pokemons;

/**
* selected pokemon
*/
// second query to fetch the larger image for the selected pokemon
const largeImageQuery = gql`
query Pokemon($slug: String!) {
  pokemon(where: { slug: $slug }) {
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
const selectedPokemon = ref();
const selectedPokemonImage = ref();

// cette fonction utilise la requête de la liste avec les images petites
async function selectPokemon(pokemonSlug: string) {
  for (const pokemon of pokemons.value) {
    if (pokemon.slug === pokemonSlug) {
      selectedPokemon.value = pokemon;
      const { data }: any = await useAsyncQuery(largeImageQuery, {
        slug: pokemonSlug,
      });
      selectedPokemonImage.value = data.value.pokemon.image;
      break;
    };
  }
};

// cette fonction utilise la requête de la liste avec les images petites
/* function selectPokemon(pokemonSlug: string) {
  for (const pokemon of pokemons.value) {
    console.log(pokemon.slug);
    if (pokemon.slug === pokemonSlug) {
      selectedPokemon.value = pokemon;
      break;
    };
  }
}; */



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
  <div class="flex flex-col gap-8 sm:flex-row">
    <div class="space-y-4 sm:w-1/2">
      <!-- TODO Search bar -->
      <div class="flex flex-col justify-center gap-4 2xl:flex-row ">
        <div class="mx-auto space-y-4">
          <label class="block">
            Rechercher par nom de pokémon:
            <input class="w-full p-1 bg-yellow-500 rounded-lg sm:w-auto" type="search" id="query" autofocus
              placeholder="Nom du pokémon" />
          </label>
          <!-- TODO DROPDOWN WITH TYPes of pokemons -->
          <label class="block">
            Type de pokémon:
            <select id="type_de_pokemon" name="type_de_pokemon" class="p-1 bg-yellow-500 rounded-lg">
              <option v-for="pokemon in pokemons" :key="pokemon?.typesDePokemon.id" :value="pokemon?.typesDePokemon.nom">
                {{ pokemon?.typesDePokemon.nom }}</option>
              <option value="volvo">Volvo</option>
            </select>
          </label>
        </div>
      </div>
      <!-- list of pokemons -->
      <ul v-if="pokemons" class="grid grid-cols-2 gap-8 sm:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-6">
        <li v-for="pokemon in pokemons" :key="pokemon?.id">
          <button @click="selectPokemon(pokemon?.slug);"
            class="hover:bg-yellow-500 hover:ring-8 hover:ring-yellow-400 hover:rounded-xl">
            <NuxtImg :src="pokemon?.image.url" :alt="pokemon?.nom" />
            <h2 class="text-lg text-center">{{ pokemon?.nom }}</h2>
            <ul>
              <li v-for="cat in pokemon?.typesDePokemon" :key="cat?.id"
                class="inline-block w-auto text-justify bg-yellow-400 rounded-lg text-red-950">
                {{ cat?.nom }}
              </li>
            </ul>
          </button>
        </li>
      </ul>
      <ul v-else>
        <li>Loading...</li>
      </ul>
    </div>
    <!-- selected pokemon div -->
    <!-- this div is hidden until a pokemon is selected -->
    <!-- when a pokemon is selected this div shows its data -->
    <div v-if="selectedPokemon" class="flex flex-col items-center order-first mx-auto space-y-4 sm:w-1/2 sm:order-none">
      <h2 class="text-2xl font-extrabold text-justify uppercase text-red-950">{{
        selectedPokemon?.nom }}
      </h2>
      <!-- <NuxtImg :src="selectedPokemon?.image.url" :alt="selectedPokemon?.nom" /> -->
      <!-- the selectedPokemonImage reference displays the larger image for the selected pokemon from the query that gets a larger image -->
      <NuxtImg v-if="selectedPokemonImage" :src="selectedPokemonImage.url" :alt="selectedPokemon?.nom" />
      <div v-else>
        L'image en haute définition est en cours de chargement...
      </div>
      <div class="text-justify text-red-950">{{ selectedPokemon?.description }}</div>
      <div class="text-justify text-red-950">Taille: {{ selectedPokemon?.height }} mètres</div>
      <div class="text-justify text-red-950">Masse: {{ selectedPokemon?.mass }} kg</div>
      <div class="text-justify text-red-950">Couleur (hex): {{ selectedPokemon?.color.hex }}</div>
      <!-- ajouter la couleur dans tsconfig.json pour pouvoir la charger dynamiquement, sinon se servir d'une balise html "style" inline remplie de CSS pure -->
      <div class="w-6 h-6" :class="`bg-[${selectedPokemon?.color.hex}]`"></div>
      <div class="text-justify text-red-950">Points de vie: {{ selectedPokemon?.pointDeVie }}</div>
      <!-- v-for pour les types de pokemon -->
      <section>
        <p class="text-justify text-red-950">Type(s) du pokemon: {{ selectedPokemon?.typesDePokemon?.nom }}</p>
        <ul class="list-disc">
          <li v-for="cat in selectedPokemon?.typesDePokemon" :key="cat?.id" class="text-justify text-red-950">
            {{ cat?.nom }}
          </li>
        </ul>
      </section>
      <!-- v-for pour les attaques -->
      <section>
        <div class="text-justify text-red-950">Attaque(s) du pokémon: {{ selectedPokemon?.typesDePokemon?.nom }}</div>
        <ul class="list-disc">
          <li v-for="attaque in selectedPokemon?.attaques" :key="attaque?.id" class="text-justify text-red-950">
            <NuxtImg class="inline-flex rounded-lg shadow-2xl" :src="attaque?.image.url" :alt="attaque?.nom" />
            {{ attaque?.nom }}: {{ attaque?.description }}
            Dégâts: {{ attaque?.degats }}
            Type de l'attaque: <span class="">{{ attaque?.typeDePokemon.nom }}</span>
          </li>
        </ul>
      </section>
    </div>
    <div v-else>
      <p class="text-justify text-red-950">S'il vous plait sélectionnez un pokémon de la liste.</p>
    </div>
  </div>
</template>