<template>
    <v-card>
      <v-card-title flat color="white">
        <v-dialog v-model="show" persistent max-width="500px">
          <v-btn slot="activator" @click="addBrand" color="primary" dark>新增</v-btn>
          <my-brand-form @close="closeWindows" :oldBrand="oldBrand" :isEdit="isEdit"></my-brand-form>
        </v-dialog>
        <!--空间隔离组件-->
        <v-spacer />
        <v-flex xs6 sm3>
          <v-text-field
            name="input-1-3"
            label="请输入要搜索的内容"
            single-line
            append-icon="search" v-model="search" hide-details
          ></v-text-field>
        </v-flex>
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="brands"
        :pagination.sync="pagination"
        :loading="loading"
        :total-items="totalBrands"
        class="elevation-1"
      >
      <template slot="items" slot-scope="props">
        <td>{{ props.item.id }}</td>
        <td class="text-xs-right">{{ props.item.name }}</td>
        <td class="text-xs-center"><img v-if="props.item.image" :src="props.item.image" width="130" height="40"/></td>
        <td class="text-xs-right">{{ props.item.letter }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon class="mx-0" @click="editItem(props.item)">
            <v-icon color="teal">edit</v-icon>
          </v-btn>
          <v-btn icon class="mx-0" @click="deleteItem(props.item)">
            <v-icon color="pink">delete</v-icon>
          </v-btn>
        </td>
      </template>
    </v-data-table>
    </v-card>
</template>
<script>
  import MyBrandForm from "./MyBrandForms";
  export default {
      components:{
          MyBrandForm
      },
    name:"mybrand",
    data () {
      return {
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        search:"",
        show:false,//控制对话框的显示
        headers: [ // 头信息
          {text: 'id', align: 'center', value: 'id'},
          {text: '名称', align: 'center', value: 'name', sortable: false},
          {text: 'LOGO', align: 'center', value: 'image', sortable: false},
          {text: '首字母', align: 'center', value: 'letter'},
          {text: '操作', align: 'center', value: 'letter'},
        ],
        oldBrand:{},//即将被编辑的品牌数
        isEdit:false
      }
    },
    methods: {
      addBrand(){
        // 控制弹窗可见：
        this.show = true;
        // this.isEdit = false;
        // this.oldBrand = null;
      },
      closeWindows(){
        // 控制弹窗可见：
        this.show = false;
          // 重新加载数据
          this.getDataFromServer();
      },
      editItem(data){
        this.$http.get("/item/category/byBid",{params:{bid:data.id}}).then(resp=>{
          this.show = true;
          this.oldBrand = data;
          this.oldBrand.categories = resp.data;
          this.isEdit = true;
        });
      },
      deleteItem(data){
          this.$http.get("/item/brand/delete",{params:{bid:data.id}}).then(resp=>{
              this.$message.error("删除成功!");
              this.getDataFromServer();
          }).catch(()=>{
              this.$message.error("删除失败!");
          });

      },
      getDataFromServer(){ // 从服务端加载数据的函数
        this.$http.get("/item/brand/page",{
            params:{
              page:this.pagination.page,
              rows:this.pagination.rowsPerPage,
              sortBy:this.pagination.sortBy,
              desc:this.pagination.descending,
              key:this.search
            }
        }).then(resp=>{
          console.log(resp)
          this.brands = resp.data.items;
          this.totalBrands = resp.data.total; // 赋值数据总条数
          this.loading = false; // 数据加载完成
        })
      }
    },
    watch:{
        pagination:{
          deep:true,
          handler(){
            this.getDataFromServer()
          }
        },
        search(){
          this.pagination.page = 1;
          this.getDataFromServer();
        }
    },
    // 渲染后执行
    mounted(){
      this.getDataFromServer() // 调用数据初始化函数
    }
  }
</script>
