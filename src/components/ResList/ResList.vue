<template>
  <section class="ResList">
    <div class="item" v-for="(i, idx) in props.modelValue" :key="idx">
      <img v-if="i.upload_blob" class="thumb" :src="i.upload_result ? i.upload_blob : LoadingImg" />
      <div class="value" :class="{ active: !i.upload_result }">
        <p><input :value="i.upload_result ? formatURL(i.upload_result) : ''" type="text" readonly @click="i.upload_result && copyCodeValue(formatURL(i.upload_result))" /> <span>URL</span></p>
        <p><input :value="i.upload_result ? formatURL(i.upload_result, 'md') : ''" type="text" readonly @click="i.upload_result && copyCodeValue(formatURL(i.upload_result, 'md'))" /> <span>Markdown</span></p>
      </div>
      <HoverCard v-if="i.upload_result" :open-delay="0" :close-delay="0">
        <HoverCardTrigger as-child>
          <QrcodeVue class="qrcode" :value="formatURL(i.upload_result)" :size="56" level="H" />
        </HoverCardTrigger>
        <HoverCardContent class="w-max h-max"><QrcodeVue class="qrcode scale" :value="formatURL(i.upload_result)" :size="666" level="H" /></HoverCardContent>
      </HoverCard>
    </div>

    <!-- 一键复制按钮 -->
    <button v-if="canCopy" @click="copyAllUrls" class="copy-button">
      一键复制所有图片URL
    </button>
  </section>
</template>

<script setup lang="ts">
import QrcodeVue from 'qrcode.vue';
import { useToast } from '@/components/ui/toast/use-toast';
const { toast } = useToast();
import LoadingImg from '@/assets/images/loading.gif';
const props = defineProps(['modelValue', 'nodeHost']);
import { HoverCard, HoverCardContent, HoverCardTrigger } from '@/components/ui/hover-card';

// URL格式化
const formatURL = (v: any, key?: string) => {
  let FILE_ID = '';
  const ERROR_MSG = `${v._vh_filename} 上传失败`;
  try {
    FILE_ID = v.data.link.split('/').slice(-1)[0];
  } catch {}
  if (key == 'md') {
    return FILE_ID ? `![${v._vh_filename}](${props.nodeHost}/v2/${FILE_ID})` : ERROR_MSG;
  }
  return FILE_ID ? `${props.nodeHost}/v2/${FILE_ID}` : ERROR_MSG;
};

// 复制单个URL
const copyCodeValue = async (v: string) => {
  let vhCopyStatus: any = false;
  try {
    await navigator.clipboard.writeText(v);
    vhCopyStatus = true;
  } catch {
    const i = document.createElement('textarea');
    i.value = v;
    document.body.appendChild(i);
    i.select();
    vhCopyStatus = document.execCommand('copy');
    document.body.removeChild(i);
  } finally {
    if (vhCopyStatus) toast({ title: 'Tips', description: '复制成功' });
  }
};

// 获取所有有效的URL
const getAllValidUrls = () => {
  return props.modelValue
    .map(i => i.upload_result ? formatURL(i.upload_result) : null)
    .filter(url => url !== null) as string[];
};

// 检查是否有可复制的URL
const canCopy = computed(() => getAllValidUrls().length > 0);

// 一键复制所有图片URL
const copyAllUrls = async () => {
  const allUrls = getAllValidUrls().join('\n');  // 拼接所有URL
  await copyCodeValue(allUrls);  // 调用复制方法
};
</script>

<style scoped lang="less">
@import 'ResList.less';

/* 按钮样式 */
.copy-button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.copy-button:hover {
  background-color: #45a049;
}
</style>
