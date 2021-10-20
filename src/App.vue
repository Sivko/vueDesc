<template>
  <div id="app" :class="rulesMove?'drop':''">
    <header>
      <div class="row">
        <h1>Документы</h1>
        <div class="headerBtn">
          <div class="headerBtn-bookmark"><i class="fa fa-bookmark-o" aria-hidden="true"></i></div>
          <div class="headerBtn-newType" @click.prevent="received.categories.push({title: 'Lorem', id: Math.random(), show: true})"><i class="fa">+</i>Новый тип</div>
          <div class="headerBtn-newDocument" @click.prevent="received.other.push({title: 'Other Date', id: Math.random()})"><i class="fa">+</i>Новый документ</div>
        </div>
      </div>
      <div class="row">
        <div class="searchForm">
          <input type="text" placeholder="Поиск..." v-model="search">
          <span id="clear" v-show="search.length" @click="search = ''"><i class="fa fa-times" aria-hidden="true"></i></span>
          <button type="submit" id="search"></button>
        </div>
      </div>
    </header>
<!--    <button >click</button>-->
<!--    <button v-on:click="shuffle">Перемешать</button>-->
    <transition-group name="flip-list" tag="div"
                      v-bind:css="false"
                      class="mainBlock"
                      v-on:before-enter="beforeEnter"
                      v-on:enter="enter"
                      v-on:leave="leave">
      <div v-for="(category, indexCategory ) in computedCategory" v-bind:key="category.id">
        <div v-bind:draggable="rulesMove" @dragstart='startDrag($event, "category", category.id)' @dragend='endDrag($event, "category", category.id)' @dragenter='enterDrag($event,"category", category.id)' @dragleave='leaveDrag($event,"category", category.id)' >
          <div class='drop-zone' @drop.prevent='onDrop($event, "category",category.id)' @dragover.prevent @dragenter.prevent>
            <div class="item category">
            <span class="item-toggle" @click="received.categories[indexCategory].show = !received.categories[indexCategory].show">
              <i :class="category.show ? 'rotate fa fa-angle-down':'fa fa-angle-down'" aria-hidden="true"></i>
            </span>
              <div class="item-title">{{ category.title }}</div>
              <div class="item-circles"><span class="item-circle"></span><span class="item-circle"></span><span class="item-circle"></span></div>
              <div class="item-alert">Обязательный</div>
              <div class="item-description">{{category.description}}</div>
              <div class="item-actions">
                <span><i class="fa fa-pencil" aria-hidden="true"></i></span>
                <span @click="received.categories.splice(indexCategory, 1)"><i class="fa fa-trash-o" aria-hidden="true"></i></span>
                <span @mousedown="rulesMove = true" @mouseup="rulesMove = false"><i class="fa fa-arrows-v" aria-hidden="true"></i></span>
              </div>
            </div>
          </div>
        </div>
        <transition-group name="list-complete" tag="div" v-show="received.categories[indexCategory].show">
          <div class="list-complete-item" v-for="(item,indexItem) in category.items" v-bind:key="item.id" v-bind:draggable="rulesMove" @dragstart='startDrag($event,"document", item.id, category.id)' @dragend='endDrag($event,"document", item.id)' @dragenter='enterDrag($event, "document", item.id)' @dragleave='leaveDrag($event, "document", item.id)'>
            <div class='drop-zone' @drop.prevent='onDrop($event, "document", item.id, category.id)' @dragover.prevent @dragenter.prevent>
              <div class="item document ">
                <div class="item-title">{{ item.title }}</div>
                <div class="item-description"></div>
                <div class="item-actions">
                  <span><i class="fa fa-pencil" aria-hidden="true"></i></span>
                  <span @click="received.categories[indexCategory].items.splice(indexItem, 1)"><i class="fa fa-trash-o" aria-hidden="true"></i></span>
                  <span @mousedown="rulesMove = true" @mouseup="rulesMove = false"><i class="fa fa-arrows-v" aria-hidden="true"></i></span>
                </div>
              </div>
            </div>
          </div>
        </transition-group>
      </div>
      <div v-bind:key="Math.random()" v-if="!computedOther.length">
        <div class='drop-zone otherContent' @drop.prevent='onDrop($event, "other")' @dragover.prevent @dragenter.prevent>
          <div class="item other" style="border: none; height: auto">
            <h2 style="padding: 10px 0">Нет данных без категории</h2>
          </div>
        </div>
      </div>
      <div v-else v-for="(item,index) in computedOther" :class="index===0?'otherContent':''" v-bind:key="item.id" v-bind:draggable="rulesMove" @dragstart='startDrag($event,"other", item.id)' @dragend='endDrag($event,"other", item.id)' @dragenter='enterDrag($event, "other", item.id)' @dragleave='leaveDrag($event, "other", item.id)'>
        <div class='drop-zone' @drop.prevent='onDrop($event, "other",item.id)' @dragover.prevent @dragenter.prevent>
          <div class="item other">
            <div class="item-title">{{item.title}}</div>
            <div class="item-actions">
              <span><i class="fa fa-pencil" aria-hidden="true"></i></span>
              <span @click="received.other.splice(index, 1)"><i class="fa fa-trash-o" aria-hidden="true"></i></span>
              <span @mousedown="rulesMove = true" @mouseup="rulesMove = false"><i class="fa fa-arrows-v" aria-hidden="true"></i></span>
            </div>
          </div>
        </div>
      </div>
    </transition-group>
    <div class="modal-container" id="modal-opened" style="display: none">
      <div class="modal">
        <div class="modal__details">
          <h1 class="modal__title">Редактирование</h1>
          <p class="modal__description"></p>
        </div>
        <p class="modal__text">
          <input type="text" placeholder="title">
        </p>
        <button class="modal__btn">Сохранить &rarr;</button>
        <a href="#modal-closed" class="link-2" ></a>
      </div>
    </div>
  </div>
