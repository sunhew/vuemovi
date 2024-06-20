<template>
  <HeaderSection />

  <main id="main" role="main" class="detail-view">
    <div class="container text_top">
      <div class="movie-header">
        <img
          :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path"
          :alt="movie.title"
          class="poster"
        />
        <div class="movie-info">
          <h2>{{ movie.title }}</h2>
          <p><strong>장르:</strong> {{ genreNames }}</p>
          <p><strong>상영 시간:</strong> {{ movie.runtime }}분</p>
          <p>
            <strong>개봉일:</strong> {{ movie.release_date }}
            <button v-if="trailerUrl" @click="showTrailer = true" class="trailer-button">
              예고편 재생
            </button>
          </p>
          <p><strong>평점:</strong> {{ movie.vote_average }}</p>
          <p v-if="movie.tagline" class="tagline">{{ movie.tagline }}</p>
          <p><strong>줄거리:</strong> {{ movie.overview }}</p>
        </div>
      </div>

      <div class="cast-crew-details line_top" v-if="cast.length || crew.length">
        <div class="tabs">
          <button :class="{ active: activeTab === 'cast' }" @click="activeTab = 'cast'">
            출연 배우
          </button>
          <button :class="{ active: activeTab === 'crew' }" @click="activeTab = 'crew'">
            제작진
          </button>
        </div>
        <div class="cast-container" v-if="activeTab === 'cast'">
          <ul>
            <li v-for="actor in cast" :key="actor.id">
              <img
                v-if="actor.profile_path"
                :src="'https://image.tmdb.org/t/p/w500/' + actor.profile_path"
                :alt="actor.name"
              />
              <img v-else src="@/assets/img/noimg.jpg" alt="No Image" class="noimg" />
              <p>{{ actor.name }} as {{ actor.character }}</p>
            </li>
          </ul>
        </div>
        <div class="crew-container" v-if="activeTab === 'crew'">
          <ul>
            <li v-for="member in crew" :key="member.id">
              <img
                v-if="member.profile_path"
                :src="'https://image.tmdb.org/t/p/w500/' + member.profile_path"
                :alt="member.name"
              />
              <img v-else src="@/assets/img/noimg.jpg" alt="No Image" class="noimg" />
              <p>{{ member.name }} - {{ member.job }}</p>
            </li>
          </ul>
        </div>
      </div>

      <div class="series-details line_top" v-if="series.length">
        <h3>시리즈</h3>
        <ul>
          <li
            v-for="item in series"
            :key="item.id"
            @click="goToMovieDetail(item.id)"
            class="series-item"
          >
            <img :src="'https://image.tmdb.org/t/p/w500/' + item.poster_path" :alt="item.title" />
            <p>{{ item.title }}</p>
          </li>
        </ul>
      </div>

      <div class="similar-movies line_top" v-if="similarMovies.length">
        <h3>비슷한 장르의 추천 영화</h3>
        <ul>
          <li
            v-for="movie in similarMovies"
            :key="movie.id"
            @click="goToMovieDetail(movie.id)"
            class="similar-movie-item"
          >
            <img :src="'https://image.tmdb.org/t/p/w500/' + movie.poster_path" :alt="movie.title" />
            <p>{{ movie.title }}</p>
          </li>
        </ul>
      </div>
    </div>

    <div v-if="showTrailer" class="trailer-modal">
      <div class="trailer-content">
        <button class="close-button" @click="showTrailer = false">닫기</button>
        <iframe
          width="560"
          height="315"
          :src="trailerUrl"
          frameborder="0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
      </div>
    </div>
  </main>

  <FooterSection />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'

const movie = ref({})
const cast = ref([])
const crew = ref([])
const series = ref([])
const similarMovies = ref([])
const genreNames = ref('')
const trailerUrl = ref('')
const showTrailer = ref(false)
const activeTab = ref('cast')
const searchResults = ref([])
const latestMovies = ref([])
const popularMovies = ref([])
const showSearchResults = ref(false)
const route = useRoute()
const router = useRouter()
const apikey = '88a535afaab2b14f78898e1f97747c4e'

const fetchMovieDetails = async (movieID) => {
  try {
    const response = await axios.get(
      `https://api.themoviedb.org/3/movie/${movieID}?api_key=${apikey}&language=ko-KR&append_to_response=credits,videos`
    )
    movie.value = response.data
    cast.value = response.data.credits.cast
    crew.value = response.data.credits.crew
    genreNames.value = response.data.genres.map((genre) => genre.name).join(', ')
    const trailers = response.data.videos.results
    const youtubeTrailer = trailers.find(
      (video) => video.site === 'YouTube' && video.type === 'Trailer'
    )
    if (youtubeTrailer) {
      trailerUrl.value = `https://www.youtube.com/embed/${youtubeTrailer.key}`
    }
    if (response.data.belongs_to_collection) {
      fetchSeries(response.data.belongs_to_collection.id)
    }
    fetchSimilarMovies(response.data.genres[0].id)
  } catch (error) {
    console.log(error)
  }
}

