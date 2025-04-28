<script setup>
import { ref } from 'vue';
import ArrayVisualizer from './components/visualizers/ArrayVisualizer.vue';
import StackVisualizer from './components/visualizers/StackVisualizer.vue';
import QueueVisualizer from './components/visualizers/QueueVisualizer.vue';
import TreeVisualizer from './components/visualizers/TreeVisualizer.vue';
import GraphVisualizer from './components/visualizers/GraphVisualizer.vue';

const activeStructure = ref('array');

const structures = [
  { id: 'array', name: '数组', icon: 'fa-table-cells' },
  { id: 'stack', name: '栈', icon: 'fa-layer-group' },
  { id: 'queue', name: '队列', icon: 'fa-arrow-right-arrow-left' },
  { id: 'tree', name: '二叉树', icon: 'fa-diagram-project' },
  { id: 'graph', name: '图', icon: 'fa-share-nodes' }
];
</script>

<template>
  <div class="container">
    <header>
      <h1>数据结构交互式可视化平台</h1>
      <div class="structure-selector">
        <button
          v-for="structure in structures"
          :key="structure.id"
          :class="{ active: activeStructure === structure.id }"
          @click="activeStructure = structure.id"
        >
          <i class="fa-solid" :class="structure.icon"></i>
          <span>{{ structure.name }}</span>
        </button>
      </div>
    </header>

    <main>
      <div class="visualizer-container">
        <ArrayVisualizer v-if="activeStructure === 'array'" />
        <StackVisualizer v-else-if="activeStructure === 'stack'" />
        <QueueVisualizer v-else-if="activeStructure === 'queue'" />
        <TreeVisualizer v-else-if="activeStructure === 'tree'" />
        <GraphVisualizer v-else-if="activeStructure === 'graph'" />
      </div>
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Microsoft YaHei', Arial, sans-serif;
  background-color: var(--neutral-100);
  color: var(--neutral-800);
  background-image: linear-gradient(135deg, var(--neutral-100) 0%, var(--neutral-200) 100%);
  min-height: 100vh;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

header {
  text-align: center;
  margin-bottom: 2rem;
}

h1 {
  font-size: 2.5rem;
  margin-bottom: 1.5rem;
  color: var(--primary-dark);
  font-weight: 700;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-fill-color: transparent;
}

.structure-selector {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-bottom: 2rem;
}

.structure-selector button {
  padding: 0.75rem 1.25rem;
  border: none;
  border-radius: var(--radius-lg);
  background-color: var(--neutral-200);
  color: var(--neutral-700);
  cursor: pointer;
  transition: all var(--transition-normal);
  font-size: 1rem;
  min-width: 120px;
}

.structure-selector button i {
  font-size: 1.25rem;
  margin-right: 0.5rem;
}

.structure-selector button:hover {
  background-color: var(--neutral-300);
  transform: translateY(-2px);
}

.structure-selector button.active {
  background-color: var(--primary-color);
  color: white;
  box-shadow: var(--shadow-md);
}

.visualizer-container {
  background-color: white;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  padding: 2rem;
  min-height: 600px;
  border: 1px solid var(--neutral-200);
  position: relative;
  overflow: hidden;
}
</style>
