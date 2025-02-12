<template>
    <el-collapse-item name="1" disabled>
        <template slot="title">
            <ConfigTitle icon="el-icon-magic-stick" name="工具"></ConfigTitle>
        </template>
        <div class="tool-wrapper">
            <el-tooltip v-for="tool in tools" :key="tool.name" :content="tool.tip" :open-delay="500">
                <div class="tool-item" :class="{ active: currentToolModel === tool }" @click="handleToolClick(tool)">
                    <i :class="tool.icon"></i>
                </div>
            </el-tooltip>
        </div>
    </el-collapse-item>
</template>

<script>
import ConfigTitle from './ConfigTitle.vue';

export default {
    name: 'ToolsCollapse',
    props: ['tools', 'currentTool'],
    components: { ConfigTitle },
    computed: {
        currentToolModel: {
            get() {
                return this.currentTool
            },
            set(val) {
                this.$emit('update:currentTool', val)
            }
        }
    },
    created() {
        this.currentToolModel = this.tools[2]
    },  
    methods: {
        handleToolClick(tool) {
            this.currentToolModel = tool
        },
    }
}
</script>

<style scoped>
.tool-wrapper {
    display: flex;
    padding: 0 18px;
    margin-bottom: 18px;
}

.tool-wrapper .tool-item {
    width: 40px;
    height: 38px;
    border-radius: 4px;
    margin-right: 8px;
    border: 1px solid #E4E7ED;
    color: #303133;
    cursor: pointer;
    user-select: none;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 16px;
}

.tool-wrapper .tool-item:hover {
    background: #f3f3f3;
}

.tool-wrapper .tool-item.active {
    background: #409EFF;
    color: #fff;
}

.box-info-wrapper {
    max-height: calc(100vh - 160px);
    overflow: auto;
}

.box-info-wrapper .box-info-item {
    padding: 18px;
    padding-top: 12px;
    position: relative;
    border-top: 1px solid #EBEEF5;
}

.box-info-wrapper .box-info-item.active::before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 2px;
    background: #409EFF;
    z-index: 99;
}

.box-info-wrapper .box-info-item .box-info-title {
    color: #606266;
    margin-bottom: 4px;
}

.box-info-wrapper .box-info-item .box-info-form {
    display: flex;
    gap: 18px;
}

.form-item {
    display: flex;
    gap: 8px;
    color: #606266;
    margin-bottom: 6px;

    input {
        border: none;
        border-bottom: 1px solid #EBEEF5;
        outline: none;
        width: 60px;
        color: #606266;
        font-size: 12px;
    }
}
</style>