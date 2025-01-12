<script>
    import { onMount } from 'svelte';
    import loader from '@monaco-editor/loader';

    const defaultCode = `// 1. WebGL 컨테이너를 선택하고 엔진을 초기화하세요
const container = document.getElementById('webGLContainer');
// 2. Px.Core.Initialize로 엔진을 초기화한 후
// 3. building.FBX 파일을 로드하고
// 4. 모델의 투명도와 가시성을 제어하는 기능을 구현해보세요

// 예시:
// 투명도 설정 (0~100 사이의 값)
Px.Model.Transparent.Set('모델ID', 50);  // 50% 투명도
// 모든 모델 투명도 설정
Px.Model.Transparent.SetAll(30);  // 30% 투명도
// 투명도 초기화
Px.Model.Transparent.Restore();

// 가시성 제어
Px.Model.Visible.Hide('모델ID');  // 특정 모델 숨기기
Px.Model.Visible.Show('모델ID');  // 특정 모델 보이기
// 여러 모델 동시 제어
Px.Model.Visible.HideByArray(['모델ID1', '모델ID2']);
Px.Model.Visible.ShowByArray(['모델ID1', '모델ID2']);
// 전체 모델 제어
Px.Model.Visible.HideAll();
Px.Model.Visible.ShowAll();`;
  
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

// 3. 모델 계층 구조 가져오기
const getModelHierarchy = () => {
    const hierarchy = Px.Model.Visible.GetHierarchy();
    console.log('모델 계층 구조:', hierarchy);
    return hierarchy;
};

