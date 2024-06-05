<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import moment from "moment";

// defineProps<{ msg: string }>();

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
    fetchHeroes();
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
const handleSetPage = (pageToGo: number) => {
    if (pageToGo >= 1 && pageToGo <= totalPages.value) {
        page.value = pageToGo;
    }
};
</script>

<template>
    <div class="container text-center">
        <!-- <div class="row row-cols-1 row-cols-md-2 g-4"> -->
        <div class="row row-cols-1 row-cols-md-3 g-4">
            <div class="col" v-for="item in heroes">
                <div class="card p-0">
                    <img
                        :src="item.image"
                        class="card-img-top"
                        alt="Character picture"
                    />
                    <div class="card-body">
                        <h5 class="card-title">{{ item.name }}</h5>
                        <p class="card-text"></p>
                    </div>
                    <ul class="list-group list-group-flush">
                        <li class="list-group-item">
                            Species:
                            {{ item.species }}
                            {{ item?.type ? `/ ${item.type}` : null }}
                        </li>
                        <li class="list-group-item">
                            Status:
                            <span
                                v-bind:class="{
                                    'text-danger': item.status === 'Dead',
                                    'text-success': item.status === 'Alive',
                                }"
                            >
                                {{ item.status }}
                            </span>
                        </li>
                        <li class="list-group-item">
                            Origin: {{ item.origin.name }}
                        </li>
                    </ul>
                    <div class="card-footer">
                        <small class="text-body-secondary"
                            >Created:
                            {{
                                moment(item.created).format("DD-MM-YYYY")
                            }}</small
                        >
                    </div>
                </div>
            </div>
        </div>
    </div>

    <nav aria-label="Page navigation example">
        <ul class="pagination">
            <li class="page-item" v-bind:class="{ disabled: page === 1 }">
                <a class="page-link" href="#" @click="handlePrevPage"
                    >Previous</a
                >
            </li>
            <!-- Always show first page -->
            <li class="page-item" v-if="page > 1">
                <a class="page-link" href="#" @click="() => handleSetPage(1)"
                    >1</a
                >
            </li>

            <li class="page-item disabled" v-if="page > 2">
                <a class="page-link" href="#">...</a>
            </li>

            <!-- Prev page -->
            <li class="page-item" v-if="page > 2">
                <a
                    class="page-link"
                    href="#"
                    @click="() => handleSetPage(page - 1)"
                    >{{ page - 1 }}</a
                >
            </li>

            <!-- Current page -->
            <li class="page-item" v-bind:class="{ active: page }" v-if="page">
                <a class="page-link" href="#">{{ page }}</a>
            </li>

            <!-- Next page -->
            <li class="page-item" v-if="page < totalPages - 1">
                <a
                    class="page-link"
                    href="#"
                    @click="() => handleSetPage(page + 1)"
                    >{{ page + 1 }}</a
                >
            </li>

            <li class="page-item" v-if="page < totalPages - 2">
                <a class="page-link disabled" href="#">...</a>
            </li>

            <!-- Always show last page -->
            <li class="page-item" v-if="page < totalPages">
                <a
                    class="page-link"
                    href="#"
                    @click="() => handleSetPage(totalPages)"
                    >{{ totalPages }}</a
                >
            </li>
            <li
                class="page-item"
                v-bind:class="{ disabled: page === totalPages }"
            >
                <a class="page-link" href="#" @click="handleNextPage">Next</a>
            </li>
        </ul>
    </nav>
</template>

<style scoped></style>
