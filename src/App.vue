<template>
  <div id="app">
    <el-row type="flex" justify="end">
      <el-col :span="1">
        <el-button @click="open">关于</el-button>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center">
      <el-col :span="16">
        <el-input placeholder="请输入RSS链接" v-model="link" clearable></el-input>
      </el-col>
      <el-col :span="2">
        <el-input placeholder="请输入生成多少" v-model="count" clearable></el-input>
      </el-col>
      <el-col :span="2">
        <el-button type="primary" @click="trs">点击生成</el-button>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center">
      <el-col :span="20">
        <el-table ref="multipleTable" :data="datas" height="700" tooltip-effect="dark" style="width: 100%" @selection-change="handleSelectionChange" :row-key="datas.links">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="title" label="标题" width="800"></el-table-column>
          <el-table-column prop="enclosure.link" label="下载地址" show-overflow-tooltip></el-table-column>
        </el-table>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center">
      <el-col :span="20">
        <el-input
  type="textarea"
  :rows="2"
  placeholder="请输入内容"
  v-model="opt">
</el-input>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center">
      <el-col :span="20">
        <div style="margin-top: 20px">
          <el-button @click="copy">复制选中</el-button>
          <el-button @click="reverseSelect">反选</el-button>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script src="/js/clipboard.min.js"></script>

<script>
import $ from 'jquery'
export default {
  name: 'App',
  data() {
    return {
      link: '',
      datas: [],
      opt: "",
      count:20
    }
  },
  methods: {
    //解析RSS to 对象
    trs() {
      $.ajax({
        url: "https://api.rss2json.com/v1/api.json",
        method: "GET",
        dataType: "json",
        data: {
          rss_url: `${this.link}`,
          api_key: "jff7q6kirojkkvw6xitpdyq7sfqgkvpvnoshxblz", // put your api key here
          count: `${this.count}`
        },
      }).done((response) => {
        if (response.status != "ok") {
          throw response.message;
        }
        console.log("====== " + response.feed.title + " ======");
        console.log(this.datas)
        for (let i in response.items) {
          console.log(response.items[i]);
          this.datas.push(response.items[i])
        }
      });
    },
    //复制函数
    copy() {
      navigator.clipboard.writeText(this.opt)
        .then(() => {
          this.$message('您所选的链接已复制');
        })
        .catch((error) => {
          this.$message('复制文本时出错');
        });
    },
    //收集选中选项
    handleSelectionChange(selection) {
      this.opt = '';
      for (let i in this.selection) {
        this.opt += this.selection[i].enclosure.link + '\n';
      }
    },
    //反选
    reverseSelect() {
    this.datas.forEach(row => {
    this.$refs.multipleTable.toggleRowSelection(row);
  })
},
  //弹出框
open() {
        this.$alert(`<p>API by <a target="_blank" href="https://rss2json.com/">rsstojson</a></p>
        <p>当前只支持解析生成<a  target="_blank" href="https://mikanani.me/">mikan</a>的RSS链接</p>
        <p>By <a target="_blank" href="https://blog.amzayo.com/">amzayo</a></p>`, {
          dangerouslyUseHTMLString: true
        });
      }
  }
}
</script>

<style>
</style>
