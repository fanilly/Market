<template>
<div class="container">
  <div class="top">
    <yd-navbar title="我的订单" bgcolor="#d94927" color="#fff">
      <section slot="left" @click="handleBack">
        <yd-navbar-back-icon color="#fff"></yd-navbar-back-icon>
      </section>
    </yd-navbar>
  </div>
  <yd-popup v-model="show" position="center" width="90%">
    <div style="background-color:#fff;">
      <yd-cell-group>
        <yd-cell-item>
          <span slot="left">单号：</span>
          <input slot="right" v-model="mouchNo" type="number" placeholder="请输入物流单号">
        </yd-cell-item>

        <yd-cell-item arrow type="label">
          <span slot="left">快递：</span>
          <select slot="right" v-model="testval">
                <option value="">请选择快递</option>
                <option v-for="(item,index) of brands" :key="index" :value="item.brandDesc">{{item.brandName}}</option>
          </select>
        </yd-cell-item>
      </yd-cell-group>

      <yd-button size="large" type="danger" @click.native="confirmRefund">确认退款</yd-button>
    </div>
  </yd-popup>
  <div class="order">
    <yd-tab active-color="#e8380d" :callback="loadingNow">
      <yd-tab-panel :label="'待发货('+nums[0]+')'" tabkey="1">
        <div class="list">
          <div class="list-item" v-for="item,key in data1" :key="key">
            <div class="list-tit">
              订单号:{{item.orderNo}} <span>{{item.createTime.substr(0,10)}}</span>
            </div>
            <div class="list-item-body" v-for="sub,key in item.goodslist" :key="key">
              <router-link :to="{ name: 'detail', params: {goodsId : sub.goodsId} }">
                <yd-flexbox>
                  <div class="goods-img">
                    <img :src="sub.goodsThums" alt="">
                  </div>
                  <yd-flexbox-item class="list-body-right">
                    <p>{{sub.goodsName}}</p>
                    <div class="goods-type">
                      规格：“<span>{{sub.goodsAttrName}}</span>”
                    </div>
                    <p class="goods-price">
                      单价：<i>{{sub.goodsPrice}} {{item.payType == 2 ? '两' : '元'}}</i><span>×{{sub.goodsNums}}</span>
                    </p>
                  </yd-flexbox-item>
                </yd-flexbox>
              </router-link>
            </div>
            <div class="list-item-bot">
              总计：<i>{{item.totalMoney}} {{item.payType == 2 ? '两' : '元'}}</i> <span class="gray" @click="cancelOrder" :id="item.orderId">取消订单</span>
            </div>
          </div>
        </div>
      </yd-tab-panel>
      <yd-tab-panel :label="'待签收('+nums[1]+')'" tabkey="2">
        <div class="list">
          <div class="list-item" v-for="item,key in data2" :key="key">
            <div class="list-tit">
              订单号:{{item.orderNo}} <span>{{item.createTime.substr(0,10)}}</span>
            </div>
            <div class="list-tit">
              物流单号:{{item.orderNum}}
            </div>
            <div class="list-item-body" v-for="sub,key in item.goodslist" :key="key">
              <router-link :to="{ name: 'detail', params: {goodsId : sub.goodsId} }">
                <yd-flexbox>
                  <div class="goods-img">
                    <img :src="sub.goodsThums" alt="">
                  </div>
                  <yd-flexbox-item class="list-body-right">
                    <p>{{sub.goodsName}}</p>
                    <div class="goods-type">
                      规格：“<span>{{sub.goodsAttrName}}</span>”
                    </div>
                    <p class="goods-price">
                      单价：<i>{{sub.goodsPrice}} {{item.payType == 2 ? '两' : '元'}}</i><span>×{{sub.goodsNums}}</span>
                    </p>
                  </yd-flexbox-item>
                </yd-flexbox>
              </router-link>
            </div>
            <div class="list-item-bot">
              总计：<i>{{item.totalMoney}} {{item.payType == 2 ? '两' : '元'}}</i> <span @click="gotGoods" :id="item.orderId">确认收货</span>
              <span class="yellow" @click="showWuLiu" :id="key">查看物流</span>
              <span class="gray" @click="handleRefund" :id="item.orderId">退货</span>
            </div>
          </div>
        </div>
      </yd-tab-panel>
      <yd-tab-panel :label="'已完成('+nums[2]+')'" tabkey="3">
        <yd-infinitescroll :callback="loadList" ref="infinitescrollDemo">
          <div class="list" slot="list">
            <div class="list-item" v-for="item,key in data3" :key="key">
              <div class="list-tit">
                订单号:{{item.orderNo}} <span>{{item.createTime.substr(0,10)}}</span>
              </div>
              <div class="list-tit">
                物流单号:{{item.orderNum}}
              </div>
              <div class="list-item-body" v-for="sub,key in item.goodslist" :key="key">
                <router-link :to="{ name: 'detail', params: {goodsId : sub.goodsId} }">
                  <yd-flexbox>
                    <div class="goods-img">
                      <img :src="sub.goodsThums" alt="">
                    </div>
                    <yd-flexbox-item class="list-body-right">
                      <p>{{sub.goodsName}}</p>
                      <div class="goods-type">
                        规格：“<span>{{sub.goodsAttrName}}</span>”
                      </div>
                      <p class="goods-price">
                        单价：<i>{{sub.goodsPrice}} {{item.payType == 2 ? '两' : '元'}}</i><span>×{{sub.goodsNums}}</span>
                      </p>
                    </yd-flexbox-item>
                  </yd-flexbox>
                </router-link>
              </div>
              <div class="list-item-bot">
                总计：<i>{{item.totalMoney}} {{item.payType == 2 ? '两' : '元'}}</i>
                <span v-if=" item.isAppraises != 1 ? true : false ">
                    <router-link :to="{ name: 'comment', query: {orderId:item.orderId} }">
                      立即评价
                    </router-link>
                  </span>
                <span class="yellow" @click="showWuLiu1" :id="key">查看物流</span>
              </div>
            </div>
          </div>
          <!-- 数据全部加载完毕显示 -->
          <span slot="doneTip">啦啦啦，啦啦啦，没有更多啦~~</span>
          <!-- 加载中提示，不指定，将显示默认加载中图标 -->
          <img slot="loadingTip" src="http://static.ydcss.com/uploads/ydui/loading/loading10.svg" />
        </yd-infinitescroll>
      </yd-tab-panel>
    </yd-tab>
  </div>