</template>

<script>
let _ = require('lodash')
let Velocity = require('velocity-animate')
export default {
  name: 'App',
  data: () => {
    return {
      search: '',
      moveBtn: false, /*What element we move*/
      moveItem: '', /*What item we move*/
      rulesMove: true, /*Rules form move items*/
      tempDropElement: '', /*Why Dom element active hover to Drop*/
      received: {
        categories: [{
          id: Math.random(),
          title: 'Обязательные для всех!!',
          show: true,
          description: 'Документы, обязательные для всех сотрудниковьез исключения, Документы, обязательные для всех сотрудниковьез исключения',
          items: [{
            id: Math.random(),
            title: 'Паспорт',
            description: 'Для всех',
            requirement: true,
          },{
            id: Math.random(),
            title: 'ИНН',
            description: 'Для всех',
            requirement: true
          },
          ]
        }],
        other : [
          {title: 'Test task for condric', description: 'Россия, Белоруссия и т.д.', id: Math.random()},
          {title: 'Test task', description: 'Россия, Белоруссия и т.д.', id: Math.random()},
          {title: 'Test', description: 'Россия', id: Math.random()}
        ]
      },
    }
  },
  created: function() {
    let awesomeIcons = document.createElement('link')
    awesomeIcons.rel = "stylesheet"
    awesomeIcons.href = "https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css"
    document.head.appendChild(awesomeIcons)
    let font = document.createElement('link')
    font.rel = "stylesheet"
    font.href = "https://fonts.googleapis.com/css2?family=Fira+Sans:wght@300&display=swap"
    document.head.appendChild(font)
    let googleApi = document.createElement('link')
    googleApi.rel = "preconnect"
    googleApi.href = "https://fonts.gstatic.com"
    document.head.appendChild(googleApi)
    let googleApiTwo = document.createElement('link')
    googleApiTwo.rel = "preconnect"
    googleApiTwo.href = "https://fonts.googleapis.com"
    document.head.appendChild(googleApiTwo)
    let normilize = document.createElement('link')
    normilize.rel = "stylesheet"
    normilize.href = "https://cdnjs.cloudflare.com/ajax/libs/normalize/8.0.1/normalize.css"
    document.head.appendChild(normilize)
  },
  computed: {
    computedCategory: function () {
      let vm = this
      return this.received.categories.filter(function (item) {
        return item.title.toLowerCase().indexOf(vm.search.toLowerCase()) !== -1
      })
    },
    computedOther: function () {
      let vm = this
      return this.received.other.filter(function (item) {
        return item.title.toLowerCase().indexOf(vm.search.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    closest: function(el, selector) {
      if (el.matches(selector))
        return el
      if (Element.prototype.closest) {
        return el.closest(selector);
      }
      let parent = el;
      while (parent) {
        if (parent.matches(selector)) {
          return parent;
        }
        parent = parent.parentElement;
      }
      return null;
    },
    shuffle: function () {
      this.received.categories = _.shuffle(this.received.categories)
    },
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      let delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
            el,
            { opacity: 1, height: '46px' },
            { complete: (function(){el.style.height='auto';done}) }
        )
      }, delay)
    },
    leave: function (el, done) {
      let delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
            el,
            { opacity: 0, height: 0 },
            { complete: done }
        )
      }, delay)
    },
    startDrag(evt, item, index, categoryId = null) { /*Начало Драга*/
      this.moveItem = item
      // evtPropagation()
      evt.dataTransfer.dropEffect = 'move'
      evt.dataTransfer.effectAllowed = 'move'
      evt.dataTransfer.setData('itemID', index)
      evt.dataTransfer.setData('categoryId', categoryId)
      // console.log("Начало Драга", this.moveItem)
      evt.target.classList.add('selected')
    },
    endDrag(evt, item, index) { /*Закинуть в новое место*/
      evt.dataTransfer.dropEffect = 'move'
      evt.dataTransfer.effectAllowed = 'move'
      evt.dataTransfer.setData('itemID', index)
      evt.target.classList.remove('selected')
      this.rulesMove = false
      document.querySelectorAll('.drag-this').forEach(function(item){
        item.classList.remove('drag-this')
      })
    },
    enterDrag(evt, item, index) { /*Над чем элемент*/ /*Main Logis Working rules*/
      console.log(index, "Я стою на "+ item + " элементом " + this.moveItem + " А значит "+this.rulesMove)
      if (this.moveItem === item
       || this.moveItem  === 'document' && item === 'category'
       || this.moveItem  === 'document' && item === 'other'
       || this.moveItem  === 'other'    && item === 'document'
       || this.moveItem  === 'other'    && item === 'category') {
         this.rulesMove = true
        // if (item === 'category' || item === 'other' || item === 'document') {
        //   evt.target.classList.add('drag-this')
          console.log(this.closest(evt.target,`.${item}`))
          console.log("MetjodsEnter Work")
          this.tempDropElement = this.closest(evt.target,`.${item}`)
          this.closest(evt.target,`.${item}`).classList.add('drag-this')
      } else {
        this.rulesMove = false;
        return false;
      }
    },
    leaveDrag(evt, item, index) { /*Убрали драг*/
      let that = this
      console.log("MetjodsLeave Work")

      let x = document.querySelectorAll('.drag-this').forEach(function(item, i, arr){
        if (item !== that.tempDropElement) item.classList.remove('drag-this')
      })

    },
    onDrop (evt, item, itemIdBefore, category = null) {
      const itemID = evt.dataTransfer.getData('itemID') /*What i Move*/
      const categoryId = evt.dataTransfer.getData('categoryId') /*What in categ Move*/
      evt.target.classList.remove('selected')
      // evt.target.classList.remove('drag-this')

      this.closest(evt.target,`.${item}`).classList.remove('drag-this')
      let idxBeforeEl,idxDragEl
      if (!this.rulesMove) return false
      if (item === 'other' && this.received.other.length === 0) { /*If other array = 0*/
        let indexCategory,indexDocument,tempElArr
        for (let x=0; x<this.received.categories.length; x++) {
          if (categoryId == this.received.categories[x].id) {
            indexCategory = x; break;
          }
        }
        for (let i=0;i<this.received.categories[indexCategory].items.length;i++) {
          if (itemID == this.received.categories[indexCategory].items[i].id){
            indexDocument = i; break;
          }
        }
        tempElArr = this.received.categories[indexCategory].items[indexDocument]
        this.received.categories[indexCategory].items.splice(indexDocument, 1)
        this.received.other.splice(0, 0, tempElArr)
        return;
      }
      console.log(`Дропаю ${this.moveItem} в ${item}`)
      this.rulesMove = true
      if (item === this.moveItem && item === 'category'){
        for (let i = 0; i<this.received.categories.length; i++) {
          if (itemID == this.received.categories[i].id) {
            idxDragEl = i
          }
        }
        let tempElArr = this.received.categories[idxDragEl]
        if (itemIdBefore == itemID) return false
        this.received.categories.splice(idxDragEl, 1 )
        for (let i = 0; i<this.received.categories.length; i++) {
          if (itemIdBefore == this.received.categories[i].id) {
            idxBeforeEl = i+1;
          }
        }
        this.received.categories.splice(idxBeforeEl, 0, tempElArr)
      }

      if (this.moveItem==='document' && item === 'other'){
        let indexCategory,indexDocument,tempElArr
        for (let x=0; x<this.received.categories.length; x++) {
          if (categoryId == this.received.categories[x].id) {
            indexCategory = x; break;
          }
        }
        for (let i=0;i<this.received.categories[indexCategory].items.length;i++) {
          if (itemID == this.received.categories[indexCategory].items[i].id){
            indexDocument = i; break;
          }
        }
        tempElArr = this.received.categories[indexCategory].items[indexDocument]
        this.received.categories[indexCategory].items.splice(indexDocument, 1)
        for (let i=0;i<this.received.other.length;i++) {
          if (itemIdBefore == this.received.other[i].id) {
            this.received.other.splice(i+1, 0, tempElArr); break;
          }
        }
      }

      if (this.moveItem==='other' && item === 'document') {
        console.log('Я Указал other и перемещаю в '+ item + " ID- "+itemIdBefore + "Категория ", category)
        let indexCategory,indexDocument,indexOther, tempElArr
        for (let i=0;i<this.received.other.length; i++) {
          if (this.received.other[i].id == itemID) {
            tempElArr = {...this.received.other[i]}
            indexOther = i;
            break
          }
        }
        for (let i=0;i<this.received.categories.length; i++) {
          if (this.received.categories[i].id == category) {
            indexCategory = i;break
          }
        }
        console.log(indexCategory,category)
        for (let x=0;x<this.received.categories[indexCategory].items.length; x++) {
          if (this.received.categories[indexCategory].items[x].id == itemIdBefore) { /*Индекс эл. после которого вставка*/
            indexDocument = x;break
          }
        }
        this.received.categories[indexCategory].items.splice(indexDocument+1, 0 , tempElArr)
        this.received.other.splice(indexOther, 1)
        return true
      }

      if (this.moveItem==='other' && item === 'category' ) {
        console.log('Я Указал other и перемещаю в '+ item + " IDcat- "+itemIdBefore)
        let indexCategory,indexOther, tempElArr
        for (let i=0;i<this.received.other.length; i++) {
          if (this.received.other[i].id == itemID) {
            tempElArr = {...this.received.other[i]}
            indexOther = i;
            break
          }
        }
        for (let i=0;i<this.received.categories.length; i++) {
          if (this.received.categories[i].id == itemIdBefore) {
            indexCategory = i;break
          }
        }
        if (typeof this.received.categories[indexCategory].items === 'undefined') {
          this.$set(this.received.categories[indexCategory], 'items', [tempElArr])
          console.log(tempElArr, this.received.categories[indexCategory], "ASDASD")
        } else this.received.categories[indexCategory].items.splice(0, 0,tempElArr)
        this.received.other.splice(indexOther, 1)
        return true
      }

      if (this.moveItem==='other' && item === 'other' ) {
        for (let i = 0; i<this.received.other.length; i++) {
          if (itemID == this.received.other[i].id) {
            idxDragEl = i
          }
        }
        let tempElArr = this.received.other[idxDragEl]
        if (itemIdBefore == itemID) return false
        this.received.other.splice(idxDragEl, 1 )
        for (let i = 0; i<this.received.other.length; i++) {
          if (itemIdBefore == this.received.other[i].id) {
            idxBeforeEl = i+1;
          }
        }
        this.received.other.splice(idxBeforeEl, 0, tempElArr)
        return true
      }

      if (this.moveItem==='document' && item === 'document'){
        let indexCategory,whereCategory,whereBeforeItem,indexDocument,tempElArr
        for (let x=0; x<this.received.categories.length; x++) {
          if (categoryId == this.received.categories[x].id) {
            indexCategory = x;
          }
          if (category == this.received.categories[x].id) {
            whereCategory = x
          }
        }
        for (let i=0;i<this.received.categories[indexCategory].items.length;i++) {
          if (itemID == this.received.categories[indexCategory].items[i].id){
            indexDocument = i; break
          }
        }
        for (let i=0;i<this.received.categories[whereCategory].items.length;i++) {
          if (itemIdBefore == this.received.categories[whereCategory].items[i].id){
            whereBeforeItem = i; break
          }
        }
        tempElArr = this.received.categories[indexCategory].items[indexDocument]
        console.log(itemID,"itemID",indexCategory,"indexCategory",indexDocument,"indexDocument",whereBeforeItem,"whereBeforeItem",whereCategory,"whereCategory.received.categories[indexCategory]")
        this.received.categories[indexCategory].items.splice(indexDocument, 1)
        this.received.categories[whereCategory].items.splice(whereBeforeItem, 0, tempElArr);
      }

      if (this.moveItem==='document' && item === 'category') {
        console.log('Я Указал other и перемещаю в ' + item + " IDcat- " + itemIdBefore)
        let indexCategory, indexDocument, whereCategory,tempElArr
        if (category == categoryId) return;

        for (let i = 0; i < this.received.categories.length; i++) {
          if (this.received.categories[i].id == itemIdBefore) {
            whereCategory = i; break
          }
        }
        for (let i = 0; i < this.received.categories.length; i++) {
          if (this.received.categories[i].id == categoryId) {
            tempElArr = {...this.received.categories[i]}
            indexCategory = i;
            break
          }
        }

        for (let i = 0; i < this.received.categories[indexCategory].items.length; i++) {
          if (this.received.categories[indexCategory].items[i].id == itemID) {
            indexDocument = i;
            break
          }
        }
        console.log(indexCategory, indexDocument, whereCategory,tempElArr,"indexCategory, indexDocument, whereCategory,tempElArr")

        if (typeof this.received.categories[whereCategory].items === 'undefined') {
          this.$set(this.received.categories[whereCategory], 'items', [tempElArr])
          console.log(tempElArr, this.received.categories[whereCategory], "ASDASD")
        } else this.received.categories[whereCategory].items.splice(0, 0, tempElArr)
        this.received.categories[indexCategory].items.splice(indexDocument, 1)
        return true
      }
    }
  }
}
</script>

