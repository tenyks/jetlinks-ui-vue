<template>
    <j-modal
        class="edit-dialog-container"
        width="1050px"
        visible
        :title="dialogTitle"
        :confirmLoading="loading"
        @ok="confirm"
        @cancel="emits('cancel')"
    >
        <j-form ref="formRef" :model="form.data" layout="vertical">
            <j-row :gutter="24">
                <j-col :span="12">
                    <j-form-item
                        name="name"
                        label="名称"
                        :rules="[
                            { required: true, message: '请输入名称' },
                            { max: 64, message: '最多可输入64个字符' },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.name"
                            placeholder="请输入名称"
                        />
                    </j-form-item>
                </j-col>
                <j-col :span="12">
                    <j-form-item
                        name="typeId"
                        label="类型"
                        :rules="[{ required: true, message: '请选择类型' }]"
                    >
                        <j-select
                            v-model:value="form.data.typeId"
                            :options="form.typeOptions"
                            placeholder="请选择类型"
                            :disabled="!!form.data.id"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24" v-if="form.data.typeId === 'rdb'">
                <j-col :span="24">
                    <j-form-item
                        :name="['shareConfig', 'url']"
                        label="URL"
                        :rules="[
                            {
                                required: true,
                                message: '请输入URL',
                                trigger: 'change',
                            },
                            { validator: checkUrl, trigger: 'blur' },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.shareConfig.url"
                            placeholder="请输入r2bdc或者jdbc连接地址，示例：r2dbc:mysql://127.0.0.1:3306/test"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24" v-if="form.data.typeId === 'rabbitmq'">
                <j-col :span="24">
                    <j-form-item
                        :name="['shareConfig', 'adminUrl']"
                        label="管理地址"
                        :rules="[
                            { required: true, message: '请输入管理地址' },
                            { validator: validateAdminUrl },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.shareConfig.adminUrl"
                            placeholder="请输入管理地址，示例：http://localhost:15672"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24" v-if="form.data.typeId === 'rabbitmq'">
                <j-col :span="24">
                    <j-form-item
                        :name="['shareConfig', 'addresses']"
                        label="链接地址"
                        :rules="[
                            { required: true, message: '请输入链接地址' },
                            { validator: validateAddress },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.shareConfig.addresses"
                            placeholder="请输入链接地址，示例：localhost:5672"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24" v-show="form.data.typeId">
                <j-col :span="12">
                    <j-form-item
                        :name="['shareConfig', 'username']"
                        label="用户名"
                        :rules="[
                            { required: true, message: '请输入用户名' },
                            {
                                max: 64,
                                message: '最多可输入64个字符',
                            },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.shareConfig.username"
                            placeholder="请输入用户名"
                        />
                    </j-form-item>
                </j-col>
                <j-col :span="12">
                    <j-form-item
                        :name="['shareConfig', 'password']"
                        label="密码"
                        :rules="[
                            { required: true, message: '请输入密码' },
                            {
                                max: 64,
                                message: '最多可输入64个字符',
                            },
                        ]"
                    >
                        <j-input-password
                            v-model:value="form.data.shareConfig.password"
                            placeholder="请输入密码"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24" v-if="form.data.typeId === 'rabbitmq'">
                <j-col :span="24">
                    <j-form-item
                        :name="['shareConfig', 'virtualHost']"
                        label="虚拟域"
                        :rules="[
                            { required: true, message: '请输入虚拟域' },
                            {
                                max: 64,
                                message: '最多可输入64个字符',
                            },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.shareConfig.virtualHost"
                            placeholder="请输入虚拟域"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24" v-if="form.data.typeId === 'rdb'">
                <j-col :span="24">
                    <j-form-item
                        :name="['shareConfig', 'schema']"
                        label="schema"
                        :rules="[
                            { required: true, message: '请输入schema' },
                            {
                                max: 64,
                                message: '最多可输入64个字符',
                            },
                        ]"
                    >
                        <j-input
                            v-model:value="form.data.shareConfig.schema"
                            placeholder="请输入schema"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
            <j-row :gutter="24">
                <j-col :span="24">
                    <j-form-item name="description" label="说明">
                        <j-textarea
                            v-model:value="form.data.description"
                            placeholder="请输入说明"
                            :rows="3"
                            showCount
                            :maxlength="200"
                        />
                    </j-form-item>
                </j-col>
            </j-row>
        </j-form>
    </j-modal>
</template>

<script setup lang="ts">
import {
    getDataTypeDict_api,
    saveDataSource_api,
} from '@/api/system/dataSource';
import { FormInstance } from 'ant-design-vue';
import { message } from 'jetlinks-ui-components';
import { Rule } from 'ant-design-vue/lib/form';
import type { dictItemType, optionItemType, sourceItemType } from '../typing';

const emits = defineEmits(['confirm', 'cancel']);
const props = defineProps<{
    data: sourceItemType;
}>();
// 弹窗相关
const dialogTitle = computed(() =>
    props.data.id ? '编辑数据源' : '新增数据源',
);
const loading = ref(false);

const checkUrl = (_rule: Rule, value: string): Promise<any> => {
    if (!value) return Promise.resolve();
    const arr = value.split(':');
    if (arr?.[0] === 'jdbc' || arr?.[0] === 'r2dbc') {
        return Promise.resolve();
    } else {
        return Promise.reject('请输入正确的URL');
    }
};

/**
 * 管理地址校验
 */
const validateAdminUrl = (_rule: Rule, value: string): Promise<any> => {
    return new Promise((resolve, reject) => {
        if (!value) {
            resolve('');
        } else {
            const arr = value.split('://');
            if (arr[0] === 'http' || arr[0] === 'https') {
                resolve('');
            } else {
                reject('请输入正确的管理地址');
            }
        }
    });
};
/**
 * 链接地址校验
 * @param _rule
 * @param value
 */
const validateAddress = (_rule: Rule, value: string): Promise<any> => {
    return new Promise((resolve, reject) => {
        if (!value) {
            resolve('');
        } else {
            const reg =
                /(http|ftp|https):\/\/[\w\-_]+(\.[\w\-_]+)+([\w\-\.,@?^=%&:/~\+#]*[\w\-\@?^=%&/~\+#])?/;
            if (reg.test(value)) {
                resolve('');
            } else {
                reject('请输入正确的链接地址');
            }
        }
    });
};

const getTypeOption = () => {
    getDataTypeDict_api().then((resp: any) => {
        const result = resp.result as dictItemType[];
        form.typeOptions = result.map((item) => ({
            label: item.name,
            value: item.id,
        }));
    });
};

const formRef = ref<FormInstance>();
const form = reactive({
    data: {
        ...props.data,
    } as sourceItemType,
    typeOptions: [] as optionItemType[],
});

watch(
    () => props.data,
    (newValue) => {
        form.data = { ...newValue, shareConfig: { ...newValue?.shareConfig } };
    },
    {
        immediate: true,
        deep: true,
    },
);

onMounted(() => {
    getTypeOption();
});

const confirm = () => {
    loading.value = true;
    formRef.value
        ?.validate()
        .then(async (_data: any) => {
            const resp = await saveDataSource_api({ ...props.data, ..._data });
            if (resp.status === 200) {
                message.success('操作成功');
                emits('confirm');
                formRef.value?.resetFields();
            }
        })
        .finally(() => {
            loading.value = false;
        });
};
</script>
