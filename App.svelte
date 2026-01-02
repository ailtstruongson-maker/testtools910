<script lang="ts">
  import { onMount } from 'svelte';
  import { 
    appState, originalData, filterState, processedData, 
    currentView, isProcessing, departmentMap, fileInfo 
  } from './lib/stores/dashboardStore';
  import { getSalesData } from './lib/services/dbService';
  
  import Sidebar from './lib/components/layout/Sidebar.svelte';
  import LandingPageView from './lib/components/views/LandingPageView.svelte';
  import DashboardView from './lib/components/views/DashboardView.svelte';
  import ProcessingLoader from './lib/components/common/ProcessingLoader.svelte';
  import AiAssistant from './lib/components/ai/AiAssistant.svelte';
  import Toast from './lib/components/common/Toast.svelte';
  
  // Lazy load các view nặng
  import TaxCalculatorView from './lib/components/views/tax/TaxCalculatorView.svelte';
  import CouponConverterView from './lib/components/views/CouponConverterView.svelte';
  import DailyChecklistReportView from './lib/components/views/DailyChecklistReportView.svelte';
  import EmployeeEfficiencyView from './lib/components/views/employee-efficiency/EmployeeEfficiencyView.svelte';

  onMount(async () => {
    // Chỉ load dữ liệu từ DB khi khởi động, không load lại khi chuyển tab
    const savedSales = await getSalesData();
    if (savedSales && savedSales.data.length > 0) {
      originalData.set(savedSales.data);
      fileInfo.set({ 
        filename: savedSales.filename, 
        savedAt: savedSales.savedAt.toLocaleString('vi-VN') 
      });
      // Nếu đã có dữ liệu, chuyển thẳng vào dashboard
      appState.set('dashboard');
    }

    if ((window as any).lucide) (window as any).lucide.createIcons();
  });
</script>

<div class="min-h-screen flex bg-slate-50 dark:bg-slate-950 overflow-hidden font-sans">
  <Sidebar>
    <div class="h-screen overflow-y-auto w-full custom-scrollbar bg-slate-50 dark:bg-slate-950">
      <div class="max-w-7xl mx-auto p-4 md:p-8 w-full min-h-full flex flex-col relative">
        
        <!-- TRẠNG THÁI HIỂN THỊ CÁC VIEW -->
        <div class="flex-grow">
          {#if $currentView === 'dashboard'}
            {#if $appState === 'upload'}
              <LandingPageView />
            {:else}
              <!-- DashboardView sẽ tự giữ state trong store, không load lại -->
              <DashboardView />
            {/if}
          {:else if $currentView === 'tax'}
            <TaxCalculatorView />
          {:else if $currentView === 'coupon'}
            <CouponConverterView />
          {:else if $currentView === 'checklist'}
            <DailyChecklistReportView />
          {:else if $currentView === 'efficiency'}
            <EmployeeEfficiencyView />
          {/if}
        </div>

        <footer class="p-10 mt-auto text-center opacity-30">
          <p class="text-[10px] font-black uppercase tracking-[0.3em]">Intelligence Hub 2.0 • Ultra Fast Core</p>
        </footer>
      </div>
    </div>
  </Sidebar>

  <!-- LOADER TOÀN CỤC - LUÔN NẰM TRÊN CÙNG -->
  {#if $appState === 'loading'}
    <ProcessingLoader />
  {/if}

  <AiAssistant />
  <Toast />

  <input type="file" id="main-file-input" class="hidden" accept=".xlsx,.xls" on:change={(e) => {
    const file = e.currentTarget.files?.[0];
    if (file) import('./lib/stores/dashboardStore').then(m => m.triggerFileUpload(file));
  }} />
</div>

<style>
  :global(.custom-scrollbar::-webkit-scrollbar) {
    width: 6px;
  }
  :global(.custom-scrollbar::-webkit-scrollbar-thumb) {
    background: rgba(99, 102, 241, 0.1);
    border-radius: 10px;
  }
</style>