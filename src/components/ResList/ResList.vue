<template>
  <section class="ResList">
    <!-- 一键复制按钮 -->
    <button v-if="isAllUrlsValid" @click="copyAllUrls" class="copy-all-btn">一键复制所有URL</button>

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
import { computed } from 'vue';
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
  } catch {
    return ERROR_MSG; // 上传失败时返回错误信息
  }
  
  if (key == 'md') {
    return FILE_ID ? `![${v._vh_filename}](${props.nodeHost}/v2/${FILE_ID})` : ERROR_MSG;
  }
  return FILE_ID ? `${props.nodeHost}/v2/${FILE_ID}` : ERROR_MSG;
};

// 复制单个CODE
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

// 一键复制所有URL
const copyAllUrls = async () => {
  // 过滤上传成功的URL并移除上传失败的项
  const urls = props.modelValue
    .filter((item: any) => item.upload_result)  // 只选择上传成功的项
    .map((item: any) => formatURL(item.upload_result))  // 获取格式化后的URL
    .filter(url => !url.includes('上传失败'));  // 移除包含 "上传失败" 的项

  if (urls.length > 0) {
    const urlText = urls.join('\n');  // 将所有URL用换行符连接
    await copyCodeValue(urlText);
  } else {
    toast({ title: '提示', description: '没有有效的URL可复制' });
  }
};

// 检查是否所有条目都有有效的URL
const isAllUrlsValid = computed(() => {
  // 只要有一个有效URL就显示按钮
  return props.modelValue.some((item: any) => item.upload_result && formatURL(item.upload_result) !== `${item._vh_filename} 上传失败`);
});
</script>

<style scoped lang="less">
@import 'ResList.less';

.copy-all-btn {
  margin: 10px 0;
  padding: 10px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s ease;
}

.copy-all-btn:hover {
  background-color: #0056b3;
}

.copy-all-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}
</style>
