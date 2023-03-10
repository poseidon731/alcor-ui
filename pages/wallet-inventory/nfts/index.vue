<template lang="pug">
.nfts
  WalletNFTTab(
    :tabData='tabData',
    :currentTab='currentTab',
    :handleTab='handleTab',
    :handleSearch='handleSearch',
    :collectionData='collectionData',
    :handleCollection='handleCollection',
    :searchValue='search',
    :handleSearchValue='handleSearchValue'
  )
  //- HorizontalMenu(
  //-   :tabs='currentTab != "inventory" ? (currentTab === "listings" ? horizontalTabData.normalTabs : horizontalTabData.auctionTabs) : []',
  //-   :currentTab='currentHorizontalTab',
  //-   :handleTab='handleHorizonalTab'
  //- )
  div(v-if='detailCollectionMode')
    .d-flex.justify-content-between.align-items-center.mb-4
      h1.text-capitalize.set-collection-name {{ setCollectionName }}
      .progress-panel
        .completed-sets-count Computer Sets: 2
        el-progress.completed-sets-progress(
          :text-inside='true',
          :stroke-width='26',
          :percentage='70',
          color='#67C23A'
        )
    WalletSetTab
    .grid-container
      .d-flex.justify-content-center(
        v-for='(item, index) in collectionSets',
        :key='index'
      )
        NormalCard(
          :data='item',
          :mode='currentTab === "sets" && detailCollectionMode ? "setsList" : ""',
          :kindBut='currentTab != "inventory" ? "all" : ""'
          :getOverData='getInventoryOverData'
          :overData='overData'
        )
  div(v-else)
    div(v-if='currentTab === "inventory" || currentTab === "listings" || currentTab === "auctions" || currentTab === "sets"')
      .grid-container(v-if='loading')
        CustomSkeletonVue(
          v-for='item in 12',
          :key='item',
          :width='220',
          :height='380'
        )
      .grid-container(v-else)
        .d-flex.justify-content-center(
          v-for='(item, index) in inventoryData',
          :key='index'
        )
          NormalCard(
            :data='item',
            :mode='currentTab',
            :kindBut='currentTab != "inventory" ? "all" : ""'
            :getOverData='getInventoryOverData'
            :overData='overData'
          )
    div(v-else-if='currentTab === "sold" || currentTab === "bought"')
      .d-flex.justify-content-between(v-if='loading')
        CustomSkeletonVue(:width='600', :height='380')
        CustomSkeletonVue(:width='220', :height='380')
      div(v-else)
        DetailWithCardPanel(
          v-for='(item, index) in inventoryData',
          :key='index',
          :data='item',
          :mode='currentTab'
        )
</template>

<script>
import { mapState } from 'vuex'
import WalletNFTTab from '~/components/wallet/WalletNFTTab'
import CustomSkeletonVue from '~/components/CustomSkeleton'
import HorizontalMenu from '~/components/wallet/HorizontalMenu'
import NormalCard from '~/components/nft_markets/cards/NormalCard'
import DetailWithCardPanel from '~/components/nft_markets/DetailWithCardPanel'
import WalletSetTab from '~/components/wallet/WalletSetTab.vue'

