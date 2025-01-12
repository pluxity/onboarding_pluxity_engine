<script>
  import { onMount } from 'svelte';

  onMount(async () => {
    try {
        // POI 데이터 로드 및 저장
        const poiResponse = await fetch('/poi.json');
        const poiData = await poiResponse.json();
        localStorage.setItem('poiData', JSON.stringify(poiData));

        // POI2 데이터 로드 및 저장
        const poi2Response = await fetch('/poi2.json');
        const poiData2 = await poi2Response.json();
        localStorage.setItem('poiData2', JSON.stringify(poiData2));
    } catch (error) {
        console.error('POI 데이터 로드 실패:', error);
    }
  });

  // localStorage 초기화 함수
  async function resetLocalStorage() {
    try {
      // POI 데이터 다시 로드
      const poiResponse = await fetch('/poi.json');
      const poiData = await poiResponse.json();
      localStorage.setItem('poiData', JSON.stringify(poiData));

      // POI2 데이터 다시 로드
      const poi2Response = await fetch('/poi2.json');
      const poiData2 = await poi2Response.json();
      localStorage.setItem('poiData2', JSON.stringify(poiData2));

      alert('로컬스토리지가 초기화되었습니다.');
    } catch (error) {
      console.error('초기화 실패:', error);
      alert('초기화 중 오류가 발생했습니다.');
    }
  }
</script>

<div class="py-4">
  <article class="prose prose-lg max-w-none">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-4xl font-bold m-0">PLUXITY Px Engine</h1>
      <button 
        on:click={resetLocalStorage}
        class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
      >
        🔄 로컬스토리지 초기화
      </button>
    </div>

    <section class="mb-6">
      <h2 class="text-2xl font-bold mb-3">개요</h2>
      <p class="text-gray-600">
        PLUXITY Px Engine은 3D 시각화를 위한 강력한 엔진입니다.
        이 문서에서는 엔진의 기본적인 사용법과 주요 기능들을 설명합니다.
      </p>
    </section>

    <section class="mb-6">
      <h2 class="text-2xl font-bold mb-3">시작하기</h2>
      <p class="text-gray-600">
        왼쪽 사이드바의 메뉴를 통해 각 기능들을 살펴보세요.
        예제 코드를 직접 실행해보면서 엔진의 동작을 이해할 수 있습니다.
      </p>
    </section>
  </article>
</div>
