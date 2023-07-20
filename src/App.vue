<template>
  <div id="app" class="container">
    <el-row type="flex" justify="center" class="top15px">
      <el-col>
        <h1>动漫RSS订阅提取下载地址</h1>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top15px">
      <el-col>
        <el-input placeholder="请输入RSS链接" v-model="link" clearable></el-input>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top15px">
      <el-col>
        <el-row type="flex" justify="space-between">
          <el-col :span="7">
            <el-input placeholder="最多生成几条" v-model="count" clearable></el-input>
          </el-col>
          <el-col :span="7" style="text-align: center;">
            <el-select v-model="type" placeholder="" class="left30px" @change="trs" style="width: 100%">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
              </el-option>
            </el-select>
          </el-col>
          <el-col :span="7">
            <el-button type="primary" id="trsbtn" @click="trs" style="width: 100%">点击生成</el-button>
          </el-col>
        </el-row>
      </el-col>
    </el-row>
    <el-row class="top15px">
      <el-col>
        <div @click="showOrHid">
          <h3>点击<spn v-if="filterBoxStatus">折叠</spn><spn v-if="!filterBoxStatus">展开</spn>筛选</h3>
          <h3>由于各字幕组命名标准不同，筛选结果仅供参考</h3>
        </div>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="md20px">
      <el-col>
        <el-collapse-transition>
          <div v-show="filterBoxStatus" class="filterBox" ref="filterBox">
            <el-row type="flex" justify="center">
              <el-col class="hidden-lg-and-up">
                <el-input placeholder="可以手动输入关键字过滤，使用空格隔开" v-model="customerFilter.inputString" clearable
                  v-on:input="filter"></el-input>
              </el-col>
            </el-row>
            <el-row type="flex" justify="center" class="top15px">
              <el-col>
                <el-row type="flex" justify="space-between">
                  <el-col :span="11">
                    <el-select @change="filter" v-model="resolution.selected" collapse-tags multiple placeholder="分辨率"
                      style="width: 100%">
                      <el-option v-for="item in resolution.option" :key="item.value" :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                  </el-col>
                  <el-col :span="11">
                    <el-select @change="filter" v-model="subLan.selected" collapse-tags multiple placeholder="字幕语言"
                      style="width: 100%">
                      <el-option v-for="item in subLan.option" :key="item.value" :label="item.label" :value="item.value">
                      </el-option>
                    </el-select>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <el-row type="flex" justify="center" class="top15px">
              <el-col>
                <el-row type="flex" justify="space-between">
                  <el-col :span="11">
                    <el-select @change="filter" v-model="subEM.selected" collapse-tags multiple placeholder="字幕嵌入方式"
                      style="width: 100%">
                      <el-option v-for="item in subEM.option" :key="item.value" :label="item.label" :value="item.value">
                      </el-option>
                    </el-select>
                  </el-col>
                  <el-col :span="11">
                    <el-select @change="filter" v-model="compilations.selected" collapse-tags multiple placeholder="只显示合集"
                      style="width: 100%">
                      <el-option v-for="item in compilations.option" :key="item.value" :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </div>
        </el-collapse-transition>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top15px">
      <el-col>
        <el-table class="table" ref="multipleTable" :data="showDatas" height="table" tooltip-effect="dark"
          style="width: 100%" @selection-change="handleSelectionChange" :row-key="showDatas.links">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="title" label="标题">
            <template slot="header">
              <el-row>
                <el-col :span="4" style="line-height: 28px;">
                  标题
                </el-col>
                <el-col :span="18">
                  <el-input class="hidden-md-and-down" v-model="customerFilter.inputString" v-on:input="filter"
                    size="mini" placeholder="可以手动输入关键字过滤，使用空格隔开" />
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column prop="link" label="下载地址" show-overflow-tooltip width="100"></el-table-column>
        </el-table>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="top15px">
      <el-col>
        <el-input type="textarea" :rows="2" placeholder="如果复制失败，您可以在这里手动复制结果" v-model="opt">
        </el-input>
      </el-col>
    </el-row>
    <el-row type="flex" justify="space-around" class="top15px">
      <el-col :span="23">
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
import 'element-ui/lib/theme-chalk/display.css';
export default {
  name: 'App',
  data() {
    return {
      link: '',
      originDatas:[],
      showDatas: [],
      width:0,
      filterBoxStatus:true,
      opt: "",
      count:'',
      customerFilter:{inputString:'',keywords:[]},
      options: [{
          value: 'mikan',
          label: '蜜柑动漫'
        },{
          value: 'dmhy',
          label: '动漫花园'
        },{
          value: 'bangumi',
          label: '萌番组'
        },{
          value: 'acgrip',
          label: 'acgrip'
        },{
          value: 'comicat',
          label: '漫猫动漫'
        },{
          value: 'kiss',
          label: '爱恋动漫'
        },{
          value: 'nyaa',
          label: 'nyaa'
        }],
        //解析分类：
        //蜜柑/动漫花园/萌番组/acgrip
        //漫猫/爱恋
        //nyaa
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
      },
      compilations:{
        option:[{
        value: '合集',
        label: '是'
      }],
      selected:[]
      },
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
  url: `${process.env.VUE_APP_API_URL}`,
  method: "GET",
  dataType: "json",
  data: {
    url: `${decodeURI(this.link)}`,
    count:`${this.count || 10000}`
  },
}).done((response) => {
  const tempOriginDatas = [];
  const tempShowDatas = [];
  for (let i in response.items) {
    const tempObj = {};
    if(this.type === 'mikan' || this.type === 'dmhy' || this.type === 'bangumi' || this.type === 'acgrip'){
      tempObj.title = response.items[i].title;
      if(this.type === 'bangumi'){//萌番组链接可能含中文，需要编码
        tempObj.link = encodeURI(response.items[i].enclosures[0].url);
      }else{
        tempObj.link = response.items[i].enclosures[0].url;
      }
    }else if(this.type === 'comicat' || this.type === 'kiss'){
      // 拆分获取到的链接，后续拼接成能用的种子地址
      const templink = response.items[i].enclosures[0].url.split("hash=")
      tempObj.title = response.items[i].title;
      tempObj.link = `magnet:?xt=urn:btih:${templink[templink.length-1]}`;
    }else if(this.type ==='nyaa'){
      tempObj.title = response.items[i].title;
      tempObj.link = response.items[i].link;;
    }
    tempOriginDatas.push(tempObj)
    tempShowDatas.push(tempObj)
  }
  this.originDatas = tempOriginDatas;
  this.showDatas = tempShowDatas;
  this.filter();
  this.$message({
    type: 'success',
    message: `下载地址提取成功`
  });
}).fail((response) => {
  this.$message({
    type: 'error',
    message: `出错了，可能是链接不正确或者后端抽风了`
  });
});
    },    
    // 条件筛选
    filter(){
      this.customerFilter.keywords = this.customerFilter.inputString.split(" ");

      const customerFilterPattern = new RegExp(this.customerFilter.keywords.join('|'), 'i');
      const resolutioncPattern = new RegExp(this.resolution.selected.join('|'), 'i');
      const subLanPattern = new RegExp(this.subLan.selected.join('|'), 'i');
      const subEMPattern = new RegExp(this.subEM.selected.join('|'), 'i');
      const compilationsPattern = new RegExp(this.compilations.selected.join('|'), 'i');
      this.showDatas = this.originDatas.filter((item) => {
        let t = item.title
          if(resolutioncPattern.test(t) && subLanPattern.test(t) && subEMPattern.test(t) && compilationsPattern.test(t) && customerFilterPattern.test(t)){
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
    this.showDatas.forEach(row => {this.width
    this.$refs.multipleTable.toggleRowSelection(row);
  })
},
// 折叠框
showOrHid(){
      const filterBox = document.getElementsByClassName('filterBox')[0];
      this.filterBoxStatus = !this.filterBoxStatus;
      if(this.width <= 1199 ){
        if(this.filterBoxStatus === false){
          filterBox.style.marginTop = 0 +'px';
      }else{
          filterBox.style.marginTop = 20 +'px';
      }
    }else if(this.width >= 1199 && this.filterBoxStatus === true){
      filterBox.style.marginTop = 0 +'px';
    }
},
    updateViewportWidth() {
      this.width = window.innerWidth;
    },
  

  //弹出框

open() {
        this.$alert(`<p>前端项目：<a  target="_blank" href="https://github.com/petra1026/animation_rss_downloadlink_extraction">animation_rss_downloadlink_extraction</a></p>
        <p>后端项目：<a  target="_blank" href="https://github.com/ayusharma/RSS-to-JSON">RSS-to-JSON</a></p>
        <p>目前支持解析的网站：</p>
        <p><a  target="_blank" href="https://mikanani.me/">蜜柑计划</a></p>
        <p><a  target="_blank" href="https://share.dmhy.org/">动漫花园</a></p>
        <p><a  target="_blank" href="https://bangumi.moe/">萌番组</a></p>
        <p><a  target="_blank" href="https://acg.rip/">acgrip</a></p>
        <p><a  target="_blank" href="https://www.comicat.org/">漫猫动漫</a></p>
        <p><a  target="_blank" href="http://www.kisssub.org/">爱恋动漫</a></p>
        <p><a  target="_blank" href="https://nyaa.si">nyaa</a></p>
        <p>By <a target="_blank" href="https://amzayo.com/">amzayo</a></p>`, {
          dangerouslyUseHTMLString: true
        });
      },
  },
  mounted() {
    this.updateViewportWidth();
    this.$refs.filterBox.style.transition = 'all 0.5s';
    window.addEventListener('resize', this.updateViewportWidth);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.updateViewportWidth);
  },
  watch:{
    width(newValue){
      const filterBox = document.getElementsByClassName('filterBox')[0];
      if(newValue >= 1199){
        this.filterBoxStatus = true;
        filterBox.style.marginTop = 0 +'px';
      }else{
        this.filterBoxStatus = false;
        filterBox.style.marginTop = 20 +'px';
      }
    }
  }
}
</script>

