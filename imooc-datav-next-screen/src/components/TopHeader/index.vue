<template>
  <div class="top-header">
<!--    <img src="https://www.youbaobao.xyz/resource/imooc.png" class="logo" />-->
    <div class="log-wrapper">
      <imooc-logo stroke="#d81e06" stroke-width="50"></imooc-logo>
    </div>
    <div class="logo-text">
      <div class="cn-text">慕课外卖业务数据大盘</div>
      <div class="en-text">Imooc Delivery Overview of Business Data</div>
    </div>
    <div class="right-text">
      <div class="date">{{ date }}</div>
      <div class="time">{{ time }}</div>
    </div>
  </div>
</template>

<script>
import { useClock } from '@/hooks/useClock.js'
export default {
  name: 'TopHeader',
  setup () {
    const { time, date } = useClock()
    return {
      time,
      date
    }
  }
}
</script>

<style lang="scss" scoped>
.top-header {
  display: flex;
  align-items: center;
  width: 100%;
  height: 100%;
  background: rgb(36, 31, 32);
  padding: 0 64px;
  box-sizing: border-box;

  .logo {
    width: 130px;
    height: 130px;
  }

  .log-wrapper {
    width: 160px;
    height: 160px;
  }

  .logo-text {
    flex: 1;
    margin-left: 40px;

    .cn-text {
      color: #fff;
      font-weight: 700;
      font-size: 60px;
      letter-spacing: 2px;
    }

    .en-text {
      color: #fff;
      font-size: 35px;
      letter-spacing: 1px;
    }
  }

  .right-text {
    display: flex;
    align-items: center;
    justify-content: flex-start;

    .date {
      width: 330px;
      text-align: right;
      font-size: 52px;
      color: #fff;
      font-family: DIN;
      font-weight: 500;
    }

    .time {
      width: 220px;
      font-size: 52px;
      color: #c5fb79;
      font-family: DIN;
      font-weight: 500;
      margin-left: 30px;
    }
  }
}
</style>
