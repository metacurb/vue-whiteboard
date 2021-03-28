<template>
  <vue-whiteboard
    class="whiteboard"
    :color="color"
    :line-styles="{
      'mix-blend-mode': 'multiply',
    }"
    ref="whiteboard"
    size="10px"
  />
  <div class="box yellow" @click="color = '#ffc93b'"></div>
  <div class="box red" @click="color = '#ff555f'"></div>
  <div class="box black" @click="color = '#212121'"></div>
  <div class="box blue" @click="color = '#3494ff'"></div>
  <div @click="$refs.whiteboard.clear">clear</div>
  <div @click="$refs.whiteboard.undo">undo</div>
  <div @click="$refs.whiteboard.redo">redo</div>
  <div @click="save">saveSvg</div>
  <img :src="image" width="500" height="500" />
</template>

<script>
import VueWhiteboard from './components/vue-whiteboard'

export default {
  name: 'App',
  components: {
    VueWhiteboard,
  },
  data: () => ({
    color: '#333333',
    image: null,
  }),
  methods: {
    async save() {
      const res = await this.$refs.whiteboard.save()
      this.image = res
    },
  },
}
</script>

<style lang="scss">
html,
body,
#app {
  margin: 0;
  height: 100%;
}

.whiteboard {
  margin: 20px;
  width: 500px;
  height: 500px;
  border: 1px solid black;
}

.box {
  width: 15px;
  height: 15px;
}

.black {
  background: #212121;
}

.yellow {
  background: #ffc93b;
}

.blue {
  background: #3494ff;
}

.red {
  background: #ff555f;
}
</style>
