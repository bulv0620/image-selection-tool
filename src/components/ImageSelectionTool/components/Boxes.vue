<template>
    <el-collapse-item class="boxes" title="画框列表" name="2">
        <template slot="title">
            <ConfigTitle icon="el-icon-tickets" name="画框列表"></ConfigTitle>
        </template>
        <el-empty v-if="boxes.length === 0" description="暂无画框" :image-size="50"></el-empty>
        <div v-else class="box-info-wrapper" :style="{ maxHeight: boxInfoWrapperMaxHeight }">
            <div 
                class="box-info-item"
                v-for="box in boxes" 
                :key="box.id" 
                :ref="`box-info-item-${box.id}`"
                :class="{ 
                    active: currentBox === box ,
                }"
                @click="handleBoxClick(box)"
                @mouseenter="handleMouseenter(box)"
                @mouseleave="handleMouseleave(box)"
            >
                <p class="box-info-title">
                    <NameEditor v-model="box.name" :boxes="boxes"></NameEditor>
                    <i class="el-icon-delete delete-icon" @click.stop="handleDelete(box)"></i>
                </p>
                <div class="box-info-form">
                    <div class="box-info-form__left">
                        <BoxFormItem v-model="box.width" label="宽"></BoxFormItem>
                        <BoxFormItem v-model="box.height" label="高"></BoxFormItem>
                    </div>
                    <div class="box-info-form__right">
                        <BoxFormItem v-model="box.x" label="X"></BoxFormItem>
                        <BoxFormItem v-model="box.y" label="Y"></BoxFormItem>
                    </div>
                </div>
            </div>
        </div>
    </el-collapse-item>
</template>

<script>
import ConfigTitle from './ConfigTitle.vue';
import BoxFormItem from './BoxFormItem.vue';
import NameEditor from './NameEditor.vue';

export default {
    name: 'BoxesCollapse',
    components: { ConfigTitle, BoxFormItem, NameEditor },
    props: ['boxes', 'currentBox'],
    data() {
        return {
            boxInfoWrapperMaxHeight: '260px' 
        }
    },
    computed: {
        currentBoxModel: {
            get() {
                return this.currentBox
            },
            set(val) {
                this.$emit('update:currentBox', val)
            }
        }
    },
    watch: {
        currentBox(val) {
            if(val) {
                this.$refs[`box-info-item-${val.id}`][0].scrollIntoViewIfNeeded()
            }
        }
    },
    mounted() {
        this.getBoxInfoWrapperMaxHeight()

        window.addEventListener('resize', this.getBoxInfoWrapperMaxHeight)
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.getBoxInfoWrapperMaxHeight)
    },
    methods: {
        handleBoxClick(box) {
            this.currentBoxModel = box
        },

        getBoxInfoWrapperMaxHeight() {
            const containerHeight = document.querySelector('.config-wrapper').clientHeight
            const otherHeight = Array.from(document.querySelectorAll('.config-wrapper .el-collapse-item')).filter(el => !el.className.includes('boxes')).reduce((acc, cur) => acc + cur.clientHeight, 0)
            const headerHeight = document.querySelector('.config-wrapper .el-collapse-item .el-collapse-item__header').clientHeight

            this.boxInfoWrapperMaxHeight = `${ containerHeight - otherHeight - headerHeight - 4 }px`
        },

        handleDelete(box) {
            const index = this.boxes.findIndex(el => el === box)

            if(box === this.currentBoxModel) {
                if(this.boxes.length === 1) {
                    this.currentBoxModel = null
                }
                else if(index === this.boxes.length - 1) {
                    this.currentBoxModel = this.boxes[index - 1]
                }
                else {
                    this.currentBoxModel = this.boxes[index + 1]
                }
            }

            // eslint-disable-next-line vue/no-mutating-props
            this.boxes.splice(index, 1)
        },

        handleMouseenter(box) {
            this.boxes.forEach(el => {
                el.flash = false
            });
            box.flash = true
        },

        
        handleMouseleave(box) {
            box.flash = false
        }
    }
}
</script>

<style scoped>
.box-info-wrapper {
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
    width: 4px;
    background: #409EFF;
    z-index: 99;
}

.box-info-wrapper .box-info-item .box-info-title {
    color: #606266;
    margin-bottom: 4px;
    display: flex;
    justify-content: space-between;
}

.box-info-wrapper .box-info-item .box-info-title .delete-icon {
    color: #F56C6C; 
    cursor: pointer;
}

.box-info-wrapper .box-info-item .box-info-form {
    display: flex;
    gap: 18px;
}

/* WebKit 浏览器的自定义滚动条 */
::-webkit-scrollbar {
  width: 6px;               /* 纵向滚动条宽度 */
  height: 6px;              /* 横向滚动条高度 */
}

::-webkit-scrollbar-track {
  background: #f1f1f1;      /* 滚动条轨道背景色 */
}

::-webkit-scrollbar-thumb {
  background: #ddd;         /* 滚动滑块颜色 */
  border-radius: 4px;
  box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
}

::-webkit-scrollbar-thumb:hover {
  background: #ccc;         /* 悬停时颜色变深 */
}
</style>