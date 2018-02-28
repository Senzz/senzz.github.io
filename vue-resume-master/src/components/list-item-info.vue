<template>
  <ListItem class="list-item-info">
    <EditImage :src="image" class="img"></EditImage>
    <span :contenteditable="contenteditable">
      <a v-if='isLink' :href="'https://' + title" target="_blank">{{title}}</a>
      <a v-else>{{ title }}</a>
    </span>
  </ListItem>
</template>
<script>
  import ListItem from '@/components/list-item'
  import EditImage from '@/components/edit-image'
  export default {
    name: 'ListItemInfo',
    components: {
      ListItem,
      EditImage
    },
    props: {
      isLink: {
        type: Boolean,
        default: false
      },
      image: {
        type: String,
        default: require('@/assets/social-github.png')
      },
      title: {
        type: String,
        default: 'github.com/luosijie'
      }
    }
  }
</script>
<style lang="less">
  .list-item-info{
    display: flex;
    align-items: center;
    border-bottom: 1px solid #e9e8e8;
    height: 80px;
    .img{
      margin: 0 20px 0 50px;
    }
    span{
      font-size: 20px;
      color: #555;
    }
  }
  a{
    text-decoration: none;
  }
</style>
