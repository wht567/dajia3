<template lang="pug">
a-table(v-if="data && data.length > 0" :columns="columns" :data-source="data" rowKey="q" :scroll={x: scrollX} :pagination="false" size="small")
  template(#name="{text, record}")
    a-input(:value="text" @change="e => handleChange(e.target.value, record, 'name')")
  template(#a="{text, record}")
    a-textarea(:value="text" @change="e => handleChange(e.target.value, record, 'a')" :autoSize="{minRows: 1}")
  template(#time="{text, record}")
    a-spin(:spinning="saving" size="small") 
      span {{text}}
</template>

<script>
import dayjs from 'dayjs'
import { updateTableById } from '../api'
import { message } from 'ant-design-vue'
const columns = [
  {
    title: '问题',
    dataIndex: 'q',
    width: 140
  },
  {
    title: '姓名',
    dataIndex: 'name',
    width: 140,
    slots: { customRender: 'name' }
  },
  {
    title: '优化',
    dataIndex: 'a',
    width: 580,
    slots: { customRender: 'a' }
  },
  {
    title: '时间',
    dataIndex: 'time',
    slots: { customRender: 'time' }
  }
]

export default {
  name: 'hello-form2',
  props: {
    record: Object
  },
  data() {
    const data = this.record.a
      ? JSON.parse(this.record.a)
      : this.unSatisfiesProblems(this.record).map(v => ({
          q: v,
          name: this.record.person,
          a: '',
          operation: 'save',
          time: ''
        }))
    return {
      data,
      columns,
      saving: false,
      debounce_save: null
    }
  },
  computed: {
    scrollX() {
      return this.columns.reduce((sum, { width }) => sum + width, 50)
    }
  },
  methods: {
    handleChange(value, record, column) {
      const newData = [...this.data]
      const target = newData.filter(item => record.q === item.q)[0]
      if (target) {
        target[column] = value
        this.data = newData
        this.debounce_save(record)
      }
    },
    save(record) {
      const newData = [...this.data]
      const target = newData.filter(item => record.q === item.q)[0]
      if (target) {
        this.saving = true
        target['time'] = dayjs().format('YYYY/MM/DD HH:mm:ss')
        let a = JSON.stringify(this.data)
        updateTableById(this.record.id, a)
          .then(res => {
            message.success(res)
            this.data = newData
            this.saving = false
            // this.$emit('save', this.record.id)
          })
          .catch(err => {
            message.error(err)
            this.saving = false
          })
        console.log(a)
      }
    },
    toNum(str) {
      try {
        return parseFloat(str)
      } catch (error) {
        return 0
      }
    },
    isIncome(text, record) {
      text = this.toNum(text)
      if (record.platform == '美团') return text < 1500
      else if (record.platform == '饿了么') return text < 1000
      return false
    },
    isIncomeAvg(text) {
      text = this.toNum(text)
      return text < 1500
    },
    isConsumeRatio(text) {
      text = this.toNum(text)
      return text > 5
    },
    isCostRatio(text) {
      text = this.toNum(text)
      return text > 50
    },
    isSettlea30(text) {
      text = this.toNum(text)
      return text < 70
    },
    unSatisfies(record) {
      const { income, income_avg, consume_ratio, cost_ratio, settlea_30, platform } = record
      let list = []
      if (this.isIncome(income, record))
        list.push({
          title: '收入',
          value: income,
          threshold: platform == '美团' ? '1500' : '1000',
          problem: '低收入'
        })
      if (this.isIncomeAvg(income_avg))
        list.push({
          title: '平均收入',
          value: income_avg,
          threshold: '1500',
          problem: '低收入'
        })
      if (this.isConsumeRatio(consume_ratio))
        list.push({
          title: '推广比例',
          value: consume_ratio,
          threshold: '5%',
          problem: '高推广'
        })
      if (this.isCostRatio(cost_ratio))
        list.push({
          title: '成本比例',
          value: cost_ratio,
          threshold: '50%',
          problem: '高成本'
        })
      if (this.isSettlea30(settlea_30))
        list.push({
          title: '比30日',
          value: settlea_30,
          threshold: '70%',
          problem: '严重超跌'
        })
      return list
    },
    unSatisfiesProblems(record) {
      return Array.from(new Set(this.unSatisfies(record).map(v => v.problem)))
    },
    debounce(fn) {
      let timeout = null
      return function() {
        clearTimeout(timeout)
        timeout = setTimeout(() => fn.apply(this, arguments), 600)
      }
    }
  },
  created() {
    this.debounce_save = this.debounce(this.save)
  }
}
</script>
<style scoped>
.editable-row-operations a {
  margin-right: 8px;
}
</style>
