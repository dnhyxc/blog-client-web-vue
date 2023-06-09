<!--
 * 我的主页
 * @author: dnhyxc
 * @since: 2023-01-30
 * index.vue
-->
<template>
  <Loading :loading="personalStore.loading" class="personal-wrap">
    <template #default>
      <div class="header">
        <div class="left">
          <div class="head-wrap">
            <Image :url="personalStore.userInfo?.headUrl || HEAD_IMG" :transition-img="HEAD_IMG" class="head-img" />
          </div>
        </div>
        <div class="right">
          <div class="userInfo">
            <div class="username">{{ personalStore.userInfo?.username || '-' }}</div>
            <div class="job">{{ personalStore.userInfo?.job || '-' }}</div>
            <div class="motto">{{ personalStore.userInfo?.motto || '-' }}</div>
            <el-tooltip v-if="personalStore.userInfo?.introduce?.length! > 50" placement="top" effect="light">
              <template #content>
                <span class="introduce-tip">{{ personalStore.userInfo?.introduce }}</span>
              </template>
              <div class="desc">{{ personalStore.userInfo?.introduce || '-' }}</div>
            </el-tooltip>
            <div v-else class="desc">{{ personalStore.userInfo?.introduce || '-' }}</div>
          </div>
          <div class="actions">
            <div class="top">
              <i
                v-for="icon in iconLinks"
                v-show="icon.href"
                :key="icon.name"
                :class="`${icon.className} icon font iconfont ${icon.name}`"
                @click.stop="onClickLink(icon.href, icon.label)"
              />
            </div>
            <div v-if="isShowCollectActions" class="bottom">
              <el-button class="edit-btn" type="primary" @click="toSetting">修改个人资料</el-button>
            </div>
          </div>
        </div>
      </div>
      <el-scrollbar ref="scrollRef" wrap-class="scrollbar-wrapper">
        <div
          v-if="isMounted"
          v-infinite-scroll="getMyArticleList"
          :infinite-scroll-delay="300"
          :infinite-scroll-disabled="disabled"
          :infinite-scroll-distance="2"
          class="pullup-content"
        >
          <div class="content">
            <div v-if="personalStore.currentTabKey === '1'" class="collect-count-info">
              <span class="add-collect" @click.stop="onAddCollect">
                <i class="iconfont icon-add" />
                新建收藏集
              </span>
              <span class="collect-count">
                {{ !userId || userId === loginStore.userInfo?.userId ? '我' : '他' }}创建的
                {{ personalStore.collectTotal }}
              </span>
              <span class="collect-count">
                {{ !userId || userId === loginStore.userInfo?.userId ? '我' : '他' }}收藏的文章
                {{ personalStore.collectedCount }}
              </span>
            </div>
            <el-tabs v-model="personalStore.currentTabKey" type="border-card" class="el-tabs" @tab-change="onTabChange">
              <el-tab-pane
                v-for="tab in tabs"
                :key="tab.value"
                :label="tab.name"
                :class="`${!personalStore.articleList?.length && 'tab-pane'}`"
              >
                <div v-if="tab.value !== '2'" class="list-wrap">
                  <LineCard
                    v-for="data in personalStore.articleList"
                    :key="data.id"
                    :data="data"
                    class="author-line-card"
                    :delete-article="deleteArticle"
                    :like-list-article="likeListArticle"
                  />
                </div>
                <div v-else class="list-wrap">
                  <LineCard
                    v-for="data in personalStore.articleList"
                    :key="data.id"
                    :data="data"
                    is-collect
                    class="author-line-card collect-card"
                    @click.stop="toDetail(data.id!)"
                  >
                    <template #title>
                      <div class="collect-name">
                        <span class="title">
                          {{ data.name }}
                          <i v-show="Number(data.status) === 2" class="iconfont icon-33" />
                        </span>
                        <span v-if="isShowCollectActions" class="actions">
                          <i class="edit iconfont" @click.stop="onEditCollect(data as any)">编辑</i>
                          <i class="del iconfont" @click.stop="deleteCollection(data.id)">删除</i>
                        </span>
                      </div>
                    </template>
                    <template #content>
                      <div class="collect-content">
                        <span class="collect-desc">{{ data.desc }}</span>
                        <div class="collect-info">
                          <span class="collect-date">
                            {{ formatDate(data.createTime!, 'YYYY-MM-DD') }}更新 · {{ data.articleIds?.length }}
                            篇文章
                          </span>
                        </div>
                      </div>
                    </template>
                  </LineCard>
                </div>
                <Empty v-if="showEmpty" />
              </el-tab-pane>
            </el-tabs>
          </div>
          <ToTopIcon v-if="scrollTop >= 500" :on-scroll-to="onScrollTo" class="to-top" />
        </div>
        <div v-if="noMore" class="no-more">没有更多了～～～</div>
      </el-scrollbar>
    </template>
  </Loading>
  <AddCollectModel
    v-model:collect-visible="collectVisible"
    v-model:build-visible="buildVisible"
    :default-values="currentCollectValues"
    :is-edit="isEdit"
  />
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watchEffect } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';
import { useDeleteArticle, useScroller } from '@/hooks';
import { articleStore, loginStore, personalStore } from '@/store';
import { formatDate, scrollTo, checkUrl } from '@/utils';
import { ArticleItem, CollectParams } from '@/typings/common';
import { HEAD_IMG, ICONLINKS, ABOUT_ME_TABS, ABOUT_TABS } from '@/constant';
import AddCollectModel from '@/components/AddCollectModel/index.vue';
import Empty from '@/components/Empty/index.vue';

