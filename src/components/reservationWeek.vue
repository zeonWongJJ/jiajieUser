<template>
  <div class="periodicReservation">
    <van-popup class="body" v-model="show" position="bottom" :close-on-click-overlay="false">
      <div class="center">
        <div class="setLong" v-if="once == 2 || once == -1">
          <div class="label">服务时长</div>
          <div class="btn">
            <div class="left" @click="setLong(0)">-</div>
            <div class="middle">{{defaultLong}}小时</div>
            <div class="right" @click="setLong(1)">+</div>
          </div>
        </div>
        <div class="time">
          <div class="label">
            <div class="li" :id="'li'+item.type" :class="{checked : defaultWeek == item.type}" v-for="item in week" @click="selectWeek(item.type)">{{item.name}}</div>
          </div>
          <div class="arr">
            <div class="li"
                 :class="[{checked : checked && periodicData[defaultWeek] && periodicData[defaultWeek].order.indexOf(item.start_at) >= 0},
							{nochecked : getGray(index) || !item.can_order}]"
                 v-for="(item,index) in arr" @click="selectTime(item,index)">
              <div>{{item.display_text}}</div>
              <div v-if="item.can_order_desc">{{item.can_order_desc}}</div>
              <div v-else-if="item.charge">{{item.charge}}元</div>
            </div>
            <!--为了最后一行能左对齐-->
            <div class="li" style="height: 0px;visibility: hidden;padding: 0;"></div>
            <div class="li" style="height: 0px;visibility: hidden;padding: 0;"></div>
            <!--为了最后一行能左对齐-->
          </div>
        </div>
        <div class="startTime">
          <div class="left">开始时间</div>
          <div>{{startTime ? formatTime(startTime, 1) : '请选择'}}</div>
        </div>
        <div class="startTime" @click="pickerShow">
          <div class="left">结束时间</div>
          <div class="right">{{endTime ? formatTime(endTime, 1) : '请选择'}}</div>
        </div>
        <div class="startTime">
          <div>预计服务次数</div>
          <div>{{total}}</div>
        </div>
      </div>
      <van-button  size="large" class="blue" @click="finish">确认选择</van-button>
    </van-popup>
    <van-popup v-model="pickerValue" position="bottom" style="width: 100%;">
      <van-datetime-picker
        v-model="currentDate"
        type="date"
        :min-date="minDate"
        @confirm="pickerConfirm"
        @cancel="pickerValue=false"
      />
    </van-popup>
  </div>
</template>

