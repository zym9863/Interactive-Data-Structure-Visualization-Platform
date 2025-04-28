<script setup>
import { ref, onMounted } from 'vue';

// 二叉树节点类
class TreeNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}

// 二叉树数据模型
const root = ref(null);
const newValue = ref('');
const message = ref('');
const treeContainer = ref(null);

// 操作函数
const insertNode = () => {
  if (newValue.value.trim() === '') {
    message.value = '请输入有效值';
    return;
  }

  const value = newValue.value.trim();

  if (!root.value) {
    root.value = new TreeNode(value);
    message.value = `创建根节点: ${value}`;
  } else {
    // 使用层序遍历(BFS)插入节点
    const queue = [root.value];
    let inserted = false;

    while (queue.length > 0 && !inserted) {
      const current = queue.shift();

      if (!current.left) {
        current.left = new TreeNode(value);
        message.value = `插入左子节点: ${value}`;
        inserted = true;
      } else if (!current.right) {
        current.right = new TreeNode(value);
        message.value = `插入右子节点: ${value}`;
        inserted = true;
      } else {
        queue.push(current.left);
        queue.push(current.right);
      }
    }
  }

  newValue.value = '';
  renderTree();
};

const clearTree = () => {
  root.value = null;
  message.value = '二叉树已清空';
  renderTree();
};

// 初始化示例数据
const initExampleData = () => {
  root.value = new TreeNode('A');
  root.value.left = new TreeNode('B');
  root.value.right = new TreeNode('C');
  root.value.left.left = new TreeNode('D');
  root.value.left.right = new TreeNode('E');
  root.value.right.right = new TreeNode('F');

  message.value = '已初始化示例二叉树';
  renderTree();
};

// 渲染二叉树
const renderTree = () => {
  if (!treeContainer.value) return;

  // 清空容器
  treeContainer.value.innerHTML = '';

  if (!root.value) {
    const emptyMessage = document.createElement('div');
    emptyMessage.className = 'empty-message';
    emptyMessage.textContent = '二叉树为空，请添加节点';
    treeContainer.value.appendChild(emptyMessage);
    return;
  }

  // 创建SVG元素
  const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
  svg.setAttribute('width', '100%');
  svg.setAttribute('height', '400');

  // 添加CSS样式
  const style = document.createElementNS('http://www.w3.org/2000/svg', 'style');
  style.textContent = `
    .tree-node {
      animation: nodeAppear 0.5s ease-out;
      transition: all 0.3s ease;
    }
    .tree-node:hover {
      filter: brightness(1.1);
      transform: scale(1.05);
    }
    @keyframes nodeAppear {
      from { opacity: 0; transform: scale(0.5); }
      to { opacity: 1; transform: scale(1); }
    }
  `;
  svg.appendChild(style);
  treeContainer.value.appendChild(svg);

  // 计算树的深度
  const getDepth = (node) => {
    if (!node) return 0;
    return 1 + Math.max(getDepth(node.left), getDepth(node.right));
  };

  const depth = getDepth(root.value);
  const nodeSize = 40;
  const levelHeight = 80;
  const svgWidth = treeContainer.value.clientWidth;

  // 递归绘制节点和连线
  const drawNode = (node, x, y, level) => {
    if (!node) return;

    // 获取CSS变量
    const style = getComputedStyle(document.documentElement);
    const primaryColor = style.getPropertyValue('--primary-color').trim() || '#4361ee';
    const secondaryColor = style.getPropertyValue('--secondary-color').trim() || '#f72585';
    const accentColor = style.getPropertyValue('--accent-color').trim() || '#4cc9f0';

    // 绘制连线
    if (node !== root.value) {
      const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      line.setAttribute('x1', x);
      line.setAttribute('y1', y - levelHeight / 2);
      line.setAttribute('x2', x);
      line.setAttribute('y2', y);
      line.setAttribute('stroke', primaryColor);
      line.setAttribute('stroke-width', '2');
      line.setAttribute('stroke-dasharray', '4,2');
      svg.appendChild(line);
    }

    // 绘制节点阴影
    const shadow = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    shadow.setAttribute('cx', x + 2);
    shadow.setAttribute('cy', y + 2);
    shadow.setAttribute('r', nodeSize / 2);
    shadow.setAttribute('fill', 'rgba(0, 0, 0, 0.1)');
    svg.appendChild(shadow);

    // 绘制节点
    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    circle.setAttribute('cx', x);
    circle.setAttribute('cy', y);
    circle.setAttribute('r', nodeSize / 2);

    // 根节点使用不同颜色
    if (node === root.value) {
      circle.setAttribute('fill', secondaryColor);
    } else if (level % 2 === 0) {
      circle.setAttribute('fill', primaryColor);
    } else {
      circle.setAttribute('fill', accentColor);
    }

    // 添加动画效果
    circle.setAttribute('class', 'tree-node');
    svg.appendChild(circle);

    // 绘制节点值
    const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    text.setAttribute('x', x);
    text.setAttribute('y', y + 5);
    text.setAttribute('text-anchor', 'middle');
    text.setAttribute('fill', 'white');
    text.setAttribute('font-weight', 'bold');
    text.textContent = node.value;
    svg.appendChild(text);

    // 计算下一级节点的位置
    const nextLevel = level + 1;
    const spread = svgWidth / Math.pow(2, nextLevel + 1);

    // 递归绘制左右子节点
    if (node.left) {
      const leftX = x - spread;
      const leftY = y + levelHeight;

      // 绘制左连线
      const leftLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      leftLine.setAttribute('x1', x);
      leftLine.setAttribute('y1', y);
      leftLine.setAttribute('x2', leftX);
      leftLine.setAttribute('y2', leftY);
      leftLine.setAttribute('stroke', primaryColor);
      leftLine.setAttribute('stroke-width', '2');

      // 添加左侧标记
      const leftMarker = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      leftMarker.setAttribute('x', (x + leftX) / 2 - 15);
      leftMarker.setAttribute('y', (y + leftY) / 2 - 5);
      leftMarker.setAttribute('fill', primaryColor);
      leftMarker.setAttribute('font-size', '12');
      leftMarker.textContent = 'L';

      svg.appendChild(leftLine);
      svg.appendChild(leftMarker);

      drawNode(node.left, leftX, leftY, nextLevel);
    }

    if (node.right) {
      const rightX = x + spread;
      const rightY = y + levelHeight;

      // 绘制右连线
      const rightLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      rightLine.setAttribute('x1', x);
      rightLine.setAttribute('y1', y);
      rightLine.setAttribute('x2', rightX);
      rightLine.setAttribute('y2', rightY);
      rightLine.setAttribute('stroke', primaryColor);
      rightLine.setAttribute('stroke-width', '2');

      // 添加右侧标记
      const rightMarker = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      rightMarker.setAttribute('x', (x + rightX) / 2 + 10);
      rightMarker.setAttribute('y', (y + rightY) / 2 - 5);
      rightMarker.setAttribute('fill', primaryColor);
      rightMarker.setAttribute('font-size', '12');
      rightMarker.textContent = 'R';

      svg.appendChild(rightLine);
      svg.appendChild(rightMarker);

      drawNode(node.right, rightX, rightY, nextLevel);
    }
  };

  // 开始绘制树
  const startX = svgWidth / 2;
  const startY = 50;
  drawNode(root.value, startX, startY, 0);
};

