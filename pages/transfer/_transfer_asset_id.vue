<template lang="pug">
  .container
    .d-flex.align-items-center.mb-4
      img.pr-2(src='~/assets/icons/transfer_page/Vector_transfer.svg')
      p.font-page-title.p-7.m-0 Transfer
    p.font-label.font-input Transfer to
      el-input.bg-input-black(placeholder="WAX Address/Account Name")
    p.font-label.font-input Memo
      el-input.bg-input-black(placeholder="Transfer Memo")
    .card-frame
      Transfer_card
    el-input.bg-input-grey.my-4(placeholder="Search NFTs" prefix-icon="el-icon-search" v-model="input2")
    el-select.bg-input-grey.mb-4.w-100(v-model="value" large placeholder="Choose Collection")
      el-option(v-for="item in options" :key="item.value" :label="item.label" :value="item.value")
    el-row.mb-4
      el-col.pr-1(:span="12")
        el-input.bg-input-black(placeholder="Min Mint")
      el-col.pl-1(:span="12")
        el-input.bg-input-black(placeholder="Max Mint")
    el-checkbox-group.mb-4(v-model="checkList")
      el-checkbox(size="medium" label="Only Duplicates")
      el-checkbox(label="Only backed NFTs")
      el-checkbox(label="Only whitelisted NFTs")
    .grid-container
      Transfer_card.p-2(v-for="item in 15" :key='item')
    el-button.w-100.mt-4(size="large" type="success") Send Transfer
</template>

<script>
import Transfer_card from '~/components/wallet/cards/transfer_card.vue'
export default {
  components: {
    Transfer_card,
  },
  data() {
    return {
      options: [
        {
          value: 'Option1',
          label: 'Option1',
        },
        {
          value: 'Option2',
          label: 'Option2',
        },
        {
          value: 'Option3',
          label: 'Option3',
        },
        {
          value: 'Option4',
          label: 'Option4',
        },
        {
          value: 'Option5',
          label: 'Option5',
        },
      ],
      value: '',
      checkList: ['selected and disabled', 'Option A']
    }
  }
}
</script>

<style lang="scss">
.bg-input-black .el-input__inner {
  background-color: black;
}
.bg-input-grey .el-input__inner {
  background-color: #333333;
}
</style>
<style lang="scss" scoped>
.grid-container {
    display: grid;
    grid-template-columns: 20% 20% 20% 20% 20%;
    gap: 0px;
  }
.container {
  .font-page-title {
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 700;
    font-size: 18px;
    line-height: 21px;
    color: #ffffff;
  }
  .font-label {
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 400;
    font-size: 18px;
    line-height: 21px;
  }
  .card-frame {
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid #67c23a;
    border-radius: 4px;
    padding: 20px;
  }
}
</style>
