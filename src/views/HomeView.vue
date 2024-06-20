<template>
  <HeaderSection />

  <main id="main" role="main">
    <div class="view__inner container" v-if="route.query.query && searchResults.length > 0">
      <section class="view__card style1">
        <h3 class="text_top">검색 결과</h3>
        <div class="card-container">
          <div class="card" v-for="movie in searchResults" :key="movie.ko.id">
            <div class="card-inner">
              <div class="card-front">
                <img
                  :src="'https://image.tmdb.org/t/p/w500/' + movie.ko.poster_path"
                  :alt="movie.ko.title"
                />
                <span>{{ movie.ko.title }}</span>
              </div>
              <div class="card-back">
                <div class="toggle-container">
                  <h3>{{ movie.showEnglish ? movie.en.title : movie.ko.title }}</h3>
                </div>
                <p>
                  {{ movie.showEnglish ? 'Release Date' : '개봉일' }}:
                  {{ movie.showEnglish ? movie.en.release_date : movie.ko.release_date }}
                </p>
                <p>
                  {{ movie.showEnglish ? 'Rating' : '평점' }}:
                  {{ movie.showEnglish ? movie.en.vote_average : movie.ko.vote_average }}
                </p>
                <p>{{ movie.showEnglish ? movie.en.overview : movie.ko.overview }}</p>
                <div class="button-container">
                  <button class="action-btn" @click="playTrailer(movie.ko.id)">예고편</button>
                  <button class="action-btn" @click="viewDetails(movie.ko.id)">상세정보</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>

    <div class="view__inner container" v-else>
      <!-- 최신 영화 슬라이드 섹션 -->
      <section class="view__card style1">
        <h3 class="text_top">최신 영화</h3>
        <div class="slider">
          <ul @mouseover="pauseAnimation" @mouseleave="resumeAnimation">
            <li v-for="movie in latestMovies" :key="movie.ko.id">
              <div class="card">
                <div class="card-inner">
                  <div class="card-front">
                    <img
                      :src="'https://image.tmdb.org/t/p/w500/' + movie.ko.poster_path"
                      :alt="movie.ko.title"
                    />
                    <span>{{ movie.ko.title }}</span>
                  </div>
                  <div class="card-back">
                    <div class="toggle-container">
                      <h3>{{ movie.showEnglish ? movie.en.title : movie.ko.title }}</h3>
                    </div>
                    <p>
                      {{ movie.showEnglish ? 'Release Date' : '개봉일' }}:
                      {{ movie.showEnglish ? movie.en.release_date : movie.ko.release_date }}
                    </p>
                    <p>
                      {{ movie.showEnglish ? 'Rating' : '평점' }}:
                      {{ movie.showEnglish ? movie.en.vote_average : movie.ko.vote_average }}
                    </p>
                    <p>{{ movie.showEnglish ? movie.en.overview : movie.ko.overview }}</p>
                    <div class="button-container">
                      <button class="action-btn" @click="playTrailer(movie.ko.id)">예고편</button>
                      <button class="action-btn" @click="viewDetails(movie.ko.id)">상세정보</button>
                    </div>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </section>

      <!-- 인기 영화 섹션 -->
      <section class="view__card style1">
        <h3>인기 영화</h3>
        <div class="card-container">
          <div class="card" v-for="movie in popularMovies" :key="movie.ko.id">
            <div class="card-inner">
              <div class="card-front">
                <img
                  :src="'https://image.tmdb.org/t/p/w500/' + movie.ko.poster_path"
                  :alt="movie.ko.title"
                />
                <span>{{ movie.ko.title }}</span>
              </div>
              <div class="card-back">
                <div class="toggle-container">
                  <h3>{{ movie.showEnglish ? movie.en.title : movie.ko.title }}</h3>
                </div>
                <p>
                  {{ movie.showEnglish ? 'Release Date' : '개봉일' }}:
                  {{ movie.showEnglish ? movie.en.release_date : movie.ko.release_date }}
                </p>
                <p>
                  {{ movie.showEnglish ? 'Rating' : '평점' }}:
                  {{ movie.showEnglish ? movie.en.vote_average : movie.ko.vote_average }}
                </p>
                <p>{{ movie.showEnglish ? movie.en.overview : movie.ko.overview }}</p>
                <div class="button-container">
                  <button class="action-btn" @click="playTrailer(movie.ko.id)">예고편</button>
                  <button class="action-btn" @click="viewDetails(movie.ko.id)">상세정보</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
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
import HeaderSection from '@/components/HeaderSection.vue'
import FooterSection from '@/components/FooterSection.vue'
import { onMounted, ref, watch } from 'vue'
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'

