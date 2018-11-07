<template>
<div>
    <div>
        <el-alert title="这个一个COS 上传测试页面！">
        </el-alert>
        <el-alert :title="result">
        </el-alert>
    </div>
    <!-- 上传测试功能开始 -->
    <div>
        <el-alert :title="fileAuth"></el-alert>
        <el-upload class="upload-demo" :show-file-list="showfilelist" ref="upload" multiple action="https://xiaochengxu-1252144493.cos.ap-beijing.myqcloud.com" :on-preview="handlePreview" :on-remove="handleRemove" :file-list="fileList" :http-request="myUpload" :headers="uploadHeader" :auto-upload="false">
            <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
            <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>
        
        <div v-for="i in uploadFileList" style="width:300px; margin-top:20px; font-szie:12px;">
            <div>{{i.name}}</div>
            <el-progress :text-inside="true" :stroke-width="18" :status="i.status" :percentage="i.progress" color="#8e71c7"></el-progress>
        </div>
        
    </div>
</div>
</template>

<script>
import axios from 'axios'
export default {
    data() {
        return {
            uploadHeader: {
                Authorization: '000'
            },
            showfilelist:true,
            status: "",
            progress: 0,
            fileAuth: '暂无授权码',
            datafile: '',
            result: '还没开始！',
            headers: {
                headers: {}
            },
            uploadFileList: [],
            fileList: []
        };
    },
    methods: {

        myUpload(content) {
            var P = new Promise((resolve, reject) => {
                axios.get('http://127.0.0.1:5000/test1000/COS', {
                    params: {
                        method: 'put',
                        pathname: '/' + content.file.name
                    }
                }).then((res) => {
                    this.fileAuth = res.data
                    this.authorization = res.data
                    console.log(res.data)
                    console.log(typeof this.fileAuth)
                    this.headers.headers["Authorization"] = this.fileAuth;
                    this.result = '正在上传……';
                    console.log('授权成功' + content.file.name)
                    console.log(this.headers)
                    this.status = "";
                    this.uploadFileList.push({'name':content.file.name,'progress':0,'status':''})
                    
                    resolve('授权成功')

                })
            }).then(() => {
                console.log('开始上传' + content.file.name)
                var i,indexnumber;
                for(i in this.uploadFileList)
                {
                  if( this.uploadFileList[i].name===content.file.name)
                  {
                    indexnumber=i
                    break;
                  }
                }

                var that = this;
                var config = {

                    headers: this.headers.headers,
                    onUploadProgress: function (progressEvent) {
                        // 对原生进度事件的处理
                        that.uploadFileList[indexnumber].progress = Math.round(progressEvent.loaded / progressEvent.total * 10000) / 100

                    }
                };
                axios.put(content.action + '/' + content.file.name, content.file, config).then((res) => {
                    console.log(res)
                    
                    this.uploadFileList[indexnumber].status = "success";
                    if (this.result === '正在上传……') {
                        this.result = `上传完毕：${content.file.name} ，`
                        console.log('上传完毕：' + content.file.name)
                    } else {
                        this.result += `上传完毕：${content.file.name} ，`
                        console.log('上传完毕：' + content.file.name)
                    }
                })
            })
        },
        bupone(p) {
            this.progress = p
        },
        submitUpload(file, fileList) {
            this.$refs.upload.submit();
        },
        handleRemove(file, fileList) {
            console.log(file, fileList);
        },
        handlePreview(file) {
            console.log(file);
        }

    }
}
</script>
