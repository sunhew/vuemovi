<template>
  <header id="header" role="banner">
    <div class="header__inner container">
      <h1 @click="goHome">
        <!-- 클릭 시 홈으로 이동하는 함수 호출 -->
        <v-icon name="CoSearch" scale="3"></v-icon>
        <!-- 아이콘 컴포넌트 사용 -->
        Hello <span>Movie</span>
        <!-- 제목 텍스트 -->
      </h1>
      <div class="search">
        <input
          type="search"
          v-model="searchQuery"
          placeholder="영화 검색"
          @keyup.enter="searchMovies"
          spellcheck="false"
        /><!-- 검색어를 바인딩 --><!-- 입력 필드의 플레이스홀더 텍스트 --><!-- 엔터 키를 누르면 검색 함수 호출 -->
        <!-- 맞춤법 검사 비활성화 -->
        <button @click="searchMovies">검색</button>
        <!-- 클릭 시 검색 함수 호출 -->
      </div>
    </div>
  </header>
</template>

<script setup>
import { ref } from 'vue' // ref 함수 임포트
import { useRouter } from 'vue-router' // useRouter 함수 임포트

const searchQuery = ref('') // 검색어를 저장할 ref 변수
const router = useRouter() // 라우터 인스턴스

const searchMovies = () => {
  if (searchQuery.value.trim() === '') {
    // 검색어가 비어 있으면
    alert('검색어를 입력해주세요.') // 알림 표시
    return // 함수 종료
  }
  router.push({ name: 'home', query: { query: searchQuery.value } }) // 검색어를 쿼리로 포함하여 홈으로 이동
}

const goHome = () => {
  searchQuery.value = '' // 검색어 초기화
  router.push({ name: 'home' }).then(() => {
    // 홈으로 이동 후
    window.scrollTo(0, 0) // 페이지 상단으로 스크롤
  })
}
</script>

<style lang="scss">
.header__inner {
  padding: 6px; /* 내부 여백 설정 */
  display: flex; /* 플렉스 박스 레이아웃 사용 */
  justify-content: space-between; /* 요소들을 양쪽 끝으로 정렬 */
  position: fixed; /* 고정 위치 설정 */
  left: 50%; /* 수평 중앙 정렬 */
  transform: translate(-50%); /* 수평 중앙 정렬 보정 */
  top: 0; /* 상단에 위치 */
  z-index: 1000; /* 다른 요소들보다 위에 표시 */
  background-color: rgba(234, 255, 254, 0.1); /* 배경색 및 투명도 설정 */
  backdrop-filter: blur(15px); /* 배경 블러 효과 */
  height: 100px;

  h1 {
    width: 20%; /* 너비 20% 설정 */
    cursor: pointer; /* 커서를 포인터로 변경 */
    display: flex; /* 플렉스 박스 레이아웃 사용 */
    align-items: center; /* 수직 중앙 정렬 */

    span {
      color: var(--pointColor); /* 포인트 색상 설정 */
    }
  }

  .search {
    width: 80%; /* 너비 80% 설정 */
    position: relative; /* 상대적 위치 설정 */

    input {
      position: absolute; /* 절대적 위치 설정 */
      background-color: var(--mainBg); /* 배경색 설정 */
      border: 1px solid var(--black700); /* 테두리 설정 */
      padding: 1rem; /* 내부 여백 설정 */
      right: 0; /* 오른쪽 정렬 */
      top: 24px; /* 상단 여백 */
      height: 20px; /* 높이 설정 */
      width: 30%; /* 너비 30% 설정 */
    }
    button {
      position: absolute; /* 절대적 위치 설정 */
      right: 10px; /* 오른쪽 여백 */
      top: 30px; /* 상단 여백 */
      background-color: var(--mainBg); /* 배경색 설정 */

      &:hover {
        color: var(--pointBg); /* 호버 시 포인트 색상 설정 */
      }
    }
  }
}
</style>
