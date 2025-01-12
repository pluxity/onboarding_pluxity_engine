<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하고 엔진을 초기화하세요
const container = document.getElementById('webGLContainer');
// 2. Px.Core.Initialize로 엔진을 초기화한 후
// 3. building.FBX 파일을 로드하고
// 4. POI를 추가한 후 POI 더블클릭 시 해당 위치로 이동하는 기능을 구현하세요`;
  
    const sampleCode = `// WebGL 컨테이너 선택
const container = document.getElementById('webGLContainer');

// 1. 엔진 초기화 및 FBX 로드
const initializeEngine = (container) => {
    return new Promise((resolve, reject) => {
        Px.Core.Initialize(container, () => resolve(container));
    });
};

// 2. FBX 파일 로드
const loadBuildingFbx = () => {
    return new Promise((resolve, reject) => {
        Px.Loader.LoadFbx({
            url: '/building.FBX',
            onLoad: (url, floorData) => resolve(floorData),
            onError: reject
        });
    });
};

// 3. POI 데이터 로드 및 필터링
const loadAndFilterPois = () => {
    const poiData2 = JSON.parse(localStorage.getItem('poiData2') || '[]');
    return poiData2.filter(poi => poi.property.assignYn === 'Y');
};

// 4. POI 추가 및 표시
const addAndShowPois = (pois) => {
    return new Promise((resolve, reject) => {
        if (pois.length === 0) return resolve();
        
        Px.Poi.AddFromDataArray(
            pois,
            () => {
                pois.forEach(poi => Px.Poi.Show(poi.id));
                Px.Model.Transparent.SetAll(50);
                resolve(pois);
            },
            reject
        );
    });
};

// 5. POI 더블클릭 이벤트 핸들러
const handlePoiDoubleClick = (poiInfo) => {
    if (!poiInfo || !poiInfo.id) return;
    
    Px.Camera.MoveToPoi({
        id: poiInfo.id,
        isAnimation: true,
        duration: 1000,
        distanceOffset: 5,
        onComplete: () => {
            console.log('POI 이동 완료:', poiInfo.id);
        }
    });
};

// 메인 실행 흐름
const main = async () => {
    try {
        await initializeEngine(container);
        console.log('엔진 초기화 완료');

        await loadBuildingFbx();
        console.log('건물 불러오기 완료!');

        const filteredPois = loadAndFilterPois();
        await addAndShowPois(filteredPois);
        console.log('POI 표시 완료');

        // 이벤트 시스템 활성화
        Px.Event.On();
        
        // POI 더블클릭 이벤트 리스너 등록
        Px.Event.AddEventListener('dblclick', 'poi', handlePoiDoubleClick);

    } catch (error) {
        console.error('오류 발생:', error);
    }
};

// 실행
main();`;
  
    const implementationCode = `MoveToPoi: function (option) {
    const id = option.id;
    const isAnimation = option.isAnimation;
    const duration = option.duration;
    const distanceOffset = option.distanceOffset || 0;
    const onComplete = option.onComplete;
    const isTopView = option.isTopView;
    const topViewDistance = option.topViewDistance;

    if (this.poiMgr.poiList.hasOwnProperty(id)) {
        const poi = this.poiMgr.poiList[id];
        poi.updateMatrixWorld(true);

        const dirPointRight = new Vector3(0, 0, 1);
        dirPointRight.applyQuaternion(poi.pointObject.quaternion);

        const dirPointWorldDir = new Vector3();
        poi.pointObject.getWorldDirection(dirPointWorldDir);

        const poiWorldPos = new Vector3();
        poi.pointObject.getWorldPosition(poiWorldPos);
        
        // POI로 카메라 이동
        this.camera.moveToPoi(option);
    }
}`;
  
    const functionSignature = `MoveToPoi(option: {
    id: string;                    // POI의 고유 식별자
    isAnimation?: boolean;         // 애니메이션 사용 여부
    duration?: number;             // 이동 시간 (밀리초)
    distanceOffset?: number;       // 거리 오프셋
    onComplete?: () => void;       // 이동 완료 후 콜백
    isTopView?: boolean;           // 상단 뷰 사용 여부
    topViewDistance?: number;      // 상단 뷰 거리
}): void;`;
  
    let editor;
    let container;
    let showSample = false;
  
    onMount(() => {
        if (container) {
            const webGLContainer = document.createElement('div');
            webGLContainer.id = 'webGLContainer';
            webGLContainer.style.width = '100%';
            webGLContainer.style.height = '100%';
            container.appendChild(webGLContainer);
        }
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

    onMount(initEditor);
</script>

<div class="py-4">
    <article class="prose prose-lg max-w-none">
        <h1 class="text-4xl font-bold mb-6">POI로 카메라 이동하기</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                3D 공간에서 특정 POI(관심 지점)로 카메라를 이동시키는 기능을 구현합니다.
                POI 데이터를 로드하고, 사용자가 선택한 POI로 카메라를 부드럽게 이동시킵니다.
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
                    <li><code>id</code>: 이동할 POI의 고유 식별자</li>
                    <li><code>isAnimation</code>: 애니메이션 사용 여부 (기본값: false)</li>
                    <li><code>duration</code>: 이동에 걸리는 시간 (밀리초)</li>
                    <li><code>distanceOffset</code>: POI와의 거리 오프셋</li>
                    <li><code>onComplete</code>: 이동 완료 후 실행될 콜백 함수</li>
                    <li><code>isTopView</code>: 상단 뷰 사용 여부</li>
                    <li><code>topViewDistance</code>: 상단 뷰 사용 시 거리</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{@html implementationCode}</code></pre>
            </div>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 Px 엔진의 POI 이동 구현을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>POI의 존재 여부를 확인합니다.</li>
                    <li>POI의 월드 좌표와 방향을 계산합니다.</li>
                    <li>카메라를 POI 위치로 이동시킵니다.</li>
                    <li>애니메이션과 거리 오프셋을 적용합니다.</li>
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
                                <li>building.FBX 파일 로드</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">2️⃣ POI 데이터 처리</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>localStorage에서 POI 데이터 로드</li>
                                <li>assignYn 속성으로 POI 필터링</li>
                                <li>필터링된 POI 추가 및 표시</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">3️⃣ POI 이동 기능 구현</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>POI 선택 UI 구현</li>
                                <li>선택된 POI로 카메라 이동</li>
                                <li>이동 애니메이션 적용</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">4️⃣ 에러 처리 및 최적화</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>POI 존재 여부 확인</li>
                                <li>이동 완료 콜백 처리</li>
                                <li>메모리 누수 방지</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <div class="text-yellow-700">
                        <strong>💡 도전과제 진행 시 주의사항:</strong>
                        <ul class="list-disc ml-6 mt-2">
                            <li>POI 데이터가 유효한지 확인하세요</li>
                            <li>이동 애니메이션은 부드럽게 처리하세요</li>
                            <li>카메라 이동 중 예외 상황을 고려하세요</li>
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
                        class="border-2 border-dashed border-gray-700 h-[500px] rounded-lg bg-black"
                    ></div>
                </div>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">주의사항</h2>
            <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                <p class="text-yellow-700">
                    MoveToPoi 함수를 호출하기 전에 반드시 POI가 추가되어 있어야 합니다.
                    또한, 이동 중에는 다른 POI로의 이동을 방지하는 것이 좋습니다.
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