<template>
  <v-stepper v-model="step">
    <v-stepper-header>
      <v-stepper-step :complete="step > 1" step="1">选择商品</v-stepper-step>
      <v-divider/>
      <v-stepper-step step="2">秒杀信息</v-stepper-step>
    </v-stepper-header>
    <v-stepper-items>
      <!--1、选择商品-->
      <v-stepper-content step="1">
        <v-flex class="xs10 mx-auto px-3">
          <v-form v-model="valid" ref="basic">
            <v-layout row>
              <v-flex xs5>
                <!--商品分类-->
                <v-cascader
                  url="/item/category/of/parent"
                  required
                  showAllLevels
                  v-model="goods.categories"
                  label="请选择商品分类"/>
              </v-flex>
              <v-spacer/>
              <v-flex xs5>
                <!--品牌-->
                <v-autocomplete
                  :items="brandOptions"
                  item-text="name"
                  item-value="id"
                  label="所属品牌"
                  v-model="goods.brandId"
                  required
                  autocomplete
                  clearable
                  dense chips
                  :rules="[v => !!v || '品牌不能为空']"
                >
                  <template slot="selection" slot-scope="data">
                    <v-chip small>{{ data.item.name}}</v-chip>
                  </template>
                </v-autocomplete>
              </v-flex>
            </v-layout>
            <v-layout>
              <v-flex xs5>
                <!--商品 spu-->
                <v-autocomplete
                        :items="goodsOptions"
                        item-text="name"
                        item-value="id"
                        label="选择商品"
                        v-model="goods.spuId"
                        required
                        autocomplete
                        clearable
                        dense chips
                        :rules="[v => !!v || '商品不能为空']"
                >
                  <template slot="selection" slot-scope="data">
                    <v-chip small>{{ data.item.name}}</v-chip>
                  </template>
                </v-autocomplete>
              </v-flex>
              <v-spacer/>
              <v-flex xs5>
                <!--商品 sku-->
                <v-autocomplete
                        :items="skuOptions"
                        item-text="title"
                        item-value="id"
                        label="选择SKU"
                        v-model="goods.skuId"
                        required
                        autocomplete
                        clearable
                        dense chips
                        :rules="[v => !!v || 'SKU不能为空']"
                >
                  <template slot="selection" slot-scope="data">
                    <v-chip small>{{ data.item.title}}</v-chip>
                  </template>
                </v-autocomplete>
              </v-flex>
            </v-layout>
            <v-flex>
              <v-text-field label="商品标题" v-model="goods.title" :counter="200" required :rules="[v => !!v || '商品标题不能为空']" hide-details/>
              <v-text-field label="商品卖点" v-model="goods.subTitle" :counter="200" hide-details/>
              <!--图片上传组件-->

                <v-card class="elevation-2 flex xs11 mx-auto my-2">
                  <img width="200px" :src="goods.skuPic" alt="">
                </v-card>


            </v-flex>
          </v-form>
        </v-flex>
      </v-stepper-content>
      <!--2、商品描述-->
      <v-stepper-content step="2">
        <v-layout>
          <v-flex xs5>
              <v-menu
                      v-model="menu2"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      transition="scale-transition"
                      offset-y
                      min-width="290px"

              >
                <template v-slot:activator="{ on }">
                  <v-text-field
                          v-model="date"
                          label="请选择秒杀日期"
                          prepend-icon="event"
                          readonly
                          v-on="on"
                          :rules="[v => !!v || '请选择秒杀日期']"
                  ></v-text-field>
                </template>
                <v-date-picker v-model="date" @input="menu2 = false" locale="zh-cn"></v-date-picker>
              </v-menu>
          </v-flex>
          <v-spacer/>
          <v-flex xs5>
              <v-select
                      :items="miaoshatiems"
                      v-model="mstime"
                      label="秒杀开始时间"
                      dense
                      outlined
                      :rules="[v => !!v || '请选择秒杀时间']"
              ></v-select>
          </v-flex>
        </v-layout>
        <v-flex>
        <v-text-field label="商品数量" v-model="goods.num"   hide-details/>
        <v-text-field label="商品原价" v-model="goods.oldPrice"  hide-details/>
        <v-text-field label="秒杀价格" v-model="goods.costPrice"   hide-details/>
        </v-flex>
        <!--提交按钮-->
        <v-flex xs3 offset-xs9>

          <v-btn color="info" @click="submit">保存秒杀信息</v-btn>
        </v-flex>
      </v-stepper-content>
    </v-stepper-items>
  </v-stepper>
</template>

