<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          controls
          
          :src="url"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="icon" alt="" />
          </div>
          <span class="name">{{info.artistName}}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{info.name}}</h2>
          <span class="date">发布：{{info.publishTime}}</span>
          <span class="number">播放：{{info.playCount}}次</span>
          <p class="desc">
            {{info.desc}}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap">
        <p class="title">精彩评论<span class="number">({{hotNum}})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item , index) in hotComments" :key="index" >
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}：</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                <span class="comment">{{item.beReplied[0].content}}</span>
              </div>
              
            </div>
          </div>
        </div>
      </div>
      <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">最新评论<span class="number">({{total}})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item ,index) in comments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}：</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                <span class="comment">{{item.beReplied[0].content}}</span>
              </div>
              
            </div>
          </div>
        </div>
      </div>
      <!-- 分页器 -->
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="5"
        :limit="limit"
      >
      </el-pagination>
    </div>
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div class="item" v-for="(item , index) in mvs" :key="index">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{item.playCount}}</div>
              </div>
              <span class="time">{{item.duration}}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{item.name}}</div>
              <div class="singer">{{item.artistName}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
//导入axios
import axios from 'axios'
export default {
  name: 'mv',
  data() {
    return {
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      //mv地址
      url:'',
      //相关mv
      mvs:[],
      //mv信息
      info:{},
      //歌手封面
      icon:'',
      //热门评论
      hotComments:[],
      //热评数量
      hotNum:'',
      //最新评论
      comments:[],
      //评论数量
      total:0
    };
  },
  created(){
    //MV地址
    axios({
        url:'https://autumnfish.cn/mv/url',
        method:'get',
        params:{
          //获取url中的id
          id :this.$route.query.q
        }
      }).then(res=>{
        //console.log(res)
        this.url = res.data.data.url
      });
    //相关MV
    axios({
        url:'https://autumnfish.cn/simi/mv',
        method:'get',
        params:{
          mvid :this.$route.query.q
        }
      }).then(res=>{
        //console.log(res)
        this.mvs = res.data.mvs
        for(let i=0;i<this.mvs.length;i++){
          let min = parseInt( this.mvs[i].duration/1000/60);
          let sec = parseInt(this.mvs[i].duration/1000%60);
          if(min<10){
            min = '0' + min
          };
          if(sec<10){
            sec = '0' + sec
          };
          this.mvs[i].duration = min + ':' + sec;
          if(this.mvs[i].playCount>10000){
            this.mvs[i].playCount = parseInt(this.mvs[i].playCount/10000) +'万'
          };
        }
      })
    //mv信息
    axios({
      url:'https://autumnfish.cn/mv/detail',
      method:'get',
      params:{
        mvid : this.$route.query.q
        }
    }).then(res=>{
      //console.log(res)
      this.info = res.data.data
      //获取歌手信息
      axios({
          url:'https://autumnfish.cn/artists',
          method:'get',
          params:{
            id:this.info.artists[0].id
          }
        }).then(res=>{
          //console.log(res)
          //歌手的封面信息
          this.icon = res.data.artist.picUrl
        })
    })
  //评论
  axios({
        url:'https://autumnfish.cn/comment/mv',
        method:'get',
        params:{
          id:this.$route.query.q,
          limit:10,
          offset:0
        }
      }).then(res=>{
        console.log(res);
        this.hotComments = res.data.hotComments;
        this.hotNum = res.data.hotComments.length;
        this.comments = res.data.comments;
        this.total = res.data.total
      })
    
  },
  methods: {
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      //保存当前页
      this.page = val
     axios({
        url:'https://autumnfish.cn/comment/mv',
        method:'get',
        params:{
          id:this.$route.query.q,
          limit:10,
          offset:(this.page-1)*10
        }
      }).then(res=>{
        console.log(res);
        this.comments = res.data.comments;
      })

    }
  }
};
</script>

<style></style>
