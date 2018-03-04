<template>
  <div id="app">
    <div class="resume">
      <div class="left">
        <Brief></Brief>
        <ContextList title="CONTACT" :icon="require('@/assets/title-contact.png')">
          <ListItemInfo title="188-250-75868" 
                        :image="require('@/assets/contact-phone.png')"
                        slot="listItem">
          </ListItemInfo>
          <ListItemInfo title="825618507@qq.com" 
                        :image="require('@/assets/contact-mail.png')"
                        slot="listItem">
          </ListItemInfo>
          <!-- <ListItemInfo title="lsj***inhao" 
                        :image="require('@/assets/social-wechat.png')"
                        slot="listItem">           
          </ListItemInfo> -->
          <ListItemInfo title="825618507" 
                        :image="require('@/assets/social-qq.png')"
                        slot="listItem">           
          </ListItemInfo>
        </ContextList>
        <ContextList title="SOCIAL" :icon="require('@/assets/social.png')">
          <ListItemInfo title="github.com/Senzz" 
                        :isLink="true"
                        :image="require('@/assets/social-github.png')"
                        slot="listItem">      
          </ListItemInfo>
          <ListItemInfo title="senzz.github.io"
                        :isLink="true" 
                        :image="require('@/assets/blog.png')"
                        slot="listItem">               
          </ListItemInfo>
        </ContextList>
        <ContextList title="About me" :icon="require('@/assets/about_us.png')">
          <ListItemAbout slot="listItem"></ListItemAbout>
        </ContextList>
      </div>
      <div class="right">
        <ContextList title="Skill">
          <ListItemSkill slot="listItem" title="JavaScript" :percent="80"></ListItemSkill>
          <ListItemSkill slot="listItem" title="Html5 + css3" :percent="60"></ListItemSkill>
          <ListItemSkill slot="listItem" title="Vue" :percent="85"></ListItemSkill>
          <ListItemSkill slot="listItem" title="Nodejs" :percent="40"></ListItemSkill>          
          <ListItemSkill slot="listItem" title="express" :percent="80"></ListItemSkill>
        </ContextList>
        <ContextList title="Education">
          <ListItemEducation slot="listItem" school="广东药科大学" major="经济学" date="2013.09 — 2017.06"></ListItemEducation>
        </ContextList>
        <ContextList title="Working Experience">
          <ListItemExperience 
            v-for='item of workExperience'
            :key="item.Airbnb"
            slot="listItem" 
            :company="item.company" 
            :job="item.job" 
            :date="item.date"
            :content="item.content"></ListItemExperience>
        </ContextList>
        <ContextList title="Project Experience">
          <ListItemExperience 
            v-for='item of jobExperience'
            :key="item.Airbnb"
            slot="listItem"
            :isLink='item.isLink'
            :href='item.href'
            :company="item.company" 
            :job="item.job" 
            :date="item.date"
            :content="item.content"></ListItemExperience>
        </ContextList>
      </div>
    </div>
    <!-- <button class="sap" @click="saveAsImage">Save as Image</button> -->
  </div>
</template>

<script>
import ContextList from '@/components/context-list'
import ListItemAbout from '@/components/list-item-about'
import ListItemSkill from '@/components/list-item-skill'
import ListItemEducation from '@/components/list-item-education'
import ListItemExperience from '@/components/list-item-experience'
import ListItemInfo from '@/components/list-item-info'
import html2canvas from '@/assets/js/html2canvas.js'
import FileSaver from 'file-saver'
import Brief from '@/components/brief'

