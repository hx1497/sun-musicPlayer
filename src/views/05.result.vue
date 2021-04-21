<template>
  <div class="result-container">
    <div class="title-wrap">
      <!-- 标题 -->
      <h2 class="title">{{ this.$route.query.q }}</h2>
      <span class="sub-title">找到{{ count }}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr 
            v-for="(item , index) in songlist" 
            :key="index" 
            class="el-table__row"
            @dblclick="playMusic(item.id)"
            >
              <td>{{index + 1}}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <!-- 歌曲名 -->
                    <span>{{ item.name}}</span>
                    <!-- mv的图标 -->
                    <span v-if="item.mvid!=0" class="iconfont icon-mv"></span>
                  </div>
                  <span v-if="item.alias.length!=0">{{item.alias[0]}}</span>
                </div>
              </td>
              <!-- 歌手名 -->
              <td>{{item.artists[0].name}}</td>
              <!-- 专辑名 -->
              <td>{{ item.album.name}}</td>
              <!-- 第二名称 -->
              <td>{{item.duration}}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item,index) in playlist" :key="index" @click="toPlaylist(item.id)">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{item.playCount}}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{item.name}}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item , index) in mvlist" :key="index" @click="toMv(item.id)">
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
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
//导入axios
import axios from 'axios'
export default {
  name: 'result',
  data() {
    return {
      //tag切换时改变的数据
      activeIndex: 'songs',
      //保存 查询歌曲
      songlist:[],
      //保存 查询歌单
      playlist:[],
      //保存 查询mv
      mvlist:[],
      //搜索结果数
      count:0
    };
  },
  //生命周期钩子  回调函数
  created(){
    axios({
        url:'https://autumnfish.cn/search',
        method:'get',
        params:{
          keywords:this.$route.query.q,
          type:1,
          limit:10
        }
      }).then(res=>{
        //console.log(res)
        this.songlist = res.data.result.songs
        //计算歌曲时间
        for(let i = 0 ; i < this.songlist.length ; i++){
          let min = parseInt(this.songlist[i].duration/1000/60);
          let sec = parseInt(this.songlist[i].duration/1000%60);
          if(min<10){
            min = '0' + min
          };
          if(sec<10){
            sec = '0' + sec
          };
          this.songlist[i].duration = min + ':' + sec
          //console.log(this.songlist[i].duration)
        }
        //保存总数
        this.count = res.data.result.songCount
      })
  },
  //tab切换时的侦听器
  watch:{
    activeIndex(){
      //songs  歌曲
      //lists  歌单
      //mv     mv
      let type = 1;
      let limit = 10;
      switch (this.activeIndex) {
        case 'songs':
          type = 1;
          limit = 10;
          break;
        case 'lists':
          type = 1000;
          limit = 10;
          break;  //在执行操作后在break处终止,每条判断后都要加break，否则一直直执行到有break为止
        case 'mv':
          type = 1004;
          limit = 8;
          break;
      
        default:
          break;
      };

      axios({
        url:'https://autumnfish.cn/search',
        method:'get',
        params:{
          keywords:this.$route.query.q,
          type ,//type:type,
          limit //limit:limit
        }
      }).then(res=>{
        console.log(res)
        //判断是歌曲页、歌单页还是mv页
        if(type==1){
          this.songlist = res.data.result.songs
          //计算歌曲时间
          for(let i = 0 ; i < this.songlist.length ; i++){
            let min = parseInt(this.songlist[i].duration/1000/60);
            let sec = parseInt(this.songlist[i].duration/1000%60);
            if(min<10){
              min = '0' + min
            };
            if(sec<10){
              sec = '0' + sec
            };
            this.songlist[i].duration = min + ':' + sec
            //console.log(this.songlist[i].duration)
          }
          //保存总数
          this.count = res.data.result.songCount
        }else if(type==1000){
          //歌单的逻辑
          this.playlist = res.data.result.playlists
          this.count = res.data.result.playlistCount
          for(let i = 0 ; i<this.playlist.length ; i++){
            if(this.playlist[i].playCount>10000){
              this.playlist[i].playCount = parseInt(this.playlist[i].playCount/10000) + '万'
            }
          }
        }else if(type==1004){
          this.mvlist = res.data.result.mvs;
          this.count = res.data.result.mvCount
          for(let i = 0 ; i < this.mvlist.length ; i++){
            let min = parseInt(this.mvlist[i].duration/1000/60);
            let sec = parseInt(this.mvlist[i].duration/1000%60);
            if(min<10){
              min = '0' + min
            };
            if(sec<10){
              sec = '0' + sec
            };
            this.mvlist[i].duration = min + ':' + sec;
            if(this.mvlist[i].playCount>10000){
              this.mvlist[i].playCount = parseInt(this.mvlist[i].playCount/10000) + '万'
            }
            }
        } 
      })
    }
  },
  //方法
  methods:{
    //去歌单详情页
    toPlaylist(id){
      //跳转并携带数据即可  push后接/+要转向的页面名称+?+键值对（键随意设置，值为传递的参数）
      this.$router.push(`/playlist?q=${id}`)
    },
    //去MV详情页
    toMv(id){
      this.$router.push(`/mv?q=${id}`)
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
    }   
  }
};
</script>

<style >

</style>
