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
      <div
        class="flex h-full flex-col items-center justify-evenly gap-4 rounded-2xl border border-gray-200 bg-[#669bbc] p-4"
      >
        <span class="font-poppins text-[2.75rem] font-semibold text-[#003049]">
          Integrantes
        </span>
        <div class="flex flex-col gap-2">
          <span class="font-poppins text-[1.25rem] text-[#003049]"
            >David andres camelo suarez
          </span>
          <span class="font-poppins text-[1.25rem] text-[#003049]">
            Miguel Angel Buitrago Castillo
          </span>
          <span class="font-poppins text-[1.25rem] text-[#003049]"
            >Cristian Liu Chois Amaya
          </span>
          <span class="font-poppins text-[1.25rem] text-[#003049]"
            >Santiago Villota</span
          >
        </div>
      </div>
    </div>
    <div class="mt-12 flex items-center justify-evenly gap-16">
      <div
        class="rounded-lg border border-gray-200 bg-[#669bbc]/60 p-4 hover:bg-[#669bbc]/90"
      >
        <NuxtLink
          to="https://github.com/mbuitragoc/visual-computing"
          target="_blank"
        >
          <div class="flex items-center gap-2">
            <Icon name="fa-brands:github" class="text-4xl text-white"></Icon>
            <span class="font-poppins text-white">GitHub for webpage</span>
          </div>
        </NuxtLink>
      </div>
      <div
        class="rounded-lg border border-gray-200 bg-[#669bbc]/60 p-4 hover:bg-[#669bbc]/90"
      >
        <NuxtLink
          to="https://github.com/dcamelos/visual-computing-WK-1"
          target="_blank"
        >
          <div class="flex items-center gap-2">
            <Icon name="fa-brands:github" class="text-4xl text-white"></Icon>
            <span class="font-poppins text-white">Github for p5.js</span>
          </div>
        </NuxtLink>
      </div>
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
