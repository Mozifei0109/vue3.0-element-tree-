<script setup>
import { reactive, toRefs, ref, getCurrentInstance } from "vue"
import { Plus } from "@element-plus/icons-vue"
import SelectPeople from "./SelectPeople.vue"
const showSelectPeople = ref(false)
const checkedUser = ref([])
const defaultCheckedUser = ref([])

// 选择参与人
const handleSelectPeople = () => {
  showSelectPeople.value = true
}

const closeSelectPeople = (val) => {
  showSelectPeople.value = val
}

const handleCheckedUserInfoVal = (val) => {
  checkedUser.value = val
  defaultCheckedUser.value = val.map((item) => {
    return item.id
  })
}
</script>

<template>
  <div>
    <div class="people_content">
      选择人员:<el-button :icon="Plus" circle size="large" @click="handleSelectPeople" />
    </div>
    <div v-if="checkedUser.length" class="checkedList">
      <div v-for="item in checkedUser" :key="item.id" class="checkedListItem">
        <div class="tree_icon" :title="item?.userName">
          {{ item?.userName?.substr(0, 1) ?? "" }}
        </div>
      </div>
    </div>
  </div>

  <SelectPeople
    v-if="showSelectPeople"
    :showSelectPeople="showSelectPeople"
    :defaultCheckedUserId="defaultCheckedUser"
    @closeSelectPeople="closeSelectPeople($event)"
    @handleCheckedUser="handleCheckedUserInfoVal"
  />
</template>

<style scoped>
.people_content {
  text-align: left;
  margin-right: 7px;
}
.checkedList,
.checkedListItem {
  display: flex;
  align-items: center;
}

.tree_icon {
  width: 32px;
  height: 32px;
  background: #f3b816;
  border-radius: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  margin: 0 10px;
  cursor: pointer;
}
</style>

