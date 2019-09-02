<template>
  <div class="lottery">
    <div class="lottery_content">
      <div class="lottery_user" ref="outer">
        <div class="lottery_slider" ref="inner">
          <p v-for="(item,index) in lotteryList" :key="index" :class="{'active':indexActive === index}" style="opacity:0.5">恭喜{{item.mobilePhoneStr}}，抽中{{item.name}}</p>
        </div>
      </div>
      <div class="lottery_circul">
        <div class="lot_pointer">
          <img @click="rotateNow" ref="pointer" src="../assets/images/抽奖.png" alt="">
        </div>
        <div class="lottery_circul_box" ref="cirlbox">
          <div class="roltbox" v-for="(item,index) in giftList" :key="index">
            <div class="coupon" v-if=" item.type == 'coupon'">
              <div>
                <p><span>{{item.award}}</span> &nbsp;元</p>
              </div>
            </div>
            <div class="empty" v-if=" item.type == 'empty'">
              谢谢<br> 参与
            </div>
            <div class="gift" v-if=" item.type == 'gift'">
              <img :src="item.picUrls" alt="奖品">
              <div style="color:#fff;">
                {{item.giftName}}
              </div>

            </div>
          </div>
        </div>
      </div>
      <div class="lottery_bottom">
        <span style="">抽奖次数({{lotterynum}})</span>
        <span @click="$router.push('/lotRecord')">中奖记录>></span>
      </div>

    </div>

    <div v-show="isMask1" class="mask">
      <div class="mask-box1">
        
        <div class="miss">很遗憾，大奖与您擦肩而过， 请再接再厉！</div>
        <div class="del" @click="showMask"><img src="../assets/images/icon_delete@2x.png" alt="xx.png"></div>
      </div>
    </div>

    <div v-show="isMask2" class="mask">
      <div class="mask-box3">
        
        <div class="mask-tit">
          <p >恭喜你!</p>
          <p>抽中了{{actLott.giftName}}</p>
        </div>
        <div class="mask-body">
          <div >{{actLott.award}}元</div>
          <div>
            <p v-if="actLott.couponCode =='red_coupon'">现金</p>
            <p v-else style="opacity:0.5;">全场通用</p>
          </div>

        </div>
        <div class="del" @click="showMask"><img src="../assets/images/icon_delete@2x.png" alt="xx.png"></div>
      </div>
    </div>

    <div v-show="isMask3" class="mask">
      <div class="mask-box2">
        <div class="mask-tit">
          <p >恭喜你!</p>
          <p>抽中了{{actLott.giftName}}</p>
        </div>
        <div  class="mask-body">
          <p  class="dialogTitle">请准确填写您的收货地址:</p>
          <p><span>姓名</span><input v-model="ruleForm.name" placeholder="请输入姓名" type="text"></p>
          <p><span>手机号</span><input v-model="ruleForm.tel" placeholder="请输入手机号" type="text"></p>
          <p>
            <span>邮寄地址</span>
              <select v-model="ruleForm.province" placeholder="请选择..." @change='changeCity(0)'>
                  <option v-for='(item,index) in provinceList' :key='index' :label="item.name" :value="item.id"></option>  
              </select>
              <select v-model="ruleForm.city" placeholder="请选择..." @change='changeCity(1)'>
                <option v-for='(item,index) in cityList' :key='index' :label="item.name" :value="item.id"></option>  
              </select>
              
              <select v-model="ruleForm.area" placeholder="请选择...">
                <option v-for='(item,index) in areaList' :key='index' :label="item.name" :value="item.id"></option>  
              </select>
          </p>
          <div class="dialog-address">
              <textarea  cols="25" rows="3" placeholder="请输入详细地址"  v-model="ruleForm.address"></textarea>    
          </div>

          <div  class="mask3-footer">
            <div style="border-right:1px solid #ddd;" @click="showMask">取消</div>
            <div style="color:#ff464c;" @click="topUpOrWiFunc">确定</div>
          </div>
        </div>
        
        <div class="del" @click="showMask"><img src="../assets/images/icon_delete@2x.png" alt="xx.png"></div>
      </div>
    </div>

    <div v-show="isMask4" class="mask">
      <div class="mask-box4">
        <div class="mask-tit">
          <p >恭喜你!</p>
          <p>抽中了{{actLott.giftName}}</p>
        </div>
        <div class="del" @click="showMask"><img src="../assets/images/icon_delete@2x.png" alt="xx.png"></div>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      indexActive: 1,
      isMask1: false,
      isMask2: false,
      isMask3: false,
      isMask4: false,
      ruleForm: {
        name: "",
        tel: "",
        province: "",
        city: "",
        area: "",
        address: "",
      },
      provinceList: [],
      cityList: [],
      areaList: [],
      canTap: true,
      lotterynum: 100,
      lotteryList: [],
      giftList: [
        {type:'coupon',couponCode:'red_envelope',award:'100',giftName:'100元现金劵'},
        {type:'empty'},
        {type:'empty'},
        {type:'gift',picUrls:require('../assets/images/phone.jpg'),giftName:'华为P30'},
        {type:'coupon',couponCode:'red_envelope',award:'10',giftName:'10元现金劵'},
        {type:'empty'},
        {type:'empty'},
        {type:'coupon',couponCode:'red_envelope',award:'10',giftName:'10元现金劵'},
      ],
      actLott: {},
      actLotRecord: {},
    };
  },
  methods: {
    // 滚动
    rollList() {
      api.rollList({ size: 10 }).then(res => {
        if (res.code === '200') {
          this.lotteryList = res.data;
          this.lotteryList.forEach((item, index) => {
            //  item.postPhoneStr = item.userId;
            item.mobilePhoneStr = item.mobilePhone.slice(0, 3) + "******" + item.mobilePhone.slice(-2, item.mobilePhone.length)
          })
        } else {
          _.toast(res.message)
        }
      })
    },
    slider() {
      var self = this;
      var outer = this.$refs.outer;
      var inner = this.$refs.inner;
      var aP = inner.querySelectorAll("p");
      var speed = 0;
      var timer = null,
        timers = null; //存储计时器;
      var max = inner.offsetHeight / 2; //864
      var w = aP[0].offsetHeight; //p的高度
      timer = setInterval(startMove, 100);
      function startMove() {
        speed--;
        if (speed % w == 0) {
          self.indexActive += 1;
          clearInterval(timer);
          timers = setTimeout(function () {
            timer = setInterval(startMove, 100);
          }, 3000);
        }
        if (speed <= -max) {
          setTimeout(function () {
            speed = 0;
            self.indexActive = 1;
          }, 3000);
        }
        inner.style.top = speed + "px";
      }
    },
    //抽奖旋转函数
    rotateNow() {
      if (this.canTap) {

        if (this.lotterynum > 0) {
          this.canTap = false;
          this.lotterynum -= 1;
          let self = this;

          var degs = "";
          let index =Math.floor(Math.random()*8);
          this.actLott = this.giftList[index];
          console.log( this.actLott,index);
          switch (index) {
            case 0:
              // degs < 23 || degs >= 338;
              degs = Math.floor(Math.random() * 44 - 22);
              break;
            case 7:
              degs = Math.floor(Math.random() * 44 + 23);
              break;
            case 6:
              degs = Math.floor(Math.random() * 44 + 68);
              break;
            case 5:
              degs = Math.floor(Math.random() * 44 + 113);
              break;
            case 4:
              degs = Math.floor(Math.random() * 44 + 158);
              break;
            case 3:
              degs = Math.floor(Math.random() * 44 + 203);
              break;
            case 2:
              degs = Math.floor(Math.random() * 44 + 248);
              break;
            case 1:
              degs = Math.floor(Math.random() * 44 + 293);
              break;
          }


          var pointer = this.$refs.cirlbox;
          var speed = 0;
          var timer1 = null;
          var cirls = Math.floor(Math.random() * 2 + 3); //圈数
          timer1 = setInterval(startMove, 3);//运动函数
          function startMove() {
            if (speed > (cirls - 1) * 360 + degs) {
              if (((cirls * 360 + degs - speed) / 360) < 0.1) {
                speed += 0.1;
              } else {
                speed = speed + (cirls * 360 + degs - speed) / 360;
              }
            } else if (speed <= (cirls - 1) * 360 + degs) {
              speed++;
            }

            pointer.style.transform = "rotate(" + speed + "deg)";
            if (speed > cirls * 360 + degs) {
              clearInterval(timer1);
              if (JSON.stringify(self.actLott) == "{}") {
                setTimeout(() => {
                  self.canTap = true;
                  self.isMask1 = true;
                }, 3000)
              } else {
                if (self.actLott.type == 'empty') {
                  setTimeout(() => {
                    self.canTap = true;
                    self.isMask1 = true;
                  }, 3000)
                } else if (self.actLott.type == 'gift') {
                  setTimeout(() => {
                    self.canTap = true;
                    this.ruleForm = {
                      name: "",
                      tel: "",
                      province: "",
                      city: "",
                      area: "",
                      address: "",
                    },
                      self.isMask3 = true;
                  }, 3000)
                } else if (self.actLott.type == 'coupon') {
                  console.log(self.actLott)
                  setTimeout(() => {
                    self.canTap = true;
                    self.isMask2 = true;
                  }, 3000)
                } else {
                  setTimeout(() => {
                    self.canTap = true;
                    self.isMask4 = true;
                  }, 3000)
                }
              }
            }
          }

        } else {
          console.log('抽奖次数已用完')
        }
      }
    },

    topUpOrWiFunc() {
      if (this.ruleForm.name) {
        if (this.ruleForm.tel && (/^1[34578]\d{9}$/.test(this.ruleForm.tel))) {
          if (this.ruleForm.province && this.ruleForm.city && this.ruleForm.area && this.ruleForm.address) {
            let postData = {
              id: this.actLotRecord.id,
              postName: this.ruleForm.name,
              postPhone: this.ruleForm.tel,
              postAddress: _.cityName(this.ruleForm.province) + ' ' + _.cityName(this.ruleForm.city) + ' ' + _.cityName(this.ruleForm.area) + ' ' + this.ruleForm.address,
            }
            api.writeAddress(postData).then(res => {
              if (res.code === '200') {
                // this.giftList = res.data;
                this.isMask3 = false;
                _.toast(res.message)
                setTimeout(() => {
                  this.$router.push('/lottery')
                }, 2000);
              } else {
                this.isMask3 = false;
                _.toast(res.message)
              }
            })
          } else {
            this.isMask3 = false;
            _.toast('地址请输入完整，可前往中奖纪录重新填写地址');
          }
        } else {
          this.isMask3 = false;
          _.toast('手机号码不符合规范，可前往中奖纪录重新填写地址');
        }
      } else {
        this.isMask3 = false;
        _.toast('姓名不能为空，可前往中奖纪录重新填写地址');
      }

    },
    changeCity(type) {
      if (type == 0) {
        this.cityList = _.cityList(this.ruleForm.province);
        this.ruleForm.city = "";
        this.ruleForm.area = "";
      }
      if (type == 1) {
        this.areaList = _.cityList(this.ruleForm.city);
        this.ruleForm.area = "";
      }
    },
    showMask() {
      this.isMask1 = false;
      this.isMask2 = false;
      this.isMask3 = false;
      this.isMask4 = false;
      var pointer = this.$refs.cirlbox;
      pointer.style.transform = "rotate(0deg)";
    },
  }
};
</script>
<style scoped lang="scss">
@import "../assets/scss/index.scss";
.lottery {
  width: 100%;
  height: 100%;
  background: url("../assets/images/抽奖bg.png") no-repeat;
  background-size: 100% 100%;
  overflow: scroll;
  background-color: #e9eef4;
  position: relative;

  .lottery_header {
    text-align: center;
    position: absolute;
    z-index: 100;
    width: 100%;
    top: 0;
    left: 0;
    // background: linear-gradient(to right, #fd5d5e, #ee4343);
    height: px2rem(90px);
    > img {
      position: absolute;
      width: px2rem(20px);
      height: px2rem(30px);
      top: 50%;
      left: px2rem(20px);
      margin-top: px2rem(-15px);
    }
    span {
      line-height: px2rem(90px);
      font-size: px2rem(36px);
      color: #fff;
    }
    div {
      position: absolute;
      top: 50%;
      right: px2rem(20px);
      color: #fff;
      font-size: px2rem(30px);
      margin-top: px2rem(-21px);
      img {
        width: px2rem(42px);
        height: px2rem(42px);
      }
    }
  }
  .lottery_content {
    width: 100%;
    height: 100%;
    position: relative;
  }
  .lottery_user {
    width: px2rem(550px);
    height: px2rem(120px);
    overflow: hidden;
    position: absolute;
    top: 25%;
    left: 0;
    right: 0;
    margin: 0 auto 0;
    .lottery_slider {
      width: px2rem(550px);
      position: absolute;
      top: 0;
      left: 0;
      p {
        padding-top: px2rem(10px);
        height: px2rem(30px);
        text-align: center;
        font-size: px2rem(24px);
        opacity: 0.5;
        font-family: "MFShangHei_Noncommercial";
        color: rgb(158, 92, 19);
      }
      .active {
        opacity: 1 !important;
      }
    }
  }
  .lottery_circul {
    width: px2rem(586px);
    height: px2rem(586px);
    position: absolute;
    top: 42%;
    left: 0;
    right: 0;
    margin: 0 auto 0;
    .lot_pointer {
      width: px2rem(142px);
      height: px2rem(142px);
      position: absolute;
      z-index: 100;
      top: px2rem(222px);
      left: px2rem(220px);
      // padding-top: 50% !important;
      // transform: translateY(px2rem(-82px));
      margin: 0 auto;
      img {
        width: px2rem(142px);
        // width: px2rem(142px);
      }
    }
    .lottery_circul_box {
      width: 100%;
      height: 100%;
      background: url("../assets/images/bejing-choujiang.png") no-repeat;
      background-size: 100% 100%;
      position: relative;
      .roltbox {
        position: absolute;
        top: px2rem(80px);
        left: px2rem(240px);
        width: px2rem(100px);
        height: px2rem(50px);
        // border: 1px solid white;
        transform-origin: px2rem(50px) px2rem(212px);
        // transform-origin: px2rem(293px) px2rem(293px);
      }
      .coupon {
        background: url("../assets/images/101_03.png") no-repeat;
        background-size: 100% 100%;
        width: 100%;
        height: 100%;
        > div {
          display: inline-block;
          width: px2rem(130px);
          height: px2rem(50px);
          vertical-align: middle !important;
          margin-left: px2rem(6px);
          .experience {
            transform: translateY(px2rem(-10px)) scale(0.5) !important;
          }
          p {
            transform: translateY(px2rem(-4px)) translateX(px2rem(-10px)) scale(0.5);
            color: #fff;
            margin: 0 !important;
          }
          p:nth-child(1) {
            // transform: translateX(-10PX);
            span {
              font-size: 27px;
            }
          }
          p:nth-child(2) {
            transform: translateY(px2rem(-22px))  scale(0.5);
          }
        }
        // }
      }
      .gift {

        transform: translateY(px2rem(-30px));
        text-align: center;
        width: 100%;
        height: 100%;
        // }
        img {
          width: px2rem(80px);
          height: px2rem(80px);
        }
      }
      .empty {
        text-align: center;
        color: #fff;
        font-size: px2rem(26px);
      }
      > .roltbox:nth-child(2) {
        transform: rotate(45deg);
      }
      > .roltbox:nth-child(3) {
        transform: rotate(90deg);
      }
      > .roltbox:nth-child(4) {
        transform: rotate(135deg);
      }
      > .roltbox:nth-child(5) {
        transform: rotate(180deg);
      }
      > .roltbox:nth-child(6) {
        transform: rotate(225deg);
      }
      > .roltbox:nth-child(7) {
        transform: rotate(270deg);
      }
      > .roltbox:nth-child(8) {
        transform: rotate(315deg);
      }
    }
  }
  textarea {
      padding: px2rem(15px);
      margin-top: px2rem(15px);
      margin-left: px2rem(120px);
      font-family: "Arial Normal", "Arial";
      font-style: normal;
      border-color: #ddd;
      font-size: px2rem(26px);
      color: rgb(153, 153, 153);
      border-radius: 5px;
      text-decoration: none;
    }
  
  input{
    width: px2rem(292px);
    height: px2rem(30px);
    padding: px2rem(15px);
    font-size: px2rem(26px);
    border:1px solid #ddd;
    color:#999999;
    border-radius: 5px;
  }
  select {
    width: px2rem(120px);
    height: px2rem(60px);
    border-radius: 5px;
    color:#999999;
    margin-top:px2rem(10px);
    margin-right: 3px;
    font-size: px2rem(30px);
    border:1px solid #ddd;
    background-color: #fff;
    // border: none;
    // color: #a9abac;
    padding: 3px;
  }

  .mask{
    position: absolute;
    top:0;
    bottom:0;
    right:0;
    left:0;
    z-index: 9999;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.5) ;
    .mask-box1{
      width:px2rem(550px);
      margin: 50% auto;
      position: relative;
      // transform: translateY(px2rem(312px));
      height: px2rem(410px);
      background:url("../assets/images/中奖-弹框_03.png") 0 0  no-repeat;
      background-size: 100% 100%;
      .del{
        position: absolute;
        top: px2rem(20px) ;
        right:px2rem(-40px) ;
        img{
          width: px2rem(42px) ;
          height: px2rem(42px) ; 
        }
      }
      .miss{
        font-size: px2rem(34px);
        font-family: "Arial";
        text-align: center;
        // transform: translateX(px2rem(-20px));
        padding: px2rem(210px) px2rem(30px)  0;
        color: #fff;
      }
    }
    .mask-box3{
      width:px2rem(730px);
      margin: 35% auto;
      position: relative;
      height: px2rem(690px);
      background:url("../assets/images/中奖-弹框2_03.png") 0 0  no-repeat;
      background-size:  px2rem(655px) 100%;
      .mask-tit{
        padding-top: px2rem(20px) ; 
        font-size: px2rem(44px) ; 
        font-family: "Adobe Heiti Std";
        color: rgb(255, 255, 255);
        text-align: center;
        p:nth-child(1){
          font-size: px2rem(56px) ; 
          margin:px2rem(110px) 0 px2rem(10px);
        }
        // p:nth-child(2){
        //   width:px2rem(500px);
        // }
      }
      .mask-body{
        width: px2rem(400px) ;
          height: px2rem(100px) ; 
          margin:  px2rem(130px) auto 0;
          transform: translateX(px2rem(8px));
          >div{
            display: inline-block;
            width: 48%;
            vertical-align: middle;
            font-size: px2rem(48px) ; 
            text-align: center;
            color:#fff;
            p{
              font-size: px2rem(32px) ; 
            }
          }
      }
      .del{
        position: absolute;
        top: px2rem(30px) ;
        right:px2rem(40px) ;
        img{
          width: px2rem(42px) ;
          height: px2rem(42px) ; 
        }
      }
    }
    .mask-box2{
      width:px2rem(580px);
      margin: 25% auto;
      position: relative;
      // transform: translateY(px2rem(312px));
      height: px2rem(860px);
      background:url("../assets/images/img_dii@2x.png") 0 0  no-repeat;
      background-size: 100% 100%;
      // overflow: hidden;
      .del{
        position: absolute;
        top: px2rem(-50px) ;
        right:px2rem(0px) ;
        img{
          width: px2rem(42px) ;
          height: px2rem(42px) ; 
        }
      }
      .mask-tit{
        padding-top: px2rem(20px) ; 
        font-size: px2rem(48px) ; 
        font-family: "Adobe Heiti Std";
        color: rgb(255, 255, 255);
        text-align: center;
      }
      .mask-body{
        font-size: px2rem(28px);
        margin: px2rem(145px) px2rem(25px) 0;
        p{
          margin-bottom:px2rem(15px);
          color:#999999;
          span:nth-child(1){
            display: inline-block;
            width:  px2rem(122px);
          }
        }
        .mask3-footer{
          margin-top:  px2rem(16px);
          border-top:1px solid #ddd;
          >div{
            display: inline-block;
            height: px2rem(100px); 
            line-height: px2rem(100px); 
            width: 49%;
            text-align: center;
            font-size: px2rem(32px);
            font-weight: 600;
          }
        }
      }
    }
    .mask-box4{
      width:px2rem(580px);
      margin: 50% auto;
      position: relative;
      // transform: translateY(px2rem(312px));
      height: px2rem(340px);
      background:url("../assets/images/img_dii_2@3x(1).png") 0 0  no-repeat;
      background-size: 100% 100%;
      .mask-tit{
        padding-top: px2rem(20px) ; 
        font-size: px2rem(48px) ; 
        font-family: "Adobe Heiti Std";
        color: rgb(255, 255, 255);
        text-align: center;
        p:nth-child(1){
          font-size: px2rem(50px) ; 
          margin:px2rem(20px) 0 px2rem(10px);
        }
      }
      .del{
        position: absolute;
        top: px2rem(-40px) ;
        right:px2rem(-40px) ;
        img{
          width: px2rem(42px) ;
          height: px2rem(42px) ; 
        }
      }
    }
  }
}
</style>

