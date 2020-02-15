<template>
  <div id="app">
      <div class="row no-gutters">
        <div class="col-sm-3">
          <div class="toolbox">
            <div class="sticky-top bg-white shadow-sm p-2">
              <div class="form-group d-flex">
                <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
                <div class="flex-fill">
                  <select id="cityName" class="form-control" v-model="select.city" @change="removeMaker();updateMap(); select.name=''">
                      <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                      {{c.CityName}}
                      </option>
                  </select>
                </div>
              </div>
              <div class="form-group d-flex">
                <label for="area" class="mr-2 col-form-label text-right">地區</label>
                <div class="flex-fill">
                  <select id="area" class="form-control" v-model="select.name"  @change="removeMaker();updateMap();">

                    <template  v-for="name in cityName" v-if="name.CityName===select.city" >
                     <option value="" selected>請選擇</option>
                      <option :value="city.AreaName" v-for="city in name.AreaList" :key="city.AreaName">
                        {{city.AreaName}}
                      </option>
                    </template>
                  </select>
                </div>
              </div>
              <p class="mb-0 small text-muted text-right">請先選擇區域查看（綠色表示還有口罩）</p>
            </div>
            <ul class="list-group">
              <template v-for="item in pharmacies">
                <a class="list-group-item text-left" @click.prevent="openPopup(item)">
                  <h3>{{item.properties.name}}</h3>
                  <p class="mb-0">
                    成人口罩：{{item.properties.mask_adult}} | 兒童口罩：{{item.properties.mask_child}}
                  </p>
                    <p class="mb-0">地址：
                    {{item.properties.address}}
                  </p>
                  <p class="mb-0">電話：
                    {{item.properties.phone}}
                  </p>
                  <button type="button" class="btn  btn-sm btn-danger mr-2" v-if="item.properties.mask_adult">成人口罩  {{item.properties.mask_adult}}</button>
                  <button type="button" class="btn  btn-sm btn-secondary mr-2" v-else>成人口罩  0</button>
                  <button type="button" class="btn  btn-sm btn-warning" v-if="item.properties.mask_child">小孩口罩  {{item.properties.mask_child  }}</button>
                  <button type="button" class="btn  btn-sm btn-secondary" v-else>小孩口罩  0</button>
                </a>
              </template>
            </ul>
          </div>
        </div>
        <div class="col-sm-9">
          <div id="map"></div>
        </div>
      </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityName from './assets/cityName.json';
let osmMap={};

export default {
  name: 'App',
  data(){
    return {
      data:[],
      cityName,
      select:{
        city:'臺北市',
        name:''
      },
      pharmacies:[],
      layer:{}
    }
  },
  methods: {
    updateMap(){
      console.log(this.data);
      this.pharmacies=this.data.filter((pharmacy)=>
        (pharmacy.properties.county===this.select.city&&pharmacy.properties.town===this.select.name)
      );

      this.pharmacies.forEach((pharmacy)=>{
         const {geometry,properties}=pharmacy;
        L.marker([
            geometry.coordinates[1],
            geometry.coordinates[0],
        ]).addTo(osmMap).bindPopup(`
            <strong class="mb-1 d-block">${properties.name}</strong>
            <div class="mb-1">地址:
              <a href="https://www.google.com.tw/maps/place/${properties.address}">${properties.address}</a>
            </div>
            <div class="mb-1">電話:${properties.phone}</div>
            <div class="mb-1 ">注意事項:<span class="text-danger">${properties.note.replace('-','無')}</span></div>
            <small class="mb-2 d-block">最後更新時間:${properties.updated}</small>
            <button type="button" class="btn  btn-sm ${properties.mask_adult?`btn-danger`:`btn-secondary`}">成人口罩  ${properties.mask_adult ? `${properties.mask_adult}`:`0`}</button>
            <button type="button" class="btn  btn-sm ${properties.mask_child?`btn-warning`:`btn-secondary`}">小孩口罩  ${properties.mask_child ? `${properties.mask_child}`:`0`}</button>
        `);
      });
    },
    removeMaker(){
      osmMap.eachLayer((layer)=>{
        if(layer instanceof L.Marker){
          osmMap.removeLayer(layer);
        }
      });
    },
    openPopup(item){
      const {geometry,properties}=item;
      osmMap.panTo([geometry.coordinates[1], geometry.coordinates[0]]);
       L.marker([
            geometry.coordinates[1],
            geometry.coordinates[0],
        ]).addTo(osmMap).bindPopup(`
            <strong class="mb-1 d-block">${properties.name}</strong>
            <div class="mb-1">地址:
              <a href="https://www.google.com.tw/maps/place/${properties.address}">${properties.address}</a>
            </div>
            <div class="mb-1">電話:${properties.phone}</div>
            <div class="mb-1 ">注意事項:<span class="text-danger">${properties.note.replace('-','無')}</span></div>
            <small class="mb-2 d-block">最後更新時間:${properties.updated}</small>
            <button type="button" class="btn  btn-sm ${properties.mask_adult?`btn-danger`:`btn-secondary`}">成人口罩  ${properties.mask_adult ? `${properties.mask_adult}`:`0`}</button>
            <button type="button" class="btn  btn-sm ${properties.mask_child?`btn-warning`:`btn-secondary`}">小孩口罩  ${properties.mask_child ? `${properties.mask_child}`:`0`}</button>
        `).openPopup();
      
    }
  },
  mounted(){
      const url="https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
      this.$http.get(url).then((response) => {
          this.data=response.data.features;
          this.updateMap();
      });
      osmMap = L.map('map', {
        center: [25.03, 121.55],
        zoom: 15
      });

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>'
        }).addTo(osmMap);
      
  }
}
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  height: 100vh;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
</style>
