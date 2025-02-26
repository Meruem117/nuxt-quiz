<template>
  <div class="flex flex-col space-y-2">
    <el-breadcrumb separator="/" class="text-lg">
      <el-breadcrumb-item :to="{ path: '/topic' }">Topic</el-breadcrumb-item>
      <el-breadcrumb-item>{{ topicName }} - {{ state.title }}</el-breadcrumb-item>
    </el-breadcrumb>
    <el-table :data="state.list" border stripe style="width: 100%">
      <el-table-column prop="id" label="Id" fixed width="90" />
      <el-table-column prop="question" label="Question" show-overflow-tooltip width="120" />
      <el-table-column prop="up" label="Up" width="120" />
      <!-- <el-table-column prop="upId" label="Up Id" fixed width="120" /> -->
      <!-- <el-table-column prop="topic" label="Topic" fixed width="120" />
      <el-table-column prop="topicId" label="Topic Id" fixed width="120" /> -->
      <el-table-column prop="type" label="Type" width="150">
        <template #default="scope">
          <el-tag>{{ TYPE[scope.row.type] }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="optionA" label="Option A" show-overflow-tooltip width="90" />
      <el-table-column prop="optionB" label="Option B" show-overflow-tooltip width="90" />
      <el-table-column prop="optionC" label="Option C" show-overflow-tooltip width="90" />
      <el-table-column prop="optionD" label="Option D" show-overflow-tooltip width="90" />
      <el-table-column prop="answer" label="Answer" width="90" />
      <el-table-column prop="pass" label="Pass" width="120">
        <template #default="scope">
          <el-tag :type="PASS[scope.row.pass].type">{{ PASS[scope.row.pass].text }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" label="Create Time" width="180" />
      <el-table-column prop="updateTime" label="Update Time" width="180" />
      <el-table-column label="Operations" fixed="right">
        <template v-slot="scope" #default>
          <el-button type="primary" size="small" @click="openDetail(scope.row)">Detail</el-button>
          <el-button type="danger" size="small" @click="deleteOne(scope.row)">Delete</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="flex pt-2">
      <div class="flex justify-start w-1/2">
        <el-button type="primary" @click="openDialog" class="w-16">Add</el-button>
      </div>
      <div class="flex justify-end w-1/2">
        <el-pagination background layout="prev, pager, next" :total="state.total" :page-size="state.size"
          @current-change="changePage" />
      </div>
    </div>
    <el-dialog v-model="state.visible" :title="state.isAdd ? `Add ${state.title}` : `${state.title} Detail`"
      :append-to-body="true">
      <el-form ref="questionForm" :model="state.data" :rules="state.rules" label-position="right" label-width="100px">
        <el-form-item label="Question" prop="question">
          <el-input v-model="state.data.question" type="textarea" autosize clearable />
        </el-form-item>
        <el-row :gutter="12">
          <el-col :span="12">
            <el-form-item label="Type" prop="type">
              <el-radio-group v-model="state.data.type">
                <el-radio v-for="(item, index) in TYPE" :key="index" :label="index">{{ item }}</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Pass" prop="pass" v-show="!state.isAdd">
              <el-select v-model="state.data.pass" style="width: 100%">
                <el-option v-for="(item, index) in PASS" :key="index" :label="item.text" :value="index.toString()" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="12">
          <el-col :span="12">
            <el-form-item label="Option A" prop="optionA">
              <el-input v-model="state.data.optionA" type="text" clearable />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Option B" prop="optionB">
              <el-input v-model="state.data.optionB" type="text" clearable />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="12">
          <el-col :span="12">
            <el-form-item label="Option C" prop="optionC">
              <el-input v-model="state.data.optionC" type="text" clearable />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Option D" prop="optionD">
              <el-input v-model="state.data.optionD" type="text" clearable />
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="Answer" prop="answer">
          <el-input v-model="state.data.answer" type="text" clearable />
        </el-form-item>
      </el-form>
      <template #footer>
        <span>
          <el-button @click="state.visible = false">Cancel</el-button>
          <el-button v-show="state.isAdd" type="primary" @click="addOne(questionForm)">Add</el-button>
          <el-button v-show="!state.isAdd" type="primary" @click="updateOne(questionForm)">Update</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { FormInstance, FormRules, ElMessage, ElMessageBox } from 'element-plus'
import type { questionItem } from '~/models/question'
import { getQuestionPage, getQuestionById, updateQuestionById, deleteQuestionById, addQuestion } from '~/services/question'
import { DEFAULT_PAGE_SIZE, PASS, TYPE } from '~/constant'

interface stateItem {
  title: string,
  page: number,
  size: number,
  total: number,
  list: questionItem[],
  visible: boolean,
  isAdd: boolean,
  data: questionItem,
  rules: FormRules
}

const route = useRoute()
const topicId = Number(route.params.topicId)
const topicName = String(route.query.name)
const questionForm = ref<FormInstance>()
const state: stateItem = reactive({
  title: 'Question',
  page: 1,
  size: DEFAULT_PAGE_SIZE,
  total: 0,
  list: [],
  visible: false,
  isAdd: false,
  data: {} as questionItem,
  rules: {
    question: { required: true, message: 'Please input question', trigger: 'blur' },
    type: { required: true, message: 'Please select question type', trigger: 'blur' },
    optionA: { required: true, message: 'Please input option a', trigger: 'blur' },
    optionB: { required: true, message: 'Please input option b', trigger: 'blur' },
    answer: { required: true, message: 'Please select correct answer', trigger: 'blur' }
  }
})

async function load(): Promise<void> {
  const res = await getQuestionPage({
    page: state.page,
    size: state.size
  }, topicId)
  state.list = res.data.records
  state.total = res.data.total
}

function changePage(page: number) {
  state.page = page
  load()
}

async function openDetail(data: questionItem): Promise<void> {
  const res = await getQuestionById(data.id)
  state.data = res.data
  state.isAdd = false
  state.visible = true
}

function openDialog(): void {
  state.data = {} as questionItem
  state.isAdd = true
  state.visible = true
}

function deleteOne(data: questionItem): void {
  ElMessageBox.confirm('Are you sure to delete?', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    type: 'warning',
    customClass: 'message-box'
  }).then(async () => {
    const res = await deleteQuestionById({ id: data.id })
    if (res.data) {
      ElMessage.success('Delete successfully')
      load()
    } else {
      ElMessage.error('Delete failed')
    }
  }).catch(() => {
    ElMessage({
      type: 'info',
      message: 'Delete canceled',
    })
  })
}

async function updateOne(form: FormInstance | undefined): Promise<void> {
  if (!form) return
  const validate = await form.validate()
  if (!validate) return
  ElMessageBox.confirm('Are you sure to update?', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    type: 'warning',
    customClass: 'message-box'
  }).then(async () => {
    const res = await updateQuestionById(state.data)
    if (res.data) {
      state.visible = false
      ElMessage.success('Update successfully')
      load()
    } else {
      ElMessage.error('Update failed')
    }
  }).catch(() => {
    ElMessage({
      type: 'info',
      message: 'Update canceled',
    })
  })
}

async function addOne(form: FormInstance | undefined): Promise<void> {
  if (!form) return
  const validate = await form.validate()
  if (!validate) return
  ElMessageBox.confirm('Are you sure to add?', {
    confirmButtonText: 'Confirm',
    cancelButtonText: 'Cancel',
    type: 'warning',
    customClass: 'message-box'
  }).then(async () => {
    const res = await addQuestion({ ...state.data, topicId, topic: topicName })
    if (res.data > 0) {
      state.visible = false
      ElMessage.success('Add successfully')
      load()
    } else {
      ElMessage.error('Add failed')
    }
  }).catch(() => {
    ElMessage({
      type: 'info',
      message: 'Add canceled',
    })
  })
}

onMounted(() => {
  load()
})
</script>
