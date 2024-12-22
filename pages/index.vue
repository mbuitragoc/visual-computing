<template>
  <div class="container mx-auto px-4 py-12">
    <h1 class="mb-12 text-center font-poppins text-[4rem] font-bold text-white">
      Visual Computing Projects
    </h1>
    <div class="grid grid-cols-1 gap-8 md:grid-cols-2 lg:grid-cols-3">
      <NuxtLink
        v-for="project in projects"
        :key="project.id"
        :to="'/' + project.id"
        class="transform overflow-hidden rounded-xl border border-gray-800 bg-[#669bbc] transition-transform duration-200 hover:scale-105"
      >
        <div class="project-card h-full rounded-lg">
          <img
            :src="project.image"
            :alt="project.title"
            class="h-48 w-full object-cover opacity-80"
          />
          <div class="p-6">
            <h2 class="mb-3 text-xl font-bold text-[#003049]">
              {{ project.title }}
            </h2>
            <p class="text-[#fdf0d5]">{{ project.description }}</p>
          </div>
        </div>
      </NuxtLink>
    </div>
  </div>
</template>

<script lang="ts" setup>
const projects = [
  {
    id: "asteroids",
    title: "Asteroids Game",
    description:
      "A simple asteroids game built with Vue 3 Composition API, TypeScript and p5.js.",
    image: "/images/asteroids.jpeg",
  },
];
</script>

<style></style>