const route = useRoute();
const router = useRouter();
const { scrollRef, scrollTop } = useScroller();
const { deleteArticle } = useDeleteArticle({ pageType: 'personal', accessUserId: loginStore.userInfo?.userId });

const { authorId: userId } = route.query;

const isMounted = ref<boolean>(false);
// 收藏集弹窗显隐状态
const collectVisible = ref<boolean>(false);
// 创建收藏集弹窗显隐状态
const buildVisible = ref<boolean>(false);
// 传入创建收藏集弹窗，判断是否是编辑
const isEdit = ref<boolean>(false);
const currentCollectValues = ref<CollectParams>({
  name: '',
  desc: '',
  status: '1',
});

const noMore = computed(() => {
  const { articleList, total } = personalStore;
  return articleList.length >= total && articleList.length;
});
const disabled = computed(() => personalStore.loading || noMore.value);
const showEmpty = computed(
  () => personalStore.loading !== null && !personalStore.loading && !personalStore.articleList?.length,
);
// 判断是否展示删除、编辑收藏集的按钮
const isShowCollectActions = computed(() => !userId || userId === loginStore.userInfo?.userId);

// 根据用户信息动态计算tabs
const tabs = computed(() => {
  if (userId && userId !== loginStore.userInfo.userId) {
    return ABOUT_TABS;
  } else {
    return ABOUT_ME_TABS;
  }
});

// 动态获取ICONLINKS
const iconLinks = computed(() => {
  return ICONLINKS.map((i) => {
    return {
      ...i,
      href: loginStore.userInfo?.[i.label],
    };
  });
});

onMounted(async () => {
  // 防止页面加载报错
  isMounted.value = true;
  // 清空原始数据
  personalStore.clearArticleList();
  if (userId && userId !== loginStore.userInfo?.userId) {
    // 获取个人主页信息
    await personalStore.getUserInfo(userId as string);
  } else {
    // 当userId等于登录人的userId时，直接将loginStore中的用户信息赋值给personalStore
    personalStore.userInfo = loginStore.userInfo;
  }
  getMyArticleList();
});

