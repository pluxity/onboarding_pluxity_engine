<script>
    import { onMount } from 'svelte';

    let container;
    let testResults = [];
    let isTestRunning = false;
    let selectedTest = 'event'; // 'event', 'poi', 'loader' 등

    function addTestResult(name, passed, message = '') {
        testResults = [...testResults, { name, passed, message, time: new Date().toLocaleTimeString() }];
    }

    async function runTests() {
        if (isTestRunning) return;
        isTestRunning = true;
        testResults = [];

        try {
            // 공통 테스트: 엔진 초기화
            await testInitialize();
            
            // 공통 테스트: FBX 로드
            await testLoadFbx();

            // 선택된 테스트 실행
            switch (selectedTest) {
                case 'event':
                    await runEventTests();
                    break;
                case 'poi':
                    await runPoiTests();
                    break;
                // 다른 테스트 케이스들...
            }
        } catch (error) {
            console.error('테스트 실행 중 오류 발생:', error);
            addTestResult('테스트 실행 오류', false, error.message);
        } finally {
            isTestRunning = false;
        }
    }

    // 공통 테스트 함수들
    async function testInitialize() {
        try {
            await new Promise((resolve, reject) => {
                Px.Core.Initialize(container, () => {
                    addTestResult('엔진 초기화', true, 'Core 모듈이 성공적으로 초기화되었습니다.');
                    resolve();
                });
            });
        } catch (error) {
            addTestResult('엔진 초기화', false, error.message);
            throw error;
        }
    }

    async function testLoadFbx() {
        try {
            await new Promise((resolve, reject) => {
                Px.Loader.LoadFbx({
                    url: '/drawing.FBX',
                    onLoad() {
                        addTestResult('FBX 로드', true, 'FBX 파일이 성공적으로 로드되었습니다.');
                        resolve();
                    },
                    onError(error) {
                        reject(new Error(`FBX 로드 실패: ${error}`));
                    }
                });
            });
        } catch (error) {
            addTestResult('FBX 로드', false, error.message);
            throw error;
        }
    }

    // 이벤트 관련 테스트
    async function runEventTests() {
        try {
            // POI 데이터 로드 및 표시
            await testLoadPois();
            
            // 이벤트 시스템 활성화
            await testEventSystem();
            
            // 이벤트 리스너 등록
            await testEventListener();
        } catch (error) {
            console.error('이벤트 테스트 실패:', error);
            throw error;
        }
    }

    async function testLoadPois() {
        try {
            const poiData = JSON.parse(localStorage.getItem('poiData') || '[]');
            const assignedPois = poiData.filter(poi => poi.property.assignYn === 'Y');
            
            if (assignedPois.length === 0) {
                addTestResult('POI 데이터 확인', false, 'localStorage에 표시할 POI가 없습니다.');
                return;
            }

            await new Promise((resolve, reject) => {
                Px.Poi.AddFromDataArray(
                    assignedPois,
                    () => {
                        assignedPois.forEach(poi => Px.Poi.Show(poi.id));
                        addTestResult('POI 표시', true, `${assignedPois.length}개의 POI가 성공적으로 표시되었습니다.`);
                        resolve();
                    },
                    (error) => {
                        reject(new Error(`POI 추가 실패: ${error}`));
                    }
                );
            });
        } catch (error) {
            addTestResult('POI 표시', false, error.message);
            throw error;
        }
    }

    async function testEventSystem() {
        try {
            Px.Event.On();
            addTestResult('이벤트 시스템', true, '이벤트 시스템이 활성화되었습니다.');
        } catch (error) {
            addTestResult('이벤트 시스템', false, error.message);
            throw error;
        }
    }

    async function testEventListener() {
        try {
            let eventFired = false;
            
            Px.Event.AddEventListener('dblclick', 'poi', (poiInfo) => {
                eventFired = true;
                console.log('POI 더블클릭:', poiInfo);
                addTestResult('이벤트 발생 확인', true, `POI(${poiInfo.id}) 더블클릭 이벤트가 발생했습니다.`);
            });

            addTestResult('이벤트 리스너', true, '더블클릭 이벤트 리스너가 등록되었습니다. POI를 더블클릭하여 테스트해보세요.');
        } catch (error) {
            addTestResult('이벤트 리스너', false, error.message);
            throw error;
        }
    }

    // POI 관련 테스트
    async function runPoiTests() {
        try {
            // POI 추가 테스트
            await testAddPoi();
            
            // POI 표시/숨김 테스트
            await testPoiVisibility();
            
            // POI 삭제 테스트
            await testRemovePoi();
        } catch (error) {
            console.error('POI 테스트 실패:', error);
            throw error;
        }
    }

    async function testAddPoi() {
        try {
            const testPoi = {
                id: 'test-poi-' + Date.now(),
                property: {
                    assignYn: 'Y',
                    position: { x: 0, y: 0, z: 0 }
                }
            };

            await new Promise((resolve, reject) => {
                Px.Poi.AddFromDataArray(
                    [testPoi],
                    () => {
                        addTestResult('POI 추가', true, 'POI가 성공적으로 추가되었습니다.');
                        resolve();
                    },
                    (error) => {
                        reject(new Error(`POI 추가 실패: ${error}`));
                    }
                );
            });
        } catch (error) {
            addTestResult('POI 추가', false, error.message);
            throw error;
        }
    }

    async function testPoiVisibility() {
        // POI 표시/숨김 테스트 구현
    }

    async function testRemovePoi() {
        // POI 삭제 테스트 구현
    }

    onMount(() => {
        if (container) {
            const webGLContainer = document.createElement('div');
            webGLContainer.id = 'webGLContainer';
            webGLContainer.style.width = '100%';
            webGLContainer.style.height = '100%';
            container.appendChild(webGLContainer);
        }
    });
