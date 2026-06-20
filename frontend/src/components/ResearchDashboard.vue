<template>
  <div v-if="stats" class="bg-slate-800 rounded-lg p-4 border border-slate-700 space-y-4">
    <h3 class="text-lg font-bold text-cyan-400">药物研究风险看板</h3>

    <div class="grid grid-cols-2 lg:grid-cols-4 gap-3">
      <div class="bg-slate-900 rounded-lg p-3 border border-slate-700">
        <div class="text-xs text-slate-500">总分子数</div>
        <div class="text-2xl font-bold text-cyan-400">{{ stats.totalMolecules }}</div>
      </div>
      <div class="bg-slate-900 rounded-lg p-3 border border-slate-700">
        <div class="text-xs text-slate-500">药物类别</div>
        <div class="text-2xl font-bold text-purple-400">{{ stats.totalCategories }}</div>
      </div>
      <div class="bg-slate-900 rounded-lg p-3 border border-slate-700">
        <div class="text-xs text-slate-500">高风险分子</div>
        <div class="text-2xl font-bold text-red-400">{{ stats.highRiskCount }}</div>
      </div>
      <div class="bg-slate-900 rounded-lg p-3 border border-slate-700">
        <div class="text-xs text-slate-500">紧急/高优先级</div>
        <div class="text-2xl font-bold text-orange-400">{{ (stats?.urgentCount ?? 0) + (stats?.highPriorityCount ?? 0) }}</div>
      </div>
    </div>

    <div>
      <h4 class="text-sm font-bold text-slate-400 mb-2">按药物类别汇总</h4>
      <div class="space-y-2">
        <div v-for="cat in stats.categories" :key="cat.category" class="bg-slate-900 rounded-lg border border-slate-700 overflow-hidden">
          <div @click="toggleCategory(cat.category)" class="cursor-pointer p-3 hover:bg-slate-800 transition">
            <div class="flex items-center justify-between">
              <div class="flex items-center gap-2">
                <span class="text-sm font-bold text-slate-200">{{ cat.category }}</span>
                <span class="text-xs px-1.5 py-0.5 rounded bg-slate-700 text-slate-400">{{ cat.count }} 个分子</span>
              </div>
              <span class="text-slate-500 text-xs">{{ expandedCategories.has(cat.category) ? '收起 ▲' : '展开 ▼' }}</span>
            </div>
            <div class="mt-2 flex gap-1 h-2 rounded-full overflow-hidden">
              <div v-if="cat.highRisk > 0" class="bg-red-500 transition-all" :style="{ width: (cat.highRisk / cat.count * 100) + '%' }" :title="`高风险: ${cat.highRisk}`"></div>
              <div v-if="cat.mediumRisk > 0" class="bg-orange-500 transition-all" :style="{ width: (cat.mediumRisk / cat.count * 100) + '%' }" :title="`中风险: ${cat.mediumRisk}`"></div>
              <div v-if="cat.lowRisk > 0" class="bg-green-500 transition-all" :style="{ width: (cat.lowRisk / cat.count * 100) + '%' }" :title="`低风险: ${cat.lowRisk}`"></div>
            </div>
            <div class="flex gap-3 mt-1 text-xs text-slate-500">
              <span class="text-red-400">高风险 {{ cat.highRisk }}</span>
              <span class="text-orange-400">中风险 {{ cat.mediumRisk }}</span>
              <span class="text-green-400">低风险 {{ cat.lowRisk }}</span>
            </div>
          </div>
          <div v-if="expandedCategories.has(cat.category)" class="border-t border-slate-700 p-2 space-y-1 bg-slate-950">
            <div v-for="mol in cat.molecules" :key="mol.id" @click="store.selectMolecule(mol)"
              class="cursor-pointer p-2 rounded border border-slate-700 bg-slate-900 hover:border-cyan-500 hover:bg-cyan-900/20 transition">
              <div class="flex items-center justify-between">
                <span class="text-sm font-bold text-slate-200">{{ mol.name }}</span>
                <div class="flex gap-1">
                  <span class="text-xs px-1.5 py-0.5 rounded" :class="getPriorityClass(mol.priority)">{{ mol.priority }}</span>
                  <span class="text-xs px-1.5 py-0.5 rounded" :class="getRiskClass(mol.riskLevel)">{{ mol.riskLevel }}</span>
                </div>
              </div>
              <div class="flex items-center justify-between mt-1">
                <span class="text-xs text-slate-500 font-mono">{{ mol.formula }} · MW {{ mol.mw }}</span>
                <span class="text-xs font-bold" :class="getRiskScoreClass(mol.riskScore)">风险分 {{ mol.riskScore }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div>
      <h4 class="text-sm font-bold text-slate-400 mb-2">待重点分析对象队列</h4>
      <div v-if="stats.priorityQueue.length > 0" class="space-y-1">
        <div v-for="(mol, index) in stats.priorityQueue" :key="mol.id" @click="store.selectMolecule(mol)"
          :class="['cursor-pointer p-3 rounded-lg border transition-all',
            store.currentMolecule?.id === mol.id ? 'border-cyan-500 bg-cyan-900/30' : 'border-slate-700 bg-slate-900 hover:border-slate-500']">
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-2">
              <span class="text-xs font-bold text-slate-500 w-5">#{{ index + 1 }}</span>
              <span class="text-sm font-bold text-slate-200">{{ mol.name }}</span>
              <span class="text-xs px-1.5 py-0.5 rounded bg-slate-700 text-slate-400">{{ mol.category }}</span>
            </div>
            <div class="flex gap-1">
              <span class="text-xs px-1.5 py-0.5 rounded" :class="getPriorityClass(mol.priority)">{{ mol.priority }}</span>
              <span class="text-xs px-1.5 py-0.5 rounded" :class="getRiskClass(mol.riskLevel)">{{ mol.riskLevel }}</span>
            </div>
          </div>
          <div class="mt-2">
            <div class="flex items-center justify-between">
              <span class="text-xs text-slate-500">风险分数</span>
              <span class="text-xs font-bold" :class="getRiskScoreClass(mol.riskScore)">{{ mol.riskScore }}/100</span>
            </div>
            <div class="mt-1 h-1.5 bg-slate-700 rounded-full overflow-hidden">
              <div class="h-full rounded-full transition-all" :class="getRiskBarClass(mol.riskScore)" :style="{ width: mol.riskScore + '%' }"></div>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="text-slate-500 text-sm text-center py-4 bg-slate-900 rounded-lg border border-slate-700">
        暂无紧急或高优先级分子
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useMoleculeStore } from '../store/molecule'
import type { PriorityLevel, RiskLevel } from '../types'

