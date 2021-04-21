<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <!-- 封面 -->
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <!-- 名字 -->
        <p class="title">{{playlist.name}}</p>
        <div class="author-wrap">
          <img class="avatar" :src="playlist.creator.avatarUrl" alt="" />
          <span class="name">{{playlist.creator.nickname}}</span>
          
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <!-- 分类标签 -->
          <ul>
            <li v-for="(item,index) in tags" :key="index">
              {{item}}
            </li>
            
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <!-- 简介 -->
          <span class="desc"
            >{{playlist.description}}</span
          >
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr 
            class="el-table__row" 
            v-for="(item , index) in songlist" :key="index"
            @dblclick="playMusic(item.id)"
            >
              <td>{{index + 1}}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <span v-if="item.mv!=0" class="iconfont icon-mv"></span>
                  </div>
                  <span v-if="item.alia.length!=0">{{item.alia[0]}}</span>
                </div>
              </td>
              <td>{{item.ar[0].name}}</td>
              <td>{{item.al.name}}</td>
              <td>{{item.dt}}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="评论(66)" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">
            精彩评论
            <span class="number">{{hotCount}}</span>
            </p>
          <div class="comments-wrap">
            <!-- 评论 -->
            <div v-for="(item ,index) in hotComments" :key="index" class="item">
              <div class="icon-wrap">
                <!-- 头像 -->
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <!-- 昵称 -->
                  <span class="name">{{ item.user.nickname }}：</span>
                  <!-- 内容 -->
                  <span class="comment">{{ item.content }}</span>
                </div>
                <!-- 评论回复 -->
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
               
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论
            <span class="number">
              ( {{total}} )
            </span>
            </p>
          <div class="comments-wrap">
            <div class="item" v-for="(item , index) in comments" :key="index" >
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <!-- 昵称 -->
                  <span class="name">{{ item.user.nickname }}：</span>
                  <!-- 评论内容 -->
                  <span class="comment">{{ item.content }}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">  
                  <!-- 回复昵称 -->
                  <span class="name">
                    {{item.beReplied[0].user.nickname}}：
                    </span>
                  <!-- 回复内容 -->
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
        >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'playlist',
  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      //歌单详情
      playlist: {},
      //歌单标签
      tags:[],
      //歌单的歌曲列表
      songlist:[],
      //歌单的热门评论列表
      hotComments:[],
      //热评个数
      hotCount:0,
      //评论列表
      comments:[]

    };
  },
  created(){
    //获取歌单详情
     axios({
        url:' https://autumnfish.cn/playlist/detail',
        method:'get',
        params:{
          id:this.$route.query.q
        }
      }).then(res=>{
        //console.log(res)
        this.playlist = res.data.playlist
        this.tags = res.data.playlist.tags
        this.songlist = res.data.playlist.tracks
        /* this.playlist.createTime = new Date( this.playlist.createTime*1000 ) ;
        this.playlist.createTime.toLocaleString(); */
        
        for(let i = 0 ; i<this.songlist.length ; i++){
          let min = parseInt(this.songlist[i].dt/1000/60);
          let sec = parseInt(this.songlist[i].dt/1000%60);
          if(min<10){
            min = '0' + min
          };
          if(sec<10){
            sec = '0' + sec
          };
          this.songlist[i].dt = min + ':' + sec
          //console.log(this.songlist[i].duration)
        }
      })

    //获取 热门评论
    axios({
        url:'https://autumnfish.cn/comment/hot',
        method:'get',
        params:{
          //从router传输的数据
          id:this.$route.query.q,
          //limit:5,
          //传递类型
          type:2
        }
      }).then(res=>{
        //console.log(res)
        this.hotComments = res.data.hotComments
        //保存热评个数
        this.hotCount = res.data.total
      })

      //获取最新评论
      axios({
        url:'https://autumnfish.cn/comment/playlist',
        method:'get',
        params:{
          id:this.$route.query.q,
          limit:10,
          offset:0
        }
      }).then(res=>{
        console.log(res)
        //总个数
        this.total = res.data.total
        //评论数据
        this.comments = res.data.comments
      })
  },
  methods: {
    handleCurrentChange(val) {
      //console.log(`当前页: ${val}`);
      //保存页码
      this.page = val
      //重新获取数据
       axios({
        url:'https://autumnfish.cn/comment/playlist',
        method:'get',
        params:{
          id:this.$route.query.q,
          limit:10,
          //根据页码计算
          offset:(this.page - 1)*10
        }
      }).then(res=>{
        console.log(res)
        //总个数
        this.total = res.data.total
        //评论数据
        this.comments = res.data.comments
      })
    },
    //播放音乐
    playMusic(id){
       axios({
        url:'https://autumnfish.cn/song/url',
        method:'get',
        params:{
          id //id:id
        }//将id传递给该地址后会获取一个播放地址
      }).then(res=>{
        //console.log(res)
        let url = res.data.data[0].url //将获取到的地址保存下来
        //console.log(this.$parent.musicUrl)  this.$parent.XXX 可以访问到父组件的某一变量 该变量可在此赋值
        this.$parent.musicUrl = url

      })
    }
  }
};
</script>

<style >

</style>
