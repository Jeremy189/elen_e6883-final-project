<template>
  <div>
    <a-button type="primary" @click="openModal" v-if="account === data.initiator && data.success">Initiate a use request</a-button>
    <a-table :loading="state.loading" :data-source="state.data" :columns="columns">
      <template #expandedRowRender="{ record }">
        <p style="margin: 0">
          {{ record.info }}
        </p>
      </template>
      <template #over="{text, record}">
        <a-tag color="processing" v-if="record.over === false" >
          <template #icon>
            <sync-outlined :spin="true" />
          </template>
          Pending request
        </a-tag>
        <a-tag color="success" v-else-if="record.agreeAmount >= record.goal / 2">
          <template #icon>
            <check-circle-outlined />
          </template>
          Request approved
        </a-tag>
        <a-tag color="error" v-else-if="record.agreeAmount < record.goal / 2">
          <template #icon>
            <close-circle-outlined />
          </template>
          Request declined
        </a-tag>
      </template>
      <template #action="{text, record}" v-if="amount != 0">
        <a-button v-if="(record.agree == 0 || record.agree == 2) && record.over === false" type="primary" @click="clickAgreeUse(true, record.index)">
          Approve
        </a-button>
        <a-divider type="vertical"></a-divider>
        <a-button v-if="(record.agree == 0 || record.agree == 1) && record.over === false" type="danger" @click="clickAgreeUse(false, record.index)">
          Disapprove
        </a-button>
      </template>
    </a-table>

    <Modal v-model:visible="isOpen">
      <a-card style="width: 600px; margin: 0 2em;" :body-style="{ overflowY: 'auto', maxHeight: '600px' }">
        <template #title><h3 style="text-align: center">Initiate a use request</h3></template>
        <create-form :model="model" :form="form" :fields="fields" />
      </a-card>
    </Modal>
  </div>
</template>

<script lang="ts">
import { message } from 'ant-design-vue';
import { defineComponent, ref, reactive, PropType, watch } from 'vue';
import { getAccount, getAllUse, Funding, agreeUse, newUse, Use, addListener } from '../api/contract'
import Modal from '../components/base/modal.vue'
import CreateForm from '../components/base/createForm.vue'
import { Model, Fields, Form } from '../type/form'
import { CheckCircleOutlined, SyncOutlined, CloseCircleOutlined } from '@ant-design/icons-vue'

const columns = [
  {
    dataIndex: 'info',
    key: 'info',
    title: 'Request introduction'
  },
  {
    dataIndex: 'goal',
    key: 'goal',
    title: 'Request amount(eth)'
  },
  {
    dataIndex: 'agreeAmount',
    key: 'agreeAmount',
    title: 'Approve request amount(eth)'
  },
  {
    dataIndex: 'disagree',
    key: 'disagree',
    title: 'Disapprove request amount(eth)'
  },
  {
    dataIndex: 'over',
    key: 'over',
    title: 'state',
    slots: { customRender: 'over' }
  },
  {
    dataIndex: 'action',
    key: 'action',
    title: 'operation',
    slots: { customRender: 'action' }
  }
]

export default defineComponent({
  name: 'Use',
  props: {
    id: Number,
    data: Object as PropType<Funding>,
    amount: Number,
  },
  components: { Modal, CreateForm, CheckCircleOutlined, SyncOutlined, CloseCircleOutlined },
  setup(props) {
    const state = reactive<{loading: boolean, data: Use[]}>({
      loading: true,
      data: []
    })
    const account = ref('')

    // ================
    const isOpen = ref(false);
    function openModal() { isOpen.value = true }
    function closeModal() { isOpen.value = false }

    const model = reactive<Model>({
      info: '',
      goal: 0
    })
    const fields = reactive<Fields>({
      info: {
        type: 'textarea',
        label: 'Request introduction'
      },
      goal: {
        type: 'number',
        min: 0,
        label: 'Request amount'
      }
    })
    const form = reactive<Form>({
      submitHint: 'Initiate a request',
      cancelHint: 'Cancel',
      cancel: () => {
        closeModal();
      },
      finish: async () => {
        try {
          await newUse(props.id as number, model.goal, model.info);
          message.success('Initiate the request successfully')
          fetchData();
          closeModal();
        } catch (e) {
          message.error('Failed to initiate request')
        }
      }
    })

    watch(() => props.data, () => {
      if(props.data) {
        fields.goal.max = props.data.amount;
      }
    })

    // ==================
    async function fetchData() {
      state.loading = true;
      try {
        state.data = await getAllUse(props.id as number);
        account.value = await getAccount();
        state.loading = false;
      } catch (e) {
        console.log(e);
        message.error("Failed to get request!");
      }
    }

    // ======= ==========
    async function clickAgreeUse(agree: boolean, useID: number) {
      try {
        await agreeUse(props.id as number, useID, agree);
        message.success('Operation successful')
        fetchData();
      } catch (e) {
        console.log(e);
        message.error('Operation failed')
      }
    }
    addListener(fetchData)
    fetchData();

    return { state, columns, account, isOpen, model, fields, form, openModal, clickAgreeUse }
  }
});
</script>