const store = useMoleculeStore()
const expandedCategories = ref<Set<string>>(new Set())

const stats = computed(() => store.dashboardStats)

function toggleCategory(category: string) {
  if (expandedCategories.value.has(category)) {
    expandedCategories.value.delete(category)
  } else {
    expandedCategories.value.add(category)
  }
}

function getPriorityClass(priority: PriorityLevel): string {
  switch (priority) {
    case '紧急': return 'bg-red-900/50 text-red-400 border border-red-700'
    case '高': return 'bg-orange-900/50 text-orange-400 border border-orange-700'
    case '中': return 'bg-yellow-900/50 text-yellow-400 border border-yellow-700'
    case '低': return 'bg-green-900/50 text-green-400 border border-green-700'
  }
}

function getRiskClass(level: RiskLevel): string {
  switch (level) {
    case '高风险': return 'bg-red-900/50 text-red-400 border border-red-700'
    case '中风险': return 'bg-orange-900/50 text-orange-400 border border-orange-700'
    case '低风险': return 'bg-green-900/50 text-green-400 border border-green-700'
  }
}

function getRiskScoreClass(score: number): string {
  if (score >= 50) return 'text-red-400'
  if (score >= 25) return 'text-orange-400'
  return 'text-green-400'
}

function getRiskBarClass(score: number): string {
  if (score >= 50) return 'bg-red-500'
  if (score >= 25) return 'bg-orange-500'
  return 'bg-green-500'
}
</script>
