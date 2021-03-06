<template>
 <div :id="id" class="base-scroll-list">
   <div
     class="base-scroll-list-header"
     :style="{
        backgroundColor: actualConfig.headerBg,
        height: `${actualConfig.headerHeight}px`,
        fontSize: `${actualConfig.headerFontSize}px`,
        color: actualConfig.headerColor
     }"
   >
     <div
       class="header-item base-scroll-list-text"
       v-for="(headerItem, index) in headerData"
       :key="headerItem + index"
       :style="{
         width: `${columnWidths[index]}px`,
         ...headerStyle[index]
       }"
       v-html="headerItem"
       :align="aligns[index]"
     >
     </div>
   </div>
   <div
     class="base-scroll-list-rows-wrapper"
     :style="{
       height: `${height - actualConfig.headerHeight}px`
     }"
   >
     <div
       class="base-scroll-list-rows base-scroll-list-text"
       v-for="(rowData, index) in currentRowsData"
       :key="rowData.rowIndex"
       :style="{
         height: `${rowHeights[index]}px`,
         lineHeight: `${rowHeights[index]}px`,
         backgroundColor: rowData.rowIndex % 2 === 0 ? rowBg[1] : rowBg[0],
         fontSize: `${actualConfig.rowFontSize}px`,
         color: actualConfig.rowColor
       }"
     >
       <div
         class="base-scroll-list-columns"
         v-for="(colData, colIndex) in rowData.data"
         :key="colData + colIndex"
         :style="{
           width: `${columnWidths[colIndex]}px`,
           ...rowStyle[colIndex]
         }"
         v-html="colData"
         :align="aligns[colIndex]"
       >
       </div>
     </div>
   </div>
 </div>
</template>

<script>
import { ref, watch } from 'vue'
import { v4 as uuidv4 } from 'uuid'
import { useScreen } from '../../hooks/useScreen'
import cloneDeep from 'lodash/cloneDeep'
import assign from 'lodash/assign'

const defaultConfig = {
  // 标题数据，格式：['a','b','c']
  headerData: [],
  // 标题样式，格式：[{},{},{}]
  headerStyle: [],
  // 行样式，格式：[{},{},{}]
  rowStyle: [],
  // 行背景色
  rowBg: [],
  // 标题背景颜色
  headerBg: 'rgb(90, 90, 90)',
  // 标题背景颜色
  headerHeight: 32,
  // 标题是否展示序号
  headerIndex: false,
  // 标题序号展示内容
  headerIndexContent: '#',
  // 标题序号展示样式
  headerIndexStyle: {
    width: '50px'
  },
  // 序号列的内容
  headerIndexData: [],
  // 行序号内容展示样式
  rowIndexStyle: {
    width: '50px'
  },
  // 表格数据项，二维数组
  data: [],
  // 要显示行数(每页数据量)
  rowNum: 0,
  // 居中方式
  aligns: [],
  // 标题字体大小
  headerFontSize: 28,
  // 行字体大小
  rowFontSize: 28,
  // 标题字体颜色
  headerColor: '#fff',
  // 行字体颜色
  rowColor: '#000',
  // 移动的位置
  moveNum: 1,
  // 动画间隔
  duration: 2000
}