<style>

.flip-list-move {
  transition: transform 1.5s;
  /*transition: all 15s;*/
}
.flip-list-leave-active {
  /*transition: none;*/
}

.selected {
  opacity: 0.5;
  color: red;
  background: #c1c1c1;
  /*font-size: 10px;*/
}

.drop-zone {
  /*background-color: #eee;*/
  /*margin-bottom: 10px;*/
  /*padding: 10px;*/
  /*border-bottom: solid 2px #000;*/
  width: 100%;
}

/* .category:hover, .document:hover, .other:hover{*/
/*  border-bottom: solid 3px blue;*/
/*}*/
.drag-this{
  /*border-bottom: solid 3px blue;*/
  -webkit-box-shadow: 0px -5px 0px 0px rgba(0, 102, 255, 1) inset;
  -moz-box-shadow: 0px -5px 0px 0px rgba(0, 102, 255, 1) inset;
  box-shadow: 0px -5px 0px 0px rgba(0, 102, 255, 1) inset;
}

.drag-el {
  background-color: #fff;
  margin-bottom: 10px;
  padding: 5px;
}

.list-complete-item {
  transition: all 1s;
  /*display: inline-block;*/
}
.list-complete-enter, .list-complete-leave-to
  /* .list-complete-leave-active до версии 2.1.8 */ {
  opacity: 0;
  transform: translateX(30px);
}
.list-complete-leave-active {
  position: absolute;
}

