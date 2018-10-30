<template>
  <div>
    <div>
      <el-alert title="这个一个COS 上传测试页面！">
      </el-alert>
    </div>
    <!-- 上传测试功能开始 -->
    <div>
      <el-alert :title="fileAuth"></el-alert>
      <el-upload class="upload-demo" ref="upload" action="https://xiaochengxu-1252144493.cos.ap-beijing.myqcloud.com" :on-preview="handlePreview" :on-remove="handleRemove" :file-list="fileList" :http-request="myUpload" :headers="uploadHeader" :auto-upload="false">
        <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
        <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
      </el-upload>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    data() {
      return {
        uploadHeader: {
          Authorization: '21131'
        },
        fileAuth: '暂无授权码',
        datafile: '',
        headers: {
          headers: {}
        },
        fileList: [{
          name: 'food.jpeg',
          url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
        }, {
          name: 'food2.jpeg',
          url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
        }]
      };
    },
    methods: {
  
      myUpload(content) {
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
          this.headers.headers["Authorization"] = this.fileAuth
          console.log('headers:')
          console.log(this.headers)
        })
  
        console.log(content.file.name)
        setTimeout(() => {
          axios.put(content.action + '/' + content.file.name, content.file, this.headers).then((res) => {
            console.log('执行完毕！')
          })
        }, 1000)
  
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
