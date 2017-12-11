<template>
  <div class="pos">
    <el-row>
      <el-col :span='7' class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width:100%">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column  label="操作" width="100" fixed="right">
                <template scope="scope">
                  <el-button type="text" size="small" @click="deleteList(scope.row)">
                    删除
                  </el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">
                    增加
                  </el-button>
                </template> 
               </el-table-column>
            </el-table>
            <div class="totalDiv">
              <small>数量：</small>{{totalCount}} <small>金额：</small>{{totalMoney}}元
            </div>
            <div class="div-btn">
              <el-button type="warning">
                挂单
              </el-button>
              <el-button type="danger" @click="delAllGoods">
                删除
              </el-button>
              <el-button type="success" @click="checkout">
                结账
              </el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">

          </el-tab-pane>
          <el-tab-pane label="外卖">

          </el-tab-pane>
        </el-tabs>
      </el-col>
       <el-col :span="17">
         <div class="often-goods">
           <div class="title">常用商品</div>
           <div class="ofter-goods-list">
             <ul>
               <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                 <span>{{goods.goodsName}}</span>
                 <span class="o-price">￥{{goods.price}}元</span>
               </li> 
             </ul>
           </div>   
         </div>
         <div class="goods-type"> 
           <el-tabs>
             <el-tab-pane label="汉堡">
               <div>
                 <ul class="cookList">
                    <li v-for="goods in type0goods" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{goods.price}}元</span>
                     </li>
                 </ul>
               </div>
             </el-tab-pane>
             <el-tab-pane label="小食">
                <ul class="cookList">
                  <li v-for="goods in type1goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                    </li>
                </ul>
             </el-tab-pane>
             <el-tab-pane label="饮料">
                <ul class="cookList">
                  <li v-for="goods in type2goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                    </li>
                </ul>
             </el-tab-pane>
             <el-tab-pane label="套餐">
                <ul class="cookList">
                  <li v-for="goods in type3goods">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">￥{{goods.price}}元</span>
                    </li>
                </ul>
             </el-tab-pane>
           </el-tabs>
         </div>
      </el-col>
    </el-row>  
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "pos",
  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0goods: [],
      type1goods: [],
      type2goods: [],
      type3goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created: function() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(res => {
        this.oftenGoods = res.data;
      })
      .catch(() => {
        alert("网络错误");
      });
    axios
      .get("http://jspang.com/DemoApi/typeGoods.php")
      .then(res => {
        this.type0goods = res.data[0];
        this.type1goods = res.data[1];
        this.type2goods = res.data[2];
        this.type3goods = res.data[3];
      })
      .catch(() => {
        alert("网络错误");
      });
  },
  mounted: function() {
    let orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + "px";
  },
  methods: {
    addOrderList(goods) {
      this.totalMoney = 0;
      this.totalCount = 0;
      // 商品是否存在列表中
      let isHave = false;
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId === goods.goodsId) {
          isHave = true;
        }
      }
      // 跟进判断的值编写业务逻辑
      if (isHave) {
        // 改变列表中商品的数量
        let arr = this.tableData.filter(o => o.goodsId === goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          price: goods.price,
          count: 1
        };
        this.tableData.push(newGoods);
      }
      this.getAllMoney()
    },
    // 删除单个商品
    deleteList(goods) {
      this.tableData = this.tableData.filter(o => o.goodsId !== goods.goodsId);
      this.getAllMoney()
    },
    delAllGoods(){
      this.tableData = []
      this.totalCount = 0
      this.totalMoney = 0
    },
    // 汇总数量和金额
    getAllMoney() {
      this.totalCount = 0;
      this.totalMoney = 0;
      if (this.tableData) {
        // 计算汇总数量和金额
        this.tableData.forEach(element => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + element.price * element.count;
        });
      }
    },
    checkout() {
      if (this.totalCount != 0) {
        this.tableData =[]
        this.totalCount = 0
        this.totalMoney = 0
        this.$message({
          message: '结账成功，感谢！',
          type: 'success'
        });
      } else {
        this.$message.error('不能空结账！')
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
  height: 100%;
}
.div-btn {
  margin-top: 10px;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
  text-align: left;
}
.ofter-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 10px;
  background-color: #fff;
}
.o-price {
  color: #98bcff;
}
.goods-type {
  clear: both;
}
.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auot;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
}
.cookList li span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}
.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.goods-type {
  height: auto;
  overflow: hidden;
  border-top: 1px solid #d3dce6;
}
.totalDiv {
  background-color: #fff;
  padding: 10px;
  border-bottom: 1px solid #d3dce6;
}
</style>