#app {
  width: 1200px;
  margin:  0 auto;
  user-select: none;
  font-family: 'Fira Sans', sans-serif;
}
* {box-sizing: border-box;padding: 0; margin: 0;}
header {
  padding:  20px 0;
}
header .row {
  display:  flex;
  justify-content: space-between;
  align-items: center;
}
header .headerBtn {
  width: 40% ;
  text-align: right;
}

header .headerBtn .headerBtn-bookmark {
  padding: 5px 7px;
}
header .headerBtn div{
  border-radius: 50px;
  display: inline-block;
  padding: 5px 30px;
  margin-left: 10px;
  font-size: 12px;
  font-weight: bold;
  text-align: center;
  border: 1px solid #D3D8DF;
  transition: all .5s;
  position: relative;
}
header .headerBtn>div:hover {
  cursor: pointer;
  transform: translateY(-3px);
}

.headerBtn-newType i, .headerBtn-newDocument i{
  font-size: 17px;
  color: #0066FF;
  font-weight: bold;
  position: absolute;
  left: 10px;
  top: 1px;
  margin-right: 10px;
}

.row {
  width: 100%;
}
.searchForm {background: #F9F0DA;
  background: #A3D0C3;
  position: relative;
  width: 560px;
}
.searchForm input, .searchForm button {
  border: none;
  outline: none;
}
.searchForm input {
  width: 100%;
  height: 42px;
  padding-left: 30px;
  font-size: 15px;
  font-weight: bold;
  line-height: 16px;
  border-bottom: solid 1px #C2C5CD;
}
.searchForm button#search {
  height: 42px;
  width: 42px;
  position: absolute;
  top: 0;
  left: -15px;
  cursor: pointer;
  background: none;
}
.searchForm button#search:before {
  content: "\f002";
  font-weight: 100;
  font-family: FontAwesome;
  font-size: 20px;
  color: #C2C5CD;
}
.searchForm span#clear {
  position: absolute;
  right: 0;
  font-weight: 100;
  font-family: FontAwesome;
  font-size: 20px;
  color: #FF238D;
  margin-top: 10px;
  cursor: pointer;
}