// 监听窗口大小变化，重新渲染树
onMounted(() => {
  renderTree();
  window.addEventListener('resize', renderTree);
});
</script>

<template>
  <div class="tree-visualizer">
    <h2>二叉树可视化</h2>

    <div class="control-panel">
      <div class="input-group">
        <div class="input-with-icon">
          <i class="fa-solid fa-keyboard"></i>
          <input
            type="text"
            v-model="newValue"
            placeholder="输入节点值"
            @keyup.enter="insertNode"
          />
        </div>
        <button @click="insertNode" class="insert-btn">
          <i class="fa-solid fa-plus-circle"></i>
          <span>插入节点</span>
        </button>
        <button @click="clearTree" class="clear-btn">
          <i class="fa-solid fa-trash-can"></i>
          <span>清空</span>
        </button>
        <button @click="initExampleData" class="example-btn">
          <i class="fa-solid fa-wand-magic-sparkles"></i>
          <span>示例</span>
        </button>
      </div>

      <div class="message" v-if="message">
        {{ message }}
      </div>
    </div>

    <div class="visualization">
      <div class="tree-container" ref="treeContainer"></div>
    </div>

    <div class="explanation">
      <h3>二叉树说明</h3>
      <p>二叉树是一种树形数据结构，其中每个节点最多有两个子节点，通常称为左子节点和右子节点。</p>
      <p>特点：</p>
      <ul>
        <li>每个节点最多有两个子节点</li>
        <li>具有层次结构，适合表示具有层级关系的数据</li>
        <li>可以通过前序、中序、后序和层序等多种方式遍历</li>
      </ul>
      <p>应用场景：</p>
      <ul>
        <li>表达式解析</li>
        <li>哈夫曼编码</li>
        <li>优先队列（堆）</li>
        <li>二叉搜索树用于高效查找</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.tree-visualizer {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.tree-visualizer h2 {
  color: var(--primary-color);
  font-size: 1.75rem;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.tree-visualizer h2::before {
  content: '\f1bb';
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

.input-group {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.input-with-icon {
  position: relative;
  flex: 1;
  min-width: 200px;
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

.insert-btn {
  background-color: var(--primary-color);
}

.insert-btn:hover {
  background-color: var(--primary-dark);
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

.tree-container {
  width: 100%;
  height: 450px;
  border: 2px solid var(--neutral-300);
  border-radius: var(--radius-lg);
  background-color: var(--neutral-100);
  position: relative;
  overflow: auto;
  box-shadow: var(--shadow-md);
  padding: 1rem;
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
  content: '\f1bb';
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
</style>