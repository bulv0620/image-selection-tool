<template>
    <div class="main">
        <div class="container image-work-space__container">
            <div 
                v-if="imageSrc"
                class="image-container"
                @mousedown="startDrag"
                @mousemove="handleDrag"
                @mouseup="endDrag"
                @mouseleave="endDrag"
                @wheel.prevent="handleZoom"
                ref="container"
                :style="{
                    cursor: currentCursor
                }"
            >
                <img 
                    class="dynamic-img"
                    :src="imageSrc" 
                    :style="imageStyle"
                    ref="image"
                    @load="handleImageLoad"
                >
            
                <div 
                    class="selection-box"
                    :style="selectionStyle"
                    v-show="isSelecting"
                ></div>

                <div 
                    v-for="box in boxes"
                    :key="box.id"
                    class="seleced-box"
                    :class="{ 
                        active: currentBox === box,
                        valid: currentTool.name === 'edit',
                        flash: box.flash && currentBox !== box
                    }"
                    :style="boxesStyle(box)"
                    @click="handleSelectedBoxClick(box)"
                    @mousedown="startDragBox($event, box)"
                    @mouseup="endDragBox($event, box)"
                >   
                    <!-- <div v-show="currentTool.name === 'edit'" class="title">{{ `选框 ${box.id}` }}</div> -->
                    <div v-show="currentTool.name === 'edit' && currentBox === box" class="delete" @click="handleDelete(box)">
                        <i class="el-icon-close delete-icon"></i>
                    </div>
                    <div v-show="currentTool.name === 'edit' && currentBox === box" class="anchor lt"></div>
                    <div v-show="currentTool.name === 'edit' && currentBox === box" class="anchor rt"></div>
                    <div v-show="currentTool.name === 'edit' && currentBox === box" class="anchor lb"></div>
                    <div v-show="currentTool.name === 'edit' && currentBox === box" class="anchor rb"></div>
                </div>
            </div>
        </div>
        <div class="bar">
            <span class="text tip" @click="openTipDialog"><i class="el-icon-question"></i></span>
            <span class="text">W: {{ imgWidth }}</span>
            <span class="text">H: {{ imgHeight }}</span>
            <span class="text"></span>
            <span class="text">X: {{ x }}</span>
            <span class="text">Y: {{ y }}</span>
            <input ref="fileSelector" type="file" @change="handleFileSelect" accept="image/*" style="margin-left: auto;">
        </div>
        <TipDialog ref="tipDialog"></TipDialog>
    </div>
</template>

<script>
import TipDialog from './TipDialog.vue';

