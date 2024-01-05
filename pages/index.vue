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
      css
    }
    typesDePokemon {
      id
      nom
      couleur {
        css
      }
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

// list of pokemon types
const pokemonTypesQuery = gql`
query pokemonTypesQuery {
  typesDePokemon {
    nom
    id
    couleur {
      css
    }
  }
}
`;
const pokemonTypes = ref();
async function getPokemonTypesData() {
  const { data }: any = await useAsyncQuery(pokemonTypesQuery);
  return data;
}
async function main() {
  const data = await getPokemonTypesData();
  pokemonTypes.value = data.value.typesDePokemon;
}
main();

// Filtrage des pokémons par type (dropdown) et par le nom (input type search)
const selectedType = ref('Tout type');
const searchQuery = ref('');
const updateSelectedType = (event: { target: any }) => {
  selectedType.value = event.target.value;
};
const filteredPokemons = computed(() => {
  return pokemons.value.filter((pokemon: { typesDePokemon: any[]; nom: string; }) => {
    const filterByType = selectedType.value === 'Tout type' || pokemon.typesDePokemon.some((type: { id: string; }) => type.id === selectedType.value);
    const filterByName = !searchQuery.value || pokemon.nom.toLowerCase().includes(searchQuery.value.toLowerCase());
    return filterByType && filterByName;
  });
});
</script>

<template>
  <div class="flex flex-col gap-8 sm:flex-row">
    <div class="space-y-4 sm:w-1/2">
      <!-- Search bar -->
      <div class="flex flex-col justify-center gap-4 2xl:flex-row ">
        <div class="mx-auto space-y-4">
          <label class="block">
            Rechercher par nom de pokémon:
            <input v-model="searchQuery" class="w-full p-1 placeholder-gray-400 bg-yellow-500 rounded-lg sm:w-auto"
              type="search" id="searchQuery" autofocus placeholder="Nom du pokémon" />
          </label>
          <!-- dropdown type de pokémons -->
          <label class="block">
            Type de pokémon:
            <select @change="updateSelectedType" id="type_de_pokemon" name="type_de_pokemon"
              class="p-1 bg-yellow-500 rounded-lg">
              <option>Tout type</option>
              <option v-for="pokemonType in pokemonTypes" :key="pokemonType.id" :value="pokemonType.id"
                :style="{ 'background-color': pokemonType.couleur.css }">
                {{ pokemonType.nom }}
              </option>
            </select>
          </label>
        </div>
      </div>
      <!-- list of pokémons -->
      <ul v-if="filteredPokemons" class="grid grid-cols-2 gap-8 sm:grid-cols-3 lg:grid-cols-4 2xl:grid-cols-6">
        <li v-for="pokemon in filteredPokemons" :key="pokemon?.id">
          <button @click="selectPokemon(pokemon?.slug);"
            class="hover:bg-yellow-500 hover:ring-8 hover:ring-yellow-500 hover:rounded-xl">
            <NuxtImg :src="pokemon?.image.url" :alt="pokemon?.nom" />
            <h2 class="text-lg text-center">{{ pokemon?.nom }}</h2>
            <ul>
              <li v-for="cat in pokemon?.typesDePokemon" :key="cat?.id" :style="{ 'background-color': cat.couleur.css }"
                class="inline-block w-auto p-1 text-justify rounded-lg text-red-950 uppercase">
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
    <div v-if="selectedPokemon" class="flex flex-col items-center order-first mx-auto space-y-4 sm:w-1/2 sm:order-none">
      <h2 class="text-3xl font-extrabold text-justify uppercase text-red-950">{{
        selectedPokemon?.nom }}
      </h2>
      <NuxtImg v-if="selectedPokemonImage" :src="selectedPokemonImage.url" :alt="selectedPokemon?.nom" />
      <div v-else>
        L'image en haute définition est en cours de chargement...
      </div>
      <div class="space-y-4">
        <div class="text-justify text-red-950">{{ selectedPokemon?.description }}</div>
        <div class="text-justify text-red-950">Taille: {{ selectedPokemon?.height }} mètres</div>
        <div class="text-justify text-red-950">Masse: {{ selectedPokemon?.mass }} kg</div>
        <div class="text-justify text-red-950">Couleur (hex): {{ selectedPokemon?.color.hex }}</div>
        <!-- ajouter la couleur dans tsconfig.json pour pouvoir la charger dynamiquement, sinon se servir d'une balise html "style" inline remplie de CSS pure (:style) -->
        <div class="w-6 h-6" :class="`bg-[${selectedPokemon?.color.hex}]`"
          :style="{ 'background-color': selectedPokemon?.color.css }">
        </div>
        <div class="text-justify text-red-950">Points de vie: {{ selectedPokemon?.pointDeVie }}</div>
        <section>
          <p class="text-justify text-red-950 text-lg">Type(s) du pokemon: {{ selectedPokemon?.typesDePokemon?.nom }}</p>
          <ul class="list-disc">
            <li v-for="cat in selectedPokemon?.typesDePokemon" :key="cat?.id" class="text-justify text-red-950">
              {{ cat?.nom }}
            </li>
          </ul>
        </section>
        <section>
          <div class="text-justify text-red-950 text-lg">Attaque(s) du pokémon: {{ selectedPokemon?.typesDePokemon?.nom }}
          </div>
          <ul class="list-disc">
            <li v-for="attaque in selectedPokemon?.attaques" :key="attaque?.id" class="text-justify text-red-950">
              <NuxtImg class="inline-flex rounded-lg shadow-2xl" :src="attaque?.image.url" :alt="attaque?.nom" />
              {{ attaque?.nom }}: {{ attaque?.description }}
              Dégâts: {{ attaque?.degats }}.
              Type de l'attaque: <span class="">{{ attaque?.typeDePokemon?.nom }}</span>.
            </li>
          </ul>
        </section>
      </div>
    </div>
    <div v-else>
      <p class="text-justify text-red-950">S'il vous plait sélectionnez un pokémon de la liste.</p>
    </div>
  </div>
</template>