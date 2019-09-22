<template>
  <div class="ion-page">
    <ion-header>
      <ion-toolbar>
        <ion-title>Hello World</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div class="wrapper">
        <div class="elemnts-container"> 
            <div @click="clickHandle" data-index="0" class="element active"></div>
            <div @click="clickHandle" data-index="1" class="element"></div>
            <div @click="clickHandle" data-index="2" class="element"></div>
        </div>
      </div>

      <br>    
      <!-- @ionSlidesDidLoad="DragHandle" -->
      <ion-slides  
          @ionSlideTouchStart="setGlobalRange"
          @ionSlidesDidLoad="DragHandle"
          @ionSlideTouchEnd="finalHandle">
        <ion-slide>
          <h1>Slide 1</h1>
        </ion-slide>
        <ion-slide>
          <h1>Slide 2</h1>
        </ion-slide>
        <ion-slide>
          <h1>Slide 3</h1>
        </ion-slide>
      </ion-slides>

    </ion-content>
  </div>
</template>

<style>
  .transition {
    transition: all linear .2s;
  }

  .center {
    height: 100vh;
    width: 1px;
    border-right: 2px solid red;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .wrapper {
    width: 100%;
  }
  
  .elemnts-container {
    display: block;
    /* flex-direction: row;
    justify-content: flex-start; */
    white-space: nowrap;
  }
  
  .elemnts-container::after {
    content: "";
    clear: both;
    display: table;
  }

  .element {
    height: 80px;
    /* float: left; */
    width: 80px;
    display: inline-block;
    /* zoom: .9; */
    opacity: .5;
    vertical-align: top;
  }

  .element.active {
    opacity: 1;
  }

  .element + .element {
    margin-left: 20px;
  }

  ion-slide:nth-child(1) h1,
  .element:nth-child(1) {
    background: blue;
  }

  ion-slide:nth-child(2) h1,
  .element:nth-child(2) {
    background: red;
  }

  ion-slide:nth-child(3) h1,
  .element:nth-child(3) {
    background: purple;
  }

  ion-slide h1 {
    margin: 0;
    height: 200px;
    width: 100%;
  }

  ion-content {
    --padding-start: 0px;
    --padding-end: 0px;
  }

</style>

<script>
export default {
  data() {
    return {
      space: 0,
      width: 0,
      range: 0,
      position: 0,
      state: 0,
    }
  },
  methods: {
    setGlobalRange() {
      var el = document.querySelector('.elemnts-container')
      this.position = this.getComputedTranslateXY(el)[0]
      this.state = 1
      
    },

    async finalHandle(e) {
      this.state = 0
      let number = await e.target.getActiveIndex();
      console.log(number)

      let el = document.querySelector(`.element:nth-child(${number+1})`)

      $('.element').removeClass('active');
      $(el).addClass('active');

      this.centerElement(el)
    },
    DragHandle(e) {

      var _this = this;
      let wrapper = e.target.querySelector('.swiper-wrapper')
      let el = e.target
      let width = el.getBoundingClientRect().width;

      var observer = new MutationObserver(function(mutations) {
        mutations.forEach(function(mutation) {
          if (mutation.type == "attributes") {

            if (_this.state == 0) {
              console.log('nope')
              return;
            }

             e.target.getSwiper().then(swipper => {
               var values = _this.getComputedTranslateXY(wrapper)
                var transx = values[0]
                var moved = transx + (width * swipper.activeIndex);
                var percent = (100 * moved) / width;
                var result = (_this.range * percent) / 100
                var elContainer = document.querySelector('.elemnts-container')
                var final = _this.position + result;
                $('.elemnts-container').removeClass('transition')
                $('.elemnts-container').css({
                  'transform': `translate3d(${final}px, 0px, 0px)`
                });
             })
          }
        });
      })

      var ele = document.querySelector('.swiper-wrapper');
      observer.observe(ele, {
        attributes: true //configure it to listen to attribute changes
      });
      
      


      // e.target.getSwiper().then(swipper => {
      //   swipper.on('sliderMove',() => {
      //     window.setTimeout( () => {

      //       var values = _this.getComputedTranslateXY(wrapper)
      //       var transx = values[0]
      //       var moved = transx + (width * swipper.activeIndex);
      //       var percent = (100 * moved) / width;
      //       var result = (_this.range * percent) / 100
      //       var elContainer = document.querySelector('.elemnts-container')
      //       var final = _this.position + result;
      //       $('.elemnts-container').removeClass('transition')
      //       $('.elemnts-container').css({
      //         'transform': `translate3d(${final}px, 0px, 0px)`
      //       });
              
      //     }, 0)
      //   })
      // })

    },

    getComputedTranslateXY(obj) {
      const transArr = [];
        if(!window.getComputedStyle) return;
        const style = getComputedStyle(obj),
            transform = style.transform || style.webkitTransform || style.mozTransform;
        let mat = transform.match(/^matrix3d\((.+)\)$/);    
        if(mat) return parseFloat(mat[1].split(', ')[13]);
        mat = transform.match(/^matrix\((.+)\)$/);
        mat ? transArr.push(parseFloat(mat[1].split(', ')[4])) : 0;
        mat ? transArr.push(parseFloat(mat[1].split(', ')[5])) : 0;
        return transArr;
    },

    centerElement(element) {
      $('.elemnts-container').addClass('transition')
      var wrapper = document.querySelector('.wrapper');
      let leftContainerWidth = document.querySelector('.elemnts-container').getBoundingClientRect().left;
      var leftWidth = element.getBoundingClientRect().left;
      var lef = leftWidth - leftContainerWidth;
      var widht = element.offsetWidth;
      var center = wrapper.offsetWidth / 2;
      var move = center - (widht/2) - lef;

      $('.elemnts-container').css({
        'transform': `translate3d(${move}px, 0px, 0px)`
      })
    },

    clickHandle(e) {
      $('.element').removeClass('active');
      $(e.target).addClass('active');
      let index = e.target.dataset.index

      let sw = document.querySelector("ion-slides")
      sw.slideTo(index, 200).then(() => {
        this.centerElement(e.target);
      })
    }
  },

  mounted() {

    // window.setInterval(() => {
    //   var el = document.querySelector('.swiper-wrapper');
    //   var sw = document.querySelector('ion-slides');

    //   this.DragHandle({
    //     target: sw
    //   });
    //   console.log(this.getComputedTranslateXY(el));
    // }, 1);


    
    var el = document.querySelector('.element.active')
    var wid = $(el).innerWidth();
    var max = wid * 3;

    this.width = wid;
    this.space = ((window.innerWidth - max) / 2) - 16;    
    
    this.range = this.width + this.space
  
    $('.element + .element').css('margin-left', `${this.space}px`)

    window.setTimeout(() => {
      this.centerElement(el);
    }, 200);

    window.setTimeout(() => {
      this.centerElement(el);
      $('.elemnts-container').addClass('transition');
    }, 300);
  }

}
</script>