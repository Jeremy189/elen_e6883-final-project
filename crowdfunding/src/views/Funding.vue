<template>
  <div>
    <a-card 
      class="ant-card-shadow" 
      :loading="state.loading" 
      :tab-list="tabList"
      :active-tab-key="key"
      @tabChange="onTabChange"
    >
      <template #title>
        <h3>
          {{state.data.title}}
          <span style="float:right">
            You have donated {{state.myAmount}} Eth
            <a-button type="primary" v-if="new Date(state.data.endTime * 1000) > new Date() && state.data.success == false" @click="openModal">I want to donate</a-button>
            <a-button type="danger" v-if="!state.data.success && state.myAmount != 0" @click="returnM"> refund </a-button>
          </span>
        </h3>
      </template>
      <a-descriptions bordered v-if="key==='info'">
        <a-descriptions-item label="Crowdfunding title" :span="2">
          {{state.data.title}}
        </a-descriptions-item>
        <a-descriptions-item label="Crowdfunding creator" :span="2">
          {{state.data.initiator}}
        </a-descriptions-item>
        <a-descriptions-item label="Deadline" :span="2">
           {{new Date(state.data.endTime * 1000).toLocaleString()}}
        </a-descriptions-item>
        <a-descriptions-item label="Current state">
          <a-tag color="success" v-if="state.data.success === true">
            <template #icon>
              <check-circle-outlined />
            </template>
            Crowdfunding success
          </a-tag>
          <a-tag color="processing" v-else-if="new Date(state.data.endTime * 1000) > new Date()" >
            <template #icon>
              <sync-outlined :spin="true" />
            </template>
            Crowdfunding
          </a-tag>
          <a-tag color="error" v-else>
            <template #icon>
              <close-circle-outlined />
            </template>
            Crowdfunding failed
          </a-tag>
        </a-descriptions-item>
        <a-descriptions-item label="Target amount">
          <a-statistic :value="state.data.goal">
            <template #suffix>
              Eth
            </template>
          </a-statistic>
        </a-descriptions-item>
        <a-descriptions-item label="Current amount">
          <a-statistic :value="state.data.amount">
            <template #suffix>
              Eth
            </template>
          </a-statistic>
        </a-descriptions-item>
        <a-descriptions-item label="Crowdfunding progress">
          <a-progress type="circle" :percent="state.data.success ? 100 : state.data.amount * 100 / state.data.goal" :width="80" />
        </a-descriptions-item>
        <a-descriptions-item label="Crowdfunding introduction">
          {{state.data.info}}
        </a-descriptions-item>
      </a-descriptions>

      <Use v-if="key==='use'" :id="id" :data="state.data" :amount="state.myAmount"></Use>

    </a-card>

    <Modal v-model:visible="isOpen">
      <a-card style="width: 600px; margin: 0 2em;" :body-style="{ overflowY: 'auto', maxHeight: '600px' }">
        <template #title><h3 style="text-align: center">Donate</h3></template>
        <create-form :model="model" :form="form" :fields="fields" />
      </a-card>
    </Modal>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed } from 'vue';
import { getOneFunding, Funding, getAccount, getMyFundingAmount, contribute, returnMoney, addListener } from '../api/contract'
import { useRoute } from 'vue-router'
import { message } from 'ant-design-vue';
import { CheckCircleOutlined, SyncOutlined, CloseCircleOutlined } from '@ant-design/icons-vue'
import Modal from '../components/base/modal.vue'
import CreateForm from '../components/base/createForm.vue'
import Use from '../components/Use.vue'
import { Model, Fields, Form } from '../type/form'

const column = [
  {
    dataIndex: ''
  }
]

const tabList = [
  {
    key: 'info',
    tab: 'introduction',
  },
  {
    key: 'use',
    tab: 'request',
  },
];

export default defineComponent({
  name: 'Funding',
  components: { Modal, CreateForm, CheckCircleOutlined, SyncOutlined, CloseCircleOutlined, Use },
  setup() {
    // ===================
    const route = useRoute();
    const id = parseInt(route.params.id as string);
    const account = ref('');
    const state = reactive<{data: Funding | {}, loading: boolean, myAmount: number}>({
      data: {},
      loading: true,
      myAmount: 0
    })

    // =======================
    const isOpen = ref(false);
    function openModal() { isOpen.value = true }
    function closeModal() { isOpen.value = false }

    const model = reactive<Model>({
      value: 1
    })
    const fields = reactive<Fields>({
      value: {
        type: 'number',
        min: 1,
        label: 'Donation amount'
      }
    })
    const form = reactive<Form>({
      submitHint: 'Donate',
      cancelHint: 'Cancel',
      cancel: () => {
        closeModal();
      },
      finish: async () => {
        try {
          await contribute(id, model.value);
          message.success('Donation is successful')
          fetchData();
          closeModal();
        } catch (e) {
          message.error('Donation is failed')
        }
      }
    })

    async function returnM() {
      try {
        await returnMoney(id);
        message.success('Refund successfully');
        fetchData()
      } catch(e) {
        message.error('Refund failed')
      }
    }

    // ====================
    const key = ref('info');
    const onTabChange = (k : 'use' | 'info') => {
      key.value = k;
    }

    // ====================
    async function fetchData() {
      state.loading = true;
      try {
        [state.data, state.myAmount] = await Promise.all([getOneFunding(id), getMyFundingAmount(id)]);
        state.loading = false;
        //@ts-ignore
        fields.value.max = state.data.goal - state.data.amount;
      } catch (e) {
        console.log(e);
        message.error('Failed to get details');
      }
    }

    addListener(fetchData)

    getAccount().then(res => account.value = res)
    fetchData();

    return {state, account, isOpen, openModal, form, model, fields, tabList, key, onTabChange, id, returnM}
  }
});
</script>
