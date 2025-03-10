<template>
    <el-dialog
        title="提示"
        :visible.sync="dialogVisible"
        width="40%"
        top="4vh"
    >
        <el-image
            style="width: 100%"
            :src="img"
            fit="contain"
            :preview-src-list="srcList"
            >
        </el-image>
    </el-dialog>
</template>

<script>
import img from '../assets/tip.png'
export default {
    data() {
        return {
            dialogVisible: false,
            img,
            srcList: [img]
        };
    },
    methods: {
        open() {
            this.dialogVisible = true
        }
    }
};
</script>

<style scoped>
</style>