<template>
  <div>
    <h1>投标报价评分系统</h1>
    <el-row class="info">
      <span class="label">招标人指标：</span>
      <span class="labelInput"><el-input type="tel" v-model="tendereeTarget"></el-input></span>
      <span class="label">浮动点：</span>
      <span class="labelInput">
        <el-select v-model="floatPoint">
          <el-option label="1%" value="1%"></el-option>
          <el-option label="0.5%" value="0.5%"></el-option>
          <el-option label="0%" value="0%"></el-option>
          <el-option label="-0.5%" value="-0.5%"></el-option>
          <el-option label="-1%" value="-1%"></el-option>
          <el-option label="-1.5%" value="-1.5%"></el-option>
          <el-option label="-2%" value="-2%"></el-option>
        </el-select>
      </span>
      <span class="label">评标指标：</span>
      <span class="">{{standardTarget}}</span>
      <span class="submitBt">
        <el-button type="warning" @click="addTestData">添加测试数据</el-button>
        <el-button type="warning" @click="addNew">新增公司</el-button>
        <el-button type="warning" @click="calcResult">计算结果</el-button>
      </span>
    </el-row>
    <div class="listTable">
      <div class="tableHead">
        <el-row>
        <el-col :span="1">顺序</el-col>
        <el-col :span="4">投标单位</el-col>
        <el-col :span="4">投标报价</el-col>
        <el-col :span="5">大多数投标人报价平均值</el-col>
        <el-col :span="4">是否有效投标</el-col>
        <el-col :span="3">报价得分</el-col>
        <el-col :span="1">名次</el-col>
        <el-col :span="2">操作</el-col>
      </el-row>
      </div>
      <div class="tableBody">
        <el-row v-for="(item,index) in companys" :key="index" :class="{'highlight': item.ranking == 1}">
          <el-col :span="1">{{index + 1}}</el-col>
          <el-col :span="4">
            <el-input v-model="item.name" size="small"></el-input>
          </el-col>
          <el-col :span="4">
            <el-input v-model="item.tenderOffer" type="tel" size="small"></el-input>
          </el-col>
          <el-col :span="5">{{item.otherOfferAverage}}</el-col>
          <el-col :span="4">
            <i :class="item.isValid ? 'el-icon-check right' : 'el-icon-close error'"></i>
          </el-col>
          <el-col :span="3">{{item.score}}</el-col>
          <el-col :span="1">{{item.ranking}}</el-col>
          <el-col :span="2" style="position: relative;">
            <el-button type="primary" icon="delete" @click="delRow(index)" size="small" class="delBt" title="删除"></el-button>
          </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        tendereeTarget: '',
        floatPoint: '0%',
        standardTarget: '',
        companys: [],
        company: {
          name: '',
          tenderOffer: '',
          otherOfferAverage: '',
          isValid: '',
          score: '',
          ranking: ''
        }
      }
    },
    created() {
      for (let i = 0; i < 7; i++){
        this.companys.push(Object.assign({},this.company));
      }
    },
    methods: {
      addNew() {
        this.companys.push(Object.assign({},this.company));
      },
      addTestData() {
//        let testData = [
//          4501.5607,4681.3000,4498.5250,5431.3783,5378.0000,4522.5000,5399.9039
//        ];
//        this.floatPoint = '1.25%';
//        this.tendereeTarget = 4638.0000;
        let testData = [
          61230486.51, 60936722.70, 61746060.14, 61302518.19, 58253673.83, 58164669.29, 58103681.21
        ];
        this.floatPoint = '1%';
        this.tendereeTarget = 61500000;
        this.companys.forEach((item, index) => {
          item.tenderOffer = testData[index];
        });
        this.calcResult();
      },
      valid() {
        let flag = true;
        this.companys.forEach((item) => {
          if (!this.tendereeTarget) {
            this.$message({
              showClose: true,
              message: '请先填写招标人指标',
              type: 'error'
            });
            flag = false;
          }
          if (!item.tenderOffer) {
            this.$message({
              showClose: true,
              message: '请先填写投标报价',
              type: 'error'
            });
            flag = false;
          }
        });
        return flag;
      },
      getSum(array) {
        let sum = 0;
        array.forEach(function (item) {
          sum += item.tenderOffer - 0;
        });
        return sum;
      },
      isValid(item) {
        let flag = false;
        let tenderOffer = item.tenderOffer || 0;
        let otherOfferAverage = item.otherOfferAverage || 0;
        if(tenderOffer <= this.tendereeTarget*1.05 && tenderOffer >= this.tendereeTarget*0.93){
          flag = true;
        }
        if(tenderOffer <= otherOfferAverage*1.05 && tenderOffer >= otherOfferAverage*0.93){
          flag = true;
        }
        return flag;
      },
      getStandardTarget() { // 评标指标
        let validCompanys = this.companys.filter(function (com) {
          return com.isValid;
        });
        let validLength = validCompanys.length;
        let sum = this.getSum(validCompanys);
        let average = 0;
        let standardTarget = 0;
        if(validLength > 3) {
          let tenderArr = validCompanys.map(function (item) {
            return item.tenderOffer - 0;
          });
          let max = Math.max(...tenderArr);
          let min = Math.min(...tenderArr);
          console.log('max:'+ max+';' + 'min:' +min);
          average = (sum - max - min)/(validLength - 2);
        }else if(validLength === 3){
          average = sum/validLength;
        }
        standardTarget =  Math.round(average/(1 + Math.abs(Number.parseFloat(this.floatPoint)/100)));
        return standardTarget;
      },
      calcResult() {
        if(!this.valid()) return;
        let sum = this.getSum(this.companys);
        let l = this.companys.length;
        this.companys.forEach((item) => {
          item.otherOfferAverage = Math.round((sum - item.tenderOffer)/(l - 1));
          item.isValid = this.isValid(item);
        });
        this.standardTarget = this.getStandardTarget();
        this.companys.forEach(item => {
          if (item.isValid) {
          let ratio = item.tenderOffer/this.standardTarget-1;
            let subFlag = Math.ceil(Math.abs(ratio)/0.005);
            item.score = 100 -0.5*subFlag;
          }
        });
        let orderArr = this.companys.filter(function (item) {
          return item.isValid;
        }).map(function (item) {
          return item.score;
        }).sort(function (a, b) {
          return a - b < 0;
        });
        this.companys.forEach(item => {
          if (item.isValid) {
            item.ranking = orderArr.indexOf(item.score) + 1;
          }
        });
      },
      delRow(index) {
        this.companys.splice(index, 1);
        this.calcResult();
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus">
  .right
    color: #13CE66
  .error
    color: #FF4949

  .el-row
    line-height: 36px
    height: 36px
    .el-col
      height: 100%


  .info
    text-align left
    span
      display inline-block
      text-align center
      &.label
        width: 120px
      &.labelInput
        width 180px
        & > *
          width 100%
      &.submitBt
        float right


  .listTable
    margin-top: 10px
    input
      text-align end
    .tableHead
      background: #20A0FF
      color: #fff
    .el-row
      box-sizing border-box
      &:hover
        background lighten(bisque, 30%)
      .el-input--small
        display block
        width: 90%
        margin: 0 auto
        input
          margin-top 1px
    .highlight
      border 2px solid #ff4949
      background: bisque
      border-radius 5px
    .delBt
      position absolute
      top: 50%
      transform translateY(-50%)


</style>
