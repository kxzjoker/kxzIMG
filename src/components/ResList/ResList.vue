<template>
  <section class="ResList">
    <!-- 添加一键复制按钮 -->
    <div class="copy-all-button">
      <button @click="copyAllURLs">一键复制所有图片URL</button>
    </div>

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

// 一键复制所有图片的URL
const copyAllURLs = async () => {
  const allURLs = props.modelValue
    .filter((item: any) => item.upload_result)  // 过滤出有上传结果的项
    .map((item: any) => formatURL(item.upload_result)); // 获取每个项的URL

  if (allURLs.length > 0) {
    const allURLsText = allURLs.join('\n'); // 把URLs用换行符拼接成一个长文本
    try {
      await navigator.clipboard.writeText(allURLsText);  // 复制到剪贴板
      toast({ title: 'Tips', description: '所有图片URL已复制！' });
    } catch {
      const i = document.createElement('textarea');
      i.value = allURLsText;
      document.body.appendChild(i);
      i.select();
      document.execCommand('copy');
      document.body.removeChild(i);
      toast({ title: 'Tips', description: '所有图片URL已复制！' });
    }
  } else {
    toast({ title: '警告', description: '没有找到有效的图片URL！' });
  }
};
</script>

<style scoped lang="less">
@import 'ResList.less';

// 添加样式
.copy-all-button {
  text-align: center;
  margin-bottom: 20px;
}

.copy-all-button button {
  padding: 10px 20px;
  font-size: 14px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

.copy-all-button button:hover {
  background-color: #45a049;
}
</style>
