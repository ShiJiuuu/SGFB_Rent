<template>
  <div class="time-slot-management">
    <div class="page-header">
      <el-icon class="header-icon"><Clock /></el-icon>
      <span>预约时间管理</span>
    </div>

    <el-card>
      <div class="tip-text">
        配置每周各天的可预约时间范围，系统将自动按5分钟间隔生成可选时间点。
      </div>

      <div class="slot-grid">
        <div
          v-for="day in slots"
          :key="day.dayOfWeek"
          class="slot-card"
        >
          <div class="slot-header">
            <span class="day-label">{{ day.name }}</span>
          </div>

          <div class="period-list">
            <div
              v-for="period in day.periods"
              :key="period.periodIndex"
              class="period-row"
              :class="{ 'disabled-row': !period.enabled }"
            >
              <div class="period-header">
                <el-switch
                  v-model="period.enabled"
                  size="small"
                />
                <span class="period-label">{{ periodLabels[period.periodIndex] }}</span>
              </div>

              <div class="slot-body" v-if="period.enabled">
                <div class="time-range">
                  <div class="time-input-group">
                    <span class="time-label">开始</span>
                    <el-time-select
                      v-model="period.startStr"
                      placeholder="开始时间"
                      start="00:00"
                      step="00:05"
                      end="23:55"
                      style="width: 120px"
                      size="small"
                    />
                  </div>
                  <div class="time-input-group">
                    <span class="time-label">结束</span>
                    <el-time-select
                      v-model="period.endStr"
                      placeholder="结束时间"
                      start="00:00"
                      step="00:05"
                      end="23:55"
                      style="width: 120px"
                      size="small"
                    />
                  </div>
                </div>
              </div>

              <div class="period-footer">
                <el-button
                  type="primary"
                  size="small"
                  @click="handleSave(day.dayOfWeek, period.periodIndex, period)"
                  :loading="period.saving"
                >
                  保存
                </el-button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { Clock } from '@element-plus/icons-vue'

const slots = ref([])

const dayNames = ['', '周一', '周二', '周三', '周四', '周五', '周六', '周日']
const periodLabels = ['上午', '下午']

onMounted(() => {
  fetchSlots()
})

async function fetchSlots() {
  try {
    const response = await fetch('/api/time-slots')
    const result = await response.json()
    if (result.success && result.data) {
      const grouped = {}
      for (let i = 1; i <= 7; i++) grouped[i] = []

      result.data.forEach(s => {
        if (grouped[s.dayOfWeek]) grouped[s.dayOfWeek].push(s)
      })

      slots.value = dayNames.slice(1).map((name, idx) => {
        const dayOfWeek = idx + 1
        const periods = [0, 1].map(pi => {
          const found = grouped[dayOfWeek].find(s => s.periodIndex === pi)
          return {
            id: found ? found.id : null,
            dayOfWeek,
            periodIndex: pi,
            enabled: found ? (found.enabled === 1) : false,
            startStr: found && found.timeRangeStart ? found.timeRangeStart.substring(0, 5) : '',
            endStr: found && found.timeRangeEnd ? found.timeRangeEnd.substring(0, 5) : '',
            saving: false
          }
        })
        return { dayOfWeek, name, periods }
      })
    }
  } catch (e) {
    console.error('获取时间配置失败', e)
    ElMessage.error('获取时间配置失败')
  }
}

async function handleSave(dayOfWeek, periodIndex, period) {
  if (period.enabled && (!period.startStr || !period.endStr)) {
    ElMessage.warning('请填写完整的开始和结束时间')
    return
  }

  period.saving = true
  try {
    const response = await fetch('/api/time-slots', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        id: period.id,
        dayOfWeek: dayOfWeek,
        periodIndex,
        timeRangeStart: period.startStr ? period.startStr + ':00' : null,
        timeRangeEnd: period.endStr ? period.endStr + ':00' : null,
        enabled: period.enabled ? 1 : 0
      })
    })
    const result = await response.json()
    if (result.success) {
      ElMessage.success(`${dayNames[dayOfWeek]} ${periodLabels[periodIndex]} 保存成功`)
      fetchSlots()
    } else {
      ElMessage.error(result.message || '保存失败')
    }
  } catch (e) {
    console.error('保存失败', e)
    ElMessage.error('保存失败')
  } finally {
    period.saving = false
  }
}
</script>

<style scoped>
.time-slot-management {
  width: 100%;
  min-height: 100%;
  background-color: #fff;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(31, 60, 105, 0.06);
  animation: fadeInUp 0.4s cubic-bezier(0.25, 0.8, 0.35, 1);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(16px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

:deep(.el-card) {
  border-radius: 10px;
  border: none;
  box-shadow: 0 2px 8px rgba(31, 60, 105, 0.05);
}

:deep(.el-card__body) {
  padding: 20px;
}

.page-header {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 22px;
  font-weight: 600;
  color: #303133;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid #ebeef5;
}

.header-icon {
  font-size: 26px;
  color: #409eff;
}

.tip-text {
  color: #909399;
  font-size: 14px;
  margin-bottom: 20px;
  padding: 10px 14px;
  background: linear-gradient(135deg, #f4f7fb, #f4f4f5);
  border-radius: 8px;
  border-left: 3px solid #409eff;
}

.slot-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 16px;
}

.slot-card {
  border: 1px solid #ebeef5;
  border-radius: 10px;
  padding: 16px;
  transition: all 0.25s cubic-bezier(0.25, 0.8, 0.35, 1);
  background: #fff;
  animation: fadeInUp 0.4s cubic-bezier(0.25, 0.8, 0.35, 1) backwards;
}

.slot-card:nth-child(1) { animation-delay: 0.03s; }
.slot-card:nth-child(2) { animation-delay: 0.06s; }
.slot-card:nth-child(3) { animation-delay: 0.09s; }
.slot-card:nth-child(4) { animation-delay: 0.12s; }
.slot-card:nth-child(5) { animation-delay: 0.15s; }
.slot-card:nth-child(6) { animation-delay: 0.18s; }
.slot-card:nth-child(7) { animation-delay: 0.21s; }

.slot-card:hover {
  box-shadow: 0 6px 20px rgba(31, 60, 105, 0.1);
  transform: translateY(-3px);
  border-color: #c6e2ff;
}

.period-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.period-row {
  border: 1px solid #f0f0f0;
  border-radius: 8px;
  padding: 10px;
  transition: all 0.2s;
}

.period-row:hover {
  background: #f7fafd;
  border-color: #e1ecf7;
}

.period-row.disabled-row {
  opacity: 0.55;
}

.period-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.period-label {
  font-size: 13px;
  color: #606266;
  font-weight: 500;
}

.slot-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 14px;
}

.day-label {
  font-size: 16px;
  font-weight: 600;
  color: #303133;
  position: relative;
  padding-left: 10px;
}

.day-label::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 3px;
  height: 14px;
  border-radius: 999px;
  background: linear-gradient(180deg, #66b1ff, #409eff);
}

.slot-body {
  margin-bottom: 14px;
}

.time-range {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.time-input-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.time-label {
  font-size: 13px;
  color: #606266;
  width: 36px;
}

.period-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 8px;
}

@media screen and (max-width: 768px) {
  .time-slot-management {
    padding: 20px 15px;
  }

  .page-header {
    font-size: 18px;
    margin-bottom: 20px;
    padding-bottom: 15px;
  }

  .slot-grid {
    grid-template-columns: 1fr;
  }
}
</style>
