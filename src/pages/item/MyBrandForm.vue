<template>
  <v-form v-model="valid" ref="myBrandForm">
    <v-text-field
      v-model="brand.name"
      label="Fill in the brand name"
      required
      :rules="[v => !!v || 'Brand name cannot be empty',v => v.length > 1 || 'at least two characters']"
    />
    <v-text-field
      v-model="brand.letter"
      label="Fill in the initial letter of brand name"
      required
      :rules="[v => !!v || 'Initial letter cannot be empty',v => /^[A-Z]{1}$/.test(v) || 'only from A to Z in upper case']"
    />
    <v-cascader
      url="/item/category/list"
      multiple required
      v-model="brand.categories"
      label="Choose a category"
    />
    <v-layout row>
      <v-flex xs4 class="subheader">
        Brand LOGO：
      </v-flex>
      <v-flex>
        <!--第一处修改：/upload修改为/upload/image-->
        <v-upload
          v-model="brand.image"
          url="/upload/image"
          :multiple="false"
          :pic-width="250"
          :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">Submit</v-btn>
      <v-btn @click="clear" >Reset</v-btn>
    </v-layout>
  </v-form>

</template>

<script>
  export default {
    name: "my-brand-form",
    props: {
      oldBrand:{type:Object},
      isEdit: {type: Boolean, default: false},
    },
    data(){
      return{
        valid:false,
        brand:{
          id:0,
          name:"",
          letter:"",
          image:"",
          categories:[],
        },
        imageDialogVisible:false,
      }
    },
    watch:{
      oldBrand:{
        deep:true,
        handler(val){
          if(val){
            this.brand=Object.deepCopy(val);
          }else{
            this.clear();
          }
        }
      }
    },

    methods:{
      // 提交表单
      submit(){
          // 1、表单校验
        if (this.$refs.myBrandForm.validate()) {
          // 2、定义一个请求参数对象，通过解构表达式来获取brand中的属性
          this.brand.id=this.oldBrand.id;
          const {categories, letter, ...rest} = this.brand;
          // 3、数据库中只要保存分类的id即可，因此我们对categories的值进行处理,只保留id，并转为字符串
          rest.categories = categories.map(c => c.id).join(",");
          // 4、将字母都处理为大写
          rest.letter = letter.toUpperCase();
          this.verify().then(() => {
            this.$http({
              method:this.isEdit ? 'put' :'post',
              url:"/item/brand",
              data:this.$qs.stringify(rest),
            }).then(
              () =>{
                //关闭对话框
                this.$emit('reload');
                this.$message.success("Saved！");
                this.clear();
              }
            ).catch(
              ()=>{
                this.$message.success("Failed！");
              }
            );
          }).catch(() => {
            this.$router.push("/login");
          });
        }
      },
      clear(){
        // 重置表单
        this.$refs.myBrandForm.reset();
        // 需要手动清空商品分类
        this.categories = [];
        this.oldBrand=null;
      },
      // 图片上传出成功后操作
      handleImageSuccess(res) {
        this.brand.image = res;
      },
      removeImage(){
        this.brand.image = "";
      },
      closeWindow(){
        this.$emit("close");
      }
    },
  }
</script>

<style scoped>

</style>
