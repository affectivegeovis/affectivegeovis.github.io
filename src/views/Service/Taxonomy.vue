<template>
    <div class="taxonomy-page">
        <div class="intro-container">
            <p> We have collected and analyzed a corpus containing 62 affective geovisualization design practices.
                On the left is a table showing a design taxonomy, 
                where each row represents a design work from our corpus, 
                and each column corresponds to a coded dimension adapted from the PPP model. 
                By combining quantitative clustering with manual interpretation, we identified four distinctive paradigms.
                On the right is a 3D scatterplot displaying the Correspondence Analysis Biplot of the four identified paradigms. 
                You can select a project from the table or zoom into the scatterplot to explore its position in the taxonomy and view detailed image.</p>
        </div>
        <div class="taxonomy-container">
            <!-- 左侧大表格 -->
            <div class="table-section">
                <barbrush 
                    ref="barbrushRef" 
                    :selected-project-name="selectedProjectName"
                    @project-selected="handleProjectSelected"/>
            </div>
            
            <!-- 右侧布局 -->
            <div class="right-section">
                <!-- 右上散点图 -->
                <div class="scatter-section">
                    <scatter-plot 
                        ref="scatterRef" 
                        :scatter-data="scatterData"
                        :selected-point-name="selectedProjectName"
                        @point-clicked="handlePointClick"/>
                </div>
                
                <!-- 右下项目详情 -->
                <div class="detail-section">
                    <div v-if="selectedProject" class="project-details">
                        
                        <div class="detail-content">
                            <div class="detail-left">
                                <img :src="require('@/assets/images/each/'+selectedProject.images )"  alt="img"   style="aspect-ratio: 16 / 9;width: 100%;"> 
                                </div>
                            <div class="detail-right">
                                <p><span class="custom-tag1" :style="{'color':'#B78296','border':`1px solid #B78296`}" v-for="(item1, index) in selectedProject.person" :key="index">{{ item1 }}</span></p>
                                <p><span class="custom-tag1" :style="{'color':'#4E98C0','border':`1px solid #4E98C0`}" v-for="(item1, index) in selectedProject.place" :key="index">{{ item1 }}</span></p>
                                <p><span class="custom-tag1" :style="{'color':'#AF967E','border':`1px solid #AF967E`}" v-for="(item1, index) in selectedProject.Process1" :key="index">{{ item1 }}</span></p>
                                <p><span class="custom-tag1" :style="{'color':'#58AE98','border':`1px solid #58AE98`}" v-for="(item1, index) in selectedProject.Process2" :key="index">{{ item1 }}</span></p>
                                <p><span class="custom-tag1" :style="{'color':'grey','border':`1px solid grey`}" v-for="(item1, index) in selectedProject.Genre" :key="index">{{ item1 }}</span></p>
                            </div>
                        </div>
                        <div class="left-block">
                            <span style="text-align: left;font-size: 1.2rem;font-weight: bolder;color:black">{{ selectedProject.name  }}</span>
                            <a v-if="selectedProject.Link1" v-for="(selectedProject1, index1) in selectedProject.Link1" :key="index1" :href="selectedProject.Link1[index1]" target="_blank" style="display: inline-block; text-decoration: none;margin: 0; padding: 0;"> 
                                <i class="fas fa-book" style="color:black;padding: 0; margin-left: 5px;font-size: small"></i>
                            </a>
                            <a v-if="selectedProject.Link2" v-for="(selectedProject1, index1) in selectedProject.Link2" :key="index1" :href="selectedProject.Link2[index1]" target="_blank" style="display: inline-block; text-decoration: none;margin: 0; padding: 0;"> 
                                <i class="fas fa-book" style="color:#746f6e;padding: 0; margin-left: 5px;font-size: small"></i>
                            </a>
                            <a v-if="selectedProject.Link3" v-for="(selectedProject1, index1) in selectedProject.Link3" :key="index1" :href="selectedProject.Link3[index1]" target="_blank" style="display: inline-block; text-decoration: none;margin: 0; padding: 0;"> 
                                <i class="fas fa-desktop" style="color:black;padding: 0; margin-left: 5px;font-size: small"></i>
                            </a>
                            <a v-if="selectedProject.Link4" v-for="(selectedProject1, index1) in selectedProject.Link4" :key="index1" :href="selectedProject.Link4[index1]" target="_blank" style="display: inline-block; text-decoration: none;margin: 0; padding: 0;"> 
                                <i class="fa-regular fa-comment-dots" style="color:black;padding: 0; margin-left: 5px;font-size: small"></i>
                            </a>
                       </div>
                       <p style="line-height:1.2 ;font-size: 1rem;text-align: left;">{{ selectedProject.author ? selectedProject.author  : '' }} </p>
                    </div>
                    <div v-else class="no-selection">
                        Please select a project
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
 