const fetchSeries = async (collectionId) => {
  if (!collectionId) return
  try {
    const response = await axios.get(
      `https://api.themoviedb.org/3/collection/${collectionId}?api_key=${apikey}&language=ko-KR`
    )
    series.value = response.data.parts
  } catch (error) {
    console.log(error)
  }
}

const fetchSimilarMovies = async (genreId) => {
  try {
    const response = await axios.get(
      `https://api.themoviedb.org/3/discover/movie?api_key=${apikey}&language=ko-KR&with_genres=${genreId}`
    )
    similarMovies.value = response.data.results
  } catch (error) {
    console.log(error)
  }
}

const searchMovies = async (query) => {
  try {
    const searchResponseKo = await axios.get(
      `https://api.themoviedb.org/3/search/movie?api_key=${apikey}&query=${query}&language=ko-KR`
    )
    const searchResponseEn = await axios.get(
      `https://api.themoviedb.org/3/search/movie?api_key=${apikey}&query=${query}&language=en-US`
    )
    searchResults.value = searchResponseKo.data.results.map((movie, index) => ({
      ko: movie,
      en: searchResponseEn.data.results[index],
      showEnglish: false
    }))
    showSearchResults.value = true
    if (searchResults.value.length === 0) {
      alert('검색 결과가 없습니다.')
    }
  } catch (error) {
    console.log(error)
  }
}

const fetchMovies = async () => {
  try {
    const latestResponseKo = await axios.get(
      `https://api.themoviedb.org/3/movie/now_playing?api_key=${apikey}&language=ko-KR&page=1`
    )
    const latestResponseEn = await axios.get(
      `https://api.themoviedb.org/3/movie/now_playing?api_key=${apikey}&language=en-US&page=1`
    )
    latestMovies.value = latestResponseKo.data.results.map((movie, index) => ({
      ko: movie,
      en: latestResponseEn.data.results[index],
      showEnglish: false
    }))
  } catch (error) {
    console.log(error)
  }
}

const bestMovies = async () => {
  try {
    const popularResponseKo = await axios.get(
      `https://api.themoviedb.org/3/movie/popular?api_key=${apikey}&language=ko-KR&page=1`
    )
    const popularResponseEn = await axios.get(
      `https://api.themoviedb.org/3/movie/popular?api_key=${apikey}&language=en-US&page=1`
    )
    popularMovies.value = popularResponseKo.data.results.map((movie, index) => ({
      ko: movie,
      en: popularResponseEn.data.results[index],
      showEnglish: false
    }))
  } catch (error) {
    console.log(error)
  }
}

const goToMovieDetail = (movieId) => {
  router.push({ name: 'detail', params: { movieID: movieId } })
}

watch(route, () => {
  if (route.query.query) {
    searchMovies(route.query.query)
  } else {
    fetchMovieDetails(route.params.movieID)
    showSearchResults.value = false
  }
})

onMounted(() => {
  if (route.query.query) {
    searchMovies(route.query.query)
  } else {
    fetchMovieDetails(route.params.movieID)
  }
  fetchMovies()
  bestMovies()
})
</script>

