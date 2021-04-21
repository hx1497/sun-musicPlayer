<template>
  <div class="discovery-container">
    <!-- 轮播图 -->
    <el-carousel class="" :interval="4000" type="card">
      <!-- 循环获取到的接口数据 -->
      <el-carousel-item v-for="(item , index) in banners" :key="index">
        <img :src="item.imageUrl" alt="" />
      </el-carousel-item>
    </el-carousel>
    <!-- 推荐歌单 -->
    <div class="recommend">
      <h3 class="title">
        推荐歌单
      </h3>
      <div class="items">
        <div class="item" v-for="(item , index) in list" :key='index' @click="toPlaylist(item.id)">
          <div class="img-wrap">
            <div class="desc-wrap">
              <span class="desc">{{ item.copywriter }}</span>
            </div>
            <img :src="item.picUrl" alt="" />
            <span class="iconfont icon-play"></span>
          </div>
          <p class="name">{{ item.name }}</p>
        </div>
      </div>
    </div>
    <!-- 最新音乐 -->
    <div class="news">
      <h3 class="title">
        最新音乐
      </h3>
      <div class="items">
        <div class="item" v-for="(item , index) in songs" :key="index">
          <div class="img-wrap">
            <!-- 封面 -->
            <img :src="item.picUrl" alt="" />
            <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
          </div>
          <div class="song-wrap">
            <!-- 歌名 -->
            <div class="song-name">{{ item.name }}</div>
            <!-- 歌手名 -->
            <div class="singer">{{ item.song.artists[0].name }}</div>
          </div>
        </div>
      </div>
    </div>
    <!-- 推荐MV -->
    <div class="mvs">
      <h3 class="title">推荐MV</h3>
      <div class="items">
        <div class="item" @click="toMv(item.id)" v-for="(item , index) in mvs" :key="index">
          <div class="img-wrap">
            <img :src="item.picUrl" alt="" />
            <span class="iconfont icon-play"></span>
            <div class="num-wrap">
              <div class="iconfont icon-play"></div>
              <div class="num">{{ item.playCount }}</div>
            </div>
          </div>
          <div class="info-wrap">
            <div class="name">{{ item.name }}</div>
            <div class="singer">{{ item.artistName }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// 导入 axios
import axios from 'axios'
export default {
  name: 'discovery',
  data(){
    return {
      //轮播图
      banners:[],
      //推荐歌单
      list:[],
      //最新音乐
      songs:[],
      //推荐mv
      mvs:[]
    }
  },
    /* 
      axios({
        url:'',
        method:'get',
        params:{}
      }).then(res=>{})
    */
  created(){
    //console.log('created')
    //轮播图接口
    axios({
      url: 'https://autumnfish.cn/banner',
      method: 'get',
      //params:{}
    }).then(res=>{
      //console.log(res)
      this.banners = res.data.banners
    })

    //推荐歌单
    axios({
      url: 'https://autumnfish.cn/personalized',
      method: 'get',
      params: {
        //获取的数据量
        limit : 10
      }
    }).then(res=>{
      console.log(res)
      this.list = res.data.result
    })

    //最新音乐
    axios({
      url: 'https://autumnfish.cn/personalized/newsong',
      method: 'get',
      //params: {}
    }).then(res=>{
      //console.log(res)
      this.songs = res.data.result
    })

    //推荐mv
    axios({
        url:'https://autumnfish.cn/personalized/mv',
        method:'get',
        //params:{}
      }).then(res=>{
        /* console.log(res) */
        this.mvs = res.data.result
      })
  },
  methods:{
     toPlaylist(id){
      //跳转并携带数据即可  push后接/+要转向的页面名称+?+键值对（键随意设置，值为传递的参数）
      this.$router.push(`/playlist?q=${id}`)
    },
    //播放音乐函数
    playMusic(id){
      //console.log(id)
      axios({
        url:'https://autumnfish.cn/song/url',
        method:'get',
        params:{
          id//id:id
        }
      }).then(res=>{
        //console.log(res)
        let url = res.data.data[0].url
        //console.log(this.$parent.musicUrl)
        //设置给父组件的播放地址
        this.$parent.musicUrl = url 
      })
    },

    //播放mv
    toMv(id){
      this.$router.push(`/mv?q=${id}`);
    }
  }
};
</script>

<style >

</style>
