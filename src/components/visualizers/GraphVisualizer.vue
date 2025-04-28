<script setup>
import { ref, onMounted } from 'vue';

// 图数据模型
const nodes = ref([]);
const edges = ref([]);
const message = ref('');
const graphContainer = ref(null);

// 节点和边的输入
const newNodeValue = ref('');
const sourceNode = ref('');
const targetNode = ref('');

// 操作函数
const addNode = () => {
  if (newNodeValue.value.trim() === '') {
    message.value = '请输入有效的节点值';
    return;
  }

  const nodeValue = newNodeValue.value.trim();

  // 检查节点是否已存在
  if (nodes.value.some(node => node.value === nodeValue)) {
    message.value = `节点 ${nodeValue} 已存在`;
    return;
  }

  // 创建新节点，随机位置
  const containerWidth = graphContainer.value ? graphContainer.value.clientWidth : 500;
  const containerHeight = graphContainer.value ? graphContainer.value.clientHeight : 300;

  nodes.value.push({
    id: Date.now().toString(),
    value: nodeValue,
    x: Math.random() * (containerWidth - 100) + 50,
    y: Math.random() * (containerHeight - 100) + 50
  });

  message.value = `添加节点 ${nodeValue} 成功`;
  newNodeValue.value = '';
  renderGraph();
};

const addEdge = () => {
  if (sourceNode.value.trim() === '' || targetNode.value.trim() === '') {
    message.value = '请输入有效的源节点和目标节点';
    return;
  }

  const source = sourceNode.value.trim();
  const target = targetNode.value.trim();

  // 检查节点是否存在
  const sourceExists = nodes.value.some(node => node.value === source);
  const targetExists = nodes.value.some(node => node.value === target);

  if (!sourceExists || !targetExists) {
    message.value = '源节点或目标节点不存在';
    return;
  }

  // 检查边是否已存在
  if (edges.value.some(edge => edge.source === source && edge.target === target)) {
    message.value = `边 ${source} -> ${target} 已存在`;
    return;
  }

  // 添加新边
  edges.value.push({
    id: Date.now().toString(),
    source,
    target
  });

  message.value = `添加边 ${source} -> ${target} 成功`;
  sourceNode.value = '';
  targetNode.value = '';
  renderGraph();
};

const removeNode = (nodeValue) => {
  // 移除节点
  nodes.value = nodes.value.filter(node => node.value !== nodeValue);

  // 移除与该节点相关的所有边
  edges.value = edges.value.filter(edge =>
    edge.source !== nodeValue && edge.target !== nodeValue
  );

  message.value = `删除节点 ${nodeValue} 及其相关边成功`;
  renderGraph();
};

const removeEdge = (source, target) => {
  edges.value = edges.value.filter(edge =>
    !(edge.source === source && edge.target === target)
  );

  message.value = `删除边 ${source} -> ${target} 成功`;
  renderGraph();
};

const clearGraph = () => {
  nodes.value = [];
  edges.value = [];
  message.value = '图已清空';
  renderGraph();
};

// 初始化示例数据
const initExampleData = () => {
  // 清空现有数据
  nodes.value = [];
  edges.value = [];

  // 添加示例节点
  const containerWidth = graphContainer.value ? graphContainer.value.clientWidth : 500;
  const containerHeight = graphContainer.value ? graphContainer.value.clientHeight : 300;

  const nodePositions = [
    { value: 'A', x: containerWidth / 2, y: 50 },
    { value: 'B', x: containerWidth / 4, y: containerHeight / 2 },
    { value: 'C', x: 3 * containerWidth / 4, y: containerHeight / 2 },
    { value: 'D', x: containerWidth / 3, y: containerHeight - 80 },
    { value: 'E', x: 2 * containerWidth / 3, y: containerHeight - 80 }
  ];

  nodePositions.forEach(node => {
    nodes.value.push({
      id: Date.now() + Math.random().toString(),
      value: node.value,
      x: node.x,
      y: node.y
    });
  });

  // 添加示例边
  const edgeConnections = [
    { source: 'A', target: 'B' },
    { source: 'A', target: 'C' },
    { source: 'B', target: 'D' },
    { source: 'C', target: 'E' },
    { source: 'D', target: 'E' }
  ];

  edgeConnections.forEach(edge => {
    edges.value.push({
      id: Date.now() + Math.random().toString(),
      source: edge.source,
      target: edge.target
    });
  });

  message.value = '已初始化示例图';
  renderGraph();
};

