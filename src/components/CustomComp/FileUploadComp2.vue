<template>
  <div class="FileUploadComp">
    <form class="FileCompForm" :id="lalala" enctype="multipart/form-data">
      <div class="upload-img" v-loading="loading21">
        <div class="FileList" v-for="(file, index) in fileListComment" v-bind:key="index">
          <img :src="file.previewUrl" :alt="file.showName">
          <div class="cover" @click="delUploadFileComment(file.attachmentId)">
            <i class="el-icon-delete"></i>
          </div>
        </div>
        <!-- v-if="fileListComment.length < 5"-->
        <div class="FileCompIptWrap">
           <i class="el-icon-plus"></i>
           <input type="file" class="aaa" :disabled="commentDis" name="myfile" :id="lalala + '_myfile2'" @change="getFileName">
        </div>
        <p style="font-size: 12px;color:red;position: absolute;left:0;bottom: 0"  v-if="fileListComment.length === 5">最多上传5张图片!</p>
      </div>
    </form>
    <!--<div style="font-size: 12px">-->
      <!--<span style="color: #409EFF" v-if="fileListComment.length > 0" v-for="(file, index) in fileListComment" v-bind:key="index">-->
        <!--<span style="color: #333" @click="FilePreEmitFuc(file.previewUrl, file.fileName, file.attachmentId)">{{index+1}}、{{file.fileName}}</span>-->
        <!--<div style="color: #409EFF;display: inline-block;cursor: pointer;" class="el-icon-close" @click="delUploadFileComment(file.attachmentId)"></div>;&nbsp;&nbsp;-->
      <!--</span>-->
    <!--</div>-->
  </div>
</template>

<script>
export default {
  name: 'FileUploadComp',
  props: {
    clearInfo: {
      type: Boolean,
      default: false
    },
    fileFormId: {
      type: String,
      default: '选择文件'
    },
    FileDataList: {
      // type: Array,
      // default: []
    },
    buttonStr: {
      type: String,
      default: '选择文件'
    },
    selectType: {
      type: String,
      default: '附件'
    },
    filUrl: {
      type: String,
      default: '/file/uploadFileAjax'
    },
    limtImg: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      lalala: '',
      loading21: false,
      uploadFileName: '',
      uploadFileRe: '',
      commentDis: false,
      fileListComment: [],
      fileListCommentLen: 0
    }
  },
  created () {
    this.$store.state.uploadCount = this.$store.state.uploadCount + 1
    this.lalala = 'lala_' + this.$store.state.uploadCount
    // this.$Bus.$on('proFileList', list => {
    //   this.log('FileDataList:', list)
    //   this.fileListComment = list
    //   if (list.length >= 5) {
    //     this.commentDis = true
    //   }
    // })
  },
  watch: {
    FileDataList (val, old) {
      this.log('FileDataList:', val)
      if (val) {
        this.fileListComment = val
        if (val.length >= 5) {
          this.commentDis = true
        }
      }
    },
    clearInfo (val, old) {
      var that = this
      // that.log('接收到父组件传值', val)
      if (val) {
        // that.log('父组件传过来的：', val)
        that.fileListComment = []
        $('#' + that.lalala + '_myfile2').val('')
        that.uploadFileName = ''
      }
    },
    fileListCommentLen: function (val, oVal) {
      if (val >= 5) {
        this.commentDis = true
      } else if (val < 5) {
        this.commentDis = false
      }
    }
  },
  methods: {
    FilePreEmitFuc: function (previewUrl, fileName, attachmentId) {
      this.$emit('FilePreEmit', {previewUrl, fileName, attachmentId})
    },
    getFileName: function (filename) {
      var that = this
      console.log(filename)
      var filePath = $('#' + that.lalala + '_myfile2').val()
      var arr = filePath.split('\\')
      var fileName = arr[arr.length - 1]
      // this.uploadFileName = fileName
      console.log(fileName)
      if (filePath) {
        if (that.limtImg) {
          if (that.isImage(fileName)) {
            that.addMarkInfo4()
            this.uploadFileName = fileName
          } else {
            this.$message.warning('请上传图片')
            this.uploadFileName = ''
          }
        } else {
          that.addMarkInfo4()
          this.uploadFileName = fileName
        }
        // that.addMarkInfo4()
      }
    },
    delUploadFileComment: function (id) {
      // console.log('id', id)
      var that = this
      that.$confirm('确认删除此附件，确定删除？', '', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        that.ajax('/file/deleteFile', {attachmentId: id}).then(res => {
          // this.log('选择所属项目:', res)
          if (res.code === 200) {
            that.$message.success('删除成功！')
            for (var i = 0; i < that.fileListComment.length; i++) {
              if (id === that.fileListComment[i].attachmentId) {
                that.fileListComment.splice(i, 1)
              }
            }
            this.$emit('delUploadFileComment', that.fileListComment)
            that.fileListCommentLen = that.fileListComment.length
            console.log(that.fileListComment)
            // console.log('edit', that.fileListComment)
            $('#myfile2').val('')
          }
        })
      }).catch(() => {
        return false
      })
    },
    addMarkInfo4 () {
      var that = this
      that.loading21 = true
      var url = that.$store.state.baseServiceUrl
      var formData = new FormData($('#' + that.lalala)[0])
      // this.log(456)
      if (formData) {
        $.ajax({
          type: 'post',
          url: url + that.filUrl,
          data: formData,
          cache: false,
          processData: false,
          contentType: false,
          crossDomain: true,
          xhrFields: {
            withCredentials: true
          }
        }).then(function (data) {
          that.log('upload:', data)
          if (data.code === 200) {
            // that.log('uploadFileAjax:', data)
            // that.attachmentId2 = data.data.attachmentId
            var obj = {
              attachmentId: data.data.attachmentId,
              fileName: data.data.showName,
              previewUrl: data.data.previewUrl
            }
            that.fileListComment.push(obj)
            // that.log('fileListComment:', that.fileListComment.length)
            that.fileListCommentLen = that.fileListComment.length
            that.$message({
              type: 'success',
              message: '文件' + data.msg
            })
            that.loading21 = false
            that.$emit('FileDataEmit', that.fileListComment)
          } else if (data.code === 300) {
            that.$message({
              type: 'error',
              message: data.msg
            })
            that.loading21 = false
          } else {
            that.$message({
              type: 'error',
              message: data.msg
            })
            that.loading21 = false
          }
        })
      } else {
        // that.loading = false
        that.$message({
          type: 'error',
          message: '内容不能为空'
        })
        that.loading21 = false
      }
    }
  }
}

