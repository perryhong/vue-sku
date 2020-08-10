<template>
  <div class="container">
    <div v-for="(item, index) in specList" :key="index">
      <p class="title">{{item.title}}</p>
      <div class="spec-row">
        <span
          v-for="item2 in item.list"
          :key="item2.id"
          :class="{
            'spec-choice': true,
            'spec-active': isActive(item2),
            'disabled': !isOption(item2)
          }"
          @click="handleClick(isOption(item2), item2, index)"
        >
          {{item2.value}}
        </span>
      </div>
    </div>
  </div>
</template>

<script>

  const specList = [
    {
      title: '颜色',
      list: [{
        value: '红色',
        id: '1'
      }, {
        value: '紫色',
        id: '2'
      }]
    },
    {
      title: '套餐',
      list: [{
        value: '套餐一',
        id: '3'
      }, {
        value: '套餐二',
        id: '4'
      }]
    },
    {
      title: '内存',
      list: [{
        value: '64G',
        id: '5'
      }, {
        value: '128G',
        id: '6'
      }, {
        value: '256G',
        id: '7'
      }]
    }
  ];

  const specCombinationList = [
    {
      id: '1',
      specs: [{
        value: '紫色',
        id: '2'
      }, {
        value: '套餐一',
        id: '3'
      }, {
        value: '64G',
        id: '5'
      }]
    },
    {
      id: '2',
      specs: [{
        value: '紫色',
        id: '2'
      }, {
        value: '套餐一',
        id: '3'
      }, {
        value: '128G',
        id: '6'
      }]
    },
    {
      id: '3',
      specs: [{
        value: '紫色',
        id: '2'
      }, {
        value: '套餐二',
        id: '4'
      }, {
        value: '128G',
        id: '6'
      }]
    },
    {
      id: '4',
      specs: [{
        value: '红色',
        id: '1'
      }, {
        value: '套餐二',
        id: '4'
      }, {
        value: '256G',
        id: '7'
      }]
    }
  ];

  export default {
    name: 'sku',
    data() {
      return {
        specList,
        specCombinationList,
        specsS: [],
        adjoinArray: []
      };
    },
    computed: {
      optionSpecs() {
        return this.getSpecOptions(this.specsS);
      },
      vertex() {
        return this.specList.reduce((total, current) => [...total, ...current.list], []);
      },
      len() {
        return this.vertex.length;
      }
    },
    created() {
      this.init();
    },
    methods: {
      isActive(val) {
        return this.specsS.find((item) => item && item.id === val.id);
      },
      isOption(val) {
        return this.optionSpecs.find((item) => item.id === val.id);
      },
      init() {
        this.adjoinArray = Array(this.len * this.len).fill(0)
        this.specsS = Array(this.specList.length).fill(null);
        console.log(this.specsS)
        this.initSpec();
        console.log(this.adjoinArray);
        this.initSameLevel();
        console.log(this.adjoinArray);
      },
      initSpec() {
        this.specCombinationList.forEach((item) => this.fillInSpec(item.specs));
      },
      initSameLevel() {
        this.specList.forEach((item) => {
          const params = [];
          // 获取同级别顶点
          item.list.forEach((val) => {
            if (this.optionSpecs.includes(val)) params.push(val);
          });
          // 同级点位创建
          this.fillInSpec(params);
        });
      },
      fillInSpec(params) {
        params.forEach((param) => {
          this.setAdjoinVertexs(param, params);
        });
      },
      setAdjoinVertexs(side, sides) {
        let pIndex = '';
        for (let i = 0; i < this.vertex.length; i += 1) {
          if (side.id === this.vertex[i].id) {
            pIndex = i;
            break;
          }
        }
        sides.forEach((item) => {
          let index = '';
          for (let i = 0; i < this.vertex.length; i += 1) {
            if (item.id === this.vertex[i].id) {
              index = i;
              break;
            }
          }
          this.adjoinArray[pIndex * this.len + index] = 1;
        });
      },
      getVertexCol(param) {
        let idx = ''
        for (let i = 0; i < this.vertex.length; i += 1) {
          if (param.id === this.vertex[i].id) {
            idx = i;
            break;
          }
        }
        const col = [];
        this.vertex.forEach((item, pIndex) => {
          col.push(this.adjoinArray[idx + this.len * pIndex]);
        });
        return col;
      },
      getColSum(params) {
        const paramsVertex = params.map((param) => this.getVertexCol(param));
        const paramsVertexSum = [];
        this.vertex.forEach((item, index) => {
          const rowTotal = paramsVertex
            .map((value) => value[index])
            .reduce((total, current) => {
              let newTotal = total;
              newTotal += current || 0;
              return newTotal;
            }, 0);
          paramsVertexSum.push(rowTotal);
        });
        return paramsVertexSum;
      },
      getUnion(params) {
        const paramsColSum = this.getColSum(params);
        const union = [];
        paramsColSum.forEach((item, index) => {
          if (item && this.vertex[index]) union.push(this.vertex[index]);
        });
        return union;
      },
      getIntersection(params) {
        const paramsColSum = this.getColSum(params);
        const intersection = [];
        paramsColSum.forEach((item, index) => {
          if (item >= params.length && this.vertex[index]) intersection.push(this.vertex[index]);
        });
        return intersection;
      },
      getSpecOptions(params) {
        let specOptionCanChoose = [];
        if (params.some(Boolean)) {
          specOptionCanChoose = this.getIntersection(params.filter(Boolean));
        } else {
          specOptionCanChoose = this.getUnion(this.vertex);
        }
        return specOptionCanChoose;
      },
      handleClick(bool, item, idx) {
        // 排除可选规格里面没有的规格
        if (!bool) return;
        // 根据text判断是否已经被选中了，如果已选则取消选中
        if (!this.specsS[idx]) {
          this.$set(this.specsS, idx, item);
        } else {
          this.$set(this.specsS, idx, this.specsS[idx].id === item.id ? null : item);
        }
      }
    }
  };
</script>

<style lang="scss" scoped>
  .container {
    width: 400px;
    position: absolute;
    margin: auto;
    left: 0;
    right: 0;
    top: 50px;
  }
  .title {
    font-size: 16px;
    padding: 5px 0;
    color: #262626;
  }
  .spec-row {
    margin: 5px 0 5px 0;
  }
  .spec-choice {
    display: inline-block;
    margin-left: 20px;
    background-color: #f3f3f3;
    padding: 5px 10px 5px 10px;
    color: #505257;
    border: 1px solid #f3f3f3;
  }
  .spec-active {
    background-color: #fef6f4;
    color: #e34a40;
    border: 1px solid #e34a40;
  }
  .disabled {
    color: #bebebe;
  }
</style>