</div>
</template>
<script>
import config from '@/config.js';
let userId,tempOrderId;
export default {
  data() {
    return {
      userId: '',
      page: 1,
      testval:'',
      nums: ['--', '--', '--'],
      params: [],
      show: false,
      mouchNo:'',
      data1: [],
      data2: [],
      data3: [],
      brands: []
    }
  },
  beforeCreate() {
    this.userId = window.localStorage.getItem('userId');
    userId = window.localStorage.getItem('userId');
  },
  created() {
    this.getNums();
    this.userId = window.localStorage.getItem('userId');
    console.log(window.localStorage.getItem('userId'));
    let url = `${config.host}index.php?m=Mobile&c=Orders&a=queryDeliveryByPage&userId=${window.localStorage.getItem('userId')}`;
    console.log(url);
    this.$dialog.loading.open('数据加载中');
    this.$http.get(url).then((res) => {
      console.log('++++++++');
      console.log(res);
      console.log('++++++++');
      let data = res.body;
      for (let i = 0; i < data.length; i++) {
        for (let j = 0; j < data[i].goodslist.length; j++) {
          data[i].goodslist[j].goodsThums = config.host + data[i].goodslist[j].goodsThums;
        }
      }
      this.data1 = data;
      this.$dialog.loading.close();
    })
  },
  methods: {
    getNums() {
      let url = `${config.host}index.php?m=Mobile&c=Orders&a=orderNum&userId=${window.localStorage.getItem('userId')}`;
      this.$http.get(url).then(res => {
        console.log(res);
        this.nums = res.body;
      })
    },
    //取消订单
    cancelOrder(e) {
      let orderId = e.target.id;
      this.refund(orderId, 'cancel')
    },
    //退款
    handleRefund(e) {
      let orderId = e.target.id;
      this.refund(orderId, 'refund')
    },
    confirmRefund(){

      console.log('-------------------++++++++++++++++++++++++++++++++');
      console.log(123);
      console.log(this.testval);
      if(!this.mouchNo){
        this.$dialog.toast({
          mes: '单号不能为空！',
          timeout: 1500
        });
      }else if(!this.testval){
        this.$dialog.toast({
          mes: '请选择物流品种！',
          timeout: 1500
        });
      }else{
        let url = `${config.host}index.php?m=Mobile&c=orders&a=refund&orderId=${tempOrderId}&mouchNum=${this.mouchNo}&mouchType=${this.testval}`;
        console.log(url);
        this.$dialog.loading.open('提交中');
        this.$http.get(url).then((res) => {
          console.log(res);
          this.$dialog.loading.close();
          if (res.body.status == 1) {
            this.$dialog.toast({
              mes: '退款申请提交成功',
              timeout: 2000
            });
            setTimeout(function() {
              window.location.reload();
            }, 1200);
          } else {
            this.$dialog.toast({
              mes: '退款失败！',
              timeout: 1500
            })
          }
        })
      }
    },
    refund(orderId, flag) {
      if (flag !== 'cancel') {
        this.show = true;
        tempOrderId = orderId;
      } else {
        this.$dialog.confirm({
          title: flag === 'cancel' ? '取消订单' : '确认退货',
          opts: () => {
            //点击了确认 开始退货
            let url = `${config.host}index.php?m=Mobile&c=orders&a=refund&orderId=${orderId}`;
            this.$dialog.loading.open('提交中');
            this.$http.get(url).then((res) => {
              console.log(res);
              this.$dialog.loading.close();
              if (res.body.status == 1) {
                if (flag === 'cancel') {
                  this.$dialog.toast({
                    mes: '订单取消成功',
                    timeout: 1500
                  });
                } else {
                  this.$dialog.toast({
                    mes: '退款申请提交成功',
                    timeout: 2000
                  });
                }
                setTimeout(function() {
                  window.location.reload();
                }, 1200);
              } else {
                if (flag === 'cancel') {
                  this.$dialog.toast({
                    mes: '取消订单失败！',
                    timeout: 1500
                  })
                } else {
                  this.$dialog.toast({
                    mes: '退款失败！',
                    timeout: 1500
                  })
                }
              }
            })
          }
        });
      }
    },
    handleBack() {
      this.$router.go(-1);
    },
    //确认收货
    gotGoods(e) {
      // console.log(e.target.id);
      let orderId = e.target.id;
      let url = `${config.host}index.php?m=Mobile&c=Orders&a=orderConfirm&userId=${userId}&orderId=${orderId}`;
      this.$http.get(url).then((res) => {
        if (res.body.status == 1) {
          this.$dialog.toast({
            mes: '已确认收货',
            timeout: 1500
          });
          window.location.reload();
        } else {
          this.$dialog.toast({
            mes: '网络异常，请重试！',
            timeout: 1500
          })
        }
      })
    },
    // 选项卡切换时执行的回调，加载切换页得数据
    loadingNow(label, tabkey) {
      console.log('---------------------------');
      console.log(tabkey, label);
      if(tabkey == 0){
        this.data1 = [];
        this.$dialog.loading.open('数据加载中');
        let url = `${config.host}index.php?m=Mobile&c=Orders&a=queryDeliveryByPage&userId=${window.localStorage.getItem('userId')}`;
        console.log(url);
        this.$dialog.loading.open('数据加载中');
        this.$http.get(url).then((res) => {
          console.log('++++++++');
          console.log(res);
          console.log('++++++++');
          let data = res.body;
          for (let i = 0; i < data.length; i++) {
            for (let j = 0; j < data[i].goodslist.length; j++) {
              data[i].goodslist[j].goodsThums = config.host + data[i].goodslist[j].goodsThums;
            }
          }
          this.data1 = data;
          this.$dialog.loading.close();
        });
      }
      // console.log(label, tabkey);
      if (tabkey == 1) {
        this.data2 = [];
        if (this.data2.length == 0) this.$dialog.loading.open('数据加载中');
        console.log('----------------------------------2');
        let url = `${config.host}index.php?m=Mobile&c=Orders&a=queryReceiveByPage&userId=${userId}`;
        this.$http.get(url).then((res) => {
          console.log(res.body);
          let data = res.body.list;
          this.brands = res.body.brands;
          for (let i = 0; i < data.length; i++) {
            for (let j = 0; j < data[i].goodslist.length; j++) {
              data[i].goodslist[j].goodsThums = config.host + data[i].goodslist[j].goodsThums;
            }
          }
          this.data2 = data;
          this.$dialog.loading.close();
        })
      };
      if (tabkey == 2) {
        this.data3 = [];
        console.log('---------------------------------');
        if (this.data3.length == 0) this.$dialog.loading.open('数据加载中');
        let url = `${config.host}index.php?m=Mobile&c=Orders&a=queryCompleteOrders&userId=${userId}&p=${this.page}`;
        this.$http.get(url).then((res) => {
          console.log('success');
          console.log(res.body);
          console.log(userId);
          let data = res.body;
          console.log(data);
          for (let i = 0; i < data.length; i++) {
            for (let j = 0; j < data[i].goodslist.length; j++) {
              data[i].goodslist[j].goodsThums = config.host + data[i].goodslist[j].goodsThums;
            }
          }
          this.data3 = data;
          this.$dialog.loading.close();
        })
      }
    },
    showWuLiu(e) {
      let index = e.target.id;
      console.log(index);
      let url = `https://www.kuaidi100.com/chaxun?com=${this.data2[index].deliverType}&nu=${this.data2[index].orderNum}`;
      window.location.href = url;
    },
    showWuLiu1(e) {
      let index = e.target.id;
      console.log(index);
      let url = `https://www.kuaidi100.com/chaxun?com=${this.data3[index].deliverType}&nu=${this.data3[index].orderNum}`;
      window.location.href = url;
    },
    loadList() {
      this.page++;
      let url = `${config.host}index.php?m=Mobile&c=Orders&a=queryCompleteOrders&userId=${this.userId}&p=${this.page}`;
      this.$http.get(url).then(function(response) {
        const _list = response.body;

        this.data3 = [...this.data3, ..._list];

        if (_list.length < 10) {
          /* 所有数据加载完毕 */
          this.$refs.infinitescrollDemo.$emit('ydui.infinitescroll.loadedDone');
          return;
        }

        /* 单次请求数据完毕 */
        this.$refs.infinitescrollDemo.$emit('ydui.infinitescroll.finishLoad');

        this.page++;
      });
    }
  }
}
</script>
<style scoped>
.list {
  background: #f4f8fe;
}