.item {
  position: relative;
  width: 100%;
  display: flex;
  flex-wrap: nowrap;
  align-items: center;
  height: 48px;
  border: 1px solid #DFE4EF;
  line-height: 47px;
  overflow: hidden;
}
/*.item div {float: left;}*/

.item.document {
  height: 35px;
  line-height: 34px;
  padding-left: 16px;
  width: 98%;
  /*float: right;*/
  display: flex;
  margin: 0 0 0 auto;
  border-top: none;
}

.item.other {
  height: 35px;
  line-height: 34px;
  padding-left: 16px;
  border-top: none;
}
.mainBlock>div:nth-child(2){
  /*border-top: 1px solid #DFE4EF;*/
}


.mainBlock>div:first-child {
  /*margin-bottom: 35px;*/
}

.item-toggle {
  border: 1px solid #D3D8DF;
  width: 22px;
  height: 22px;
  margin-left: 16px;
  margin-right: 16px;
  /*float: left;*/
  border-radius: 50%;
  /*margin-top: 12px;*/
  position: relative;
  color: #0066FF;
}
.rotate {
  transform: rotate(180deg);
}
.item-toggle i {
  top: 0; left: 0; right: 0; bottom: 0;
  position: absolute;
  text-align: center;
  line-height: 20px;
  transition: all .5s;
}
.item-title {
  margin-right: 16px;
  /*padding-left: 16px;*/
  font-size: 15px;
  font-weight: bold;
  display: flex;
  flex-direction: row;
  white-space: nowrap;
}
.item-circles {
  margin-right: 16px;
  display: flex;
  flex-direction: row;
  /*margin-top: 20px;*/
}
.item-circle {
  border-radius: 50%;
  height: 8px;
  width: 8px;
  background: red;
  margin-right: 6px;
  /*float: left;*/
}
span.item-circle:nth-child(1n) {
  background: #d06767;
}
span.item-circle:nth-child(2n) {
  background: rgba(0, 128, 0, 0.84);
}
span.item-circle:nth-child(3n) {
  background: #bebe3e;
}
.document .item-circles {
  margin-top: 13px;
}
.item-alert {
  color:  #FF238D;
  font-size:  11px;
  margin-right: 16px;
  display: flex;
  flex-direction: row;
  white-space: nowrap;
}
.item-description {
  font-size: 11px;
  color: #8E9CBB;
  white-space: nowrap;
  overflow: hidden;
  display: flex;
  flex-direction: row;
}
.item .item-actions {
  position: absolute;
  /*float: right;*/
  right: 0;
  /*background: rgb(255,255,255);*/
  /*background: linear-gradient(90deg, rgba(255,255,255,0) 0%, rgba(255,255,255,1) 10%);*/
  padding: 0 17px;
}
.item .item-actions span {
  margin-right: 26px;
  transition: all .5s;
  cursor: pointer;
}
.item .item-actions span:last-child {
  margin-right: 0;
}
.item .item-actions span:nth-child(1){
  color: #8E9CBB;
}
.item .item-actions span:nth-child(2){
  color: #FF238D;
}
.item .item-actions span:nth-child(3){
  color: #8E9CBB;
}
.item .item-actions span:hover {
  opacity: 0.7;
}

