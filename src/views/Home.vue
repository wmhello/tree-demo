<template>
  <div class="home" style="width: 400px;text-align:right;">
    <button @click="addData">新建数据</button>
    <tree v-for="child in list" :item.sync="child" :key="child.id" @treeItemSelect="treeItemSelect"></tree>
  </div>
</template>

<script>
// @ is an alias to /src
import treeData from "./data";
import tree from "@/components/Tree";
export default {
  name: "Home",
  data: () => ({
    treeData,
    list: [],
    currentId: null
  }),
  components: {
    tree
  },
  created() {
    this.initTreeData();
  },
  methods: {
    addData() {
      console.log("传到id为" + this.currentId + "子元素");
      // 用api获取原始数据
      // 原始数据为了显示，必须进行转化
      this.initTreeData();
    },
    resetSelect() {
      let reduceDataFunc = data => {
        data.map((m, i) => {
          m.isSelect = false;
          if (m.children.length > 0) {
            reduceDataFunc(m.children);
          }
        });
      };
      reduceDataFunc(this.list);
    },
    treeItemSelect(relation) {
      this.resetSelect();
      let arrRelation = relation.split(/#+/g);
      arrRelation.pop();
      arrRelation.shift();
      this.currentId = parseInt(arrRelation[arrRelation.length - 1]);
      let setSelect = (data, arr, level) => {
        data.forEach((item, index) => {
          // console.log(item.id + "==>" + parseInt(arr[level]));
          if (item.id === parseInt(arr[level])) {
            item.isSelect = true;
            if (item.children.length > 0) {
              item.isSelect = false;
              if (arr.length === level + 1) {
                item.isSelect = true;
              }
              setSelect(item.children, arr, level + 1);
            }
          } else {
            item.isSelect = false;
          }
        });
      };
      setSelect(this.list, arrRelation, 0);
      // this.$forceUpdate();
    },
    initTreeData() {
      console.log("处理前的:", JSON.parse(JSON.stringify(this.treeData)));
      // 这里一定要转化，要不然他们的值监听不到变化
      let tempData = JSON.parse(JSON.stringify(this.treeData));
      let reduceDataFunc = (data, level, pid, relation) => {
        data.map((m, i) => {
          m.isSelect = false;
          m.children = m.children || [];
          m.level = level;
          m.pid = pid;
          m.relation = relation + "#" + m.id + "#";
          if (m.children.length > 0) {
            reduceDataFunc(m.children, level + 1, m.id, m.relation);
          }
        });
      };
      reduceDataFunc(tempData, 1, 0, "0");
      console.log("处理后的:", tempData);
      this.list = tempData;
    }
  }
};
</script>
