<script>
  import '../app.css';
  import { page } from '$app/stores';
  
  let isSidebarOpen = true;
  
  const toggleSidebar = () => {
    isSidebarOpen = !isSidebarOpen;
  };

  $: isPlayground = $page.url.pathname === '/playground';
</script>

<div class="min-h-screen bg-gray-100">
  <!-- 헤더 -->
  <header class="fixed top-0 left-0 right-0 bg-[#04AA6D] text-white shadow-lg z-50">
    <div class="container mx-auto px-4">
      <div class="flex items-center justify-between h-16">
        <div class="flex items-center">
          {#if !isPlayground}
            <button 
              class="md:hidden mr-2" 
              on:click={toggleSidebar}
              aria-label="메뉴 토글"
            >
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
              </svg>
            </button>
          {/if}
          <h1 class="text-xl font-bold">PLUXITY Px Engine</h1>
        </div>
        <nav class="hidden md:flex space-x-4">
          <a href="/" class="hover:bg-[#059862] px-3 py-2 rounded">홈</a>
          <a href="/playground" class="hover:bg-[#059862] px-3 py-2 rounded">플레이그라운드</a>
        </nav>
      </div>
    </div>
  </header>

  <!-- 사이드바와 메인 콘텐츠 -->
  <div class="flex pt-16">
    {#if !isPlayground}
      <!-- 사이드바 -->
      <aside class="{isSidebarOpen ? 'translate-x-0' : '-translate-x-full'} fixed md:relative md:translate-x-0 z-40 w-64 h-[calc(100vh-4rem)] bg-white shadow-lg transition-transform duration-300 ease-in-out overflow-y-auto">
        <nav class="p-4">
          <div class="space-y-8">
            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Core
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/core/initialize" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  Initialize
                </a>
              </div>
            </div>

            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Loader
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/loader/load-fbx" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  Load FBX
                </a>
              </div>
            </div>

            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Poi
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/poi/add-by-mouse" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  AddByMouse
                </a>
                <a href="/poi/add-from-data-array" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  AddFromDataArray
                </a>
              </div>
            </div>

            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Event
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/event/add-event-listener" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  AddEventListener
                </a>
              </div>
            </div>

            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Model
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/model" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  Model
                </a>
              </div>
            </div>

            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Camera
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/camera" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  Camera
                </a>
                <a href="/building/move-to-poi" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  MoveToPoi
                </a>
              </div>
            </div>

            <div>
              <h3 class="px-3 text-sm font-semibold text-gray-500 uppercase tracking-wider">
                Px.Building
              </h3>
              <div class="mt-2 space-y-1">
                <a href="/building/load-fbx" class="flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50 hover:text-gray-900">
                  Load Building FBX
                </a>
              </div>
            </div>
          </div>
        </nav>
      </aside>
    {/if}

    <!-- 메인 콘텐츠 -->
    <main class="flex-1 {isPlayground ? '' : 'p-4 md:ml-64'}">
      <div class="{isPlayground ? '' : 'max-w-6xl mx-auto'}">
        <slot />
      </div>
    </main>
  </div>
</div>
