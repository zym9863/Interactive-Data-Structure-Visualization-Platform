<script setup>
import { ref } from 'vue';

// 栈数据模型
const stackData = ref([]);
const newValue = ref('');
const message = ref('');
const animatingIndex = ref(null);

// 操作函数
const push = () => {
  if (newValue.value.trim() === '') {
    message.value = '请输入有效值';
    return;
  }

  // 添加动画效果
  animatingIndex.value = stackData.value.length;
  stackData.value.push(newValue.value);
  message.value = `入栈元素 ${newValue.value} 成功`;
  newValue.value = '';

  setTimeout(() => {
    animatingIndex.value = null;
  }, 500);
};

const pop = () => {
  if (stackData.value.length === 0) {
    message.value = '栈为空，无法出栈';
    return;
  }

  // 添加动画效果
  animatingIndex.value = stackData.value.length - 1;

  setTimeout(() => {
    const popped = stackData.value.pop();
    message.value = `出栈元素 ${popped} 成功`;
    animatingIndex.value = null;
  }, 300);
};

const peek = () => {
  if (stackData.value.length === 0) {
    message.value = '栈为空，无法查看栈顶';
    return;
  }

  // 添加动画效果
  animatingIndex.value = stackData.value.length - 1;
  message.value = `栈顶元素为: ${stackData.value[stackData.value.length - 1]}`;

  setTimeout(() => {
    animatingIndex.value = null;
  }, 1000);
};

const clearStack = () => {
  stackData.value = [];
  message.value = '栈已清空';
};

// 初始化示例数据
const initExampleData = () => {
  stackData.value = ['5', '4', '3', '2', '1'];
  message.value = '已初始化示例数据';
};
</script>

<template>
  <div class="stack-visualizer">
    <h2>栈可视化</h2>

    <div class="control-panel">
      <div class="input-group">
        <div class="input-with-icon">
          <i class="fa-solid fa-keyboard"></i>
          <input
            type="text"
            v-model="newValue"
            placeholder="输入值"
            @keyup.enter="push"
          />
        </div>
        <button @click="push" class="push-btn">
          <i class="fa-solid fa-arrow-down"></i>
          <span>入栈 (Push)</span>
        </button>
        <button @click="pop" :disabled="stackData.length === 0" class="pop-btn">
          <i class="fa-solid fa-arrow-up"></i>
          <span>出栈 (Pop)</span>
        </button>
        <button @click="peek" :disabled="stackData.length === 0" class="peek-btn">
          <i class="fa-solid fa-eye"></i>
          <span>查看栈顶</span>
        </button>
        <button @click="clearStack" :disabled="stackData.length === 0" class="clear-btn">
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
      <div class="stack-container">
        <div class="stack-label">栈底</div>
        <div class="stack-items">
          <div
            v-for="(item, index) in stackData"
            :key="index"
            class="stack-item"
            :class="{
              'animating': animatingIndex === index,
              'stack-top': index === stackData.length - 1
            }"
          >
            {{ item }}
          </div>

          <div v-if="stackData.length === 0" class="empty-message">
            栈为空，请添加元素
          </div>
        </div>
        <div class="stack-label" v-if="stackData.length > 0">栈顶</div>
      </div>
    </div>

    <div class="explanation">
      <h3>栈说明</h3>
      <p>栈是一种后进先出(LIFO: Last In First Out)的线性数据结构。</p>
      <p>特点：</p>
      <ul>
        <li>只能在栈顶进行插入和删除操作</li>
        <li>入栈(Push)：将元素添加到栈顶，时间复杂度 O(1)</li>
        <li>出栈(Pop)：移除栈顶元素并返回，时间复杂度 O(1)</li>
        <li>查看栈顶(Peek)：查看栈顶元素但不移除，时间复杂度 O(1)</li>
      </ul>
      <p>应用场景：</p>
      <ul>
        <li>函数调用栈</li>
        <li>表达式求值</li>
        <li>括号匹配</li>
        <li>浏览器的前进/后退功能</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.stack-visualizer {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.stack-visualizer h2 {
  color: var(--primary-color);
  font-size: 1.75rem;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.stack-visualizer h2::before {
  content: '\f5fd';
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

.push-btn {
  background-color: var(--primary-color);
}

.push-btn:hover {
  background-color: var(--primary-dark);
}

.pop-btn {
  background-color: var(--secondary-color);
}

.pop-btn:hover {
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

.stack-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.stack-label {
  font-weight: bold;
  color: var(--neutral-600);
  background-color: var(--neutral-200);
  padding: 0.25rem 1rem;
  border-radius: var(--radius-full);
  font-size: 0.9rem;
}

.stack-items {
  display: flex;
  flex-direction: column-reverse;
  gap: 3px;
  min-height: 300px;
  width: 100%;
  max-width: 350px;
  border: 2px solid var(--neutral-300);
  border-radius: var(--radius-lg);
  padding: 1.25rem;
  background-color: var(--neutral-100);
  position: relative;
  box-shadow: var(--shadow-md);
}

.stack-item {
  padding: 1rem;
  background-color: var(--primary-color);
  color: white;
  border-radius: var(--radius-md);
  text-align: center;
  transition: all var(--transition-normal);
  font-weight: 600;
  font-size: 1.1rem;
  box-shadow: var(--shadow-sm);
}

.stack-item.stack-top {
  background-color: var(--secondary-color);
  transform: translateY(-3px);
  box-shadow: var(--shadow-md);
  position: relative;
}

.stack-item.stack-top::after {
  content: '栈顶';
  position: absolute;
  top: -20px;
  right: 10px;
  font-size: 0.75rem;
  background-color: var(--secondary-color);
  color: white;
  padding: 0.15rem 0.5rem;
  border-radius: var(--radius-full);
  font-weight: normal;
}

.stack-item.animating {
  transform: scale(1.05);
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
  content: '\f5fd';
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