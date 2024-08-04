<template>
  <main class="main">
    <HomeNavbar />
    <div v-if="pending">
      Loading ...
    </div>
    <div v-else>
      <h2 class="title">Discover Your Favourites. </h2>
      <div class="container">
        <NuxtLink
          v-for="movie in shows"
          :to="`/${movie.media_type}/${movie.id}`"
          :key="movie.id"
        >
          <HomeMovieCard
            :title="movie.original_title || movie.original_name"
            :image="movie.poster_path"
            :vote="movie.vote_average"
            :date="movie.release_date"
            :id="movie.id"
          />
        </NuxtLink>
      </div>
    </div>
  </main>
</template>

<script setup>

const shows = ref([])
const page = ref(1)
const API_KEY = "fc0f0776a79e75e8888a8335a281da5c"

const movieAPI = async (url, page = 1, options = {}) => {
  const apiUrl = `https://api.themoviedb.org/3${url}?api_key=${API_KEY}&page=${page}`
  const response = await fetch(apiUrl, options)
  const data = await response.json()
  return data
}

onMounted(async () => {
  try {
    const apiRes = await movieAPI("/trending/all/day")
    shows.value = apiRes.results
    console.log("Initial data:", shows.value)

    window.onscroll = async () => {
      let bottomOfWindow = Math.max(window.pageYOffset, document.documentElement.scrollTop, document.body.scrollTop) + window.innerHeight === document.documentElement.offsetHeight

      if (bottomOfWindow) {
        page.value++
        const data = await movieAPI("/trending/all/day", page.value)
        shows.value = [...shows.value, ...data.results]
        console.log("New data fetched:", data.results)
      }
    }
  } catch (error) {
    console.error("Error fetching data:", error)
  }
})
</script>

<style lang="scss">
.main {
  max-width: 1500px;
  width: 95%;
  margin: auto;
}
.title {
  margin: 20px 0;
  color: var(--orange);
}
.container {
  display: grid;
  gap: 20px;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  text-align: center;
}

a {
  text-decoration: none;
  color: black;
}

@media screen and (min-width: 760px) {
  .main {
    width: 80%;
  }
  .container {
    a:first-child {
      grid-column: 1/3;
      h1 {
        font-size: 2rem;
      }
      .play {
        display: block !important;
        width: 80px !important;
      }
    }
  }
}
</style>