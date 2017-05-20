<template>
  <div>
    <h1>投标报价评分系统</h1>
    <el-row class="info">
      <span class="label">招标人指标：</span>
      <span class="labelInput"><el-input type="tel" v-model="tendereeTarget"></el-input></span>
      <span class="label">浮动点：</span>
      <span class="labelInput">
        <el-select v-model="floatPoint">
          <el-option label="1%" value="%1"></el-option>
          <el-option label="0.5%" value="0.5%"></el-option>
          <el-option label="0%" value="0%"></el-option>
          <el-option label="-0.5%" value="-0.5%"></el-option>
          <el-option label="-1%" value="-1%"></el-option>
          <el-option label="-1.5%" value="-1.5%"></el-option>
          <el-option label="-2%" value="-2%"></el-option>
        </el-select>
      </span>
      <span class="submitBt">
        <el-button type="warning" @click="addTestData">添加测试数据</el-button>
        <el-button type="warning" @click="addNew">新增公司</el-button>
        <el-button type="warning" @click="calcResult">计算结果</el-button>
      </span>
    </el-row>
    <div class="listTable">
      <div class="tableHead">
        <el-row>
        <el-col :span="2">顺序</el-col>
        <el-col :span="4">投标单位</el-col>
        <el-col :span="4">投标报价</el-col>
        <el-col :span="5">大多数投标人报价平均值</el-col>
        <el-col :span="4">是否有效投标</el-col>
        <el-col :span="3">报价得分</el-col>
        <el-col :span="2">名次</el-col>
      </el-row>
      </div>
      <div class="tableBody">
        <el-row v-for="(item,index) in companys" :key="index">
          <el-col :span="2">{{index}}</el-col>
          <el-col :span="4">
            <el-input v-model="item.name" size="small"></el-input>
          </el-col>
          <el-col :span="4">
            <el-input v-model="item.tenderOffer" type="tel" size="small"></el-input>
          </el-col>
          <el-col :span="5">{{item.otherOfferAverage}}</el-col>
          <el-col :span="4">
            <i :class="item.isValid ? 'el-icon-check right' : 'el-icon-close error'" v-if="item.isValid"></i>
          </el-col>
          <el-col :span="3">{{item.score}}</el-col>
          <el-col :span="2">{{item.ranking}}</el-col>
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
        floatPoint: '0.5%',
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
        let testData = [
          4501.5607,4681.3000,4498.5250,5431.3783,5378.0000,4522.5000,5399.9039
        ];
        this.tendereeTarget = 4638.0000;
        this.companys.forEach((item, index) => {
          item.tenderOffer = testData[index];
        });
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
      calcResult() {
        if(!this.valid) return;
        let sum = this.getSum(this.companys);
        let l = this.companys.length;
        this.companys.forEach((item) => {
          item.otherOfferAverage = Math.round((sum - item.tenderOffer)/(l - 1));
        })
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
    /*.tableBody*/
      /*.el-row*/
        /*&:nth-child(2n)*/
          /*background #99A9BF*/

</style>
