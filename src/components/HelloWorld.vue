<script setup lang="ts">
import { ref, onMounted, watch } from "vue";

defineProps<{ msg: string }>();

const BASE_URL = "https://rickandmortyapi.com/api";

const page = ref<number>(1);
const totalPages = ref<number>(1);
const heroes = ref<any[]>([]);
const loading = ref<boolean>(false);

const fetchHeroes = async () => {
    console.log("Component was mounted");
    try {
        const heroesResponse = await fetch(
            `${BASE_URL}/character/?page=${page.value}`
        );
        const jsonResponse = await heroesResponse.json();
        console.log("jsonResponse == ", jsonResponse);
        heroes.value = jsonResponse.results;
        totalPages.value = jsonResponse.info.pages;
    } catch (error) {
        console.error(error);
        heroes.value = [];
    } finally {
        loading.value = false;
    }
};

onMounted(fetchHeroes);
watch([page, totalPages], () => {
    // console.log("heroes changed == ", heroes.value);
    // console.log("page changed == ", page.value);
    // console.log("totalPages changed == ", totalPages.value);
    // console.log("loading changed == ", loading.value);
    fetchHeroes();
});
watch([heroes, page, totalPages, loading], () => {
    console.log("heroes changed == ", heroes.value);
    console.log("page changed == ", page.value);
    console.log("totalPages changed == ", totalPages.value);
    console.log("loading changed == ", loading.value);
});

const handleNextPage = () => {
    if (page.value < totalPages.value) {
        page.value++;
    }
};
const handlePrevPage = () => {
    if (page.value > 1) {
        page.value--;
    }
};
</script>

<template>
    <li v-for="item in heroes">{{ item.name }} - {{ item.status }}</li>

    <h1>{{ msg }}</h1>

    <div class="card">
        <button type="button" @click="handlePrevPage">Prev</button>
        <button type="button" @click="">{{ page }} / {{ totalPages }}</button>
        <button type="button" @click="handleNextPage">Next</button>
    </div>
</template>

<style scoped>
.read-the-docs {
    color: #888;
}
</style>
