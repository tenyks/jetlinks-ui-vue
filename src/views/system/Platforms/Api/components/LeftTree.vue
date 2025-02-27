<template>
    <j-spin :spinning="spinning">
        <template #indicator>
            <AIcon type="LoadingOutlined" />
        </template>
        <j-tree
            :tree-data="treeData"
            @select="clickSelectItem"
            v-model:selected-keys="selectedKeys"
            showLine
            class="left-tree-container"
        >
            <template #title="{ name }">
                {{ name }}
            </template>
        </j-tree>
    </j-spin>
</template>

<script setup lang="ts">
import { TreeProps } from 'ant-design-vue';

import {
    apiOperations_api,
    getApiGranted_api,
    getTreeOne_api,
    getTreeTwo_api,
} from '@/api/system/apiPage';
import type { modeType, treeNodeTpye } from '../typing';

const emits = defineEmits(['select']);
const props = defineProps<{
    mode: modeType;
    hasHome?: boolean;
    code?: string;
}>();

const treeData = ref<TreeProps['treeData']>([]);
const selectedKeys = ref<string[]>([]);
const spinning = ref(false);
const getTreeData = () => {
    let tree: treeNodeTpye[] = [];
    spinning.value = true;
    getTreeOne_api().then((resp: any) => {
        tree = resp.urls.map((item: any) => ({
            ...item,
            key: item.url,
        }));
        const allPromise = tree.map((item) => getTreeTwo_api(item.name));
        // 若类型不为api，根据不同类型添加得到不同的过滤数组
        if (props.mode === 'appManger') allPromise.push(apiOperations_api());
        else if (props.mode === 'home')
            allPromise.push(getApiGranted_api(props.code as string));
        Promise.all(allPromise)
            .then((values) => {
                values.forEach((item: any, i) => {
                    if (props.mode === 'api') {
                        tree[i].schemas = item.components.schemas;
                        tree[i].children = combData(item.paths);
                    } else if (i < values.length - 1) {
                        const paths = filterPath(
                            item.paths,
                            values[values.length - 1].result as string[],
                        );
                        tree[i].children = combData(paths);
                        tree[i].schemas = item.components.schemas;
                    }
                });
                if (props.hasHome) {
                    tree.unshift({
                        key: 'home',
                        name: '首页',
                        schemas: {},
                        children: [],
                    });
                    selectedKeys.value = ['home'];
                }

                treeData.value = tree;
            })
            .finally(() => {
                spinning.value = false;
            });
    });
};
const clickSelectItem: TreeProps['onSelect'] = (key: any[], node: any) => {
    if (key[0] === 'home') return emits('select', node.node.dataRef, {});

    if (!node.node.parent && key[0] !== 'home') return;
    emits('select', node.node.dataRef, node.node?.parent.node.schemas);
};

onMounted(() => {
    getTreeData();
});

const combData = (dataSource: object) => {
    const apiList: treeNodeTpye[] = [];
    const keys = Object.keys(dataSource);

    keys.forEach((key) => {
        const method = Object.keys(dataSource[key] || {})[0];
        const name = dataSource[key][method].tags[0];
        let apiObj: treeNodeTpye | undefined = apiList.find(
            (item) => item.name === name,
        );
        if (apiObj) {
            apiObj.apiList?.push({
                url: key,
                method: dataSource[key],
            });
        } else {
            apiObj = {
                name,
                key: name,
                apiList: [
                    {
                        url: key,
                        method: dataSource[key],
                    },
                ],
            };

            apiList.push(apiObj);
        }
    });

    return apiList;
};
/**
 * 过滤能展示的接口  模式mode为api时不需要过滤
 * @param path 源数据
 * @param filterArr 过滤数组
 */
const filterPath = (path: object, filterArr: string[]) => {
    for (const key in path) {
        if (Object.prototype.hasOwnProperty.call(path, key)) {
            const value = path[key];
            for (const prop in value) {
                if (Object.prototype.hasOwnProperty.call(value, prop)) {
                    const item = value[prop];
                    if (!filterArr.includes(item.operationId))
                        delete value[prop];
                }
            }
            if (Object.keys(value).length === 0) delete path[key];
        }
    }
    return path;
};
</script>

<style lang="less">
.left-tree-container {
    .ant-tree-list {
        .ant-tree-list-holder-inner {
            .ant-tree-switcher-noop {
                display: none !important;
            }
        }
    }
}
</style>
