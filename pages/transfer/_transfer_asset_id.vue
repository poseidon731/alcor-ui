<template lang="pug">
  .container
    .d-flex.align-items-center.mb-4
      img.pr-2(src='~/assets/icons/transfer_page/Vector_transfer.svg')
      p.font-page-title.p-7.m-0 Transfer
    p.font-label.font-input Transfer to
      el-input.bg-input-black(placeholder="WAX Address/Account Name" v-model="to")
    p.font-label.font-input Memo
      el-input.bg-input-black(placeholder="Transfer Memo" v-model="memo")
    .card-frame
      Transfer_card(:img='imageBackground' imgWidth='300px' :mintNum='template_mint' :collectionName='collectionName' :immutableName='immutableName')
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
      Transfer_card.p-2(v-for="item in invData" :key='item' :img="'https://ipfs.atomichub.io/ipfs/'+item.data.img" :mintNum='item.template_mint' :collectionName='item.collection.collection_name' :immutableName='item.template.immutable_data.name')
    el-button.w-100.mt-4(size="large" type="success" @click="transferAsset") Send Transfer
</template>

<script>
import Transfer_card from '~/components/wallet/cards/transfer_card.vue'
export default {
  components: {
    Transfer_card,
  },
  data() {
    return {
      to: '',
      memo: '',
      asset_id: 0,
      assetData: '',
      invData: [],
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
  },
  computed: {
    owner() {
      return this.assetData.owner
    },
    videoBackground() {
      if (this.mode === 'market-sales' || this.mode === 'market-auctions') {
        if (this.assetData.data.video) {
          return this.assetData.data
        } else return false
      } else if (this.mode === 'templates' || this.mode === 'setsList') {
        if (this.assetData.immutable_data.video) {
          return this.assetData.immutable_data
        } else return false
      } else if (
        this.mode === 'inventory' ||
        this.mode === 'sets' ||
        this.mode === 'assetsInventory'
      ) {
        if (this.assetData.data.video) {
          return this.assetData.data
        } else return false
      } else if (
        this.mode === 'listings' ||
        this.mode === 'auctions' ||
        this.mode === 'sold' ||
        this.mode === 'bought'
      ) {
        if (this.assetData && this.assetData.data.video) {
          return this.assetData.data.video
        } else return false
      } else return false
    },
    imageBackground() {
      if (this.assetData && this.assetData.data.img) {
        return this.assetData.data.img.includes('https://') ? this.assetData.data.img : 'https://ipfs.atomichub.io/ipfs/' +
            this.assetData.data.img
      } else return false
    },
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
    }
  },
  mounted() {
    this.asset_id = this.$route.params.transfer_asset_id
    this.getAssetData()
    if (this.assetData.owner) {
      this.getAssetsInventory(this.assetData.owner)
    }
  },
  watch: {
    owner(new_data, old_data) {
      if (!old_data && new_data) {
        this.getAssetsInventory(new_data)
      }
    }
  },
  methods: {
    async getAssetData() {
      await this.getSpecificAsset(this.asset_id)
      // this.getAssetsTransfer(this.asset_id)
      // this.getAssetsSales(this.asset_id)
      // this.getAssetsLog(this.asset_id)
    },
    async getAssetsInventory(owner) {
      console.log(owner)
      const data = await this.$store.dispatch('api/getAssetsInventory', {
        owner
      })
      this.invData = data
      console.log(this.invData, "{{{{{{{{{{}}}}}}}}}}")
    },
    async getSpecificAsset(asset_id) {
      this.loading = true
      const data = await this.$store.dispatch('api/getSpecificAsset', {
        asset_id
      })
      console.log(data, "============asset data")
      this.assetData = data
      await this.getTemplatePrice(data.template.template_id)
      await this.getChartData(
        data.template.template_id,
        data.schema.schema_name,
        data.is_burnable
      )
      this.attributeKeys = Object.keys(data.data)
      this.loading = false
    },
    async transferAsset() {
      if (this.to != '' && this.memo != '') {
        const actions = [{
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
            asset_ids: [this.asset_id],
            memo: this.memo,
          },
        }]
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
    grid-template-columns: 25% 25% 25% 25%;
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
