<template>
    <div>
        <input 
            class="input" 
            :class="{error: isRepeat || !this.editedName}"
            ref="input" 
            v-if="isEditing" 
            v-model="editedName" 
            @keypress.enter="handleConfirm"
            @blur="handleConfirm"
        />
        <p v-else @click="handleNameClick">{{ value }}</p>
    </div>
</template>

<script>
export default {
    name: 'NameEditor',
    props: {
        value: {
            type: String,
            required: true
        },
        boxes: {
            type: Array,
            default: () => []
        }
    },
    data() {
        return {
            isEditing: false,
            editedName: ''
        }
    },
    computed: {
        isRepeat() {
            return !!this.boxes.find(el => el.name === this.editedName && this.editedName !== this.value)
        }
    },
    methods: {
        handleNameClick() {
            this.editedName = this.value
            this.isEditing = true
            this.$nextTick(() => {
                this.$refs.input.focus()
            })
        },

        handleConfirm() {
            if(!this.isEditing) return 
            if(!this.isRepeat && this.editedName) {
                // 没有重复就更新
                this.$emit('input', this.editedName)
            }
            this.isEditing = false
        }
    }
}
</script>

<style scoped>
.input {
    border: none;
    border-bottom: 1px solid #EBEEF5;
    outline: none;
    color: #606266;
    font-size: 12px;
}

.input.error {
    border-bottom: 1px solid #F56C6C;
}
</style>