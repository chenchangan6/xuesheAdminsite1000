<template>
<div>
    <el-alert title="这是上传组件测试！"></el-alert>
    <div>
        <Upload/>
    </div>
</div>
</template>
<script>
import Upload from '~/components/upload.vue'
export default {
    components: {
    Upload
    }
}
</script>
