<template>
  <ListItem class="list-item-experience">
    <div class="heading">
      <span class="company" :contenteditable="contenteditable">{{ company }}</span>
      <span class="job" :contenteditable="contenteditable">
        <a v-if='isLink' :href="href" target="_blank">{{job}}</a>
        <a v-else>{{ job }}</a>
      </span>
      <span class="date" :contenteditable="contenteditable">{{ date }}</span>
    </div>
    <div v-html="content" :contenteditable="contenteditable">
      
    </div>
  </ListItem>
</template>
<script>
  import ListItem from '@/components/list-item'
  export default {
    name: 'ListItemExperience',
    components: {
      ListItem
    },
    props: {
      company: {
        type: String,
      },
      job: {
        type: String
      },
      isLink: {
        type: Boolean,
        default: false
      },
      href: {
        type: String
      },
      date: {
        type: String,
        default: '2011.9 - 2015.6'
      },
      content: {
        type: String
      }
    }
  }
</script>
<style lang="less">
  .list-item-experience{
    .heading{
      font-size: 18px;
      font-weight: bold;
      display: flex;
      justify-content: space-between;
      margin-bottom: 12px;
      .company{
        width: 40%
      }
      .job{
        width: 35%;
      }
      .date{
        width: 25%;
        text-align: right;
      }
    }
  }
</style>
