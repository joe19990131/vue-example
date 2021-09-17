<template>
  <div class="point-change-body">
    <MemberNav class="memberNav" page="point"></MemberNav>
    <div class="point-change-content">
      <span>{{ $t('text.point_change_title') }}</span>
      <div class="content-header">
        <div class="header-left">
          <div class="search">
            <span>{{ $t('text.point_change_search_title') }}</span>
            <div class="search-row">
              <validation-observer ref="form" v-slot="{ handleSubmit }">
                <b-form @submit.stop.prevent="handleSubmit(search)">
                  <div class="date-picker-area">
                    <Input
                      type="date"
                      class="date-picker"
                      :rules="`required|date_reverse:${[endDate, 'sd']}|date_out_range:${endDate}`"
                      v-model="startDate"
                      :inputStyle="{ lineHeight: '28px', fontSize: '12px', width: '150px', width: '100%', height: '48px !important' }"
                      :min="twoYearsAgo"
                      :max="today"
                    />
                    <Input
                      type="date"
                      class="date-picker"
                      :rules="`required|date_reverse:${[startDate, 'ed']}|date_out_range:${startDate}`"
                      v-model="endDate"
                      :inputStyle="{ lineHeight: '28px', fontSize: '12px', width: '150px', width: '100%', height: '48px !important' }"
                      :min="twoYearsAgo"
                      :max="today"
                    />
                    <b-button class="search-btn" type="submit">
                      {{ $t('text.search') }}
                    </b-button>
                  </div>
                  <div class="date-picker-area-mobile">
                    <Input
                      type="date"
                      class="date-picker"
                      :rules="`required|date_reverse:${[endDate, 'sd']}|date_out_range:${endDate}`"
                      v-model="startDate"
                      :inputStyle="{ lineHeight: '28px', fontSize: '10px', width: '120px', width: '100%', height: '48px !important' }"
                      :min="twoYearsAgo"
                      :max="today"
                    />
                    <Input
                      type="date"
                      class="date-picker"
                      :rules="`required|date_reverse:${[startDate, 'ed']}|date_out_range:${startDate}`"
                      v-model="endDate"
                      :inputStyle="{ lineHeight: '28px', fontSize: '10px', width: '120px', width: '100%', height: '48px !important' }"
                      :min="twoYearsAgo"
                      :max="today"
                    />
                    <b-button class="search-btn" type="submit">
                      {{ $t('text.search') }}
                    </b-button>
                  </div>
                </b-form>
              </validation-observer>
            </div>
          </div>
        </div>
        <div class="header-right">
          <div class="total-point">
            <span class="title" style="fontSize:20px">{{ $t('text.point_change_current_point') }}</span>
            <div style="textAlign: right; width: 60%;">
              <span class="context">{{ memberSummary.remainPoints && memberSummary.remainPoints | currency }}</span>
              <span>{{ `${' '}` }}</span>
              <span style="fontSize:18px; color: rgba(189, 53, 25, 1); fontWeight:bold; display: inline-block;">{{ pointName }}</span>
            </div>
          </div>
          <div class="point-detail">
            <div class="expire-point">
              <span class="title">{{ $t('text.point_change_expire_point') }}</span>
              <span class="context">{{ memberSummary.expiringSoonPoints && memberSummary.expiringSoonPoints | currency }}{{ pointName }}</span>
            </div>
            <div class="expire-time">
              <span class="title">{{ $t('text.point_change_expire_time') }}</span>
              <span class="context">{{ memberSummary && memberSummary.expiry | date }}</span>
              <span></span>
            </div>
          </div>
        </div>
      </div>
      <span class="empty-title" v-if="filtedItems.length === 0 && listForMobile && listForMobile.length === 0">{{ $t('text.reward_record_empty') }}</span>
      <b-table class="table" v-if="filtedItems.length !== 0" striped :items="filtedItems" :fields="fields" bordered>
        <template #cell(activityName)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <span>{{ filtedItems[data.index].type === 1 ? filtedItems[data.index].activityName : events[filtedItems[data.index].type - 1] }}</span>
        </template>
        <template #cell(points)="data">
          <span :style="{ color: filtedItems[data.index].type === 2 || filtedItems[data.index].type === 3 || filtedItems[data.index].type === 5 ? '#1d9149' : '#e02020' }"
            >{{ filtedItems[data.index].type === 2 || filtedItems[data.index].type === 3 || filtedItems[data.index].type === 5 ? '- ' : '+ ' }}{{ data.value | currency }}</span
          >
        </template>
        <template #cell(productNames)="data">
          <span>{{ localeFormat(data.value) }}</span>
        </template>
        <template #cell(createdAt)="data">
          <span>{{ data.value | time }}</span>
        </template>
        <template #cell(expiry)="data">
          <span
            :style="{
              color: dateValue(data.value) >= dateValue(today) ? '#2b2b2b' : '#e02020',
            }"
            >{{ data.value === '' ? '' : data.value | date }}{{ dateValue(data.value) >= dateValue(today) ? '' : $t('text.point_change_table_failed') }}</span
          >
        </template>
      </b-table>
      <div class="card-wrapper">
        <div class="cards" v-for="(item, key) in listForMobile" :key="key">
          <span>{{ item.month }}</span>
          <PointCard v-for="(element, key) in item.data" :key="key" :data="element"></PointCard>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MemberNav from 'Components/memberNav/MemberNav';
