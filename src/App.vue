<template>
  <div class="app-container">
    <!-- Header头部区域 -->
    <Header></Header>
    <!-- 循环渲染每一个商品的信息 -->
    <Goods
      v-for="item in list"
      :key="item.id"
      :id="item.id"
      :title="item.goods_name"
      :pic="item.goods_img"
      :price="item.goods_price"
      :state="item.goods_state"
      :count="item.goods_count"
      @state-change="getNewState"
    >
      <Counter :num="item.goods_count" :id="item.id"></Counter>
    </Goods>

    <!-- Footer区域 -->
    <Footer
      :isfull="fullState"
      :amount="amt"
      :all="total"
      @full-change="getFullState"
    ></Footer>
  </div>
</template>

<script>
//导入axios请求库
import axios from "axios";
//导入需要的组件
import Header from "@/components/Header/Header.vue";
import Goods from "@/components/Goods/Goods.vue";
import Footer from "@/components/Footer/Footer.vue";

import Counter from "@/components/Counter/Counter.vue";

//导入eventBus.js
import bus from "@/components/eventBus.js";
// import Counter from "./components/Counter/Counter.vue";
export default {
  data() {
    return {
      //用来存储购物车的列表数据，默认为空数组
      list: [],
    };
  },

  //计算属性
  computed: {
    //动态计算出全选的状态是true 还是 false
    fullState() {
      return this.list.every((item) => item.goods_state);
    },
    //已勾选商品的总价格
    amt() {
      return this.list
        .filter((item) => item.goods_state)
        .reduce((total, item) => {
          return (total += item.goods_price * item.goods_count);
        }, 0);
    },
    //已勾选商品的总数量
    total() {
      return this.list
        .filter((item) => item.goods_state)
        .reduce((t, item) => {
          return (t += item.goods_count);
        }, 0);
    },
  },

  components: {
    Header,
    Goods,
    Footer,
    Counter,
  },
  methods: {
    //封装请求列表数据的方法
    async initCartList() {
      const { data: res } = await axios.get("https://www.escook.cn/api/cart");
      //只要请求回来的数据，在页面渲染期间要使用到，则必须转存到data中
      console.log(res);
      if (res.status === 200) {
        this.list = res.list;
      }
    },
    //接收子组件传递过来的数据 val的格式{id，value}
    getNewState(val) {
      this.list.some((item) => {
        if (item.id === val.id) {
          item.goods_state = val.value;
          //终止后续的循环
          return true;
        }
      });
    },
    //接收Footer子组件传递过来的全选按钮的状态
    getFullState(val) {
      this.list.forEach((item) => (item.goods_state = val));
    },
  },
  created() {
    this.initCartList();

    //EvenrBus
    bus.$on("share", (val) => {
      this.list.some((item) => {
        if (item.id === val.id) {
          item.goods_count = val.value;
          return true;
        }
      });
    });
  },
};
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>
