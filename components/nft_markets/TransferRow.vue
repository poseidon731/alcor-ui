<template lang="pug">
.row.transfer-row.m-2.border-radius5
  .col-2.d-flex.p-2.align-items-center
    a(href='#') Transfer
  .col-6.d-flex.p-2.align-items-center.row
    .col-5.d-flex.align-items-center
      img.mr-1.coin-icon(src='~/assets/icons/wax.png')
      a(href='#') {{ data.sender_name }}
    .col-2.text-center
      img.mx-2(src='~/assets/images/double_arrow.svg')
    .col-5.d-flex.align-items-center
      img.mr-1.coin-icon(src='~/assets/icons/wax.png')
      a(href='#') {{ data.recipient_name }}
  .d-flex.flex-fill.p-2.align-items-center
    .tab-content-date {{ +data.created_at_time | moment("MM/DD/YYYY HH:mm a") }}
</template>
<script>
export default {
  props: ['data'],
  name: 'TransferRow',
  data() {
    return {}
  },
}
</script>
<style lang="scss">
.transfer-row {
  background: #161617;
  height: 45px;
  .el-tab-pane {
    overflow-y: scroll;
    overflow-x: hidden;
    height: 200px;
  }
  .transfer-col {
    display: flex;
  }
  a {
    color: #67c23a !important;
    font-size: 14px !important;
  }
  .coin-icon {
    width: 24px;
    height: 24px;
  }
}
</style>