export default {
    name: 'WorkSpace',
    components: { TipDialog },
    props: ['currentTool', 'currentBox', 'boxes'],
    data() {
        return {
            // 最小允许尺寸
            BOX_MIN_WIDTH: 10,
            BOX_MIN_HEIGHT: 10,
            // 图片
            imageSrc: null,
            // 缩放比例
            scale: 1,
            // 图片在容器中的坐标
            translateX: 0, 
            translateY: 0,
            // 图片拖拽
            isDragging: false,
            startDragX: 0, // 起始光标坐标
            startDragY: 0,
            startTranslateX: 0, // 起始图片坐标
            startTranslateY: 0,
            // 绘制选框
            isSelecting: false,
            startX: 0, // 绘制起始光标坐标
            startY: 0,
            currentX: 0, // 光标当前位置坐标
            currentY: 0,
            // 光标对应图片的坐标
            x: 0,
            y: 0,
            // 图片像素宽高
            imgWidth: 0, 
            imgHeight: 0,
            // 选框编辑相关
            draggingBox: null,
            dragType: null, // 'move'/'resize'
            resizeDirection: null,
            startDragBoxX: 0,
            startDragBoxY: 0,
            initialBoxState: {},
        }
    },
    computed: {
        imageStyle() {
            return {
                transform: `translate(${this.translateX}px, ${this.translateY}px) scale(${this.scale})`,
                transformOrigin: '0 0',
                pointerEvents: 'none'
                
            };
        },
        selectionStyle() {
            return {
                left: Math.min(this.startX, this.currentX) + 'px',
                top: Math.min(this.startY, this.currentY) + 'px',
                width: Math.abs(this.currentX - this.startX) + 'px',
                height: Math.abs(this.currentY - this.startY) + 'px'
            };
        },
        currentCursor() {
            switch (this.currentTool?.name) {
                case 'drag':
                    return 'grab'
                case 'select':
                    return 'crosshair'
                case 'magnifier':
                    return 'zoom-in'
                default:
                    return 'default'
            }
        },
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
        currentTool() {
            // 切换工具就取消当前的选框拖拽
            this.draggingBox = null
            this.dragType = null
            this.resizeDirection = null
            this.startDragBoxX = 0
            this.startDragBoxY = 0
            this.initialBoxState = {}
        }
    },
    methods: {
        openTipDialog() {
            this.$refs.tipDialog.open()
        },
        handleSelectedBoxClick(box) {
            this.currentBoxModel = box
        },
        boxesStyle(box) {
            const data = this.convertToContainerCoordinates(box.x, box.y, this.translateX, this.translateY, this.scale)
            
            let cursor = 'default'
            if(this.currentTool.name === 'edit') {
                if(this.currentBox === box) {
                    cursor = 'move'
                }
                else {
                    cursor = 'pointer'
                }
            }
            
            return {
                left: data.x + 'px',
                top: data.y + 'px',
                width: box.width * this.scale + 'px',
                height: box.height * this.scale + 'px',
                cursor:  cursor,
                pointerEvents: this.currentTool.name === 'edit' ? '' : 'none',
            }
        },
        handleFileSelect(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.imageSrc = e.target.result;
                    this.$refs.fileSelector.blur()

                    // eslint-disable-next-line vue/no-mutating-props
                    this.boxes.splice(0)
                };
                reader.readAsDataURL(file);
            }
        },

        handleImageLoad() {
            // 获取容器和图片的尺寸
            const container = this.$refs.container.getBoundingClientRect();
            const img = {
                height: this.$refs.image.clientHeight,
                width: this.$refs.image.clientWidth
            };

            // 计算容器宽度与图片宽度的比例
            const scaleX = container.width / img.width;
            // 计算容器高度与图片高度的比例
            const scaleY = container.height / img.height;
            // 选择较小的比例作为缩放比例
            const scale = Math.min(scaleX, scaleY);

            // 计算图片在容器中居中的偏移量
            const translateX = (container.width - img.width * scale) / 2;
            const translateY = (container.height - img.height * scale) / 2;

            // 更新组件的状态
            this.scale = scale;
            this.translateX = translateX;
            this.translateY = translateY;

            this.imgWidth = img.width
            this.imgHeight = img.height
        },

        startDrag(e) {
            if(e.button !== 0) return 

            if (this.currentTool.name === 'select') {
                // 按住Shift开始绘制选框
                const rect = this.$refs.container.getBoundingClientRect();
                this.startX = e.clientX - rect.left;
                this.startY = e.clientY - rect.top;
                this.currentX = this.startX;
                this.currentY = this.startY;
                this.isSelecting = true;
            } else if(this.currentTool.name === 'drag') {
                // 正常拖拽图片
                this.isDragging = true;
                this.startDragX = e.clientX;
                this.startDragY = e.clientY;
                this.startTranslateX = this.translateX;
                this.startTranslateY = this.translateY;
            }
        },

        handleDrag(e) {
            if(this.draggingBox) {
                this.handleDragBox(e, this.draggingBox)
                return
            }
            if (this.currentTool.name === 'select' && this.isSelecting) {
                // 绘制选框
                const rect = this.$refs.container.getBoundingClientRect();
                this.currentX = e.clientX - rect.left;
                this.currentY = e.clientY - rect.top;
            } else if(this.currentTool.name === 'drag' && this.isDragging) {
                // 拖拽图片
                const dx = e.clientX - this.startDragX;
                const dy = e.clientY - this.startDragY;
                this.translateX = this.startTranslateX + dx;
                this.translateY = this.startTranslateY + dy;
            }

            if(this.imageSrc) {
                const data = this.convertToImageCoordinates(e.clientX, e.clientY, this.translateX, this.translateY, this.imgWidth, this.imgHeight, this.scale)
                this.x = data.x.toFixed(1)
                this.y = data.y.toFixed(1)
            }
            else {
                this.x = 0
                this.y = 0
            }
        },

        endDrag() {
            if(this.isSelecting) {
                const rectInfo = {
                    x: Math.min(this.startX, this.currentX),
                    y: Math.min(this.startY, this.currentY),
                    width: Math.abs(this.currentX - this.startX),
                    height: Math.abs(this.currentY - this.startY)
                }

                if(rectInfo.width > this.BOX_MIN_WIDTH && rectInfo.height > this.BOX_MIN_HEIGHT) {
                    const data = this.convertToImageCoordinates(rectInfo.x, rectInfo.y, this.translateX, this.translateY, this.imgWidth, this.imgHeight, this.scale)

                    const box = {
                        id: this.boxes.length === 0 ? 1 : Math.max(...this.boxes.map(el => el.id)) + 1,
                        x: data.x.toFixed(2),
                        y: data.y.toFixed(2),
                        width: (rectInfo.width / this.scale).toFixed(2),
                        height: (rectInfo.height / this.scale).toFixed(2),
                        name: this.generateDefaultName(this.boxes),
                        flash: false
                    }

                    // eslint-disable-next-line vue/no-mutating-props
                    this.boxes.push(box)
                }
            }

            this.isDragging = false;
            this.isSelecting = false;
        },

        generateDefaultName(existingFrames) {
            // 匹配所有默认名称的数字部分
            const maxNum = existingFrames.reduce((max, frame) => {
                const match = frame.name.match(/^选框(\d+)$/);
                return match ? Math.max(max, parseInt(match[1])) : max;
            }, 0);
            return `选框${maxNum + 1}`;
        },

        handleZoom(e) {
            if(this.currentTool.name !== 'magnifier') return 
            const delta = e.deltaY > 0 ? 0.9 : 1.1;
            const newScale = this.scale * delta;
            
            // 限制缩放范围
            if (newScale < 0.1 || newScale > 5) return;
            
            // 计算基于鼠标位置的缩放
            const rect = this.$refs.container.getBoundingClientRect();
            const offsetX = e.clientX - rect.left;
            const offsetY = e.clientY - rect.top;
            
            const imgX = (offsetX - this.translateX) / this.scale;
            const imgY = (offsetY - this.translateY) / this.scale;
            
            this.scale = newScale;
            this.translateX = offsetX - imgX * this.scale;
            this.translateY = offsetY - imgY * this.scale;
        },

        startDragBox(e, box) {
            e.stopPropagation()
            if(e.button !== 0) return 

            if(this.currentBox !== box) return 
            if (e.target.classList.contains('anchor')) {
                // 处理调整大小
                this.dragType = 'resize'
                this.resizeDirection = e.target.classList[1] // lt/rt/lb/rb
            } else {
                // 处理移动
                this.dragType = 'move'
            }

            this.draggingBox = box
            const rect = this.$refs.container.getBoundingClientRect()
            this.startDragBoxX = e.clientX - rect.left
            this.startDragBoxY = e.clientY - rect.top
            
            // 保存初始状态（容器坐标系）
            this.initialBoxState = {
                x: box.x,
                y: box.y,
                width: box.width,
                height: box.height,
                containerX: this.convertToContainerCoordinates(box.x, box.y, this.translateX, this.translateY, this.scale).x,
                containerY: this.convertToContainerCoordinates(box.x, box.y, this.translateX, this.translateY, this.scale).y,
                containerWidth: box.width * this.scale,
                containerHeight: box.height * this.scale
            }
        },

        handleDragBox(e, box) {
            e.stopPropagation()
            if (!this.draggingBox || box !== this.draggingBox) return

            const rect = this.$refs.container.getBoundingClientRect()
            const currentX = e.clientX - rect.left
            const currentY = e.clientY - rect.top
            const deltaX = currentX - this.startDragBoxX
            const deltaY = currentY - this.startDragBoxY

            if (this.dragType === 'move') {
                // 计算新的容器坐标
                const newContainerX = this.initialBoxState.containerX + deltaX
                const newContainerY = this.initialBoxState.containerY + deltaY
                
                // 转换为图片坐标系
                const imgCoords = this.convertToImageCoordinates(
                    newContainerX, 
                    newContainerY,
                    this.translateX,
                    this.translateY,
                    this.imgWidth,
                    this.imgHeight,
                    this.scale
                )
                
                this.draggingBox.x = Number(imgCoords.x.toFixed(2))
                this.draggingBox.y = Number(imgCoords.y.toFixed(2))
            } 
            else if (this.dragType === 'resize') {
                const direction = this.resizeDirection
                let newWidth = this.initialBoxState.containerWidth
                let newHeight = this.initialBoxState.containerHeight
                let newContainerX = this.initialBoxState.containerX
                let newContainerY = this.initialBoxState.containerY

                // 根据方向计算新的尺寸和位置
                switch (direction) {
                    case 'lt':
                        newWidth -= deltaX
                        newHeight -= deltaY
                        newContainerX += deltaX
                        newContainerY += deltaY
                        break
                    case 'rt':
                        newWidth += deltaX
                        newHeight -= deltaY
                        newContainerY += deltaY
                        break
                    case 'lb':
                        newWidth -= deltaX
                        newHeight += deltaY
                        newContainerX += deltaX
                        break
                    case 'rb':
                        newWidth += deltaX
                        newHeight += deltaY
                        break
                }

                // 限制最小尺寸
                if (newWidth < this.BOX_MIN_WIDTH) newWidth = 10
                if (newHeight < this.BOX_MIN_HEIGHT) newHeight = 10

                // 转换为图片坐标系
                const imgCoords = this.convertToImageCoordinates(
                    newContainerX, 
                    newContainerY,
                    this.translateX,
                    this.translateY,
                    this.imgWidth,
                    this.imgHeight,
                    this.scale
                )

                const imgWidth = newWidth / this.scale
                const imgHeight = newHeight / this.scale

                // 更新框体信息
                this.draggingBox.x = Number(imgCoords.x.toFixed(2))
                this.draggingBox.y = Number(imgCoords.y.toFixed(2))
                this.draggingBox.width = Number(imgWidth.toFixed(2))
                this.draggingBox.height = Number(imgHeight.toFixed(2))
            }
        },

        endDragBox(e) {
            e.stopPropagation()
            this.draggingBox = null
            this.dragType = null
            this.resizeDirection = null
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
        
        /**
         * 获取当前容器中x、y对应的图片像素x、y
         * @param mouseX 鼠标在容器中的x
         * @param mouseY 鼠标在容器中的y
         * @param imageX 图片的x
         * @param imageY 图片的y
         * @param imgWidth 图片的实际像素宽度
         * @param imgHeight 图片实际的像素高度
         * @param scale 图片的缩放比例
         */
         convertToImageCoordinates(mouseX, mouseY, imageX, imageY, imgWidth, imgHeight, scale) {
            // 计算图片的实际尺寸
            const actualWidth = imgWidth * scale;
            const actualHeight = imgHeight * scale;

            // 计算鼠标相对于图片左上角的坐标
            const relativeX = mouseX - imageX;
            const relativeY = mouseY - imageY;

            // 将相对坐标转换为图片实际像素坐标
            const pixelX = relativeX * (imgWidth / actualWidth);
            const pixelY = relativeY * (imgHeight / actualHeight);

            return { x: pixelX, y: pixelY };
        },

        /**
         * 获取对应的图片像素x、y在容器中的x、y
         * @param pixelX 像素位置x
         * @param pixelY 像素位置y
         * @param imageX 图片x
         * @param imageY 图片y
         * @param scale 图片缩放比例
         */
        convertToContainerCoordinates(pixelX, pixelY, imageX, imageY, scale) {
            // 将像素坐标转换为容器中的坐标
            const containerX = imageX + (pixelX * scale);
            const containerY = imageY + (pixelY * scale);

            return { x: containerX, y: containerY };
        },
    }
}
</script>

