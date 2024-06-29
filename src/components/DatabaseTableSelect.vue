<template>
  <div id="databaseTable">
    <el-container>
      <el-header>
        <h1>数据表选择</h1>
      </el-header>
      <el-main>
        <el-form :model="form" label-width="120px">
          <el-form-item label="选择数据库" :error="errors.selectedDatabase">
            <el-select v-model="form.selectedDatabase" placeholder="请选择数据库" @change="onDatabaseChange">
              <el-option v-for="database in databases" :key="database" :label="database" :value="database"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="选择数据表" :error="errors.selectedTable">
            <el-select v-model="form.selectedTable" placeholder="请选择数据表" :disabled="!form.selectedDatabase">
              <el-option v-for="table in tables" :key="table" :label="table" :value="table"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onNextStep">下一步</el-button>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';
import { ElMessage } from 'element-plus';
import { bitable } from '@lark-base-open/connector-api';

// 定义接口
interface DatabaseTables {
  [key: string]: string[];
}

// 表单数据和状态
const form = reactive({
  selectedDatabase: '',
  selectedTable: ''
});

const errors = reactive({
  selectedDatabase: '',
  selectedTable: ''
});

const databases = ref<string[]>([]);
const databaseTables = ref<DatabaseTables>({});
const tables = ref<string[]>([]);

// 方法：从 API 获取数据库和数据表
const fetchDatabasesAndTables = async () => {
  try {
    const response = await axios.get('https://run.mocky.io/v3/64650c88-c042-4a5a-9858-67cac00f72b6'); // 本地 JSON 文件路径
    const data = response.data;

    databases.value = Object.keys(data);
    databaseTables.value = data;
  } catch (error) {
    ElMessage.error('获取数据库和数据表列表失败');
  }
};

// 方法：处理数据库选择变化
const onDatabaseChange = (database: string) => {
  tables.value = databaseTables.value[database] || [];
  form.selectedTable = ''; // 重置选中的数据表
  errors.selectedTable = ''; // 清除数据表选择错误消息
};

// 方法：验证表单
const validateForm = () => {
  let valid = true;

  // 验证选择数据库
  if (!form.selectedDatabase) {
    errors.selectedDatabase = '请选择数据库';
    valid = false;
  } else {
    errors.selectedDatabase = '';
  }

  // 验证选择数据表
  if (!form.selectedTable) {
    errors.selectedTable = '请选择数据表';
    valid = false;
  } else {
    errors.selectedTable = '';
  }

  return valid;
};

// 方法：处理下一步按钮点击
const onNextStep = () => {
  if (validateForm()) {
    const config = {
      database: form.selectedDatabase,
      table: form.selectedTable,
    };
    console.log('config', config);
    bitable.saveConfigAndGoNext(config);

    ElMessage.success('验证通过，进入下一步');
    // 执行下一步操作
  } else {
    ElMessage.error('请完成表单选择');
  }
};

// 组件挂载时调用
onMounted(fetchDatabasesAndTables);
</script>

<style>
#databaseTable {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  padding-top: 10px;
}
</style>