watchEffect(() => {
  // currentTabKey为1的时候，说明是收藏tab，需要获取收藏的文章总数和收藏集数量
  if (personalStore.currentTabKey === '1') {
    personalStore.getCollectedTotal();
    personalStore.getCollectionTotal();
  }
});

onUnmounted(() => {
  // 清空原始数据
  personalStore.clearArticleList();
});

// 获取各tab中的文章列表
const getMyArticleList = async () => {
  await personalStore.getMyArticleList();
};

// tab 切换
const onTabChange = (value: string) => {
  // 设置选中tab，value：0：我/他的文章，1：我的收藏，2：点赞文章
  personalStore.currentTabKey = value;
  // 切换时清空原有数据
  personalStore.clearArticleList();
  // 切换tab时，重新加载数据
  getMyArticleList();
};

// 文章点赞
const likeListArticle = async (id: string, data?: ArticleItem) => {
  await articleStore.likeListArticle({ id, pageType: 'personal', data });
  // 取消点赞文章重新刷新列表之后，自动滚动到之前查看页面的位置
  if (personalStore.currentTabKey === '2') {
    onScrollTo(scrollTop.value);
  }
};

// 新增收藏集
const onAddCollect = () => {
  currentCollectValues.value = { name: '', desc: '', status: '1' };
  buildVisible.value = true;
};

// 编辑收藏集
const onEditCollect = (data: CollectParams) => {
  currentCollectValues.value = data;
  buildVisible.value = true;
  isEdit.value = true;
};

// 删除收藏集
const deleteCollection = (id: string) => {
  personalStore.delCollection(id);
};

// 前往收藏集详情
const toDetail = (id: string) => {
  router.push(`/collect/${id}?authorId=${userId || loginStore.userInfo?.userId}`);
};

// 去修改资料
const toSetting = () => {
  router.push('/setting');
};

// 点击juejin、github等链接
const onClickLink = (href: string, name: string) => {
  if (checkUrl(href)) {
    // 使用浏览器打开链接
    // shell.openExternal(href);
    window.open(href)
  } else {
    ElMessage({
      message: `${name} 链接无法使用`,
      type: 'warning',
      offset: 80,
    });
  }
};

// 置顶
const onScrollTo = (to?: number) => {
  scrollTo(scrollRef, to || 0);
};
</script>

<style scoped lang="less">
@import '@/styles/index.less';

