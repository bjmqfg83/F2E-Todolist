<template lang="pug">
  .todoitem.container(v-if="todos")
    draggable(@end="changePos" v-model="todos")
      .row.todolist.my-3(v-for="(todo,index) in todos" :class="{bgmarked:todo.isstared,inprogress:!todo.checked,finish:todo.checked}")
        .col-12.mx-auto.my-2
          label
            input(type="checkbox" @click="todofinish($event)" :data-check="todo.id" :checked="Boolean(todo.checked)").mb-4.border.ml-2
            i.fas.fa-check
          .d-inline-block.font-weight-bolder.ml-3.my-2.pt-2(style="max-width: 465px")
            p.title.mb-1(:class="{finishline:todo.checked}") {{todo.todoTitle}}
            .tododetail
              i.far.fa-calendar-alt.mr-2
              span.time.mr-3 {{todo.startDate}} - {{todo.endDate}}
              i.far.fa-file.mr-1
              span.file.mr-3 {{todo.fileName}}
              i.far.fa-comment-dots.mr-1
              span.comment {{todo.comment}}
          i(@click="focus($event)" :data-stared="todo.id" :data-index="index"  v-if="!todo.isstared").star.far.fa-star.float-right
          i(@click="blur($event)" :data-stared="todo.id"  :data-index="index" v-if="todo.isstared").star.fas.fa-star.float-right
          i.far.fa-edit.edit.float-right.mr-3.my-4(@click="adjusttodo($event, todo)")
        .edittodo.container.border
          .todoheader.row
            .col-12.mx-auto
              input(type="checkbox" id="editcomplete").border.ml-2.mt-4
              label(for="editcomplete")
              .d-inline-block.font-weight-bolder.ml-3
                input.title.mb-1.pl-2.pr-3(type="text"  v-model="todo.todoTitle"  required)
              i.far.fa-edit.edit.float-right.mr-3.my-4
              i.star.far.fa-star.float-right.mr-3(@click="focus($event)" :data-stared="todo.id" :data-index="index"  v-if="!todo.isstared")
              i.star.fas.fa-star.float-right.mr-3(@click="blur($event)" :data-stared="todo.id"  :data-index="index" v-if="todo.isstared")
          .todobody
            .row
              .deadline.col-12
                .title.ml-5.mb-1
                  i.far.fa-calendar-alt.mr-2
                  span Deadline
                .setting.ml-5
                  input(type="date" v-model="todo.startDate").ml-4.py-1.pl-2.border#startdate
                  input(type="date" v-model="todo.endDate").ml-2.py-1.pl-2.border#enddate
            .row
              .upload.col-12.mt-2
                .title.ml-5
                  i.far.fa-file.mr-2
                  span File
                .setting.pl-5.my-1
                  label.my-auto(for="editupload")
                    input(type="file" id="editupload" style="display: none"  @change="getfileName(index)")
                    i.fas.fa-plus-square.ml-4
                  span.ml-3 {{todo.fileName}}
            .row
              .comment.col-12
                .title.ml-5
                  i.far.fa-comment-dots.mr-1
                  span Comment
                .setting.pl-4.mb-3
                  textarea(name="comment", cols="52", rows="5", v-model="todo.comment" placeholder="Type Something Here...").ml-5.border.pl-3.pt-2.mt-1
          .todofooter
            .row
              .cancel.col-6.d-flex.justify-content-center.text-danger.py-3.bg-light(@click="canceledit($event, index)")
                i.fas.fa-times.my-auto.mr-2
                span.my-auto Cancel
              .save.col-6.d-flex.justify-content-center.bg-primary(@click="finishedit($event,todo)")
                i.my-auto.fas.fa-plus.text-light.mr-2
                span.my-auto.text-light Save
    p {{taskleft}} tasks left