// 拖拽功能
let draggedNode = null;

const startDrag = (event, node) => {
  draggedNode = node;
  event.preventDefault();
};

const onDrag = (event) => {
  if (!draggedNode) return;

  const container = graphContainer.value.getBoundingClientRect();
  const x = event.clientX - container.left;
  const y = event.clientY - container.top;

  // 更新节点位置
  const nodeIndex = nodes.value.findIndex(n => n.id === draggedNode.id);
  if (nodeIndex !== -1) {
    nodes.value[nodeIndex].x = Math.max(20, Math.min(container.width - 20, x));
    nodes.value[nodeIndex].y = Math.max(20, Math.min(container.height - 20, y));
    renderGraph();
  }
};

const endDrag = () => {
  draggedNode = null;
};

// 渲染图
const renderGraph = () => {
  if (!graphContainer.value) return;

  // 清空容器
  graphContainer.value.innerHTML = '';

  if (nodes.value.length === 0) {
    const emptyMessage = document.createElement('div');
    emptyMessage.className = 'empty-message';
    emptyMessage.textContent = '图为空，请添加节点和边';
    graphContainer.value.appendChild(emptyMessage);
    return;
  }

  // 创建SVG元素
  const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  svg.setAttribute('width', '100%');
  svg.setAttribute('height', '100%');

  // 获取CSS变量
  const style = getComputedStyle(document.documentElement);
  const primaryColor = style.getPropertyValue('--primary-color').trim() || '#4361ee';
  const secondaryColor = style.getPropertyValue('--secondary-color').trim() || '#f72585';
  const accentColor = style.getPropertyValue('--accent-color').trim() || '#4cc9f0';

  // 添加CSS样式
  const styleElement = document.createElementNS('http://www.w3.org/2000/svg', 'style');
  styleElement.textContent = `
    @keyframes nodeAppear {
      from { opacity: 0; transform: scale(0.5); }
      to { opacity: 1; transform: scale(1); }
    }
    .node {
      animation: nodeAppear 0.5s ease-out;
    }
  `;
  svg.appendChild(styleElement);

  graphContainer.value.appendChild(svg);

  // 绘制边
  edges.value.forEach(edge => {
    const sourceNode = nodes.value.find(node => node.value === edge.source);
    const targetNode = nodes.value.find(node => node.value === edge.target);

    if (sourceNode && targetNode) {
      // 创建边
      const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      line.setAttribute('x1', sourceNode.x);
      line.setAttribute('y1', sourceNode.y);
      line.setAttribute('x2', targetNode.x);
      line.setAttribute('y2', targetNode.y);
      line.setAttribute('stroke', primaryColor);
      line.setAttribute('stroke-width', '2');

      // 添加删除边的点击事件
      line.addEventListener('click', () => removeEdge(edge.source, edge.target));
      line.classList.add('edge');

      // 添加边的提示文本
      const midX = (sourceNode.x + targetNode.x) / 2;
      const midY = (sourceNode.y + targetNode.y) / 2;
      const edgeLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      edgeLabel.setAttribute('x', midX);
      edgeLabel.setAttribute('y', midY - 10);
      edgeLabel.setAttribute('text-anchor', 'middle');
      edgeLabel.setAttribute('fill', primaryColor);
      edgeLabel.setAttribute('font-size', '12');
      edgeLabel.textContent = `${edge.source} → ${edge.target}`;
      edgeLabel.classList.add('edge-label');

      svg.appendChild(line);
      svg.appendChild(edgeLabel);

      // 创建箭头
      const angle = Math.atan2(targetNode.y - sourceNode.y, targetNode.x - sourceNode.x);
      const arrowSize = 10;
      const arrowX = targetNode.x - 20 * Math.cos(angle);
      const arrowY = targetNode.y - 20 * Math.sin(angle);

      const arrowMarker = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
      const x1 = arrowX - arrowSize * Math.cos(angle - Math.PI / 6);
      const y1 = arrowY - arrowSize * Math.sin(angle - Math.PI / 6);
      const x2 = arrowX - arrowSize * Math.cos(angle + Math.PI / 6);
      const y2 = arrowY - arrowSize * Math.sin(angle + Math.PI / 6);

      arrowMarker.setAttribute('points', `${arrowX},${arrowY} ${x1},${y1} ${x2},${y2}`);
      arrowMarker.setAttribute('fill', primaryColor);
      arrowMarker.classList.add('arrow');
      svg.appendChild(arrowMarker);

      // 创建边标签
      const textX = (sourceNode.x + targetNode.x) / 2;
      const textY = (sourceNode.y + targetNode.y) / 2 - 10;
      const edgeText = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      edgeText.setAttribute('x', textX);
      edgeText.setAttribute('y', textY);
      edgeText.setAttribute('text-anchor', 'middle');
      edgeText.setAttribute('fill', '#666');
      edgeText.setAttribute('font-size', '12');
      edgeText.textContent = `${edge.source} → ${edge.target}`;
      svg.appendChild(edgeText);
    }
  });

  // 绘制节点
  nodes.value.forEach(node => {
    // 创建节点组
    const group = document.createElementNS('http://www.w3.org/2000/svg', 'g');
    group.classList.add('node-group');

    // 创建节点阴影
    const shadow = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    shadow.setAttribute('cx', node.x + 2);
    shadow.setAttribute('cy', node.y + 2);
    shadow.setAttribute('r', 22);
    shadow.setAttribute('fill', 'rgba(0, 0, 0, 0.2)');

    // 创建节点
    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    circle.setAttribute('cx', node.x);
    circle.setAttribute('cy', node.y);
    circle.setAttribute('r', 22);

    // 使用不同颜色区分节点
    const nodeIndex = nodes.value.findIndex(n => n.id === node.id);
    if (nodeIndex % 3 === 0) {
      circle.setAttribute('fill', primaryColor);
    } else if (nodeIndex % 3 === 1) {
      circle.setAttribute('fill', secondaryColor);
    } else {
      circle.setAttribute('fill', accentColor);
    }

    circle.classList.add('node');

    // 添加拖拽事件
    circle.addEventListener('mousedown', (e) => startDrag(e, node));

    // 创建节点文本
    const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    text.setAttribute('x', node.x);
    text.setAttribute('y', node.y + 5);
    text.setAttribute('text-anchor', 'middle');
    text.setAttribute('fill', 'white');
    text.setAttribute('font-weight', 'bold');
    text.textContent = node.value;
    text.classList.add('node-label');

    // 创建删除按钮
    const deleteBtn = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    deleteBtn.setAttribute('cx', node.x + 18);
    deleteBtn.setAttribute('cy', node.y - 18);
    deleteBtn.setAttribute('r', 10);
    deleteBtn.setAttribute('fill', '#e74c3c');
    deleteBtn.classList.add('delete-btn');

    const deleteBtnText = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    deleteBtnText.setAttribute('x', node.x + 18);
    deleteBtnText.setAttribute('y', node.y - 14);
    deleteBtnText.setAttribute('text-anchor', 'middle');
    deleteBtnText.setAttribute('fill', 'white');
    deleteBtnText.setAttribute('font-size', '14');
    deleteBtnText.setAttribute('font-weight', 'bold');
    deleteBtnText.textContent = '×';

    // 添加删除节点的点击事件
    deleteBtn.addEventListener('click', () => removeNode(node.value));
    deleteBtnText.addEventListener('click', () => removeNode(node.value));

    group.appendChild(shadow);
    group.appendChild(circle);
    group.appendChild(text);
    group.appendChild(deleteBtn);
    group.appendChild(deleteBtnText);

    // 添加节点ID标签
    const idLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    idLabel.setAttribute('x', node.x);
    idLabel.setAttribute('y', node.y + 35);
    idLabel.setAttribute('text-anchor', 'middle');
    idLabel.setAttribute('fill', nodeIndex % 3 === 0 ? primaryColor :
                                 nodeIndex % 3 === 1 ? secondaryColor : accentColor);
    idLabel.setAttribute('font-size', '12');
    idLabel.textContent = `ID: ${node.id.substring(0, 4)}...`;

    group.appendChild(idLabel);
    svg.appendChild(group);
  });

  // 添加拖拽事件监听
  graphContainer.value.addEventListener('mousemove', onDrag);
  graphContainer.value.addEventListener('mouseup', endDrag);
  graphContainer.value.addEventListener('mouseleave', endDrag);
};