/*modalWindow*/

.modal-container {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background: hsla(0, 0%, 40%, .6);
}

/* using :target */
.modal-container:target {
  display: flex;
}

.modal {
  width: 40rem;
  padding: 4rem 2rem;
  border-radius: .8rem;

  color: #fff;
  background: rgb(48,104,207);
  background: linear-gradient(308deg, rgba(48,104,207,1) 0%, rgba(146,143,222,1) 100%);
  box-shadow: .4rem .4rem 2.4rem .2rem hsla(236, 50%, 50%, 0.3);
  position: relative;
  margin: 0 auto;
  overflow: hidden;
}

.modal__details {
  text-align: center;

  margin-bottom: 7px;
  padding-bottom: 7px;
  border-bottom: 1px solid hsla(0, 0%, 100%, .4);
}

.modal__title {
  /*font-size: 3.2rem;*/
  margin: 0;
}

.modal__description {
  /*margin-top: 2rem;*/

  /*font-size: 1.6rem;*/
  /*font-style: italic;*/
}

.modal__text {
  /*padding: 0 4rem;*/
  /*margin-bottom: 4rem;*/

  /*font-size: 1.6rem;*/
  /*line-height: 2;*/
}

.modal__text::before {
  content: '';

  position: absolute;
  top: 0%;
  left: 100%;
  transform: translate(-50%, -50%);

  width: 18rem;
  height: 18rem;
  border: 1px solid hsla(0, 0%, 100%, .2);
  border-radius: 100rem;

  pointer-events: none;
}

