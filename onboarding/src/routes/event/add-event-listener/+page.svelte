<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하세요
const container = document.getElementById('webGLContainer');

// 2. 엔진을 초기화하고 FBX를 로드한 후
// 3. localStorage에서 POI 데이터를 가져와서 assignYn이 'Y'인 POI들을 표시하세요
// 4. 이벤트를 활성화하고 더블클릭 이벤트를 등록하세요`;
  
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
                            
                            // 이벤트 활성화
                            Px.Event.On();
                            
                            // POI 더블클릭 이벤트 등록
                            Px.Event.AddEventListener('dblclick', 'poi', (poiInfo) => {
                                console.log('POI 더블클릭:', poiInfo);
                                alert('이벤트 호출!');
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
  
    const implementationCode = `// Px.Event 구현
Event: {
    On: function () {
        core.event.on();
    },
    Off: function () {
        core.event.off();
    },
    AddEventListener: function (action, type, func) {
        core.event.addEventListener(action, type, func);
    }
}

// core.event.addEventListener 구현
addEventListener(action, type, func) {
    if (this.callbacks.hasOwnProperty(type)) {
        if (this.callbacks[type].hasOwnProperty(action)) {
            const existCheck = this.callbacks[type][action].indexOf(func);
            if (existCheck === -1) {
                this.callbacks[type][action].push(func);
            }
        }
    }
}`;
  
    const functionSignature = `AddEventListener(
    action: 'click' | 'dblclick' | 'mouseover' | 'mouseout',
    type: 'poi' | 'model',
    callback: (info: any) => void
): void;`;
  
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
        <h1 class="text-4xl font-bold mb-6">이벤트 리스너 추가하기</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                AddEventListener 함수를 사용하면 POI나 모델에 대한 이벤트 핸들러를 등록할 수 있습니다.
                이벤트를 등록하기 전에 반드시 Event.On()을 호출하여 이벤트 시스템을 활성화해야 합니다.
            </p>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">함수 시그니처</h2>
            <pre><code class="language-typescript">{functionSignature}</code></pre>
            
            <div class="mt-4 space-y-2">
                <h3 class="font-semibold">매개변수</h3>
                <ul class="list-disc pl-6 space-y-2 text-gray-600">
                    <li><code>action</code>: 이벤트 유형 (click, dblclick, mouseover, mouseout)</li>
                    <li><code>type</code>: 대상 유형 (poi, model)</li>
                    <li><code>callback</code>: 이벤트 발생 시 호출될 콜백 함수</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <pre><code class="language-javascript">{implementationCode}</code></pre>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 이벤트 처리 과정을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>이벤트 시스템을 활성화/비활성화할 수 있습니다.</li>
                    <li>이벤트 핸들러가 중복 등록되지 않도록 체크합니다.</li>
                    <li>type과 action에 따라 적절한 콜백을 호출합니다.</li>
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
                        2. localStorage에서 POI 데이터를 가져와서 표시하세요.<br>
                        3. Event.On()으로 이벤트를 활성화하세요.<br>
                        4. POI 더블클릭 이벤트를 등록하세요.
                    </p>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <p class="text-yellow-700">
                        💡 이벤트 리스너는 Event.On() 호출 이후에만 동작합니다.
                        POI나 모델이 화면에 표시된 이후에 이벤트를 등록하는 것이 좋습니다.
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
                    이벤트 리스너를 등록하기 전에 반드시 Event.On()을 호출해야 합니다.
                    이벤트가 더 이상 필요하지 않을 때는 Event.Off()를 호출하여 이벤트 시스템을 비활성화할 수 있습니다.
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