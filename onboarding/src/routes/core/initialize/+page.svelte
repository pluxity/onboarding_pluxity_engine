<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';
  
    const defaultCode = `// 1. WebGL 컨테이너를 선택하세요
const container = document.getElementById('webGLContainer');

// 2. 엔진을 초기화하세요
// 3. 초기화 완료 후 콘솔에 메시지를 출력하세요`;
  
    const sampleCode = `// WebGL 컨테이너 선택
const container = document.getElementById('webGLContainer');

// 엔진 초기화
Px.Core.Initialize(container, () => {
    console.log('엔진 초기화 완료!');
});`;
  
    const implementationCode = `Initialize: function (container, callback) {
    try {
        // 컨테이너 유효성 검사
        if (!container || !(container instanceof HTMLElement)) {
            throw 'Px.Core.Initialize: 유효하지 않은 컨테이너입니다.';
        }

        // WebGL 컨텍스트 생성
        const canvas = document.createElement('canvas');
        canvas.style.width = '100%';
        canvas.style.height = '100%';
        container.appendChild(canvas);

        const gl = canvas.getContext('webgl2');
        if (!gl) {
            throw 'Px.Core.Initialize: WebGL2를 지원하지 않는 브라우저입니다.';
        }

        // 엔진 초기화
        core = new PxCore(gl);
        core.init();

        // 콜백 실행
        if (callback && typeof callback === 'function') {
            callback();
        }

    } catch (e) {
        console.error(e);
        throw e;
    }
}`;
  
    const functionSignature = `Initialize(
    container: HTMLElement,
    callback?: () => void
): void;`;
  
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

            // 창기 WebGL 컨테이너 설정
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
  
<svelte:head>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.36.1/min/vs/editor/editor.main.min.css">
</svelte:head>
  
<div class="py-4">
    <article class="prose prose-lg max-w-none">
        <h1 class="text-4xl font-bold mb-6">Px.Core.Initialize</h1>
  
        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                Px 엔진을 초기화하고 WebGL 컨텍스트를 생성합니다.
                이 함수는 엔진을 사용하기 전에 반드시 호출되어야 하며, 초기화가 완료되면 콜백을 통해 알림을 받을 수 있습니다.
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
                    <li><code>container</code>: WebGL 컨텍스트를 생성할 DOM 요소</li>
                    <li><code>callback</code>: 초기화 완료 후 호출될 콜백 함수</li>
                </ul>
            </div>
        </section>
  
        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">엔진 내부 구조</h2>
            <div class="bg-gray-800 text-white p-4 rounded-lg overflow-x-auto">
                <pre><code>{@html implementationCode}</code></pre>
            </div>
            <div class="mt-4 text-gray-600">
                <p>위 코드는 Px 엔진의 초기화 과정을 보여줍니다:</p>
                <ul class="list-disc pl-6 space-y-2">
                    <li>컨테이너의 유효성을 검사합니다.</li>
                    <li>WebGL 캔버스를 생성하고 컨테이너에 추가합니다.</li>
                    <li>WebGL2 컨텍스트를 생성합니다.</li>
                    <li>엔진 코어를 초기화합니다.</li>
                    <li>초기화가 완료되면 콜백을 실행합니다.</li>
                </ul>
            </div>
        </section>
  
        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">실습하기</h2>
            
            <div class="mb-4 space-y-4 text-gray-600">
                <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
                    <p class="text-blue-700">
                        <strong>도전과제:</strong><br>
                        1. WebGL 컨테이너를 선택하는 코드를 작성해보세요.<br>
                        2. Px.Core.Initialize를 호출하여 엔진을 초기화해보세요.<br>
                        3. 초기화 완료 시 콘솔에 메시지를 출력해보세요.
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
                    Initialize 함수는 반드시 DOM이 완전히 로드된 후에 호출되어야 합니다.
                    또한, WebGL2를 지원하지 않는 브라우저에서는 오류가 발생할 수 있습니다.
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