<script>
  export default{
    name: 'reservationWeek',
    props: {
      value: {
        type: Boolean,
        default: false
      },
      service_id: {
        type: [Number, String],
        default: 0
      },
      once: {
        type: [Number, String],
        default: () => -1
      }
    },
    data() {
      return{
        defaultLong: 2,
        show: false,
        arr: [],
        week: [
          // {
          //   type: '1',
          //   name: '周一'
          // },
          // {
          //   type: '2',
          //   name: '周二'
          // },
          // {
          //   type: '3',
          //   name: '周三'
          // },
          // {
          //   type: '4',
          //   name: '周四'
          // },
          // {
          //   type: '5',
          //   name: '周五'
          // },
          // {
          //   type: '6',
          //   name: '周六'
          // },
          // {
          //   type: '0',
          //   name: '周日'
          // },
        ],
        defaultWeek: '1',
        periodicData: {},
        checked: 0,//无意义，用来刷新dom
        allOrder: {},
        startTime: '',
        selectStart: false,
        endTime: '',
        currentDate: new Date(),
        pickerValue: false,
        allStartTime: [],
        reqOrder: {},
        total: ''
      }
    },
    watch: {
      value (val) {
        this.show = val
        if (val) {
          console.log(this.$el.querySelector('#liabc'))
          this.$el.querySelector('#li' + this.defaultWeek).scrollIntoView();
        }
      },
      defaultLong (val) {
        this.setArr()
      },
      endTime (val) {
        if (val) {
          this.getOrder()
        }
      }

    },
    computed: {
      minDate() {
        return this.startTime ? new Date(this.startTime + 24 * 3600 * 1000) : new Date(new Date().valueOf() + 24 * 3600 * 1000)
      }
    },
    created(){
      for (var i = 0; i < 7; i++) {
        let newDate = moment().add(i, 'd')
        let obj = {}
        obj.type = newDate.day()
        obj.name = moment.weekdaysShort(newDate.day())
        // obj.date = newDate.months() + 1 + '-' + newDate.date()
        this.week.push(obj)
      }
      this.selectWeek(new Date().getDay())
    },
    methods:{
      pickerShow() {
        if (this.startTime) {
          this.currentDate = this.endTime ? new Date(this.endTime.valueOf() + 24 * 3600 * 1000) : this.minDate
          this.pickerValue = true
        } else {
          this.$dialog.alert({
            message: '请先选择服务时间',
          });
        }
      },
      pickerConfirm() {
        this.endTime = this.currentDate
        this.pickerValue=false
      },
      getGray(index) {
        let gray = false
        if(this.periodicData[this.defaultWeek]){
          let arr = this.periodicData[this.defaultWeek].order
          let defaultAt = this.defaultLong * 3600
          let indexAt = this.arr[index].start_at
          arr.forEach(item =>{
            if(indexAt > item && indexAt < item + defaultAt || indexAt < item && indexAt > item - defaultAt){
              gray = true
            }
          })
        }
        return gray
      },
      setArr() {
        let req = {}
        req.service_id = this.service_id
        req.service_week = this.defaultWeek
        req.service_length = this.defaultLong
        this.$fetch('service_get_ordercycle',req).then(rs => {
          this.arr = rs
        })
      },
      getH(str) {
        let newDate = new Date(str *1000)
        if (newDate) {
          let h = newDate.getHours()
          let m = newDate.getMinutes()
          return this.add0(h) + ':' + this.add0(m)
        } else {
          return ''
        }
      },
      getDay(time) {
        let data = new Date(time)
        if (data) {
          let day = data.getDay()
          let cnDay = ''
          switch (day){
            case 0 :
              cnDay = '周日'
              break
            case 1 :
              cnDay = '周一'
              break
            case 2 :
              cnDay = '周二'
              break
            case 3 :
              cnDay = '周三'
              break
            case 4 :
              cnDay = '周四'
              break
            case 5:
              cnDay = '周五'
              break
            case 6 :
              cnDay = '周六'
              break
          }
          return cnDay
        } else {
          console.log('时间格式有误：' + time)
          return ''
        }
      },
      //转换时间
      formatTime(time, noHM) {
        let data = new Date(time)
        if (data) {
          let year = data.getFullYear()
          let month = this.add0(data.getMonth() + 1)
          let day = this.add0(data.getDate())
          let hour = this.add0(data.getHours())
          let minute = this.add0(data.getMinutes())
          return noHM ? year + '-' + month + '-' + day + ' (' + this.getDay(time) + ')' : year + '-' + month + '-' + day + ' ' + hour + ':' + minute
        } else {
          console.log('时间格式有误：' + time)
          return ''
        }
      },
      add0(time) {
        var time = Number(time)
        if (time < 10) {
          time = '0' + time
        }
        return time
      },
      setLong(type){//设置服务时长
        if (this.periodicData[this.defaultWeek]) {
          this.$dialog.confirm({
            message: '已选择' + this.defaultLong + '服务时长，是否重新选择'
          }).then(() => {
            this.checked = 1;
            delete this.periodicData[this.defaultWeek]
          }).catch(() => {
            // on cancel
          })
        } else {
          if (type) {
            this.defaultLong += 0.5
          } else {
            this.defaultLong -= 0.5
          }
          if (this.defaultLong < 2) {this.defaultLong = 2}
          if (this.defaultLong > 6) {this.defaultLong = 6}
          this.setArr()
        }
      },
      // close() {//关闭组件
      //   this.$emit('input',false)
      // },
      selectWeek(type) {
        this.defaultWeek = type
        if (this.periodicData[type]) {
          this.defaultLong = this.periodicData[type].long
        }
        this.setArr()
      },
      selectTime(time,index){//添加服务时间
        if (time.can_order && !this.getGray(index)) {
          if (this.periodicData[this.defaultWeek] && this.defaultLong == this.periodicData[this.defaultWeek].long) {
            // let had = false
            // let arr = this.periodicData[this.defaultWeek].order
            // arr.forEach(item =>{
            //   if (item == time.start_at) {
            //     had = true
            //   }
            // })
            // if (had) {//以选择，删除
            //   let index = arr.indexOf(time.start_at)
            //   arr.splice(index, 1)
            //   if (arr.length == 0) {
            //     delete this.periodicData[this.defaultWeek]
            //   }
            // } else {//添加
            //   arr.push(time.start_at);
            // }
            this.periodicData[this.defaultWeek].order.indexOf(time.start_at) > -1 ? delete this.periodicData[this.defaultWeek] : this.periodicData[this.defaultWeek].order = [time.start_at]
          } else {//啥都没，直接添加
            this.periodicData[this.defaultWeek] = {}
            this.periodicData[this.defaultWeek].long = this.defaultLong
            this.periodicData[this.defaultWeek].order = [time.start_at]
          }
          //每次添加服务时间都添加首次服务时间
          this.addStartTime()
          this.checked++ //没啥意义，为了刷新dom
        }
      },
      addStartTime() {
        this.allStartTime = []
        Object.keys(this.periodicData).forEach(key => {
          this.allStartTime.push(this.periodicData[key].order[0] * 1000)
        })
        this.allStartTime.length > 0 ? this.startTime = Math.min.apply(Math, this.allStartTime) : this.startTime = ''
        this.getOrder()
      },
      getOrder() {
        if (this.endTime) {
          this.reqOrder = {}
          let startDay = new Date(this.startTime).getDay()
          let daykeys = Object.keys(this.periodicData)
          let index = daykeys.indexOf(startDay)
          let newDaykeys = daykeys.slice(index, daykeys.long).concat(daykeys.slice(0, index))
          this.reqOrderPush(newDaykeys, 0)
          this.allOrder.startTime = this.formatTime(this.startTime)
          this.allOrder.endTime = this.formatTime(this.endTime)
        }
      },
      finish(){
        if (this.startTime && !this.endTime){
          this.$dialog.alert({
            message: '请选择服务结束时间',
          });
        }else{
          this.$emit('input',false)
          this.$emit('finish',this.allOrder)
        }
      },
      reqOrderPush(newDaykeys, i) {
        let end = false
        let allOrder = []
        let reqOrder = this.reqOrder
        let _this = this
        newDaykeys.some(key => {
          let order = _this.periodicData[key].order[0] * 1000 + i * 7 * 24 * 3600 * 1000
          if (order > _this.endTime.valueOf()) {
            end = true
            return
          }
          let time = _this.formatTime(order)
          allOrder.push({time:time,long:this.periodicData[key].long})
          if (!reqOrder[key]) {
            reqOrder[key] = {}
            reqOrder[key].order = []
          } else if (!reqOrder[key].order) {
            reqOrder[key].order = []
          }
          reqOrder[key].order.push(time)
          reqOrder[key].long = _this.periodicData[key].long
        })
        this.allOrder.order = reqOrder
        let total = 0
        Object.keys(reqOrder).forEach(key => {
          total = total + reqOrder[key].order.length
        })
        this.total = total
        !end ? this.reqOrderPush(newDaykeys, i + 1) : 1
      }
    }
  }
  function sortNumber(a,b){
    return a - b
  }
</script>

<style lang="less"  scoped>
  @import "./periodcReservation.less";
  .right {
    display: flex;
    align-items: center;
  }
</style>
