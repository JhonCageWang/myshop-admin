<template>
  <main-panel title="编辑商品" backPath="/products">
    <el-tabs v-model="activeName" v-loading="pageLoading">
      <el-tab-pane label="基础信息" name="info">
        <el-row>
          <el-form ref="form" :rules="rules" :model="form" label-width="80px" size="small">
            <el-form-item label="名称" prop="name">
              <el-input v-model="form.name"></el-input>
            </el-form-item>
            <el-form-item label="摘要">
              <el-input type="textarea" :rows="2" v-model="form.goodsBrief" auto-complete="off"></el-input>
            </el-form-item>
            <el-form-item label="货品编码" prop="goodsSn">
              <el-input v-model="form.goodsSn"></el-input>
            </el-form-item>
            <el-form-item label="分类" prop="categories">
              <el-tree ref="catTree" :data="categories" :props="props" node-key="id" @check-change="onSelectCat" show-checkbox></el-tree>
            </el-form-item>
            <el-form-item label="品牌">
              <el-select ref="brandTree" :data="brands" v-model="form.brandId">
                <el-option
                  v-for="item in brands"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="属性">
              <el-row v-for="(item, index) in form.attrs" :key="index">
                <el-input v-model="item.name" placeholder="属性名" style="width: 100px;" :style="{marginTop: index > 0 ? '10px' : ''}"></el-input>：
                <el-input v-model="item.value" placeholder="属性值" style="width: 500px;" :style="{marginTop: index > 0 ? '10px' : ''}"></el-input>
                <el-button type="text" icon="el-icon-plus" @click="attrAdd(index)"></el-button>
                <el-button type="text" icon="el-icon-delete" v-if="index > 0" @click="attrDestroy(index)"></el-button>
              </el-row>
            </el-form-item>
            <el-form-item label="封面" class="no-line-height" prop="listPicUrl">
              <croppa v-model="coverCroppa"
                :width="200"
                :height="200"
                canvas-color="default"
                placeholder="点击选择, PNG/JPEG, <500K"
                :placeholder-font-size="12"
                placeholder-color="default"
                accept="image/png,image/jpeg"
                :file-size-limit="512000"
                :quality="4"
                :prevent-white-space="true"
                :initial-image="initImgs.cover"
                @file-size-exceed="onFileSizeExceed"
                @file-type-mismatch="onFileTypeMismatch"
                @image-remove="onImageRemove(form.listPicUrl, 'cover')">
              </croppa>
            </el-form-item>
            <el-form-item label="展示图" class="no-line-height" prop="gallery">
              <croppa v-model="imgCroppa1"
                :width="200"
                :height="200"
                canvas-color="default"
                placeholder="点击选择, PNG/JPEG, <500K"
                :placeholder-font-size="12"
                placeholder-color="default"
                accept="image/png,image/jpeg"
                :file-size-limit="512000"
                :quality="4"
                :prevent-white-space="true"
                :initial-image="initImgs.gallery[0]"
                @file-size-exceed="onFileSizeExceed"
                @file-type-mismatch="onFileTypeMismatch"
                @image-remove="onImageRemove(form.gallery[0].imgUrl, 'img1')">
              </croppa>
              <croppa v-model="imgCroppa2"
                :width="200"
                :height="200"
                canvas-color="default"
                placeholder="点击选择, PNG/JPEG, <500K"
                :placeholder-font-size="12"
                placeholder-color="default"
                accept="image/png,image/jpeg"
                :file-size-limit="512000"
                :quality="4"
                :prevent-white-space="true"
                :initial-image="initImgs.gallery[1]"
                @file-size-exceed="onFileSizeExceed"
                @file-type-mismatch="onFileTypeMismatch"
                @image-remove="onImageRemove(form.gallery[1], 'img2')">
              </croppa>
              <croppa v-model="imgCroppa3"
                :width="200"
                :height="200"
                canvas-color="default"
                placeholder="点击选择, PNG/JPEG, <500K"
                :placeholder-font-size="12"
                placeholder-color="default"
                accept="image/png,image/jpeg"
                :file-size-limit="512000"
                :quality="4"
                :prevent-white-space="true"
                :initial-image="initImgs.gallery[2]"
                @file-size-exceed="onFileSizeExceed"
                @file-type-mismatch="onFileTypeMismatch"
                @image-remove="onImageRemove(form.gallery[2], 'img3')">
              </croppa>
              <croppa v-model="imgCroppa4"
                :width="200"
                :height="200"
                canvas-color="default"
                placeholder="点击选择, PNG/JPEG, <500K"
                :placeholder-font-size="12"
                placeholder-color="default"
                accept="image/png,image/jpeg"
                :file-size-limit="512000"
                :quality="4"
                :prevent-white-space="true"
                :initial-image="initImgs.gallery[3]"
                @file-size-exceed="onFileSizeExceed"
                @file-type-mismatch="onFileTypeMismatch"
                @image-remove="onImageRemove(form.gallery[3], 'img4')">
              </croppa>
            </el-form-item>
            <el-form-item label="描述" class="no-line-height" prop="goodsDesc">
              <quill-editor
                v-model="form.goodsDesc"
                ref="editor"
                :options="editorOption"
                style="height: 500px;">
              </quill-editor>
            </el-form-item>
            <el-form-item label="常见问题" style="margin-top: 86px;">
              <el-row v-for="(item, index) in form.faq" :key="index">
                <el-input v-model="item.question" placeholder="问题名称" style="width: 200px;" :style="{marginTop: index > 0 ? '10px' : ''}"></el-input>：
                <el-input v-model="item.answer" placeholder="回答" style="width: 600px;" :style="{marginTop: index > 0 ? '10px' : ''}"></el-input>
                <el-button type="text" icon="el-icon-plus" @click="faqAdd(index)"></el-button>
                <el-button type="text" icon="el-icon-delete" v-if="index > 0" @click="faqDestroy(index)"></el-button>
              </el-row>
            </el-form-item>
            <el-form-item label="库存" prop="goodsNumber">
              <el-input-number v-model.number="form.goodsNumber" controls-position="right" :min="1" :max="100000"></el-input-number>
              <el-tooltip v-if="specs.length" effect="dark" content="存在商品规格组合时，该库存失效，实际库存为规格组合的库存。" placement="top">
                <i class="el-icon-info" style="color: red;"></i>
              </el-tooltip>
            </el-form-item>
            <el-form-item label="价格" prop="retailPrice">
              <el-input v-model.number="form.retailPrice" style="width: 130px;"></el-input>
              <el-tooltip v-if="specs.length" effect="dark" content="存在商品规格组合时，该价格失效，实际价格为规格组合的价格。" placement="top">
                <i class="el-icon-info" style="color: red;"></i>
              </el-tooltip>
            </el-form-item>
            <el-form-item label="新品">
              <el-radio class="radio" v-model="form.isNew" :label="1">是</el-radio>
              <el-radio class="radio" v-model="form.isNew" :label="0">否</el-radio>
            </el-form-item>
            <el-form-item label="排序">
              <el-input-number v-model="form.sortOrder" :min="1" :max="1000"></el-input-number>
            </el-form-item>
            <!-- <el-form-item label="物流">
              <el-radio class="radio" v-model="form.need_express" :label="1">是</el-radio>
              <el-radio class="radio" v-model="form.need_express" :label="0">否</el-radio>
            </el-form-item> -->
            <el-form-item label="在售">
              <el-switch
                v-model="form.isOnSale"
                active-color="#13ce66"
                inactive-color="#ff4949"
                active-text="是"
                inactive-text="否"
                :active-value="1"
                :inactive-value="0">
              </el-switch>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="save" :loading="loading">保存</el-button>
              <router-link :to="{ path: '/products' }">
                <el-button>取消</el-button>
              </router-link>
            </el-form-item>
          </el-form>
        </el-row>
      </el-tab-pane>
      <el-tab-pane label="规格组合" name="combination">
        <el-row :gutter="24">
          <el-col :span="10" :xs="24" :sm="24" :md="10" :lg="10" style="margin-bottom: 20px;">
            <el-button type="success" icon="el-icon-plus" size="mini" @click="newSpec" style="margin-bottom: 20px;">新增规格类型</el-button>
            <el-form ref="combinationForm" :rules="rulesAttr" :model="formSpec" label-width="80px" size="small">
              <template v-for="(sitem, index) in formSpec.specArr">
                <el-form-item label="类型" :prop="'specArr.' + index + '.selectedSpecs'" :rules="{required: true, validator: specValidate}">
                  <el-select v-model="sitem.selectedSpecs" placeholder="请选择" style="width: 120px;margin-right: 10px;">
                    <el-option v-for="item in specs" :key="item.id" :label="item.name" :value="item.id"></el-option>
                  </el-select>
                  <el-button type="text" icon="el-icon-plus" @click="add"></el-button>
                  <el-button type="text" icon="el-icon-delete" @click="remove(index)" v-show="index > 0"></el-button>
                </el-form-item>
                <el-form-item label="规格" :prop="'specArr.' + index + '.selectedSpecVal'" :rules="{required: true, message: '请填写具体规格'}">
                  <el-input v-model="sitem.selectedSpecVal"></el-input>
                </el-form-item>
              </template>
              <el-form-item label="规格编码" prop="goodsSn">
                <el-input v-model="formSpec.goodsSn"></el-input>
              </el-form-item>
              <el-form-item label="库存" prop="specQuantity">
                <el-input-number v-model.number="formSpec.goodsNumber" controls-position="right" :min="1" :max="100000"></el-input-number>
              </el-form-item>
              <el-form-item label="价格" prop="specPrice">
                <el-input-number v-model.number="formSpec.retailPrice"></el-input-number>
              </el-form-item>
              <el-form-item label="规格图" class="no-line-height" prop="listPicUrl">
                <croppa v-model="productCroppa"
                  :width="200"
                  :height="200"
                  canvas-color="default"
                  placeholder="点击选择, PNG/JPEG, <500K"
                  :placeholder-font-size="12"
                  placeholder-color="default"
                  accept="image/png,image/jpeg"
                  :file-size-limit="512000"
                  :quality="4"
                  :prevent-white-space="true"
                  :initial-image="formSpec.listPicUrl"
                  @file-size-exceed="onFileSizeExceed"
                  @file-type-mismatch="onFileTypeMismatch"
                  @image-remove="onImageRemove(formSpec.listPicUrl, 'product')">
                </croppa>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="saveCombinations" :loading="loading2">保存</el-button>
                <router-link :to="{ path: '/products' }">
                  <el-button>取消</el-button>
                </router-link>
              </el-form-item>
            </el-form>
          </el-col>
          <el-col :span="14" :xs="24" :sm="24" :md="14" :lg="14">
            <el-table :data="goodsSpecs" stripe border v-loading="tbloading">
              <el-table-column type="index" width="50" align="center"></el-table-column>
              <el-table-column label="规格" min-width="80" show-overflow-tooltip>
                <template slot-scope="scope">
                  <span v-if="scope.row.specs.length > 0" v-for="item in scope.row.specs" :key="item.id">
                    <strong>{{ item.name }}</strong>:&nbsp;{{ item.value }}<br/>
                  </span>
                  <span v-else>无</span>
                </template>
              </el-table-column>
              <el-table-column prop="goodsSn" label="规格编码" width="90" align="right" show-overflow-tooltip></el-table-column>
              <el-table-column prop="goodsNumber" label="库存" width="60" align="right" show-overflow-tooltip></el-table-column>
              <el-table-column prop="retailPrice" label="单价" width="60" align="right" show-overflow-tooltip></el-table-column>
              <el-table-column prop="picUrl" label="规格图" width="200" align="right">
                <template slot-scope="scope">
                  <el-image
                    style="width: 100px; height: 100px"
                    :src="scope.row.picUrl"
                    :fit="fit"></el-image>
                </template>
              </el-table-column>
              <el-table-column label="操作" width="200">
                <template slot-scope="scope">
                  <el-button size="mini" icon="el-icon-edit" @click="editProduct(scope.row)">编辑</el-button>
                  <el-button type="danger" size="mini" icon="el-icon-delete" @click="destroy(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-col>
        </el-row>
      </el-tab-pane>
    </el-tabs>
    <el-dialog title="新增规格类型" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="addSpec" label-width="80px" size="mini" ref="specForm">
        <el-form-item label="规格类型" prop="name" :rules="{required: true, message: '请填写规格类型名称'}">
          <el-input v-model="addSpec.name"></el-input>
        </el-form-item>
        <el-form-item label="排序">
          <el-input-number v-model="addSpec.sortOrder" :min="1" :max="1000"></el-input-number>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="mini" @click="dialogFormVisible = false">取 消</el-button>
        <el-button size="mini" type="primary" @click="saveSpec">确 定</el-button>
      </div>
    </el-dialog>
  </main-panel>
