<template>
    <Layout>
        <template v-slot:navbar>
            <div class="navbar">
                <img class="navbarImage" src="../assets/icons/nav.svg" alt="">
            </div>
        </template>

        <template v-slot:content1>
            <div class="content1">
                <div class="icons1">
                    <el-row :gutter="20">
                        <!-- 病虫害识别 -->
                        <el-col :span="6">
                            <router-link to="/resultShow">
                                <div for="fileInput" @click="openFile">
                                    <svg class="icon-bingchonghai" aria-hidden="true">
                                        <use xlink:href="#icon-bingchonghai"></use>
                                    </svg>
                                </div>
                            </router-link>
                            <div class="text">Disease and Pest <br>Identification</div>
                        </el-col>
                        <input id="fileInput" type="file" accept="image/*" style="display: none;"
                            @change="handleFileInputChange">
                        <!-- 病虫害识别 -->

                        <Icons1 iconName="cechan" text="Using Photography to Estimate<br> Tree Fruit Yield" />
                        <Icons1 iconName="huaqi" text="Using Photography to Determine<br>Flowering Time" />
                        <Icons1 iconName="shaoqi" text="Using Photography to Determine <br>Shoot Stage" />

                        <Icons1 iconName="fangan" text="Recommended Medication <br> Formulations" />
                        <Icons1 iconName="nongyao" text="Pesticide Mixture <br>Testing" />
                        <Icons1 iconName="chaxun" text="Pesticide Registration <br>and Inquiry" />
                        <Icons1 iconName="huansuan" text="Pesticide Concentration <br>Conversion (PPM)" />

                        <Icons1 iconName="duishui" text="Pesticide Dilution <br>Calculation" />
                        <!-- 知识图谱 -->
                        <el-col :span="6">
                            <el-dropdown>
                                <svg class="icon-tupu" aria-hidden="true">
                                    <use xlink:href="#icon-tupu"></use>
                                </svg>
                                <el-icon class="el-icon--right">
                                    <arrow-down />
                                </el-icon>
                                <template #dropdown>
                                    <el-dropdown-menu v-for="action in actions" :key="action"
                                        @click="handleActionClick(action)">
                                        <el-dropdown-item>{{ action }}</el-dropdown-item>
                                    </el-dropdown-menu>
                                </template>
                            </el-dropdown>
                            <div class="text">Disease and Pest <br>Knowledge Graph</div>
                        </el-col>
                        <!-- 知识图谱 -->
                        <!-- ai专家 -->
                        <el-col :span="6">
                            <router-link to="/expert">
                                <div>
                                    <svg class="icon-zhuanjia" aria-hidden="true">
                                        <use xlink:href="#icon-zhuanjia"></use>
                                    </svg>
                                </div>
                            </router-link>
                            <div class="text">Intelligent Expert <br>Systems</div>
                        </el-col>
                        <!-- ai专家 -->

                    </el-row>
                </div>
            </div>
        </template>

        <template v-slot:content2>
            <div class="content2">
                <MsgLevel1 title="Technology hotspots" more="More articles" />
                <MsgLevel2 />
            </div>
        </template>

    </Layout>
</template>

<script>
import Layout from '../components/Layout.vue'
import Icons1 from '../components/Icons1.vue'
import MsgLevel1 from '../components/MsgLevel1.vue'
import MsgLevel2 from '../components/MsgLevel2.vue'
import { storeToRefs } from 'pinia'
import { useIdentifyStore } from '../store/identify';
import { ref, watch } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import { ArrowDown } from '@element-plus/icons-vue'

