<template lang="pug">
  #app.container.row.mx-auto
    .bg-primary.col-12
      .container.todo-nav.d-flex.justify-content-around.col-12
        a(href="#" @click.prevent="showAll($event)").active My Tasks
        a(href="#" @click.prevent="showProgress($event)") In Progress
        a(href="#" @click.prevent="showComplete($event)") Completed
    //- emits up
    EditTodo(@passtoApp = "passtolist")
    //- props down
    Todolist.mt-3(:todo-group="todoObj")
</template>

<script>
import Todolist from './components/Todolist.vue'
import EditTodo from './components/EditTodo.vue'
import $ from 'jquery'
export default {
  name: 'app',
  components: {
    Todolist,
    EditTodo
  },
  data () {
    return {
      todoObj: []
    }
  },
  methods: {
    showComplete (e) {
      $(e.target).addClass('active').siblings().removeClass('active')
      $('.inprogress').css('display', 'none')
      $('.finish').css('display', 'block')
    },
    showProgress (e) {
      $(e.target).addClass('active').siblings().removeClass('active')
      $('.inprogress').css('display', 'block')
      $('.finish').css('display', 'none')
    },
    showAll (e) {
      $(e.target).addClass('active').siblings().removeClass('active')
      $('.inprogress').css('display', 'block')
      $('.finish').css('display', 'block')
    },
    passtolist (data) {
      console.log('data', data)
      this.todoObj.push({
        todoTitle: data.todoTitle,
        isstared: data.isstared,
        fileName: data.fileName,
        comment: data.comment,
        startDate: data.startDate,
        endDate: data.endDate,
        checked: false
      })
      console.log('answer', this.todoObj)
    }
  }
}
</script>
<style lang="sass">
$dark-primary: #00408b
*
  // border: 1px solid black
  font-family: Roboto-Medium,"微軟正黑體"
  box-sizing: border-box
#app
  max-width: 768px
.todo-nav
  a
    font-size: 1.5rem
    display: block
    padding-top: 24px
    padding-bottom: 24px
    cursor: pointer
    color: #fff
    text-decoration: none
    &:active,&:hover,&.active
      text-decoration: none
      border-bottom: 5px solid $dark-primary
</style>
