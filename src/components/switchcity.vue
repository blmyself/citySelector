<template>
  <div>
    <div class="input">
      <input placeholder="输入城市名或拼音查询" placeholder-style="font-size: 13px" :value="inputName" @input="bindKeyInput" @blur="bindBlur"/>
    </div>

    <div class="container-inner">
      <div class="searchLetter touchClass">
        <div class="thishotText" @click="hotCity">
          <div style="margin-top:0;">当前</div>
          <div style="margin-top:0;">热门</div>
        </div>
        <div v-for="(item, idx) in searchLetter" :key="idx" style="color:#8BC34A;font-size:20rpx;" :data-letter="item.name" @click="clickLetter">
          {{ item.name }}
        </div>
      </div>

      <div class="container">

        <block v-if="isShowLetter">
          <div class="showSlectedLetter">
            {{ toastShowLetter }}
          </div>
        </block>

        <scroll-view scroll-y="true" v-bind:style="{height: winHeight + 'px'}" :scroll-into-view="scrollTopId">
          <ul class="ul" id="completelist">
            <li class="li" v-for="(item, idx) in completeList" :key="idx" :data-city="item.city" :data-code="item.code" @click="bindCity">
              {{ item.city }}
            </li>
          </ul>

          <div v-if="condition" class="hotcity-common" id="selectcounty">选择区县</div>
          <div v-if="condition" class="county">
            <div class="hotcity" v-for="(item, idx) in countyList" :key="idx">
              <div class="weui-grid" style="margin-right: 16rpx;" :data-code="item.id" :data-city="item.fullname" @click="bindCounty">
                <div class="weui-grid__label">{{ item.fullname }}</div>
              </div>
            </div>
          </div>

          <div class="selectCity">
            <div class="hotcity-common" @click="reGetLocation" id="currentcity">重新定位城区</div>
            <div class="thisCityName" :data-city="city" :data-code="currentCityCode">{{ city }}</div>

            <div class="hotcity-common">热门城市</div>
            <div class="weui-grids">
              <div class="weui-grid" :data-code="110000" data-city="北京市" @click="bindCity">
                <div class="weui-grid__label">北京市</div>
              </div>
              <div class="weui-grid" :data-code="310000" data-city="上海市" @click="bindCity">
                <div class="weui-grid__label">上海市</div>
              </div>
              <div class="weui-grid" :data-code="440100" data-city="广州市" @click="bindCity">
                <div class="weui-grid__label">广州市</div>
              </div>
            </div>
            <div class="weui-grids">
              <div class="weui-grid" :data-code="440300" data-city="深圳市" @click="bindCity">
                <div class="weui-grid__label">深圳市</div>
              </div>
              <div class="weui-grid" :data-code="330100" data-city="杭州市" @click="bindCity">
                <div class="weui-grid__label">杭州市</div>
              </div>
              <div class="weui-grid" :data-code="320100" data-city="南京市" @click="bindCity">
                <div class="weui-grid__label">南京市</div>
              </div>
            </div>
            <div class="weui-grids">
              <div class="weui-grid" :data-code="420100" data-city="武汉市" @click="bindCity">
                <div class="weui-grid__label">武汉市</div>
              </div>
              <div class="weui-grid" :data-code="120000" data-city="天津市" @click="bindCity">
                <div class="weui-grid__label">天津市</div>
              </div>
              <div class="weui-grid" :data-code="610100" data-city="西安市" @click="bindCity">
                <div class="weui-grid__label">西安市</div>
              </div>
            </div>
          </div>

          <div class="selection" v-for="(item, idx) in cityList" :key="idx">
            <div class="item_letter" :id="item.initial">{{ item.initial }}</div>
            <div class="item_city" v-for="(cityItem, index) in item.cityInfo" :key="index" :data-code="cityItem.code" :data-city="cityItem.city" @click="bindCity">
              {{cityItem.city}}
            </div>
          </div>
        </scroll-view>
      </div>
    </div>
  </div>
</template>