<style lang="scss">
.detail-view {
  .container {
    margin-top: 120px;
    display: flex;
    flex-direction: column;
    align-items: center;

    .movie-header {
      display: flex;
      flex-direction: row;
      gap: 20px;
      margin-bottom: 20px;
      background-color: var(--pointBg);
      border-radius: 15px;

      .poster {
        width: 300px;
        height: auto;
        border-radius: 10px;
        border-right: 1px dashed var(--black);
      }

      .movie-info {
        display: flex;
        flex-direction: column;
        justify-content: center;
        gap: 10px;

        h2 {
          font-family: var(--fontJ);
          font-weight: bold;
          font-size: 32px;
          color: var(--white);
        }

        p {
          font-size: 1.3rem;
          margin: 0;

          strong {
            font-weight: bold;
            font-size: 1.5rem;
          }

          .trailer-button {
            margin-left: 10px;
            padding: 5px 10px;
            font-size: 14px;
            cursor: pointer;
            background-color: var(--pointBg);
            color: var(--white);
            border: none;
            border-radius: 5px;

            &:hover {
              background-color: var(--pointColorhover2);
            }
          }
        }

        .tagline {
          margin-top: 10px;
          font-style: italic;
          font-size: 2rem;
          color: var(--black500);
        }
      }
    }

    .cast-crew-details {
      width: 100%;
      overflow-x: hidden;

      .tabs {
        display: flex;
        justify-content: center;
        margin-bottom: 20px;

        button {
          padding: 10px 20px;
          margin: 0 10px;
          cursor: pointer;
          background-color: var(--pointColorhover2);
          color: var(--white);
          border: none;
          border-radius: 5px;
          font-size: 1.3rem;

          &.active {
            background-color: var(--pointColor);
            color: var(--black);
            font-size: 1.3rem;
          }
        }
      }

      .cast-container,
      .crew-container {
        display: flex;
        overflow-x: scroll;
        padding-bottom: 10px;

        // 스크롤바 스타일
        &::-webkit-scrollbar {
          width: 8px;
          height: 12px;
        }

        &::-webkit-scrollbar-thumb {
          background: linear-gradient(to right, #74c9a6, #9dbcdb);
          border-radius: 5px;
        }

        &::-webkit-scrollbar-thumb:hover {
          background: var(--pointColor);
        }

        ul {
          display: flex;
          flex-wrap: nowrap;
          gap: 20px;
          list-style: none;
          padding: 0;
          margin: 0;

          li {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 120px;
            flex: 0 0 auto;

            img {
              width: 100px;
              height: 150px;
              border-radius: 10px;
              margin-bottom: 10px;
              object-fit: cover;
            }

            .noimg {
              width: 100px;
              height: 150px;
              border-radius: 10px;
              margin-bottom: 10px;
              object-fit: cover;
            }

            p {
              font-size: 0.9rem;
              text-align: center;
              margin: 0;
            }
          }
        }
      }
    }

    .series-details {
      width: 100%;
      margin-top: 20px;

      h3 {
        font-family: var(--fontJ);
        font-weight: 300;
        font-size: 28px;
        margin-bottom: 20px;
      }

      ul {
        display: flex;
        flex-wrap: nowrap;
        gap: 20px;
        list-style: none;
        padding: 0;
        margin: 0;
        overflow-x: auto;

        li {
          display: flex;
          flex-direction: column;
          align-items: center;
          width: 120px;
          flex: 0 0 auto;

          img {
            width: 100px;
            height: 150px;
            border-radius: 10px;
            margin-bottom: 10px;
            object-fit: cover;
          }

          .noimg {
            width: 100px;
            height: 150px;
            border-radius: 10px;
            margin-bottom: 10px;
            object-fit: cover;
          }

          p {
            font-size: 14px;
            text-align: center;
            margin: 0;
          }
        }
      }
    }

    .similar-movies {
      width: 100%;
      margin-top: 20px;

      h3 {
        font-family: var(--fontJ);
        font-weight: 300;
        font-size: 28px;
        margin-bottom: 20px;
      }

      ul {
        display: flex;
        flex-wrap: nowrap;
        gap: 20px;
        list-style: none;
        padding: 0;
        margin: 0;
        overflow-x: scroll;
        margin-bottom: 10px;

        // 스크롤바 스타일
        &::-webkit-scrollbar {
          width: 8px;
          height: 10px;
        }

        &::-webkit-scrollbar-track {
        }

        &::-webkit-scrollbar-thumb {
          background: linear-gradient(to right, #74c9a6, #9dbcdb);
          border-radius: 5px;
        }

        &::-webkit-scrollbar-thumb:hover {
          background: var(--pointColor);
        }

        li {
          display: flex;
          flex-direction: column;
          align-items: center;
          width: 120px;
          flex: 0 0 auto;

          img {
            width: 100px;
            height: 150px;
            border-radius: 10px;
            margin-bottom: 10px;
            object-fit: cover;
          }

          .noimg {
            width: 100px;
            height: 150px;
            border-radius: 10px;
            margin-bottom: 10px;
            object-fit: cover;
          }

          p {
            font-size: 14px;
            text-align: center;
            margin: 0;
          }
        }
      }
    }
  }
}

.trailer-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 1000;

  .trailer-content {
    position: relative;
    width: 80%;
    max-width: 800px;
    background-color: #fff;
    padding: 20px;
    border-radius: 10px;

    iframe {
      width: 100%;
      height: 450px;
    }

    .close-button {
      position: absolute;
      top: 10px;
      right: 10px;
      padding: 5px 10px;
      cursor: pointer;
      background-color: #ff0000;
      color: white;
      border: none;
      border-radius: 5px;
    }
  }
}
</style>
