<template>
  <div id="app">
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <h1>动漫RSS订阅提取下载地址</h1>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <el-input placeholder="请输入RSS链接" v-model="link" clearable></el-input>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <el-row type="flex" justify="space-between">
          <el-col :span="6">
        <el-input placeholder="最多生成的条数" v-model="count" clearable></el-input>
      </el-col>
      <el-col :span="12" style="text-align: center;">
        <span class="seletTip">选择链接网址：</span>
        <el-select v-model="type" placeholder="" class="left30px" @change="trs">
          <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="6">
        <el-button type="primary" id="trsbtn" @click="trs">点击生成</el-button>
      </el-col>
        </el-row>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <div>
          <h3>由于各字幕组命名标准不同，可能有些条件并不能匹配到，还请见谅</h3>
        </div>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <el-row type="flex" justify="space-between">
          <el-col :span="5">
            <el-select @change="filter" v-model="resolution.selected" collapse-tags multiple placeholder="分辨率" style="width: 100%">
            <el-option v-for="item in resolution.option" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          </el-col>
          <el-col :span="5">
            <el-select @change="filter" v-model="subLan.selected" collapse-tags multiple placeholder="字幕语言" style="width: 100%">
            <el-option v-for="item in subLan.option" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          </el-col>
          <el-col :span="5">
            <el-select @change="filter" v-model="subEM.selected" collapse-tags multiple placeholder="字幕嵌入方式" style="width: 100%">
            <el-option v-for="item in subEM.option" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          </el-col>
        </el-row>
        <!-- <div class="opc"> -->
          
        <!-- </div> -->
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <el-table ref="multipleTable" :data="showDatas" height="700" tooltip-effect="dark" style="width: 100%"
          @selection-change="handleSelectionChange" :row-key="showDatas.links">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="title" label="标题" ></el-table-column>
          <el-table-column prop="link" label="下载地址" show-overflow-tooltip width="100"></el-table-column>
        </el-table>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top20px">
      <el-col :span="16">
        <el-input type="textarea" :rows="2" placeholder="如果复制失败，您可以在这里手动复制结果" v-model="opt">
        </el-input>
      </el-col>
    </el-row>
    <el-row type="flex" justify="space-around" class="top20px">
      <el-col :span="7">
        <el-button @click="copy">复制选中</el-button>
        <el-button @click="reverseSelect">反选</el-button>
      </el-col>
      <el-col :span="1"><el-button id="about" @click="open">关于</el-button></el-col>
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
      originDatas:[],
      showDatas: [],
      opt: "",
      count:'',
      options: [{
          value: 'mikan',
          label: '蜜柑动漫'
        }, {
          value: 'manmao/kiss',
          label: '漫猫/爱恋动漫'
        }],
      type:'mikan',
      resolution:{
        option:[{
        value: '720',
        label: '720P'
      },{
        value: '1080',
        label: '1080P'
      },{
        value: '1440',
        label: '1440P'
      },{
        value: '2560',
        label: '2560P'
      }],
      selected:[]
      },
      subLan:{
        option:[{
        value: '简体|GB|简日',
        label: '简体中文'
      },{
        value: 'BIG5|繁日|繁体',
        label: '繁体中文'
      }],
      selected:[]
      },
      // 字幕嵌入方式
      subEM:{
        option:[{
        value: '内嵌',
        label: '内嵌字幕'
      },{
        value: '内封',
        label: '内封字幕'
      },{
        value: '外挂',
        label: '外挂字幕'
      }],
      selected:[]
      }
    }
  },
  methods: {
    //解析RSS to 对象
    trs(){
      if(this.link.trim() === ''){
        this.$message({
              type: 'info',
              message: `请输入要提取的订阅`
            });
          return false;
      }
      this.originDatas = [];
      this.showDatas = [];
      $.ajax({
        url: "https://api.rss2json.com/v1/api.json",
        method: "GET",
        dataType: "json",
        data: {
          rss_url: `${this.link}`,
          api_key: "jff7q6kirojkkvw6xitpdyq7sfqgkvpvnoshxblz", // 请填写你自己的api_key
          count: this.count || 999//最大解析数量
        },
      }).done((response) => {
        for (let i in response.items) {
          const tempObj = {};
          if(this.type === 'mikan'){
            tempObj.title = response.items[i].title;
            tempObj.link = response.items[i].enclosure.link;
          }else if(this.type === 'manmao/kiss'){
            // 拆分获取到的链接，后续拼接成能用的种子地址
            const templink = response.items[i].enclosure.link.split("hash=")
            tempObj.title = response.items[i].title;
            tempObj.link = `magnet:?xt=urn:btih:${templink[templink.length-1]}`;
          }
          this.originDatas.push(tempObj)
          this.showDatas.push(tempObj)
        }
        this.$message({
              type: 'success',
              message: `下载地址提取成功`
            });
      }).fail((response) => {
          this.$message({
              type: 'error',
              message: `出错了，请确认地址是否正确`
            });
      });
    },
    // 条件筛选
    filter(){
      const resolutioncPattern = new RegExp(this.resolution.selected.join('|'), 'i');
      const subLanPattern = new RegExp(this.subLan.selected.join('|'), 'i');
      const subEMPattern = new RegExp(this.subEM.selected.join('|'), 'i');
      this.showDatas = this.originDatas.filter((item) => {
        let t = item.title
          if(resolutioncPattern.test(t) && subLanPattern.test(t) && subEMPattern.test(t)){
            return item;
        }
      })
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
      this.selection=selection;
      this.opt = '';
      for (let i in this.selection) {
        this.opt += this.selection[i].link + '\n';
      }
    },
    //反选
    reverseSelect() {
    this.showDatas.forEach(row => {
    this.$refs.multipleTable.toggleRowSelection(row);
  })
},
  //弹出框
open() {
        this.$alert(`<p>API by <a target="_blank" href="https://rss2json.com/">rsstojson</a></p>
        <p>目前支持解析的网站：</p>
        <p><a  target="_blank" href="https://mikanani.me/">mikan</a></p>
        <p><a  target="_blank" href="https://www.comicat.org/">漫猫</a></p>
        <p><a  target="_blank" href="http://www.kisssub.org/">爱恋</a></p>
        <p>By <a target="_blank" href="https://blog.amzayo.com/">amzayo</a></p>`, {
          dangerouslyUseHTMLString: true
        });
      }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

body {
  min-width: 996px;
  background-image: url('https://jihulab.com/amzayo/1/-/raw/main/img/background/web.webp');
}

h1,
h3 {
  color: aliceblue;
  text-align: center;
}

#trsbtn,
#about {
  float: right;
}

.top20px {
  margin-top: 20px;
}

.seletTip {
  line-height: 40px;
  font-size: 14px;
  color: white;
}

.opc {
  display: flex;
  justify-content: space-between;
}
</style>
