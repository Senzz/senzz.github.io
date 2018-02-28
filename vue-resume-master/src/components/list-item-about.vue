<template>
  <ListItem>
    <pre :contenteditable="contenteditable">
      为人做事认真、细致、踏实;
      求知欲强，能承受一定的压力;
      喜欢迎接新挑战，有较强的责任心和荣誉感;
      具有团队合作精神，具有较强的独立工作能力;
    </pre>
  </ListItem>
</template>
<script>
  import ListItem from '@/components/list-item'
  export default {
    name: 'ListItemAbout',
    components: {
      ListItem
    }
  }
</script>

<style scoped>
pre{
  margin-left: 20px;
  line-height: 28px;
}
</style>