.personal-wrap {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;

  .header {
    display: flex;
    justify-content: flex-start;
    padding: 10px;
    background-image: linear-gradient(225deg, var(--bg-lg-color1) 0%, var(--bg-lg-color2) 100%);
    border-radius: 5px;
    margin-bottom: 10px;

    .left {
      width: 130px;
      height: 130px;
      margin-right: 10px;

      .head-wrap {
        width: 100%;
        height: 100%;
        border-radius: 5px;

        .head-img {
          width: 100%;
          height: 100%;
          border-radius: 5px;
          .imgStyle();
        }

        :deep {
          .image-item {
            border-top-left-radius: 5px;
            border-top-right-radius: 5px;
          }
        }
      }
    }

    .right {
      flex: 1;
      display: flex;
      justify-content: space-between;

      .userInfo {
        display: flex;
        justify-content: space-between;
        flex-direction: column;
        margin-right: 10px;
        color: var(--font-2);

        .username {
          font-size: 18px;
          font-weight: 700;
          .ellipsisMore(1);
        }

        .job,
        .motto,
        .desc {
          font-size: 14px;
          .ellipsisMore(1);
        }

        .desc {
          cursor: pointer;
        }
      }

      .actions {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: flex-end;

        .top {
          display: flex;
          justify-content: center;
          align-items: center;
          width: 100%;
          .icon {
            display: block;
            width: auto;
            margin-left: 10px;
            cursor: pointer;

            &:first-child {
              margin-left: 0;
            }
          }
          .font {
            display: flex;
            justify-content: center;
            align-content: center;
            font-size: 28px;
            color: var(--theme-blue);
            padding: 2px;
            border-radius: 50px;

            &:hover {
              color: var(--active-color);
            }
          }

          .blog-icon {
            font-size: 25px;
          }
        }

        .bottom {
          width: 100%;

          .edit-btn {
            width: 100%;
          }
        }
      }
    }
  }

  .content {
    position: relative;
    border-radius: 5px;
    flex: 1;
    .el-tabs {
      border: 1px solid var(--card-border);
      border-radius: 5px;
      background-color: transparent;

      :deep {
        .el-tabs__item.is-active {
          background-color: transparent;
          font-weight: 700;
        }
        .el-tabs__content {
          padding: 10px;
        }

        .el-tabs__header,
        .el-tabs__nav-wrap {
          border-top-left-radius: 5px;
          border-top-right-radius: 5px;
          background-color: transparent;
        }
        .el-tabs__header {
          border-bottom: 1px solid var(--card-border);

          .el-tabs__item {
            color: var(--font-color);
          }

          .el-tabs__item.is-active {
            border-left-color: var(--card-border);
            border-right-color: var(--card-border);
            color: var(--theme-blue);
          }
        }
      }

      .tab-pane {
        min-height: 200px;
      }

      .list-wrap {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;

        .author-line-card {
          width: calc(50% - 5px);
          padding: 10px 10px;
          box-shadow: 0 0 5px var(--shadow-color);
          background-image: linear-gradient(225deg, var(--bg-lg-color1) 0%, var(--bg-lg-color2) 100%);
          margin-bottom: 10px;
          border-radius: 5px;
          margin-right: 10px;

          :deep {
            .art-info {
              flex: 1;
            }
          }

          &:nth-child(even) {
            margin-right: 0;
            &:last-child {
              margin-bottom: 0;
            }
          }

          &:nth-child(odd) {
            &:nth-last-child(2) {
              margin-bottom: 0;
            }
            &:last-child {
              margin-bottom: 0;
            }
          }

          .img-wrap {
            box-sizing: border-box;
            display: flex;
            width: 150px;

            .img {
              display: block;
              width: 100%;
              height: auto;
              border-radius: 5px;
              .imgStyle();
            }
          }
        }

        .collect-card {
          display: flex;
          flex-direction: column;

          .collect-name {
            display: flex;
            justify-content: space-between;
            width: 100%;

            .title {
              display: flex;
              align-items: center;
              flex: 1;
              font-size: 16px;
              font-weight: 700;
              margin-right: 20px;
              .ellipsisMore(1);

              .icon-33 {
                display: inline-block;
                font-size: 15px;
                margin-left: 3px;
              }
            }

            .actions {
              .edit {
                margin-right: 10px;
                color: var(--theme-blue);
                font-size: 14px;
              }

              .del {
                color: @font-danger;
                font-size: 14px;
              }
            }
          }

          .collect-content {
            display: flex;
            flex-direction: column;
            justify-content: space-between;

            .collect-desc {
              margin-bottom: 10px;
              font-size: 14px;
              .ellipsisMore(2);
            }

            .collect-info {
              font-size: 14px;
            }
          }
        }
      }
    }
    .collect-count-info {
      position: absolute;
      top: 9px;
      right: 22px;
      z-index: 12;

      .add-collect {
        font-size: 14px;
        color: var(--theme-blue);
        cursor: pointer;
        .clickNoSelectText();
      }

      .collect-count {
        font-size: 14px;
        background-image: linear-gradient(135deg, var(--head-lg-color1) 10%, var(--head-lg-color2) 100%);
        padding: 2px 5px 3px;
        border-radius: 5px;
        color: @font-2;
        margin-left: 10px;
      }
    }
  }

  .no-more {
    text-align: center;
    color: var(--font-4);
    margin: 15px 0 0;
  }
}

.introduce-tip {
  display: inline-block;
  width: 300px;
}
</style>
