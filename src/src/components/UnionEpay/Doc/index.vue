<template>
  <div>
    <svg-icon icon-class="question" @click="goto"/>
  </div>
</template>

<script>
export default {
  name: 'UnionEpayDoc',
  data() {
    return {
      url: 'https://unionepay.com/'
    }
  },
  methods: {
    goto() {
      window.open(this.url)
    }
  }
}
</script>
