<template>
  <div>
    <el-upload
      :action="action"
      :file-list="fileList"
      list-type="picture-card"
      :limit="limit"
      :accept="accept"
      :class="{hide: this.fileList.length >= this.limit}"
      :on-error="handleError"
      :before-upload="beforeUpload"
      :on-success="handleImageSuccess"
      multiple
    >

      <i slot="default" class="el-icon-plus"></i>

      <div slot="file" slot-scope="{ file }">
        <div>

          <img
            class="el-upload-list__item-thumbnail"
            :src="isPDF(file.url) ? pdf : file.url"
            alt=""/>

          <span
            v-if="file.status === 'success'"
            class="el-upload-list__item-actions"
          >
          <span
            v-if="preview"
            class="el-upload-list__item-preview"
            @click="handlePreview(file)"
          >
            <i class="el-icon-zoom-in"></i>
          </span>
          <span
            v-if="download && !isPDF(file.url)"
            class="el-upload-list__item-delete"
            @click="handleDownload(file)"
          >
            <i class="el-icon-download"></i>
          </span>
          <span
            v-if="deleted"
            class="el-upload-list__item-delete"
            @click="handleRemove(file)"
          >
            <i class="el-icon-delete"></i>
          </span>
        </span>
          <span
            v-else
            :class="[
            uploading ? 'uploading' : '',
            'el-upload-list__item-actions',
          ]"
          >
          <i class="el-icon-loading"/><i style="font-size: 14px">上传中</i>
        </span>


        </div>

      </div>
    </el-upload>

    <el-dialog :visible.sync="previewVisible"
               title="预览"
               width="800"
               append-to-body>
      <img width="100%" :src="previewImgUrl" alt="" style="display: block;"/>
    </el-dialog>
  </div>
</template>

<script>

import logo from '@/assets/images/pdf.png'

export default {
  name: 'FileUploadPlus',
  props: {
    value: {
      type: [String, Array],
      default: ''
    },
    // 上传的地址
    action: {
      type: String,
      default: process.env.VUE_APP_BASE_API + "/system/file/upload", //上传接口
    },
    // 设置上传的请求头部
    headers: {
      type: Object,
      default: () => {
        return {}
      },
    },
    // 上传文件大小限制， 默认 50M，单位M
    size: {
      type: [Number, String],
      default: 50,
    },
    // 文件上传格式， 默认jpeg, png,jpg
    accept: {
      type: String,
      default: 'image/jpeg,image/png',
    },
    // 是否显示删除操作按钮
    deleted: {
      type: Boolean,
      default: true,
    },
    // 是否显示预览操作按钮
    preview: {
      type: Boolean,
      default: true,
    },
    // 是否显示下载操作按钮
    download: {
      type: Boolean,
      default: true,
    },
    // 上传文件个数限制，默认0 不限制
    limit: {
      type: [Number, String],
      default: 0,
    },
  },
  data() {
    return {
      fileList: [], // 默认文件列表
      hideUpload: true, // 超出限制掩藏上传按钮
      uploading: true, // 是否上传中,上传时隐藏上传按钮
      previewImgUrl: '', // 预览图片地址
      previewVisible: false, // 是否显示预览
      pdf: logo,
      files: [], // 文件url数组
    }
  },
  watch: {
    value: {
      /*handler(val, old){
        console.log('FileUpload=', val)
        if(val){
          this.files = val
          this.fileList = [] // 先清空
          for(var i=0; i<val.length;i++){
            this.fileList.push({
              url: val[i] // 转化
            })
          }
          this.handleChange()
        }
      },*/
      handler(val) {
        this.fileList = [] // 先清空
        if (val) {
          this.files = val
          if(typeof val === "string"){
            this.fileList.push({
              url: val // 转化
            });
          }else{
            for (var i = 0; i < val.length; i++) {
              this.fileList.push({
                url: val[i].url // 转化
              })
            }
          }
          this.handleChange()
        }
      },
      deep: true,
      immediate: true // 避免在子组件中监听失效
    }
  },
  methods: {
    emitInput() {
      this.$emit('input', this.files)
    },
    // 判断是否pdf
    isPDF(url) {
      if (!url) {
        return false
      }
      const fileType = ['pdf']
      const index = url.lastIndexOf('.')
      const type = url.substr(index + 1)
      return fileType.indexOf(type) > -1
    },
    // 文件上传成功
    handleImageSuccess(res) {
      if (res.code === 200) {
        this.files.push(res.data.file)
        this.emitInput()
      } else {
        this.$message.error('文件上传失败')
      }
    },
    // 上传前文件大小判断
    beforeUpload(file) {
      const {size} = this
      const overSize = size > 0 && file.size < 1024 * 1024 * size
      if (!overSize) this.$message.error(`上传文件大小不能超过 ${size}MB!`)
      this.uploading = overSize // 是否上传中
      return overSize
    },
    // 上传出错返回
    handleError(event, file, fileList) {
      console.log(event, file, fileList, 'error')
      this.$message.error('服务出错，上传失败！')
      this.handleChange()
    },
    // 删除图片
    async handleRemove(file) {
      this.$confirm(`确认删除文件?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const {fileList} = this
        this.files = this.files.filter((v) => v !== file.url)
        this.emitInput()
      }).catch(() => {
      })
    },
    // 图片预览
    handlePreview(file) {
      if (this.isPDF(file.url)) {
        window.open(file.url, "_blank");
      } else {
        this.previewImgUrl = file.url
        this.previewVisible = true
      }
    },
    handleChange(file, list) {
      const {limit, fileList} = this
      if (limit > 0 && fileList.length >= limit) this.hideUpload = true
      else this.hideUpload = false
      this.uploading = false
    },
    handleDownload(file) {
      window.open(file.url, "_blank");
      /*const a = document.createElement('a')
      a.href = file.url
      a.click() // 模拟点击事件，实现图片文件的同源下载
      */
    },
  },
}
</script>

<style lang="scss" scoped>
::v-deep.hide .el-upload--picture-card {
  display: none;
}

.hideUpload .el-upload--picture-card {
  display: none;
}

.el-upload-list--picture-card .uploading.el-upload-list__item-actions {
  opacity: 1;
}

/*添加、删除文件时去掉动画过渡*/
.el-upload-list__item {
  transition: none !important;
}

.el-upload-list--picture-card .el-upload-list__item-thumbnail {
  width: 148px;
  height: 148px;
}

.el-upload-list--picture-card .el-upload-list__item {
  background-color: inherit;
  border: none;
  width: 148px;
  height: 148px;
}

::v-deep.hide .el-upload--picture-card {
  display: none;
}
// 去掉动画效果
::v-deep .el-list-enter-active,
::v-deep .el-list-leave-active {
  transition: all 0s;
}

::v-deep .el-list-enter, .el-list-leave-active {
  opacity: 0;
  transform: translateY(0);
}
</style>
