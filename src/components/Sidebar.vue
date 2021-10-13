<template>
  <div v-show="!isCollapsed" class="root ">
    <div class="d-flex flex-column flex-shrink-0 p-3 bg-light sidebar" :style="sideBarWidthStyle">
      <a href="/" class="d-flex align-items-center mb-3 mb-md-0 me-md-auto link-dark text-decoration-none">
        <span class="fs-8 seller-list-span">Seller list</span>
      </a>
      <hr>
      <div>
        <div class="form-group input-group-sm">
        <input type="search" class="form-control"  v-model="sellerSearch">
        </div>
      </div>
      <div class="seller-scroller">
        <div v-for="seller in sellersSearched" :key="seller.id">
          <ul class="nav nav-pills flex-column mb-auto" >
            <li class="nav-item">
              <input type="checkbox" class="custom-checkbox">
              <div class="checkbox-info">
                <div> {{seller.name}}
                  <span class="btn-sm btn-u" @click="() => updateSeller(seller.id)">[update]</span>
                  <span class="btn-sm btn-u" @click="() => updateSellerInfo(seller.id)">[info-u]</span>
                </div>
                <div class="s-id"> {{seller.id}} - {{seller.lastUpdated?.split("T")[0]}} <span @click="() => removeSeller(seller.id)">[x]</span></div>
                <div class="items-amt">{{seller.itemsAmount}} Items</div>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <hr>
      <div>
        <div class="form-group input-group-sm">
          <input type="search" class="form-control" placeholder="Name"  v-model="newSeller.name">
          <input type="search" class="form-control" placeholder="Id"  v-model="newSeller.id">
          <button type="button" class="form-control" @click="addSeller">
            Add seller
          </button>
          <button class="form-control"  @click="runUpdate">
            <img src="~../assets/Spin-1s-200px.gif" class="spinner" v-if="isUpdating"/>
            Update Seller
          </button>
          <button class="form-control"  @click="runTranslate">
            Translations
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import _ from "lodash";

export default {
  name: "Sidebar",
  data(){
    return {
      newSeller: {},
      data: {},
      isCollapsed: false,
      isUpdating: false,
      sellerSearch: ""
    }
  },
  computed: {
    sideBarWidthStyle(){
      this.isCollapsed;
      return { width: (this.isCollapsed) ? "50px;" : "230px;" }
    },
    sellersSearched(){
      this.data;
      return _.filter(this.data, seller => {
        return seller.name.toLowerCase().includes(this.sellerSearch.toLowerCase()) || seller.id.toLowerCase().includes(this.sellerSearch)
      })
    }
  },
  methods: {
    addSeller(){
      fetch("http://localhost:8070/sellers/add?" + new URLSearchParams({
        name: this.newSeller.name,
        id: this.newSeller.id
      }))
          .then(response => response.json())
          .then(data => this.data = data);
    },
    getSellers(){
      fetch("http://localhost:8070/sellers")
          .then(response => response.json())
          .then(data => this.data = data);
    },
    removeSeller(id){
      fetch("http://localhost:8070/sellers/remove?" + new URLSearchParams({
        id: id
      }))
          .then(response => response.json())
          .then(data => this.data = data);
    },
    runUpdate(){
      this.isUpdating = true;
      fetch("http://localhost:8070/sellers/update")
          .then(response => response.json())
          .then(data => {
            this.data = data;
            this.isUpdating = false;
            this.runUpdate();
          });
    },
    updateSeller(sellerid){
      this.isUpdating = true;
      fetch("http://localhost:8070/crawler/update?sellerid=" + sellerid)
          .then(response => response.json())
          .then(data => {
            this.data = data;
            this.isUpdating = false;
            this.runUpdate();
          });
    },
    updateSellerInfo(sellerid){
      this.isUpdating = true;
      fetch("http://localhost:8070/crawler/info?sellerid=" + sellerid)
          .then(response => response.json())
          .then(data => {
            this.data = data;
            this.isUpdating = false;
            this.runUpdate();
          });
    },
    runTranslate(){
      fetch("http://localhost:8070/items/translate")
    }
  },
  mounted() {
    this.getSellers();
  }
}
</script>

<style scoped>

  .spinner{
    width: 20px;
  }
  .seller-list-span{
    padding-top: 40px;
  }

  .sidebar{
    height: -webkit-fill-available;
  }

  .root{
    position: absolute;
    z-index: 999;
    width: 230px;
    height: -webkit-fill-available;
  }
  .nav-item{
    text-align: left;
    display: flex;
    align-content: center;
  }

  .seller-scroller{
    overflow-y: scroll;
  }

  .custom-checkbox{
    min-height: 15px;
    min-width: 15px;
    margin-right: 10px;
  }

  .btn-u{
    font-size: 8px;
  }

  .checkbox-info{
    display: flex;
    flex-direction: column;
    font-size: 12px;
  }

  .s-id{
    font-size: 9px;
  }

  .form-control{
    padding: 3px!important;
    margin-bottom: 4px;
  }

  .items-amt{
    font-size: 9px;
    color: gray;
  }
</style>