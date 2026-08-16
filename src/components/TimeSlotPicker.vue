<template>
  <div class="time-slot-picker">
    <div class="picker-row">
      <div class="date-section">
        <div class="section-label">{{ label }}</div>
        <div class="calendar-container">
          <div class="calendar-header">
            <button type="button" class="nav-btn" @click="prevMonth">&lt;</button>
            <span class="month-label">{{ monthLabel }}</span>
            <button type="button" class="nav-btn" @click="nextMonth">&gt;</button>
          </div>
          <div class="calendar-weekdays">
            <span v-for="d in weekdays" :key="d">{{ d }}</span>
          </div>
          <div class="calendar-grid">
            <span
              v-for="(day, idx) in calendarDays"
              :key="idx"
              class="calendar-day"
              :class="{
                'empty': !day.date,
                'disabled': !day.selectable,
                'selected': isSelectedDate(day.date),
                'today': isToday(day.date)
              }"
              @click="selectDate(day)"
            >
              {{ day.date || '' }}
            </span>
          </div>
        </div>
      </div>

      <div class="time-section" v-if="selectedDate">
        <div class="section-label">选择时间</div>
        <div class="time-grid" v-if="timePoints.length > 0">
          <button
            v-for="point in timePoints"
            :key="point"
            type="button"
            class="time-btn"
            :class="{ 'selected': modelValue && modelValue.endsWith(point) }"
            @click="selectTime(point)"
          >
            {{ point }}
          </button>
        </div>
        <div class="no-time" v-else>
          <span>暂无可预约时间</span>
        </div>
      </div>

      <div class="time-section placeholder" v-else>
        <div class="section-label">选择时间</div>
        <div class="placeholder-text">请先选择日期</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  label: {
    type: String,
    default: '选择日期与时间'
  }
})

const emit = defineEmits(['update:modelValue'])

const weekdays = ['一', '二', '三', '四', '五', '六', '日']
const today = new Date()
today.setHours(0, 0, 0, 0)

const currentYear = ref(today.getFullYear())
const currentMonth = ref(today.getMonth())
const selectedDate = ref(null)
const timePoints = ref([])
const loading = ref(false)

const monthLabel = computed(() => {
  return `${currentYear.value}年${currentMonth.value + 1}月`
})

const calendarDays = computed(() => {
  const year = currentYear.value
  const month = currentMonth.value
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)

  const days = []
  let startWeekday = firstDay.getDay()
  startWeekday = startWeekday === 0 ? 6 : startWeekday - 1

  for (let i = 0; i < startWeekday; i++) {
    days.push({ date: null, selectable: false })
  }

  for (let d = 1; d <= lastDay.getDate(); d++) {
    const date = new Date(year, month, d)
    const isPast = date < today
    days.push({
      date: d,
      selectable: !isPast,
      fullDate: date
    })
  }

  return days
})

function prevMonth() {
  if (currentMonth.value === 0) {
    currentMonth.value = 11
    currentYear.value--
  } else {
    currentMonth.value--
  }
}

function nextMonth() {
  if (currentMonth.value === 11) {
    currentMonth.value = 0
    currentYear.value++
  } else {
    currentMonth.value++
  }
}

function isToday(d) {
  if (!d) return false
  const date = new Date(currentYear.value, currentMonth.value, d)
  return date.getTime() === today.getTime()
}

function isSelectedDate(d) {
  if (!d || !selectedDate.value) return false
  const date = new Date(currentYear.value, currentMonth.value, d)
  return date.getTime() === selectedDate.value.getTime()
}

function getDayOfWeek(date) {
  let dow = date.getDay()
  return dow === 0 ? 7 : dow
}

async function selectDate(day) {
  if (!day.selectable || !day.date) return

  selectedDate.value = day.fullDate

  loading.value = true
  try {
    const dow = getDayOfWeek(day.fullDate)
    const response = await fetch(`/api/time-slots/points/${dow}`)
    const result = await response.json()
    if (result.success) {
      timePoints.value = result.data || []
    } else {
      timePoints.value = []
    }
  } catch (e) {
    console.error('获取时间点失败', e)
    timePoints.value = []
  } finally {
    loading.value = false
  }
}

