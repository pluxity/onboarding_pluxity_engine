<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하고 엔진을 초기화하세요
const container = document.getElementById('webGLContainer');
// 2. Px.Core.Initialize로 엔진을 초기화한 후
// 3. building.FBX 파일을 로드하세요`;
  
    const sampleCode = `// WebGL 컨테이너 선택
const container = document.getElementById('webGLContainer');

// 1. 엔진 초기화 및 FBX 로드
const initializeEngine = (container) => {
    return new Promise((resolve, reject) => {
        Px.Core.Initialize(container, () => resolve(container));
    });
};

// 2. FBX 파일 로드 및 층 정보 분석
const loadBuildingFbx = () => {
    return new Promise((resolve, reject) => {
        Px.Loader.LoadFbx({
            url: '/building.FBX',
            onLoad: (url, floorData) => resolve(floorData),
            onError: reject
        });
    });
};

// 3. 층 정보 정렬
const sortFloorData = (floorData) => {
    const sortedFloors = [...floorData].sort((a, b) => {
        if (a.name === '전체') return 1;
        if (b.name === '전체') return -1;

        const [aNum, bNum] = [a, b].map(floor => 
            parseInt(floor.name.replace(/[BF]/g, '')));
        const [aIsBasement, bIsBasement] = [a, b].map(floor => 
            floor.name.startsWith('B'));

        if (!aIsBasement && bIsBasement) return -1;
        if (aIsBasement && !bIsBasement) return 1;

        return aIsBasement ? aNum - bNum : bNum - aNum;
    });

    return [...sortedFloors, { name: '전체' }];
};

// 4. POI 데이터 로드 및 필터링
const loadAndFilterPois = () => {
    const poiData2 = JSON.parse(localStorage.getItem('poiData2') || '[]');
    return poiData2.filter(poi => poi.property.assignYn === 'Y');
};

// 5. POI 추가 및 표시
const addAndShowPois = (pois) => {
    return new Promise((resolve, reject) => {
        if (pois.length === 0) return resolve();
        
        Px.Poi.AddFromDataArray(
            pois,
            () => {
                pois.forEach(poi => Px.Poi.Show(poi.id));
                Px.Model.Transparent.SetAll(50);
                resolve();
            },
            reject
        );
    });
};

// 6. 층 선택 UI 업데이트
const updateFloorUI = (sortedFloors) => {
    const select = document.querySelector('#floorSelect');
    select.innerHTML = sortedFloors
        .map(floor => \`<option value="\${floor.name}">\${floor.name}</option>\`)
        .join('');
    select.value = '전체';
};

// 7. 층 변경 이벤트 핸들러
const handleFloorChange = (selectedFloor) => {
    const showAll = () => {
        Px.Model.Visible.ShowAll();
        Px.Poi.ShowAll();
    };

    const showSelectedFloor = (floor) => {
        Px.Model.Visible.Show(floor);
        const allPois = Px.Poi.GetDataAll();
        allPois
            .filter(poi => poi.property.floorName === floor)
            .forEach(poi => Px.Poi.Show(poi.id));
    };

    Px.Poi.HideAll();
    Px.Model.Visible.HideAll();

    selectedFloor === '전체' 
        ? showAll()
        : showSelectedFloor(selectedFloor);
};

// 메인 실행 흐름
const main = async () => {
    try {
        await initializeEngine(container);
        console.log('엔진 초기화 완료');

        const floorData = await loadBuildingFbx();
        console.log('건물 불러오기 완료!');
        console.log('층 정보:', floorData);

        const sortedFloors = sortFloorData(floorData);
        console.log('정렬된 층 정보:', sortedFloors);

        const filteredPois = loadAndFilterPois();
        await addAndShowPois(filteredPois);
        console.log('POI 표시 완료');

        updateFloorUI(sortedFloors);
        
        // 이벤트 리스너 등록
        document.querySelector('#floorSelect')
            .addEventListener('change', (e) => handleFloorChange(e.target.value));

    } catch (error) {
        console.error('오류 발생:', error);
    }
};

// 실행
main();`;
  
    const implementationCode = `LoadFbx: function (option) {
    try {
        // Core모듈 체크
        if (!core) {
            throw 'Px.Loader.LoadFbx: Core 모듈이 초기화 되지 않았습니다.';
        }

        // 옵션 체크
        if (!option || !option.url) {
            throw 'Px.Loader.LoadFbx: 유효하지 않은 옵션입니다.';
        }

        // 로드
        PxFbxLoader.load(option.url, function (object) {
            core.workObjGroup.add(object);

            // 가시화 처리에 층정보 설정
            const floorData = PxFbxLoader.analyzeFloorInfo(object);
            core.modelVisibility.setFloorData(floorData);

            core.camera.fitToObject(object);

            if (option.onLoad) {
                option.onLoad(option.url, floorData);
            }
        }, function (err) {
            throw err;
        });

    } catch (e) {
        if (option && option.onError) {
            option.onError(e);
        }
    }
}`;
  
    const functionSignature = `LoadFbx(option: {
    url: string;
    onLoad?: (url: string, floorData: any) => void;
    onError?: (error: any) => void;
}): void;`;
  
    let editor;
    let container;
    let showSample = false;
    let floorList = [];
    let selectedFloor = null;
  
    // 층 정보 업데이트 함수를 전역으로 노출
    onMount(() => {
        window.updateFloorList = (floorData) => {
            floorList = Object.keys(floorData).map(floor => ({
                id: floor,
                name: `${floor}층`
            }));
            if (floorList.length > 0) {
                selectedFloor = floorList[0].id;
            }
        };

        if (container) {
            const webGLContainer = document.createElement('div');
            webGLContainer.id = 'webGLContainer';
            webGLContainer.style.width = '100%';
            webGLContainer.style.height = '100%';
            container.appendChild(webGLContainer);
        }

        return () => {
            delete window.updateFloorList;
        };
    });

    async function initEditor() {
        try {
            const monaco = await loader.init();
            
            editor = monaco.editor.create(document.getElementById('codeEditor'), {
                value: defaultCode,
                language: 'javascript',
                theme: 'vs-dark',
                minimap: { enabled: false },
                fontSize: 14,
                lineNumbers: 'on',
                scrollBeyondLastLine: false,
                automaticLayout: true,
                tabSize: 2,
                wordWrap: 'on',
                wrappingIndent: 'same',
                formatOnPaste: true,
                formatOnType: true
            });

            window.addEventListener('resize', () => {
                editor?.layout();
            });
        } catch (error) {
            console.error('Monaco Editor 로드 실패:', error);
        }
    }
  
    function runCode() {
        if (!container) return;
        
        try {
            const code = editor?.getValue() || '';
            eval(code);
        } catch (error) {
            console.error('실행 중 오류 발생:', error);
            container.innerHTML = `<div class="text-red-500 p-4">
                <p class="font-bold">오류 발생:</p>
                <pre class="mt-2">${error.message}</pre>
            </div>`;
        }
    }
  
    function loadSample() {
        if (editor) {
            editor.setValue(sampleCode);
            showSample = true;
        }
    }

    function handleFloorChange() {
        if (selectedFloor) {
            console.log('선택된 층:', selectedFloor);
            // 여기에 층 변경 로직 추가
        }
    }

    onMount(initEditor);
