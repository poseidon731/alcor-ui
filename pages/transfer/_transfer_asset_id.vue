<template lang="pug">
.container
  nuxt-link(:to='"/wallet-inventory/nfts"', :exact='true')
    a#return-btn Return
  .d-flex.align-items-center.mb-4
    img.pr-2(src='~/assets/icons/transfer_page/Vector_transfer.svg')
    p.font-page-title.p-7.m-0 Transfer
  p.font-label.font-input Transfer to
    el-input.bg-input-black(
      placeholder='WAX Address/Account Name',
      v-model='to'
    )
  p.font-label.font-input Memo
    el-input.bg-input-black(placeholder='Transfer Memo', v-model='memo')
  .InventorySelection
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
  .d-flex
    el-input.bg-input-grey.my-4.mr-4(
      type='text',
      :value='searchValue',
      @input='debounceSearch',
      @focus='focusInput',
      @blur='blurInput',
      placeholder='Search NFTs',
      prefix-icon='el-icon-search'
      clearable

    )
    el-dropdown.filter-input-group.border-bottom--gray.d-flex.flex-column.justify-content-center(
          trigger='click'
        )
          .el-dropdown-link.d-flex.align-items-center.justify-content-between
            img.me-1(src='~/assets/images/filter.svg', alt='')
            p.m-0 Collections ({{invData.length}})
            i.el-icon-arrow-down.el-icon--right
          el-dropdown-menu.collection-dropdown(slot='dropdown')
            button.btn.btn-collection.w-100.mb-1.d-flex.align-items-center(
              @click='() => handleCollection("")'
            )
              img(src='~/assets/images/default.png')
              p.ml-1.flex-fill.text-left.collection-name All
            button.btn.btn-collection.w-100.mb-1.d-flex.align-items-center(
              v-for='(item, index) in invData',
              :key='index',
              @click='() => handleCollection(item.collection.collection_name)'
            )
              img(v-if='item.collection.img && item.collection.img.includes("https://")', :src='item.collection.img')
              img(v-else-if='item.collection.img', :src='"https://ipfs.io/ipfs/" + item.collection.img')
              img(v-else, src='~/assets/images/default.png')
              p.ml-1.flex-fill.text-left.collection-name {{ item.collection.name }}
  //- el-select.bg-input-grey.mb-4.w-100(
  //-   v-model='value',
  //-   large,
  //-   placeholder='Choose Collection',
  //-   @change="updateDropDwon"
  //- )
  //-   el-option(
  //-     v-for='item in invData',
  //-     :key='item.asset_id',
  //-     :label='item.name',
  //-     :value='item.name'
  //-   )
  el-row.mb-4
    el-col.pr-1(:span='12')
      el-input.bg-input-black(type="number" v-model="minMint" placeholder='Min Mint' @input='minMintSearch')
    el-col.pl-1(:span='12')
      el-input.bg-input-black(type="number" v-model="maxMint" placeholder='Max Mint' @input='maxMintSearch' )
  el-checkbox-group.mb-4(v-model='checkList')
    el-checkbox(size='medium', label='Only Duplicates')
    el-checkbox(label='Only backed NFTs')
    el-checkbox(label='Only whitelisted NFTs')
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
  el-button.w-100.mt-4(size='large', type='success', @click='transferAsset') Send Transfer
</template>

<script>
import simpleCard from '~/components/wallet/cards/simpleCard.vue'
import customSkeletonVue from '~/components/CustomSkeleton'