function selectTime(point) {
  const year = selectedDate.value.getFullYear()
  const month = String(selectedDate.value.getMonth() + 1).padStart(2, '0')
  const day = String(selectedDate.value.getDate()).padStart(2, '0')
  const dateStr = `${year}-${month}-${day} ${point}`
  emit('update:modelValue', dateStr)
}

watch(() => props.modelValue, (newVal) => {
  if (!newVal && selectedDate.value) {
    selectedDate.value = null
    timePoints.value = []
  }
})
</script>

<style scoped>
.time-slot-picker {
  width: 100%;
}

.picker-row {
  display: flex;
  gap: 20px;
  align-items: flex-start;
}

.date-section {
  flex: 0 0 auto;
}

.time-section {
  flex: 1;
}

.time-section.placeholder {
  display: flex;
  flex-direction: column;
}

.placeholder-text {
  color: #c0c4cc;
  font-size: 14px;
  margin-top: 8px;
}

.section-label {
  font-size: 14px;
  color: #606266;
  margin-bottom: 8px;
  font-weight: 500;
}

.calendar-container {
  background: #fff;
  border: 1px solid #dcdfe6;
  border-radius: 10px;
  padding: 12px;
  width: 280px;
  max-width: 100%;
  box-sizing: border-box;
  box-shadow: 0 2px 8px rgba(31, 60, 105, 0.05);
  transition: box-shadow 0.25s ease, border-color 0.25s ease;
}

.calendar-container:hover {
  box-shadow: 0 4px 14px rgba(31, 60, 105, 0.09);
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.month-label {
  font-size: 15px;
  font-weight: 600;
  color: #303133;
}

.nav-btn {
  background: none;
  border: 1px solid #dcdfe6;
  border-radius: 6px;
  padding: 4px 10px;
  cursor: pointer;
  color: #606266;
  font-size: 14px;
  transition: all 0.2s cubic-bezier(0.25, 0.8, 0.35, 1);
}

.nav-btn:hover {
  background: #ecf5ff;
  color: #409eff;
  border-color: #409eff;
  transform: scale(1.06);
}

.nav-btn:active {
  transform: scale(0.95);
}

.calendar-weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  text-align: center;
  margin-bottom: 8px;
}

.calendar-weekdays span {
  font-size: 12px;
  color: #909399;
  font-weight: 500;
  line-height: 28px;
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 2px;
}

.calendar-day {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  color: #303133;
  height: 32px;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.25, 0.8, 0.35, 1);
  user-select: none;
}

.calendar-day:not(.empty):not(.disabled):hover {
  background: #ecf5ff;
  color: #409eff;
  transform: scale(1.12);
}

.calendar-day:not(.empty):not(.disabled):active {
  transform: scale(0.95);
}

.calendar-day.disabled {
  color: #c0c4cc;
  cursor: not-allowed;
}

.calendar-day.selected {
  background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%) !important;
  color: #fff !important;
  box-shadow: 0 3px 8px rgba(64, 158, 255, 0.4);
}

.calendar-day.today:not(.selected) {
  border: 1px solid #409eff;
  color: #409eff;
}

.time-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 6px;
  max-height: 240px;
  overflow-y: auto;
  padding: 4px;
}

.time-btn {
  padding: 8px 4px;
  font-size: 13px;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  background: #fff;
  color: #606266;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.25, 0.8, 0.35, 1);
  text-align: center;
}

.time-btn:hover {
  background: #ecf5ff;
  color: #409eff;
  border-color: #409eff;
  transform: translateY(-1px);
  box-shadow: 0 2px 6px rgba(64, 158, 255, 0.15);
}

.time-btn.selected {
  background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%);
  color: #fff;
  border-color: transparent;
  box-shadow: 0 3px 8px rgba(64, 158, 255, 0.35);
}

.no-time {
  padding: 20px;
  text-align: center;
  color: #909399;
  font-size: 14px;
  border: 1px dashed #dcdfe6;
  border-radius: 8px;
}

@media screen and (max-width: 1600px) {
  .picker-row {
    flex-direction: column;
    gap: 16px;
  }

  .time-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
</style>
