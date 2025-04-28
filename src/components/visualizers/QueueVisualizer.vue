<script setup>
import { ref } from 'vue';

// 队列数据模型
const queueData = ref([]);
const newValue = ref('');
const message = ref('');
const animatingIndex = ref(null);

// 操作函数
const enqueue = () => {
  if (newValue.value.trim() === '') {
    message.value = '请输入有效值';
    return;
  }

  // 添加动画效果
  animatingIndex.value = queueData.value.length;
  queueData.value.push(newValue.value);
  message.value = `入队元素 ${newValue.value} 成功`;
  newValue.value = '';

  setTimeout(() => {
    animatingIndex.value = null;
  }, 500);
};

const dequeue = () => {
  if (queueData.value.length === 0) {
    message.value = '队列为空，无法出队';
    return;
  }

  // 添加动画效果
  animatingIndex.value = 0;

  setTimeout(() => {
    const dequeued = queueData.value.shift();
    message.value = `出队元素 ${dequeued} 成功`;
    animatingIndex.value = null;
  }, 300);
};

const peek = () => {
  if (queueData.value.length === 0) {
    message.value = '队列为空，无法查看队首';
    return;
  }

  // 添加动画效果
  animatingIndex.value = 0;
  message.value = `队首元素为: ${queueData.value[0]}`;

  setTimeout(() => {
    animatingIndex.value = null;
  }, 1000);
};

const clearQueue = () => {
  queueData.value = [];
  message.value = '队列已清空';
};

// 初始化示例数据
const initExampleData = () => {
  queueData.value = ['A', 'B', 'C', 'D', 'E'];
  message.value = '已初始化示例数据';
};
</script>

<template>
  <div class="queue-visualizer">
    <h2>队列可视化</h2>

    <div class="control-panel">
      <div class="input-group">
        <div class="input-with-icon">
          <i class="fa-solid fa-keyboard"></i>
          <input
            type="text"
            v-model="newValue"
            placeholder="输入值"
            @keyup.enter="enqueue"
          />
        </div>
        <button @click="enqueue" class="enqueue-btn">
          <i class="fa-solid fa-right-to-bracket"></i>
          <span>入队 (Enqueue)</span>
        </button>
        <button @click="dequeue" :disabled="queueData.length === 0" class="dequeue-btn">
          <i class="fa-solid fa-right-from-bracket"></i>
          <span>出队 (Dequeue)</span>
        </button>
        <button @click="peek" :disabled="queueData.length === 0" class="peek-btn">
          <i class="fa-solid fa-eye"></i>
          <span>查看队首</span>
        </button>
        <button @click="clearQueue" :disabled="queueData.length === 0" class="clear-btn">
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
      <div class="queue-container">
        <div class="queue-labels">
          <div class="queue-label">队首 (Front)</div>
          <div class="queue-label">队尾 (Rear)</div>
        </div>
        <div class="queue-items">
          <div
            v-for="(item, index) in queueData"
            :key="index"
            class="queue-item"
            :class="{
              'animating': animatingIndex === index,
              'queue-front': index === 0,
              'queue-rear': index === queueData.length - 1
            }"
          >
            {{ item }}
          </div>

          <div v-if="queueData.length === 0" class="empty-message">
            队列为空，请添加元素
          </div>
        </div>
      </div>
    </div>

    <div class="explanation">
      <h3>队列说明</h3>
      <p>队列是一种先进先出(FIFO: First In First Out)的线性数据结构。</p>
      <p>特点：</p>
      <ul>
        <li>只能在队尾进行插入操作，在队首进行删除操作</li>
        <li>入队(Enqueue)：将元素添加到队尾，时间复杂度 O(1)</li>
        <li>出队(Dequeue)：移除队首元素并返回，时间复杂度 O(1)</li>
        <li>查看队首(Peek)：查看队首元素但不移除，时间复杂度 O(1)</li>
      </ul>
      <p>应用场景：</p>
      <ul>
        <li>任务调度</li>
        <li>打印机队列</li>
        <li>消息队列</li>
        <li>广度优先搜索(BFS)算法</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.queue-visualizer {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.queue-visualizer h2 {
  color: var(--primary-color);
  font-size: 1.75rem;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.queue-visualizer h2::before {
  content: '\f0c9';
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

.enqueue-btn {
  background-color: var(--primary-color);
}

.enqueue-btn:hover {
  background-color: var(--primary-dark);
}

.dequeue-btn {
  background-color: var(--secondary-color);
}

.dequeue-btn:hover {
  background-color: #e3106f;
}

.peek-btn {
  background-color: var(--accent-color);
}

.peek-btn:hover {
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

.queue-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.queue-labels {
  display: flex;
  justify-content: space-between;
  width: 100%;
}

.queue-label {
  font-weight: bold;
  color: var(--neutral-600);
  background-color: var(--neutral-200);
  padding: 0.25rem 1rem;
  border-radius: var(--radius-full);
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.queue-label:first-child::before {
  content: '\f060';
  font-family: 'Font Awesome 6 Free';
  font-weight: 900;
  font-size: 0.8rem;
}

.queue-label:last-child::after {
  content: '\f061';
  font-family: 'Font Awesome 6 Free';
  font-weight: 900;
  font-size: 0.8rem;
}

.queue-items {
  display: flex;
  flex-direction: row;
  gap: 0.5rem;
  min-height: 100px;
  width: 100%;
  border: 2px solid var(--neutral-300);
  border-radius: var(--radius-lg);
  padding: 1.25rem;
  background-color: var(--neutral-100);
  position: relative;
  overflow-x: auto;
  box-shadow: var(--shadow-md);
}

.queue-item {
  padding: 1rem;
  min-width: 70px;
  height: 70px;
  background-color: var(--primary-color);
  color: white;
  border-radius: var(--radius-md);
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all var(--transition-normal);
  font-weight: 600;
  font-size: 1.1rem;
  box-shadow: var(--shadow-sm);
  position: relative;
}

.queue-item.queue-front {
  background-color: var(--secondary-color);
  transform: translateY(-5px);
  box-shadow: var(--shadow-md);
}

.queue-item.queue-front::after {
  content: '队首';
  position: absolute;
  top: -20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.75rem;
  background-color: var(--secondary-color);
  color: white;
  padding: 0.15rem 0.5rem;
  border-radius: var(--radius-full);
  font-weight: normal;
  white-space: nowrap;
}

.queue-item.queue-rear {
  background-color: var(--accent-color);
  transform: translateY(-5px);
  box-shadow: var(--shadow-md);
}

.queue-item.queue-rear::after {
  content: '队尾';
  position: absolute;
  top: -20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.75rem;
  background-color: var(--accent-color);
  color: white;
  padding: 0.15rem 0.5rem;
  border-radius: var(--radius-full);
  font-weight: normal;
  white-space: nowrap;
}

.queue-item.animating {
  transform: scale(1.1);
  box-shadow: 0 0 15px rgba(67, 97, 238, 0.3);
  z-index: 10;
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
  content: '\f0c9';
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