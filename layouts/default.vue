<template>
    <Head>
        <Title>Pokedex IA</Title>
        <Meta name="description"
            content="Explorez un Pokedex complet et interactif propulsé par IA. Découvrez des Pokémon uniques avec des informations détaillées et des images générées par intelligence artificielle." />
        <Meta property="og:description" content="Découverte Interactive de Pokémon | Pokedex Complet Alimenté par IA" />
        <Meta property="og:image" content="https://nuxt-hygraph-example.vercel.app/img/pokedex.webp" />
        <Meta property="og:url" content="https://nuxt-hygraph-example.vercel.app/" />
        <Meta property="og:type" content="website" />
        <Meta property="og:locale" content="fr_FR" />
        <Meta property="og:site_name" content="Pokedex IA" />
        <Meta name="twitter:card" content="summary_large_image" />
        <Meta name="twitter:site" content="@CastorObese" />
        <Meta name="twitter:creator" content="@CastorObese" />
        <Meta name="twitter:title" content="Pokedex IA" />
        <Meta name="twitter:description" content="Découverte Interactive de Pokémon | Pokedex Complet Alimenté par IA" />
        <Meta name="theme-color" content="#DC143C" />
    </Head>

    <div class="min-h-screen bg-red-500">
        <nav
            class="container mx-auto px-3 h-24 flex items-center mb-4 shadow-2xl rounded-b-2xl bg-[radial-gradient(ellipse_at_left,_var(--tw-gradient-stops))] from-red-500 to-red-900">
            <!-- there was mb-10 -->
            <NuxtLink to="/">
                <h1 class="font-serif text-5xl">Pokedex</h1>
            </NuxtLink>
        </nav>
        <div class="container px-3 pb-4 mx-auto">
            <slot />
        </div>
    </div>
</template>