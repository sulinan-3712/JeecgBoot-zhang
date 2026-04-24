<template>
  <template v-if="getShow">
    <div class="register-shell enter-x">
      <LoginFormTitle class="register-title enter-x" />
      <Form class="register-form enter-x" :model="formData" :rules="getFormRules" ref="formRef">
        <FormItem name="account" class="form-item enter-x">
          <Input class="glass-input fix-auto-fill" size="large" v-model:value="formData.account" :placeholder="t('sys.login.userName')" />
        </FormItem>
        <FormItem name="mobile" class="form-item enter-x">
          <Input size="large" v-model:value="formData.mobile" :placeholder="t('sys.login.mobile')" class="glass-input fix-auto-fill" />
        </FormItem>
        <FormItem name="sms" class="form-item enter-x">
          <CountdownInput
            size="large"
            class="glass-input fix-auto-fill"
            v-model:value="formData.sms"
            :placeholder="t('sys.login.smsCode')"
            :sendCodeApi="sendCodeApi"
          />
        </FormItem>
        <FormItem name="password" class="form-item enter-x">
          <StrengthMeter size="large" v-model:value="formData.password" :placeholder="t('sys.login.password')" class="glass-input" />
        </FormItem>
        <FormItem name="confirmPassword" class="form-item enter-x">
          <InputPassword
            size="large"
            visibilityToggle
            v-model:value="formData.confirmPassword"
            :placeholder="t('sys.login.confirmPassword')"
            class="glass-input"
          />
        </FormItem>

        <FormItem class="form-item policy-item enter-x" name="policy">
          <Checkbox v-model:checked="formData.policy" size="small">
            {{ t('sys.login.policy') }}
          </Checkbox>
        </FormItem>

        <Button type="primary" class="register-submit-btn enter-x" size="large" block @click="handleRegister" :loading="loading">
          {{ t('sys.login.registerButton') }}
        </Button>
        <Button size="large" block class="register-back-btn mt-4 enter-x" @click="handleBackLogin">
          {{ t('sys.login.backSignIn') }}
        </Button>
      </Form>
    </div>
  </template>
</template>
<script lang="ts" setup>
  import { reactive, ref, unref, computed, toRaw } from 'vue';
  import LoginFormTitle from './LoginFormTitle.vue';
  import { Form, Input, Button, Checkbox } from 'ant-design-vue';
  import { StrengthMeter } from '/@/components/StrengthMeter';
  import { CountdownInput } from '/@/components/CountDown';
  import { useI18n } from '/@/hooks/web/useI18n';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useLoginState, useFormRules, useFormValid, LoginStateEnum, SmsEnum } from './useLogin';
  import { register, getCaptcha } from '/@/api/sys/user';
  const FormItem = Form.Item;
  const InputPassword = Input.Password;
  const { t } = useI18n();
  const { handleBackLogin, getLoginState } = useLoginState();
  const { notification } = useMessage();
  const formRef = ref();
  const loading = ref(false);
  const formData = reactive({
    account: '',
    password: '',
    confirmPassword: '',
    mobile: '',
    sms: '',
    policy: false,
  });
  const { getFormRules } = useFormRules(formData);
  const { validForm } = useFormValid(formRef);
  const getShow = computed(() => unref(getLoginState) === LoginStateEnum.REGISTER);
  /**
   * 注册
   */
  async function handleRegister() {
    const data = await validForm();
    if (!data) return;
    try {
      loading.value = true;
      const resultInfo = await register(
        toRaw({
          username: data.account,
          password: data.password,
          phone: data.mobile,
          smscode: data.sms,
        })
      );
      if (resultInfo && resultInfo.data.success) {
        notification.success({
          description: resultInfo.data.message || t('sys.api.registerMsg'),
          duration: 3,
        });
        handleBackLogin();
      } else {
        notification.warning({
          message: t('sys.api.errorTip'),
          description: resultInfo.data.message || t('sys.api.networkExceptionMsg'),
          duration: 3,
        });
      }
    } catch (error) {
      notification.error({
        message: t('sys.api.errorTip'),
        description: error.message || t('sys.api.networkExceptionMsg'),
        duration: 3,
      });
    } finally {
      loading.value = false;
    }
  }
  //发送验证码的函数
  function sendCodeApi() {
    return getCaptcha({ mobile: formData.mobile, smsmode: SmsEnum.REGISTER });
  }
</script>

<style lang="less" scoped>
  .register-shell {
    padding: 40px 40px 28px;
  }

  .register-title {
    margin-bottom: 28px;
  }

  :deep(.register-title h2) {
    font-size: 34px;
    color: #e5eefc;
  }

  .register-form {
    margin-top: 0;
  }

  .form-item {
    margin-bottom: 16px;
  }

  :deep(.glass-input.ant-input),
  :deep(.glass-input .ant-input),
  :deep(.glass-input.ant-input-affix-wrapper),
  :deep(.glass-input .ant-input-affix-wrapper),
  :deep(.glass-input.ant-input-group-wrapper .ant-input-wrapper),
  :deep(.glass-input .@{namespace}-countdown-input) {
    height: 48px;
    color: #fff;
    border: 1px solid rgb(148 163 184 / 28%);
    border-radius: 12px;
    background: rgb(15 23 42 / 48%);
    transition: all 0.2s ease;
  }

  :deep(.glass-input.ant-input-affix-wrapper .ant-input),
  :deep(.glass-input.ant-input) {
    color: #fff;
    -webkit-text-fill-color: #fff;
  }

  :deep(.glass-input .ant-input-affix-wrapper .ant-input),
  :deep(.glass-input.ant-input-affix-wrapper .ant-input) {
    border: none;
    box-shadow: none;
    background: transparent;
  }

  :deep(.glass-input .ant-input::placeholder) {
    color: rgb(148 163 184 / 75%);
  }

  :deep(.glass-input.ant-input:focus),
  :deep(.glass-input.ant-input-affix-wrapper-focused),
  :deep(.glass-input.ant-input-affix-wrapper:focus-within),
  :deep(.glass-input .ant-input-affix-wrapper-focused),
  :deep(.glass-input .ant-input-affix-wrapper:focus-within),
  :deep(.glass-input.ant-input-group-wrapper .ant-input-wrapper:focus-within),
  :deep(.glass-input .@{namespace}-countdown-input:focus-within) {
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgb(59 130 246 / 18%);
  }

  :deep(.glass-input.ant-input-group-wrapper .ant-input-group-addon) {
    border: 1px solid rgb(148 163 184 / 28%);
    border-left: none;
    border-radius: 0 12px 12px 0;
    background: rgb(30 41 59 / 72%);
  }

  .policy-item {
    margin-bottom: 20px;
  }

  :deep(.ant-checkbox-wrapper) {
    color: rgb(203 213 225 / 85%);
  }

  .register-submit-btn {
    height: 48px;
    border: none;
    border-radius: 12px;
    background: linear-gradient(135deg, #4facfe, #00f2fe);
    font-size: 16px;
    font-weight: 500;
  }

  .register-back-btn {
    height: 44px;
    border: 1px solid rgb(148 163 184 / 30%);
    border-radius: 12px;
    background: rgb(15 23 42 / 42%);
    color: rgb(226 232 240 / 95%);
  }

  .register-back-btn:hover {
    border-color: rgb(96 165 250 / 65%);
    color: #dbeafe;
    background: rgb(30 41 59 / 70%);
  }
</style>