<script>
import city from '../utils/city.js';
import { CITY_GET_LOCATION, CITY_SELECT_COUNTY, CITY_CHANGE_CODE, CITY_CHANGE_COUNTY } from '../store/mutation-types';
import { mapActions, mapGetters } from 'vuex';
export default {
  computed: {
    ...mapGetters('city', {
      city: 'city',
      county: 'county',
      currentCityCode: 'currentCityCode',
      defaultCity: 'defaultCity',
      defaultCounty: 'defaultCounty',
      countyList: 'countyList'
    })
  },
  data() {
    return {
      searchLetter: [],
      showLetter: '',
      winHeight: 0,
      cityList: [],
      isShowLetter: false,
      // 置顶id
      scrollTopId: '',
      hotcityList: [],
      inputName: '',
      completeList: [],
      condition: false,
      detailAddress: '',
      toastShowLetter: ''
    };
  },
  created() {
    const searchLetter = city.searchLetter;
    const cityList = city.cityList();
    const sysInfo = wx.getSystemInfoSync();
    console.log(sysInfo);
    const winHeight = sysInfo.windowHeight;
    const itemH = winHeight / searchLetter.length;
    let tempArr = [];

    searchLetter.map(
      (item, index) => {
        let temp = {};
        temp.name = item;
        temp.tHeight = index * itemH;
        temp.bHeight = (index + 1) * itemH;
        tempArr.push(temp);
      }
    );

    this.winHeight = winHeight;
    this.itemH = itemH;
    this.searchLetter = tempArr;
    this.cityList = cityList;

    this.getLocation();
  },
  methods: {
    ...mapActions('city', {
      'getLocation': CITY_GET_LOCATION,
      'selectCounty': CITY_SELECT_COUNTY,
      'changeCity': CITY_CHANGE_CODE,
      'changeCounty': CITY_CHANGE_COUNTY
    }),
    clickLetter(e) {
      const showLetter = e.currentTarget.dataset.letter;
      this.toastShowLetter = showLetter;
      this.isShowLetter = true;
      this.scrollTopId = showLetter;

      const self = this;
      setTimeout(() => {
        self.isShowLetter = false;
      }, 500);
    },
    reGetLocation() {
      this.getLocation();
    },
    // 选择城市
    bindCity(e) {
      this.condition = true;
      this.changeCity({
        city: e.currentTarget.dataset.city,
        code: e.currentTarget.dataset.code
      });
      this.scrollTopId = 'selectcounty';
      this.completeList = [];

      this.selectCounty();
    },
    bindCounty(e) {
      this.county = e.currentTarget.dataset.city;
      this.changeCounty({
        county: e.currentTarget.dataset.city
      });
      wx.switchTab({
        url: '/pages/index'
      });
    },
    hotCity() {
      this.scrollTopId = 'currentcity';
    },
    bindScroll(e) {
      console.log(e.detail);
    },
    bindBlur(e) {
      this.inputName = '';
    },
    bindKeyInput(e) {
      this.inputName = e.mp.detail.value;
      // 空搜索框时 取消匹配显示
      if (this.inputName.length < 1) {
        this.completeList = [];
      }
      this.scrollTopId = 'completelist';
      this.auto();
    },
    auto() {
      let inputSd = this.inputName.trim();
      let sd = inputSd.toLowerCase();
      let num = sd.length;
      const cityList = city.cityObjs;
      let finalCityList = [];

      let temp = cityList.filter(
        item => {
          let text = item.short.slice(0, num).toLowerCase();
          // eslint-disable-next-line
          return (text && text == sd);
        }
      );

      // 在城市数据中，添加简拼到“shorter”属性，就可以实现简拼搜索
      let tempShorter = cityList.filter(
        itemShorter => {
          if (itemShorter.shorter) {
            let textShorter = itemShorter.shorter.slice(0, num).toLowerCase();
            // eslint-disable-next-line
            return (textShorter && textShorter == sd);
          }
        }
      );

      let tempChinese = cityList.filter(
        itemChinese => {
          let textChinese = itemChinese.city.slice(0, num);
          // eslint-disable-next-line
          return (textChinese && textChinese == sd);
        }
      );

      if (temp[0]) {
        temp.map(
          item => {
            let testObj = {};
            testObj.city = item.city;
            testObj.code = item.code;
            finalCityList.push(testObj);
          }
        );
        this.completeList = finalCityList;
      } else if (tempShorter[0]) {
        tempShorter.map(
          item => {
            let testObj = {};
            testObj.city = item.city;
            testObj.code = item.code;
            finalCityList.push(testObj);
          }
        );
        this.completeList = finalCityList;
      } else if (tempChinese[0]) {
        tempChinese.map(
          item => {
            let testObj = {};
            testObj.city = item.city;
            testObj.code = item.code;
            finalCityList.push(testObj);
          }
        );
        this.completeList = finalCityList;
      }
    }
  }
};
</script>

