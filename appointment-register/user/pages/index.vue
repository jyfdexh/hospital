<template>
  <div class="home page-component">
    <el-carousel indicator-position="outside">
      <el-carousel-item v-for="item in 2" :key="item">
        <img src="~assets/images/web-banner1.png" alt="">
      </el-carousel-item>
    </el-carousel>

    <!-- 搜索 -->
    <!-- 搜索 -->
    <div class="search-container">
      <div class="search-wrapper">
        <div class="hospital-search">
          <el-autocomplete
            class="search-input"
            prefix-icon="el-icon-search"
            v-model="hosname"
            :fetch-suggestions="querySearchAsync"
            :trigger-on-focus="false"
            @select="handleSelect"
            placeholder="点击输入医院名称"
          >
            <span slot="suffix" class="search-btn v-link highlight clickable selected">搜索 </span>
          </el-autocomplete>
        </div>
      </div>
    </div>

    <!-- bottom -->
    <div class="bottom">
      <div class="left">
        <div class="home-filter-wrapper">
          <div class="title"> 医院</div>
          <div>
            <div class="filter-wrapper">
              <span class="label">等级：</span>
              <div class="condition-wrapper">
                <span class="item v-link clickable" :class="hostypeActiveIndex == index ? 'selected' : ''" v-for="(item,index) in hostypeList" :key="item.id" @click="hostypeSelect(item.value, index)">{{ item.name }}</span>
              </div>
            </div>
            <div class="filter-wrapper">
              <span class="label">地区：</span>
              <div class="condition-wrapper">
                <span class="item v-link clickable" :class="provinceActiveIndex == index ? 'selected' : ''" v-for="(item,index) in districtList" :key="item.id" @click="districtSelect(item.value, index)">{{ item.name }}</span>
              </div>
            </div>
          </div>
        </div>
        <div class="v-scroll-list hospital-list">
          <div class="v-card clickable list-item" v-for="item in list" :key="item.id">
            <div class="">
              <div class="hospital-list-item hos-item" index="0" @click="show(item.hoscode)">
                <div class="wrapper">
                  <div class="hospital-title"> {{ item.hosname }}</div>
                  <div class="bottom-container">
                    <div class="icon-wrapper">
                      <span class="iconfont"></span>{{ item.param.hostypeString }}
                    </div>
                    <div class="icon-wrapper">
                      <span class="iconfont"></span>每天{{ item.bookingRule.releaseTime }}放号
                    </div>
                  </div>
                </div>
                <img :src="'data:image/jpeg;base64,'+item.logoData" :alt="item.hosname" class="hospital-img">
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="right">
        <div class="common-dept">
          <div class="header-wrapper">
            <div class="title"> 常见科室</div>
            <div class="all-wrapper"><span>全部</span>
              <span class="iconfont icon"></span>
            </div>
          </div>
          <div class="content-wrapper">
            <span class="item v-link clickable dark">神经内科 </span>
            <span class="item v-link clickable dark">消化内科 </span>
            <span class="item v-link clickable dark">呼吸内科 </span>
            <span class="item v-link clickable dark">内科 </span>
            <span class="item v-link clickable dark">神经外科 </span>
            <span class="item v-link clickable dark">妇科 </span>
            <span class="item v-link clickable dark"> 产科 </span>
            <span class="item v-link clickable dark">儿科 </span>
          </div>
        </div>
        <div class="space">
          <div class="header-wrapper">
            <div class="title-wrapper">
              <div class="icon-wrapper"><span
                class="iconfont title-icon"></span>
              </div>
              <span class="title">平台公告</span>
            </div>
            <div class="all-wrapper">
              <span>全部</span>
              <span class="iconfont icon"></span>
            </div>
          </div>
          <div class="content-wrapper">
            <div class="notice-wrapper">
              <div class="point"></div>
              <span class="notice v-link clickable dark">关于延长北京大学国际医院放假的通知 </span>
            </div>
            <div class="notice-wrapper">
              <div class="point"></div>
              <span class="notice v-link clickable dark">北京中医药大学东方医院部分科室医生门诊医 </span>
            </div>
            <div class="notice-wrapper">
              <div class="point"></div>
              <span class="notice v-link clickable dark"> 武警总医院号源暂停更新通知 </span>
            </div>
          </div>
        </div>
        <div class="suspend-notice-list space">
          <div class="header-wrapper">
            <div class="title-wrapper">
              <div class="icon-wrapper">
                <span class="iconfont title-icon"></span>
              </div>
              <span class="title">停诊公告</span>
            </div>
            <div class="all-wrapper">
              <span>全部</span>
              <span class="iconfont icon"></span>
            </div>
          </div>
          <div class="content-wrapper">
            <div class="notice-wrapper">
              <div class="point"></div>
              <span class="notice v-link clickable dark"> 中国人民解放军总医院第六医学中心(原海军总医院)呼吸内科门诊停诊公告 </span>
            </div>
            <div class="notice-wrapper">
              <div class="point"></div>
              <span class="notice v-link clickable dark"> 首都医科大学附属北京潞河医院老年医学科门诊停诊公告 </span>
            </div>
            <div class="notice-wrapper">
              <div class="point"></div>
              <span class="notice v-link clickable dark">中日友好医院中西医结合心内科门诊停诊公告 </span>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import hospitalApi from '@/api/hosp/hospital'
