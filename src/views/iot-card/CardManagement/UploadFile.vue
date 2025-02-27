<template>
    <j-space align="end">
        <j-upload
            v-model:fileList="modelValue.upload"
            name="file"
            :action="FILE_UPLOAD"
            :headers="{
                'X-Access-Token': LocalStore.get(TOKEN_KEY),
            }"
            :maxCount="1"
            :showUploadList="false"
            @change="uploadChange"
            :accept="
                props?.file ? `.${props?.file}` : '.xlsx'
            "
            :before-upload="beforeUpload"
        >
            <j-button>
                <template #icon><AIcon type="UploadOutlined" /></template>
                文件上传
            </j-button>
        </j-upload>
        <div style="margin-left: 20px">
            <j-space>
                下载模板
                <a @click="downFile('xlsx')">.xlsx</a>
                <a @click="downFile('csv')">.csv</a>
            </j-space>
        </div>
    </j-space>
    <div style="margin-top: 20px" v-if="importLoading">
        <j-badge v-if="flag" status="processing" text="进行中" />
        <j-badge v-else status="success" text="已完成" />
        <span>总数量：{{ count }}</span>
        <p style="color: red">{{ errMessage }}</p>
    </div>
</template>

<script lang="ts" setup>
import { FILE_UPLOAD } from '@/api/comm';
import { TOKEN_KEY } from '@/utils/variable';
import { LocalStore, onlyMessage } from '@/utils/comm';
import { downloadFileByUrl } from '@/utils/utils';
import { exportCard, _import } from '@/api/iot-card/cardManagement';
import { message } from 'jetlinks-ui-components';

type Emits = {
    (e: 'update:modelValue', data: string[]): void;
};
const emit = defineEmits<Emits>();

const props = defineProps({
    // 组件双向绑定的值
    modelValue: {
        type: Array,
        default: () => [],
    },
    product: {
        type: String,
        default: '',
    },
    file: {
        type: String,
        default: 'xlsx',
    },
});

const importLoading = ref<boolean>(false);
const flag = ref<boolean>(false);
const count = ref<number>(0);
const errMessage = ref<string>('');

const downFile = async (type: string) => {
    const res: any = await exportCard(type);
    if (res) {
        const blob = new Blob([res], { type: type });
        const url = URL.createObjectURL(blob);
        downloadFileByUrl(url, `物联卡导入模板`, type);
    }
};

const beforeUpload = (_file: any) => {
    const fileType = props?.file === 'csv' ? 'csv' : 'xlsx';
    const isCsv = _file.type === 'text/csv';
    const isXlsx =
        _file.type ===
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';
    if (!isCsv && fileType !== 'xlsx') {
        onlyMessage('请上传.csv格式文件', 'warning');
    }
    if (!isXlsx && fileType !== 'csv') {
        onlyMessage('请上传.xlsx格式文件', 'warning');
    }
    return (isCsv && fileType !== 'xlsx') || (isXlsx && fileType !== 'csv');
};

const uploadChange = async (info: Record<string, any>) => {
    importLoading.value = true;
    if (info.file.status === 'done') {
        const resp: any = info.file.response || { result: '' };
        flag.value = true;
        _import(props.product, { fileUrl: resp.result })
            .then((response: any) => {
                count.value = response.result?.total || 0
                message.success('导入成功');
            })
            .catch((err) => {
                errMessage.value = err?.response?.data?.message || '导入失败'
            })
            .finally(() => {
                flag.value = false;
            });
    }
};
</script>