<style scoped>
.main {
    width: 100%;
    height: 100%;
    
}

.main .container {
    width: 100%;
    height: calc(100% - 40px);
    background: #eee;
    overflow: hidden;
    position: relative;
}

.image-container {
  width: 100%;
  height: 100%;
  overflow: hidden;
  user-select: none;
}

img {
  position: absolute;
  /* transition: transform 0.1s; */
}

.selection-box {
  position: absolute;
  border: 2px dashed #409EFF;
  background: rgba(33, 150, 243, 0.2);
  pointer-events: none;
  box-sizing: border-box;
}

.seleced-box {
  position: absolute;
  border: 2px solid #409EFF;
  background: rgba(33, 150, 243, 0.1);
  box-sizing: border-box;
  z-index: 99;
}

.seleced-box.valid:hover {
  background: rgba(33, 150, 243, 0.2);
}

.seleced-box.active {
  z-index: 100;
}

.seleced-box.flash {
    border: 2px solid #E6A23C40;
    background: #E6A23C10;
}

.seleced-box.flash::before {
    content: "";
    position: absolute;
    top: -2px;
    left: -2px;
    right: -2px;
    bottom: -2px;
    border: 2px solid #E6A23C;
    animation: bounce 1.5s ease-in-out infinite;
}

@keyframes bounce {
    0%, 100% {
    transform: scale(1);
    }
    33% {
        transform: scale(1.07);
    }
    66% {
        transform: scale(0.95);
    }
}

