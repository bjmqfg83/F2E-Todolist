<template lang="pug">
  .addtodo.container.px-0.mt-4
    .py-3.mt-2.add
      i.fas.fa-plus.ml-4(@click="showedit($event)")
      span.d-inline-block.ml-3 Add Task
    .edittodo.container.mt-4.border(:class="{bgmarked:isstared}")
      .todoheader.row
        .col-12.mx-auto
          input(type="checkbox" id="editcomplete").border.ml-2.mt-4
          label(for="editcomplete")
          .d-inline-block.font-weight-bolder.ml-3
            input.title.mb-1.pl-2.pr-3(type="text" :placeholder="titlemsg" v-if="!typeinput" v-model='todoitem' @keydown.enter='edititle($event)' required)
            p.title.mb-1(v-if="typeinput") {{todoitem}}
          i.far.fa-edit.edit.float-right.mr-3.my-4(@click="checkedit($event)")
          i(v-if="!isstared" @click="togglefocus").star.far.fa-star.float-right.mr-3.my-4
          i(v-else @click="togglefocus").star.fas.fa-star.float-right.mr-3.my-4
      .todobody
        .row
          .deadline.col-12
            .title.ml-5.mb-1
              i.far.fa-calendar-alt.mr-2
              span Deadline
            .setting.ml-5
              input(type="date").ml-4.py-1.pl-2.border#startdate
              input(type="date").ml-2.py-1.pl-2.border#enddate
        .row
          .upload.col-12.mt-2
            .title.ml-5
              i.far.fa-file.mr-2
              span File
            .setting.pl-5.my-1
              label.my-auto(for="uploadfile")
                input(type="file" id="uploadfile" style="display: none" @change="getfileName")
                i.fas.fa-plus-square.ml-4
              span.ml-3 {{fileName}}
        .row
          .comment.col-12
            .title.ml-5
              i.far.fa-comment-dots.mr-1
              span Comment
            .setting.pl-4.mb-3
              textarea(name="comment", cols="52", rows="5", placeholder="Type Something Here...").ml-5.border.pl-3.pt-2.mt-1
      .todofooter
        .row
          .cancel.col-6.d-flex.justify-content-center.text-danger.py-2(@click="canceledit")
            i.fas.fa-times.my-auto.mr-2
            span.my-auto Cancel
          .save.col-6.d-flex.justify-content-center.bg-primary(@click="addtodo")
            i.my-auto.fas.fa-plus.text-light.mr-2
            span.my-auto.text-light Save
</template>