<style>
@media screen and (min-width: 1200px) {
  .container {
    width: 1127px;
    margin-left: auto !important;
    margin-right: auto !important;
  }

  .table {
    height: 600px;
  }
}

@media screen and (min-width: 922px) and (max-width:1199px) {
  .container {
    width: 880px;
    margin-left: auto !important;
    margin-right: auto !important;
  }

  .table {
    height: 600px;
  }

}

@media screen and (min-width: 768px) and (max-width:922px) {
  .container {
    width: 723px;
    margin-left: auto !important;
    margin-right: auto !important;
  }

  .table {
    height: 500px;
  }
}

@media screen and (max-width: 768px) {
  .container {
    width: 100%;
    padding: 0px 10px 0 10px !important;
    box-sizing: border-box;

  }

  .table {
    height: 430px;
  }

}

* {
  margin: 0;
  padding: 0;
  transition: all .5s;
}

body {
  /* min-width: 996px; */
  background-image: url('https://jihulab.com/amzayo/1/-/raw/main/img/background/web.webp');
}

h1,
h3 {
  color: aliceblue;
  text-align: center;
}

h1 {
  font-size: 30px;
}

h3 {
  font-size: 14px;
}

#trsbtn,
#about {
  float: right;
}

.top15px {
  margin-top: 15px;
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
