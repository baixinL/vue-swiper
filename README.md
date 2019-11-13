# vue-swiper


在vue使用swiper,loop:true实现轮播时，发现出现空白页。
 
情景：最后切换到第一张的时候出现个<img src=''>的元素。<br><br>
 
 猜测原因时，swiper在在图片资源数据未完全加载前执行init(),导致克隆到的不完整的元素。


1.swiperOption配置init:true,出现空白页，dom中的img的src为空,原因是swiper在图片未加载，数据未加入完成的之前克隆了这个元素，所以src是空的；


2.我尝试swiperOption配置init:false,手动在vue的mounted中加入this.swiper.init(),还是不行；


3.在2的前提下,将this.swiper.init()外中裹上一层setTimeOut,发现可以了，；在我尝试切换到低网速的情况查看时，又出现空白页，明显使用setTimeOut来解决不靠谱的，这个延时时长不好确定；


4.然后我放弃3，在vue的updated()内加入this.swiper.init()和裹上一层$nextTick，情况如同3，一样不靠谱；

我在想如何才能确保图片一定已加载完成呢？

5.我有尝试监听img的load事件，试图在最后一张图片load之后，this.swiper.init(),按理来说应该可以解决吧，然而并没有，因为分开监听每一图片，可能会出现最后一张比前面还要早加载出来，通过判断最后一张图片是否加载完成，显示也是不靠谱的，不过已经很接近问题的终点了，哈哈哈。。。
接着，重点来了。
6.我使用了window.addEventListener('load', (event) => {}),并将放在vue的mounted中，感觉vue的beforeMount应该可是可以的，问题就解决啦。原因时window的load是页面、相关文件、图片等等都加载完成之后才触发的，所有可以确保图片已加载完成。。。
我的总结是:   这个问题是在确保图片数据已加载完成之后让swiper初始化。。。。。
或者还有其它的方法可以解决这个问题


第一分享 除bug 心得，请多多指教和支持。。。谢谢
 
#以下是我的代码实现：<br><br>

	<template>
	  <div class="banner">
	    <swiper :options="swiperOption" ref="mySwiper">
	      <swiper-slide v-for="(item, index) in bannersList" :key="index">
			<img v-lazy="'static/'+item.prodcutImg" class="swiper-img"  v-on:load="imgOnload(index)" alt />
	      </swiper-slide>
	      <div class="swiper-pagination" slot="pagination"></div>
	    </swiper>

	  </div>
	</template>

	<script>
	import "../../dist/static/css/swiper.css";
	import { swiper, swiperSlide } from 'vue-awesome-swiper'
	  export default {
	    name: 'BannerSwiper',
	    props:["bannersList"],
	    data() {
	      return {
		swiperOption: {
		  loop: true,
		  slidesPerView: 'auto',
		  loopedSlides: 8,
		  init:false,
		  speed:800,
		  freeMode:true,
		  pagination: {
		    el: '.swiper-pagination'
		  },
		  autoplay: {
		      delay: 2000,
		      stopOnLastSlide: false,
		      disableOnInteraction: false
		  }
		}
	      }
	    },
	    components: {
	      swiper,
	      swiperSlide
	    },
	    computed: {
	      swiper() {
		return this.$refs.mySwiper.swiper
	      }
	    },
	    mounted() {
	      window.addEventListener('load', (event) => {
		console.log('page is fully loaded');
		this.swiper.init()

	      });

	    },
	    updated() {
	      console.log("updated");
	    },
	    methods:{
	      imgOnload(index){
		console.log("imgOnload",index);
	      }
	    }
	  }
	</script>


<br><br>
#一下是我的console的打印：<br><br>
	updated (这里是vue的updated)
	imgOnload 1(这里是vue的每张图片加载完成后)
	imgOnload 3
	imgOnload 0
	imgOnload 2
	imgOnload 0
	page is fully loaded (这里是window.onload)
	imgOnload 1
	imgOnload 2
	imgOnload 3
