<template>
  <div class="app-container">
    <el-form ref="form" :model="form" label-width="100px">
      <el-tabs type="border-card" style="margin-top:15px;">
        <el-tab-pane label="基本设置">
          <el-form-item label="文章标题">
            <el-input v-model="form.title" placeholder="请输入文章标题" clearable />
          </el-form-item>
          <el-form-item label="作者">
            <el-input v-model="author" style="width:25%" :disabled="true" placeholder="作者名" />
          </el-form-item>
          <el-form-item label="发布时间">
            <el-date-picker
              v-model="displayTime"
              type="datetime"
              format="yyyy-MM-dd HH:mm:ss"
              placeholder="请选择时间"
            />
          </el-form-item>
          <el-form-item label="更换封面图">
            <el-upload
              class="upload-demo"
              action="upload"
              :before-upload="beforeCoverImgUpload"
              :http-request="uploadCoverImg"
              :show-file-list="false"
              accept=".jpg,.jpeg,.png"
            >
              <el-button size="small" type="primary">
                点击上传
                <i class="el-icon-upload el-icon--right" />
              </el-button>
              <el-button
                type="primary"
                size="small"
                icon="el-icon-search"
                @click.stop="dialogshowVisible = true"
              >查看原封面图</el-button>
            </el-upload>
          </el-form-item>
          <el-form-item label="加入推荐">
            <el-checkbox-group v-model="form.type">
              <el-checkbox label="精文" border size="medium" name="type" />
              <el-checkbox label="热文" border size="medium" name="type" />
              <el-checkbox label="新文" border size="medium" name="type" />
            </el-checkbox-group>
          </el-form-item>
          <el-form-item label="审核状态">
            <el-radio-group v-model="form.review">
              <el-radio label="审核" border size="medium" />
              <el-radio label="未审核" border size="medium" />
            </el-radio-group>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="内容设置">
          <el-upload
            class="avatar-uploader"
            action="upload"
            multiple
            :before-upload="beforeCoverImgUpload"
            :http-request="uploadEditorImg"
            :show-file-list="false"
            accept=".jpg,.jpeg,.png"
          />
          <quill-editor
            ref="myQuillEditor"
            v-model="form.content"
            :options="editorOption"
          />
        </el-tab-pane>
        <el-tab-pane label="SEO设置">
          <el-form-item label="关键字">
            <el-input v-model="form.keywords" placeholder="请输入关键字" clearable />
          </el-form-item>
          <el-form-item label="文章描述" prop="desc">
            <el-input
              v-model="form.description"
              type="textarea"
              :autosize="{ minRows: 10, maxRows: 30}"
              placeholder="请输入文章描述"
            />
          </el-form-item>
        </el-tab-pane>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">提交</el-button>
        </el-form-item>
      </el-tabs>
    </el-form>
    <el-dialog width="40%" title="查看原封面图" :visible.sync="dialogshowVisible" append-to-body>
      <img :src="form.coverPicture" width="100%">
    </el-dialog>
  </div>
</template>