import PointCard from 'Components/card/PointCard';
import moment from 'moment';
import store from 'Store/store';
export default {
  name: 'PointsChange',
  components: {
    MemberNav,
    PointCard,
  },
  data() {
    return {
      today: moment().format('YYYY-MM-DD'),
      twoYearsAgo: moment()
        .add(-2, 'y')
        .format('YYYY-MM-DD'),
      startDate: moment()
        .add(-30, 'd')
        .format('YYYY-MM-DD'),
      endDate: moment()
        .add(0, 'd')
        .format('YYYY-MM-DD'),
      isSearch: false,

      filtedItems: [],
      memberSummary: {
        expiringSoonPoints: null,
        remainPoints: null,
      },
      listForMobile: null,
    };
  },

  computed: {
    maxDateRange() {
      const maxDateRange = moment(this.startDate)
        .add(30, 'd')
        .format('YYYY-MM-DD');
      return maxDateRange;
    },
    fields() {
      return [
        {
          key: 'activityName',
          label: this.$t('text.point_change_field_event_name'),
          sortable: false,
        },
        {
          key: 'productNames',
          label: this.$t('text.point_change_field_goods_name'),
          sortable: false,
        },
        {
          key: 'createdAt',
          label: this.$t('text.point_change_field_time'),
          sortable: false,
        },
        {
          key: 'points',
          label: this.pointName + this.$t('text.point_change_field_change'),
          sortable: false,
        },
        {
          key: 'expiry',
          label: this.$t('text.point_change_field_limit_time'),
          sortable: false,
        },
      ];
    },
    pointName() {
      if (store.state.member.memberSetting && store.state.member.memberSetting.pointSetting) {
        return ' ' + store.state.member.memberSetting.pointSetting.name;
      }
      return '';
    },
    events() {
      return [
        this.$t('text.point_change_event_1'),
        this.$t('text.point_change_event_2'),
        this.$t('text.point_change_event_3'),
        this.$t('text.point_change_event_4'),
        this.$t('text.point_change_event_5'),
      ];
    },
    defaultLang() {
      if (store.state.main.productInfo) {
        return store.state.main.productInfo.defaultLang;
      }
      return '';
    },
  },
  created() {
    //const days = moment(this.endDate).diff(moment(this.startDate), 'days');
    store.dispatch('member/getMemberSummary').then(() => {
      this.memberSummary = store.state.member.memberSummary;
    });
    if (this.startDate !== null && this.endDate !== null) {
      this.isSearch = true;
      let requestData = {
        From: this.startDate,
        To: this.endDate,
        TimezoneId: 1, // 待串
      };
      store.dispatch('member/getPointChangeList', requestData).then(() => {
        this.filtedItems = store.state.member.pointChangeList;
        let list = store.state.member.pointChangeList;
        let map = [];
        let nList = [];
        list.forEach((item) => {
          if (!map.includes(item.createdAt.substr(0, 7))) {
            nList.push({
              month: item.createdAt.substr(0, 7),
              data: [item],
            });
            map.push(item.createdAt.substr(0, 7));
          } else {
            //因為forEach不支援break，所以使用every實現
            nList.every((nItem) => {
              if (nItem.month === item.createdAt.substr(0, 7)) {
                nItem.data.push(item);
                return false;
              }
              return true;
            });
          }
        });
        this.listForMobile = nList;
      });
    }
  },
  watch: {
    endDate: function() {},
  },
  methods: {
    search() {
      if (this.startDate !== null && this.endDate !== null) {
        this.isSearch = true;
        let requestData = {
          From: moment.utc(this.startDate).format('YYYY-MM-DD'),
          To: moment.utc(this.endDate).format('YYYY-MM-DD'),
          TimezoneId: 1, // 待串
        };
        store.dispatch('member/getPointChangeList', requestData).then(() => {
          this.filtedItems = store.state.member.pointChangeList;
          let list = store.state.member.pointChangeList;
          let map = [];
          let nList = [];
          list.forEach((item) => {
            if (!map.includes(item.createdAt.substr(0, 7))) {
              nList.push({
                month: item.createdAt.substr(0, 7),
                data: [item],
              });
              map.push(item.createdAt.substr(0, 7));
            } else {
              //因為forEach不支援break，所以使用every實現
              nList.every((nItem) => {
                if (nItem.month === item.createdAt.substr(0, 7)) {
                  nItem.data.push(item);
                  return false;
                }
                return true;
              });
            }
          });
          this.listForMobile = nList;
        });
      }
    },
    dateValue(date) {
      return moment(date);
    },
    localeFormat(item) {
      if (item[this.locale]) {
        return item[this.locale];
      }
      if (item[this.defaultLang]) {
        return item[this.defaultLang];
      }
      if (item['zh_TW']) {
        return item['zh_TW'];
      }
      if (item['zh_CN']) {
        return item['zh_CN'];
      }
      if (item['en']) {
        return item['en'];
      }
    },
  },
  filters: {
    time(time) {
      if (time) {
        return moment
          .utc(time)
          .local()
          .format('YYYY.MM.DD HH:mm:ss');
      }
    },
    date(time) {
      if (time) {
        return `${time.substr(0, 4)}/${time.substr(5, 2)}/${time.substr(8, 2)}`;
      }
    },
    currency(price) {
      let filtedPrise = '';
      if (price !== null) {
        filtedPrise = price.toFixed(0).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
      }
      return filtedPrise;
      /* return price.toLocaleString('en-US');  寫法二：轉為千分號*/
    },
  },
};
</script>

