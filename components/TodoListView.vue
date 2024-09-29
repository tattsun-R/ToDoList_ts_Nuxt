<script setup lang="ts">
import { ref } from "vue";
import { statuses } from "../const/statuses";

// Cookieを使用してタスクを管理
const today = new Date();
const items = useCookie("items", { default: () => [] });
const inputContent = ref<string>("");
const inputLimit = ref<string>("");
const inputState = ref<any>(statuses.NOT_START);
const errorMessage = ref<string>("");
const isErrorMessage = ref<boolean>(false);
const isShowModal = ref<boolean>(false);
let deleteItemId = ref<number | null>(null); //削除対象のItemのID
let deleteItemContent = ref<string>(""); //削除対象のItemの内容

function onEdit(id: number) {
  let isOnEditOther = false;
  items.value.forEach((item: any) => {
    if (item.onEdit) {
      isOnEditOther = true;
      return;
    }
  });

  if (isOnEditOther) {
    errorMessage.value = "他に編集中のタスクがあります";
    isErrorMessage.value = true;
    return;
  }

  inputContent.value = items.value[id].content;
  inputLimit.value = items.value[id].limit;
  inputState.value = items.value[id].state;
  items.value[id].onEdit = true;
}

function onUpdate(id: number) {
  if (inputContent.value === "" || inputLimit.value === "") {
    errorMessage.value = "タスクの内容と期限を入力してください。";
    isErrorMessage.value = true;
    return;
  }

  const updatedItem = {
    id: id,
    content: inputContent.value,
    limit: inputLimit.value,
    state: inputState.value,
    onEdit: false,
  };

  items.value.splice(id, 1, updatedItem);
  isErrorMessage.value = false;
}

function showDeleteModal(id: number) {
  isShowModal.value = true;
  deleteItemId.value = id;
  deleteItemContent.value = items.value[id].content;
}

function onDeleteItem() {
  if (deleteItemId.value !== null) {
    items.value.splice(deleteItemId.value, 1);
    isShowModal.value = false;

    items.value = items.value.map((item: any, index: number) => ({
      id: index,
      content: item.content,
      limit: item.limit,
      state: item.state,
      onEdit: item.onEdit,
    }));
  }
}

function onHideModal() {
  isShowModal.value = false;
}
</script>

<template>
  <!-- エラーメッセージ表示 -->
  <div v-if="isErrorMessage" class="text-red-500 my-2 p-2">
    <p>{{ errorMessage }}</p>
  </div>

  <!-- タスク一覧テーブル -->
  <table class="table-auto w-full border-separate border-spacing-1">
    <thead>
      <tr>
        <th class="border-b-2 border-sky-500">ID</th>
        <th class="border-b-2 border-sky-500">やること</th>
        <th class="border-b-2 border-sky-500">期限</th>
        <th class="border-b-2 border-sky-500">状態</th>
        <th class="border-b-2 border-sky-500">編集</th>
        <th class="border-b-2 border-sky-500">削除</th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="item in items"
        :key="item.id"
        :class="{ 'text-red-500': new Date(item.limit) < today }"
        class="text-center"
      >
        <td class="px-1 py-3 m-1">{{ item.id }}</td>
        <td class="rounded px-2 py-1">
          <span v-if="!item.onEdit">{{ item.content }}</span>
          <input
            v-else
            v-model="inputContent"
            type="text"
            class="w-full p-1 border-2 border-sky-200 rounded"
          />
        </td>
        <td class="px-2 py-1">
          <span v-if="!item.onEdit">{{ item.limit }}</span>
          <input
            v-else
            v-model="inputLimit"
            type="date"
            class="w-full p-1 border-2 border-sky-200 rounded"
          />
        </td>
        <td class="px-2 py-1">
          <span
            v-if="!item.onEdit"
            :class="{
              'bg-gray-300 ': item.state.id === 0, // 未実施
              'bg-yellow-300': item.state.id === 1, // 実行中
              'bg-green-300 px-4': item.state.id === 2, // 完了
            }"
            class="p-2 rounded"
          >
            {{ item.state.value }}
          </span>
          <select
            v-else
            v-model="inputState"
            class="w-full p-1 border-2 border-sky-200 rounded"
          >
            <option
              v-for="state in statuses"
              :key="state.id"
              :value="state"
              :selected="state.id == item.state.id"
            >
              {{ state.value }}
            </option>
          </select>
        </td>
        <td>
          <button v-if="!item.onEdit" @click="onEdit(item.id)">編集</button>
          <button v-else @click="onUpdate(item.id)">完了</button>
        </td>
        <td>
          <button @click="showDeleteModal(item.id)">削除</button>
        </td>
      </tr>
    </tbody>
  </table>

  <!-- モーダル削除確認 -->
  <div
    v-if="isShowModal"
    class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity flex items-center justify-center"
  >
    <div class="bg-white rounded-lg shadow-lg p-6 text-center max-w-md w-full">
      <h2>タスクを削除しますか？</h2>
      <p>{{ deleteItemContent }}</p>
      <button
        @click="onDeleteItem"
        class="bg-red-500 hover:bg-red-700 text-white font-bold mt-4 py-2 px-4 rounded"
      >
        削除
      </button>
      <button
        @click="onHideModal"
        class="ml-2 bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded"
      >
        キャンセル
      </button>
    </div>
  </div>
</template>
