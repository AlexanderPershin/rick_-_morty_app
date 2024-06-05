<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import moment from "moment";

// defineProps<{ msg: string }>();

const BASE_URL = "https://rickandmortyapi.com/api";

const page = ref<number>(1);
const totalPages = ref<number>(1);
const heroes = ref<any[]>([]);

const filterName = ref<string>("");
const filterStatus = ref<string>("");
const statusOptions = ref([
    { text: "All", value: "" },
    { text: "Alive", value: "Alive" },
    { text: "Dead", value: "Dead" },
]);

const loading = ref<boolean>(false);

const fetchHeroes = async () => {
    console.log("Component was mounted");
    try {
        let url = `${BASE_URL}/character/?page=${page.value}`;
        if (filterName.value) {
            url += `&name=${filterName.value}`;
        }
        if (filterStatus.value) {
            url += `&status=${filterStatus.value}`;
        }
        const heroesResponse = await fetch(url);
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

watch([filterName, filterStatus], () => {
    console.log("filterName == ", filterName);
    console.log("filterStatus == ", filterStatus);
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

const handleFilter = (e: Event) => {
    e.preventDefault();
    page.value = 1;
    fetchHeroes();
};
const handleClearFilters = () => {
    page.value = 1;
    filterName.value = "";
    filterStatus.value = "";
    fetchHeroes();
};
</script>

<template>
    <div class="container">
        <form
            class="d-flex m-1 sticky-top"
            role="search"
            @submit="handleFilter"
        >
            <input
                class="form-control me-2"
                type="search"
                placeholder="Search"
                aria-label="Search"
                v-model="filterName"
            />
            <select v-model="filterStatus" class="form-control me-2">
                <option v-for="option in statusOptions" :value="option.value">
                    {{ option.text }}
                </option>
            </select>
            <div
                class="btn-group"
                role="group"
                aria-label="Basic outlined example"
            >
                <button class="btn btn-outline-success" type="submit">
                    Apply
                </button>
                <button
                    class="btn btn-outline-danger"
                    v-if="filterName || filterStatus"
                    type="button"
                    @click="handleClearFilters"
                >
                    Clear
                </button>
            </div>
        </form>

        <div class="text-center" v-if="loading">
            <div class="spinner-border" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>

        <div class="container text-center" v-if="!loading">
            <div class="row row-cols-1 row-cols-md-3 g-4">
                <div
                    v-if="!heroes.length"
                    class="p-3 my-5 mx-1 w-100 d-flex justify-content-center align-items-center"
                >
                    Characers list is empty. Try different filters
                </div>
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

        <nav
            aria-label="Page navigation example"
            class="m-1 sticky-bottom"
            v-if="!loading && heroes.length"
        >
            <ul class="pagination">
                <li class="page-item" v-bind:class="{ disabled: page === 1 }">
                    <a class="page-link" href="#" @click="handlePrevPage"
                        >Previous</a
                    >
                </li>
                <!-- Always show first page -->
                <li class="page-item" v-if="page > 1">
                    <a
                        class="page-link"
                        href="#"
                        @click="() => handleSetPage(1)"
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
                <li
                    class="page-item"
                    v-bind:class="{ active: page }"
                    v-if="page"
                >
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
                    <a class="page-link" href="#" @click="handleNextPage"
                        >Next</a
                    >
                </li>
            </ul>
        </nav>
    </div>
</template>

<style scoped></style>