.list-item {
  width: 100%;
  padding: 0.1rem 0;
  background: #ffffff;
  margin: 0.1rem 0;
  margin-bottom: .2rem;
}

.list-tit {
  height: .7rem;
  line-height: .7rem;
  border-bottom: 1px solid #e5e5e5;
  padding: 0 .2rem;
  color: #666;
}

.list-tit span {
  float: right;
  font-size: .24rem;
  color: #999;
}

.list-item-body {
  width: 100%;
  padding: .2rem;
  background: #fefefe;
  margin-bottom: .1rem;
  padding-bottom: .1rem;
  border-bottom: 1px solid #f4f4f4;
}

.list-item .list-item-body:last-child {
  margin-bottom: 0;
}

.list-item-body .goods-img {
  width: 1.8rem;
  height: 1.8rem;
  overflow: hidden;
}

.list-item-body .goods-img img {
  width: 100%;
}

.list-body-right {
  padding-left: .2rem;
}

.list-body-right .goods-type {
  color: #666;
  font-size: .24rem;
  line-height: .4rem;
}

.list-body-right .goods-type span {
  color: #999;
}

.goods-price {
  font-size: .24rem;
  color: #666;
}

.goods-price i {
  color: #e8380d;
}

.goods-price span {
  float: right;
}

.list-item-bot {
  height: .7rem;
  line-height: .7rem;
  padding: 0 .2rem;
}

.list-item-bot i {
  color: #e8380d;
}

.list-item-bot span {
  float: right;
  /* display: inline-block; */
  padding: 0 .25rem;
  height: .6rem;
  line-height: .6rem;
  color: #fff;
  margin-left: .2rem;
  border-radius: 4px;
  background: #e8380d;
}

.yellow {
  background: #ffae01 !important;
}

.gray {
  background-color: #999 !important;
}

.list-item-bot span {
  margin-left: .1rem !important;
  padding: 0px 0.15rem;
}
</style>
