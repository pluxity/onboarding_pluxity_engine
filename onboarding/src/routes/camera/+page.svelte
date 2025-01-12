<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하고 엔진을 초기화하세요
const container = document.getElementById('webGLContainer');
// 2. Px.Core.Initialize로 엔진을 초기화한 후
// 3. building.FBX 파일을 로드하고
// 4. 각 버튼에 이벤트를 연결하세요

// 예시:
document.getElementById('perspectiveBtn').onclick = () => Px.Camera.SetPerspective();
document.getElementById('orthographicBtn').onclick = () => Px.Camera.SetOrthographic();

// 나머지 버튼들에도 알맞은 이벤트를 연결해보세요!`;
  
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

// 3. 버튼 이벤트 연결
const setupButtonEvents = () => {
    // 투영 모드 버튼
    document.getElementById('perspectiveBtn').onclick = () => Px.Camera.SetPerspective();
    document.getElementById('orthographicBtn').onclick = () => Px.Camera.SetOrthographic();

    // 상태 관리 버튼
    document.getElementById('saveStateBtn').onclick = () => {
        const state = Px.Camera.GetState();
        localStorage.setItem('cameraState', JSON.stringify(state));
        console.log('카메라 상태 저장 완료!', localStorage.getItem('cameraState'));
    };
    document.getElementById('loadStateBtn').onclick = () => {
        const state = JSON.parse(localStorage.getItem('cameraState'));
        if (state) Px.Camera.SetState(state);
        console.log('카메라 상태 불러오기 완료!', state);
    };

    // 회전 버튼들
    const rotateButtons = {
        'rotateUpBtn': ['StartRotateUp', 'StopRotateUp'],
        'rotateDownBtn': ['StartRotateDown', 'StopRotateDown'],
        'rotateLeftBtn': ['StartRotateLeft', 'StopRotateLeft'],
        'rotateRightBtn': ['StartRotateRight', 'StopRotateRight']
    };

    Object.entries(rotateButtons).forEach(([btnId, [startFn, stopFn]]) => {
        const btn = document.getElementById(btnId);
        btn.addEventListener('mousedown', () => Px.Camera[startFn]());
        btn.addEventListener('mouseup', () => Px.Camera[stopFn]());
        btn.addEventListener('mouseleave', () => Px.Camera[stopFn]());
    });

    // 정지 버튼
    document.getElementById('stopBtn').onclick = () => Px.Camera.Stop();

    // 줌 버튼
    const zoomInBtn = document.getElementById('zoomInBtn');
    zoomInBtn.addEventListener('mousedown', () => Px.Camera.StartZoomIn());
    zoomInBtn.addEventListener('mouseup', () => Px.Camera.StopZoomIn());
    zoomInBtn.addEventListener('mouseleave', () => Px.Camera.StopZoomIn());

    const zoomOutBtn = document.getElementById('zoomOutBtn');
    zoomOutBtn.addEventListener('mousedown', () => Px.Camera.StartZoomOut());
    zoomOutBtn.addEventListener('mouseup', () => Px.Camera.StopZoomOut());
    zoomOutBtn.addEventListener('mouseleave', () => Px.Camera.StopZoomOut());

    // FPS 모드 버튼
    document.getElementById('fpsBtn').onclick = () => {
        if (Px.Camera.FPS.IsOn()) {
            Px.Camera.FPS.Off();
        } else {
            Px.Camera.FPS.On();
        }
    };
};

// 메인 실행 흐름
const main = async () => {
    try {
        await initializeEngine(container);
        console.log('엔진 초기화 완료');

        await loadBuildingFbx();
        console.log('건물 불러오기 완료!');

        // 버튼 이벤트 설정
        setupButtonEvents();

    } catch (error) {
        console.error('오류 발생:', error);
    }
};

