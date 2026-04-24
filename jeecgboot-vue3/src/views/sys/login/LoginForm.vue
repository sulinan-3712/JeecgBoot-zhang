<template>
  <div v-show="getShow" class="login-form-shell enter-x">
    <div class="login-head">
      <img src="/@/assets/images/su.png" alt="logo" class="login-avatar" />
      <h2 class="login-title">Welcome Back</h2>
      <p class="login-sub-title">登录到 sulinan 平台</p>
    </div>

    <Form class="login-form" :model="formData" :rules="getFormRules" ref="formRef" @keypress.enter="handleLogin">
      <FormItem name="account" class="form-item">
        <Input v-model:value="formData.account" :placeholder="t('sys.login.userName')" class="glass-input fix-auto-fill">
          <template #prefix><UserOutlined class="input-prefix-icon" /></template>
        </Input>
      </FormItem>
      <FormItem name="password" class="form-item">
        <InputPassword visibilityToggle v-model:value="formData.password" :placeholder="t('sys.login.password')" class="glass-input">
          <template #prefix><LockOutlined class="input-prefix-icon" /></template>
        </InputPassword>
      </FormItem>

      <div class="captcha-row">
        <FormItem name="inputCode" class="form-item captcha-input-item">
          <Input v-model:value="formData.inputCode" :placeholder="t('sys.login.inputCode')" class="glass-input">
            <template #prefix><SafetyCertificateOutlined class="input-prefix-icon" /></template>
          </Input>
        </FormItem>
        <div class="captcha-image-wrap" @click="handleChangeCheckCode">
          <img v-if="randCodeData.requestCodeSuccess" class="captcha-image" :src="randCodeData.randCodeImage" />
          <img v-else class="captcha-image" src="../../../assets/images/checkcode.png" />
        </div>
      </div>

      <div class="assist-row">
        <Checkbox v-model:checked="rememberMe" size="small">
          {{ t('sys.login.rememberMe') }}
        </Checkbox>
        <Button type="link" size="small" class="forget-link" @click="setLoginState(LoginStateEnum.RESET_PASSWORD)">
          {{ t('sys.login.forgetPassword') }}
        </Button>
      </div>

      <FormItem class="form-item login-submit-item">
        <Button type="primary" block @click="handleLogin" :loading="loading" class="login-submit-btn">
          {{ t('sys.login.loginButton') }}
        </Button>
      </FormItem>

      <FormItem class="form-item register-entry-item">
        <Button block class="register-entry-btn" @click="setLoginState(LoginStateEnum.REGISTER)">
          {{ t('sys.login.registerButton') }}
        </Button>
      </FormItem>

      <Divider class="other-login-divider">其他登录方式</Divider>

      <div class="third-login-row" :class="`${prefixCls}-sign-in-way`">
        <button type="button" class="third-login-btn" @click="onThirdLogin('github')" title="GitHub">
          <GithubFilled />
          <span>GitHub</span>
        </button>
        <button type="button" class="third-login-btn" @click="onThirdLogin('wechat_open')" title="微信">
          <WechatFilled />
          <span>微信</span>
        </button>
        <button type="button" class="third-login-btn" @click="onThirdLogin('dingtalk')" title="钉钉">
          <DingtalkCircleFilled />
          <span>钉钉</span>
        </button>
      </div>
    </Form>
  </div>
  <!-- 第三方登录相关弹框 -->
  <ThirdModal ref="thirdModalRef"></ThirdModal>
