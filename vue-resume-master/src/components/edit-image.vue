<template>
  <div class="edit-image" :style="{ width: width, height: height}">
    <img :src="imgSrc" alt="image" :class="{ circle: isCircle }">
    <input :style="{ display: contenteditable ? 'block' : 'none' }" :disabled="!contenteditable" type="file" accept="image/gif,image/jpeg,image/jpg,image/png" @change="changeImage">
  </div>
</template>
<script>
  export default {
    name: 'EditImage',
    props: {
      width: {
        type: String,
      },
      height: {
        type: String,
      },
      src: {
        type: String,
        default: require('@/assets/social-github.png')
      },
      isCircle: {
        type: Boolean,
        default: false
      }
    },
    data: function () {
      return {
        imgSrc: this.src
      }
    },
    methods: {
      changeImage: function (evt) {
        if (this.contenteditable) {
          return
        }
        let _this = this
        let reader = new FileReader()
        let file = evt.target.files[0]
        reader.readAsDataURL(file)
        reader.onload = function (evt) {
          _this.imgSrc = evt.target.result
        }
      }
    }
  }
</script>
<style lang="less">
  @media screen and (max-width: 500px) {
    div.edit-image{
      width: 40px;
      height: 40px;
    }
  }
  .edit-image{
    position: relative;
    display: inline-block;
    z-index: 999;
    width: 60px;
    height: 60px;
    img{
      width: 100%;
      height: 100%;
    }
    input{
      display: inline-block;
      position: absolute;
      cursor: pointer;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      opacity: 0;
    }
  }
  .circle{
    border-radius: 106px;
  }
</style>
