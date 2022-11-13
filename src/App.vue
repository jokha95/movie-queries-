<script setup>
import { ref, computed, watch } from 'vue';
import MyButton from './components/MyButton.vue';
import MOVIES from './assets/movies.json';

// TIP: when we clone the input array,
// we can use a map function instead of .slice()
// and perform data changes if needed!
const movies = MOVIES.map((movie) => {
  // Object.assign into a new object ({}) is the way to clone objects in JS (sorry about that!)
  return Object.assign({}, movie, {
    score: Number(movie.score),
    // HINT: we can normalize here the genre of each movie, for Level 5 of this challenge ;)
  });
});

// reactive vars
const itemsPerPage = ref(5);
const page = ref(1);
const filterTitle = ref('');
const yearFilter = ref([]);
const genreFilter = ref([]);

function goToNext() {
  page.value++;
}

function goToPrev() {
  page.value--;
}

function filterByYear(movie) {
  // (yearsSelected.value.length === 0) means no years were selected
  if (yearFilter.value.length === 0) return true;
  return yearFilter.value.includes(movie.year);
}

function filterByGenre(movie) {
  // (yearsSelected.value.length === 0) means no years were selected
  if (genreFilter.value.length === 0) return true;
  return genreFilter.value.includes(movie.genre.toLowerCase());
}

function filterByTitle(movie) {
  if (filterTitle.value.length < 2) return true;
  const lowerCaseTitle = filterTitle.value.toLowerCase();
  return movie.title.toLowerCase().includes(lowerCaseTitle);
}

function avergageScore(list) {
  const sum = list.map((m) => m.score).reduce((a, b) => a + b, 0);
  return sum / list.length || 0;
}

const filterMoviesAverageScore = computed(() => {
  return avergageScore(filteredMovies.value);
});

const totalAverageScore = computed(() => {
  return avergageScore(movies);
});

const currentPageMoviesAverageScore = computed(() => {
  return avergageScore(moviesSnapshot.value);
});

const filteredMovies = computed(() => {
  const fm = movies
    .filter(filterByYear)
    .filter(filterByTitle)
    .filter(filterByGenre);
  fm.sort((movie1, movie2) => {
    return movie2.score - movie1.score;
  });
  return fm;
});

const moviesSnapshot = computed(() => {
  return filteredMovies.value.slice(
    (page.value - 1) * itemsPerPage.value,
    page.value * itemsPerPage.value
  );
});

const moviesSnapshotYear = computed(() => {
  const yearsSet = new Set();
  movies.filter((m) => {
    yearsSet.add(m.year);
  });

  //return movies;
  console.log(yearsSet);
  const array = Array.from(yearsSet).sort();
  console.log(array);
  return array;
});

const moviesSnapshotGenre = computed(() => {
  const genresSet = new Set();
  movies.filter((m) => {
    genresSet.add(m.genre.toLowerCase());
  });

  //return movies;
  console.log(genresSet);
  const gArray = Array.from(genresSet);
  console.log(gArray);
  return gArray;
});

const totalPages = computed(() => {
  return Math.ceil(filteredMovies.value.length / itemsPerPage.value);
});

// TIP: we can watch on multiple reactive variables at the same time :)
watch([itemsPerPage, yearFilter], () => {
  page.value = 1;
});

watch(filterTitle, (newValue) => {
  if (newValue.length >= 2) {
    page.value = 1;
  }
});
</script>

<template>
  <input type="number" v-model="itemsPerPage" /> <br />
  filter avergage score: {{ filterMoviesAverageScore }}<br />
  current page avg score: {{ currentPageMoviesAverageScore }}<br />
  total avg score: {{ totalAverageScore }}<br />
  <ul>
    <li v-for="movie in moviesSnapshot" :key="movie.id">
      {{ movie.title }} || {{ movie.year }} || {{ movie.score }} ||
      {{ movie.genre }}
    </li>
  </ul>
  <div>
    <h3>
      Filters
      <input v-model="filterText" />
      <button @click="reset">Reset</button>
    </h3>
  </div>

  <div class="genre">
    <p>Filter By Genre{{ genreFilter }}</p>

    <select name="" id="" v-model="genreFilter" multiple>
      <option v-for="genre in moviesSnapshotGenre">
        {{ genre }}
      </option>
    </select>
  </div>
  <br />

  <div class="genre">
    <p>Filter By Year{{ yearFilter }}</p>
    <select name="" id="" v-model="yearFilter" multiple>
      <option v-for="year in moviesSnapshotYear">
        {{ year }}
      </option>
    </select>
  </div>

  <h3>Pagination Controls</h3>
  <MyButton :disabled="page === 1" @click="page = 1"> first page </MyButton>
  <MyButton :disabled="page === 1" @click="goToPrev"> &lt; </MyButton>
  {{ page }} / {{ totalPages }}
  <MyButton :disabled="page === totalPages" @click="goToNext"> &gt; </MyButton>
  <MyButton :disabled="page === totalPages" @click="page = totalPages">
    last page
  </MyButton>

  <hr />
</template>