const latestMovies = ref([])
const popularMovies = ref([])
const searchResults = ref([])
const showTrailer = ref(false)
const trailerUrl = ref('')
const apikey = '88a535afaab2b14f78898e1f97747c4e'
const route = useRoute()
const router = useRouter()

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
    if (searchResults.value.length === 0) {
      alert('검색 결과가 없습니다.')
    }
  } catch (error) {
    console.log(error)
  }
}

const playTrailer = async (movieID) => {
  try {
    const response = await axios.get(
      `https://api.themoviedb.org/3/movie/${movieID}/videos?api_key=${apikey}&language=ko-KR`
    )
    const trailers = response.data.results
    const youtubeTrailer = trailers.find(
      (video) => video.site === 'YouTube' && video.type === 'Trailer'
    )
    if (youtubeTrailer) {
      trailerUrl.value = `https://www.youtube.com/embed/${youtubeTrailer.key}`
      showTrailer.value = true
    } else {
      alert('예고편을 찾을 수 없습니다.')
    }
  } catch (error) {
    console.log(error)
  }
}

const viewDetails = (movieID) => {
  router.push({ name: 'detail', params: { movieID } })
}

watch(
  () => route.query.query,
  (newQuery) => {
    if (newQuery) {
      searchMovies(newQuery)
    } else {
      searchResults.value = []
    }
  }
)

onMounted(fetchMovies)
onMounted(bestMovies)
</script>

<style lang="scss">
.view__card {
  margin-top: 110px; /* 카드 섹션의 상단 마진 */
  width: 100%; /* 카드 섹션의 너비를 100%로 설정 */
  overflow: hidden; /* 넘치는 콘텐츠를 숨김 */

  h3 {
    font-family: 'theJamsil'; /* 폰트 패밀리 설정 */
    font-weight: 300; /* 폰트 굵기 설정 */
    margin-bottom: 10px; /* 하단 마진 */
    font-size: 26px; /* 폰트 크기 설정 */
  }
}

.toggle-container {
  position: relative; /* 상대적 위치 설정 */
  margin-bottom: 1rem; /* 하단 마진 */
  width: 100%; /* 너비 100% */
}

.toggle-btn {
  position: absolute; /* 절대적 위치 설정 */
  bottom: -19px; /* 하단에 위치 */
  right: 0; /* 오른쪽에 위치 */
  padding: 0.5rem 1rem; /* 버튼의 내부 여백 */
  border: none; /* 테두리 제거 */
  background-color: var(--pointColor); /* 배경색 설정 */
  color: var(--white); /* 글자색 설정 */
  border-radius: 5px; /* 테두리 반경 설정 */
  cursor: pointer; /* 마우스 포인터를 손가락 모양으로 변경 */
  font-size: 1rem; /* 폰트 크기 설정 */
}

.toggle-btn:hover {
  background-color: var(--pointColorhover); /* 버튼에 마우스를 올렸을 때 배경색 변경 */
  color: var(--black200);
}

.button-container {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 1rem;
}

.action-btn {
  padding: 0.5rem 1rem;
  border: none;
  background-color: var(--pointColor);
  color: var(--white);
  border-radius: 5px;
  cursor: pointer;
  font-size: 1rem;
}

.action-btn:hover {
  background-color: var(--pointColorhover);
  color: var(--black200);
}

