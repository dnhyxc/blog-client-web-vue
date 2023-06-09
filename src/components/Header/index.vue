<!--
 * Header组件
 * @author: dnhyxc
 * @since: 2023-01-28
 * index.vue
-->
<template>
  <div :class="`${checkOS() === 'mac' && 'mac-header-wrap'} header-wrap`" @dblclick="onDblclick">
    <div class="left">
      <div class="icon-wrap">
        <i class="page-icon iconfont icon-haidao_" />
      </div>
      <el-tooltip effect="light" content="后退" placement="bottom">
        <i class="font iconfont icon-arrow-left-bold" @click="goBack" />
      </el-tooltip>
      <el-tooltip effect="light" content="前进" placement="bottom">
        <i class="font iconfont icon-arrow-right-bold" @click="goForward" />
      </el-tooltip>
      <div class="title">{{ commonStore.crumbsInfo.crumbsName }}</div>
    </div>
    <div class="right">
      <div class="search-wrap">
        <el-dropdown
          v-if="!commonStore.showSearch && NEED_HEAD_SEARCH.includes(route.path)"
          effect="light"
          content="搜索"
          placement="bottom"
        >
          <i class="font iconfont icon-sousuo2 search-icon" @click="onClickSearch" />
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item @click="onCheckSearchType(1)">普通搜索</el-dropdown-item>
              <el-dropdown-item @click="onCheckSearchType(2)">高级搜索</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
        <el-tooltip
          v-if="!commonStore.showSearch && !NEED_HEAD_SEARCH.includes(route.path) && route.path !== '/search'"
          effect="light"
          content="高级搜索"
          placement="bottom"
        >
          <i class="font iconfont icon-sousuo2 senior-search" @click="onClickSearch" />
        </el-tooltip>
        <el-tooltip v-if="commonStore.showSearch" effect="light" content="高级搜索" placement="bottom">
          <i class="iconfont icon-qiehuan" @click="onCheckSearchType(2)" />
        </el-tooltip>
        <el-input
          v-if="commonStore.showSearch"
          ref="searchRef"
          v-model.trim="search"
          class="search-inp"
          placeholder="请输入搜索内容"
          @keyup.enter="onEnter"
        >
          <template #suffix>
            <el-icon class="el-input__icon" @click="onEnter"><Search /></el-icon>
          </template>
        </el-input>
      </div>
      <el-popover
        v-if="loginStore.userInfo?.userId"
        v-model:visible="messageStore.visible"
        placement="bottom"
        title="我的消息列表"
        :width="300"
        trigger="click"
        popper-class="msg-popover"
      >
        <Messages />
        <template #reference>
          <div class="bell">
            <el-tooltip effect="light" content="消息" placement="bottom">
              <div class="msg">
                <span
                  v-if="messageStore.msgCount"
                  :class="`${messageStore.msgCount > 99 && 'max-msg-count'} msg-count`"
                >
                  {{ messageStore.msgCount > 99 ? '99+' : messageStore.msgCount }}
                </span>
                <i class="bell-font iconfont icon-notification" />
              </div>
            </el-tooltip>
          </div>
        </template>
      </el-popover>
      <div class="setting">
        <el-tooltip effect="light" content="设置" placement="bottom">
          <i class="font iconfont icon-shezhi" @click="toSetting" />
        </el-tooltip>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect, nextTick, onUnmounted, watch } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { Search } from '@element-plus/icons-vue';
import { MENULIST, NEED_HEAD_SEARCH } from '@/constant';
import { commonStore, messageStore, loginStore } from '@/store';
import { checkOS } from '@/utils';
import Messages from '@/components/Messages/index.vue';

const router = useRouter();
const route = useRoute();

const toggle = ref<boolean>(false);
const showSearch = ref<boolean>(false);
const search = ref<string>('');
const searchRef = ref<HTMLInputElement | null>(null);
const timerRef = ref<ReturnType<typeof setTimeout> | null>();

// 监听路由变化，设置当前选中菜单
watchEffect(() => {
  const menu = MENULIST.find((i) => route.path.includes(i.path));
  commonStore.setCrumbsInfo({
    crumbsName: menu?.name || '设置',
    crumbsPath: route.path,
  });
  commonStore.setActivePath(route.path);
});

// 清除副作用
onUnmounted(() => {
  if (timerRef.value) {
    clearTimeout(timerRef.value);
  }
  messageStore.visible = false;
});

// 监听页面搜索关键词，如果articleStore.keyword为空，则清除输入框内容
watch(
  () => commonStore.keyword,
  (newVal) => {
    if (!newVal) {
      search.value = '';
    }
  },
);

// 双击放大窗口
const onDblclick = () => {
  toggle.value = !toggle.value;
  // ipcRenderer.send('window-max');
};

// 后退
const goBack = () => {
  router.back();
};

// 前进
const goForward = () => {
  router.go(1);
};

// 点击去设置页
const toSetting = () => {
  commonStore.setCrumbsInfo({
    crumbsName: '账号设置',
    crumbsPath: '/setting',
  });
  router.push('/setting');
};

