<template>
  <template v-if="getShow">
    <div class="forget-shell enter-x">
      <a-steps class="forget-steps" :current="currentTab">
        <a-step title="手机验证" />
        <a-step title="更改密码" />
        <a-step title="完成" />
      </a-steps>
      <div>
        <step1 v-if="currentTab === 0" @nextStep="nextStep" />
        <step2 v-if="currentTab === 1" @nextStep="nextStep" @prevStep="prevStep" :accountInfo="accountInfo" />
        <step3 v-if="currentTab === 2" @prevStep="prevStep" @finish="finish" />
      </div>
    </div>
  </template>
</template>
<script lang="ts" setup>
  import { reactive, ref, computed, unref } from 'vue';
  import { useLoginState, useFormRules, LoginStateEnum } from './useLogin';
  import step1 from '../forget-password/step1.vue';
  import step2 from '../forget-password/step2.vue';
  import step3 from '../forget-password/step3.vue';
  const { handleBackLogin, getLoginState } = useLoginState();
  const { getFormRules } = useFormRules();

  const formRef = ref();
  const loading = ref(false);
  const currentTab = ref(0);
  const formData = reactive({
    account: '',
    mobile: '',
    sms: '',
  });
  const getShow = computed(() => unref(getLoginState) === LoginStateEnum.RESET_PASSWORD);
  const accountInfo = reactive({
    obj: {
      username: '',
      phone: '',
      smscode: '',
    },
  });
  /**
   * 下一步
   * @param data
   */
  function nextStep(data) {
    accountInfo.obj = data;
    if (currentTab.value < 4) {
      currentTab.value += 1;
    }
  }
  /**
   * 上一步
   * @param data
   */
  function prevStep(data) {
    accountInfo.obj = data;
    if (currentTab.value > 0) {
      currentTab.value -= 1;
    }
  }
  /**
   * 结束
   */
  function finish() {
    currentTab.value = 0;
  }
</script>

<style lang="less" scoped>
  .forget-shell {
    padding: 40px 40px 28px;
  }

  .forget-steps {
    margin-bottom: 24px;
  }

  :deep(.forget-steps .ant-steps-item-title) {
    color: rgb(203 213 225 / 88%) !important;
  }

  :deep(.forget-steps .ant-steps-item-description) {
    color: rgb(148 163 184 / 75%);
  }

  :deep(.forget-steps .ant-steps-item-process .ant-steps-item-title) {
    color: #e5eefc !important;
  }

  :deep(.forget-steps .ant-steps-item-wait .ant-steps-item-icon),
  :deep(.forget-steps .ant-steps-item-finish .ant-steps-item-icon) {
    background: rgb(15 23 42 / 55%);
    border-color: rgb(148 163 184 / 28%);
  }
</style>
