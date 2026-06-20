<template>
  <div class="min-h-screen bg-slate-900 text-slate-200">
    <header class="border-b border-slate-700 px-6 py-4 flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-bold text-cyan-400">药物分子 3D 结构与 ADMET 性质预测</h1>
        <p class="text-sm text-slate-500 mt-1">SMILES解析 · Three.js球棍模型 · ADMET预测 · Tanimoto相似搜索</p>
      </div>
      <button @click="store.toggleDashboard()"
        class="px-4 py-2 rounded-lg text-sm font-bold transition-all border"
        :class="store.showDashboard ? 'bg-cyan-600 border-cyan-500 text-white hover:bg-cyan-500' : 'bg-slate-800 border-slate-600 text-slate-300 hover:bg-slate-700'">
        {{ store.showDashboard ? '隐藏看板' : '显示看板' }}
      </button>
    </header>

    <div v-if="store.showDashboard" class="p-4">
      <ResearchDashboard />
    </div>

    <div class="flex flex-col lg:flex-row gap-4 p-4">
      <div class="lg:w-1/4 space-y-4">
        <MoleculeSearch />
      </div>
      <div class="lg:w-1/2 space-y-4">
        <MoleculeViewer />
      </div>
      <div class="lg:w-1/4 space-y-4">
        <AdmetPanel />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { useMoleculeStore } from './store/molecule'
import MoleculeViewer from './components/MoleculeViewer.vue'
import AdmetPanel from './components/AdmetPanel.vue'
import MoleculeSearch from './components/MoleculeSearch.vue'
import ResearchDashboard from './components/ResearchDashboard.vue'

const store = useMoleculeStore()
onMounted(() => store.loadMolecules())
</script>
