<template>
    <div class="config-title">
        <i :class="icon"></i>
        <span style="margin-left: 12px;">{{ name }}</span>
    </div>
</template>

<script>
export default {
    name: 'ConfigTitle',
    props: ['name', 'icon']
}
</script>

<style scoped>
.config-title {
    padding-left: 16px;
    font-size: 14px;
    color: #606266;
}
</style>