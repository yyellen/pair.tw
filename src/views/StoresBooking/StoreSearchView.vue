<template>
  <div class="container">
    <div class="wrapper">
      <ul class="d-flex nav nav-tabs" id="myTab" role="tablist">
        <li class="flex-grow-1 nav-item" role="presentation">
          <button
            class="nav-link active"
            id="mapsearch-tab"
            data-bs-toggle="tab"
            data-bs-target="#mapsearch"
            type="button"
            role="tab"
            aria-controls="mapsearch"
            aria-selected="true"
            style="width: 100%"
          >
            地圖搜尋
          </button>
        </li>
        <li class="flex-grow-1 nav-item" role="presentation">
          <button
            class="nav-link"
            id="listsearch-tab"
            data-bs-toggle="tab"
            data-bs-target="#listsearch"
            type="button"
            role="tab"
            aria-controls="listsearch"
            aria-selected="false"
            style="width: 100%"
          >
            清單搜尋
          </button>
        </li>
        <li class="nav-item" role="presentation">
          <button
            class="nav-link"
            id="favstore-tab"
            data-bs-toggle="tab"
            data-bs-target="#favstore"
            type="button"
            role="tab"
            aria-controls="favstore"
            aria-selected="false"
            style="width: 80px"
          >
            <i class="fa-regular fa-heart"></i>
          </button>
        </li>
      </ul>
      <SearchBar @searchStore="searchStore" />
      <div class="tab-content" id="myTabContent">
        <div
          class="tab-pane fade show active rounded-bottom"
          id="mapsearch"
          role="tabpanel"
          aria-labelledby="mapsearch-tab"
        >
          <!-- <SearchBar @searchStore="searchStore" /> -->
          <MapSearch />
          <!-- 要改成點地標彈出對應的單一店家卡片 -->
          <div class="d-flex flex-column pt-4">
            <template v-for="(store, index) in searchStores" :key="index">
              <StoreListCard
                :store="store"
                :userInfo="userInfo"
                @update="getUserFavStore"
              ></StoreListCard>
            </template>
          </div>
        </div>

        <div
          class="tab-pane fade rounded-bottom"
          id="listsearch"
          role="tabpanel"
          aria-labelledby="listsearch-tab"
        >
          <!-- <SearchBar @searchStore="searchStore" /> -->
          <template v-for="(store, index) in searchStores" :key="index">
            <StoreListCard :store="store" :userInfo="userInfo"></StoreListCard>
          </template>
        </div>
        <div
          class="tab-pane fade rounded-bottom"
          id="favstore"
          role="tabpanel"
          aria-labelledby="favstore-tab"
        >
          <!-- <SearchBar @searchStore="searchStore" /> -->
          <template v-for="(store, index) in favStores" :key="index">
            <StoreListCard :store="store" :userInfo="userInfo"></StoreListCard>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SearchBar from "../../components/StoreBooking/SearchBar.vue";
import MapSearch from "../../components/StoreBooking/MapSearch.vue";
// import StoreListBox from "../../components/StoreBooking/StoreListBox.vue";
import StoreListCard from "../../components/StoreBooking/StoreListCard.vue";

import { db } from "@/firebase/config";
import {
  collection,
  getDocs,
  getDoc,
  query,
  orderBy,
  doc,
} from "firebase/firestore";
export default {
  name: "StoresView",
  components: {
    SearchBar,
    MapSearch,
    StoreListCard,
    // StoreListBox,
  },
  data() {
    return {
      user: this.$store.state.user,
      allStores: [],
      userInfo: "",
      searchStores: "",
      favStores: [],
    };
  },
  created() {
    // this.$store.dispatch("getAllStore");
    this.getAllStore();
    this.getUserInfo().then(() => {
      this.getUserFavStore();
    });
    console.log("All stores getted!");
    this.searchStores = this.allStores;
  },

  methods: {
    async getAllStore() {
      const dataBase = await collection(db, "Store");
      const dbResult = await getDocs(query(dataBase, orderBy("StoreName")));
      dbResult.docs.forEach((doc) => {
        this.allStores.push({ ...doc.data() });
      });
    },
    async getUserInfo() {
      let getUserInfo = await getDoc(doc(db, "Account", this.user.uid));
      this.userInfo = getUserInfo.data();
      // console.log(this.userInfo);
    },
    dofilter(val) {
      let oldStoresData = [...this.allStores];
      // let resultData = [];

      if (val.category && val.category != "全類別") {
        oldStoresData = oldStoresData.filter(
          (list) => list.Category == val.category
        );
      }
      if (val.district && val.district != "全地區") {
        oldStoresData = oldStoresData.filter(
          (list) => list.StoreLocation.District == val.district
        );
      }
      console.log("oldStoresData", oldStoresData);
      this.searchStores = oldStoresData;
      console.log("Final", this.searchStores);
      this.filterSearch(val);
    },
    filterSearch(val) {
      console.log(val.searchWords);
      let searchResult = [];
      let storeName = this.searchStores.filter((searchResult) =>
        searchResult.StoreName.match(val.searchWords)
      );
      searchResult = [...searchResult, ...storeName];
      this.searchStores = searchResult;
      return searchResult;
    },
    searchStore(val) {
      this.dofilter(val);
    },
    async getUserFavStore() {
      var favList = this.userInfo.FavStores;
      this.favStores = [];
      if (favList.length !== 0) {
        favList.forEach((id) => {
          // console.log(id);
          getDoc(doc(db, "Store", id)).then((doc) => {
            var result = doc.data();
            this.favStores.push(result);
          });
        });
      } else {
        this.favStores.length = 0
      }
    },
  },
};
</script>

<style scoped>
/* 下拉式選單被吃掉 */
.wrapper {
  padding: 0 !important;
  overflow: visible;
  /* overflow-x: hidden; */
}

/* 導覽頁籤tab */
.nav-item {
  background-color: #f6d55e;
}
.nav-item button {
  color: #4590d4 !important;
}
.nav-item .active {
  background-color: #4590d4 !important;
  color: #ffffff !important;
}
/* 導覽頁籤tab > 愛心圖示變色 */
.fa-heart {
  color: #ee5c5c;
}
.nav-item .active .fa-heart {
  color: #ffffff !important;
}
.tab-pane {
  background: #ffffff;
  padding: 1rem;
}
</style>
