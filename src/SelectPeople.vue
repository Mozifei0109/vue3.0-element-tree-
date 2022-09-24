<script setup name="SelectPeople">
import { reactive, toRefs, ref, getCurrentInstance, computed, onMounted, watch, nextTick } from "vue"
import { treeInfo } from "./userTreeJson"
const props = defineProps({
  showSelectPeople: {
    type: Boolean,
    default: true,
  },
  defaultCheckedUserId: {
    type: Array, //默认选中的人员
  },
  isOnly: {
    type: Number,
    default: 1, // 0-代表单选, 1-代表多选
  },
})

const { isOnly, defaultCheckedUserId, showSelectPeople } = toRefs(props)
const emit = defineEmits(["closeSelectPeople", "handleCheckedUser"])

// 人员树
const filterText = ref("")
const checkedId = ref("")
const treeData = ref([])
const treeRef = ref()
const checkedNodes = ref([])
const checkedArray = ref([])
const defaultProps = {
  children: "children",
  label: "rolesName",
  id: "id",
  parentId: "parentId",
}

console.log(treeInfo, "treeInfo====treeInfo")

watch(filterText, (val) => {
  treeRef.value.filter(val)
})

// 筛选人员（过滤节点）
const filterNode = (value, data) => {
  if (!value) return true

  return data.rolesName.includes(value)
}

const getRolesAndUserTreeApi = () => {
  const dataVal = getTreeData(treeInfo)
  treeData.value = dataVal
}

//  树形结构处理
const getTreeData = (data) => {
  for (var i = 0; i < data.length; i++) {
    let item = data[i]
    if (item.userList) {
      for (let j = 0; j < item.userList.length; j++) {
        const el = item.userList[j]
        el["rolesName"] = el.userName
        el["parentId"] = item.parentId
        el["isUser"] = true
      }
      const children = [...item.child, ...item.userList]
      item["children"] = children

      if (item.child.length < 1) {
        // children若为空数组，则不作操作
      } else {
        // children若不为空数组，则继续 递归调用 本方法
        getTreeData(item.child)
      }
    }
  }

  return data
}

// 操作
const handleCheckChange = (nodeObj, SelectedObj) => {
  const parentChildNodes =
    treeRef.value.getNode(nodeObj).parent.data.child ?? treeRef.value.getNode(nodeObj).parent.data[0].child // 获取选择的数据

  if (isOnly.value == 1) {
    /*_______________多选项 最后选中值事件________________*/
    SelectedObj.checkedKeys.push(-1)
    SelectedObj.checkedKeys.sort((old, New) => {
      return old - New
    })
  } else {
    /*_______________单选项 最后选中值数组对象事件________________*/
    parentChildNodes.forEach((ele, j) => {
      SelectedObj.checkedNodes.forEach((checked, i) => {
        if (ele.value == checked.value) {
          SelectedObj.checkedNodes.splice(i, 1)
          SelectedObj.checkedNodes.push(nodeObj)
        }

        SelectedObj.checkedNodes = UNIQUE(SelectedObj.checkedNodes) // 去重
      })
    })
  }

  SelectedObj.checkedKeys.forEach((value) => {
    parentChildNodes.forEach((ele) => {
      if (value == ele.value) {
        let index = SelectedObj.checkedKeys.indexOf(value)
        SelectedObj.checkedKeys.splice(index, 1)
      }
      SelectedObj.checkedKeys.push(nodeObj.value)
      SelectedObj.checkedKeys = [...new Set(SelectedObj.checkedKeys)] // 去重
      SelectedObj.checkedKeys.sort((old, New) => {
        return old - New
      })
    })
  })

  nextTick(() => {
    treeRef.value.setChecked(SelectedObj.checkedKeys, true) // 选中树
  })

  if (isOnly.value === 1) {
    checkedNodes.value = SelectedObj.checkedNodes.filter((item) => {
      if (item.isUser) {
        return item
      }
    })
    if (treeData.value[0].child.length == checkedNodes.value.length - 1) {
      checkedNodes.value = [checkedNodes.value[0]]
    }
  } else {
    // checkedNodes.value = SelectedObj.checkedNodes
    checkedNodes.value = SelectedObj.checkedNodes.filter((item) => {
      // 这是选中的节点数据
      if (item.isUser) {
        return item
      }
    })
  }
}

const handleCheckChangeOne = (data, checked) => {
  if (isOnly.value !== 1) {
    if (checked) {
      checkedId.value = data.id
      treeRef.value.setCheckedKeys([data.id])
    } else {
      if (checkedId.value === data.id) {
        treeRef.value.setChecked([data.id], false)
      }
    }
  }
}

//选中人员
const handleHoldPeopleInfo = () => {
  emit("handleCheckedUser", checkedNodes.value)
  emit("closeSelectPeople", false)
}

onMounted(() => {
  getRolesAndUserTreeApi()
  if (defaultCheckedUserId.value.length) {
    checkedArray.value = defaultCheckedUserId.value
  }
})
// 关闭窗口
const closeSelectPeople = () => {
  emit("closeSelectPeople", false)
}
</script>

<template>
  <div class="select_people">
    <el-dialog
      width="27.35416vw"
      top="3vw"
      :title="'人员组织'"
      :close-on-click-modal="false"
      destroy-on-close
      :model-value="showSelectPeople"
      @close="closeSelectPeople()"
      draggable
    >
      <div class="dialog_box">
        <el-input placeholder="搜索人员名称" class="input-with-select" v-model="filterText" />

        <el-tree
          show-checkbox
          node-key="id"
          ref="treeRef"
          class="filter-tree"
          :data="treeData"
          :props="defaultProps"
          default-expand-all
          :filter-node-method="filterNode"
          @check="handleCheckChange"
          @check-change="handleCheckChangeOne"
          :default-checked-keys="checkedArray"
        >
          <template #default="{ node, data }">
            <span class="custom-tree-node">
              <span v-if="data.isUser" style="display: flex; align-items: center">
                <div class="tree_icon">
                  {{ data?.rolesName?.substr(0, 1) ?? "" }}
                </div>
                <span>{{ data.rolesName }}({{ data.account }})</span>
              </span>
              <span v-else>{{ node.label }}</span>
            </span>
          </template>
        </el-tree>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="closeSelectPeople()">取消</el-button>
          <el-button type="primary" @click="handleHoldPeopleInfo()">保存</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<style scoped>
.custom-tree-node {
  display: flex;
  align-items: center;
}
.tree_icon {
  width: 32px;
  height: 32px;
  background: #387dfb;
  border-radius: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  margin: 0 10px;
}
:deep(.el-tree) {
  padding: 20px;
}
:deep(.el-tree-node__content) {
  height: unset;
  padding: 5px 0;
}
</style>