export default {
    components: {
        Layout,
        Icons1,
        MsgLevel1,
        MsgLevel2
    },
    setup() {
        const actions = ref(["Two-spotted spider mite", "Bacterial spot of tomato", "Early blight of tomato", "Late blight of tomato", "Tomato leaf mold", "Tomato bacterial speck", "Tomato target spot", "Tomato mosaic virus", "Tomato yellow leaf curl disease"]);
        // 拍照上传预览的逻辑处理
        const store = useIdentifyStore();
        const { imageUrl, uploadResult } = storeToRefs(store);

        const openFile = () => {
            const fileInput = document.getElementById('fileInput');
            // 点击图标时触发文件选择器
            fileInput.click();
        }
        const handleFileInputChange = (event) => {
            const file = event.target.files[0];
            const reader = new FileReader();
            reader.onload = () => {
                imageUrl.value = reader.result;
                console.log('图片地址:', imageUrl.value);
                // 将 imageUrl 发送到服务器或进行其他操作
            };
            reader.readAsDataURL(file);

            // 上传操作：创建 FormData 对象, 并将图片文件添加到以后端预期的字段名 “file” 中
            const formData = new FormData();
            formData.append('url', file);
            /* 发送 POST 请求，将图片文件上传到 Flask 后端
            fetch('http://10.0.8.14:5002', {
                method: 'POST',
                body: formData,
            })*/
            axios.post('http://8.130.28.121:8008/bch/', formData)
                .then((response) => {
                    if (response.status === 200) {
                        // 获取后端返回的预测结果
                        return response.data;
                    } else {
                        // 如果请求未成功，则打印错误信息
                        console.error('上传图片失败');
                        alert('上传图片失败');
                        uploadResult.value = null;
                    }
                })
                .then((result) => {
                    uploadResult.value = result;
                    console.log('后端响应:', result);
                })
                .catch((error) => {
                    console.error('上传图片时出现错误:', error);
                });
        };
        // 拍照上传路由守卫
        const router = useRouter();
        router.beforeEach((to, from, next) => {
            if (to.name === 'ResultShow' && !imageUrl.value) {
                console.log('imageUrl 为空，阻止路由离开');
                next(from); // 阻止路由离开
            } else {
                console.log('允许路由离开');
                next(); // 允许路由离开
            }
        });

        watch(imageUrl, (newValue, oldValue) => {
            if (newValue !== '') {
                console.log('自动跳转到下一个路由');
                // 执行跳转
                router.push('/resultShow');
            }
        });

        // 请求知识图谱
        /*const handleActionClick = async (action) => {
            console.log('handleActionClick：', action);
            router.push('/tupu');
            /*try {
                // 发送action请求到后端
                const response = await axios.post('http://localhost:5002', { name });
                console.log('responseData：', response.data);
                const data = response.data;
                // 跳转到图谱页面
                router.push('/tupu');
            } catch {
                console.log('请求知识图谱错误', error)
            }
        };*/

        // 知识图谱假数据
        const handleActionClick = (action) => {
            console.log('handleActionClick:', action);
            let diseaseUrls = {
                "Two-spotted spider mite": { nodeJsonUrl: 'data/tupu/sbym_node.json', linksJsonUrl: 'data/tupu/sbym_links.json' },
                "Bacterial spot of tomato": { nodeJsonUrl: 'data/tupu/xjxbd_node.json', linksJsonUrl: 'data/tupu/xjxbd_links.json' },
                "Early blight of tomato": { nodeJsonUrl: 'data/tupu/zyb_node.json', linksJsonUrl: 'data/tupu/zyb_links.json' },
                "Late blight of tomato": { nodeJsonUrl: 'data/tupu/wyb_node.json', linksJsonUrl: 'data/tupu/wyb_links.json' },
                "Tomato leaf mold": { nodeJsonUrl: 'data/tupu/ymb_node.json', linksJsonUrl: 'data/tupu/ymb_links.json' },
                "Tomato bacterial speck": { nodeJsonUrl: 'data/tupu/qxpb_node.json', linksJsonUrl: 'data/tupu/qxpb_links.json' },
                "Tomato target spot": { nodeJsonUrl: 'data/tupu/bbb_node.json', linksJsonUrl: 'data/tupu/bbb_links.json' },
                "Tomato mosaic virus": { nodeJsonUrl: 'data/tupu/hyb_node.json', linksJsonUrl: 'data/tupu/hyb_links.json' },
                "Tomato yellow leaf curl disease": { nodeJsonUrl: 'data/tupu/hhqy_node.json', linksJsonUrl: 'data/tupu/hhqy_links.json' },
            };

            let nodeJsonUrl = diseaseUrls[action]?.nodeJsonUrl || '';
            let linksJsonUrl = diseaseUrls[action]?.linksJsonUrl || '';

            router.push({
                path: '/tupu',
                query: { nodeJsonUrl, linksJsonUrl }
            });
        };


        return {
            imageUrl,
            uploadResult,
            openFile,
            handleFileInputChange,
            actions,
            handleActionClick
        };
    },
}
</script>

<style lang="less" scoped>
// 单独为病虫害加css
.el-col {
    border-radius: 4px;
    padding: 15px;
}

.example-showcase .el-dropdown-link {
    cursor: pointer;
    color: var(--el-color-primary);
    display: flex;
    align-items: center;
}

.text {
    margin-top: 10px;
    text-align: center;
    font-size: small;
}

.icon-bingchonghai,
.icon-tupu,
.icon-zhuanjia {
    width: 25px;
    height: 25px;
}

// 该页面正常css 
.navbar {
    margin: 10px;
}

.navbarImage {
    width: 100%;
    height: 130px;
}

.content1 {
    display: flex;
    flex-direction: row;
    justify-content: center;
    flex-wrap: wrap;

    margin: 20px;
    border-radius: 8px;
    background: #ffffff;
}

.content2 {
    display: flex;
    flex-direction: column;
    margin: 20px;
    border-radius: 8px;
    background: #ffffff;


}

.icons1 {
    margin-left: 10px;
    margin-right: 10px;
}

.el-row {
    margin-bottom: 20px;
    display: flex;
    text-align: center;
    margin: 10px;
}

.el-row:last-child {
    margin-bottom: 10px;
}
</style>