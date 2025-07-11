<script setup>
import { usePlayerStore } from '../stores/player'
import { ref } from 'vue'
import { useDialog, useMessage } from 'naive-ui'
import { saveAs } from 'file-saver'

const clickCount = ref(0);
const newName = ref('')
const message = useMessage()
const maxLength = 6  // 定义道号最大长度常量
const playerStore = usePlayerStore()
const dialog = useDialog()
const version = __APP_VERSION__

// 导出存档
const handleExportSave = async () => {
  try {
    const saveData = await playerStore.exportData()
    if (!saveData) {
      message.error('没有可导出的存档数据！')
      return
    }
    // 导出加密后的存档数据
    saveAs(
      new Blob([saveData], { type: 'application/json;charset=utf-8' }),
      `从零开始修仙${version}版本存档数据-${new Date().toISOString().slice(0, 10)}-${Date.now()}.json`
    );
    message.success('存档导出成功！')
  } catch (error) {
    message.error('导出失败：' + error.message)
  }
}

// 导入存档
const handleImportSave = (data) => {
  const reader = new FileReader();
  reader.onload = async (e) => {
    try {
      const encryptedData = e.target.result
      await playerStore.importData(encryptedData)
      message.success('存档导入成功！')
    } catch (error) {
      message.error('导入失败：' + error.message)
    }
  }
  reader.readAsText(data.file.file)
}

// 转世重修确认
const handleReincarnation = () => {
  clickCount.value++
  if (clickCount.value >= 10) {
    dialog.warning({
      title: '提示',
      content: 'GM模式已开启！',
      positiveText: '确定',
      negativeText: '取消',
      onPositiveClick: () => {
        playerStore.isGMMode = true
        playerStore.saveData()
      }
    })
    return;
  }
  dialog.warning({
    title: '转世重修确认',
    content: '确定要转世重修吗？这将清空所有数据重新开始！',
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: async () => {
      // 二次确认
      dialog.warning({
        title: '最终确认',
        content: '这是最后的确认，转世后将无法恢复！确定继续吗？',
        positiveText: '确定转世',
        negativeText: '再想想',
        onPositiveClick: async () => {
          await playerStore.clearData()
          location.href = location.origin
        }
      })
    }
  })
}

const qq = ref(false)

// 修改道号
const handleChangeName = () => {
  if (!newName.value.trim()) {
    message.warning('道号不能为空！')
    return
  }
  if (newName.value.trim().length > maxLength) {
    message.warning(`道号长度不能超过${maxLength}个字符！`)
    return
  }
  // 计算修改道号所需灵石
  const spiritStoneCost = playerStore.nameChangeCount === 0 ? 0 : Math.pow(2, playerStore.nameChangeCount) * 100
  // 第一次修改免费，之后需要消耗灵石
  if (playerStore.nameChangeCount > 0) {
    if (playerStore.spiritStones < spiritStoneCost) {
      message.error(`灵石不足！修改道号需要${spiritStoneCost}颗灵石`)
      return
    }
    playerStore.spiritStones -= spiritStoneCost
  }
  playerStore.name = newName.value.trim()
  playerStore.nameChangeCount++
  playerStore.saveData()
  message.success(playerStore.nameChangeCount === 1 ?
    '道号修改成功！首次修改免费' :
    `道号修改成功！消耗${spiritStoneCost}颗灵石`)
  newName.value = ''
}
</script>

<template>
  <div class="settings-container">
    <n-card title="游戏设置">
      <template #header-extra>
        游戏版本{{ version }}
      </template>
      <n-space vertical>
        <n-input-group>
          <n-input v-model:value="newName" placeholder="输入新的道号" clearable :maxlength="maxLength" show-count />
          <n-button type="primary" @click="handleChangeName" :disabled="!newName">
            修改道号
          </n-button>
        </n-input-group>
        <n-alert title="必看说明" type="warning">
          本游戏为开源项目，如果您在任何地方通过付费方式购买了本游戏，请及时退款并投诉举报。
        </n-alert>
        <n-space>
          <n-button type="warning" @click="handleReincarnation">
            转世重修
          </n-button>
          <n-button @click="handleExportSave" type="info">
            导出存档
          </n-button>
          <n-upload :show-file-list="false" @change="handleImportSave">
            <n-button>导入存档</n-button>
          </n-upload>
          <n-button target="_blank" href="https://github.com/setube/vue-idle-xiuxian" tag="a" type="primary">
            开源地址
          </n-button>
          <n-button type="error" @click="qq = true">
            官方群聊
          </n-button>
        </n-space>
      </n-space>
    </n-card>
    <n-modal preset="dialog" title="玩家交流群" v-model:show="qq">
      <n-card :bordered="false" size="huge" role="dialog" aria-modal="true">
        <n-space vertical>
          <n-input value="343781647" readonly type="text" />
        </n-space>
      </n-card>
    </n-modal>
  </div>
</template>

<style scoped>
</style>