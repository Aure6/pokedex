<script setup>
const query = gql`
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

const pokemon = ref();
const route = useRoute();
const { data } = await useAsyncQuery(query, {
  slug: route.params.slug,
});
console.log(data.value);
pokemon.value = data.value.pokemon;

console.log(pokemon?.typesDePokemon?.nom);
console.log(pokemon?.typesDePokemon);
</script>

<template>
  <Head v-if="pokemon">
    <Title>{{ pokemon?.nom }} - Détails du Pokémon</Title>
    <Meta name="description" :content="`Découvrez des détails sur ${pokemon?.nom}: ${pokemon?.description}`" />
    <Meta property="og:title" :content="`${pokemon?.nom} - Détails du Pokémon`" />
    <Meta property="og:description" :content="`Découvrez des détails sur ${pokemon?.nom}: ${pokemon?.description}`" />
    <Meta property="og:image" :content="pokemon?.image.url" />
    <Meta property="og:type" content="website" />
    <Meta property="og:locale" content="fr_FR" />
    <Meta name="twitter:card" content="summary_large_image" />
    <Meta name="twitter:title" :content="`${pokemon?.nom} - Détails du Pokémon`" />
    <Meta name="twitter:description" :content="`Découvrez des détails sur ${pokemon?.nom}: ${pokemon?.description}`" />
    <Meta name="twitter:image" :content="pokemon?.image.url" />
  </Head>

  <div v-if="pokemon" class="max-w-7xl space-y-4 mx-auto"> <!-- there was max-w-lg  -->
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
    <p class="text-justify text-red-950">Points de vie: {{ pokemon?.pointDeVie }}</p>
    <!-- v-for pour les types de pokemon -->
    <section>
      <p class="text-justify text-red-950">Type(s) du pokemon: {{ pokemon?.typesDePokemon?.nom }}</p>
      <ul class="list-disc">
        <li v-for="cat in pokemon?.typesDePokemon" :key="cat?.id" class="text-justify text-red-950">
          {{ cat?.nom }}
        </li>
      </ul>
    </section>
    <!-- v-for pour les attaques -->
    <section>
      <div class="text-justify text-red-950">Attaque(s) du pokemon: {{ pokemon?.typesDePokemon?.nom }}</div>
      <ul class="list-disc">
        <li v-for="attaque in pokemon?.attaques" :key="attaque?.id" class="text-justify text-red-950">
          <NuxtImg class="inline-flex shadow-2xl rounded-lg" :src="attaque?.image.url" :alt="attaque?.nom" />
          {{ attaque?.nom }}: {{ attaque?.description }}
          Dégâts: {{ attaque?.degats }}
          <div>Type de l'attaque: {{ attaque?.typeDePokemon.nom }}</div>
        </li>
      </ul>
    </section>
  </div>
  <div v-else>
    <li>Loading...</li>
  </div>
</template>