<style scoped>
.empty-title {
  color: rgba(43, 43, 43, 1);
  font-size: 20px;
  font-weight: bold;
  align-self: flex-start;
  margin-left: auto;
  margin-top: 40px;
  margin-right: auto;
  text-align: center;
}

.cards {
  display: none;
  width: 98%;
  margin-left: 1%;
  padding: 8px;
  flex-direction: column;
  justify-content: flex-start;
  align-content: center;
}

.point-change-body {
  max-width: 1440px;
  background-color: rgba(243, 243, 243, 1);
  padding: 20px 40px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-content: center;
  margin-left: auto;
  margin-right: auto;
}

.point-change-content {
  height: 970px;
  width: calc(100% - 446px - 16px);
  background-color: #fff;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-content: center;
  overflow: scroll;
}

.content-header {
  padding: 42px 40px;
  padding-bottom: 20px;
  padding-top: 0px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-content: center;
}

.header-left {
  width: 60%;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-content: flex-start;
}

.point-change-content > span:not(.empty-title) {
  color: rgba(189, 53, 25, 1);
  font-size: 32px;
  font-weight: bolder;
  align-self: flex-start;
  margin-left: 42px;
  margin-top: 42px;
}

.search {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-content: flex-start;
}

.search > span {
  align-self: flex-start;
  font-size: 18px;
  font-weight: bold;
  color: rgba(45, 57, 69, 0.7);
}

