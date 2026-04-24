<template>
  <div :class="prefixCls" class="relative w-full h-full px-4">
    <div class="login-bg-layer" />
    <div class="login-bg-mask" />
    <AppLocalePicker class="absolute text-white top-4 right-4 z-20 enter-x" :showText="false" v-if="!sessionTimeout && showLocale" />
    <AppDarkModeToggle class="absolute top-3 right-16 z-20 enter-x" v-if="!sessionTimeout" />
    <div class="relative z-10 flex items-center justify-center h-full py-8">
      <div :class="`${prefixCls}-form`" class="login-glass-card">
        <LoginForm />
        <ForgetPasswordForm />
        <RegisterForm />
        <MobileForm />
        <QrCodeForm />
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
  import { AppLocalePicker, AppDarkModeToggle } from '/@/components/Application';
  import LoginForm from './LoginForm.vue';
  import ForgetPasswordForm from './ForgetPasswordForm.vue';
  import RegisterForm from './RegisterForm.vue';
  import MobileForm from './MobileForm.vue';
  import QrCodeForm from './QrCodeForm.vue';
  import { useDesign } from '/@/hooks/web/useDesign';
  import { useLocaleStore } from '/@/store/modules/locale';
  import { useLoginState } from './useLogin';
  defineProps({
    sessionTimeout: {
      type: Boolean,
    },
  });

  const { prefixCls } = useDesign('login');
  const localeStore = useLocaleStore();
  const showLocale = localeStore.getShowPicker;
  const { handleBackLogin } = useLoginState();
  handleBackLogin();
</script>
<style lang="less">
  @prefix-cls: ~'@{namespace}-login';
  .@{prefix-cls} {
    min-height: 100%;
    overflow: hidden;
    background: #0f172a;

    .login-bg-layer {
      position: absolute;
      inset: 0;
      background-image: url('/@/assets/loginmini/icon/wlop.png');
      background-position: center;
      background-size: cover;
      filter: blur(8px);
      transform: scale(1.05);
    }

    .login-bg-mask {
      position: absolute;
      inset: 0;
      background: linear-gradient(135deg, rgb(10 25 50 / 60%), rgb(0 0 0 / 40%));
    }

    .login-glass-card {
      width: min(420px, calc(100vw - 32px));
      min-height: 520px;
      padding: 0;
      border: 1px solid rgb(255 255 255 / 40%);
      border-radius: 20px;
      backdrop-filter: blur(20px);
      background: rgb(255 255 255 / 85%);
      box-shadow: 0 20px 60px rgb(0 0 0 / 20%);
      animation: login-card-enter 0.5s ease-out both;
    }

    .ant-divider-inner-text {
      font-size: 12px;
      color: #6b7280;
    }
  }

  html[data-theme='dark'] {
    .@{prefix-cls} {
      .login-glass-card {
        border-color: rgb(148 163 184 / 35%);
        background: rgb(15 23 42 / 72%);
      }
    }
  }

  @keyframes login-card-enter {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
</style>
