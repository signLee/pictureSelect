<!-- home -->
<template>
  <div>
    <scroll-lock :lock="isScrollSelect" :bodyLock="true">
      <div class="btn" @click="selectAll">
        <button>全选</button>
      </div>
      <div class="list-container">
        <div class="item" :class="{ selected: result.includes(item) }" v-for="(item, index) in list" :key="index"
          @touchstart="gtouchstart(item, index, $event)" @touchend="gtouchend(item, index, $event)"
          @touchmove="fnMove($event)">
          <div class="item-container" :data-index="index" :data-item="item" @click.stop="selectItem(item)">{{ item }}
          </div>
        </div>
      </div>
    </scroll-lock>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: [], //列表数据
      touchesPageX: 0, //手指移动的初始位置
      touchesPageY: 0, //手指移动的初始位置
      startSelectIndex: null, //滑动选择起始项位置
      curSelectResult: [], //记录滑动选择之前已选中的数据
      result: [], //当前滑选+滑选后的数据
      isScrollSelect: false, //是否在执行滑选的动作
      startY: 0 //获取手指初始坐标
    }
  },

  computed: {},
  mounted() {
    for (let i = 0; i < 600; i++) {
      this.list.push(i)
    }
    this.fnMove = this.throttle(this.goTouchMove, 30) //节流
  },

  methods: {
    throttle(fn, interval = 500, option) {
      let last = 0
      let timer = null
      if (!option) option = {}

      let trailing = option.trailing || false
      let result = option.result || null
      let handleFn = function () {
        // this和argument
        let _this = this
        let _arguments = arguments
        let now = new Date().getTime()
        if (now - last > interval) {
          if (timer) {
            clearTimeout(timer)
            timer = null
          }
          callFn(_this, _arguments)

          last = now
        } else if (timer == null && trailing) {
          // 只是最后一次

          timer = setTimeout(function () {
            timer = null
            callFn(_this, _arguments)
          }, interval)
        }
      }

      handleFn.cancel = function () {
        clearTimeout(timer)
        timer = null
      }

      function callFn(context, argument) {
        let res = fn.apply(context, argument)
        if (!result) {
          return res
        }
      }
      return handleFn
    },
    selectAll() {
      this.result = this.result.length === this.list.length ? [] : this.list.map(item => item)
    },
    // 单个选择&取消选择
    selectItem(item) {
      if (this.result.includes(item)) {
        let findIndex = this.result.findIndex(resultItem => resultItem === item)
        this.result.splice(findIndex, 1)
      } else {
        this.result.push(item)
      }
    },
    // 记录手指起点信息
    gtouchstart(item, index, event) {
      this.touchesPageX = event.touches[0].pageX
      this.touchesPageY = event.touches[0].pageY
      this.startSelectIndex = index
      this.startSelectItem = item
      this.curSelectResult = this.result.slice(0)
      this.isStarSelect = this.curSelectResult.includes(item) //第一个落点位置的选择状态
    },
    // 页面移动距离大于5时不出现长按效果
    goTouchMove(event) {
      let touchesPageX = event.touches[0].pageX
      let touchesPageY = event.touches[0].pageY
      let XDis = Math.abs(touchesPageX - this.touchesPageX)
      let YDis = Math.abs(touchesPageY - this.touchesPageY)
      if (XDis > 5 || YDis > 5) {
        let flag = false //当前是否执行滑选动作
        if (this.isScrollSelect) {
          flag = true
        } else {
          if (YDis === 0) {
            //单个X轴滑动选择,Y轴未移动
            flag = XDis >= 5
          } else {
            flag = XDis / YDis >= 1
          }
        }
        if (!flag) {
          return false
        }
        this.mutiSelect(touchesPageX, touchesPageY)
      }
    },
    // 多选勾选逻辑
    mutiSelect(touchesPageX, touchesPageY) {
      this.isScrollSelect = true
      // 滑动选择
      let curTarget = document.elementFromPoint(touchesPageX - window.pageXOffset, touchesPageY - window.pageYOffset) //获取当前坐标点位置底部层级最高的元素
      let curItem = curTarget && curTarget.getAttribute('data-item') //自定义属性，根据具体业务配置，需要唯一
      if (curItem) {
        let curIndex = curTarget.getAttribute('data-index') - 0
        let isDownSelect = curIndex - this.startSelectIndex >= 0 // 是否是向下滑选
        if (curIndex === this.startSelectIndex) {
          // 滑选过程中回到起点
          if (!this.isStarSelect) {
            //X轴有移动的情况才算滑选选中当前
            this.result.push(curItem)
          } else {
            this.result = [...this.curSelectResult] // 滑选回到起点
          }
        }
        //滑选需要勾选
        let start = isDownSelect ? this.startSelectIndex : curIndex
        let end = isDownSelect ? curIndex + 1 : this.startSelectIndex
        let selectArr = this.list.slice(start, end)
        // 走勾选逻辑
        if (!this.isStarSelect) {
          this.result = this.curSelectResult.concat(selectArr.map(selectItem => selectItem))
          this.result.push(this.startSelectItem)
        } else {
          // 走取消勾选逻辑
          selectArr.forEach(item => {
            let findIndex = this.result.findIndex(resultItem => resultItem === item)
            if (findIndex !== -1) {
              this.result.splice(findIndex, 1)
            }
          })
          this.result = this.result.filter(item => item !== this.startSelectItem)
        }
        this.result = [...new Set(this.result)]
      }
    },
    gtouchend() {
      this.isScrollSelect = false
      return false
    }
  }
}
</script>
<style lang="scss" scoped>
.list-container {
  background: #fff;
  padding: 12px;
  display: flex;
  justify-content: flex-start;
  flex-wrap: wrap;
  .item {
    width: 80px;
    height: 80px;
    position: relative;
     touch-action: none;
    .item-container {
      position: absolute;
      width: 80px;
      height: 80px;
      line-height: 80px;
      z-index: 1;
      background: #eee;
      text-align: center;
    }
    &.selected {
      .item-container {
        background: rgb(35, 201, 35);
      }
    }
  }
}
</style>
