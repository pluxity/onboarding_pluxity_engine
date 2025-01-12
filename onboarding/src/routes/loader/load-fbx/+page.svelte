<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하고 엔진을 초기화하세요
const container = document.getElementById('webGLContainer');
// 2. Px.Core.Initialize로 엔진을 초기화한 후
// 3. FBX 파일을 로드하세요`;
  
    const sampleCode = `// WebGL 컨테이너 선택
const container = document.getElementById('webGLContainer');

// 엔진 초기화 및 FBX 로드
Px.Core.Initialize(container, () => {
    // FBX 파일 로드
    Px.Loader.LoadFbx({
        url: '/drawing.FBX',
        onLoad(url) {
            console.log('FBX 파일 로드 완료:', url);
        },
        onError(error) {
            console.error('FBX 파일 로드 실패:', error);
        }
    });
});`;
  
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
            core.camera.fitToObject(object);

            if (option.onLoad) {
                option.onLoad(option.url);
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
    onLoad?: (url: string) => void;
    onError?: (error: any) => void;
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

        // Monaco 에디터 초기화
        loader.init().then(monaco => {
            editor = monaco.editor.create(document.getElementById('codeEditor'), {
                value: defaultCode,
                language: 'javascript',
                theme: 'vs-dark',
                automaticLayout: true,
                minimap: {
                    enabled: false
                }
            });
        });

        return () => {
            if (editor) {
                editor.dispose();
            }
        };
    });

    // 예제 코드 불러오기
    function loadSample() {
        if (editor) {
            editor.setValue(sampleCode);
        }
    }

    // 코드 실행
    function runCode() {
        if (editor) {
            const code = editor.getValue();
            try {
                // WebGL 컨테이너 초기화
                container.innerHTML = '';
                const webGLContainer = document.createElement('div');
                webGLContainer.id = 'webGLContainer';
                webGLContainer.style.width = '100%';
                webGLContainer.style.height = '100%';
                container.appendChild(webGLContainer);
                
                // 코드 실행
                eval(code);
            } catch (error) {
                console.error('코드 실행 중 오류:', error);
                alert('코드 실행 중 오류가 발생했습니다. 콘솔을 확인하세요.');
            }
        }
    }
</script>

<div class="py-4">
    <article class="prose prose-lg max-w-none">
        <h1 class="text-4xl font-bold mb-6">FBX 파일 로드하기</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                이 예제에서는 FBX 파일을 로드하는 기본적인 방법을 학습합니다.
                Px.Loader.LoadFbx 함수를 사용하여 3D 모델을 로드하고 화면에 표시하는 방법을 알아봅니다.
            </p>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">함수 시그니처</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{functionSignature}</code></pre>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{implementationCode}</code></pre>
            </div>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 Px 엔진의 FBX 로더 구현을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>Core 모듈이 초기화되었는지 확인합니다.</li>
                    <li>옵션의 유효성을 검사합니다.</li>
                    <li>FBX 파일을 비동기적으로 로드합니다.</li>
                    <li>로드된 객체를 씬에 추가합니다.</li>
                    <li>카메라를 객체에 맞게 조정합니다.</li>
                </ul>
            </div>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">도전 과제</h2>
            <div class="mb-4 space-y-4 text-gray-600">
                <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
                    <div class="mt-2 space-y-4">
                        <div>
                            <p class="font-semibold">1️⃣ 엔진 초기화 및 FBX 로드</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>WebGL 컨테이너 선택</li>
                                <li>Px.Core.Initialize로 엔진 초기화</li>
                                <li>drawing.FBX 파일 로드</li>
                            </ul>
                        </div>
                        
                        <div>
                            <p class="font-semibold">2️⃣ 콜백 함수 구현</p>
                            <ul class="list-disc ml-6 mt-1">
                                <li>onLoad 콜백에서 성공 메시지 출력</li>
                                <li>onError 콜백에서 에러 처리</li>
                            </ul>
                        </div>
                    </div>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <div class="text-yellow-700">
                        <strong>💡 주의사항:</strong>
                        <ul class="list-disc ml-6 mt-2">
                            <li>drawing.FBX 파일이 public 디렉토리에 있어야 합니다.</li>
                            <li>Core 모듈이 초기화되어야 FBX를 로드할 수 있습니다.</li>
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

                <div class="p-4 bg-gray-900">
                    <p class="text-gray-400 mb-2">WebGL 컨테이너:</p>
                    <div 
                        bind:this={container} 
                        class="border-2 border-dashed border-gray-700 h-[500px] rounded-lg bg-black"
                    ></div>
                </div>
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