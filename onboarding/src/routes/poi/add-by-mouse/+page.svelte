<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하세요
const container = document.getElementById('webGLContainer');

// 2. 엔진을 초기화하고 FBX를 로드한 후
// 3. localStorage.getItem('poiData')를 통해 POI 데이터를 가져오세요
// 4. ID가 1인 POI를 찾아서 마우스로 배치해보세요`;
  
    const sampleCode = `// WebGL 컨테이너 선택
const container = document.getElementById('webGLContainer');

// 엔진 초기화
Px.Core.Initialize(container, () => {
    // FBX 파일 로드
    Px.Loader.LoadFbx({
        url: '/drawing.FBX',
        async onLoad() {
            console.log('도면 불러오기 완료!');
            
            try {
                // localStorage에서 POI 데이터 가져오기
                const poiData = JSON.parse(localStorage.getItem('poiData') || '[]');
                console.log('POI 데이터:', poiData);
                
                // ID가 1인 POI 찾기
                const targetPoi = poiData.find(poi => poi.id === 1);
                if (targetPoi) {
                    // POI 추가
                    Px.Poi.AddByMouse({
                        ...targetPoi,
                        onComplete: (id, x, y, z) => {
                            console.log('POI 배치 완료:', { id, x, y, z });
                            
                            // 위치 정보와 상태 업데이트
                            const updatedPoiData = poiData.map(poi => 
                                poi.id === id 
                                    ? { 
                                        ...poi, 
                                        position: { x, y, z },
                                        property: {
                                            ...poi.property,
                                            assignYn: 'Y',
                                            position: JSON.stringify({ x, y, z })
                                        }
                                    } 
                                    : poi
                            );
                            
                            // localStorage 업데이트
                            localStorage.setItem('poiData', JSON.stringify(updatedPoiData));
                            console.log('POI 데이터 업데이트 완료');
                        }
                    });
                }
            } catch (error) {
                console.error('POI 데이터 처리 실패:', error);
            }
        },
        onError(error) {
            console.error('도면 로드 실패:', error);
        }
    });
});`;
  
    const implementationCode = `AddByMouse: function (option) {
    core.poiMgr.addPoiByMouse(
        option.type,
        option.url,
        option.id,
        option.group,
        option.iconUrl,
        option.lineHeight,
        option.displayText,
        option.property,
        option.scale,
        option.onComplete
    );
}`;
  
    const functionSignature = `AddByMouse(option: {
    type: string;
    url: string;
    id: number | string;
    group: string;
    iconUrl: string;
    lineHeight: number;
    displayText: string;
    property: any;
    scale?: { x: number; y: number; z: number };
    onComplete?: (id: number | string, x: number, y: number, z: number) => void;
}): void;`;
  
    let editor;
    let container;
    let showSample = false;
  
    onMount(async () => {
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

            // 초기 WebGL 컨테이너 설정
            if (container) {
                const webGLContainer = document.createElement('div');
                webGLContainer.id = 'webGLContainer';
                webGLContainer.style.width = '100%';
                webGLContainer.style.height = '100%';
                container.appendChild(webGLContainer);
            }

            window.addEventListener('resize', () => {
                editor?.layout();
            });
        } catch (error) {
            console.error('Monaco Editor 로드 실패:', error);
        }
    });
  
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
        }
        showSample = true;
    }
</script>

<div class="py-4">
    <article class="prose prose-lg max-w-none">
        <h1 class="text-4xl font-bold mb-6">Px.Poi.AddByMouse</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                마우스를 사용하여 3D 공간에 POI(관심 지점)를 추가합니다.
                이 함수는 POI를 생성하고 사용자가 마우스로 위치를 지정할 수 있게 합니다.
                위치 지정이 완료되면 콜백을 통해 최종 좌표를 받을 수 있습니다.
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
                    <li><code>type</code>: POI의 유형</li>
                    <li><code>url</code>: POI와 연관된 URL</li>
                    <li><code>id</code>: POI의 고유 식별자</li>
                    <li><code>group</code>: POI가 속한 그룹</li>
                    <li><code>iconUrl</code>: POI 아이콘 이미지 URL</li>
                    <li><code>lineHeight</code>: 라인 높이</li>
                    <li><code>displayText</code>: POI에 표시될 텍스트</li>
                    <li><code>property</code>: 추가 속성</li>
                    <li><code>scale</code>: POI의 크기 (기본값: x:100, y:100, z:100)</li>
                    <li><code>onComplete</code>: 위치 지정 완료 시 호출될 콜백 함수</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{@html implementationCode}</code></pre>
            </div>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 POI를 마우스로 배치하는 과정을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>POI의 기존 존재 여부를 확인합니다.</li>
                    <li>기존 POI가 있다면 현재 위치를 저장합니다.</li>
                    <li>새로운 POI라면 초기 위치(0,0,0)에 생성합니다.</li>
                    <li>마우스 배치 도구를 활성화합니다.</li>
                    <li>배치 완료/취소 시 적절한 처리를 수행합니다.</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">실습하기</h2>
            
            <div class="mb-4 space-y-4 text-gray-600">
                <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
                    <p class="text-blue-700">
                        <strong>도전과제:</strong><br>
                        1. FBX 파일을 로드하세요.<br>
                        2. localStorage에서 POI 데이터를 가져오고 ID가 1인 POI를 찾으세요.<br>
                        3. AddByMouse로 POI를 배치하세요.<br>
                        4. 배치 완료 시 POI의 position과 assignYn을 업데이트하세요.
                    </p>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <p class="text-yellow-700">
                        💡 POI 데이터는 localStorage 'poiData'에 저장되어 있습니다. 서버와의 통신및 정제를 간략화 했습니다.
                    </p>
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

                <div class="p-4 bg-gray-900">
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
                    AddByMouse 함수를 호출하기 전에 반드시 Px.Core.Initialize가 호출되어 있어야 하며,
                    3D 모델(FBX)이 로드되어 있어야 합니다.
                    마우스 배치 중 ESC 키를 누르면 배치가 취소됩니다.
                </p>
            </div>
        </section>
    </article>
</div>

<style>
    :global(.monaco-editor) {
        padding-top: 8px;
    }

    /* prose 스타일 오버라이드 */
    :global(.prose) {
        max-width: none;
    }
    :global(.prose pre) {
        margin: 0;
        border-radius: 0.375rem;
    }
</style> 