<template lang="pug">
.row.sales-row.m-2
  .col-2.d-flex.p-2.align-items-center
    a(href="#") Sales
  .col-6.d-flex.p-2.align-items-center.row
    .col-5.d-flex.align-items-center
      img.mr-1.coin-icon(src='~/assets/icons/wax.png')
      a(href="#") {{data.seller}}
    .col-2.text-center
     img.mx-2(src='~/assets/images/double_arrow.svg')
    .col-5.d-flex.align-items-center
      img.mr-1.coin-icon(src='~/assets/icons/wax.png')
      a(href="#") {{data.buyer}}
  .d-flex.flex-fill.p-2.align-items-center
    div.tab-content-date {{ +data.block_time | moment('MM/DD/YYYY HH:mm a') }}
</template>
<script>
export default {
  props: ['data'],
  name: 'TransferRow',
  data() {
    return {
    }
  },
}
</script>
<style lang="scss">
.sales-row {
  background: #161617;
  height: 45px;
  .el-tab-pane{
    overflow-Y: scroll;
    overflow-X: hidden;
    height:200px;
  }
  .transfer-col {
    display: flex;
  }
  a {
    color: #67C23A !important;
    font-size: 14px !important;
  }
  .coin-icon {
    width: 24px;
    height: 24px;
  }
}

</style>
