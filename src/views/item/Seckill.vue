<template>
  <v-card>
    <v-toolbar class="elevation-0">
      <v-btn color="primary" @click="addGoods">选择秒杀商品</v-btn>
      <v-spacer/>
      <v-flex xs5>
        状态：
        <v-btn-toggle mandatory v-model.lazy="state">
          <v-btn>
            全部
          </v-btn>
          <v-btn :value="1">
            未开始
          </v-btn>
          <v-btn :value="2">
            进行中
          </v-btn>
          <v-btn :value="3">
            已结束
          </v-btn>
        </v-btn-toggle>
      </v-flex>
      <v-flex xs3>
        <v-text-field
            append-icon="search"
            label="搜索"
            single-line
            hide-details
            v-model="search"
            @click:append="getDataFromServer"
        />
      </v-flex>
    </v-toolbar>
    <v-divider/>
    <v-data-table
        :headers="headers"
        :items="goodsList"
        :pagination.sync="pagination"
        :total-items="totalGoods"
        :loading="loading"
        class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center">{{props.item.categoryName}}</td>
        <td class="text-xs-center">{{ props.item.brandName }}</td>
        <td class="text-xs-center">{{ props.item.num }}</td>
        <td class="text-xs-center">{{$format(props.item.oldPrice)}}</td>
        <td class="text-xs-center">{{$format(props.item.costPrice)}}</td>
        <td class="text-xs-center">{{props.item.beginTime}}</td>
        <td class="justify-center layout px-0">
          <v-tooltip left>
            <v-btn slot="activator" icon @click="editSecKill(props.item)">
              <i class="el-icon-edit"/>
            </v-btn>
            <span> 修改</span>
          </v-tooltip>
          <v-tooltip left>
            <v-btn icon slot="activator">
              <i class="el-icon-delete"/>
            </v-btn>
            <span> 删除</span>
          </v-tooltip>
        </td>
      </template>
    </v-data-table>
    <!--弹出的对话框-->
    <v-dialog max-width="800" v-model="show" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? '修改' : '新增'}}秒杀商品</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-3" style="height: 600px">
          <seckill-form  :oldGoods="oldGoods" :step="step" @close="closeWindow" :is-edit="isEdit" ref="goodsForm"/>
        </v-card-text>
        <!--底部按钮，用来操作步骤线-->
        <v-card-actions class="elevation-10">
          <v-flex class="xs3 mx-auto">
            <v-btn @click="previous" color="primary" :disabled="step === 1">上一步</v-btn>
            <v-btn @click="next" color="primary" :disabled="step === 2">下一步</v-btn>
          </v-flex>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  // 导入自定义的表单组件
  import SeckillForm from './SeckillForm'

  export default {
    name: "seckill",
    data() {
      return {
        state: 1,
        search: '', // 搜索过滤字段
        totalGoods: 0, // 总条数
        goodsList: [], // 当前页品牌数据
        loading: true, // 是否在加载中
        pagination: {}, // 分页信息
        headers: [
          {text: 'skuId', align: 'center', sortable: false, value: 'skuId'},
          {text: '商品名称', align: 'center', sortable: false, value: 'name'},
          {text: '商品分类', align: 'center', sortable: false, value: 'categoryName'},
          {text: '品牌', align: 'center', value: 'brandName', sortable: false,},
          {text: '数量', align: 'center', value: 'num', sortable: false,},
          {text: '原价(元)', align: 'center', value: 'oldPrice', sortable: false,},
          {text: '秒杀价(元)', align: 'center', value: 'oldPrice', sortable: false,},
          {text: '开始时间', align: 'center', value: 'beginTime', sortable: true,},
          {text: '操作', align: 'center', sortable: false}
        ],
        show: false,// 控制对话框的显示
        oldGoods: {}, // 即将被编辑的商品信息
        isEdit: false, // 是否是编辑
        step: 1, // 子组件中的步骤线索引，默认为1
      }
    },
    mounted() { // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },
    watch: {
      pagination: { // 监视pagination属性的变化
        deep: true, // deep为true，会监视pagination的属性及属性中的对象属性变化
        handler() {
          // 变化后的回调函数，这里我们再次调用getDataFromServer即可
          this.getDataFromServer();
        }
      },
      state(){
        this.getDataFromServer();
      }
    },
    methods: {
      getDataFromServer() { // 从服务的加载数的方法。
        // 发起请求
        this.$http.get("/manageSec/findSecKillPage", {
          params: {
            key: this.search, // 搜索条件
            state: this.state === 0 ? null : this.state,
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
          }
        }).then(resp => { // 这里使用箭头函数
          this.goodsList = resp.data.items;
          this.totalGoods = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        }).catch(() => {
          this.goodsList = [];
          this.totalGoods = 0;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        })
      },
      addGoods() {
        // 修改标记
        this.isEdit = false;
        // 控制弹窗可见：
        this.show = true;
        // 把oldBrand变为null
        this.oldGoods = {};
      },
      async editSecKill(oldGoods) {
        // 修改标记
        this.isEdit = true;
        // 控制弹窗可见：
        this.show = true;
        // 获取要编辑的goods
        this.oldGoods = oldGoods;
      },
      closeWindow() {
        console.log(1)
        // 重新加载数据
        this.getDataFromServer();
        // 关闭窗口
        this.show = false;
        // 将步骤调整到1
        this.step = 1;
      },
      previous() {
        if (this.step > 1) {
          this.step--
        }
      },
      next() {
        if (this.step < 2 && this.$refs.goodsForm.$refs.basic.validate()) {
          this.step++
        }
      }
    },
    components: {
      SeckillForm
    }
  }
</script>

<style scoped>

</style>