export default {
  name: 'app',
  components: {
    ContextList,
    ListItemAbout,
    ListItemSkill,
    ListItemEducation,
    ListItemExperience,
    ListItemInfo,
    Brief
  },
  data () {
    return {
      workExperience: [
        { 
          company: '广州网梭信息技术有限公司', job: '前端开发', date: '2016.09 — 2017.12',
          content: `<ul>
                    <li>1. 与设计师和后台程序配合，高效率高质量地完成页面的实现工作</li>
                    <li>2. 对网站的用户体验以及访问速度进行优化</li>
                    <li>3. 负责与后台程序开发人员配合，为产品提供最优化的前端技术解决方案</li>
                    <li>4. 跟踪前端技术的发展，掌握最新动态和主流技术，优化公司产品体验</li>
                    <ul>`
        }
      ],
      jobExperience: [
        { 
          company: '汽车救援网站', job: 'https://kk519.cn', href: 'https://kk519.cn', isLink: true,  date: '2017.01—2017.03',
          content: `<p>这个是一个SPA，开发人数共2人，技术栈有: express、Vue、Vuex、Vue-router、moment.js等。</br>
我与项目组长分工合作，以模块分割任务，我主要负责的有财务管理、公司管理、系统设置等。</br>
工作内容主要是与产品和设计那边沟通好页面设计，高效高质量的页面重构，以及一些动态效果；</br>
与后台沟通接口问题，使用ajax调用后台接口，把数据渲染到页面；</br>
编写通用的组件，让它具有一定的易使用性、易扩展性、易维护性；</br>
使用nodejs来连接后台的redis，来做消息订阅，通知；</br>
让用户可以选中多个订单来把多个订单的图片一起打包下载，主要使用了nodejs的async和request模块。</br>
</p>`
        },
        { 
          company: '公司内部使用的Bug搜索网站', job: 'https://log.wshuttle.com', href: 'https://log.wshuttle.com', isLink: true,  date: '2017.04—2017.05',
          content: `<p>该项目是我个人单独开发的。用时2周。ElasticSearch作为一个搜索服务器。主要它能提供一个分布式多用户能力的全文搜索引擎。</br>
技术栈主要使用了nodejs的ElasticSearch、express模块，页面渲染使用ejs模板引擎，页面交互使用了jquery。因为之前没接触过这个，所以要看官方文档学习以及了解nodejs的ElasticSearch模块。
然后建立索引，操作增删改查等api来实现。主要需要提供一个添加的接口以供公司内部收集bug使用，然后书写查询接口，多种标签下拉查询，数量查询等，和展示接口以供页面ajax来调用。 统计页面也使用了highcharts来展示简单的统计图表。
</p>`
        },
        { 
          company: '微信公众号',   date: '2017.08—2017.09',
          content: `<p>开发人数共2人，用时2周。技术栈有: express、jquery、moment.js等。</br>
我主要完成页面的任务，另外一个同事完成server端的部署（使用nodejs作为服务端）。</br>
页面使用了ejs模板引擎。根据UI和产品文档完成页面设计，通过ajax完成接口数据展示。</br>
使用了JS—SDK来完成录音、上传录音、预览照片、上传照片、获取地理位置等功能。</br>
主要遇到的难点在：由于微信公众号的录音文件格式为amr格式，需要在其他平台听的话需要转为MP3格式。</br>
后面了解到了fluent-ffmpeg能转变到mp3格式。最后通过fluent-ffmpeg来转为成MP3，转化过程中把chunk拼接成完整的buffer，转为binary格式，通过oss上传到阿里云服务器。
</p>`
        },
        { 
          company: '微信小程序',   date: '2017.12—2017.12',
          content: `<p>开发人数共3人，时间为一周。因为微信小程序刚出就关注，所以被选作为负责人。也是nodejs作为服务器。技术栈有: express、ws等。</br>
由于另外2个同事要花时间阅读小程序文档，所以我分配了自己大部分的开发任务，以及帮助他们解决一些遇到的问题。</br>
我主要完成了：根据UI设计，把相识可服用的成组件；完成了大部分的页面和功能（聊天，下单接单，个人信息注册，列表和详情，收藏）；封装ws模块的监听、触发、发送等事件，主要用于实现简单的聊天功能；</br>
最后因为实际业务与之前的后台业务逻辑有冲突，离职前还未上线。
</p>`
        }
      ]
    }
  },
  methods: {
    saveAsImage: function () {
      let resume = document.querySelector('.resume')
      html2canvas(resume).then(function (canvas) {
        canvas.toBlob(function (blob) {
          FileSaver.saveAs(blob, 'Resume.png')
        })
      })
    }
  }
}
</script>