.slider {
  position: relative; /* 상대적 위치 설정 */
  overflow: hidden; /* 넘치는 콘텐츠를 숨김 */

  ul {
    display: flex; /* 내부 요소들을 플렉스 박스로 배치 */
    padding: 0; /* 내부 여백 제거 */
    margin-top: 20px; /* 상단 마진 */
    height: 27rem; /* 높이 설정 */
    animation: slide 20s linear infinite; /* 슬라이드 애니메이션 설정 */
    animation-play-state: running; /* 애니메이션 재생 상태 설정 */

    li {
      list-style: none; /* 리스트 스타일 제거 */
      margin-right: 10px; /* 오른쪽 마진 */
      transition: transform 0.3s; /* 트랜지션 효과 설정 */

      .card {
        perspective: 1000px; /* 카드의 3D 깊이 설정 */
        width: 15.5rem; /* 카드 너비 설정 */
        height: 27rem; /* 카드 높이 설정 */
      }

      .card-inner {
        position: relative; /* 상대적 위치 설정 */
        width: 100%; /* 너비 100% 설정 */
        height: 100%; /* 높이 100% 설정 */
        transition: transform 0.6s; /* 트랜지션 효과 설정 */
        transform-style: preserve-3d; /* 3D 변환 유지 */
      }

      &:hover .card-inner {
        transform: rotateY(180deg); /* 마우스를 올렸을 때 카드 회전 */
      }

      .card-front,
      .card-back {
        position: absolute; /* 절대적 위치 설정 */
        width: 100%; /* 너비 100% 설정 */
        height: 100%; /* 높이 100% 설정 */
        backface-visibility: hidden; /* 뒷면 보이지 않게 설정 */
        -webkit-backface-visibility: hidden; /* 웹킷 브라우저에 대한 설정 */
      }

      .card-front {
        display: flex; /* 내부 요소들을 플렉스 박스로 배치 */
        flex-direction: column; /* 세로 방향으로 배치 */
        align-items: center; /* 요소들을 중앙 정렬 */
        justify-content: center; /* 요소들을 중앙 정렬 */

        img {
          width: 100%; /* 이미지 너비 100% 설정 */
          height: auto; /* 높이는 자동으로 설정 */
          margin-bottom: 5px;
        }

        span {
          margin-top: 10px; /* 상단 마진 */
          font-weight: bold; /* 폰트 굵기 설정 */
        }
      }

      .card-back {
        transform: rotateY(180deg); /* 뒷면 회전 설정 */
        background-color: var(--pointBg); /* 배경색 설정 */
        display: flex; /* 내부 요소들을 플렉스 박스로 배치 */
        flex-direction: column; /* 세로 방향으로 배치 */
        align-items: center; /* 요소들을 중앙 정렬 */
        justify-content: center; /* 요소들을 중앙 정렬 */
        padding: 20px; /* 내부 여백 설정 */
        text-align: center; /* 텍스트 중앙 정렬 */
        color: var(--white);
      }

      &:hover {
        transform: scale(1.05); /* 마우스를 올렸을 때 확대 */
      }

      span {
        display: block; /* 블록 요소로 설정 */
        text-align: center; /* 텍스트 중앙 정렬 */
        font-size: 1.2rem; /* 폰트 크기 설정 */
        overflow: hidden; /* 넘치는 콘텐츠 숨김 */
        white-space: nowrap; /* 텍스트를 한 줄로 표시 */
        text-overflow: ellipsis; /* 텍스트 오버플로우 설정 */
      }

      .card-back p {
        display: -webkit-box; /* 플렉스 박스 레이아웃 사용 */
        -webkit-box-orient: vertical; /* 플렉스 박스의 방향을 수직으로 설정 */
        -webkit-line-clamp: 5; /* 최대 5줄까지 표시 */
        text-align: center; /* 텍스트 중앙 정렬 */
        font-size: 1.2rem; /* 폰트 크기 설정 */
        overflow: hidden; /* 넘치는 콘텐츠 숨김 */
        text-overflow: ellipsis; /* 텍스트 오버플로우 설정 */
        margin: 0.5rem 0; /* 상하단 마진 설정 */
        max-height: 12rem; /* 최대 높이 설정 */
        white-space: normal; /* 여러 줄로 표시 */
      }

      .card-back h3 {
        font-size: 1.5rem; /* 폰트 크기 설정 */
        margin-bottom: 1rem; /* 하단 마진 설정 */
        overflow: hidden; /* 넘치는 콘텐츠 숨김 */
        text-overflow: ellipsis; /* 텍스트 오버플로우 설정 */
        white-space: nowrap; /* 텍스트를 한 줄로 표시 */
      }

      &:hover {
        text-decoration: underline; /* 마우스를 올렸을 때 밑줄 설정 */
        text-decoration-color: var(--black400); /* 밑줄 색상 설정 */
        cursor: pointer; /* 마우스 포인터를 손가락 모양으로 변경 */
      }
    }

    &:hover {
      animation-play-state: paused; /* 마우스를 올렸을 때 애니메이션 일시 정지 */
    }
  }
}