<script>
    import "@/assets/css/style.css";
    import axios from 'axios';
    import barbrush from '@/views/graphs/barbrush.vue';
    import ScatterPlot from '@/views/graphs/ScatterPlot.vue';

    export default {
        components: {
            barbrush,
            ScatterPlot
        },
        data() {
            return {
                selectedProjectName: '',    // 当前选中项目的名称
                selectedProject: null,     // 当前选中项目的完整信息
                projectData: [],          // 所有项目数据
                scatterData: [],           // 散点图数据
                selectedPoint: null        // 当前选中的点
            }
        },
        watch: {
            selectedPoint(newPointName) {
                if (newPointName) {
                    this.updateSelectedProject(newPointName);
                }
            }
        },
        methods: {
            // 统一的项目选择处理函数
            handleProjectSelected(projectName) {
                // 找到最匹配的项目
                const matchedProject = this.findMatchingProject(projectName);
                if (matchedProject) {
                    this.selectedProjectName = matchedProject.name;
                    this.selectedProject = matchedProject;
                }
            },

            // 根据点名称更新选中的项目
            updateSelectedProject(pointName) {
                const matchedProject = this.findMatchingProject(pointName);
                if (matchedProject) {
                    this.selectedProject = matchedProject;
                    this.selectedProjectName = matchedProject.name;
                    console.log(this.selectedProject);
                }
            },

            // 查找匹配的项目
            findMatchingProject(name) {
                if (!name || !this.projectData.length) return null;

                // 1. 首先尝试完全匹配
                let matchedProject = this.projectData.find(p => p.name === name);
                if (matchedProject) return matchedProject;

                // 2. 尝试部分匹配（barbrush中的名称是Service中名称的一部分）
                matchedProject = this.projectData.find(p => p.name.includes(name));
                if (matchedProject) return matchedProject;

                // 3. 尝试部分匹配（Service中的名称是barbrush中名称的一部分）
                matchedProject = this.projectData.find(p => name.includes(p.name));
                if (matchedProject) return matchedProject;

                return null;
            },

            handlePointClick(pointName) {
                this.selectedPoint = pointName;
            },

            // 加载项目数据
            async loadProjectData() {
                try {
                    const response = await axios.get('/output.json');
                    this.projectData = response.data;
                    console.log('Loaded project data:', this.projectData);
                } catch (error) {
                    console.error('Failed to load project data:', error);
                }
            }
        },
        mounted() {
            this.loadProjectData();
        }
    }
</script>
 
<style scoped>
    .left-block{
        text-align: left;
    }
    .custom-tag1 {
            text-align: right;
            background-color: transparent; /* 背景颜色 */
            border-radius: 6px; /* 圆角 */
            color: black; /* 字体颜色 */
            padding: 3px 10px; /* 内边距，可根据需要调整 */
            font-size: 0.9rem;
            margin:3px;
            white-space: normal; /* 默认的 white-space 值，允许内容换行 */
            word-wrap: break-word; /* 当内容太长时，允许单词内换行 */
            display: inline-block; /* 使元素以内联块元素显示 */
            }
    .taxonomy-page {
        padding-top: 50px;
        min-height: 100vh;
        background-color: #f5f5f5;
    }
    .intro-container {
        text-align: justify; /* 居中对齐文字 */
        padding: 20px; /* 四周内边距 */
        margin: 20px; /* 四周外边距 */
        margin-bottom: 0; /* 去掉下方外边距 */
        background-color: white; /* 背景颜色 */
        border-radius: 8px; /* 圆角 */
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* 阴影效果 */
        font-size: 1rem; /* 字体大小 */
        color: #333; /* 字体颜色 */
    }

    .taxonomy-container {
        display: flex;
        width: 100%;
        height: calc(100vh - 140px);
        padding: 20px;
        gap: 10px;
        margin-top: 0; /* 去掉上方外边距 */
    }

    .table-section {
        flex: 3;
        border: 1px solid #eee;
        border-radius: 8px;
        padding: 15px;
        background-color: white;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }

    .right-section {
        flex: 2;
        display: flex;
        flex-direction: column;
        gap: 10px;
    }

    .scatter-section {
        flex: 1;
        border: 1px solid #eee;
        border-radius: 8px;
        padding: 15px;
        background-color: white;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        min-height: 45%;
        /* min-height: 300px; */
    }

    .detail-section {
        flex: 1;
        border: 1px solid #eee;
        border-radius: 8px;
        padding: 15px;
        background-color: white;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        min-height: 45%;
        /* min-height: 200px; */
        overflow-y: auto;
    }

    .project-details {
        height: 100%;
    }

    .detail-content {
        display: flex;
        gap: 5px;
        /* height: 100%; */
    }

    .detail-left {
        flex: 2;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .project-image {
        max-width: 100%;
        max-height: 100%;
        object-fit: contain;
        border-radius: 4px;
    }

    .detail-right {
        flex: 3;
        display: flex;
        flex-direction: column;
        justify-content: left;
        text-align: left;
    }

    .detail-item {
        display: flex;
        margin-bottom: 15px;
        line-height: 1.5;
    }

    .detail-label {
        width: 80px;
        color: #666;
        font-weight: 500;
    }

    .detail-value {
        flex: 1;
        color: #333;
    }

    .detail-link {
        color: #409EFF;
        text-decoration: none;
        word-break: break-all;
    }

    .detail-link:hover {
        text-decoration: underline;
    }

    .no-selection {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100%;
        color: #999;
    }

    .no-selection i {
        font-size: 48px;
        margin-bottom: 10px;
    }

    canvas {
        position: relative !important;
        z-index: 1;
    }
</style>
 