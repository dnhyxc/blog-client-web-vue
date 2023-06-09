<!--
 * 登录/注册页
 * @author: dnhyxc
 * @since: 2023-01-29
 * index.vue
-->
<template>
  <div class="login-content">
    <div class="title">账号密码登录</div>
    <el-form ref="formRef" :rules="rules" :model="loginForm" class="form-wrap">
      <el-form-item prop="username" class="form-item">
        <el-input v-model="loginForm.username" size="large" placeholder="请输入用户名" @keyup.enter="onEnter" />
      </el-form-item>
      <!-- <el-form-item
        prop="username"
        :rules="[
          {
            required: true,
            message: '用户名不能为空',
            trigger: 'blur',
          },
        ]"
        class="form-item"
      >
        <el-input v-model="loginForm.username" size="large" placeholder="请输入用户名" @keyup.enter="onEnter" />
      </el-form-item> -->
      <el-form-item prop="password" class="form-item">
        <el-input
          v-model="loginForm.password"
          size="large"
          placeholder="请输入密码"
          show-password
          @keyup.enter="onEnter"
        />
      </el-form-item>
      <el-form-item class="form-item action-list">
        <el-button type="primary" size="large" class="action" @click="onLogin(formRef)">登录</el-button>
        <el-button class="action" size="large" @click="onRegister(formRef)">注册</el-button>
      </el-form-item>
    </el-form>
    <div class="reset-wrap">
      <el-button class="action" link @click="onBackHome">返回首页</el-button>
      <el-button class="action" link @click="onForgetPwd">忘记密码</el-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router';
import { ref, reactive } from 'vue';
import type { FormInstance, FormRules } from 'element-plus';
import { loginStore } from '@/store';
import { verifyUsername, verifyPassword } from '@/utils';

const router = useRouter();

const formRef = ref<FormInstance>();

const validateUsername = (rule: any, value: any, callback: any) => {
  const { msg, status } = verifyUsername(value);
  if (value === '') {
    callback(new Error('用户名不能为空'));
  } else if (!status) {
    callback(new Error(msg));
  } else {
    callback();
  }
};

const validatePassword = (rule: any, value: any, callback: any) => {
  const { msg, status } = verifyPassword(value);
  if (value === '') {
    callback(new Error('用户名不能为空'));
  } else if (!status) {
    callback(new Error(msg));
  } else {
    callback();
  }
};

const rules = reactive<FormRules>({
  username: [{ validator: validateUsername, trigger: 'blur', required: true }],
  password: [{ validator: validatePassword, trigger: 'blur', required: true }],
});

const loginForm = reactive<{
  username: string;
  password: string;
}>({
  username: loginStore.userInfo?.username || '',
  password: '',
});

const emit = defineEmits(['switchDom']);

// 登录注册表单提交
const onSubmit = (formEl: FormInstance, type: string) => {
  formEl.validate(async (valid) => {
    if (valid) {
      if (type === 'login') {
        await loginStore.onLogin(loginForm, router);
      } else {
        loginStore.onRegister(loginForm);
      }
    } else {
      return false;
    }
  });
};

// 注册
const onRegister = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  onSubmit(formEl, 'register');
};

// 登录
const onLogin = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  onSubmit(formEl, 'login');
};

// 登陆注册回车事件
const onEnter = () => {
  if (!formRef.value) return;
  formRef.value.validate(async (valid) => {
    if (valid) {
      await loginStore.onLogin(loginForm, router);
    } else {
      return false;
    }
  });
};

// 返回首页
const onBackHome = () => {
  router.push('/home');
};

// 点击忘记密码切换组件
const onForgetPwd = () => {
  emit('switchDom', 'Reset');
};
</script>

<style lang="less" scoped>
@import '@/styles/index.less';

.login-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  box-sizing: border-box;
  height: 375px;
  padding: 20px;
  border-radius: 5px;
  background: rgba(225, 225, 225, 0.1);
  box-shadow: 0 0 2px @page-color inset;
  backdrop-filter: blur(1px);
  .title {
    height: 50px;
    line-height: 50px;
    padding: 0 50px;
    margin-bottom: 30px;
    font-size: 20px;
    font-weight: 700;
  }

  .form-wrap {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 500px;

    .form-item {
      margin-bottom: 30px;
      padding: 0 50px;
    }

    .action-list {
      display: flex;
      justify-content: space-between;
      margin-bottom: 0;
      margin-top: 10px;

      .action {
        flex: 1;
      }
    }
  }

  .reset-wrap {
    display: flex;
    justify-content: flex-end;
    padding: 0 50px;
    margin-top: 10px;

    .action {
      color: var(--theme-blue);
    }
  }
}
</style>