<style>
.container-inner {
  display: flex;
  flex-direction: row-reverse;
}

.container {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  padding: 10rpx;

}

input {
  text-align: center;
  font-size: 32rpx;
  padding: 5px;
}

.searchLetter {
  flex-shrink: 0;
  width: 80rpx;
  text-align: center;
  display: flex;
  flex-direction: column;
  color: #666;
}

.searchLetter view {
  margin-top: 20rpx;
}

.touchClass {
  background-color: #fff;
  color: #fff;
  padding-top: 16rpx;
  padding-bottom: 16rpx;
}

.showSlectedLetter {
  background-color: rgba(0, 0, 0, 0.5);
  color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 50%;
  left: 50%;
  margin: -100rpx;
  width: 200rpx;
  height: 200rpx;
  border-radius: 20rpx;
  font-size: 52rpx;
  z-index: 1;
}

.selection {
  display: flex;
  width: 100%;
  flex-direction: column;
  margin-top: 10rpx;
}

.selectCity {
  padding: 16rpx;
  background-color: #f5f5f5;
  margin-bottom: -10rpx;
}

.item_letter {
  display: flex;
  background-color: #f5f5f5;
  height: 40rpx;
  padding-left: 34rpx;
  align-items: center;
  font-size: 24rpx;
  color: #666;
}

.item_city {
  display: flex;
  background-color: #fff;
  height: 100rpx;
  padding-left: 34rpx;
  align-items: center;
  border-bottom: 1rpx solid #ededed;
  font-size: 24rpx;
  color: #666;
}

.hotcity-common {
  font-size: 24rpx;
  color: #666;
  padding-bottom: 0;
  margin: 8rpx 0;
  margin-left: 16rpx;
}

.hotcity {
  padding-right: 50rpx;
  margin: auto;
}

.thisCityName {
  display: inline-block;
  border: 1rpx solid #8BC34A;
  border-radius: 8rpx;
  padding: 10rpx 0;
  font-size: 24rpx;
  color: #8BC34A;
  text-align: center;
  min-width: 149.5rpx;
  margin: 16rpx 0;
}

.thishotText {
  color: #8BC34A;
  font-size: 20rpx;
  margin: 0 !important;
}

.slectCity {
  border-color: #8BC34A !important;
}

.slectCity view {
  color: #8BC34A !important;
}

.weui-grid {
  padding: 10rpx 0;
  width: 200rpx;
  box-sizing: border-box;
  border: 1rpx solid #ececec;
  border-radius: 8rpx;
  background-color: white;
  margin: 8rpx 0;
}

.weui-grids {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  border: none;
}

.weui-grid__label {
  display: block;
  text-align: center;
  color: #333;
  font-size: 24rpx;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}

.ul {
  display: block;
  color: grey;
  margin-left: 20rpx;
}

.li {
  display: block;
  font-weight: 100;
  font-size: 28rpx;
  padding: 16rpx 0;
}

input {
  background-color: #eee;
}

.input {
  padding: 16rpx;
  border-bottom: 1rpx solid #f1f1f1;
}

.county {
  display: flex;
  flex-wrap: wrap;
}
</style>