.modal__btn {
  position: absolute;
  padding: 1rem 1.6rem;
  border: 1px solid hsla(0, 0%, 100%, .4);
  border-radius: 100rem;
  color: inherit;
  background: transparent;
  /*font-size: 1.4rem;*/
  font-family: inherit;
  letter-spacing: .2rem;
  transition: .2s;
  cursor: pointer;
  bottom: 10px;
  right: 23px;
}

.modal__btn:hover,
.modal__btn:focus {
  border-color: hsla(0, 0%, 100%, .6);
  transform: translateY(-.2rem);
}

/* links */
/* =============================================== */
.link-1 {
  font-size: 1.8rem;

  color: var(--light);
  background: var(--background);
  box-shadow: .4rem .4rem 2.4rem .2rem hsla(236, 50%, 50%, 0.3);
  border-radius: 100rem;
  padding: 1.4rem 3.2rem;

  transition: .2s;
}

.link-1:hover,
.link-1:focus {
  transform: translateY(-.2rem);
  box-shadow: 0 0 4.4rem .2rem hsla(236, 50%, 50%, 0.4);
}

.link-2 {
  width: 4rem;
  height: 4rem;
  border: 1px solid hsla(0, 0%, 100%, .4);
  border-radius: 100rem;

  color: inherit;
  font-size: 2.2rem;

  position: absolute;
  top: 2rem;
  right: 2rem;

  display: flex;
  justify-content: center;
  align-items: center;

  transition: .2s;
}

.link-2::before {
  content: '×';

  transform: translateY(-.1rem);
}

.link-2:hover,
.link-2:focus {
  border-color: hsla(0, 0%, 100%, .6);
  transform: translateY(-.2rem);
}

/* Second Version Link */
/* =============================================== */
.second-version-link,
.second-version-link:link {
  color: hsl(236, 50%, 50%);
  padding: .8rem 1.6rem .8rem .2rem;
  border-bottom: 2px solid hsl(236, 50%, 50%);

  font-size: 1.4rem;
  font-weight: bold;

  position: absolute;
  top: 4rem;
  right: 4rem;
}

.second-version-link::after {
  content: '\2197';

  position: absolute;
  top: 0;
  right: 0;

  font-size: .9em;
}

.abs-site-link {
  position: fixed;
  bottom: 20px;
  left: 20px;
  color: hsla(0, 0%, 0%, .6);
  font-size: 1.6rem;
}
</style>
