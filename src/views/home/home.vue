<template>
  <el-container>
    <el-header>
      <h1>vue后台管理框架基础模板-面包屑版</h1>
      <div class="person-center">
          <span class="head-img"><i></i>{{name}}</span>
          <a href="javascript:void(0);" class="login-out" @click="loginOut">退出</a>
      </div>
    </el-header>
    <el-container>
      <el-aside width="240px">
        <el-menu default-active="0-0-t" class="el-menu-vertical-demo" :collapse="isCollapse" background-color="#545c64" text-color="#fff" active-text-color="#ffd04b">
          <!-- 一级 -->
          <el-submenu v-for="(nav1,index1) in menus" :index="getIndex(index1)" :key="nav1.id">

            <template slot="title">
              <i :class="nav1.entity.icon"></i>
              <span slot="title">{{nav1.entity.alias}}</span>
            </template>

            <!-- 二级 -->
            <div v-for="(nav2,index2) in nav1.childs" :key="nav2.id">
              <!--含有三级的二级 -->
              <el-submenu v-if="nav2.childs" :index="getIndex(index1,index2,'o')" :key="nav2.id">
                <template slot="title">
                  <span slot="title">{{nav2.entity.alias}}</span>
                </template>

                <!-- 三级 -->
                <el-menu-item-group v-for="(nav3,index3) in nav2.childs" :key="nav3.id">
                  <el-menu-item :class="setonlyClass(nav3.entity.id)" :index="getIndex(index1,index2,index3)" @click="changeTitle(nav3.entity.alias,nav3.entity.menu,nav3.entity.id)">{{nav3.entity.alias}}</el-menu-item>
                </el-menu-item-group>
              </el-submenu>

              <!--不含三级的二级-->
              <el-menu-item-group v-else>
                <el-menu-item :class="setonlyClass(nav2.entity.id)" :index="getIndex(index1,index2,'t')" @click="changeTitle(nav2.entity.alias,nav2.entity.menu,nav2.entity.id)">{{nav2.entity.alias}}</el-menu-item>
              </el-menu-item-group>

            </div>

          </el-submenu>

        </el-menu>
      </el-aside>
      <el-main>
        <div class="main-head-container">
          <span class="el-icon-arrow-left" v-if="islong" @click="goLeft"></span>
          <div class="main-head" :style="{paddingLeft:paddingleft,left:goleft,right:goright}">
            <el-tag
              v-for="tag in tags"
              :key="tag.name"
              closable
              :type="tag.type"
              @close="handleClose(tag)">
              <span @click="goThisPage(tag)">{{tag.main_title}}</span>
            </el-tag>
          </div>
          <span class="el-icon-arrow-right" v-if="islong" @click="goRight"></span>
        </div>
        <div class="main-page">
          <transition>
            <keep-alive>
              <router-view></router-view>
            </keep-alive>
          </transition>
        </div>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  let menu=require('../../menu.js');

  export default {
    name: 'home',
    data () {
        return {
          name:'',
          menus:menu.childs,
          isCollapse:false,
          showArray:['0'], //展开的index
          tags: [],
          len:0, //一个标签的宽度
          islong:false,
          paddingleft:'10px',
          goleft:0,//向左偏移的距离
          goright:0,//向右偏移的距离
        }
    },
    components:{

    },
    methods:{
      loginOut(){
        //退出登录
        this.$confirm('您将退出登录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // localStorage.removeItem('msk_hasLogin');
          this.$router.replace({path:'/login'});
        }).catch(() => {

        })
      },
      getIndex(index1,index2,index3){
        //赋予index唯一的标志
        let index='';
        if(index3){
          index = index1+'-'+index2+'-'+index3;
        }else if(index2){
          index = index1+'-'+index2;
        }else{
          index = index1;
        }
        index=index.toString();
        return index;
      },
      changeTitle(t,url,id){
        //home标签页面跳转
        //添加标签页面步骤：
        //------1，在menu.js中需要存在带menu属性含有值路径的值；
        //------2,添加标签页面的vue文件；
        //------3，在router中添加路径；
        this.$router.push({path:'/home'+url});

        //添加标签
        if(JSON.stringify(this.tags).indexOf(t)==-1){
          this.tags.push({
            main_title:t,
            type: '',
            id:id
          });
          this.isShowSlide();
        }
      },
      handleClose(tag){
        //删除标签的同时，点击并显示该标签前一个标签页
        if(this.tags.indexOf(tag)>0){
          document.querySelectorAll('.el-tag>span')[this.tags.indexOf(tag)-1].click();
        }else if(this.tags.indexOf(tag)==0&&this.tags.length>1){
          document.querySelectorAll('.el-tag>span')[this.tags.indexOf(tag)+1].click();
        }
        //删除标签页
        this.tags.splice(this.tags.indexOf(tag), 1);
        this.isShowSlide();

        //如果标签页全部删除，则显示第一个菜单作为标签页
        if(this.tags.length==0){
          document.querySelector('.el-menu-item').click();
        }
      },
      goThisPage(tag){
        //点击标签页，显示该标签页内容
        let _class='.menu'+tag.id;
        document.querySelector(_class).click();
      },
      setonlyClass(id){
        //动态设置类名，便于通过该类名找到某菜单，从而被点击
        return 'menu'+id;
      },
      isShowSlide(){
        //是否显示移动标签的 左右 箭头
        if(document.querySelectorAll('.el-tag').length*this.len+30>document.querySelector('.main-head-container').offsetWidth){
          this.islong=true;
          this.paddingleft="20px";
        }else{
          this.islong=false;
        }
      },
      goLeft(){
        //将标签左移
        this.goleft = (document.querySelector('.main-head').offsetLeft-30)+'px';
      },
      goRight(){
        //将标签右移
        this.goleft = (document.querySelector('.main-head').offsetLeft+30)+'px';
      }
    },
    mounted(){
      document.querySelector('.is-active').click(); //点击第一个菜单，应用标题

      setTimeout(function(){
        this.len=document.querySelector('.el-tag')&&document.querySelector('.el-tag').offsetWidth+2;
      }.bind(this), 50)
    },
    beforeMount(){

    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
    body > .el-container {
      margin-bottom: 40px;
    }
    .el-header, .el-footer {
      background-color: #B3C0D1;
      color: #333;
    }
    .el-header{
       color: #333;
       background: #fff;
       border-bottom: 1px solid #e0e4e9;
       box-shadow: 6px 1px 10px rgba(0,0,0,.17);
       height: 60px;
       line-height:60px;
       position: fixed;
       top: 0;
       z-index: 999;
       width:100%;
          h1 {
           font-size:18px;
           font-weight:bold;
          }
          .person-center {
             height: 30px;
             line-height: 30px;
             position: absolute;
             right:30px;
             top: 16px;
             font-size:14px;
          }
          .head-img>i {
             display:inline-block;
             width:30px;
             height: 30px;
             margin-bottom:-10px;
             margin-right:6px;
             background:url('../images/head.png') no-repeat center;
             background-size:contain;
          }
          .login-out {
            margin-left:30px;
            color:gray;
            outline: none;
          }
          .login-out:hover {
            border-bottom:solid 1px gray;
          }
     }

    .el-aside,.el-main {
      color: #333;
      position:absolute;
      top: 0;
      bottom:0;
      padding-top: 60px;
    }
    .el-aside {
      left:0;
      text-align:left;
      z-index:11;
      background-color: #545c64;
      .el-menu {
        border:none;
      }
    }
    .el-main {
      background-color: #fff;
      padding-left:240px;
      width:100%;
      text-align:left;
      z-index:10;
      .main-head-container {
        overflow: hidden;
        height:40px;
        line-height:40px;
        margin-top: 3px;
        border-bottom:solid 1px #ccc;
        padding-left:2px;
        position:relative;
        .el-icon-arrow-left,
        .el-icon-arrow-right {
          width:16px;
          height:30px;
          line-height:30px;
          font-weight:bold;
          cursor:pointer;
          margin-top:5px;
          background-color:#eee;
          cursor:pointer;
          text-align:center;
          position:relative;
          z-index:10;
        }
        .el-icon-arrow-left{
          float: left;
        }
        .el-icon-arrow-right{
          float:right;
        }
        .main-head {
          width:3000px;
          position:absolute;
          top: 0;
          left:0;
          z-index:9;
          -webkit-transition: all .5s ease;
             -moz-transition: all .5s ease;
              -ms-transition: all .5s ease;
               -o-transition: all .5s ease;
                  transition: all .5s ease;
          .el-tag {
            margin-right:2px;
            cursor: pointer;
            moz-user-select: -moz-none;
            -moz-user-select: none;
            -o-user-select:none;
            -khtml-user-select:none;
            -webkit-user-select:none;
            -ms-user-select:none;
            user-select:none;
          }
        }
      }
    }

    .main-page {
      padding:10px;
    }
</style>
