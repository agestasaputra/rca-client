<template>
    <div>
      <img class="img" :src="imageSrc" :alt="message.fileName" />
      <h4 class="img-title">{{ message.fileName }}</h4>
    </div>
</template>

<script>
import { reactive, toRefs, watch } from 'vue'
export default {
  name: 'Image',
  props: {
    message: {
      type: Object,
      default: () => ({})
    }
  },
  setup(props) {
    const state = reactive({
      blob: null,
      imageSrc: ''
    })

    watch(() => props.message, onMessageChanged, { immediate: true })
    watch(() => state.blob, onBlobChanged, { immediate: true })

    function onMessageChanged(currentValue) {
      if (currentValue) {
        state.blob = new Blob([currentValue.body], { type: currentValue.type })
      }
    }
    function onBlobChanged(currentValue) {
      if (currentValue) {
        const fr = new FileReader()
        fr.readAsDataURL(state.blob)
        fr.onloadend = () => {
          state.imageSrc = fr.result
        }
      }
    }

    return {
      ...toRefs(state),
      onMessageChanged,
      onBlobChanged
    }
  }
}
</script>

<style scoped>
  .img {
    width: 150px;
    height: auto;
  }
  .img-title {
    margin: 5px 0px;
  }
</style>