</script>

<div class="py-4">
    <article class="prose prose-lg max-w-none">
        <h1 class="text-4xl font-bold mb-6">건물 FBX 로드 및 층 정보 관리</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                building.FBX 파일을 로드하여 3D 건물 모델을 화면에 표시하고,
                층 정보를 분석하여 사용자가 원하는 층을 선택할 수 있습니다.
            </p>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">함수 시그니처</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{@html functionSignature}</code></pre>
            </div>
            
            <div class="mt-4 space-y-2">
                <h3 class="font-semibold">매개변수</h3>
                <ul class="list-disc pl-6 space-y-2 text-gray-600">
                    <li><code>url</code>: 로드할 FBX 파일의 경로</li>
                    <li><code>onLoad</code>: 로드 성공 시 호출될 콜백 함수 (층 정보 포함)</li>
                    <li><code>onError</code>: 로드 실패 시 호출될 콜백 함수</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{@html implementationCode}</code></pre>
            </div>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 Px 엔진의 FBX 로더 구현을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>Core 모듈이 초기화되었는지 확인합니다.</li>
                    <li>옵션의 유효성을 검사합니다.</li>
                    <li>FBX 파일을 비동기적으로 로드합니다.</li>
                    <li>로드된 객체를 분석하여 층 정보를 추출합니다.</li>
                    <li>카메라를 객체에 맞게 조정합니다.</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">실습하기</h2>
            
            <div class="mb-4 space-y-4 text-gray-600">
                <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
                    <p class="text-blue-700">
                        <strong>도전과제:</strong>
                    </p>
                    <div class="mt-2 space-y-4">
                        <div>
                            <p class="font-semibold">1️⃣ 엔진 초기화 및 FBX 로드</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>WebGL 컨테이너 선택</li>
                                <li>Px.Core.Initialize로 엔진 초기화</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">2️⃣ 층 정보 분석 및 정렬</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>FBX 파일에서 층 정보 추출</li>
                                <li>지하부터 지상층까지 정렬</li>
                                <li>전체 보기 옵션 추가</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">3️⃣ POI 데이터 처리</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>localStorage에서 POI 데이터 로드</li>
                                <li>assignYn 속성으로 POI 필터링</li>
                                <li>필터링된 POI 추가 및 표시</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">4️⃣ UI 구현 및 이벤트 처리</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>층 선택 드롭다운 UI 구현</li>
                                <li>층 변경 이벤트 핸들러 구현</li>
                                <li>선택된 층에 따른 모델과 POI 표시/숨김 처리</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">5️⃣ 에러 처리 및 최적화</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>비동기 작업의 에러 처리</li>
                                <li>성능 최적화 (불필요한 재렌더링 방지)</li>
                                <li>메모리 누수 방지를 위한 정리</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <div class="text-yellow-700">
                        <strong>💡 도전과제 진행 시 주의사항:</strong>
                        <ul class="list-disc ml-6 mt-2">
                            <li>/building.FBX 파일을 사용하세요</li>
                            <li>localStorage의 poiData2 데이터를 사용하세요</li>
                            <li>각 과제는 이전 과제가 완료되어야 진행 가능합니다.</li>
                        </ul>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-lg shadow-lg overflow-hidden">
                <div class="bg-gray-800 text-white p-4 flex justify-between items-center">
                    <span>코드 편집기</span>
                    <div class="space-x-2">
                        <button 
                            class="bg-gray-600 hover:bg-gray-700 px-4 py-2 rounded"
                            on:click={loadSample}
                        >
                            예제 코드 보기
                        </button>
                        <button 
                            class="bg-[#04AA6D] hover:bg-[#059862] px-4 py-2 rounded"
                            on:click={runCode}
                        >
                            실행하기
                        </button>
                    </div>
                </div>
                
                <div id="codeEditor" class="h-[300px]"></div>

                <div class="p-4 bg-gray-900 relative">
                    <p class="text-gray-400 mb-2">WebGL 컨테이너:</p>
                    <div 
                        bind:this={container} 
                        class="border-2 border-dashed border-gray-700 h-[500px] rounded-lg bg-black relative"
                    >
                        <div class="absolute bottom-4 right-4 bg-gray-800 p-2 rounded shadow-lg">
                            <select 
                                id="floorSelect"
                                class="bg-gray-700 text-white px-3 py-1 rounded"
                            >
                                <option value="">층을 선택하세요</option>
                            </select>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">주의사항</h2>
            <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                <p class="text-yellow-700">
                    LoadFbx 함수를 호출하기 전에 반드시 Px.Core.Initialize가 호출되어 있어야 합니다.
                    또한, 층 정보는 FBX 파일의 구조에 따라 다르게 나타날 수 있습니다.
                </p>
            </div>
        </section>
    </article>
</div>

<style>
    :global(.monaco-editor) {
        padding-top: 8px;
    }

    :global(.prose) {
        max-width: none;
    }
    :global(.prose pre) {
        margin: 0;
        border-radius: 0.375rem;
    }
</style> 