export default {
  name: 'NFTs',
  components: {
    WalletNFTTab,
    NormalCard,
    HorizontalMenu,
    DetailWithCardPanel,
    WalletSetTab,
    CustomSkeletonVue,
  },
  data: () => ({
    search: '',
    loading: false,
    limit: 20,
    inventoryData: [],
    salesData: [],
    auctionsData: [],
    currentTab: '',
    currentHorizontalTab: 'open-auctoins',
    collectionData: [],
    currentCollectionName: '',
    horizontalTabData: {
      normalTabs: [
        {
          title: 'Sales',
          slug: 'sales',
        },
        {
          title: 'Auctions',
          slug: 'auctions',
        },
      ],
      auctionTabs: [
        {
          title: 'Open Auctions',
          slug: 'open-auctoins',
        },
        {
          title: 'Lost Auctions',
          slug: 'lost-auctions',
        },
      ],
    },
    tabData: [
      {
        title: 'Inventory',
        slug: 'inventory',
      },
      {
        title: 'Listings',
        slug: 'listings',
      },
      {
        title: 'Auctions',
        slug: 'auctions',
      },
      {
        title: 'Sold',
        slug: 'sold',
      },
      {
        title: 'Bought',
        slug: 'bought',
      },
      {
        title: 'Sets',
        slug: 'sets',
      },
    ],
    selectedCollectionName: '',
    collectionSets: [],
    overData: '',
    detailCollectionMode: false,
  }),
  computed: {
    ...mapState(['network', 'user']),
    userData() {
      return this.user
    },
    setCollectionName() {
      if (this.collectionSets.length) {
        return this.collectionSets[0].collection.name
      } else return ''
    },
  },
  watch: {
    userData(newUserData, oldUserData) {
      if (!oldUserData && newUserData) {
        const tab = this.$route.hash.split('#')[1]
        if (tab) {
          if (tab.split('-')[1]) {
            this.currentTab = 'sets'
            this.detailCollectionMode = true
            this.selectedCollectionName = this.$route.hash
              .split('#')[1]
              .split('-')[1]
          } else {
            this.currentTab = tab
          }
        } else this.currentTab = 'inventory'
      }
    },
    $route(to, from) {
      if (to.hash.includes('#sets-')) {
        this.currentTab = 'sets'
        this.detailCollectionMode = true
        this.selectedCollectionName = to.hash.split('#')[1].split('-')[1]
      } else {
        this.detailCollectionMode = false
        this.currentTab = to.hash.split('#')[1]
      }
    },
    currentTab(new_tab, old_tab) {
      this.getData(new_tab)
    },
    detailCollectionMode(new_mode, old_mode) {
      if (new_mode) {
        this.getCollectionSets()
      }
    },
    currentHorizontalTab(new_tab, old_tab) {
      if (new_tab === 'sales' || new_tab === 'open-auctions') {
        this.inventoryData = this.salesData
      } else this.inventoryData = this.auctionsData
    },
  },
  mounted() {
    this.getCollectionData()
    if (this.user) {
      if (this.$route.hash.includes('#')) {
        if (this.$route.hash.includes('#set-')) {
          this.currentTab = 'sets'
        } else {
          this.currentTab = this.$route.hash.split('#')[1]
          this.getData(this.$route.hash.split('#')[1])
        }
      } else this.currentTab = 'inventory'
    }
  },
  methods: {
    handleTab(value) {
      this.currentTab = value
    },
    handleHorizonalTab(value) {
      this.currentHorizontalTab = value
    },

    async getInventoryOverData(params) {
      this.overData = ''
      const templateStats = await this.getTemplateStats(params.collection_name, params.template_id)
      const specificAsset = await this.getSpecificAsset(params.collection_name, params.template_id)
      const assetsSales = await this.getAssetsSales(params.asset_id)
      const saleData = await this.getOverSale(params.collection_name, params.template_id)
      this.overData = {
        templateStats,
        specificAsset,
        assetsSales,
        saleData
      }
    },

    async getTemplateStats(collection_name, template_id) {
      return await this.$store.dispatch('api/getTemplateStats', {
        collection_name, template_id
      })
    },

    async getSpecificAsset(collection_name, template_id) {
      return await this.$store.dispatch('api/getAssets', {
        owner: this.user.name,
        collection_name,
        template_id
      })
    },

    async getAssetsSales(asset_id) {
      return await this.$store.dispatch('api/getAssetsSales', {
        asset_id
      })
    },

    async getOverSale(collectionName, template_id) {
      return await this.$store.dispatch('api/getSaleData', {
        collectionName, template_id, symbol: 'WAX', state: 1, limit: 1
      })
    },
    getData(value) {
      if (value === 'inventory') {
        this.getAssets()
      } else if (value === 'listings') {
        this.getListing()
        this.currentHorizontalTab = 'sales'
      } else if (value === 'auctions') {
        this.getAuctions()
        this.currentHorizontalTab = 'open-auctoins'
      } else if (value === 'sold') {
        this.getSold()
      } else if (value === 'bought') {
        this.getBought()
      } else if (value === 'sets') {
        this.getCollections()
      }
    },

    async getAssets() {
      this.loading = true
      const data = await this.$store.dispatch('api/getAssets', {
        owner: this.user.name,
      })
      console.log(data)
      this.inventoryData = data
      this.loading = false
    },

    async getListing() {
      this.loading = true
      const auctionData = await this.$store.dispatch('api/getAuctionData', {
        seller: this.user.name,
        state: '0,1,4',
      })
      const salesData = await this.$store.dispatch('api/getSales', {
        seller: this.user.name,
        state: '0,1,4',
      })
      // this.salesData = salesData
      // this.auctionsData = auctionData
      this.inventoryData = [...salesData, ...auctionData]
      this.loading = false
    },

    async getAuctions() {
      this.loading = true
      const openAuctions = await this.$store.dispatch('api/getAuctionData', {
        participant: this.user.name,
      })
      const lostAuctions = await this.$store.dispatch('api/getAuctionData', {
        bidder: this.user.name,
        buyer_blacklist: this.user.name,
        state: '3',
      })
      // this.salesData = openAuctions
      // this.auctionsData = lostAuctions
      this.inventoryData = [...openAuctions, ...lostAuctions]
      this.loading = false
    },

    async getSold() {
      this.loading = true
      const auctionsData = await this.$store.dispatch('api/getAuctionData', {
        seller: this.user.name,
        state: '3',
      })
      const salesData = await this.$store.dispatch('api/getSales', {
        seller: this.user.name,
        state: '3',
      })
      // this.salesData = openAuctions
      // this.auctionsData = lostAuctions
      this.inventoryData = [...salesData, ...auctionsData]
      this.loading = false
    },

    async getBought() {
      this.loading = true
      const auctionsData = await this.$store.dispatch('api/getAuctionData', {
        buyer: this.user.name,
        state: '3',
      })
      const salesData = await this.$store.dispatch('api/getSales', {
        buyer: this.user.name,
        state: '3',
      })
      // this.salesData = openAuctions
      // this.auctionsData = lostAuctions
      this.inventoryData = [...salesData, ...auctionsData]
      this.loading = false
    },

    async getCollections() {
      this.loading = true
      const data = await this.$store.dispatch('api/getCollectionsForSet')
      this.inventoryData = data.results
      this.loading = false
    },

    async getCollectionData() {
      const data = await this.$store.dispatch('api/getCollectionData', {
        data: 'ss',
      })
      this.collectionData = data
    },

    async getCollectionSets() {
      const data = await this.$store.dispatch('api/getCollectionSets', {
        collection_name: this.selectedCollectionName,
      })
      this.collectionSets = data.data
    },

    handleCollection(value) {
      this.currentCollectionName = value
      this.getAssets()
    },

    handleSearchValue(value) {
      this.search = value
    },

    handleSearch() {
      this.getAssets()
    },
  },
}
</script>
<style lang="scss">
.el-progress.completed-sets-progress {
  width: 220px;
  background: #161617;
  border-radius: 4px;
  .el-progress-bar__outer {
    border-radius: 4px;
    .el-progress-bar__inner {
      border-radius: 4px;
    }
  }
}
</style>
<style scoped lang="scss">
.table-header {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  .el-input {
    max-width: 300px;
  }
  .el-input__inner {
    background: transparent !important;
  }
}
.items {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 56px;
}
@media only screen and (max-width: 1040px) {
  .items {
    gap: 20px;
  }
}
@media only screen and (max-width: 840px) {
  .items {
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }
}
@media only screen and (max-width: 540px) {
  .items {
    grid-template-columns: 100%;
    gap: 10px;
  }
}

.set-collection-name {
  font-size: 36px;
}

div.grid-container {
  display: grid;
  grid-template-columns: auto auto auto auto;
  gap: 30px;
}
</style>
