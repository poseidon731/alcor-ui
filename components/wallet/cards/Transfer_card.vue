<template lang="pug">
el-row
  el-col
    el-card(:body-style='{ padding: "0px" }')
      div.d-flex.p-1.justify-content-end
        div.float-right.p-1.bg-black.text-success #11
      img.w-100(src='~/assets/images/nft.svg')
      .d-flex.justify-content-between.align-items-center.p-3
        div CandyLand
        .d-flex.align-items-center
          div.small.pr-1.text-secondary Alcorex
          img(src='~/assets/icons/check-circle.svg')
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
