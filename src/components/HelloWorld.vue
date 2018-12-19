<template>
  <div class="root">
    <div class="tabs">
      <div class="tabs_scroll" ref="tabs_scroll">
        <div class="tabs_container">
          <ul class="tabs_list" ref="tabs_list" @click="clickhandler">
            <li class="tabs_item tabs_item_active" data-index="0"  ref="tab1">今日疯抢</li>
            <li class="tabs_item" data-index="1" ref="tab2">女装饰品</li>
          </ul>
          <div class="tabs_line" ref="tabLine"></div>
        </div>
      </div>
    </div>
    <div class="content" ref="content" touch-action="none"
      @touchstart="touchstart"
      @touchmove="touchmove"
      @touchend="touchend"
    >
      <div class="content_list" ref="content_list">
        <div class="content_item" ref="content_item1">
          <ul>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
            <li>第一个</li>
          </ul>
          
        </div>
        <div class="content_item" ref="content_item2">
          <ul>
            <li>第二个</li>
            <li>第二个</li>
            <li>第二个</li>
            <li>第二个</li>
            <li>第二个</li>
            <li>第二个</li>
            <li>第二个</li>
          </ul>          
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return {
      currIndex:0,
      touching:false,
      isMoved:false,
      startX:0,
      startY:0,
      translate:0,
      diffX:0,
      diffY:0,
      DIS:50, //用于判断是否切换到下一个tab，如果大于该值，则切换到下一个tab
    }
  },
  methods:{
    touchstart:function(e){
      if (this.touching) return;
      var touch = e.targetTouches[0];
      this.startX = touch.pageX;
      this.startY = touch.pageY;
      this.touching = true;
      //touchmove时，移动的距离不加过渡效果，不然页面会抖动。
      let contentList = this.$refs.content_list
      contentList.classList.remove('content_list_transition');
      console.log("hello");
    },
    touchmove:function(e){
      if (!this.touching) return;
      let touch = e.targetTouches[0]
      let pageX = touch.pageX
      let pageY = touch.pageY
      this.diffX = pageX - this.startX;
      this.diffY = pageY - this.startY;
      if (this.isMoved) {
          //e.preventDefault();//如果对阻止默认行为不做判断，页面滚动将被禁止。
          let viewWidth = window.innerWidth
          let widthdivThree = viewWidth / 3;
          //第一个tab向右滑动，最多滑动页面宽度的三分之一
          if (this.translate === 0 && this.diffX > widthdivThree) {
              this.diffX = widthdivThree;
              return;
          }
          //最后一个tab向左滑动，最多滑动页面宽度的三分之一
          let totalWidth = viewWidth
          if (this.translate === totalWidth && this.diffX < -widthdivThree) {
              this.diffX = -widthdivThree;
              return;
          }
          //每次移动，页面内容也跟着移动对应距离
          if (this.diffX !== 0) {
              let translateX = this.diffX - this.translate;
              let contentList = this.$refs.content_list
              contentList.style.cssText = '-webkit-transform:translateX(' + translateX + 'px);transform:translateX(' + translateX + 'px);';
          }
      } else {
          let diffZ = Math.sqrt(Math.pow(this.diffX, 2) + Math.pow(this.diffY, 2))
          let angle = this.getCosAngel(Math.abs(this.diffY), diffZ);
          //如果符合该条件，页面将可以左右滑动；如果不符合该条件，页面将会上下滚动。阻止了上下滑动时触发左右滑动的执行的内容
          if (diffZ > 5 && angle > 80) {
              //e.preventDefault();
              this.isMoved = true;
          }
      }
    },
    touchend:function(e){
      if (this.touching && this.isMoved) {
        let contentList = this.$refs.content_list
        contentList.classList.add('content_list_transition');
        let viewWidth = window.innerWidth
        let totalWidth = viewWidth
        //第一个向右滑动或者最后一个向左滑动时，不切换tab
        if ((this.translate === 0 && this.diffX > 0) || (this.translate === totalWidth && this.diffX < 0)) {
            this.translate = this.translate;
        }
        //如果滑动距离大于屏DIS，切换到下一个tab;否则返回到滑动之前的tab
        else if (Math.abs(this.diffX) > this.DIS) {
            this.translate = this.diffX > 0 ? (this.translate - viewWidth) : (this.translate + viewWidth);
            this.currIndex = this.diffX > 0 ? (this.currIndex - 1) : (this.currIndex + 1);
            let tab = this.$refs.tab1
            if(this.currIndex==1){
              tab = this.$refs.tab2
            }
            this.selectedTab(this.currIndex, tab);
        }
        let contentItem = this.$refs.content_item1
        if(this.currIndex==1){
          contentItem = this.$refs.content_item2
        }
        contentList.style.cssText = 'height:' + contentItem.offsetHeight + 'px;webkit-transform:translateX(-' + this.translate + 'px);transform:translateX(-' + this.translate + 'px);';
      }
      this.touching = false;
      this.isMoved = false;
    },

    clickhandler:function (e) {
      e.preventDefault();
      var target = e.target;
      if (target.classList.contains('tabs_item') && !target.classList.contains('tabs_item_active')) {
          var index = parseInt(target.dataset['index']);
          //选中tab
          this.selectedTab(index, target);
          //显示选中tab内容
          let viewWidth = window.innerWidth
          let translate = index * viewWidth;
          let contentList = this.$refs.content_list
          let contentItem = this.$refs.content_item1
          if(index==1) contentItem = this.$refs.content_item2
          contentList.style.cssText = 'height:' + contentItem.offsetHeight + 'px;-webkit-transform:translateX(-' + translate + 'px);transform:translateX(-' + translate + 'px);';
          //存储选中tab的index
          this.currIndex = index;
      }
    },
    /**
     * 选中tab后，样式的变化，包括：字体变红、重新设置线宽、线移动到选中tab下方、调整滚动区域的位置，合理显示选中tab
     * @param index：选中tab的索引
     * @param target：选中tab元素
     */
    selectedTab: function(index, target) {
        //tab字体变红
        this.changeSelectedTab(index);
        //重新设置线宽
        let tabWidth = target.offsetWidth;
        //线移动到点击内容下方
        let lineTranslateX = target.offsetLeft;
        let tabLine = this.$refs.tabLine
        tabLine.style.cssText = 'width:' + tabWidth + 'px;-webkit-transform:translateX(' + lineTranslateX + 'px);transform:translateX(' + lineTranslateX + 'px);';
        //调整选中tab的位置
        let tabs_scroll = this.$refs.tabs_scroll
        let viewWidth = window.innerWidth
        if (lineTranslateX > viewWidth / 2) {
            tabs_scroll.scrollLeft = lineTranslateX / 2;
        } else {
            tabs_scroll.scrollLeft = 0;
        }
    },
    /**
     * 选中tab,字体变红
     * @param index：选中tab的索引
     */
    changeSelectedTab: function(index) {
      let tab1 = this.$refs.tab1
      let tab2 = this.$refs.tab2
      if(index==0){
        tab1.classList.add('tabs_item_active');
        tab2.classList.remove('tabs_item_active');
      }else{
        tab2.classList.add('tabs_item_active');
        tab1.classList.remove('tabs_item_active');
      }
    },
    /**
     * 计算cos两条边夹角的角度
     * @param y：三角形一条直角边
     * @param z：三角形跟这条边相邻的斜边
     * @returns {number}：夹角角度
     */
    getCosAngel: function(y, z) {
        var cos = y / z;
        return Math.acos(cos) / Math.PI * 180;
    },
  },
  mounted(){
    console.log(this.$refs.tabs_list);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.root{
}
/*tabs导航行条*/
.tabs {
    z-index: 10;
    position: -webkit-sticky;
    position: sticky;
    top: -1px;
    padding: 0 5px;
    height: 42px;
    background-color: #FFFFFF;
    -webkit-box-shadow: 0 1px 1px 0 rgba(0, 0, 0, .12);
    -moz-box-shadow: 0 1px 1px 0 rgba(0, 0, 0, .12);
    box-shadow: 0 1px 1px 0 rgba(0, 0, 0, .12);
    overflow: hidden;
}
.tabs_scroll {
    width: 100%;
    /*高度要大于42px，不然滚动时将会显示下方的滚动条*/
    height: 60px;
    overflow-x: scroll;
    -webkit-overflow-scrolling: touch;
}
.tabs_container {
    position: relative;
    overflow: hidden;
    width: 100%;
}
.tabs_list {
    float: left;
    /*解决inline-block元素之间的间隙*/
    font-size: 0;
    /*inline-block一行显示*/
    white-space: nowrap;
    width: 100%;
}
.tabs_item {
    display: inline-block;
    box-sizing: border-box;
    height: 42px;
    min-width: 50px;
    line-height: 42px;
    text-align: center;
    padding: 0 8px;
    font-size: 16px;
    color: #666666;
    -webkit-transition: color .5s;
    -moz-transition: color .5s;
    -ms-transition: color .5s;
    -o-transition: color .5s;
    transition: color .5s;
    width: 50%;
}
.tabs_item_active {
    color: #ec1c43;
}
.tabs_line {
    position: absolute;
    left: 0;
    bottom: 0;
    height: 3px;
    background-color: #ec1c43;
    -webkit-transition: -webkit-transform .3s linear;
    transition: transform .3s linear;
    width: 50%;
}

/*tab内容展示*/
.content {
    overflow: hidden;
}
.content_list {
    float: left;
    font-size: 88px;
    white-space: nowrap;
}
.content_list_transition {
    -webkit-transition: -webkit-transform .3s linear;
    transition: transform .3s linear;
}
.content_item {
    display: inline-block;
    width: 375px;
    vertical-align: top;
}
</style>
