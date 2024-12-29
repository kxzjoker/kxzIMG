<script setup lang="ts">
import { ref, computed } from 'vue';  // 添加 computed 导入
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

// 计算是否有上传图片
const hasUploadedImages = computed(() => {
  return props.modelValue.some((item: any) => item.upload_result);
});
</script>