import dictApi from '@/api/cmn/dict'
export default {
  // asyncData：渲染组件之前异步获取数据
  asyncData({ params, error }) {
    return hospitalApi.getPageList(1, 10, null).then(response => {
      console.log(response.data);
      return {
        list: response.data.content,
        pages : response.data.totalPages
      }
    });
  },
  data() {
    return {
      searchObj: {},
      page: 1,
      limit: 10,

      hosname: '',
      hostypeList: [],
      districtList: [],

      hostypeActiveIndex: 0,
      provinceActiveIndex: 0
    }
  },

  created() {
    this.init()
  },

  mounted() {
    document.getElementById("search").style.display = 'none';

    // 添加滚动事件，检测滚动到页面底部
    window.addEventListener('scroll', this.load, true)
  },

  destroyed(){
    // 页面关闭的同时，记得将这个监听器关闭，节省性能
    window.removeEventListener('scroll', this.load, false)
  },

  methods: {
    init() {
      dictApi.findByDictCode("Hostype").then(response => {
        this.hostypeList = []
        this.hostypeList.push({"name":"全部", "value":""})
        for(let i in response.data){
          this.hostypeList.push(response.data[i]);
        }
      })
      dictApi.findByDictCode('Beijin').then(response => {
        this.districtList = []
        this.districtList.push({"name":"全部", "value":""})
        for(let i in response.data){
          this.districtList.push(response.data[i]);
        }
      })
    },

    load(event) {
      // 滚动条高度为430 页面搜索消失，头部搜索显示
      if(event.target.scrollTop > 430) {
        document.getElementById("search").style.display = 'block';
      } else {
        document.getElementById("search").style.display = 'none';
      }

      if(event.target.clientHeight + event.target.scrollTop >= event.target.scrollHeight){
        if(this.page < this.pages){      //先判断下一页是否有数据
          this.page = this.page + 1
          this.getList();              //拉取接口数据
        }
      }
    },

    getList() {
      hospitalApi.getPageList(this.page, this.limit, this.searchObj).then(response => {
        for(let i in response.data.content){
          this.list.push(response.data.content[i]);
        }
        this.pages = response.data.totalPages
      })
    },

    querySearchAsync(queryString, cb) {
      this.searchObj = []
      if(queryString == '') return
      hospitalApi.getByHosname(queryString).then(response => {
        for (let i = 0, len = response.data.length; i < len; i++) {
          response.data[i].value = response.data[i].hosname
        }
        cb(response.data)
      })
    },

    handleSelect(item) {
      window.location.href = '/hospital/' + item.hoscode
    },

    hostypeSelect(hostype, index) {
      this.list = []
      this.page = 1
      this.hostypeActiveIndex = index
      this.searchObj.hostype = hostype
      this.getList();
    },

    districtSelect(districtCode, index) {
      this.list = []
      this.page = 1
      this.provinceActiveIndex = index
      this.searchObj.districtCode = districtCode
      this.getList();
    },

    show(hoscode) {
      window.location.href = '/hospital/' + hoscode
    }
  }
}
</script>

