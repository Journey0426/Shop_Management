<template>
  <el-form label-width="80px" :model="spulist">
    <el-form-item label="SPU名称" >
      <el-input placeholder="SPU名称"
                v-model="spulist.spuName"
      ></el-input>
    </el-form-item>
    <el-form-item label="品牌">
      <el-select placeholder="请选择品牌" v-model="spulist.tmId">
        <el-option :label="tm.tmName" :value="tm.id"
        v-for="tm in trademarkList":key="tm.id"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="SPU描述" >
      <el-input
        type="textarea"
        :autosize="{ minRows: 2, maxRows: 4}"
        v-model="spulist.description"
        placeholder="请输入内容"
      >
      </el-input>
    </el-form-item>
    <el-form-item label="SPU图片">
      <!-- 上传图片：action图片上传的地址  list-type: 文件列表的类型 on-preview:图片预览的时候会出发  on-remove:当删除图片的时候会出发
        file-list：照片墙需要展示的数据【数组：数组里面的元素务必要有name、url属性】
        on-preview：图片预览功能
        on-remove:删除图片的时候会触发
       -->
      <el-upload
        action="/dev-api/admin/product/fileUpload"
        list-type="picture-card"
       :file-list="spuImageList"
        :on-preview="handlePictureCardPreview"
        :on-remove="handleRemove"
        :on-success="handleSuccess"
      >
        <i class="el-icon-plus"></i>
      </el-upload>
      <el-dialog :visible.sync="dialogVisible">
        <img width="100%" :src="dialogImageUrl" alt="">
      </el-dialog>
    </el-form-item>
    <el-form-item label="销售属性">
      <el-select :placeholder="`还有${unSelectSaleAttr.length}项未选择`" v-model="attrIdAndAttrName">
        <el-option :label="unselect.name" :value="`${unselect.id}:${unselect.name}`" v-for="unselect in unSelectSaleAttr":key="unSelectSaleAttr.id"></el-option>
      </el-select>
      <el-button icon="el-icon-plus" type="primary" plain :disabled="!attrIdAndAttrName" @click="addattrlist">添加销售属性</el-button>
      <el-table
        border
        style="width: 100%"
        :data="spulist.spuSaleAttrList"
      >
        <el-table-column
          type="index"
          label="序号"
          width="80"
        >
        </el-table-column>
        <el-table-column
          prop="saleAttrName"
          label="属性名"
          width="180"
        >
        </el-table-column>
        <el-table-column
          prop="prop"
          label="属性值名称列表"
          width="width"
        >

          <template v-slot="{row,$index}">
          <el-tag :key="tag.id" v-for="(tag,index) in row.spuSaleAttrValueList" closable :disable-transitions="false" @close="row.spuSaleAttrValueList.splice(index,1)">
            {{tag.saleAttrValueName}}
          </el-tag>
          <el-input
            class="input-new-tag"
            v-if="row.inputVisible"
            v-model="row.inputValue"
            ref="saveTagInput"
            size="small"
            @keyup.enter.native="handleInputConfirm(row)"
           @blur="handleInputConfirm(row)"
         >
         </el-input>
         <el-button v-else class="button-new-tag" size="small"  @click="addSaleAttrValue(row)">添加</el-button>
         </template>
       </el-table-column>
       <el-table-column
         prop="prop"
         label="操作"
         width="180"
       >
         <template slot-scope="{ row, $index }">
         <el-button type="danger" icon="el-icon-delete" size="small" @click="spulist.spuSaleAttrList.splice($index,1)"></el-button>
         </template>
       </el-table-column>
     </el-table>
   </el-form-item>
   <el-form-item>
     <el-button type="primary" @click="addOrUpdateSpu">保存</el-button>
     <el-button @click="cancelsup">取消</el-button>
   </el-form-item>
 </el-form>
</template>

<script>
import { reqBaseSaleAttrList, reqTradeMarkList } from '@/api/product/spu'

