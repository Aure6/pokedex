<script setup>
const query = gql`
  query Pokemon($slug: String!) {
    pokemon(where: { slug: $slug }) {
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
          transformation: {
            image: { resize: { fit: crop, height: 1024, width: 1024 } }
            document: { output: { format: webp } }
          }
        )
      }
      pointDeVie
      mass
      height
      color {
        hex
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
</script>

<template>
  <Head v-if="pokemon">
    <Title>{{ pokemon.nom }} - Détails du Pokémon</Title>
    <Meta name="description" :content="`Découvrez des détails sur ${pokemon.nom}: ${pokemon.description}`" />
    <Meta property="og:title" :content="`${pokemon.nom} - Détails du Pokémon`" />
    <Meta property="og:description" :content="`Découvrez des détails sur ${pokemon.nom}: ${pokemon.description}`" />
    <Meta property="og:image" :content="pokemon.image.url" />
    <Meta property="og:type" content="website" />
    <Meta property="og:locale" content="fr_FR" />
    <Meta name="twitter:card" content="summary_large_image" />
    <Meta name="twitter:title" :content="`${pokemon.nom} - Détails du Pokémon`" />
    <Meta name="twitter:description" :content="`Découvrez des détails sur ${pokemon.nom}: ${pokemon.description}`" />
    <Meta name="twitter:image" :content="pokemon.image.url" />
  </Head>

  <div v-if="pokemon" class="max-w-lg space-y-8 mx-auto">
    <NuxtImg class="" :src="pokemon.image.url" :alt="pokemon.nom" />
    <h2 class="text-3xl text-center">{{ pokemon.nom }}</h2>
    <p class="text-justify text-red-950">{{ pokemon.description }}</p>
    <p class="text-justify text-red-950">{{ pokemon.height }}</p>
    <p class="text-justify text-red-950">{{ pokemon.mass }}</p>
    <p class="text-justify text-red-950">{{ pokemon.color.hex }}</p>
    <!-- ajouter la couleur dans tsconfig.json pour pouvoir la charger dynamiquement, sinon se servir d'une balise html "style" remplie de CSS pure -->
    <div class="h-6 w-6" :class="`bg-[${pokemon.color.hex}]`"></div>
    <p class="text-justify text-red-950">{{ pokemon.pointDeVie }}</p>
    <p class="text-justify text-red-950">{{ pokemon.typesDePokemon }}</p>
  </div>
  <div v-else>
    <li>Loading...</li>
  </div>
</template>