</template>

<script>
import * as api from 'api'
import * as utils from 'utils'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import {quillEditor} from 'vue-quill-editor'
import Quill from 'quill'
import ImageResize from 'quill-image-resize-module'
// 需要配置一下webpack https://github.com/kensnyder/quill-image-resize-module/issues/7
Quill.register('modules/imageResize', ImageResize)
Quill.debug(false)

export default {
  components: {
    quillEditor
  },
  data () {
    let that = this
    let coverValidate = function (rule, value, callback) {
      if (!that.coverCroppa.hasImage()) {
        callback(new Error('请选择商品封面图'))
      }
      callback()
    }
    let catValidate = function (rule, value, callback) {
      debugger
      if (value.length === 0) {
        callback(new Error('请选择商品分类'))
      } else if (value.length > 1) {
        callback(new Error('暂时只支持单个分类'))
      }
      callback()
    }
    let specValidate = function (rule, value, callback) {
      if (!value) {
        callback(new Error('请选择需要添加的规格类型'))
      }

      let count = 0
      that.formSpec.specArr.forEach(item => {
        if (item.selectedSpecs && item.selectedSpecs === value) {
          count++
        }
      })
      if (count >= 2) {
        callback(new Error('不能组合相同的规格类型'))
      }
      callback()
    }
    let imageValidate = function (rule, value, callback) {
      if (!that.imgCroppa1.hasImage() && !that.imgCroppa2.hasImage() && !that.imgCroppa3.hasImage() && !that.imgCroppa4.hasImage()) {
        callback(new Error('请选择商品展示图'))
      }
      callback()
    }
    return {
      id: null,
      pageLoading: false,
      dialogFormVisible: false,
      addSpec: {
        name: '',
        sort_order: 50
      },
      specValidate: specValidate,
      form: {
        primaryProduct: {goodsSn: '', goodsNumber: 1},
        goodsSn: '',
        name: '',
        attrs: [{name: '', value: ''}],
        faq: [{question: '', answer: ''}],
        goodsBrief: '',
        categoryId: null,
        brandId: null,
        categories: [],
        goodsDesc: '',
        goodsNumber: null,
        retailPrice: null,
        isOnSale: 1,
        isNew: 0,
        needExpress: 1,
        listPicUrl: '',
        sortOrder: 100,
        gallery: []
      },
      formSpec: {
        specArr: [{
          selectedSpecs: null,
          selectedSpecVal: null
        }],
        goodsSn: '',
        goodsNumber: '',
        retailPrice: '',
        listPicUrl: ''
      },
      initImgs: {
        cover: '',
        gallery: []
      },
      coverChanged: false,
      productChanged: true,
      img1Changed: false,
      img2Changed: false,
      img3Changed: false,
      img4Changed: false,
      productCroppa: null,
      activeName: 'info',
      specs: [],
      goodsSpecs: [],
      brands: [],
      categories: [],
      removedImgs: [], // 用于图片删除，这里不删除七牛图片
      props: {
        label: 'name',
        children: 'children'
      },
      coverCroppa: null,
      imgCroppa1: null,
      imgCroppa2: null,
      imgCroppa3: null,
      imgCroppa4: null,
      loading: false,
      loading2: false,
      tbloading: false,
      rules: {
        'goodsSn': {required: true, message: '请输入商品SKU'},
        name: {required: true, message: '请输入商品名称'},
        categories: {required: true, validator: catValidate},
        'goodsNumber': [
          {required: true, message: '请填写商品库存'},
          {type: 'number', message: '库存必须为数字值'}
        ],
        retailPrice: [
          {required: true, message: '请填写商品价格'},
          {type: 'number', message: '价格必须为数字值'}
        ]
      },
      rulesAttr: {
        goodsNumber: [
          {required: true, message: '请填写商品库存'},
          {type: 'number', message: '库存必须为数字值'}
        ],
        retailPrice: [
          {required: true, message: '请填写商品价格'},
          {type: 'number', message: '价格必须为数字值'}
        ]
      },
      editorOption: {
        placeholder: '在这里编辑商品详情',
        theme: 'snow',
        debug: false,
        modules: {
          toolbar: {
            handlers: {
              image: this.handlerImg
            },
            container: [
              ['bold', 'italic', 'underline', 'strike'],
              ['blockquote', 'code-block'],
              [{'header': 1}, {'header': 2}],
              [{'list': 'ordered'}, {'list': 'bullet'}],
              [{'script': 'sub'}, {'script': 'super'}],
              [{'indent': '-1'}, {'indent': '+1'}],
              [{'direction': 'rtl'}],
              [{'size': ['small', false, 'large', 'huge']}],
              [{'header': [1, 2, 3, 4, 5, 6, false]}],
              [{'color': []}, {'background': []}],
              [{'font': []}],
              [{'align': []}],
              ['clean'],
              ['link', 'image']
            ]
          },
          imageResize: {
            modules: [ 'Resize', 'DisplaySize', 'Toolbar' ]
          }
        }
      }
    }
  },
  mounted () {
    this.id = this.$route.params.id
    this.getProductsInfo()
    this.getGoodsSpecs()
    this.getAllCats()
    this.getAllBrands()
    this.getAllSpecs()
  },
  methods: {
    getAllCats () {
      this.$http.get(api.CATEGORY + '/index?type=0').then(data => {
        this.categories = data.data
      })
    },
    getAllBrands () {
      this.$http.get(api.BRAND + '/index?size=10000&page=1').then(data => {
        this.brands = data.data.list
      })
    },
    getAllSpecs () {
      this.$http.get(api.SPECIFICATION + '/index?size=10000&page=1').then(data => {
        this.specs = data.data.list
      })
    },
    getGoodsSpecs () {
      this.tbloading = true
      this.$http.get(api.SPECIFICATION + '/goods?goodsId=' + this.id).then(data => {
        this.tbloading = false
        this.goodsSpecs = data.data
        this.goodsSpecs.forEach((item, index) => {
          item.picUrl = /^http/i.test(item.picUrl) ? item.picUrl : api.QiniuDomain + item.picUrl
        })
      }).catch(() => {
        this.tbloading = false
      })
    },
    getProductsInfo () {
      this.pageLoading = true
      this.$http.get(api.GOODS + '/info?id=' + this.id).then(data => {
        this.form = data.data
        if (this.form.attrs.length === 0) {
          this.form.attrs.push({name: '', value: ''})
        }
        this.form.categories = [this.form.categoryId]
        this.form.retailPrice = Number(this.form.retailPrice)
        this.$refs.catTree.setCheckedKeys([this.form.categoryId])
        if (this.form.listPicUrl) {
          this.initImgs.cover = /^http/i.test(this.form.listPicUrl) ? this.form.listPicUrl : api.QiniuDomain + this.form.listPicUrl
          this.coverChanged = false
        }
        if (data.data.goodsGalleryList.length > 0) {
          this.initImgs.gallery = data.data.goodsGalleryList.map((item, index) => {
              this[`img${index + 1}Changed`] = false
              return /^http/i.test(item.imgUrl) ? item.imgUrl : api.QiniuDomain + item.imgUrl
          })
        } else {
          this.img1Changed = true,
          this.img2Changed = true,
          this.img3Changed = true,this.img4Changed = true
        }
        this.form.gallery = this.initImgs.gallery
        
        this.coverCroppa.refresh()
        this.imgCroppa1.refresh()
        this.imgCroppa2.refresh()
        this.imgCroppa3.refresh()
        this.imgCroppa4.refresh()

        this.pageLoading = false
      }).catch(err => {
        this.$message.error(err + '')
        this.pageLoading = false
      })
    },
    save () {
      this.$refs.form.validate(async (valid) => {
        debugger
        if (valid) {
          debugger
          this.loading = true
          if (this.coverCroppa.hasImage() && this.coverChanged) {
            const cover = await this.coverCroppa.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(cover, api.GoodsImgPrefix)
            this.form.listPicUrl = res.key
          }
          if (this.imgCroppa1.hasImage() && this.img1Changed) {
            const img1 = await this.imgCroppa1.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(img1, api.GoodsImgPrefix)
            this.form.gallery[0] = res.key
          }
          if (this.imgCroppa2.hasImage() && this.img2Changed) {
            const img2 = await this.imgCroppa2.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(img2, api.GoodsImgPrefix)
            this.form.gallery[1] = res.key
          }
          if (this.imgCroppa3.hasImage() && this.img3Changed) {
            const img3 = await this.imgCroppa3.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(img3, api.GoodsImgPrefix)
            this.form.gallery[2] = res.key
          }
          if (this.imgCroppa4.hasImage() && this.img4Changed) {
            const img4 = await this.imgCroppa4.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(img4, api.GoodsImgPrefix)
            this.form.gallery[3] = res.key
          }
          this.form.categoryId = this.form.categories[0]
          // thinkjs不支持直接传递数组，改为json格式提交
          this.$http.post(api.GOODS + '/store', this.form, {headers: {'Content-Type': 'application/json'}}).then((data) => {
            this.loading = false
            if (data.code === 0) {
              this.$notify({title: '成功', message: '保存成功', type: 'success'})
            }
            // this.$router.push({ path: '/products' })
          }).catch(err => {
            this.loading = false
            this.$message.error(err + '')
          })
        }
      })
    },
    onSelectCat (data, checked, indeterminate) {
      if (checked) {
        this.form.categories = [data.id]
      } else {
        this.form.categories = []
      }
      debugger
    },
    onFileSizeExceed (file) {
      this.$message.error('图片大小不能超过 500K')
    },
    onFileTypeMismatch (file) {
      this.$message.error('图片只能是 JPG/PNG 格式！')
    },
    onImageRemove (path, type) {
      switch (type) {
        case 'cover':
          this.coverChanged = true
          break
        case 'product':
          this.productChanged = true
          break
        case 'img1':
          this.img1Changed = true
          break
        case 'img2':
          this.img2Changed = true
          break
        case 'img3':
          this.img3Changed = true
          break
        case 'img4':
          this.img4Changed = true
          break
      }
    },
    handlerImg () {
      var that = this
      var fileInput = this.$refs.editor.quill.container.querySelector('input.ql-image[type=file]')
      if (fileInput === null) {
        fileInput = document.createElement('input')
        fileInput.setAttribute('type', 'file')
        fileInput.setAttribute('name', 'image')
        fileInput.setAttribute('accept', 'image/png,image/jpeg,image/gif')
        fileInput.classList.add('ql-image')
        // 监听选择文件
        fileInput.addEventListener('change', function () {
          if (fileInput.files[0].size >= 500 * 1024) {
            fileInput.value = ''
            that.$message.error('图片大小超出500 K限制')
            return
          }
          utils.qupload(fileInput.files[0], api.GoodsDescPrefix).then(res => {
            let length = that.$refs.editor.quill.getSelection(true).index
            that.$refs.editor.quill.insertEmbed(length, 'image', api.QiniuDomain + res.key)
            that.$refs.editor.quill.setSelection(length + 1)
          }).catch(err => {
            this.$message.error(err + '')
          })
        })
      }
      fileInput.click()
    },
    add () {
      this.formSpec.specArr.push({
        selectedSpecs: null,
        selectedSpecVal: null
      })
    },
    remove (index) {
      this.formSpec.specArr.splice(index, 1)
    },
    destroy (row) {
      let attr = []
      row.specs.forEach(item => {
        attr.push(item.value)
      })
      this.$confirm(`确定删除商品属性“${attr.join('|')}”吗?`, '提示', {type: 'warning'}).then(() => {
        this.$http.post(api.SPECIFICATION + '/destroy', {id: row.id}, {headers: {'Content-Type': 'application/json'}}).then((data) => {
          if (data.code === 0) {
            this.$notify({title: '成功', message: '删除成功', type: 'success'})
            this.getGoodsSpecs()
          }
        })
      })
    },
    editProduct (row) {
      this.$http.get(api.SPECIFICATION + '/product?id=' + row.id).then((data) => {
        if (data.code === 0) {
            this.formSpec = data.data
            this.formSpec.listPicUrl = /^http/i.test(this.formSpec.listPicUrl) ? this.formSpec.listPicUrl : api.QiniuDomain + this.formSpec.listPicUrl
            this.productCroppa.refresh()
            this.productChanged = false
          }
        })
    },
    attrDestroy (index) {
      this.form.attrs.splice(index, 1)
    },
    attrAdd (index) {
      this.form.attrs.splice(index + 1, 0, {name: '', value: ''})
    },
    faqDestroy (index) {
      this.form.faq.splice(index, 1)
    },
    faqAdd (index) {
      this.form.faq.splice(index + 1, 0, {question: '', answer: ''})
    },
    newSpec () {
      this.addSpec.name = ''
      this.addSpec.sort_order = 50
      this.dialogFormVisible = true
    },
    saveSpec () {
      this.$refs.specForm.validate(valid => {
        if (valid) {
          this.$http.post(api.SPECIFICATION + '/store', this.addSpec).then((data) => {
            if (data.code === 0) {
              this.$notify({title: '成功', message: '添加成功', type: 'success'})
              this.dialogFormVisible = false
              this.getAllSpecs()
            }
          }).catch(err => {
            this.loading2 = false
            this.$message.error(err + '')
          })
        }
      })
    },
    saveCombinations () {
      this.$refs.combinationForm.validate(async (valid) => {
        if (valid) {
          this.loading2 = true
          this.formSpec.goodsId = this.id
          if (this.productCroppa.hasImage() && this.productChanged) {
            const cover = await this.productCroppa.promisedBlob('image/jpeg', 1)
            const res = await utils.qupload(cover, api.GoodsImgPrefix)
            this.formSpec.listPicUrl = res.key
          }
          this.$http.post(api.SPECIFICATION + '/save', this.formSpec).then((data) => {
            if (data.code === 0) {
              this.formSpec = {
                specArr: [{
                  selectedSpecs: null,
                  selectedSpecVal: null
                }],
                goodsSn: '',
                goodsNumber: '',
                retailPrice: '',
                listPicUrl: ''
              }
              this.$notify({title: '成功', message: '保存成功', type: 'success'})
              this.productCroppa.refresh()
              this.getGoodsSpecs()
            }
            this.loading2 = false
          }).catch(err => {
            this.loading2 = false
            this.$message.error(err + '')
          })
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