// 4. 투명도 제어 함수들
const transparencyControls = {
    // 선택한 모델 투명도 설정
    setTransparency: (id, opacity) => {
        Px.Model.Transparent.Set(id, opacity);
        console.log(\`모델 \${id}의 투명도를 \${opacity}%로 설정\`);
    },
    // 전체 모델 투명도 설정
    setAllTransparency: (opacity) => {
        Px.Model.Transparent.SetAll(opacity);
        console.log(\`전체 모델의 투명도를 \${opacity}%로 설정\`);
    },
    // 투명도 초기화
    resetTransparency: () => {
        Px.Model.Transparent.Restore();
        console.log('모든 모델의 투명도를 초기화');
    }
};

// 5. 가시성 제어 함수들
const visibilityControls = {
    // 단일 모델 표시/숨김
    showModel: (id) => {
        Px.Model.Visible.Show(id);
        console.log(\`모델 \${id} 표시\`);
    },
    hideModel: (id) => {
        Px.Model.Visible.Hide(id);
        console.log(\`모델 \${id} 숨김\`);
    },
    // 여러 모델 동시 제어
    showModels: (idList) => {
        Px.Model.Visible.ShowByArray(idList);
        console.log('선택한 모델들을 표시:', idList);
    },
    hideModels: (idList) => {
        Px.Model.Visible.HideByArray(idList);
        console.log('선택한 모델들을 숨김:', idList);
    },
    // 전체 모델 제어
    showAllModels: () => {
        Px.Model.Visible.ShowAll();
        console.log('모든 모델을 표시');
    },
    hideAllModels: () => {
        Px.Model.Visible.HideAll();
        console.log('모든 모델을 숨김');
    }
};

// 6. 이벤트 연결
const setupModelControls = () => {
    // 투명도 조절 이벤트
    document.getElementById('transparencyRange').addEventListener('input', (e) => {
        const opacity = parseInt(e.target.value);
        const modelId = document.getElementById('modelSelect').value;
        if (modelId === 'all') {
            transparencyControls.setAllTransparency(opacity);
        } else {
            transparencyControls.setTransparency(modelId, opacity);
        }
    });

    document.getElementById('resetTransparencyBtn').onclick = () => {
        transparencyControls.resetTransparency();
        document.getElementById('transparencyRange').value = 100;
    };

    // 가시성 제어 이벤트
    document.getElementById('hideModelBtn').onclick = () => {
        const modelId = document.getElementById('modelSelect').value;
        if (modelId === 'all') {
            visibilityControls.hideAllModels();
        } else {
            visibilityControls.hideModel(modelId);
        }
    };

    document.getElementById('showModelBtn').onclick = () => {
        const modelId = document.getElementById('modelSelect').value;
        if (modelId === 'all') {
            visibilityControls.showAllModels();
        } else {
            visibilityControls.showModel(modelId);
        }
    };
};

// 7. 모델 선택 옵션 업데이트
const updateModelSelect = (floorData) => {
    const select = document.getElementById('modelSelect');
    select.innerHTML = '<option value="all">전체 층</option>';
    
    // 지하층과 지상층 분리
    const basementFloors = floorData.filter(floor => floor.basement === '1')
        .sort((a, b) => b.order - a.order);
    const aboveFloors = floorData.filter(floor => floor.basement === '0')
        .sort((a, b) => a.order - b.order);

    // 지하층 옵션 추가
    basementFloors.forEach(floor => {
        const option = document.createElement('option');
        option.value = floor.name;
        option.textContent = floor.name;
        select.appendChild(option);
    });

    // 지상층 옵션 추가
    aboveFloors.forEach(floor => {
        const option = document.createElement('option');
        option.value = floor.name;
        option.textContent = floor.name;
        select.appendChild(option);
    });
};

// 메인 실행 흐름
const main = async () => {
    try {
        await initializeEngine(container);
        console.log('엔진 초기화 완료');

        const floorData = await loadBuildingFbx();
        console.log('건물 불러오기 완료!', floorData);

        // 층 선택 옵션 업데이트
        updateModelSelect(floorData);

        // 컨트롤 이벤트 설정
        setupModelControls();

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
                title: '층 선택',
                content: `
                    <div class="space-y-2">
                        <select id="modelSelect" class="w-full px-3 py-2 rounded text-sm bg-gray-700">
                            <option value="all">전체 층</option>
                        </select>
                    </div>
                `
            },
            {
                title: '가시성 제어',
                content: `
                    <div class="grid grid-cols-2 gap-2">
                        <button id="hideModelBtn" 
                            class="px-3 py-2 rounded font-medium text-sm bg-red-500 hover:bg-red-600">
                            숨기기
                        </button>
                        <button id="showModelBtn" 
                            class="px-3 py-2 rounded font-medium text-sm bg-green-500 hover:bg-green-600">
                            보이기
                        </button>
                    </div>
                `
            },
            {
                title: '투명도 제어',
                content: `
                    <div class="space-y-2">
                        <input type="range" id="transparencyRange" min="0" max="100" value="100" 
                            class="w-full" />
                        <div class="flex justify-between text-sm">
                            <span>0%</span>
                            <span>50%</span>
                            <span>100%</span>
                        </div>
                        <button id="resetTransparencyBtn" 
                            class="w-full px-3 py-2 rounded font-medium text-sm bg-blue-500 hover:bg-blue-600">
                            투명도 초기화
                        </button>
                    </div>
                `
            },
        ];

        sections.forEach((section, index) => {
            const sectionEl = document.createElement('div');
            sectionEl.className = `p-4 ${index < sections.length - 1 ? 'border-b border-gray-700' : ''}`;
            sectionEl.innerHTML = `
                <h3 class="font-bold mb-3 text-lg text-white">${section.title}</h3>
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
        <h1 class="text-4xl font-bold mb-6">모델 제어</h1>

        <section class="mb-6">
            <h2 class="text-2xl font-bold mb-3">개요</h2>
            <p class="text-gray-600">
                Px.Model을 사용하여 3D 모델의 투명도와 가시성을 제어하는 방법을 학습합니다.
                특정 모델을 선택하여 투명도를 조절하거나, 보이기/숨기기를 제어할 수 있습니다.
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
                        <p>2. 모델의 계층 구조를 가져와서 선택 옵션을 구성하세요.</p>
                        <p>3. 다음 기능들을 구현해보세요:</p>
                        <ul class="list-disc ml-6 mt-2">
                            <li>투명도 제어:
                                <ul class="list-circle ml-6">
                                    <li>특정 모델의 투명도 설정 (0~100%)</li>
                                    <li>전체 모델의 투명도 설정</li>
                                    <li>투명도 초기화</li>
                                </ul>
                            </li>
                            <li>가시성 제어:
                                <ul class="list-circle ml-6">
                                    <li>특정 모델 숨기기/보이기</li>
                                    <li>여러 모델 동시 제어</li>
                                    <li>전체 모델 숨기기/보이기</li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </div>

                <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4">
                    <div class="text-yellow-700">
                        <strong>💡 주의사항:</strong>
                        <ul class="list-disc ml-6 mt-2">
                            <li>투명도는 0~100 사이의 값을 사용하며, 내부적으로 0~1로 변환됩니다.</li>
                            <li>가시성 제어 시 모델 ID를 정확히 지정해야 합니다.</li>
                            <li>계층 구조를 활용하여 모델 ID를 확인하세요.</li>
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