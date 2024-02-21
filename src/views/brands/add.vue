<template>
  <main-panel title="编辑品牌" backPath="/products">
    <el-row v-loading="pageLoading">
      <el-form ref="form" :rules="rules" :model="form" label-width="80px" size="small">
        <el-form-item label="名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入品牌名称"></el-input>
        </el-form-item>
        <el-form-item label="简介" prop="simpleDesc">
          <el-input type="textarea" :rows="2" v-model="form.simpleDesc" auto-complete="off" placeholder="请输入品牌简介"></el-input>
        </el-form-item>
        <el-form-item label="起步价" prop="floorPrice">
          <el-input v-model.number="form.floorPrice" style="width: 130px;" placeholder="起步价"></el-input>
        </el-form-item>
        <el-form-item label="品牌列表/商品页展示图" class="no-line-height" prop="list_pic_url">
          <croppa v-model="coverCroppa"
            :width="375"
            :height="210"
            canvas-color="default"
            placeholder="点击选择, PNG/JPEG, <500K"
            :placeholder-font-size="12"
            placeholder-color="default"
            accept="image/png,image/jpeg"
            :file-size-limit="512000"
            :quality="2"
            :prevent-white-space="true"
            :remove-button-size="22"
            :initial-image="form.listPicUrl"
            @file-size-exceed="onFileSizeExceed"
            @file-type-mismatch="onFileTypeMismatch"
            @image-remove="onImageRemove(form.listPicUrl, 'cover')">
          </croppa>
        </el-form-item>
        <el-form-item label="首页Banner" class="no-line-height" prop="new_pic_url">
          <croppa v-model="imgCroppa1"
            :width="330"
            :height="242"
            canvas-color="default"
            placeholder="点击选择, PNG/JPEG, <500K"
            :placeholder-font-size="12"
            placeholder-color="default"
            accept="image/png,image/jpeg"
            :file-size-limit="512000"
            :quality="2"
            :prevent-white-space="true"
            :remove-button-size="22"
            :initial-image="form.newPicUrl"
            @file-size-exceed="onFileSizeExceed"
            @file-type-mismatch="onFileTypeMismatch"
            @image-remove="onImageRemove(form.newPicUrl, 'img1')">
          </croppa>
        </el-form-item>
        <el-form-item label="首页显示">
          <el-radio class="radio" v-model="form.isShow" :label="1">是</el-radio>
          <el-radio class="radio" v-model="form.isShow" :label="0">否</el-radio>
        </el-form-item>
        <el-form-item label="是否新品">
          <el-radio class="radio" v-model="form.isNew" :label="1">是</el-radio>
          <el-radio class="radio" v-model="form.isNew" :label="0">否</el-radio>
        </el-form-item>
        <el-form-item label="列表排序">
          <el-input-number v-model="form.sortOrder" :min="1" :max="1000"></el-input-number>
        </el-form-item>
        <el-form-item label="首页排序">
          <el-input-number v-model="form.newSortOrder" :min="1" :max="1000"></el-input-number>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="save" :loading="loading">保存</el-button>
          <router-link :to="{ path: '/brands' }">
            <el-button>取消</el-button>
          </router-link>
        </el-form-item>
      </el-form>
    </el-row>
  </main-panel>
</template>

<script>
import * as api from 'api'
import * as utils from 'utils'

export default {
  data () {
    let that = this
    let coverValidate = function (rule, value, callback) {
      if (!that.coverCroppa.hasImage()) {
        callback(new Error('请选择列表页/商品页显示图'))
      }
      callback()
    }
    let imageValidate = function (rule, value, callback) {
      if (!that.imgCroppa1.hasImage()) {
        callback(new Error('请选择首页/新品显示图'))
      }
      callback()
    }
    return {
      id: null,
      pageLoading: false,
      form: {
        id: null,
        name: '',
        simpleDesc: '',
        floorPrice: null,
        listPicUrl: null,
        newPicUrl: null,
        isShow: 1,
        isNew: 0,
        sortOrder: 100,
        newSortOrder: 10
      },
      coverChanged: true,
      img1Changed: true,
      coverCroppa: null,
      imgCroppa1: null,
      removedImgs: [], // 用于图片删除，这里不删除七牛图片
      loading: false,
      rules: {
        
      }
    }
  },
  mounted () {
  },
  methods: {
    save () {
      this.$refs.form.validate(async (valid) => {
        if (valid) {
          this.loading = true
          if (this.coverCroppa.hasImage() && this.coverChanged) {
            const cover = await this.coverCroppa.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(cover, api.BrandImgPrefix)
            this.form.listPicUrl = res.key
          }
          if (this.imgCroppa1.hasImage() && this.img1Changed) {
            const img1 = await this.imgCroppa1.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(img1, api.BrandImgPrefix)
            this.form.newPicUrl = res.key
          }
          // thinkjs不支持直接传递数组，改为json格式提交
          this.$http.post(api.BRAND + '/store', this.form, {headers: {'Content-Type': 'application/json'}}).then((data) => {
            this.loading = false
            this.$notify({title: '成功', message: '保存成功', type: 'success'})
            this.$router.push({ path: '/brands' })
          }).catch(err => {
            this.loading = false
            this.$message.error(err + '')
          })
        }
      })
    },
    onFileSizeExceed (file) {
      this.$message.error('图片大小不能超过 500K')
    },
    onFileTypeMismatch (file) {
      this.$message.error('图片只能是 JPG/PNG 格式！')
    },
    onImageRemove (path, type) {
      this.removedImgs.push(path)
      this.removedImgs = [...new Set(this.removedImgs)]
      switch (type) {
        case 'cover':
          this.coverChanged = true
          break
        case 'img1':
          this.img1Changed = true
          break
      }
    }
  }
}
</script>

<style scoped>

</style>
