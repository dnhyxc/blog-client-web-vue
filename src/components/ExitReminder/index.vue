<!--
 * 退出提醒
 * @author: dnhyxc
 * @since: 2023-04-26
 * index.vue
-->
<template>
  <div class="exit-reminder-wrap">
    <div class="message">
      <div class="info">您的账号已在别处重新登录了</div>
      <div class="button" @click="onQuit()">确定</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { inject } from 'vue';
import { loginStore } from '@/store';

const reload = inject<Function>('reload');

// 退出登录
const onQuit = () => {
  loginStore.onQuit();
  loginStore.logoutStatus = false;
  // window.location.reload();
  reload && reload();
};
</script>

<style scoped lang="less">
@import '@/styles/index.less';

.exit-reminder-wrap {
  position: fixed;
  top: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background-color: var(--pop-before-bg-color);
  z-index: 102;

  .message {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 350px;
    height: 220px;
    border-radius: 5px;
    margin-top: -30px;
    padding: 10px;
    box-shadow: 0 0 10px var(--shadow-mack);
    background-color: var(--background);

    .info {
      margin-bottom: 50px;
      color: var(--font-1);
      text-align: center;
      font-size: 18px;
    }

    .button {
      width: 60%;
      height: 38px;
      line-height: 38px;
      text-align: center;
      box-shadow: 0 0 5px var(--shadow-mack);
      border-radius: 5px;
      cursor: pointer;
      .clickNoSelectText();
      .bgMoveColor(135deg);
      .bgKeyframes(bgmove);
    }
  }
}
</style>