</script>

<div class="p-4">
    <div class="max-w-6xl mx-auto">
        <div class="mb-6">
            <h1 class="text-3xl font-bold mb-4">Px 엔진 테스트</h1>
            <p class="text-gray-600 mb-4">
                이 페이지에서는 Px 엔진의 다양한 기능을 테스트할 수 있습니다.
                각 테스트는 순차적으로 실행되며, 이전 단계가 성공해야 다음 단계로 진행됩니다.
            </p>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div>
                <div class="bg-white rounded-lg shadow-lg overflow-hidden mb-6">
                    <div class="bg-gray-800 text-white p-4">
                        <div class="flex justify-between items-center">
                            <span>테스트 선택</span>
                            <div class="space-x-2">
                                <select 
                                    bind:value={selectedTest}
                                    class="bg-gray-700 text-white px-3 py-1 rounded"
                                >
                                    <option value="event">이벤트</option>
                                    <option value="poi">POI</option>
                                </select>
                                <button 
                                    class="bg-[#04AA6D] hover:bg-[#059862] px-4 py-2 rounded disabled:opacity-50"
                                    on:click={runTests}
                                    disabled={isTestRunning}
                                >
                                    테스트 실행
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="p-4 bg-black">
                        <div 
                            bind:this={container} 
                            class="border-2 border-dashed border-gray-700 h-[400px] rounded-lg"
                        ></div>
                    </div>
                </div>

                <div class="bg-white rounded-lg shadow-lg overflow-hidden">
                    <div class="bg-gray-800 text-white p-4">
                        <span>테스트 설명</span>
                    </div>
                    <div class="p-4">
                        {#if selectedTest === 'event'}
                            <div class="space-y-4">
                                <h3 class="font-semibold">이벤트 테스트</h3>
                                <ul class="list-disc pl-6 space-y-2 text-gray-600">
                                    <li>엔진 초기화</li>
                                    <li>FBX 파일 로드</li>
                                    <li>POI 데이터 로드 및 표시</li>
                                    <li>이벤트 시스템 활성화</li>
                                    <li>더블클릭 이벤트 리스너 등록</li>
                                </ul>
                            </div>
                        {:else if selectedTest === 'poi'}
                            <div class="space-y-4">
                                <h3 class="font-semibold">POI 테스트</h3>
                                <ul class="list-disc pl-6 space-y-2 text-gray-600">
                                    <li>엔진 초기화</li>
                                    <li>FBX 파일 로드</li>
                                    <li>POI 추가</li>
                                    <li>POI 표시/숨김</li>
                                    <li>POI 삭제</li>
                                </ul>
                            </div>
                        {/if}
                    </div>
                </div>
            </div>

            <div>
                <div class="bg-white rounded-lg shadow-lg overflow-hidden">
                    <div class="bg-gray-800 text-white p-4">
                        <span>테스트 결과</span>
                    </div>
                    <div class="p-4">
                        {#if testResults.length === 0}
                            <p class="text-gray-500 text-center py-4">
                                테스트를 실행하면 결과가 여기에 표시됩니다.
                            </p>
                        {:else}
                            <div class="space-y-4">
                                {#each testResults as result}
                                    <div class="border-l-4 p-4 {result.passed ? 'border-green-400 bg-green-50' : 'border-red-400 bg-red-50'}">
                                        <div class="flex justify-between items-start">
                                            <h3 class="font-semibold {result.passed ? 'text-green-800' : 'text-red-800'}">
                                                {result.name}
                                            </h3>
                                            <span class="text-sm text-gray-500">{result.time}</span>
                                        </div>
                                        <p class="mt-1 text-sm {result.passed ? 'text-green-600' : 'text-red-600'}">
                                            {result.message}
                                        </p>
                                    </div>
                                {/each}
                            </div>
                        {/if}
                    </div>
                </div>
            </div>
        </div>
    </div>
</div> 