</template>
<script lang="ts" setup>
  import { reactive, ref, toRaw, unref, computed, onMounted } from 'vue';

  import { Checkbox, Form, Input, Button, Divider } from 'ant-design-vue';
  import { GithubFilled, WechatFilled, DingtalkCircleFilled, UserOutlined, LockOutlined, SafetyCertificateOutlined } from '@ant-design/icons-vue';
  import ThirdModal from './ThirdModal.vue';
  import { useI18n } from '/@/hooks/web/useI18n';
  import { useMessage } from '/@/hooks/web/useMessage';

  import { useUserStore } from '/@/store/modules/user';
  import { LoginStateEnum, useLoginState, useFormRules, useFormValid } from './useLogin';
  import { useDesign } from '/@/hooks/web/useDesign';
  import { getCodeInfo } from '/@/api/sys/user';
  import {  encryptAESCBC } from '/@/utils/cipher';

  const FormItem = Form.Item;
  const InputPassword = Input.Password;
  const { t } = useI18n();
  const { notification, createErrorModal } = useMessage();
  const { prefixCls } = useDesign('login');
  const userStore = useUserStore();

  const { setLoginState, getLoginState } = useLoginState();
  const { getFormRules } = useFormRules();

  const formRef = ref();
  const thirdModalRef = ref();
  const loading = ref(false);
  const rememberMe = ref(false);

  const formData = reactive({
    account: 'admin',
    password: '123456',
    inputCode: '',
  });
  const randCodeData = reactive({
    randCodeImage: '',
    requestCodeSuccess: false,
    checkKey: null,
  });

  const { validForm } = useFormValid(formRef);

  //onKeyStroke('Enter', handleLogin);

  const getShow = computed(() => unref(getLoginState) === LoginStateEnum.LOGIN);

  async function handleLogin() {
    const data = await validForm();
    if (!data) return;
    try {
      loading.value = true;

      // 密码使用AES加密传输
      const encryptedPassword = encryptAESCBC(data.password);
      const { userInfo } = await userStore.login(
        toRaw({
          password: encryptedPassword,
          username: data.account,
          captcha: data.inputCode,
          checkKey: randCodeData.checkKey,
          mode: 'none', //不要默认的错误提示
        })
      );
      if (userInfo) {
        notification.success({
          message: t('sys.login.loginSuccessTitle'),
          description: `${t('sys.login.loginSuccessDesc')}: ${userInfo.realname}`,
          duration: 3,
        });
      }
    } catch (error) {
      notification.error({
        message: t('sys.api.errorTip'),
        description: error.message || t('sys.api.networkExceptionMsg'),
        duration: 3,
      });
      loading.value = false;
      handleChangeCheckCode();
    }
  }
  function handleChangeCheckCode() {
    formData.inputCode = '';
    // 代码逻辑说明: [QQYUN-10775]验证码可以复用 #7674------------
    randCodeData.checkKey = new Date().getTime() + Math.random().toString(36).slice(-4); // 1629428467008;
    getCodeInfo(randCodeData.checkKey).then((res) => {
      randCodeData.randCodeImage = res;
      randCodeData.requestCodeSuccess = true;
    });
  }

  /**
   * 第三方登录
   * @param type
   */
  function onThirdLogin(type) {
    thirdModalRef.value.onThirdLogin(type);
  }
  //初始化验证码
  onMounted(() => {
    handleChangeCheckCode();
  });
</script>