export default {
  name: 'BaseScrollList',
  props: {
    config: {
      type: Object,
      default: () => ({})
    }
  },
  setup (props) {
    const id = `base-scroll-list-${uuidv4()}`
    const { width, height } = useScreen(id)
    const actualConfig = ref({})

    const headerData = ref([])
    const headerStyle = ref([])
    const rowStyle = ref([])
    const rowBg = ref([])
    const columnWidths = ref([])
    const rowHeights = ref([])
    const rowsData = ref([])
    const currentRowsData = ref([]) // 真正渲染出现的数据
    const currentIndex = ref(0) // 动画指针
    const rowNum = ref(defaultConfig.rowNum)
    const aligns = ref([])
    const isAnimationStop = ref(false)
    let avgHeight // 行高

    const handleHeader = (config) => {
      const _headerData = cloneDeep(config.headerData)
      const _headerStyle = cloneDeep(config.headerStyle)
      const _rowStyle = cloneDeep(config.rowStyle)
      const _rowsData = cloneDeep(config.data)
      const _aligns = cloneDeep(config.aligns)

      if (_headerData.length === 0) {
        return false
      }
      if (config.headerIndex) {
        _headerData.unshift(config.headerIndexContent)
        _headerStyle.unshift(config.headerIndexStyle)
        _rowStyle.unshift(config.rowIndexStyle)
        _rowsData.forEach((rows, index) => {
          // 处理序号列的数据
          if(config.headerIndexData && config.headerIndexData.length > 0 && config.headerIndexData[index]) {
            rows.unshift(config.headerIndexData[index])
          } else {
            rows.unshift(index + 1)
          }
        })
        _aligns.unshift('center')
      }

      // 动态计算header 中每一列的宽度
      let usedWidth = 0
      let usedColumnNum = 0
      // 判断是否自定义width
      _headerStyle.forEach((style, index) => {
        // 如果自定义width，则按照自定义width进行渲染
        if (style.width) {
          usedWidth += +style.width.replace('px', '')
          usedColumnNum++
        }
      })

      // 动态计算列宽时，使用剩余的宽度除以剩余的列数
      const headerLen = _headerData.length
      const avgWidth = (width.value - usedWidth) / (headerLen - usedColumnNum)
      const _columnWidths = new Array(headerLen).fill(avgWidth)
      _headerStyle.forEach((style, index) => {
        // 如果自定义width，则按照自定义width进行渲染
        if (style.width) {
          const headerWidth = +style.width.replace('px', '')
          _columnWidths[index] = headerWidth
        }
      })

      columnWidths.value = _columnWidths
      headerData.value = _headerData
      headerStyle.value = _headerStyle
      rowStyle.value = _rowStyle
      const { rowNum } = config
      if(_rowsData.length >= rowNum && _rowsData.length < rowNum * 2) {
        const newRowData = [..._rowsData, ..._rowsData]
        rowsData.value = newRowData.map((item, index) => ({
          data: item,
          rowIndex: index
        }))
      } else {
        rowsData.value = _rowsData.map((item, index) => ({
          data: item,
          rowIndex: index
        }))
      }
      aligns.value = _aligns
    }

    const handleRows = (config) => {
      // console.log(config)
      const { headerHeight } = config
      rowNum.value = config.rowNum
      const unusedHeight = height.value - headerHeight

      // 如果rowNum大于实际数据长度，则以实际数据长度为准
      if (rowNum.value > rowsData.value.length) {
        rowNum.value = rowsData.value.length
      }
      avgHeight = unusedHeight / rowNum.value
      rowHeights.value = new Array(rowNum.value).fill(avgHeight)

      // 获取行背景色
      if (config.rowBg) {
        rowBg.value = config.rowBg
      }
    }

    const startAnimation = async () => {
      const config = actualConfig.value
      const { rowNum, moveNum, duration } = config
      const totalLength = rowsData.value.length
      if (totalLength < rowNum) return false

      const index = currentIndex.value
      const _rowsData = cloneDeep(rowsData.value)
      // 将数据重新头尾连接
      // [a, b, c, d, e, f]
      //     1
      // [b, c, d, e, f, a]
      const rows = _rowsData.slice(index)
      rows.push(..._rowsData.slice(0, index))
      currentRowsData.value = rows

      // 先将所有行的高度还原
      rowHeights.value = new Array(totalLength).fill(avgHeight)
      const waitTime = 300
      if (isAnimationStop.value) {
        return
      }
      await new Promise(resolve => setTimeout(resolve, waitTime))

      // 将moveNum的行高度设置0
      rowHeights.value.splice(0, moveNum, ...new Array(moveNum).fill(0))

      currentIndex.value += moveNum
      // 是否到达最后一组数据
      const isLast = currentIndex.value - totalLength
      if(isLast >= 0) {
        currentIndex.value = isLast
      }

      if (isAnimationStop.value) {
        return
      }
      await new Promise(resolve => setTimeout(resolve, duration - waitTime))
      if (isAnimationStop.value) {
        return
      }
      await startAnimation()
    }

    const stopAnimation = () => {
      isAnimationStop.value = true
    }

    const update = () => {
      stopAnimation()
      const _actualConfig = assign(defaultConfig, props.config)
      // 赋值 rowsData
      rowsData.value = _actualConfig.rowsData || []
      handleHeader(_actualConfig)
      handleRows(_actualConfig)
      actualConfig.value = _actualConfig
      // 展示动画
      isAnimationStop.value = false
      startAnimation()
    }

    watch(() => props.config, () => {
      update()
    })

    return {
      id,
      actualConfig,
      height,
      headerData,
      headerStyle,
      rowStyle,
      rowBg,
      aligns,
      columnWidths,
      rowHeights,
      rowsData,
      currentRowsData
    }
  }
}
</script>

<style lang="scss" scoped>
.base-scroll-list {
  width: 100%;
  height: 100%;

  .base-scroll-list-text {
    //padding: 0 10px;
    box-sizing: border-box;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }
  .base-scroll-list-header {
    display: flex;
    align-items: center;
    font-size: 15px;

    .header-item {}
  }

  .base-scroll-list-rows-wrapper {
    overflow: hidden;

    .base-scroll-list-rows {
      display: flex;
      align-items: center;
      transition: all 0.3s linear;

      .base-scroll-list-columns {
        height: 100%;
      }
    }
  }
}
</style>