<style lang="less">
  * {
    box-sizing: border-box;
  }
  body{
    padding: 0;
    margin: 0;
    
    width: 100%;
  }
  ul{
    padding: 0;
    margin: 0;
    li{
      list-style: none;
    }
  }
  p{
    margin: 0 0 12px 0;
  }
  @media screen and (max-width: 1300px) {
    body{
      min-width: 700px;
    }
  }
  @media screen and (min-width: 1300px) {
    #app {
      width: 1800px;
      padding: 40px 40px;
    }
    #app .resume{
      border: 1px solid #dad8d7;
    }
  }
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: left;
    color: #2c3e50;
    background-color: white;
    // display: flex;
    // flex-direction:column;
    // align-items: center;
    // justify-content: center;
    
    margin: auto;
    
    
    .resume{
      // width: 1247px;
      // height: 1754px;
      
      background-color: white;
      // overflow: hidden;
      display: flex;
      .left{
        // float: left;
        width: 380px;
        // height: 100%;
        background-color: #fdfefe;
        .brief{
          // height: 630px;
          background-color: #f6f7f7;
          .avatar{
            height: 500px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            &>.img{
              margin-bottom: 45px;
            }
            .name{
              font-size: 32px;
              margin-bottom: 14px;
            }
            .job{
              font-size: 20px;
              color: #555;
              margin-bottom: 16px;
            }
            .location{
              display: flex;
              align-items: center;
              .location-name{
                font-size: 20px;
                font-weight: bold;
                margin-left: 10px;
              }
            }
          }
          .info{
            height: 128px;
            ul{
              height: 100%;
              display: flex;
              border-top: 1px solid #dad8d7;
              border-bottom: 1px solid #dad8d7;
              li:not(:last-child){
                border-right: 1px solid #dad8d7;
              }
              li{
                width: 33.3333333%;
                display: flex;
                flex-direction: column;
                justify-content: center;
                text-align: center;
                .value{
                  margin-bottom: 10px;
                  font-size: 24px;
                  font-weight: bold;
                }
                .key{
                  font-size: 16px;
                  font-weight: bold;
                  color: #555;
                }
              }
            }
          }
        }
      }
      .right{
        // float: left;
        // box-sizing: border-box;
        // width: 867px;
        // flex: 1;
        padding: 50px 40px;
        p{
          font-size: 20px;
          line-height: 32px;
          color: #555;
          text-align:justify;
        }
      }
      
    }
    button.sap{
      position: absolute;
      top: 65px;
      left: 50%;
      margin-left: 478px;
      background-color: white;
      border: 1px solid #dad8d7;
      cursor: pointer;
      padding: 5px 8px;
      border-radius: 4px;
      &:hover{
        background-color: #f8f8f8;
      }
    }
    .guide{
      border: 1px solid #dad8d7;
      width: 1247px;
      margin-top: 15px;
      height: 120px;
      ul{
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: space-around;
        align-items: center;
        li{
          display: flex;
          align-items: center;
          img{
            margin: 0 10px;
          }
          h3{
            margin: 0 0 6px 0;
          }
          p{
            margin: 0;
            font-size: 16px;
          }
        }
      }
    }
    .footer{
      line-height: 50px;
      a{
        text-decoration: none;
        color: #4aa74f;
        &:first-child{
          display: block;
          text-align: center;
          line-height: 0;
          margin-top: 15px;
        }
      }
    }
  }
  input:disabled{
    background-color: inherit;
  }
</style>
