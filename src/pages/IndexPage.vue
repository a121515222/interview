<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleSendData">{{
          isEdit ? '更新' : '新增'
        }}</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';
import { api } from 'src/boot/axios';
import { date } from 'quasar';
import { useQuasar } from 'quasar';
const $q = useQuasar();
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});
function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    isEdit.value = true;
    tempData.value = data;
  }
  if (btn.status === 'delete') {
    handleDelete(data.id);
  }
}
const isEdit = ref(false);
const baseUrl = process.env.API;
const getData = async () => {
  const res = await api.get(`${baseUrl}api/CRUDTest/a`);
  const { data } = res;
  blockData.value = data;
  isEdit.value = false;
};
const postData = async () => {
  const res = await api.post(
    `${baseUrl}api/CRUDTest`,
    JSON.stringify(tempData.value),
    {
      headers: {
        'Content-Type': 'application/json', // 设置请求头
      },
    }
  );
  const { data } = res;
};

const handlePostData = async () => {
  handleValidate();
  await postData();
  await getData();
  resetTempData();
};
const handleValidate = () => {
  const { name, age } = tempData.value;
  if (!name) {
    handleAlert('名稱不可為空白');
    return false;
  }
  if (!age) {
    handleAlert('年齡不可為空白');
    return false;
  }
  if (!Number.isInteger(Number(age)) || Number(age) <= 0) {
    handleAlert('年齡必須為正整數');
    return false;
  }
  return true;
};
const handleAlert = (alertMessage) => {
  $q.dialog({
    title: 'Alert',
    message: alertMessage,
  });
};
const handleSendData = async () => {
  if (isEdit.value) {
    await handleEditData();
  } else {
    await handlePostData();
  }
};
const handleEditData = async () => {
  const res = await api.patch(
    `${baseUrl}api/CRUDTest`,
    JSON.stringify(tempData.value),
    {
      headers: {
        'Content-Type': 'application/json', // 设置请求头
      },
    }
  );
  const { data } = res;
  if (data) {
    await getData();
    resetTempData();
  }
};
const deleteData = async (id) => {
  const res = await api.delete(`${baseUrl}api/CRUDTest/${id}`, {
    data: JSON.stringify(tempData.value),
    headers: {
      'Content-Type': 'application/json', // 设置请求头
    },
  });
  const { data } = res;
  if (data) {
    await getData();
  }
};
const handleDelete = async (id) => {
  $q.dialog({
    title: '確認刪除',
    message: '確定要刪除嗎?',
    ok: {
      label: '確定',
      color: 'negative',
    },
    cancel: {
      label: '取消',
      color: 'grey-8',
    },
  }).onOk(() => {
    deleteData(id);
  });
};
const resetTempData = () => {
  tempData.value = {
    id: '',
    name: '',
    age: '',
  };
};
onMounted(async () => {
  await getData();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