</script>

<style scoped>
  div img{
    width: 100%;
  }
  .upload-img {
    position: absolute;
    width: 260px;
    height: 170px;
    right: 140px;
    top:10px;
  }
  .FileList {
    width: 60px;
    height: 60px;
    margin-right: 10px;
    border: 1px solid #d9d9d9;
    border-radius: 5px;
    cursor: pointer;
    position: relative;
    margin-top: 7px;
    float: left;
  }
  .FileList img {
    width: 100%;
    height: 100%;
  }
  .cover {
    width: 100%;
    height: 100%;
    position: absolute;
    left:0;
    top:0;
    background: rgba(0,0,0,0.5);
    display: none;
    text-align: center;
    line-height: 60px;
    color: #fff;
    font-size: 18px;
    z-index: 15;
  }
  .FileList:hover .cover {
    display: block;
  }
  .FileUploadComp{
  }
  .FileCompForm{
    display: flex;
    /*width: 80%;*/
  }
  .FileCompFormIcon{
    width: 20px;
    height: 40px;
    padding-top: 10px;
    margin-right: 10px;
  }
  .FileCompIptWrap{
    width: 60px;
    height: 60px;
    border: 1px solid #99D3F5;
    border-radius: 4px;
    color: #8c939d;
    line-height: 20px;
    margin-top: 6px;
    font-size: 25px;
    text-align: center;
    line-height: 60px;
    float: left;
    cursor: pointer;
    position: relative;
  }
  .FileCompIptWrap input{
    width: 100%;
    position: absolute;
    left: 0;
    top: 0;
    opacity: 0;
  }
  .FileCompIptWrap:hover {
    background: #AADFFD;
    border-color: #78C3F3;
    color: #004974;
  }
  .FileCompIptText{
    padding-top: 10px;
    margin-left: 10px;
  }
</style>
