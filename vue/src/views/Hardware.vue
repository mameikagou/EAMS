<template>
  <n-space vertical :size="12">
    <n-form class="centerform" ref="formRef" inline :label-width="80" :model="req" :size="size">
      <n-form-item>
        <n-input size="large" v-model:value="req.HardwareName" placeholder="Find..." />
      </n-form-item>
      <n-form-item>
        <n-button attr-type="button" @click="find"> Go! </n-button>
      </n-form-item>
      <n-form-item>
        <n-button :disabled="!isadmin" attr-type="button" @click="create"> + </n-button>
      </n-form-item>
    </n-form>
  </n-space>
  <n-space vertical :size="12">
    <n-data-table
      :bordered="false"
      :single-line="false"
      :columns="columns"
      :data="data"
      :pagination="pagination"
    />
  </n-space>
</template>
<script setup>
import { h, defineComponent, onMounted } from "vue";
import {
  NButton,
  useMessage,
  NDataTable,
  NSpace,
  NForm,
  NFormItem,
  NInput,
} from "naive-ui";
// import {DataTableColumns} from 'naive-ui'

import http from '@/util/http';
import { createToast } from 'mosha-vue-toastify';
import createColumns from '@/models/HardwareTable'
import {ref} from 'vue'
import router from '@/router/index'
const req=ref({
    HardwareName:'',
    Category:'',
    Status:'',
    Location:''
})
const data=ref([])
const formRef = ref(null)
const size =ref("medium")
const isadmin=ref(false)
const postadmin = ()=>{
  http
    .get("/p/admin/isadmin", {
      validateStatus: function (status) {
        return true;
      },
    })
    .then((r) => {
      if (r.status === 200) {
        isadmin.value=true
      }
})
}
const create = ()=>{
    router.push('/createhardware')
}
const post = () => {
  http
    .post("/p/user/hlq", req.value, {
      validateStatus: function (status) {
        return true;
      },
    })
    .then((r) => {
      if (r.status === 200) {
        data.value = r.data.data;
        if (data.value === null) {
          data.value = [];
        }

    }else{
        createToast("Failed to fetch data: "+r.data.msg)
    }
})
}
onMounted(()=>{
    post()
    postadmin()
})
const find = (e)=>{
    e.preventDefault()
    post()
}
const columns=createColumns({click(row){
    const url = router.resolve({
        name:'Hardware Detail',
        query:{
            HardwareID:row.HardwareID
        }
    })
    window.open(url.href,"_blank")
},del(row){
    http
    .delete("/p/admin/hdlt", {data:{HardwareID:row.HardwareID}}, {
      validateStatus: function (status) {
        return true;
      },
    })
    .then((r) => {
      if (r.status === 200) {
        createToast("Delete Successfully ")
    }else{
        createToast("Failed to delete")
    }
})
}})
    //window.open(url.href,"_blank")

const pagination={
    pagiSize:20,
}

// defineComponent({
//         setup(){
//             const message = useMessage()
//             return {
//                 data,
//                 columns:createColumns(),
//                 pagination:{
//                     pageSize: 20
//                 }
//             }
//         }
//     })
</script>

<style lang="scss">
.centerform {
  display: flex;
  align-items: center;
  justify-content: center;
}

.n-input:not(.n-input--autosize) {
    width: 500px;
}
</style>