<script>
export default {
  name: "seckill-form",
  props: {
    oldGoods: {
      type: Object
    },
    isEdit: {
      type: Boolean,
      default: false
    },
    step: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      valid:false,
      goods: {
        categories: [], // 商品分类信息
        brandId: 0, // 品牌id信息
        spuId:0,
        skuId:0,
        name:""
      },
      date: new Date().toISOString().substr(0, 10),
      menu2: false,
      miaoshatiems: ['10:00:00', '12:00:00', '14:00:00', '16:00:00', '18:00:00', '20:00:00'],
      brandOptions: [], // 品牌列表
      goodsOptions: [],//商品列表
      skuOptions:[],//sku列表
      mstime:"10:00:00"
    };
  },
  methods: {
    submit() {
      // 表单校验。
      if(!this.$refs.basic.validate){
        this.$message.error("请先完成表单内容！");
      }
      // 先处理goods，用解构表达式接收,除了categories外，都接收到goodsParams中
      const {
        categories: [{ id: cid1 }, { id: cid2 }, { id: cid3 }],
        ...goodsParams
      } = this.goods;
      Object.assign(goodsParams, {
        cid1,
        cid2,
        cid3, // 商品分类
      });
      goodsParams.oldPrice=this.$format(goodsParams.oldPrice); // 价格需要格式化
      goodsParams.costPrice=this.$format(goodsParams.costPrice); // 价格需要格式化
      goodsParams.beginTime = new Date(this.date+" "+this.mstime);
      goodsParams.endTime = null;
      this.$http({
        method: this.isEdit ? "put" : "post",
        url: "/manageSec/",
        data: goodsParams
      })
        .then(() => {
          // 成功，关闭窗口
          this.$emit("close");
          // 提示成功
          this.$message.success("保存成功了");
        })
        .catch(() => {
          this.$message.error("保存失败！");
        });
    }
  },
  watch: {
    oldGoods: {
      deep: true,
      handler(val) {
        if (!this.isEdit) {
          Object.assign(this.goods, {
            categories: null, // 商品分类信息
            brandId: 0, // 品牌id信息
            name: "", // 标题
            subTitle: "", // 子标题
          });
        } else {
          this.goods = Object.deepCopy(val);
          this.date = val.secKillDate;
          this.mstime = val.beginTime.substring(11,19);
          console.info(this.mstime);
          // 先得到分类名称
          const names = val.categoryName.split("/");
          delete this.goods.categoryName;
          delete this.goods.brandName;
          delete this.goods.createTime;
          // 组织商品分类数据
          this.goods.categories = [
            { id: val.cid1, name: names[0] },
            { id: val.cid2, name: names[1] },
            { id: val.cid3, name: names[2] }
          ];
          this.goods.id = val.id;
          this.goods.oldPrice =this.$format(val.oldPrice); // 价格需要格式化
          this.goods.costPrice=this.$format(val.costPrice); // 价格需要格式化
        }
      }
    },
    "goods.categories": {
      deep: true,
      handler(val) {
        // 判断商品分类是否存在，存在才查询
        if (val && val.length > 0) {
          //把品牌列表设置为空
          this.brandOptions = [];
          // 根据分类查询品牌
          this.$http
            .get("/item/brand/of/category?id=" + this.goods.categories[2].id)
            .then(({ data }) => {
              this.brandOptions = data;
            });
        }
      }
    },
    "goods.brandId":{
      handler(val){
        if (val && val!= 0) {
          //把商品列表设置为空
          this.goodsOptions = [];
          // 根据分类查询品牌
          this.$http
            .get("/item/spu/for/brand?id=" + this.goods.brandId+"&cid="+this.goods.categories[2].id)
            .then(({ data }) => {
              this.goodsOptions = data;
            });
        }
      }
    },
    "goods.spuId":{
      handler(val){
        if (val && val!= 0) {
          this.goodsOptions.forEach(spu =>{
            if(this.goods.spuId == spu.id){
              this.goods.name = spu.name;
            }
          });
          this.$http
              .get("/item/sku/of/spu?id=" + this.goods.spuId)
              .then(({ data }) => {
                this.skuOptions = data;
              });

        }
      }
    },
    "goods.skuId":{
      handler(val){
        if (val && val!= 0) {
          this.skuOptions.forEach(sku =>{
            if(sku.id == this.goods.skuId){
              this.goods.title = sku.title;
              this.goods.oldPrice =  this.$format(sku.price);
              this.goods.skuPic = sku.images?sku.images:"";

            }
          });
        }
      }
    }
  },
  computed: {
    skus() {
      // 过滤掉用户没有填写数据的规格参数
      const arr = this.specialSpecs.filter(s => s.options.length > 0);
      // 通过reduce进行累加笛卡尔积
      return arr.reduce(
        (last, spec, index) => {
          const result = [];
          last.forEach(o => {
            spec.options.forEach((option, i) => {
              const obj = JSON.parse(JSON.stringify(o));
              obj[spec.name] = {v:option, id:spec.id};
              obj.indexes = (obj.indexes || '') + '_' +  i
              if (index === arr.length - 1) {
                obj.indexes = obj.indexes.substring(1);
                // 如果发现是最后一组，则添加价格、库存等字段
                Object.assign(obj, {
                  price: 0,
                  stock: 0,
                  enable: false,
                  images: []
                });
                if (this.isEdit) {
                  // 如果是编辑，则回填sku信息
                  const sku = this.goods.skus.get(obj.indexes);
                  if (sku != null) {
                    const { price, stock, enable, images } = sku;
                    Object.assign(obj, {
                      price: this.$format(price),
                      stock,
                      enable,
                      images: images ? images.split(",") : [],
                    });
                  }
                }
              }
              result.push(obj);
            });
          });
          return result;
        },
        [{}]
      );
    },
    headers() {
      if (this.skus.length <= 0) {
        return [];
      }
      const headers = [];
      Object.keys(this.skus[0]).forEach(k => {
        let value = k;
        if (k === "price") {
          // enable，表头要翻译成“价格”
          k = "价格";
        } else if (k === "stock") {
          // enable，表头要翻译成“库存”
          k = "库存";
        } else if (k === "enable") {
          // enable，表头要翻译成“是否启用”
          k = "是否启用";
        } else if (k === "images" || k === 'indexes') {
          // 图片和索引不在表格中展示
          return;
        }
        headers.push({
          text: k,
          align: "center",
          sortable: false,
          value
        });
      });
      return headers;
    }
  }
};
</script>

<style scoped>
</style>
