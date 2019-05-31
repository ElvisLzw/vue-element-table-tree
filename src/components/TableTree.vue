<template>
  <div class="hello">
    <!-- table部分 -->
    <el-table
      v-if="this.spanMethod.length"
      ref="table"
      :data="testTableArr"
      :span-method="arraySpanMethod"
      class="ele-table table-info-area"
      border
      stripe
    >
      <el-table-column
        :prop="item"
        :label="tableTitle[item].name+info[tableTitle[item].type]"
        align="center"
        v-for="(item,index) in tableTitleKey"
        :key="index"
      ></el-table-column>
    </el-table>
  </div>
</template>

<script>
import { tableData, tableTitle } from './mock'
export default {
  name: "TableTree",
  data() {
    return {
      info: {
        string: "",
        avg: "(平均值)",
        sum: "(总数)"
      },
      testTableArr: [], // table数据
      spanKey: [], // 需要合并的列,每个元素为表格的prop值（键值）
      tableTitle: {}, // table表头显示
      tableTitleKey: [], // 所有的列的prop值（键值）
      spanMethod: [], // 表格向下合并的方式
      divArr: [], // 分片数据，用于将数据分片，保证不会
      divIndex: [0] // 用于记录分片的位置
    };
  },
  mounted() {
    this.spanKey = ["erpCategory1", "season"]
    this.tableTitle = tableTitle
    this.tableTitleKey = ["erpCategory1", "season", "skcBarcode"]
    this.getInit()
  },
  methods: {
    getInit() {
      setTimeout(() => {
        // mock异步
        let response = tableData
        this.testTableArr = this.initTableData({key:response.key,value:response.value})
        this.normalizeData(this.testTableArr,this.spanKey)
      }, 200);
    },
    /**
     * 将response{key:[],value:[]}  转换成可用于表格的数据
     * @param response
     * @returns {Promise}
     */
    initTableData(response) {
      let initArr = []
      for (const indexValue in response.value) {
        let initObj = {}
        for (const indexKey in response.key) {
          initObj[response.key[indexKey]] = response.value[indexValue][indexKey]
        }
        initArr.push(initObj)
      }
      return initArr
    },
    /**
     * 合并表格
     */
    arraySpanMethod({ row, column, rowIndex, columnIndex }) {
      if (this.spanKey.length) {
        for (let keyIndex = 0; keyIndex < this.spanKey.length; keyIndex++) {
          // 找到指定需要合并的列，应用合并规则
          if (this.spanKey[keyIndex] === this.tableTitleKey[columnIndex]) {
            const _row = this.spanMethod[keyIndex][rowIndex];
            const _col = _row > 0 ? 1 : 0;
            return {
              rowspan: _row,
              colspan: _col
            };
          }
        }
      } else {
        return {
          rowspan: 1,
          colspan: 1
        };
      }
    },
    /**
     * data为传入的表格数据，结构为[{},{}]
     * spanKey为需要合并操作的列  结构为[]
     */
    normalizeData(data, spanKey) {
      this.spanMethod = [];
      this.divIndex = [0];
      // 有需要合并的字段才做处理
      if (spanKey.length) {
        for (let keyIndex = 0; keyIndex < spanKey.length; keyIndex++) {
          const key = spanKey[keyIndex];
          this.sliceData();
          this.spanMethod.push(this.spanKeyMenthod(key));
        }
      }
    },

    /**
     * key 为指定列的键值
     * contactArr 指定的列下 行的合并方式   []
     * contactDot 单元格合并展示位
     */
    spanKeyMenthod(key) {
      let divContactArr = []; // 记录当前列的向下合并方式
      // 计算每一片单独的行扩展方式
      let divIndexTemp = [...this.divIndex];
      // 重点： 记录每次循环的上一次分片位置，目的是让当前位置index插入在分片中   eg：[0,6,20]
      for (const itemIndex in this.divArr) {
        let divArrItem = this.divArr[itemIndex];
        let contactArr = [];
        let contactDot = 0;
        divArrItem.map((item, index) => {
          if (index == 0) {
            contactArr.push(1);
            contactDot = 0;
          } else {
            // 不是第一行的情况下比较和上一行的值
            if (item[key] === divArrItem[index - 1][key]) {
              // 值相同则放入0，第一个相同的合并方式数组加一
              contactArr[contactDot] += 1;
              contactArr.push(0);
            } else {
              contactArr.push(1);
              contactDot = index;
              // 记录分片的位置
              this.divIndex.push(index + divIndexTemp[itemIndex]);
              // 重点：插入分片中    eg：第一块分片位置index在0,6中插入   [0,2,3,5,6,20]  第二块分片位置index在6,20中插入[0,2,3,5,6,8,11,15,20]
              this.divIndex.sort((a, b) => {
                return a - b;
              }); // 因为是push进去  所以需要排序
              this.divIndex = [...new Set(this.divIndex)]; // 去重
            }
          }
        });
        divContactArr = divContactArr.concat(contactArr);
      }
      return divContactArr;
    },
    /**
     * 将数据分片，防止数据向下合并的时候跨级
     */
    sliceData() {
      this.divArr = [];
      for (let index = 0; index < this.divIndex.length; index++) {
        if (index == this.divIndex.length - 1) {
          // 如果最后一个则截取到数据最末尾
          this.divArr.push(this.testTableArr.slice(this.divIndex[index]));
        } else {
          // 截取两个分片位置之间的数据
          this.divArr.push(
            this.testTableArr.slice(
              this.divIndex[index],
              this.divIndex[index + 1]
            )
          );
        }
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