<script>
import $ from 'jquery'
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
Vue.use(VueAxios, axios)
export default {
  name: 'edittodo',
  data () {
    return {
      isstared: false,
      typeinput: false,
      todoitem: '',
      fileName: '',
      titlemsg: 'Type Something Here...',
      todomsg: {}
    }
  },
  methods: {
    /**
     * 關注星星
     */
    togglefocus () {
      let vm = this
      vm.isstared = !vm.isstared
    },
    /**
     * 點擊add Task時出現編輯視窗
     */
    showedit (e) {
      $(e.target).parent().next('.edittodo').css('display', 'block')
    },
    /*
      1.如果typeinput為true(表示編輯過內容),修改其class
      2.todoitem清空,將代辦事項標題清除
      3.isstared為true時改回來
      4.勾選框取消勾選
      5.清空檔案
      6.fileName清除
      7.textarea清除
      8.input date日期清除
      9.隱藏edittodo
    */
    canceledit () {
      if (this.typeinput) {
        this.typeinput = !this.typeinput
      }
      if (this.isstared) {
        this.isstared = !this.isstared
      }
      this.todoitem = ''
      this.fileName = ''
      $('.edittodo').css('display', 'none')
      $('#editcomplete').prop('checked', false)
      $('.comment .setting textarea').val('')
      $('#startdate').val('')
      $('#enddate').val('')
    },
    /*
      events up 子組件參數發送給父組件
      將參數發送給父組件app,之後的參數再由app處理
      todoTitle => 代辦事項標題
      fileName => 上傳檔案名稱
      comment => 代辦事項相關事項描述
      isstared => 是否點擊星星
      startDate、endDate =>開始、結束日期
      checked => 代辦事項是否完成
    */
    addtodo () {
      if (this.todoitem) {
        this.todomsg.todoTitle = this.todoitem
      }
      if (this.isstared) {
        this.todomsg.isstared = true
      } else {
        this.todomsg.isstared = false
      }
      this.todomsg.fileName = this.fileName
      this.todomsg.comment = $('.comment .setting textarea').val()
      this.todomsg.startDate = $('#startdate').val()
      this.todomsg.endDate = $('#enddate').val()
      this.todomsg.checked = false
      // 如果資料都有輸入則判斷是否有勾選checked
      if (this.todomsg.todoTitle !== '' && this.todomsg.startDate !== '' && this.todomsg.endDate !== '' && this.todomsg.fileName !== '' && this.todomsg.comment !== '') {
        // 如果有勾選將勾選狀態改為true
        if ($('#editcomplete').prop('checked')) {
          this.todomsg.checked = true
          console.log(this.todomsg)
        }
        // 判斷日期設定是否有誤如果有不給送出資料
        // 先判斷年份是否大於0
        if (parseInt(this.todomsg.endDate.substring(0, 4)) - parseInt(this.todomsg.startDate.substring(0, 4)) > 0) {
          this.$emit('passtoApp', this.todomsg)
          Vue.axios.post('http://localhost:6500/todos', this.todomsg).then(res => {
            console.log('success', res)
            this.canceledit()
          }).catch(err => {
            console.log('error', err)
          })
          // 如果等於則進行月份判斷
        } else if (parseInt(this.todomsg.endDate.substring(0, 4)) - parseInt(this.todomsg.startDate.substring(0, 4)) === 0) {
          if (parseInt(this.todomsg.endDate.substring(5, 7)) - parseInt(this.todomsg.startDate.substring(5, 7)) > 0) {
            this.$emit('passtoApp', this.todomsg)
            Vue.axios.post('http://localhost:6500/todos', this.todomsg).then(res => {
              console.log('success', res)
              this.canceledit()
            }).catch(err => {
              console.log('error', err)
            })
            // 如果月份相等則進行日期判斷
          } else if (parseInt(this.todomsg.endDate.substring(5, 7)) - parseInt(this.todomsg.startDate.substring(5, 7)) === 0) {
            if (parseInt(this.todomsg.endDate.substring(8, 10)) - parseInt(this.todomsg.startDate.substring(8, 10)) >= 0) {
              this.$emit('passtoApp', this.todomsg)
              Vue.axios.post('http://localhost:6500/todos', this.todomsg).then(res => {
                console.log('success', res)
                this.canceledit()
              }).catch(err => {
                console.log('error', err)
              })
            } else {
              alert('結束時間低於開始時間,請輸入正確的日期')
            }
          } else {
            alert('結束時間低於開始時間,請輸入正確的日期')
          }
        } else if (parseInt(this.todomsg.endDate.substring(0, 4)) - parseInt(this.todomsg.startDate.substring(0, 4)) < 0) {
          alert('結束時間低於開始時間,請輸入正確的日期')
        }
      } else {
        alert('輸入欄位有空,請檢查!')
      }
    },
    /**
     * input框 enter後觸發
     */
    edititle (e) {
      if ($(e.target).val() !== '') {
        this.typeinput = !this.typeinput
      } else {
        this.titlemsg = 'Title can not be empty'
        $('.todoheader input.title').css('border-color', 'red')
        $('.todoheader input.title').addClass('holderColor')
      }
    },
    /**
     * 編輯icon點擊後觸發執行
     */
    checkedit (e) {
      if ($(e.target)[0].nodeName.toLowerCase() === 'i') {
        if ($('.todoheader input.title').val() !== '') {
          this.typeinput = !this.typeinput
        }
      }
    },
    /*
    取得上傳檔案的名稱
    */
    getfileName () {
      this.fileName = $('#uploadfile')[0].files[0].name
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="sass">
  $dark-yellow: #F5A623
  $color-checked: #4A90E2
  $color-marked: #FFF2DC
  $color-hover: #3982D7
  *
    box-sizing: border-box
    // border: 1px solid black
  .addtodo
    max-width: 620px
    i.fas.fa-plus
      font-size: 24px
      cursor: pointer
      color: #c8c8c8
      + span
        font-size :28px
        color: #c8c8c8
    .add
      border: 2px solid #c8c8c8
      border-radius: 5px 5px 0px 0px
    i.star,i.edit
      cursor: pointer
    i.star.fas.fa-star
      color: $dark-yellow
    .edittodo
      background-color: #f2f2f2
      border-radius: 5px
      display: none
      .todoheader .d-inline-block.font-weight-bolder
        position: relative
        bottom: 3px
      &.bgmarked
        background-color: $color-marked
      input.title
        border: none
        border: 1px solid #ccc
        background-color: #fff
        border-radius: 5px
        outline: none
      input[type="checkbox"]
        -webkit-appearance: none
        -ms-appearance: none
        appearance: none
        outline: none
        width: 24px
        height: 24px
        background-color: #fff
        border-radius: 2px
        cursor: pointer
        position: relative
        &:checked
          background-color: $color-checked
          + label
            display: inline-block
            width: 14px
            height: 14px
            background: url("../assets/img/check.png") no-repeat
            background-size: contain
            position: absolute
            left: 28px
            top: 29px
            cursor: pointer
          ~ div > p.title
            text-decoration: line-through
            color: #757575
      .title
        font-size: 24px
      .tododetail
        font-size: 16px
        color: #757575
        cursor: pointer
.todobody
  .title
    i
      font-size: 17px
    span
      font-size: 20px
      font-weight: bolder
  .setting
    > *
      outline: none
      resize: none
    input,textarea
      border: none
      border-radius: 5px
      font-size: 14px
    i.fas.fa-plus-square
      color: #bcbcbc
      font-size: 32px
      cursor: pointer
      &:hover
        color: $color_hover
  .deadline
  .upload
    .setting span
      display: inline-block
      max-width: 110px
      position: relative
      bottom: 5px
  .comment
.todofooter
  .cancel
    background-color: #fff
  .cancel,.save
    cursor: pointer
    i
      font-size: 24px
    span
      font-size: 24px
// placeholder顏色更改
.holderColor::placeholder
  color: red
.holderColor:-ms-input-placeholder
  color: red
.holderColor::-ms-input-placeholder
  color: red
</style>