// 点击搜索
const onClickSearch = () => {
  if (NEED_HEAD_SEARCH.includes(route.path)) {
    showSearch.value = true;
    commonStore.showSearch = true;
    nextTick(() => {
      searchRef.value?.focus();
    });
  } else {
    commonStore.setCrumbsInfo({
      crumbsName: '高级搜索',
      crumbsPath: '/search',
    });
    router.push('/search');
  }
};

// 选择搜索类型
const onCheckSearchType = (value: number) => {
  if (value === 1 && NEED_HEAD_SEARCH.includes(route.path)) {
    showSearch.value = true;
    commonStore.showSearch = true;
    nextTick(() => {
      searchRef.value?.focus();
    });
  } else {
    commonStore.setCrumbsInfo({
      crumbsName: '高级搜索',
      crumbsPath: '/search',
    });
    router.push('/search');
  }
};

// 输入框回车
const onEnter = async (e: Event) => {
  const value = (e.target as HTMLInputElement).value;
  if (commonStore.keyword !== value) {
    commonStore.keyword = value;
  }
};
</script>

<style scoped lang="less">
@import '@/styles/index.less';

.header-wrap {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 35px;
  padding: 10px 18px 10px 12px;
  -webkit-app-region: drag;
  .left {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    color: var(--font-1);

    .icon-wrap {
      display: flex;
      align-items: center;
      justify-content: center;

      .page-icon {
        display: inline-block;
        min-height: 40px;
        line-height: 40px;
        font-size: 35px;
        margin-bottom: 2px;
        margin-right: 20px;
        color: @sub-2-blue;
        cursor: pointer;
        -webkit-app-region: no-drag;
        .textLg();
      }
    }
    .font {
      margin-right: 20px;
      cursor: pointer;
      -webkit-app-region: no-drag;
      color: var(--font-color);

      &:hover {
        color: var(--active-color);
        font-weight: 700;
      }
    }
    .title {
      font-size: 18px;
      font-weight: 700;
      color: var(--font-color);
    }
  }
  .right {
    display: flex;
    align-items: center;
    color: var(--font-1);
    .search-wrap {
      display: flex;
      align-items: center;
      -webkit-app-region: no-drag;
      color: var(--font-1);

      :deep {
        .el-dropdown {
          color: var(--font-1);
        }
      }

      .font {
        font-size: 15px;
        cursor: pointer;
        -webkit-app-region: no-drag;
        color: var(--font-color);
      }

      .senior-search {
        margin-top: -1px;
      }

      .icon-qiehuan {
        color: var(--font-color);
        font-size: 20px;
        cursor: pointer;
      }

      .search-inp {
        width: 180px;
        margin-left: 15px;
        -webkit-app-region: no-drag;

        .el-input__icon {
          font-size: 16px;
        }

        :deep {
          .el-input__wrapper {
            background-color: var(--input-bg-color);
            border-radius: 30px;
          }
          .el-input__inner {
            color: var(--font-color);
          }
        }
      }
    }
    .bell {
      position: relative;
      display: flex;
      align-items: center;
      -webkit-app-region: no-drag;
      margin-left: 15px;
      .msg-count {
        position: absolute;
        top: -5px;
        right: -5px;
        font-size: 12px;
        color: @font-danger;
        font-weight: 700;
        cursor: pointer;
      }

      .max-msg-count {
        right: -12px;
      }

      .bell-font {
        font-size: 17px;
        cursor: pointer;
        color: var(--font-color);
      }
    }
    .sticky {
      display: flex;
      align-items: center;
      justify-content: center;
      -webkit-app-region: no-drag;
      .font {
        font-size: 16px;
        cursor: pointer;
        margin-left: 15px;
        margin-top: 1px;
        color: var(--font-color);
      }
      .active {
        color: @sub-2-blue;
      }
    }
    .setting {
      display: flex;
      align-items: center;
      justify-content: center;
      -webkit-app-region: no-drag;
      .font {
        font-size: 19px;
        cursor: pointer;
        margin-left: 15px;
        color: var(--font-color);
      }
    }
    .page-actions {
      display: flex;
      justify-content: flex-start;
      align-items: center;
      width: 100%;
    }
    .icon {
      -webkit-app-region: no-drag;
      cursor: pointer;
      color: var(--font-color);

      .icon-text {
        margin-left: 15px;
        font-size: 16px;
      }
    }
  }

  .dl-content {
    .actions {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      padding-left: 20px;
    }
    .radio-close {
      padding: 0;
      margin-left: 0;
      margin-top: 20px;
      font-size: 16px;

      .font {
        margin-right: 10px;
        color: var(--theme-blue);
      }

      .out-icon {
        color: @font-warning;
      }
    }

    .close-info {
      display: flex;
      margin-top: 15px;

      .info-text {
        display: inline-block;
        margin-top: 2px;
      }
    }
  }

  :deep {
    .el-dialog__body {
      padding: 0 20px 15px 20px;
    }
  }
}

.mac-header-wrap {
  padding: 10 12px;
}
</style>