.seleced-box .anchor {
    width: 8px;
    height: 8px;
    background: #fff;
    border: 1px solid #000;
    position: absolute;
}

.seleced-box .title {
    position: absolute;
    font-size: 12px;
    padding: 2px 4px;
    background: #409EFF;
    color: #fff;
    left: 0;
    top: -16px;
}

.seleced-box .delete {
    position: absolute;
    font-size: 12px;
    padding: 1px 1px;
    border-radius: 50%;
    background: #F56C6C;
    right: -18px;
    top: -18px;
}

.seleced-box .delete .delete-icon {
    color: #fff; 
    cursor: pointer;
}

.seleced-box .anchor.lt {
    left: -5px;
    top: -5px;
    cursor: nwse-resize;
}

.seleced-box .anchor.rt {
    right: -5px;
    top: -5px;
    cursor: nesw-resize;
}

.seleced-box .anchor.lb {
    left: -5px;
    bottom: -5px;
    cursor: nesw-resize;
}

.seleced-box .anchor.rb {
    right: -5px;
    bottom: -5px;
    cursor: nwse-resize;
}

.main .bar {
    height: 40px;
    font-size: 14px;
    color: #606266;
    padding: 0 12px;
    display: flex;
    align-items: center;
}

.main .bar .text {
    margin-right: 8px;
}

.main .bar .tip {
    cursor: pointer;
}

</style>