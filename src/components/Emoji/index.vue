<!--
 * 表情组件
 * @author: dnhyxc
 * @since: 2023-04-13
 * index.vue
-->
<template>
  <div id="EMOJI" class="emoji-wrap">
    <div v-show="visible">
      <span
        v-for="(value, key, index) in emojiList"
        id="EMOJI_ITEM"
        :key="index"
        class="emoji-item"
        @click="addEmoji(key)"
      >
        <img id="EMOJI_IMG" class="emoji" :src="value" :title="key" width="24px" height="24px" />
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { EMOJI_TEXTS, EMOJI_URLS } from '@/constant';

interface IProps {
  showEmoji: boolean;
  addEmoji: (value: string) => void;
}

const props = withDefaults(defineProps<IProps>(), {
  showEmoji: false,
});

const emit = defineEmits(['update:showEmoji']);

const visible = computed({
  get() {
    return props.showEmoji;
  },
  set(visible: boolean) {
    emit('update:showEmoji', visible);
  },
});

// 处理表情数据
const emojiList = computed(() => {
  const result = {};
  EMOJI_TEXTS.forEach((i, index) => {
    result[`[${i}]`] = EMOJI_URLS[index + 1];
  });
  return result;
});

// 添加表情
const addEmoji = (key: string) => {
  props.addEmoji?.(key);
};
</script>

<style scoped lang="less">
@import '@/styles/index.less';

.emoji-wrap {
  .emoji-item {
    display: inline-block;
    height: 35px;
    width: 35px;

    .emoji {
      height: auto;
      width: 30px;
      cursor: pointer;
    }
  }
}
</style>
