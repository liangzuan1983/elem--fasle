<template>
<div>
    <div class="goods">
        <div class="menu-wrapper" ref="wrapperMenu" id="wrapper_menu">
            <ul>
                <li 
                    class="food-list-hook"
                    v-for="(item,index) in goods" 
                    :class="{activity_menu: menuIndex == index }"
                    @click ="chooseMenu( index)"
                    :key="item.id">
                    {{item.name}}
                </li>
            </ul>
        </div>
        <div class="foods-wrapper"  ref="menuFoodList">
            <!--右侧内容区-->
            
                <!--<h3>窗前明月光</h3>-->
                <ul class="item-right">
                    <li  v-for="(item,index) in goods"  :key="item.id">
                        <!--列表-->
                        <div class="item clear" v-for="n in 5" :key="n" >
                            <div class="item-left">
                                <img />
                            </div>
                            <div class="item-right">
                                <h4>好吃的动心{{ n }}</h4>
                                <p>好吃的动心</p>
                                <p>月售673份 <span>好评路</span></p>
                                <p>月售673份 <span>好评路</span><i class="icon iconfont icon-jiahao" @touchstart="addToCart(1,2,30,'苹果')"></i></p>
                            </div>
                        </div>
                    </li>
                </ul>
            
            
            <!--右侧内容区-->
        </div>
     </div>
    <div class="shopcart">
        <cart></cart>
    </div>
</div>  
    
</template>
<script>
//引入http请求
import axios from 'axios';
//使用移动端事件
import BScroll from "better-scroll";

import cart from "@/components/com/cart"
//从vuex中引入两个方法
import {mapState, mapMutations} from 'vuex'

export default {
  props:['shopid','restaurantid'],
  data(){
        return {
            goods: [],
            api : '/dpi/shopping/restaurant',
            menuapi: '/dpi/shopping/v2/menu',
            listHeight: [] ,  //计算y轴累加的高度
            scrollY: 0 ,   // y周方向滚动的距离

            shopListTop: [] , //右侧商品的高度结合
            menuIndexChange:true,
            menuIndex: 0,
            showLoading: true, //  判断数据是否加载完成，同样可以作为前置动画的开关
        }
  },
  components:{
      cart
  },
  computed: {
        ...mapState(['cartList'])
  },
  mounted(){
   let that = this;
    //左侧区域导航菜单
    axios.get(this.menuapi,{
        params :{
            restaurant_id : that.restaurantid
        }
    }).then(function(res){
        that.goods = res.data;
        that.dataCompleted()
       
    })
     
  },
  //定义滚动的方法
  methods: {
    ...mapMutations([
        'add_cart'
    ]),
    //定义一个方法添加商品到购物车
     addToCart(shopid,food_id,price,name){
        //更改数据仓库里面的数值
        this.add_cart({ shopid,food_id,price,name})
    },

     //判断数据是否加载完成
    dataCompleted(){
        this.showLoading = false;
    },
    //获取左侧食品列表的高度
    getFoodListHeight(){
        const listContainer = this.$refs.menuFoodList;
        const listArr = Array.from(listContainer.children[0].children);
        listArr.forEach((item, index) => {
            this.shopListTop[index] = item.offsetTop;
        });
        this.listenScroll(listContainer)
    },
    //当滑动食品列表的时，监听其scrollTop值来设置对应的食品列表标题的样式
    listenScroll(element){
         this.foodScroll = new BScroll(element, {
            probeType: 3,
            deceleration: 0.001,
            swipeTime: 2000,
            click: true,
         });
        const wrapperMenu = new BScroll('#wrapper_menu', {
            click: true,
        });
        const wrapMenuHeight = this.$refs.wrapperMenu.clientHeight;
        this.foodScroll.on('scroll', (pos)=> {
            if(!this.$refs.wrapperMenu){
                return;
            }
            this.shopListTop.forEach((item, index) => {
                if (this.menuIndexChange && Math.abs(Math.round(pos.y)) >= item) {
                    this.menuIndex = index;
                    const menuList=this.$refs.wrapperMenu.querySelectorAll('.activity_menu');
                    const el = menuList[0];
                    wrapperMenu.scrollToElement(el, 800, 0, -(wrapMenuHeight/2 - 50));
                }
            })
        })


    },
    //菜单的运动
    chooseMenu(index){
        this.menuIndex = index;
        //menuIndexChange解决运动时listenScroll依然监听的bug
        this.menuIndexChange = false;
        this.foodScroll.scrollTo(0, -this.shopListTop[index], 400);
        this.foodScroll.on('scrollEnd', () => {
            this.menuIndexChange = true;
        })
    },
      
  },
  computed: {
       
  },
  //定义watch方法监视数据的变化
  watch:{
        showLoading:function(value){
            if(!value){
                this.$nextTick( () => {
                    this.getFoodListHeight()
                })
            }
        }
  }
}
</script>
<style lang="less">
.goods{
    display: flex;
    position: absolute;
    top: 184px;
    bottom: 46px;
    width: 100%;
    overflow: hidden;
    background: white;
    z-index: 99;
    .menu-wrapper{
        width: 80px;
        background: #f3f5f7;
        overflow: hidden;
        ul{
            li{
                font-size: 11/20rem;
                text-align: center;
                padding: .7rem 0rem;
                border-bottom: .025rem solid #ccc;
                box-sizing: border-box;
                background-color: #EEE;
            }
            li.activity_menu{
               border-left: 0.15rem solid #3190e8;
                background-color: #fff;
            }
        }
    } 
    .foods-wrapper{
        flex: 1;
         .item-right{
            flex:4;
            h3{
                font-size: 12 / 20rem ;
                padding: .5rem .5rem;
                background-color: #EEE;
            }
            .item{
                padding: 10/20rem;
                .item-left{
                    width: 2.9rem;
                    height: 2.9rem;
                    background-color: #eee;
                    float: left;
                    img{
                        width: 2.9rem;
                        height: 2.9rem;
                    }
                }
                .item-right{
                    overflow: hidden;
                    padding-left: 10/20rem;
                    h4{
                        font-size: 14/20rem;
                    }
                    p{
                        font-size: 8/20rem; 
                        padding-top: .3rem;
                        i{
                            margin-left: 40px;
                            color: #007ACC;
                        }
                    }
                }
            }
        }
    }
}
.shopcart{
    position: absolute;
    height: 46px;
    bottom: 0px;
    width: 100%;
}
</style>