</template>
<script>
import $ from 'jquery'
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
import draggable from 'vuedraggable'
Vue.use(VueAxios, axios)
export default {
  name: 'todolist',
  components: {
    draggable
  },
  props: ['todo-group'],
  data () {
    return {
      requestime: 1,
      taskleft: 0,
      todos: [],
      todotemp: []
    }
  },
  updated () {
    // 此時才取得到class
    this.taskleft = $('.inprogress').length
  },
  created () {
    this.todoinit()
  },
  methods: {
    todoinit () {
      if (this.requestime === 1) {
        Vue.axios.get('http://localhost:6500/todos').then(res => {
          console.log(res)
          for (var todo of res.data) {
            this.todoGroup.push(todo)
          }
          this.todos = this.todoGroup
          console.log('todos', this.todoGroup)
          this.requestime += 1
        }).catch(err => {
          console.log(err)
        })
      }
    },
    focus (e) {
      // 點擊星星的狀態
      let starId = $(e.target).data('stared')
      // 取得isstared所屬的todolist
      let index = $(e.target).data('index')
      this.todos[index].isstared = true
      $($(e.target)[0].parentNode.parentNode).toggleClass('bgmarked')
      Vue.axios.patch(`http://localhost:6500/todos/${starId}`, { 'isstared': true }).then(res => {
        console.log(res)
      }).catch(err => {
        console.log(err)
      })
    },
    blur (e) {
      // 點擊星星的狀態
      let starId = $(e.target).data('stared')
      // 取得isstared所屬的todolist
      let index = $(e.target).data('index')
      this.todos[index].isstared = false
      $($(e.target)[0].parentNode.parentNode).toggleClass('bgmarked')
      Vue.axios.patch(`http://localhost:6500/todos/${starId}`, { 'isstared': false }).then(res => {
        console.log(res)
      }).catch(err => {
        console.log(err)
      })
    },
    todofinish (e) {
      // finishId要先改點擊狀態的id
      let finishId = $(e.target).data('check')
      if ($(e.target).prop('checked') === true) {
        $($(e.target)[0].parentNode.parentNode.parentNode).addClass('finish')
        $($(e.target)[0].parentNode.parentNode.parentNode).removeClass('inprogress')
        $($(e.target)[0].parentNode).next().find('p.title').addClass('finishline')
        this.taskleft = $('.inprogress').length
        Vue.axios.patch(`http://localhost:6500/todos/${finishId}`, { 'checked': true }).then(res => {
          console.log(res)
        }).catch(err => {
          console.log(err)
        })
      } else {
        $($(e.target)[0].parentNode.parentNode.parentNode).addClass('inprogress')
        $($(e.target)[0].parentNode.parentNode.parentNode).removeClass('finish')
        $($(e.target)[0].parentNode).next().find('p.title').removeClass('finishline')
        this.taskleft = $('.inprogress').length
        Vue.axios.patch(`http://localhost:6500/todos/${finishId}`, { 'checked': false }).then(res => {
          console.log(res)
        }).catch(err => {
          console.log(err)
        })
      }
    },
    adjusttodo (e, todo) {
      // 獲取代辦事項資料
      this.todotemp.push({
        todoTitle: todo.todoTitle,
        isstared: todo.isstared,
        fileName: todo.fileName,
        comment: todo.comment,
        startDate: todo.startDate,
        endDate: todo.endDate,
        checked: false
      })
      // 顯示編輯清單
      $(e.target).parent().next().css('display', 'block')
      // 隱藏原先todoitem
      $(e.target).parent().css('display', 'none')
      console.log('todotemp', this.todotemp)
    },
    // finish編輯清單
    finishedit (e, todo) {
      // 點擊save時將資料透過Vue.axios更新json-server內的資料
      Vue.axios.patch(`http://localhost:6500/todos/${todo.id}`, todo).then(res => {
        console.log(res)
      }).catch(err => {
        console.log(err)
      })
      $('.todolist >div:first-of-type').css('display', 'block')
      $('.edittodo').css('display', 'none')
    },
    // 取消待辦清單編輯
    canceledit (e, index) {
      // 將資料還原成未修改的狀態
      this.todos[index].checked = this.todotemp[0].checked
      this.todos[index].comment = this.todotemp[0].comment
      this.todos[index].endDate = this.todotemp[0].endDate
      this.todos[index].fileName = this.todotemp[0].fileName
      this.todos[index].isstared = this.todotemp[0].isstared
      this.todos[index].startDate = this.todotemp[0].startDate
      this.todos[index].todoTitle = this.todotemp[0].todoTitle
      $('.todolist >div:first-of-type').css('display', 'block')
      $('.edittodo').css('display', 'none')
    },
    // getfilename
    getfileName (index) {
      this.todos[index].fileName = $('#editupload')[0].files[0].name
    },
    changePos () {
      console.log(this.todos)
      let todolen = new Array(this.todos)[0].length
      for (let id = 1; id <= todolen; id++) {
        Vue.axios.patch(`http://localhost:6500/todos/${id}`, this.todos[id - 1]).then(res => {
          console.log(res)
        }).catch(err => {
          console.log(err)
        })
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="sass">
  $dark-yellow: #F5A623
  $color-checked: #4A90E2
  $color-marked: #FFF2DC
  *
    box-sizing: border-box
    // border: 1px solid black
  .edittodo
    display: none
  .todofooter
    cursor: pointer
  .finishline
    text-decoration: line-through
    color: #757575
  i.fas.fa-plus-square
    font-size: 28px
    color: #bcbcbc
    cursor: pointer
    &:hover
      color: #3982D7
  i.star,i.edit
    cursor: pointer
  i.star.fas, i.star.far
    position: absolute
    right: 70px
    top: 24px
  i.star.fas.fa-star
    color: $dark-yellow
  .todoitem
    max-width: 620px
    max-height: 102px
    .todolist
      background-color: #f2f2f2
      border-radius: 5px
      cursor: pointer
      &.bgmarked
        background-color: $color-marked
    label
      position: relative
      cursor: pointer
      i.fas.fa-check
        position: absolute
        color: #fff
        width: 14px
        height: 14px
        z-index: 10
        left: 12px
        top: 4px
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
    .title
      font-size: 24px
    .tododetail
      font-size: 16px
      color: #757575
      cursor: pointer
      max-width: 470px
      white-space: nowrap
      overflow: hidden
      text-overflow: ellipsis
</style>
