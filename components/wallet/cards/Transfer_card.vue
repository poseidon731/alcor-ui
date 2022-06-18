<template lang="pug">
el-row
  el-col
    el-card(:body-style='{ padding: "0px" }')
      img(src='~/assets/images/nft.svg')
      .card-title
        span
        .bottom-clearfix
        p CandyLand
        el-button.button(type='text')
          p Operating
</template>
<script>
export default {
  data() {
    return {
      currentDate: new Date(),
    }
  },
}
</script>
<style lang="scss" scoped>
.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 100%;
  display: block;
}
</style>
