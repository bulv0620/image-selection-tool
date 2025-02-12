<template>
    <el-row class="container">
        <el-col class="graph-wrapper" :span="20">
            <WorkSpace :current-tool="currentTool" :current-box.sync="currentBox" :boxes="boxes"></WorkSpace>
        </el-col>
        <el-col class="config-wrapper" :span="4">
            <el-collapse v-model="activeConfigMenu">
                <Tools :tools="tools" :current-tool.sync="currentTool"></Tools>
                <Boxes :boxes="boxes" :current-box.sync="currentBox"></Boxes>
            </el-collapse>
        </el-col>
    </el-row>
</template>

<script>
import Tools from './components/Tools.vue';
import Boxes from './components/Boxes.vue';
import WorkSpace from './components/WorkSpace.vue';

export default {
    name: 'ImageSelectionTool',
    components: { Tools, Boxes, WorkSpace },
    data() {
        return {
            activeConfigMenu: ['1', '2'],

            tools: [
                {
                    name: 'drag',
                    tip: '拖拽工具',
                    icon: 'el-icon-rank'
                },
                {
                    name: 'magnifier',
                    tip: '放大镜',
                    icon: 'el-icon-search'
                },
                {
                    name: 'select',
                    tip: '画框工具',
                    icon: 'el-icon-crop'
                },
                {
                    name: 'edit',
                    tip: '编辑工具',
                    icon: 'el-icon-edit'
                },
            ],
            currentTool: null,
            prevTool: null,

            boxes: [
            ],
            currentBox: null,
        }
    },
    watch: {
        currentTool(val) {
            if(val.name !== 'edit') {
                this.currentBox = null
            }
        },
        currentBox(val) {
            if(val) {
                this.currentTool = this.tools.find(el => el.name === 'edit')
            }
        }
    },
    mounted() {
        document.addEventListener('keydown', (e) => {
            if(e.code === 'Space') {
                this.handleSpaceDown()
            }
            if(e.code === 'ControlLeft') {
                this.handleCtrlDown()
            }
        })
        document.addEventListener('keyup', (e) => {
            if(e.code === 'Space') {
                this.handleSpaceUp()
            }
            if(e.code === 'ControlLeft') {
                this.handleCtrlUp()
            }
        })
        // document.addEventListener('keypress', (e) => {
        //     console.log(e.code);
        // })
    },

    methods: {
        handleMouseMove(e) {
            console.log(e);
        },

        handleSpaceDown() {
            if(this.prevTool) return 
            this.prevTool = this.currentTool
            this.currentTool = this.tools[0]
        },
        handleSpaceUp() {
            if(!this.prevTool) return
            this.currentTool = this.prevTool
            this.prevTool = null
        },

        handleCtrlDown() {
            if(this.prevTool) return 
            this.prevTool = this.currentTool
            this.currentTool = this.tools[1]
        },

        handleCtrlUp() {
            if(!this.prevTool) return
            this.currentTool = this.prevTool
            this.prevTool = null
        }
    }
}
</script>

<style scoped>
:deep(.el-collapse-item__content) {
  padding-bottom: 0;
}

.container {
    width: 100%;
    height: 100%;
}

.container .graph-wrapper {
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.img {
    max-width: 100%;
    max-height: 100%;
}

.container .config-wrapper {
    height: 100%;
    border-left: 1px solid #DCDFE6;
}
</style>