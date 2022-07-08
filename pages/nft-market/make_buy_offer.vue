<template lang="pug">
  .container
    .d-flex.align-items-center.mb-4
      img.pr-2(src='~/assets/icons/transfer_page/Vector_transfer.svg')
      p.font-page-title.p-7.m-0 Make Buy Offer
    .row
      .col-5
        .card-frame
          customSkeletonVue(v-if='loading', :width='220', :height='325')
          simpleCard(
            v-else,
            v-for='(item, index) in bulkTransfer',
            :key='index',
            :data='item',
            :addTrade="addTrade",
            :mintNum='item.template_mint',
            :collectionName='item.collection.collection_name',
            :immutableName='item.template.immutable_data.name'
          )
        el-input.bg-input-black.mt-4(placeholder="Amount of WAX" type="number" v-model='amount_wax')
        el-input.bg-input-black.my-4(placeholder="Memo" v-model='memo')
        h5.mb-2 Historical Price Data (Template based)
        h6.px-2 Lowest Market Listing:
          span.text-success 5400 WAX ($700.12)
        Chart(
          :charts='chartData',
          v-if='chartData.length',
          tab='Price',
          period='24H'
          minHeight='200'
          showPrice=false
        )
        CustomSkeletonVue(v-else, :width='400' :height='150')
        h5.mb-2 Fee Summary
        h6.px-2 Collection Fee: 0%
        h6.px-2 Alcor Fee: 1.00%
        h6.px-2 Tokenomics Fee: 0%
        .row
          .col-5
            el-button.w-100.mt-4(size="large" type="secondary") Buy Listing
          .col-7
            el-button.w-100.mt-4(size="large" type="success") Send Buy Offer
      .col-7
        el-input.bg-input-grey.mb-4(placeholder="Search NFTs" prefix-icon="el-icon-search" v-model="input2")
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
        .grid-container(v-if='loading')
          customSkeletonVue(
            v-for='item in 10',
            :key='item',
            :width='220',
            :height='325'
          )
        .grid-container(v-else)
          simpleCard(
            v-for='(item, index) in invData',
            :key='index',
            :data='item',
            :addTrade="addTrade",
            :cardState="(bulkTransfer.find(data => data.asset_id === item.asset_id) ? 'disable' : 'enable')",
            :mintNum='item.template_mint',
            :collectionName='item.collection.collection_name',
            :immutableName='item.template.immutable_data.name'
          )
</template>

<script>
import simpleCard from '~/components/wallet/cards/simpleCard.vue'
import Chart from '~/components/nft_markets/Chart'
import CustomSkeletonVue from '~/components/CustomSkeleton'
export default {
  components: {
    simpleCard,
    Chart,
    CustomSkeletonVue
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
      checkList: ['selected and disabled', 'Option A'],
      chartData: '',
      asset_id: 0,
      asset_ids: [],
      assetData: '',
      loading: true,
      invData: [],
      bulkTransfer: [],
      owner: '',
      memo: '',
      amount_wax: 0
    }
  },
  async created() {
    await this.getChartData(441285, "passes", true)
    this.asset_id = this.$route.params.id.split(',')[1]
    this.owner = this.$route.params.id.split(',')[0]
    await this.getAssetData()
    if (this.owner) {
      await this.getAssetsInventory(this.owner)
    }
  },
  computed: {
    // owner() {
    //   return this.assetData.owner
    // },
    template_mint() {
      if (this.assetData) {
        return this.assetData.template_mint
      } else return '0'
    },
    collectionName() {
      if (this.assetData) {
        return this.assetData.collection.name
      } else return ''
    },
    immutableName() {
      if (this.assetData) {
        return this.assetData.template.immutable_data.name
      } else return ''
    },
  },
  methods: {
    async getChartData(template_id, schema_name, burned) {
      this.chartData = await this.$store.dispatch('api/getChartData', {
        template_id,
        schema_name,
        burned
      })
    },
    debounceSearch(event) {
      clearTimeout(this.debounce)
      this.debounce = setTimeout(() => {
        this.recipientName = event.target.value
      }, 600)
    },
    deletebulkTransferItem(id) {
      this.bulkTransfer = this.bulkTransfer.filter((item) => item.asset_id !== id)
    },
    async handleSearch(key) {
      this.loading = true
      if (this.currentTab === 'your' && this.user.name) {
        this.assetsData = await this.$store.dispatch('api/getAssetsInventory', {
          owner: this.user.name,
          search: key,
        })
      } else if (this.currentTab === 'their' && this.recipientName) {
        this.recipientData = await this.$store.dispatch(
          'api/getAssetsInventory',
          {
            owner: this.recipientName,
            search: key,
          }
        )
      }
      this.loading = false
    },
    addTrade(item, cardState) {
      console.log(item, 'this is addtrade event')
      if (cardState != 'disable' && !this.bulkTransfer.find((data) => data.asset_id === item.asset_id)) {
        this.bulkTransfer.push(item)
        this.asset_ids.push(item.asset_id)
      } else {
        console.log(item, 'this is addtrade event')
        this.bulkTransfer = this.bulkTransfer.filter((data) => data.asset_id !== item.asset_id)
      }
    },
    async getAssetData() {
      this.loading = true
      await this.getSpecificAsset(this.asset_id)
      // this.getAssetsTransfer(this.asset_id)
      // this.getAssetsSales(this.asset_id)
      // this.getAssetsLog(this.asset_id)
      this.loading = false
    },
    async getAssetsInventory(owner) {
      this.loading = true
      const data = await this.$store.dispatch('api/getAssetsInventory', {
        owner,
      })
      this.invData = data
      this.loading = false
    },
    async getSpecificAsset(asset_id) {
      this.loading = true
      const data = await this.$store.dispatch('api/getSpecificAsset', {
        asset_id,
      })
      this.assetData = data
      this.bulkTransfer.push(data)
      this.attributeKeys = Object.keys(data.data)
      this.loading = false
    },
    async transferAsset() {
      if (this.to != '' && this.memo != '') {
        const actions = [
          {
            account: 'atomicassets',
            name: 'transfer',
            authorization: [
              {
                actor: this.assetData.owner,
                permission: 'active',
              },
            ],
            data: {
              from: this.assetData.owner,
              to: this.to,
              asset_ids: this.asset_ids,
              memo: this.memo,
            },
          },
        ]
        try {
          await this.$store.dispatch('chain/sendTransaction', actions)
          this.$notify({
            title: 'Asset Transfer',
            message: 'Asset Transferred!',
            type: 'success',
          })
        } catch (e) {
          this.$notify({
            title: 'Asset Transfer',
            message: e,
            type: 'error',
          })
        } finally {
          this.to = ''
          this.memo = ''
        }
      }
    },
  },
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
    grid-template-columns: 33.3% 33.3% 33.3%;
    gap: 0px;
    max-height: 700px;
    overflow: scroll;
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
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
    border: 1px solid #67c23a;
    border-radius: 4px;
    padding: 20px;
    max-height: 700px;
    overflow: scroll;
  }
}
</style>