<script>
import { MessageBox } from 'element-ui'
export default {
  data() {
    return {
      form: {
        uid: '',
        title: '',
        type: [],
        review: '未审核',
        keywords: '',
        description: '',
        content: '',
        addTime: undefined,
        imgUrl: '',
        coverPicture: ''
      },
      editorOption: {
        theme: 'snow',
        placeholder: '请输入',
        modules: {
          toolbar: {
            container: [
              ['bold', 'italic', 'underline', 'strike'], // 加粗 斜体 下划线 删除线
              ['blockquote', 'code-block'], // 引用  代码块
              [{ 'size': ['small', false, 'large', 'huge'] }], // 文字大小
              [{ list: 'ordered' }, { list: 'bullet' }], // 有序、无序列表
              [{ script: 'sub' }, { script: 'super' }], // 上标/下标
              [{ indent: '-1' }, { indent: '+1' }], // 缩进
              [{ 'direction': 'rtl' }], // 文本方向
              [{ header: [1, 2, 3, 4, 5, 6, false] }], // 标题
              [{ color: [] }, { background: [] }], // 字体颜色、字体背景颜色
              [{ font: [] }], // 字体种类
              [{ align: [] }], // 对齐方式
              ['clean'], // 清除文本格式
              ['image'] // 图片
            ],
            handlers: {
              image(value) {
                if (value) {
                  // 调用element图片上传
                  document.querySelector('.avatar-uploader input').click()
                } else {
                  this.quill.format('image', false)
                }
              }
            }
          }
        }
      },
      author: '',
      artId: '',
      dialogshowVisible: false
    }
  },
  computed: {
    displayTime: {
      get() {
        return +new Date(this.form.addTime)
      },
      set(val) {
        this.form.addTime = new Date(val)
      }
    }
  },
  created() {
    this.showArtInfo()
  },
  methods: {
    change(val) {
      this.form.content = val
    },
    beforeCoverImgUpload(file) {
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isLt2M) {
        this.$message.error('上传头像封面图大小不能超过 2MB!')
      }
      return isLt2M
    },
    uploadCoverImg(params) {
      const that = this
      that.$request
        .uploadFile({ file: params.file }, true, {
          headers: {
            'Content-Type': 'multipart/form-data;charset=UTF-8'
          }
        })
        .then(res => {
          if (res.msg === 'success') {
            that.form.imgUrl = res.file.imgUrl
            that.$notify({
              title: '成功',
              message: '上传封面图成功',
              type: 'success'
            })
          } else {
            that.$notify.error({
              title: '错误',
              message: '上传封面图失败'
            })
          }
        })
    },
    uploadEditorImg(params) {
      const that = this
      that.$request
        .uploadFile({ file: params.file }, true, {
          headers: {
            'Content-Type': 'multipart/form-data;charset=UTF-8'
          }
        })
        .then(res => {
          // 获取富文本组件实例
          const quill = that.$refs.myQuillEditor.quill
          if (res.msg === 'success') {
            // 获取光标所在位置
            const length = quill.getSelection().index
            // 插入图片，res为服务器返回的图片链接地址
            quill.insertEmbed(length, 'image', res.file.imgUrl)
            that.$notify({
              title: '成功',
              message: '插入图片成功',
              type: 'success'
            })
          } else {
            that.$notify.error({
              title: '错误',
              message: '插入图片失败'
            })
          }
        })
    },
    showArtInfo() {
      const { id } = this.$route.query
      this.artId = id
      this.$request.showArticle({ id }).then(res => {
        const {
          uid,
          title,
          keywords,
          description,
          content,
          addTime,
          is_hot,
          is_new,
          is_best,
          status,
          imgUrl
        } = res.result
        this.form.uid = uid._id
        this.author = uid.name
        this.form.title = title
        this.form.keywords = keywords
        this.form.description = description
        this.form.content = content
        this.form.addTime = addTime
        this.form.coverPicture = imgUrl
        this.form.review = status === 1 ? '审核' : '未审核'
        if (is_hot === 1) {
          this.form.type.push('热文')
        }
        if (is_new === 1) {
          this.form.type.push('新文')
        }
        if (is_best === 1) {
          this.form.type.push('精文')
        }
      })
    },
    onSubmit() {
      this.form.artimgUrl = this.coverPicture
      this.$request
        .editArticle({ _id: this.artId, ...this.form }, true, {
          headers: {
            'Content-Type': 'multipart/form-data;charset=UTF-8'
          }
        })
        .then(res => {
          if (res.result.nModified > 0) {
            MessageBox.confirm(
              '您以编辑成功，可以取消停留在此页面上，或者前往文章列表查看',
              '编辑成功',
              {
                confirmButtonText: '前往列表页',
                cancelButtonText: '取消',
                type: 'success'
              }
            ).then(() => {
              this.$router.push({ path: '/article/list' })
            })
          } else if (res.data.result.nModified === 0) {
            this.$notify({
              title: '警告',
              message: '你好像没进行修改数据的操作,淘气~',
              type: 'warning'
            })
          } else {
            this.$notify.error({
              title: '错误',
              message: '编辑文章失败'
            })
          }
        })
    }
  }
}
</script>

<style lang="scss">
.app-container {
  background: #ffffff;
}
.quill-editor {
line-height: normal !important;
margin-bottom: 20px;
}
.quill-editor .ql-container {
  height: 350px;
}
.ql-snow .ql-tooltip[data-mode=link]::before {
  content: "请输入链接地址:";
}
.ql-snow .ql-tooltip.ql-editing a.ql-action::after {
    border-right: 0px;
    content: '保存';
    padding-right: 0px;
}

.ql-snow .ql-tooltip[data-mode=video]::before {
    content: "请输入视频地址:";
}

.ql-snow .ql-picker.ql-size .ql-picker-label::before,
.ql-snow .ql-picker.ql-size .ql-picker-item::before {
  content: '14px';
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value=small]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value=small]::before {
  content: '10px';
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value=large]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value=large]::before {
  content: '18px';
}
.ql-snow .ql-picker.ql-size .ql-picker-label[data-value=huge]::before,
.ql-snow .ql-picker.ql-size .ql-picker-item[data-value=huge]::before {
  content: '32px';
}

.ql-snow .ql-picker.ql-header .ql-picker-label::before,
.ql-snow .ql-picker.ql-header .ql-picker-item::before {
  content: '文本';
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="1"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="1"]::before {
  content: '标题1';
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="2"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="2"]::before {
  content: '标题2';
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="3"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="3"]::before {
  content: '标题3';
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="4"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="4"]::before {
  content: '标题4';
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="5"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="5"]::before {
  content: '标题5';
}
.ql-snow .ql-picker.ql-header .ql-picker-label[data-value="6"]::before,
.ql-snow .ql-picker.ql-header .ql-picker-item[data-value="6"]::before {
  content: '标题6';
}

.ql-snow .ql-picker.ql-font .ql-picker-label::before,
.ql-snow .ql-picker.ql-font .ql-picker-item::before {
  content: '标准字体';
}
.ql-snow .ql-picker.ql-font .ql-picker-label[data-value=serif]::before,
.ql-snow .ql-picker.ql-font .ql-picker-item[data-value=serif]::before {
  content: '衬线字体';
}
.ql-snow .ql-picker.ql-font .ql-picker-label[data-value=monospace]::before,
.ql-snow .ql-picker.ql-font .ql-picker-item[data-value=monospace]::before {
  content: '等宽字体';
}
</style>