// 실행
main();`;

    let editor;
    let container;
    let showSample = false;

    // 컨트롤 패널 생성
    function createControlPanel() {
        const panel = document.createElement('div');
        panel.className = 'fixed top-1/2 right-4 transform -translate-y-1/2 bg-gray-800 bg-opacity-90 text-white rounded-lg shadow-lg';

        const sections = [
            {
                title: '투영 모드',
                content: `
                    <div class="grid grid-cols-2 gap-2">
                        <button id="perspectiveBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">원근</button>
                        <button id="orthographicBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">직교</button>
                    </div>
                `
            },
            {
                title: '상태 관리',
                content: `
                    <div class="grid grid-cols-2 gap-2">
                        <button id="saveStateBtn" class="px-3 py-2 rounded font-medium text-sm bg-gray-600 hover:bg-gray-700">저장</button>
                        <button id="loadStateBtn" class="px-3 py-2 rounded font-medium text-sm bg-gray-600 hover:bg-gray-700">불러오기</button>
                    </div>
                `
            },
            {
                title: '카메라 회전',
                content: `
                    <div class="grid grid-cols-3 gap-2">
                        <div></div>
                        <button id="rotateUpBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">↑</button>
                        <div></div>
                        <button id="rotateLeftBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">←</button>
                        <button id="stopBtn" class="px-3 py-2 rounded font-medium text-sm bg-red-500 hover:bg-red-600">■</button>
                        <button id="rotateRightBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">→</button>
                        <div></div>
                        <button id="rotateDownBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">↓</button>
                        <div></div>
                    </div>
                `
            },
            {
                title: '줌',
                content: `
                    <div class="grid grid-cols-2 gap-2">
                        <button id="zoomInBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">확대</button>
                        <button id="zoomOutBtn" class="px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">축소</button>
                    </div>
                `
            },
            {
                title: '시점 모드',
                content: `
                    <div>
                        <button id="fpsBtn" class="w-full px-3 py-2 rounded font-medium text-sm bg-gray-600 hover:bg-gray-700">FPS 모드 전환</button>
                    </div>
                `
            }
        ];

        sections.forEach((section, index) => {
            const sectionEl = document.createElement('div');
            sectionEl.className = `p-4 ${index < sections.length - 1 ? 'border-b border-gray-700' : ''}`;
            sectionEl.innerHTML = `
                <h3 class="font-bold mb-3 text-lg">${section.title}</h3>
                ${section.content}
            `;
            panel.appendChild(sectionEl);
        });

        return panel;
    }

    onMount(() => {
        if (container) {
            const webGLContainer = document.createElement('div');
            webGLContainer.id = 'webGLContainer';
            webGLContainer.style.width = '100%';
            webGLContainer.style.height = '100%';
            container.appendChild(webGLContainer);

            // 컨트롤 패널 추가
            const controlPanel = createControlPanel();
            container.appendChild(controlPanel);
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
        <h1 class="text-4xl font-bold mb-6">카메라 제어</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                카메라의 모든 기능을 통합적으로 제어할 수 있습니다.
                각 버튼에 알맞은 이벤트를 연결하여 카메라 기능을 구현해보세요.
            </p>
        </section>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">실습하기</h2>
            
            <div class="mb-4 space-y-4 text-gray-600">
                <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
                    <p class="text-blue-700">
                        <strong>도전과제:</strong>
                    </p>
                    <div class="mt-2 space-y-2">
                        <p>1. 엔진을 초기화하고 건물 모델을 로드하세요.</p>
                        <p>2. 각 버튼에 알맞은 이벤트를 연결하세요:</p>
                        <ul class="list-disc ml-6 mt-2">
                            <li>투영 모드 버튼: SetPerspective, SetOrthographic</li>
                            <li>상태 관리 버튼: GetState/SetState로 상태 저장/복원</li>
                            <li>회전 버튼: Start/StopRotate 함수들로 회전 제어</li>
                            <li>줌 버튼: Start/StopZoom 함수들로 확대/축소</li>
                            <li>FPS 버튼: FPS.On/Off로 모드 전환</li>
                        </ul>
                    </div>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <div class="text-yellow-700">
                        <strong>💡 주의사항:</strong>
                        <ul class="list-disc ml-6 mt-2">
                            <li>회전/줌 버튼은 마우스 이벤트로 구현해보세요.</li>
                            <li>상태 저장 시에는 localStorage를 활용하세요.</li>
                            <li>각 버튼의 ID를 정확히 확인하고 이벤트를 연결하세요.</li>
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
                        class="border-2 border-dashed border-gray-700 h-[500px] rounded-lg bg-black relative"
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

    :global(.prose) {
        max-width: none;
    }
    :global(.prose pre) {
        margin: 0;
        border-radius: 0.375rem;
    }
</style> 