export default {
 name: 'SPUform',
 data() {
   return {
     dialogImageUrl: '',
     dialogVisible: false,
     //spu数据
     //spu属性初始化的时候是一个空的对象,在修改SPU的时候，会想服务器发请求，返回SPU信息（对象），在修改的时候可以利用服务器返回的这个对象收集最新的数据提交给服务器
     //添加SPU，如果是添加SPU的时候并没有向服务器发请求，数据收集到哪里呀[SPU]，收集数据的时候有哪些字段呀，看文档
     spulist: {
       //三级分类的id
       category3Id: 0,
       //描述
       description: "",
       //spu名称
       spuName: "",
       //平台的id
       tmId: "",
       //收集SPU图片的信息
       spuImageList: [
         // {
         //   id: 0,
         //   imgName: "",
         //   imgUrl: "",
         //   spuId: 0,
         // },
       ],
       //平台属性与属性值收集
       spuSaleAttrList: [
         // {
         //   baseSaleAttrId: 0,
         //   id: 0,
         //   saleAttrName: "string",
         //   spuId: 0,
         //   spuSaleAttrValueList: [
         //     {
         //       baseSaleAttrId: 0,
         //       id: 0,
         //       isChecked: "string",
         //       saleAttrName: "string",
         //       saleAttrValueName: "string",
         //       spuId: 0,
         //     },
         //   ],
         // },
       ],
     },
     //tradrmark数据
     trademarkList: [],
     //图片数据
     spuImageList: [],
     //平台全部销售属性
     basesaleattrList:[],
     //手机未选择属性的id和name
     attrIdAndAttrName: "", //收集未选择的销售属性的id和name-----
   }
 },
 methods: {
   //file参数:代表的是删除的那个张图片
   //fileList:照片墙删除某一张图片以后，剩余的其他的图片
   // console.log(file, fileList,22222);
   //收集照片墙图片的数据
   //对于已有的图片【照片钱中显示的图片：有name、url字段的】，因为照片墙显示数据务必要有这两个属性
   //对于服务器而言，不需要name、url字段，将来对于有的图片的数据在提交给服务器的时候，需要处理的
   //照片墙删除某一个图片的时候会触发
   handleRemove(file, fileList) {
     //file参数:代表的是删除的那个张图片
     //fileList:照片墙删除某一张图片以后，剩余的其他的图片
     // console.log(file, fileList,22222);
     //收集照片墙图片的数据
     //对于已有的图片【照片钱中显示的图片：有name、url字段的】，因为照片墙显示数据务必要有这两个属性
     //对于服务器而言，不需要name、url字段，将来对于有的图片的数据在提交给服务器的时候，需要处理的
     this.spuImageList= fileList;
     console.log(111)
   },
   //照片墙图片预览的回调
   handlePictureCardPreview(file) {
     //将图片地址赋值给这个属性
     this.dialogImageUrl = file.url
     //对话框显示
     this.dialogVisible = true
   },
   //照片墙成功回调
   handleSuccess(response, file, fileList){
     //收集图片的信息
   this.spuImageList=fileList
   },
   //初始化spuform数据 并接受数据
   async initSpudate(spu) {
     //获取spu信息
     let spuinfo = await this.$API.spu.reqSpu(spu.id)
     if (spuinfo.code == 200) {
       this.spulist = spuinfo.data
     }
     //获取trademark信息
     let trademarklist = await this.$API.spu.reqTradeMarkList()
     if (trademarklist.code == 200) {
       this.trademarkList = trademarklist.data
     }
     ////获取SPU图标的接口
     let spuImageList = await this.$API.spu.reqSpuImageList(spu.id)
     if (spuImageList.code == 200) {
       let imaglist = spuImageList.data
//由于照片墙显示图片的数据需要数组，数组里面的元素需要有name与url字段
       //需要把服务器返回的数据进行修改
       imaglist.forEach((item)=>{
         //给列表新增name和url属性
         item.name = item.imgName;
         item.url = item.imgUrl;
       })
       //将新增数据的图片列表赋值给data中的图片列表
       this.spuImageList=imaglist
     }
     //获取平台全部销售属性----整个平台销售属性一共三个
     let basesaleattrlist = await this.$API.spu.reqBaseSaleAttrList()
     if (basesaleattrlist.code == 200) {
       this.basesaleattrList=basesaleattrlist.data
     }
   },
   //增加选择属性
   addattrlist(){
     //已经收集需要添加的销售属性的信息
     //把收集到的销售属性数据进行分割
     const [baseSaleAttrId, saleAttrName]= this.attrIdAndAttrName.split(":")
     //向SPU对象的spuSaleAttrList属性里面添加新的销售属性
     let newSaleAttr = {baseSaleAttrId,saleAttrName, spuSaleAttrValueList: []}
     this.spulist.spuSaleAttrList.push(newSaleAttr)
     //清空数据
     this.attrIdAndAttrName=''
   },
   //添加属性值
   addSaleAttrValue(row){
     //点击销售属性值当中添加按钮的时候，需要有button变为input,通过当前销售属性的inputVisible控制
     //挂载在销售属性身上的响应式数据inputVisible，控制button与input切换
     //第一个参数:对象  第二个参数:添加新的响应式属性  第三参数：新的属性的属性值
     //属性状态
     this.$set(row,'inputVisible',true)
     //属性值
     this.$set(row,'inputValue','')
   },
   //失焦
   handleInputConfirm(row){
     //解构出销售属性当中收集数据
     const {baseSaleAttrId,inputValue}=row
     if (inputValue.trim()==''){
       this.$message('输入数据不能为空')
       return
     }
     //属性值不能重复,这里也可以用some
    let result=row.spuSaleAttrValueList.every(
      (item)=>item.saleAttrValueName!=inputValue
     )
     if (!result)return
let newSaleAttrValue={baseSaleAttrId,saleAttrValueName:inputValue}
       row.spuSaleAttrValueList.push(newSaleAttrValue)
     //修改inputVisible为false，不就显示button
     row.inputVisible=false
   },
   //更新或新增
   /*
   *  map()  可以将数组字符串转变为数字 也可将数字转换为字符串 可以map(iteam,v)来去重 iteam是当前元素
                    - 根据当前数组生成一个新数组
                    - 需要一个回调函数作为参数，
                        回调函数的返回值会成为新数组中的元素
                    - 非破坏性方法不会影响原数组*/
  async addOrUpdateSpu(){
     //整理参数：需要整理照片墙的数据
     //携带参数：对于图片，需要携带imageName与imageUrl字段
     this.spulist.spuImageList=this.spuImageList.map((item)=>{
       return{
         imgName:item.name,
         //新增的有response那里面的url才是需要的
         imgUrl:(item.response&&item.response.url)||item.url
       }
     })
     //发请求
  let result= await this.$API.spu.reqSaveSpuinfoAndUpdatesupinfo(this.spulist)
     if (result.code==200){
       //提示
       this.$message({ type: "success", message: "保存成功" });
       this.$emit('canselSpu',{
         contrl:0,
         flag:this.spulist.id?'修改':'添加'
       })
     }
    //清除数据
    Object.assign(this._data,this.$options.data())
   },
   //添加属性事件
   //点击添加SPU按钮的时候，发请求的函数
   async addSpuData(category3Id){
     //添加SPU的时候收集三级分类的id
     this.spulist.category3Id=category3Id
     //获取trademark信息
     let trademarklist = await this.$API.spu.reqTradeMarkList()
     if (trademarklist.code == 200) {
       this.trademarkList = trademarklist.data
     }
     //获取平台全部销售属性----整个平台销售属性一共三个
     let basesaleattrlist = await this.$API.spu.reqBaseSaleAttrList()
     if (basesaleattrlist.code == 200) {
       this.basesaleattrList=basesaleattrlist.data
     }
   },
   //取消按钮
   cancelsup(){
     this.$emit('canselSpu',  {contrl:0, flag:''})
     //清理数据
     //Object.assign:es6中新增的方法可以合并对象
     //组件实例this._data,可以操作data当中响应式数据
     //this.$options可以获取配置对象，配置对象的data函数执行，返回的响应式数据为空的
     Object.assign(this._data, this.$options.data());
   }
 },
 computed: {
   //计算出还未选择的销售属性
   unSelectSaleAttr() {
     //整个平台的销售属性一共三个：尺寸、颜色、版本 ----saleAttrList
     //当前SPU拥有的属于自己的销售属性SPU.spuSaleAttrList  ----颜色
     //数组的过滤方法，可以从已有的数组当中过滤出用户需要的元素，并未返回一个新的数据
     let result = this.basesaleattrList.filter(item => {
       //every数组的方法，会返回一个布尔值【真，假的】
       return this.spulist.spuSaleAttrList.every(items => {
         return item.name != items.saleAttrName
       })
     })
     return result
   }
 }
}
</script>

<style>
.el-tag + .el-tag {
 margin-left: 10px;
}
.button-new-tag {
 margin-left: 10px;
 height: 32px;
 line-height: 30px;
 padding-top: 0;
 padding-bottom: 0;
}
.input-new-tag {
 width: 90px;
 margin-left: 10px;
 vertical-align: bottom;
}
</style>
