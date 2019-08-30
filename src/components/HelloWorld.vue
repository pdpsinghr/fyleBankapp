<template>
  <div v-if="allDataLoaded" class="hello">
    <div class="input-group searchgroup">
      <input id="input-icon-left" type="text" placeholder="Search" name="input-icon-left" style="border: 0px;" v-model="search" required/>
      <label class="control-label" for="input-icon-left"></label><i class="bar"></i>
    </div>
    <div class="selectop">
      <label>Per Page Items: </label>
      <select v-model="perpage">
        <option>10</option>
        <option>20</option>
        <option>30</option>
      </select>
    </div>
    <table style="width: 100%">
      <tr class="parentThHeading">
        <th>Favouirate</th>
        <th v-for="(field, k) in tableFields" :key="k">{{field}}</th>
      </tr>
      <div></div>
      <tr v-for="(x, i) in dataPerPage" :key="i">
        <td><input type="checkbox" v-model="models[x.ifsc]" :value="models[x.ifsc]" @change="makeFav(x)" /></td>
        <td v-for="(field, j) in tableFields" :key="j" @click="openData()">{{x[field]}}</td>
      </tr>
    </table>
    <div class="paginatation">
        <span @click="setPage(currentPageIndex-1)" class="buttonsnp" style="border-radius: 40px 0px 0px 40px; margin-right: -3px;">Prev</span>
        <span v-for="i in pages" @click="setPage(i - 1)" :key="i" v-bind:class="{'current': (currentPageIndex + 1) === i}" class="buttonn">{{i}}</span>
        <span @click="setPage(currentPageIndex+1)" class="buttonsnp" style="border-radius: 0px 40px 40px 0px; margin-left: -3px;">Next</span>
      </div>
  </div>
  <div v-else>
    <h1>DATA LOADING, PLEASE WAIT........</h1>
  </div>
</template>

<script>
/* eslint-disable */
import {_} from 'vue-underscore'
import axios from 'axios'
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data () {
    return {
      allDataLoaded: false,
      models: {},
      allBankList: [],
      tableFields: {bank_id: 'bank_id', ifsc: 'ifsc', bank_name:'bank_name', branch: 'branch', district: 'district', city: 'city', state: 'state', address: 'address'},
      perpage: 20,
      currentPageIndex: 0,
      search: '',
    }
  },
  mounted () {
    axios.get(`https://vast-shore-74260.herokuapp.com/banks?city=MUMBAI `).then(r => {
      this.allBankList = r.data
      this.allDataLoaded = true
      var allFavr = JSON.parse(localStorage.getItem('favBanks'))
      allFavr.map(r => {
        this.models[r] = true
      })
    }).catch(err => {
      console.log(err)
    })
  },
  computed: {
    pages: function () {
      var filteredData = this.filteredData
      var x = Math.ceil(filteredData.length / this.perpage)
      var arr = []
      for (var i = 1; i <= x; i++) {
        arr.push(i)
      }
      if (arr.slice(this.currentPageIndex, this.currentPageIndex + 10).length < 10) {
        return arr.slice(x - 10, x + 1)
      } else {
        return arr.slice(this.currentPageIndex, this.currentPageIndex + 10)
      }
    },
    dataPerPage: function () {
      return this.filteredData.slice(this.currentPageIndex * this.perpage, this.currentPageIndex * this.perpage + this.perpage)
    },
    filteredData: function () {
      var data = this.allBankList
      if (this.search !== '') {
        data = _.filter(this.allBankList, (d) => {
          var allValues = Object.values(d)
          var valu = allValues.filter(d => {
            var d = d.toString()
            return d.includes(this.search)
          }).length
          return valu > 0 ? true : false
        })
      }
      return data
    }
  },
  methods: {
    makeFav: function (data) {
      var allFavr = JSON.parse(localStorage.getItem('favBanks'))
      if (allFavr == undefined) {
        var allFavr = []
        localStorage.setItem('favBanks', JSON.stringify(allFavr))
        allFavr.push(data.ifsc)
        localStorage.setItem('favBanks', JSON.stringify(allFavr))
      } else {
          var index = allFavr.indexOf(data.ifsc)
          if (index !== -1) {
            allFavr.splice(index, 1)
            localStorage.setItem('favBanks', JSON.stringify(allFavr))
          } else {
            allFavr.push(data.ifsc)
            localStorage.setItem('favBanks', JSON.stringify(allFavr))
          }
      }
    },
    setPage: function (index) {
      var length = Math.ceil(this.filteredData.length / this.perpage)
      if (index >= 0 && index < length) {
        this.currentPageIndex = index
      }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
tr {
  height: 41px;
}
.parentThHeading {
  color: rgb(74, 227, 135);
  border-bottom: 2px solid black;
  padding: 10px;
  height: 50px;
}
 .parentTable {
   width:100%;
   text-align: center;
   flex-shrink: 0;
   height: 500px;
 }
.backParent {
  position: relative;
  width: 66%;
  text-align: right;
  top: 20px;
}
 .searchgroup {
    width: 180px;
    border-bottom: 1px solid black;
    margin: 20px 20px 0px 20px;
  }
  .paginatation {
    flex-shrink: 0;
    text-align: center;
    height: 120px;
    position: relative;
    top: 50px;
  }
  input:focus {
      outline: none;
  }
  hr {
    display: block;
    margin-top: 0.5em;
    margin-bottom: 0.5em;
    margin-left: auto;
    margin-right: auto;
    border-style: inset;
    border-width: 1px;
  }
  .buttonn {
    color: white;
    width: 50px;
    height: 30px;
    background-color: #4ae387;
    font-size: 16px;
    padding: 20px;
  }
  .buttonsnp {
    color: white;
    width: 55px;
    height: 30px;
    background-color: #4ae387;
    font-size: 16px;
    padding: 20px;
  }
  .wrapAllTable {
    background-color: white;
    box-shadow: 0 14px 28px rgba(0,0,0,0.25), 0 10px 10px rgba(0,0,0,0.22);
  }
  .current {
    background: #1EC260;
  }
  .popup {
    background: black;
    width: 100px;
    height: 100px
  }
  .th .td {
    border: 2px solid black;
    color: #0e0e0e;
  }
  .searchIcon {
    margin-right: 10px;
    margin-top: 7px;
    color: rgb(153, 162, 171);
    transform: rotate(90deg);
  }
  .capitalize {
     text-transform: capitalize;
  }
  .selectop {
    position: absolute;
    top: 62px;
    right: 109px;
  }
  tr:nth-child(even) {background-color: rgb(239,244,245) !important;}
</style>