<style lang="less" scoped>
  .login-form-shell {
    height: 100%;
    padding: 40px 40px 28px;
  }

  .login-head {
    text-align: center;
  }

  .login-avatar {
    width: 104px;
    height: 104px;
    object-fit: cover;
    border-radius: 50%;
    border: 2px solid rgb(255 255 255 / 30%);
    box-shadow: 0 8px 24px rgb(15 23 42 / 45%);
  }

  .login-title {
    margin: 14px 0 6px;
    font-size: 40px;
    font-weight: 600;
    line-height: 1.2;
    color: #e5eefc;
    letter-spacing: 0.3px;
  }

  .login-sub-title {
    margin: 0;
    font-size: 14px;
    color: rgb(203 213 225 / 72%);
  }

  .login-form {
    margin-top: 40px;
  }

  .form-item {
    margin-bottom: 16px;
  }

  :deep(.glass-input.ant-input),
  :deep(.glass-input .ant-input),
  :deep(.glass-input.ant-input-affix-wrapper) {
    height: 48px;
    color: #e2e8f0;
    border: 1px solid rgb(148 163 184 / 28%);
    border-radius: 12px;
    background: rgb(15 23 42 / 48%);
    transition: all 0.2s ease;
  }

  :deep(.glass-input.ant-input-affix-wrapper .ant-input) {
    height: 100%;
    border: none;
    border-radius: 0;
    background: transparent;
    box-shadow: none;
    color: #fff;
    -webkit-text-fill-color: #fff;
  }

  :deep(.glass-input.ant-input) {
    color: #fff;
    -webkit-text-fill-color: #fff;
  }

  :deep(.glass-input.ant-input-affix-wrapper input) {
    color: #fff;
    -webkit-text-fill-color: #fff;
  }

  :deep(.glass-input .ant-input::placeholder) {
    color: rgb(148 163 184 / 75%);
  }

  :deep(.glass-input.ant-input-affix-wrapper .ant-input-password-icon) {
    color: rgb(148 163 184 / 80%);
  }

  :deep(.glass-input .ant-input-prefix) {
    color: rgb(148 163 184 / 85%);
  }

  :deep(.glass-input.ant-input:focus),
  :deep(.glass-input.ant-input-affix-wrapper-focused),
  :deep(.glass-input.ant-input-affix-wrapper:focus-within) {
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgb(59 130 246 / 18%);
  }

  .input-prefix-icon {
    font-size: 14px;
  }

  .captcha-row {
    display: grid;
    grid-template-columns: minmax(0, 1fr) 122px;
    gap: 12px;
    align-items: start;
  }

  .captcha-input-item {
    margin-bottom: 0;
  }

  .captcha-image-wrap {
    height: 48px;
    overflow: hidden;
    cursor: pointer;
    border: 1px solid rgb(148 163 184 / 26%);
    border-radius: 12px;
    background: rgb(15 23 42 / 58%);
  }

  .captcha-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .assist-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-top: 16px;
    margin-bottom: 24px;
    font-size: 12px;
    color: rgb(203 213 225 / 75%);
  }

  .forget-link {
    padding-right: 0;
    font-size: 12px;
    color: rgb(203 213 225 / 75%);
  }

  .login-submit-item {
    margin-bottom: 18px;
  }

  .login-submit-btn {
    height: 48px;
    border: none;
    border-radius: 12px;
    background: linear-gradient(135deg, #4facfe, #00f2fe);
    font-size: 16px;
    font-weight: 500;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  .login-submit-btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 8px 20px rgb(0 0 0 / 20%);
  }

  .login-submit-btn:active {
    animation: login-btn-press 0.2s ease;
  }

  .other-login-divider {
    margin: 16px 0;
    font-size: 12px;
    color: rgb(203 213 225 / 72%);
  }

  .register-entry-item {
    margin-bottom: 12px;
  }

  .register-entry-btn {
    height: 44px;
    border: 1px solid rgb(148 163 184 / 30%);
    border-radius: 12px;
    background: rgb(15 23 42 / 42%);
    color: rgb(226 232 240 / 95%);
  }

  .register-entry-btn:hover {
    border-color: rgb(96 165 250 / 65%);
    color: #dbeafe;
    background: rgb(30 41 59 / 70%);
  }

  .third-login-row {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 10px;
  }

  .third-login-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
    height: 40px;
    border: 1px solid rgb(148 163 184 / 26%);
    border-radius: 10px;
    background: rgb(15 23 42 / 42%);
    color: rgb(203 213 225 / 85%);
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .third-login-btn:hover {
    background: rgb(30 41 59 / 75%);
    color: #f8fafc;
  }

  :deep(.ant-checkbox-wrapper) {
    color: rgb(203 213 225 / 85%);
  }

  :deep(.ant-divider::before),
  :deep(.ant-divider::after) {
    border-top-color: rgb(148 163 184 / 24%);
  }

  @keyframes login-btn-press {
    0% {
      transform: scale(1);
    }
    50% {
      transform: scale(0.97);
    }
    100% {
      transform: scale(1);
    }
  }
</style>