export default {
  components: {
    simpleCard,
    customSkeletonVue,
  },
  data() {
    return {
      loading: true,
      to: '',
      memo: '',
      asset_id: 0,
      asset_ids: [],
      assetData: '',
      bulkTransfer: [],
      invData: [],
      options: [],
      value: '',
      checkList: ['selected and disabled', 'Option A'],
      searchNFTs: '',
      searchValue: '',
      currentCollectionName: '',
      minMint: '',
      maxMint: ''
    }
  },
  computed: {
    owner() {
      return this.assetData.owner
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
    },
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
    },
  },
  methods: {
    debounceSearch(event) {
      console.log(event, "111111111")
      this.handleSearch(event)
      // clearTimeout(this.debounce)
      // this.debounce = setTimeout(() => {
      //   this.recipientName = event.target.value
      // }, 600)
    },
    handleCollection(event) {
      this.currentCollectionName = event
      this.searchCollectionName()
    },
    async searchCollectionName() {
      this.invData = await this.$store.dispatch('api/getAssetsInventory', {
        owner: this.assetData.owner,
        collectionName: this.currentCollectionName
      })
    },
    deletebulkTransferItem(id) {
      this.bulkTransfer = this.bulkTransfer.filter((item) => item.asset_id !== id)
    },
    focusInput(event) {
      event.target.parentElement.classList.add('border-bottom--cancel')
    },
    blurInput(event) {
      event.target.parentElement.classList.remove('border-bottom--cancel')
    },
    handleSearchValue(value) {
      this.searchValue = value
    },
    minMintSearch(event) {
      this.minMint = event
      this.mintSearch()
    },
    maxMintSearch(event) {
      this.maxMint = event
      this.mintSearch()
    },
    async mintSearch() {
      this.invData = this.invData.filter((data, id) => {
        data.template_mint > this.minMint
      })
      console.log(this.invData, "1111111")
      if (this.invData.length == 0)
        this.getAssetsInventory(this.assetData.owner)
      // if (this.minMint != '')
      //   this.invData = this.invData.filter((data, id) => {
      //     data.template_mint < this.minMint
      //   })
      // else
      //   this.getAssetsInventory(this.assetData.owner)
    },
    async handleSearch(key) {
      this.loading = true
      this.invData = await this.$store.dispatch('api/getAssetsInventory', {
        owner: this.assetData.owner,
        search: key,
        collectionName: this.currentCollectionName
      })
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
      console.log(data, "iiiiiiiiiiiiii")
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
      let transfer_assets_id = []
      this.bulkTransfer.map((data, id) => {
        transfer_assets_id.push(data.asset_id)
      })
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
              asset_ids: transfer_assets_id,
              memo: this.memo,
            },
          },
        ]
        try {
          const tx = await this.$store.dispatch('chain/sendTransaction', actions)
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
          this.bulkTransfer = []
          transfer_assets_id.map((asset_id, id) => this.invData = this.invData.filter((data, id) => data.asset_id != asset_id))
        }
      } else {
        this.$notify({
          title: 'Transfer Failed',
          message: 'Please Input Correct Addres and Memo',
          type: 'error',
        })
      }
    },
  },
}
</script>

<style lang="scss">
  .filter-input-group {
    width: 250px;
  }

  .filter-input-group .search-input {
    width: 80px !important;
  }

  .filter-input,
  .search-input {
    color: var(--cancel);
  }
.el-dropdown-menu.collection-dropdown {
  background: #333;
  border: 1px dashed var(--main-green) !important;
  max-height: 400px;
  width: 250px;
  overflow: auto;

  .btn-collection {
    background-color: transparent;
    height: 37px;
    color: #bec6cb;
    white-space: nowrap;
    overflow: hidden;

    img {
      min-width: 35px;
      width: 35px;
      height: 35px;
      object-fit: cover;
      border-radius: 5px;
    }

    &:hover {
      background-color: rgb(65, 65, 65);
    }

    .collection-name {
      overflow: hidden;
    }
  }
}
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
  justify-content: center;
  align-items: center;
  grid-template-columns: 20% 20% 20% 20% 20%;
  gap: 10px 0px;
  max-height: 665px;
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
  .InventorySelection {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
    height: 350px;
    border: 1px solid #67c23a;
    border-radius: 4px;
    padding: 20px;
    text-align: center;
    overflow-y: auto;
  }
  width: 100%;
  margin: auto;
}

#return-btn::before {
  content: '???';
}

#return-btn {
  font-weight: 500;
  font-size: 14px;
  color: #9f979a !important;
  line-height: 20px;
  cursor: pointer;
}
</style>
