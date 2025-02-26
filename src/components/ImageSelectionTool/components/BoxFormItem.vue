<template>
    <div class="form-item">
        <span>{{ label }}</span>
        <input class="input" type="number" :value="value" @input="$emit('input', $event.target.value)">
    </div>
</template>

<script>
export default {
    name: 'BoxFormItem',
    props: ['value', 'label']
}
</script>

<style scoped>
.form-item {
    display: flex;
    gap: 8px;
    color: #606266;
    margin-bottom: 6px;


}

.input {
    border: none;
    border-bottom: 1px solid #EBEEF5;
    outline: none;
    width: 60px;
    color: #606266;
    font-size: 12px;
}
</style>