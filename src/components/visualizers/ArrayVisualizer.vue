<script setup>
import { ref, reactive } from 'vue';

// 数组数据模型
const arrayData = ref([]);
const newValue = ref('');
const selectedIndex = ref(null);
const message = ref('');

// 操作函数
const addItem = () => {
  if (newValue.value.trim() === '') {
    message.value = '请输入有效值';
    return;
  }

  arrayData.value.push(newValue.value);
  newValue.value = '';
  message.value = `添加元素 ${arrayData.value[arrayData.value.length - 1]} 成功`;
};

const removeItem = (index) => {
  if (index >= 0 && index < arrayData.value.length) {
    const removed = arrayData.value[index];
    arrayData.value.splice(index, 1);
    message.value = `删除索引 ${index} 的元素 ${removed} 成功`;
  }
};

const updateItem = () => {
  if (selectedIndex.value === null || newValue.value.trim() === '') {
    message.value = '请选择索引并输入有效值';
    return;
  }

  if (selectedIndex.value >= 0 && selectedIndex.value < arrayData.value.length) {
    const oldValue = arrayData.value[selectedIndex.value];
    arrayData.value[selectedIndex.value] = newValue.value;
    newValue.value = '';
    message.value = `更新索引 ${selectedIndex.value} 的值从 ${oldValue} 到 ${arrayData.value[selectedIndex.value]} 成功`;
    selectedIndex.value = null;
  }
};

const selectIndex = (index) => {
  selectedIndex.value = index;
  message.value = `选择了索引 ${index}`;
};

const clearArray = () => {
  arrayData.value = [];
  message.value = '数组已清空';
};

// 初始化示例数据
const initExampleData = () => {
  arrayData.value = ['A', 'B', 'C', 'D', 'E'];
  message.value = '已初始化示例数据';
};
</script>

<template>
  <div class="array-visualizer">
    <h2>数组可视化</h2>

    <div class="control-panel">
      <div class="input-group">
        <div class="input-with-icon">
          <i class="fa-solid fa-keyboard"></i>
          <input
            type="text"
            v-model="newValue"
            placeholder="输入值"
            @keyup.enter="addItem"
          />
        </div>
        <button @click="addItem">
          <i class="fa-solid fa-plus"></i>
          <span>添加元素</span>
        </button>
        <button @click="updateItem" :disabled="selectedIndex === null" class="update-btn">
          <i class="fa-solid fa-pen-to-square"></i>
          <span>更新选中</span>
        </button>
        <button @click="clearArray" class="clear-btn">
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
      <div class="array-container">
        <div
          v-for="(item, index) in arrayData"
          :key="index"
          class="array-item"
          :class="{ 'selected': selectedIndex === index }"
          @click="selectIndex(index)"
        >
          <div class="array-value">{{ item }}</div>
          <div class="array-index">{{ index }}</div>
          <button class="remove-btn" @click.stop="removeItem(index)">
            <i class="fa-solid fa-xmark"></i>
          </button>
        </div>

        <div v-if="arrayData.length === 0" class="empty-message">
          数组为空，请添加元素
        </div>
      </div>
    </div>

    <div class="explanation">
      <h3>数组说明</h3>
      <p>数组是最基础的数据结构，它是一组连续的内存空间，用于存储相同类型的数据。</p>
      <p>特点：</p>
      <ul>
        <li>随机访问：可以通过索引直接访问任意位置的元素，时间复杂度 O(1)</li>
        <li>插入和删除：在数组中间插入或删除元素需要移动其他元素，时间复杂度 O(n)</li>
        <li>连续存储：元素在内存中连续存储</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.array-visualizer {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.array-visualizer h2 {
  color: var(--primary-color);
  font-size: 1.75rem;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.array-visualizer h2::before {
  content: '\f00a';
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

.update-btn {
  background-color: var(--accent-color);
}

.update-btn:hover {
  background-color: #3ab7de;
}

.clear-btn {
  background-color: var(--danger-color);
}

.clear-btn:hover {
  background-color: #e11d48;
}

.example-btn {
  background-color: var(--secondary-color);
}

.example-btn:hover {
  background-color: #e3106f;
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

.array-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  min-height: 120px;
  align-items: flex-start;
  padding: 1.5rem;
  background-color: var(--neutral-100);
  border-radius: var(--radius-lg);
  border: 1px dashed var(--neutral-300);
}

.array-item {
  position: relative;
  width: 70px;
  height: 70px;
  border: 2px solid var(--primary-color);
  border-radius: var(--radius-md);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: white;
  transition: all var(--transition-normal);
  cursor: pointer;
  box-shadow: var(--shadow-sm);
}

.array-item:hover {
  transform: translateY(-3px);
  box-shadow: var(--shadow-md);
}

.array-item.selected {
  border-color: var(--secondary-color);
  transform: translateY(-5px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
  background-color: rgba(247, 37, 133, 0.05);
}

.array-value {
  font-size: 1.4rem;
  font-weight: bold;
  color: var(--primary-dark);
}

.array-index {
  position: absolute;
  bottom: -22px;
  font-size: 0.85rem;
  color: var(--neutral-600);
  background-color: var(--neutral-200);
  padding: 0.15rem 0.5rem;
  border-radius: var(--radius-full);
}

.remove-btn {
  position: absolute;
  top: -10px;
  right: -10px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: var(--danger-color);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 0.9rem;
  padding: 0;
  opacity: 0;
  transition: all var(--transition-normal);
  box-shadow: var(--shadow-sm);
  transform: scale(0.8);
}

.array-item:hover .remove-btn {
  opacity: 1;
  transform: scale(1);
}

.remove-btn:hover {
  background-color: #e11d48;
  transform: scale(1.1);
}

.empty-message {
  width: 100%;
  padding: 2rem;
  text-align: center;
  color: var(--neutral-500);
  background-color: transparent;
  border-radius: var(--radius-md);
  font-size: 1.1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}

.empty-message::before {
  content: '\f187';
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
</style>