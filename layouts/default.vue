<template>
    <div class="min-h-screen bg-red-500">
        <nav class="container mx-auto px-3 h-24 flex items-center mb-10">
            <NuxtLink to="/">
                <h1 class="text-5xl">Pokedex</h1>
            </NuxtLink>
        </nav>
        <div class="container mx-auto px-3">
            <slot />
        </div>
    </div>
</template>