.search-row {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-content: center;
}

.search-row > span {
  width: 100%;
}

.date-picker-area {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-content: center;
  padding: 1px;
  width: 100%;
}
.date-picker-area-mobile {
  display: none;
  flex-direction: row;
  justify-content: flex-start;
  align-content: center;
  padding: 1px;
  width: 100%;
}

.date-picker {
  margin-left: 0px;
  margin-right: 10px;
  border-radius: 4px;
}

.date-picker >>> .base-input-frame {
  padding: 0;
}

.form-control {
  height: 56px !important;
}

.search-btn {
  height: 48px;
  width: 100px;
  text-align: center;
  line-height: 24px !important;
  background-color: rgba(184, 58, 36, 1);
  color: #fff;
  font-weight: bold;
  font-size: 20px;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 4px;
}

.header-right {
  width: 40%;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-content: flex-end;
  padding-left: 16px;
  padding-bottom: 0px;
}

.total-point {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-content: center;
  padding-bottom: 12px;
  padding-top: 4px;
  border-bottom: 1px solid lightgray;
}

.total-point > .title {
  align-self: flex-start;
  width: 50%;
}

.total-point .context {
  color: rgba(189, 53, 25, 1);
  font-size: 30px;
  font-weight: bold;
  line-height: 28px;
}

.point-detail {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-content: center;
  margin-top: 24px;
}

.expire-point,
.expire-time {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-content: center;
}

.expire-point > .context {
  font-weight: bold;
}

.expire-point > .context,
.expire-time > .context {
  width: 60%;
  text-align: right;
}

.header-right .title {
  font-size: 18px;
  color: rgba(43, 43, 43, 1);
  font-weight: bold;
  align-self: flex-start;
  width: 40%;
  text-align: left;
}

.table {
  margin: 0px 40px;
  width: calc(100% - 80px);
}

@media (max-width: 1439px) {
  .content-header {
    flex-direction: column-reverse;
    align-content: center;
    padding: 8px;
    width: 98%;
    align-self: center;
  }

  .header-right,
  .header-left {
    width: 100%;
    align-self: center;
  }

  .header-right {
    padding: 12px 16px;
    border-radius: 8px;
    box-shadow: 0 2px 14px 0 rgba(178, 178, 178, 0.5);
    background-color: #ffffff;
  }

  .table {
    display: none;
  }

  .cards {
    display: flex;
  }
  .cards > span {
    font-size: 18px;
    font-weight: bold;
    color: #2b2b2b;
    margin: 16px;
  }
  .date-picker-area {
    font-size: 12px;
    padding-top: 8px;
  }
  .date-picker-area > button {
    font-size: 16px;
  }

  .date-picker {
    width: calc(50% - 42px);
  }

  .search {
    margin-top: 32px;
  }

  .search-row {
    justify-content: center;
  }

  .search-btn {
    width: 60px;
    padding: 0px;
    text-align: center;
  }
}

@media (max-width: 1024px) {
  .point-change-content {
    width: calc(100% - 375px - 16px);
  }
}

@media (max-width: 996px) {
  .point-change-body {
    padding: 0px;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-content: center;
  }

  .point-change-content {
    height: auto;
    width: 100%;
    overflow: unset;
  }

  .memberNav {
    display: none;
  }

  .point-change-content {
    background-color: rgba(243, 243, 243, 1);
  }

  .point-change-content > span {
    align-self: center;
    margin: 0px;
    margin-top: 20px;
  }

  .point-change-content > span:not(.empty-title) {
    align-self: center;
    margin: auto;
    margin-top: 20px;
  }
}
@media (max-width: 375px) {
  .date-picker-area {
    display: none;
  }
  .date-picker-area-mobile {
    font-size: 12px;
    padding-top: 8px;
    display: flex;
  }
  .date-picker-area-mobile > button {
    font-size: 16px;
  }
  .date-picker-area-mobile > .date-picker {
    margin-right: 8px;
  }
}
@media (max-width: 320px) {
  .date-picker-area-mobile {
    max-width: 98vw;
    flex-direction: row;
    justify-content: center;
    align-content: center;
  }
}
</style>
