<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// PLUXITY Playground
// 아래의 과제를 수행해보세요!

// 1. WebGL 컨테이너를 선택하고 엔진을 초기화하세요
const container = document.getElementById('webGLContainer');

// 2. 엔진 초기화
Px.Core.Initialize(container, () => {
    // 여기에 코드를 작성하세요
});`;

    let editor;
    let container;
    let currentTask = 0;
    
    const tasks = [
        {
            title: "기본 과제 1: 엔진 초기화와 FBX 로드",
            description: "엔진을 초기화하고 drawing.FBX 파일을 로드하세요.",
            hints: [
                "Px.Core.Initialize로 엔진을 초기화하세요",
                "Px.Loader.LoadFbx 함수를 사용하여 FBX를 로드하세요",
                "콜백 함수를 통해 로드 완료/실패 처리를 구현하세요"
            ]
        },
        {
            title: "기본 과제 2: POI 추가와 데이터 관리",
            description: "마우스로 POI를 추가하고, JSON 데이터에서 POI를 로드하세요.",
            hints: [
                "Px.Poi.AddByMouse()로 마우스 클릭 이벤트를 활성화하세요",
                "Px.Poi.AddFromDataArray()로 POI 데이터를 로드하세요",
                "localStorage에서 POI 데이터를 관리하세요"
            ]
        },
        {
            title: "심화 과제 1: 이벤트 처리와 카메라 제어",
            description: "POI 클릭 이벤트를 처리하고 카메라를 제어하세요.",
            hints: [
                "Px.Event.AddEventListener로 POI 클릭 이벤트를 감지하세요",
                "Px.Camera.MoveToPoi로 카메라를 이동시키세요",
                "카메라 이동에 애니메이션을 적용하세요"
            ]
        },
        {
            title: "심화 과제 2: 모델과 빌딩 통합",
            description: "3D 모델을 로드하고 POI와 연동하세요.",
            hints: [
                "Px.Model을 사용하여 3D 모델을 관리하세요",
                "빌딩 FBX와 POI를 연동하세요",
                "모델 클릭 이벤트를 처리하세요"
            ]
        },
        {
            title: "최종 과제: 종합 애플리케이션",
            description: "지금까지 배운 모든 기능을 통합하여 하나의 애플리케이션을 만드세요.",
            hints: [
                "FBX 로드, POI 관리, 이벤트 처리를 모두 구현하세요",
                "카메라 컨트롤과 모델 인터랙션을 추가하세요",
                "사용자 경험을 고려한 UI/UX를 구현하세요"
            ]
        }
    ];

    onMount(() => {
        if (container) {
            const webGLContainer = document.createElement('div');
            webGLContainer.id = 'webGLContainer';
            webGLContainer.style.width = '100%';
            webGLContainer.style.height = '100%';
            container.appendChild(webGLContainer);
        }

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

    function runCode() {
        if (editor) {
            const code = editor.getValue();
            try {
                container.innerHTML = '';
                const webGLContainer = document.createElement('div');
                webGLContainer.id = 'webGLContainer';
                webGLContainer.style.width = '100%';
                webGLContainer.style.height = '100%';
                container.appendChild(webGLContainer);
                
                eval(code);
            } catch (error) {
                console.error('코드 실행 중 오류:', error);
                alert('코드 실행 중 오류가 발생했습니다. 콘솔을 확인하세요.');
            }
        }
    }

    function nextTask() {
        if (currentTask < tasks.length - 1) {
            currentTask++;
        }
    }

    function prevTask() {
        if (currentTask > 0) {
            currentTask--;
        }
    }
</script>

<div class="h-[calc(100vh-4rem)] flex">
    <!-- 왼쪽: WebGL 컨테이너 -->
    <div class="flex-1 bg-black" bind:this={container}></div>
    
    <!-- 오른쪽: 코드 에디터와 과제 설명 -->
    <div class="w-[600px] flex flex-col bg-gray-900">
        <!-- 과제 설명 -->
        <div class="p-4 bg-gray-800 text-white">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold">{tasks[currentTask].title}</h2>
                <div class="space-x-2">
                    <button 
                        class="px-3 py-1 bg-gray-700 rounded hover:bg-gray-600 disabled:opacity-50"
                        on:click={prevTask}
                        disabled={currentTask === 0}
                    >
                        이전
                    </button>
                    <button 
                        class="px-3 py-1 bg-gray-700 rounded hover:bg-gray-600 disabled:opacity-50"
                        on:click={nextTask}
                        disabled={currentTask === tasks.length - 1}
                    >
                        다음
                    </button>
                </div>
            </div>
            <p class="mb-4">{tasks[currentTask].description}</p>
            <div class="bg-gray-700 p-3 rounded">
                <p class="font-semibold mb-2">💡 힌트:</p>
                <ul class="list-disc pl-5 space-y-1">
                    {#each tasks[currentTask].hints as hint}
                        <li>{hint}</li>
                    {/each}
                </ul>
            </div>
        </div>

        <!-- 코드 에디터 -->
        <div class="flex-1 relative">
            <div id="codeEditor" class="absolute inset-0"></div>
        </div>

        <!-- 실행 버튼 -->
        <div class="p-4 bg-gray-800 border-t border-gray-700">
            <button 
                class="w-full py-2 bg-[#04AA6D] hover:bg-[#059862] text-white rounded font-bold"
                on:click={runCode}
            >
                실행하기
            </button>
        </div>
    </div>
</div> 