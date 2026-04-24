<template>
  <Form class="forget-form enter-x" :model="formData" :rules="getFormRules" ref="formRef">
    <FormItem name="username" class="form-item enter-x">
      <Input size="large" v-model:value="formData.username" :placeholder="t('sys.login.userName')" disabled class="glass-input" />
    </FormItem>

    <FormItem name="password" class="form-item enter-x">
      <StrengthMeter size="large" v-model:value="formData.password" :placeholder="t('sys.login.password')" class="glass-input" />
    </FormItem>

    <FormItem name="confirmPassword" class="form-item enter-x">
      <InputPassword size="large" visibilityToggle v-model:value="formData.confirmPassword" :placeholder="t('sys.login.confirmPassword')" class="glass-input" />
    </FormItem>

    <FormItem class="form-item enter-x">
      <Button type="primary" size="large" block @click="handlePrev" class="forget-submit-btn"> 上一步 </Button>
      <Button size="large" block class="mt-4 forget-back-btn" @click="handleNext"> 下一步 </Button>
    </FormItem>
  </Form>
</template>
<script lang="ts">
  import { defineComponent, reactive, ref, computed, unref, toRaw, toRefs } from 'vue';
  import { Form, Input, Button } from 'ant-design-vue';
  import { StrengthMeter } from '/@/components/StrengthMeter';
  import { useI18n } from '/@/hooks/web/useI18n';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useFormRules, useFormValid } from '../login/useLogin';
  import { passwordChange } from '/@/api/sys/user';

  export default defineComponent({
    name: 'step2',
    components: {
      Button,
      Form,
      FormItem: Form.Item,
      InputPassword: Input.Password,
      Input,
      StrengthMeter,
    },
    props: {
      accountInfo: {
        type: Object,
        default: () => ({}),
      },
    },
    emits: ['prevStep', 'nextStep'],
    setup(props, { emit }) {
      const { t } = useI18n();
      const { createErrorModal } = useMessage();
      const { accountInfo } = props;
      const formRef = ref();
      const formData = reactive({
        username: accountInfo.obj.username || '',
        password: '',
        confirmPassword: '',
      });
      const { getFormRules } = useFormRules(formData);
      const { validForm } = useFormValid(formRef);

      /**
       * 上一步
       */
      function handlePrev() {
        emit('prevStep', accountInfo.obj);
      }

      /**
       * 下一步
       */
      async function handleNext() {
        const data = await validForm();
        if (!data) return;
        const resultInfo = await passwordChange(
          toRaw({
            username: data.username,
            password: data.password,
            smscode: accountInfo.obj.smscode,
            phone: accountInfo.obj.phone,
          })
        );
        if (resultInfo.success) {
          //修改密码
          emit('nextStep', accountInfo.obj);
        } else {
          //错误提示
          createErrorModal({
            title: t('sys.api.errorTip'),
            content: resultInfo.message || t('sys.api.networkExceptionMsg'),
          });
        }
      }

      return {
        t,
        formRef,
        formData,
        getFormRules,
        handleNext,
        handlePrev,
      };
    },
  });
</script>

<style lang="less" scoped>
  .forget-form {
    padding: 0;
  }

  .form-item {
    margin-bottom: 16px;
  }

  :deep(.glass-input.ant-input),
  :deep(.glass-input .ant-input),
  :deep(.glass-input.ant-input-affix-wrapper) {
    height: 48px;
    color: #fff;
    border: 1px solid rgb(148 163 184 / 28%);
    border-radius: 12px;
    background: rgb(15 23 42 / 48%);
  }

  :deep(.glass-input.ant-input-affix-wrapper .ant-input),
  :deep(.glass-input.ant-input) {
    color: #fff;
    -webkit-text-fill-color: #fff;
  }

  :deep(.glass-input .ant-input-affix-wrapper),
  :deep(.glass-input .ant-input) {
    border: none;
    box-shadow: none;
    background: transparent;
  }

  :deep(.glass-input .ant-input::placeholder) {
    color: rgb(148 163 184 / 75%);
  }

  .forget-submit-btn {
    height: 48px;
    border: none;
    border-radius: 12px;
    background: linear-gradient(135deg, #4facfe, #00f2fe);
    font-size: 16px;
    font-weight: 500;
  }

  .forget-back-btn {
    height: 44px;
    border: 1px solid rgb(148 163 184 / 30%);
    border-radius: 12px;
    background: rgb(15 23 42 / 42%);
    color: rgb(226 232 240 / 95%);
  }
</style>
