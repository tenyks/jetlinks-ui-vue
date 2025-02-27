<template>
    <page-container>
        <j-spin :spinning="loading">
          <div v-if="type && id === ':id'">
            <Provider
              @onClick="goProviders"
              :dataSource="dataSource"
            ></Provider>
          </div>
            <FullPage  v-else>
                <j-card :bordered="false">
                    <div>
                        <div class="go-back" v-if="id === ':id'">
                            <a @click="goBack">返回</a>
                        </div>
                      <template v-if="showType === 'network'">
                        <Network
                          v-if="provider.id !== 'plugin_gateway'"
                          :provider="provider"
                          :data="data"
                        />
                        <Plugin
                          v-else
                          :provider="provider"
                          :data="data"
                        />
                      </template>

                        <Media
                            v-if="showType === 'media'"
                            :provider="provider"
                            :data="data"
                        />
                        <Channel
                            v-if="showType === 'channel'"
                            :provider="provider"
                            :data="data"
                        />
                        <Edge
                            v-if="showType === 'edge'"
                            :provider="provider"
                            :data="data"
                        />
                        <Cloud
                            v-if="showType === 'cloud'"
                            :provider="provider"
                            :data="data"
                        />
                    </div>
                </j-card>
            </FullPage>
        </j-spin>
    </page-container>
</template>

<script lang="ts" setup name="AccessConfigDetail">
import Network from '../components/Network/index.vue';
import Provider from '../components/Provider/index.vue';
import Media from '../components/Media/index.vue';
import Channel from '../components/Channel/index.vue';
import Edge from '../components/Edge/index.vue';
import Cloud from '../components/Cloud/index.vue';
import Plugin from '../components/Plugin/index.vue'
import { getProviders, detail } from '@/api/link/accessConfig';
import { accessConfigTypeFilter } from '@/utils/setting';

const route = useRoute();
const id = route.params.id as string;

const dataSource: any = ref([]);
const type = ref(false);
const loading = ref(true);
const provider = ref({});
const data = ref({});
const showType: any = ref('');

const goProviders = (param: any) => {
    showType.value = param.type;
    provider.value = param;
    type.value = false;
};

const goBack = () => {
    provider.value = {};
    type.value = true;
};

//network
const TypeMap = new Map([
    ['fixed-media', 'media'],
    ['gb28181-2016', 'media'],
    ['OneNet', 'cloud'],
    ['Ctwing', 'cloud'],
    ['modbus-tcp', 'channel'],
    ['opc-ua', 'channel'],
    ['official-edge-gateway', 'edge'],
    ['edge-child-device', 'edge'],
    ['network', 'network'],
]);
// DataMap后期优化
const DataMap = new Map();
DataMap.set('fixed-media', { type: 'media', title: '视频类设备接入' });
DataMap.set('gb28181-2016', { type: 'media', title: '视频类设备接入' });
DataMap.set('OneNet', { type: 'cloud', title: '云平台接入' });
DataMap.set('Ctwing', { type: 'cloud', title: '云平台接入' });
DataMap.set('modbus-tcp', { type: 'channel', title: '通道类设备接入' });
DataMap.set('opc-ua', { type: 'channel', title: '通道类设备接入' });
DataMap.set('official-edge-gateway', { type: 'edge', title: '官方接入' });
DataMap.set('edge-child-device', { type: 'edge', title: '官方接入' });
DataMap.set('network', { type: 'network', title: '自定义设备接入' });

const getTypeList = (result: Record<string, any>) => {
    const list = [];
    const media: any[] = [];
    const network: any[] = [];
    const cloud: any[] = [];
    const channel: any[] = [];
    const edge: any[] = [];
    result.map((item: any) => {
        if (item.id === 'fixed-media' || item.id === 'gb28181-2016') {
            item.type = 'media';
            media.push(item);
        } else if (item.id === 'OneNet' || item.id === 'Ctwing') {
            item.type = 'cloud';
            cloud.push(item);
        } else if (item.id === 'modbus-tcp' || item.id === 'opc-ua') {
            item.type = 'channel';
            channel.push(item);
        } else if (
            item.id === 'official-edge-gateway' ||
            item.id === 'edge-child-device'
        ) {
            item.type = 'edge';
            edge.push(item);
        } else {
            item.type = 'network';
            network.push(item);
        }
    });

    network.length &&
        list.push({
            list: [...network],
            title: '自定义设备接入',
        });
    media.length &&
        list.push({
            list: [...media],
            title: '视频类设备接入',
        });
    cloud.length &&
        list.push({
            list: [...cloud],
            title: '云平台接入',
        });
    channel.length &&
        list.push({
            list: [...channel],
            title: '通道类设备接入',
        });
    edge.length &&
        list.push({
            list: [...edge],
            title: '官方接入',
        });

    return list;
};

const queryProviders = async () => {
    const resp: any = await getProviders();
    if (resp.status === 200) {
        const _data = resp.result || [];
        dataSource.value = getTypeList(accessConfigTypeFilter(_data as any[]));
        // dataSource.value = getTypeList(resp.result)[0].list.filter(
        //     (item) => item.name !== '插件设备接入',
        // );

        // 快速添加接入网关
        if (route.query.save && route.query?.type) {
            const type = route.query.type;
            goProviders(
                dataSource.value
                    .find((f: any) => f.title === DataMap.get(type).title)
                    ?.list?.find((f: any) => f.id === type),
            );
        }
    }
};

const getProvidersData = async () => {
    if (id !== ':id') {
        getProviders().then((response: any) => {
            if (response.status === 200) {
                const _data = response.result || [];
                const list = getTypeList(
                    accessConfigTypeFilter(_data as any[]),
                );
                dataSource.value = list.filter(
                    (item: any) =>
                        item.channel === 'network' ||
                        item.channel === 'child-device',
                );
                detail(id).then((resp: any) => {
                    if (resp.status === 200) {
                        const dt = response.result.find(
                            (item: any) => item?.id === resp.result.provider,
                        );
                        response.result.forEach((item: any) => {
                            if (item.id === resp.result.provider) {
                                resp.result.type = TypeMap.has(item.id)
                                    ? TypeMap.get(item.id)
                                    : TypeMap.get('network');
                                showType.value = resp.result.type;
                            }
                        });

                        provider.value = dt;
                        data.value = resp.result;
                        type.value = false;
                    }
                });
            }
            loading.value = false;
        });
    } else {
        type.value = true;
        queryProviders();
        loading.value = false;
    }
};

onMounted(() => {
    loading.value = true;
    getProvidersData();
});
</script>

<style lang="less" scoped>
.go-back {
    margin: 0 0 20px 0;
}
</style>