// 监听窗口大小变化，重新渲染图
onMounted(() => {
  renderGraph();
  window.addEventListener('resize', renderGraph);
});
</script>

<template>
  <div class="graph-visualizer">
    <h2>图可视化</h2>

    <div class="control-panel">
      <div class="input-groups">
        <div class="input-group">
          <div class="input-label">
            <i class="fa-solid fa-circle-nodes"></i>
            <span>添加节点:</span>
          </div>
          <div class="input-with-icon">
            <i class="fa-solid fa-tag"></i>
            <input
              type="text"
              v-model="newNodeValue"
              placeholder="输入节点值"
              @keyup.enter="addNode"
            />
          </div>
          <button @click="addNode" class="add-node-btn">
            <i class="fa-solid fa-plus"></i>
            <span>添加节点</span>
          </button>
        </div>

        <div class="input-group">
          <div class="input-label">
            <i class="fa-solid fa-link"></i>
            <span>添加边:</span>
          </div>
          <div class="edge-inputs">
            <div class="input-with-icon">
              <i class="fa-solid fa-arrow-right-from-bracket"></i>
              <input
                type="text"
                v-model="sourceNode"
                placeholder="源节点"
              />
            </div>
            <div class="input-with-icon">
              <i class="fa-solid fa-arrow-right-to-bracket"></i>
              <input
                type="text"
                v-model="targetNode"
                placeholder="目标节点"
                @keyup.enter="addEdge"
              />
            </div>
          </div>
          <button @click="addEdge" class="add-edge-btn">
            <i class="fa-solid fa-plus"></i>
            <span>添加边</span>
          </button>
        </div>

        <div class="input-group actions-group">
          <button @click="clearGraph" class="clear-btn">
            <i class="fa-solid fa-trash-can"></i>
            <span>清空图</span>
          </button>
          <button @click="initExampleData" class="example-btn">
            <i class="fa-solid fa-wand-magic-sparkles"></i>
            <span>初始化示例</span>
          </button>
        </div>
      </div>

      <div class="message" v-if="message">
        {{ message }}
      </div>
    </div>

    <div class="visualization">
      <div class="graph-container" ref="graphContainer"></div>
    </div>

    <div class="explanation">
      <h3>图说明</h3>
      <p>图是一种非线性数据结构，由节点（顶点）和边组成，用于表示元素之间的关系。</p>
      <p>特点：</p>
      <ul>
        <li>由节点（顶点）和边组成</li>
        <li>可以表示复杂的关系网络</li>
        <li>有向图和无向图：边可以有方向或无方向</li>
        <li>加权图：边可以有权重值</li>
      </ul>
      <p>应用场景：</p>
      <ul>
        <li>社交网络</li>
        <li>地图和导航系统</li>
        <li>网络拓扑</li>
        <li>推荐系统</li>
      </ul>
      <p>操作提示：</p>
      <ul>
        <li>可以拖动节点调整位置</li>
        <li>点击节点上的红色按钮删除节点</li>
        <li>点击边可以删除边</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.graph-visualizer {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.graph-visualizer h2 {
  color: var(--primary-color);
  font-size: 1.75rem;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.graph-visualizer h2::before {
  content: '\f78b';
  font-family: 'Font Awesome 6 Free';
  font-weight: 900;
  font-size: 1.5rem;
  color: var(--primary-color);
}

.control-panel {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  background-color: var(--neutral-100);
  padding: 1.25rem;
  border-radius: var(--radius-lg);
  border: 1px solid var(--neutral-200);
}

.input-groups {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.input-group {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
  align-items: center;
}

.input-label {
  font-weight: 600;
  min-width: 120px;
  color: var(--neutral-700);
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.input-label i {
  color: var(--primary-color);
  font-size: 1.1rem;
}

.edge-inputs {
  display: flex;
  gap: 0.75rem;
  flex: 1;
}

.input-with-icon {
  position: relative;
  flex: 1;
  min-width: 150px;
}

.input-with-icon i {
  position: absolute;
  left: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--neutral-500);
}

input {
  padding: 0.75rem 0.75rem 0.75rem 2.5rem;
  border: 1px solid var(--neutral-300);
  border-radius: var(--radius-md);
  flex: 1;
  width: 100%;
  font-size: 1rem;
  transition: all var(--transition-normal);
  box-shadow: var(--shadow-sm);
}

input:focus {
  outline: none;
  border-color: var(--primary-light);
  box-shadow: 0 0 0 3px rgba(67, 97, 238, 0.15);
}

.actions-group {
  justify-content: flex-end;
  margin-top: 0.5rem;
}

.add-node-btn {
  background-color: var(--primary-color);
}

.add-node-btn:hover {
  background-color: var(--primary-dark);
}

.add-edge-btn {
  background-color: var(--accent-color);
}

.add-edge-btn:hover {
  background-color: #3ab7de;
}

.clear-btn {
  background-color: var(--danger-color);
}

.clear-btn:hover {
  background-color: #e11d48;
}

.example-btn {
  background-color: var(--warning-color);
}

.example-btn:hover {
  background-color: #f59e0b;
}

.message {
  padding: 0.75rem 1rem;
  background-color: var(--neutral-100);
  border-left: 4px solid var(--primary-color);
  margin-top: 0.5rem;
  border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
  font-size: 0.9rem;
  color: var(--neutral-700);
  animation: fadeIn 0.3s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.visualization {
  margin: 1.5rem 0;
}

.graph-container {
  width: 100%;
  height: 450px;
  border: 2px solid var(--neutral-300);
  border-radius: var(--radius-lg);
  background-color: var(--neutral-100);
  position: relative;
  overflow: hidden;
  box-shadow: var(--shadow-md);
  padding: 1rem;
  background-image: radial-gradient(var(--neutral-200) 1px, transparent 1px);
  background-size: 20px 20px;
}

.empty-message {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: var(--neutral-500);
  text-align: center;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}

.empty-message::before {
  content: '\f78b';
  font-family: 'Font Awesome 6 Free';
  font-weight: 900;
  font-size: 2.5rem;
  color: var(--neutral-400);
}

.explanation {
  background-color: var(--neutral-100);
  padding: 1.5rem;
  border-radius: var(--radius-lg);
  margin-top: 1.5rem;
  border: 1px solid var(--neutral-200);
  box-shadow: var(--shadow-sm);
}

.explanation h3 {
  margin-bottom: 1rem;
  color: var(--primary-color);
  font-size: 1.3rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.explanation h3::before {
  content: '\f05a';
  font-family: 'Font Awesome 6 Free';
  font-weight: 900;
  color: var(--primary-color);
}

.explanation ul {
  padding-left: 1.5rem;
  margin-top: 0.75rem;
}

.explanation li {
  margin-bottom: 0.5rem;
  position: relative;
  padding-left: 0.5rem;
}

.explanation p {
  margin-bottom: 0.75rem;
  line-height: 1.6;
}

/* SVG样式 */
:deep(.node) {
  cursor: move;
  transition: all var(--transition-normal);
  filter: drop-shadow(0 2px 3px rgba(0, 0, 0, 0.2));
}

:deep(.node:hover) {
  fill: var(--secondary-color);
  filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.3));
  transform: scale(1.05);
}

:deep(.node-label) {
  font-weight: bold;
  pointer-events: none;
}

:deep(.edge) {
  cursor: pointer;
  transition: all var(--transition-normal);
  stroke-linecap: round;
}

:deep(.edge:hover) {
  stroke: var(--secondary-color);
  stroke-width: 3;
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.3));
}

:deep(.delete-btn) {
  transition: all var(--transition-normal);
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.2));
}

:deep(.delete-btn:hover) {
  fill: #e11d48;
  transform: scale(1.2);
}

:deep(.arrow) {
  transition: all var(--transition-normal);
}

:deep(.edge:hover + .arrow) {
  fill: var(--secondary-color);
  transform: scale(1.2);
}

:deep(.edge:hover) {
  stroke: #e74c3c;
  stroke-width: 3;
}

:deep(.delete-btn) {
  cursor: pointer;
  transition: fill 0.3s;
}

:deep(.delete-btn:hover) {
  fill: #c0392b;
}
</style>