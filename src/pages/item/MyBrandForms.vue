<template>
  <v-card>
    <!--对话框的标题-->
    <v-toolbar dense dark color="primary">
      <v-toolbar-title>{{isEdit?'修改':'新增'}}品牌</v-toolbar-title>
      <v-spacer/>
      <!--关闭窗口的按钮-->
      <v-btn icon @click="closeWindows"><v-icon>close</v-icon></v-btn>
    </v-toolbar>
    <v-form v-model="valid" ref="myBrandForm">
      <v-text-field v-model="brand.name" label="请输入品牌名称" :rules="nameRules" required />
      <v-text-field v-model="brand.letter" label="请输入品牌首字母" :rules="letterRules" required />
      <v-cascader
        url="/item/category/list"
        multiple
        required
        v-model="brand.categories"
        label="请选择商品分类"/>
      <v-layout row>
        <v-flex xs3>
          <span style="font-size: 16px; color: #444">品牌LOGO：</span>
        </v-flex>
        <v-flex>
          <v-upload
            v-model="brand.image"
            url="/upload/image"
            :multiple="false"
            :pic-width="250"
            :pic-height="90"
          />
        </v-flex>
      </v-layout>
    </v-form>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn color="blue darken-1" flat @click="clear">重置</v-btn>
      <v-btn color="blue darken-1" flat @click="submit">提交</v-btn>
    </v-card-actions>
  </v-card>
</template>
<script>
    export default {
        name: "my-brand-form",
        props:{
            oldBrand:{
                type:Object
            },
            isEdit:{
                type:Boolean,
                default:false
            }
        },
        create(){

        },
        data() {
            return {
                valid: false, // 表单校验结果标记
                nameRules:[
                    v => !!v || "品牌名称不能为空",
                    v => v.length > 1 || "品牌名称至少2位"
                ],
                letterRules:[
                    v => !!v || "首字母不能为空",
                    v => /^[A-Z]{1}$/.test(v) || "品牌字母只能是A~Z的大写字母"
                ],
                brand: {
                    name: '', // 品牌名称
                    letter: '', // 品牌首字母
                    image: '',// 品牌logo
                    categories: [], // 品牌所属的商品分类数组
                }
            }
        },
        methods:{
            created(){
                 console.log(this.$qs);
            },
            submit(){
                // 提交表单
                if(this.$refs.myBrandForm.validate()){
                    const {categories,letter,...params}=this.brand;
                    params.cids = categories.map(c=>c.id).join(",");
                    params.letter = letter.toUpperCase();
                    //this.$http.post("/item/brand",this.$qs.stringify(params))
                    this.$http({
                        method: this.isEdit ? 'put' : 'post', // 动态判断是POST还是PUT
                        url: '/item/brand',
                        data: this.$qs.stringify(this.isEdit ? this.brand : params)
                    }).then(()=>{
                        this.$message.info("保存成功");
                    }).catch(()=>{
                        this.$message.error("保存失败");
                    });
                    this.$emit("close");
                }
            },
            closeWindows(){
                this.$emit("close");
            },
            clear(){
                // 重置表单
                this.$refs.myBrandForm.reset();
                // 需要手动清空商品分类
                this.categories = [];
            },

        },
        watch:{
            oldBrand:{
                handler(val){
                    if(val){
                        this.brand = Object.deepCopy(val);
                        console.info(this.brand);
                    }else{
                        // 为空，初始化brand
                        this.brand = {
                            name: '',
                            letter: '',
                            image: '',
                            categories: [],
                        }
                    }
                }
            },
            deep: true
        }
    }
</script>
