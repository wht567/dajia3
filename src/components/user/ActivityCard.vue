<template lang="pug">
a-list-item(:key="activity.time")
  a-list-item-meta
    template(#title)
      p.title {{title}}
      .title-meta
        div
          span.num {{title_num}}
          span.type {{`${activity.q} ${title_value}`}}
          router-link.name(:to="{name: 'user', params: {username: title_name, date: 0}}") {{title_name}}
          span.time {{title_time}}
          span.time {{title_weekday}}
          router-link.time.time-href(:to="{name: 'user', params: {username: title_name, date: time2date}}") {{title_time_}}
        div
          a-button(size="small" type="link" @click="history_click") history
          a-button(size="small" type="link" @click="detail_click") detail
    template(#description)
      shop-form(:as="shop_as" :shop_meta="shop_meta")
      shop-data(v-if="shop_data_show" :shop_data="shop_data")
      shop-history(v-if="shop_history_show" :shopid="shop_meta.shop_id")
</template>

<script>
import dayjs from 'dayjs'
import relativeTime from 'dayjs/plugin/relativeTime'
import duration from 'dayjs/plugin/duration'
import localeData from 'dayjs/plugin/localeData'
import weekday from 'dayjs/plugin/weekday'
import updateLocale from 'dayjs/plugin/updateLocale'

import 'dayjs/locale/zh-cn'

dayjs.extend(duration)
dayjs.extend(relativeTime)
dayjs.extend(localeData)
dayjs.extend(weekday)
dayjs.extend(updateLocale)

dayjs.locale('zh-cn')

dayjs.updateLocale('zh-cn', {
  weekdays: ['周日', '一', '二', '三', '四', '五', '六']
})

import ShopForm from '../shop/ShopForm'
import ShopData from '../shop/ShopData'
import ShopHistory from '../shop/ShopHistory'

export default {
  name: 'activity-card',
  components: {
    ShopForm,
    ShopData,
    ShopHistory
  },
  props: {
    activity: Object
  },
  data() {
    return {
      shop_data_show: false,
      shop_history_show: false
    }
  },
  computed: {
    title() {
      return this.activity.a
    },
    shop_meta() {
      return {
        id: this.activity.id,
        shop_id: this.activity.shop_id,
        shop_name: this.activity.shop_name,
        platform: this.activity.platform
      }
    },
    title_num() {
      return `#${this.activity.index + 1}`
    },
    title_name() {
      return this.activity.name
    },
    title_time() {
      return dayjs().from(this.activity.time_parsed)
    },
    title_time_() {
      return dayjs(this.activity.time_parsed).format('MM/DD HH:mm:ss')
    },
    title_weekday() {
      return dayjs.weekdays()[dayjs(this.activity.time_parsed).day()]
    },
    title_value() {
      return this.activity.qs.find(q=>q.type == this.activity.q).value.toFixed(2)
    },
    shop_data() {
      return {
        third_send: this.activity.third_send,
        orders: this.activity.orders,
        income: this.activity.income,
        income_avg: this.activity.income_avg,
        income_sum: this.activity.income_sum,
        cost: this.activity.cost,
        cost_avg: this.activity.cost_avg,
        cost_sum: this.activity.cost_sum,
        cost_ratio: this.activity.cost_ratio,
        cost_sum_ratio: this.activity.cost_sum_ratio,
        consume: this.activity.consume,
        consume_avg: this.activity.consume_avg,
        consume_sum: this.activity.consume_sum,
        consume_ratio: this.activity.consume_ratio,
        consume_sum_ratio: this.activity.consume_sum_ratio,
        settlea_30: this.activity.settlea_30,
        settlea_1: this.activity.settlea_1,
        settlea_7: this.activity.settlea_7,
        settlea_7_3: this.activity.settlea_7_3,
        income_score: this.activity.income_score,
        consume_score: this.activity.consume_score,
        cost_score: this.activity.cost_score,
        score: this.activity.score,
        date: this.activity.date
      }
    },
    shop_as() {
      return this.activity.as.map(a => ({
        ...a,
        value: this.activity.qs.find(q => q.type == a.q).value,
        threshold: this.activity.qs.find(q => q.type == a.q).threshold
      }))
    },
    time2date() {
      return dayjs()
        .startOf('day')
        .diff(dayjs(this.activity.time_parsed).startOf('day'), 'day')
    }
  },
  methods: {
    detail_click() {
      this.shop_data_show = !this.shop_data_show
    },
    history_click() {
      this.shop_history_show = !this.shop_history_show
    }
  }
}
</script>

<style lang="sass" scoped>
.title
  font-size: 1.1em
  font-weight: bold
  margin: 0
  white-space: pre-wrap

.title-meta
  display: flex
  justify-content: space-between
  align-items: center

.ant-btn-link
  font-size: 9px
  color: rgba(0, 0, 0, 0.38)

.name:hover, .time-href:hover
  color: #40a9ff

.num, .name, .time, .type
  font-size: 10px
  color: rgba(0, 0, 0, 0.38)
  margin: 4px 16px 10px 0

.ant-list-split, .ant-list-item
  border: none

.ant-list-item
  padding: 12px
</style>