@keyframes slide {
  0% {
    transform: translateX(0); /* 초기 위치 설정 */
  }
  100% {
    transform: translateX(-100%); /* 최종 위치 설정 */
  }
}

.card-container {
  display: grid; /* 그리드 레이아웃 사용 */
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); /* 그리드 템플릿 설정 */
  grid-auto-rows: 1fr; /* 자동 행 크기 설정 */
  gap: 50px; /* 그리드 간격 설정 */
  overflow-x: hidden; /* x축 스크롤 숨김 */
  overflow-y: hidden; /* x축 스크롤 숨김 */

  .card {
    perspective: 1000px; /* 카드의 3D 깊이 설정 */
    height: 500px; /* 카드 높이 고정 */
  }

  .card-inner {
    position: relative; /* 상대적 위치 설정 */
    width: 100%; /* 너비 100% 설정 */
    height: 100%; /* 높이 100% 설정 */
    transition: transform 0.6s; /* 트랜지션 효과 설정 */
    transform-style: preserve-3d; /* 3D 변환 유지 */
  }

  .card:hover .card-inner {
    transform: rotateY(180deg); /* 마우스를 올렸을 때 카드 회전 */
  }

  .card-front,
  .card-back {
    position: absolute; /* 절대적 위치 설정 */
    width: 100%; /* 너비 100% 설정 */
    height: 100%; /* 높이 100% 설정 */
    backface-visibility: hidden; /* 뒷면 보이지 않게 설정 */
    -webkit-backface-visibility: hidden; /* 웹킷 브라우저에 대한 설정 */
  }

  .card-front {
    display: flex; /* 내부 요소들을 플렉스 박스로 배치 */
    flex-direction: column; /* 세로 방향으로 배치 */
    align-items: center; /* 요소들을 중앙 정렬 */
    justify-content: center; /* 요소들을 중앙 정렬 */

    img {
      width: 100%; /* 이미지 너비 100% 설정 */
      height: auto; /* 높이는 자동으로 설정 */
    }

    span {
      margin-top: 10px; /* 상단 마진 */
      font-weight: bold; /* 폰트 굵기 설정 */
      background-color: var(--black400);
      width: 100%;
      margin: 0;
      text-align: center;
      height: 80px;
      color: var(--white);
      font-size: 1.4rem;
      border-radius: 0 0 15px 15px;
      border-top: 1px dashed var(--white);
    }
  }

  .card-back {
    transform: rotateY(180deg); /* 뒷면 회전 설정 */
    background-color: #fff; /* 배경색 설정 */
    display: flex; /* 내부 요소들을 플렉스 박스로 배치 */
    flex-direction: column; /* 세로 방향으로 배치 */
    align-items: center; /* 요소들을 중앙 정렬 */
    justify-content: center; /* 요소들을 중앙 정렬 */
    padding: 20px; /* 내부 여백 설정 */
    text-align: center; /* 텍스트 중앙 정렬 */
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
      background-color: var(--pointColor);
      color: white;
      border: none;
      border-radius: 5px;
    }
  }
}
</style>
