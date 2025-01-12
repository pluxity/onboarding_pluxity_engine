<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하세요
const container = document.getElementById('webGLContainer');

// 2. 엔진을 초기화하고 FBX를 로드한 후
// 3. localStorage에서 POI 데이터를 가져와서 assignYn이 'Y'인 POI들을 찾으세요
// 4. AddFromDataArray로 POI들을 한번에 추가하고 Show로 표시하세요`;
  
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
                
                // 배치 완료된 POI 찾기 (assignYn === 'Y')
                const assignedPois = poiData.filter(poi => poi.property.assignYn === 'Y');
                if (assignedPois.length > 0) {
                    // POI 일괄 추가
                    Px.Poi.AddFromDataArray(
                        assignedPois,
                        () => {
                            console.log('POI 일괄 추가 완료');
                            // 모든 POI 표시
                            assignedPois.forEach(poi => {
                                Px.Poi.Show(poi.id);
                            });
                        },
                        (error) => {
                            console.error('POI 추가 실패:', error);
                        }
                    );
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
  
    const implementationCode = `// Px.Poi.AddFromDataArray 구현
AddFromDataArray: async function (dataArray, onComplete, onError) {
    try {
        await core.poiMgr.addPoiArray(dataArray, onComplete);
    } catch (e) {
        if (onError) {
            onError(e);
        }
        console.error(e);
    }
}

// core.poiMgr.addPoiArray 구현
async addPoiArray(dataArray, callback) {
    for (let i = 0; i < dataArray.length; i++) {
        let rotation = dataArray[i].rotation;
        let scale = { x: 100, y: 100, z: 100 };
        
        if (dataArray[i].scale) {
            scale = dataArray[i].scale;
            scale.x = MathUtils.clamp(scale.x, 0.01, scale.x);
            scale.y = MathUtils.clamp(scale.y, 0.01, scale.y);
            scale.z = MathUtils.clamp(scale.z, 0.01, scale.z);
        }
        
        if (!rotation) {
            rotation = { x: 0, y: 0, z: 0 };
        }

        await this.addPoi(
            dataArray[i].type,
            dataArray[i].url,
            dataArray[i].id,
            dataArray[i].group,
            dataArray[i].position.x,
            dataArray[i].position.y,
            dataArray[i].position.z,
            rotation.x,
            rotation.y,
            rotation.z,
            scale.x,
            scale.y,
            scale.z,
            dataArray[i].iconUrl,
            dataArray[i].lineHeight,
            dataArray[i].displayText,
            dataArray[i].property
        );
    }

    if (callback) {
        callback();
    }
}`;
  
    const functionSignature = `AddFromDataArray(
    dataArray: Array<{
        type: string;
        url: string;
        id: number | string;
        group: string;
        position: { x: number; y: number; z: number };
        rotation?: { x: number; y: number; z: number };
        scale?: { x: number; y: number; z: number };
        iconUrl: string;
        lineHeight: number;
        displayText: string;
        property: any;
    }>,
    onComplete?: () => void,
    onError?: (error: any) => void
): Promise<void>;`;
  
    let editor;
    let webGLContainer;
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
            if (webGLContainer) {
                const container = document.createElement('div');
                container.id = 'webGLContainer';
                container.style.width = '100%';
                container.style.height = '100%';
                webGLContainer.appendChild(container);
            }

            window.addEventListener('resize', () => {
                editor?.layout();
            });
        } catch (error) {
            console.error('Monaco Editor 로드 실패:', error);
        }
    });
  
    function runCode() {
        if (!webGLContainer) return;
        
        try {
            const code = editor?.getValue() || '';
            eval(code);
        } catch (error) {
            console.error('코드 실행 실패:', error);
        }
    }
  
    function loadSample() {
        if (editor) {
            editor.setValue(sampleCode);
            showSample = true;
        }
    }
</script>

<div class="py-4">
    <article class="prose prose-lg max-w-none">
        <h1 class="text-4xl font-bold mb-6">POI 일괄 추가하기</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                AddFromDataArray 함수를 사용하면 여러 POI를 한 번에 추가할 수 있습니다.
                이미 배치된 POI들(assignYn이 'Y'인)을 화면에 표시할 때 유용합니다.
            </p>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">함수 시그니처</h2>
            <pre><code class="language-typescript">{functionSignature}</code></pre>
            
            <div class="mt-4 space-y-2">
                <h3 class="font-semibold">매개변수</h3>
                <ul class="list-disc pl-6 space-y-2 text-gray-600">
                    <li><code>dataArray</code>: POI 데이터 배열</li>
                    <li><code>onComplete</code>: 모든 POI 추가가 완료된 후 호출될 콜백 함수</li>
                    <li><code>onError</code>: 오류 발생 시 호출될 콜백 함수</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <pre><code class="language-javascript">{implementationCode}</code></pre>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 POI 배열을 처리하는 과정을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>각 POI의 rotation과 scale 값을 확인하고 기본값을 설정합니다.</li>
                    <li>scale이 지정된 경우 최소값(0.01)을 보장합니다.</li>
                    <li>각 POI를 순차적으로 추가합니다.</li>
                    <li>모든 POI 추가가 완료되면 콜백을 호출합니다.</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">예제</h2>

            <div class="mb-4 space-y-4 text-gray-600">
                <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
                    <p class="text-blue-700">
                        <strong>도전과제:</strong><br>
                        1. FBX 파일을 로드하세요.<br>
                        2. localStorage에서 POI 데이터를 가져오세요.<br>
                        3. assignYn이 'Y'인 POI들을 필터링하세요.<br>
                        4. AddFromDataArray로 POI들을 추가하고 Show로 표시하세요.
                    </p>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <p class="text-yellow-700">
                        💡 AddFromDataArray는 여러 POI를 한 번에 추가할 수 있으며, 각 POI의 rotation과 scale은 선택적으로 지정할 수 있습니다.
                        scale을 지정하지 않으면 기본값 {`{x: 100, y: 100, z: 100}`}이 사용됩니다.
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
                        bind:this={webGLContainer} 
                        class="border-2 border-dashed border-gray-700 h-[500px] rounded-lg bg-black"
                    ></div>
                </div>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">주의사항</h2>
            <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                <p class="text-yellow-700">
                    AddFromDataArray 함수를 호출하기 전에 반드시 Px.Core.Initialize가 호출되어 있어야 하며,
                    3D 모델(FBX)이 로드되어 있어야 합니다.
                    POI 추가가 완료되면 각 POI의 ID를 사용하여 Show 함수로 표시해야 합니다.
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