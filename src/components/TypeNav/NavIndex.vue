<template>
  <!-- 商品分类导航 -->
  <div class="type-nav">
    <div class="container">
      <div @mouseleave="loseColor" @mouseenter="enterShow">
        <h2 class="all">全部商品分类</h2>
        <transition name="sort">
          <div class="sort" v-show="show">
            <!-- 利用事件委派以及编程式导航实现点击a标签时实现路由跳转 -->
            <div class="all-sort-list2" @click="goSearch">
              <div
                class="item bo"
                v-for="(c1, index) in categoryList"
                :key="c1.categoryId"
                :class="{ cur: currentIndex == index }"
              >
                <h3 @mouseenter="focusColor(index)">
                  <a
                    :data-categoryName="c1.categoryName"
                    :data-category1Id="c1.categoryId"
                    >{{ c1.categoryName }}</a
                  >
                </h3>
                <div
                  class="item-list clearfix"
                  :style="{ display: currentIndex == index ? 'block' : 'none' }"
                >
                  <div
                    class="subitem"
                    v-for="c2 in c1.categoryChild"
                    :key="c2.categoryId"
                  >
                    <dl class="fore">
                      <dt>
                        <a
                          :data-categoryName="c2.categoryName"
                          :data-category2Id="c2.categoryId"
                          >{{ c2.categoryName }}</a
                        >
                      </dt>
                      <dd>
                        <em v-for="c3 in c2.categoryChild" :key="c3.categoryId">
                          <a
                            :data-categoryName="c3.categoryName"
                            :data-category3Id="c3.categoryId"
                            >{{ c3.categoryName }}</a
                          >
                        </em>
                      </dd>
                    </dl>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </transition>
      </div>
      <nav class="nav">
        <a href="###">服装城</a>
        <a href="###">美妆馆</a>
        <a href="###">尚品汇超市</a>
        <a href="###">全球购</a>
        <a href="###">闪购</a>
        <a href="###">团购</a>
        <a href="###">有趣</a>
        <a href="###">秒杀</a>
      </nav>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
// 这种引入的方式是把lodash全部的功能函数引入
// import _ from "lodash";
// 最好的引入方式：按需引入
import throttle from "lodash/throttle";

export default {
  name: "NavIndex",
  // 当组件挂载完毕，可以向服务器发请求
  mounted() {
    // 通知vuex发请求，获取数据，存储于仓库中
    // 把以下放到app组件的mounted里使得只挂载一次
    // this.$store.dispatch("getCategory");
    // 判断当前路由是否为search(this.$store可以获得)，当组件挂载完毕，让show属性变为false
    if (this.$route.path != "/home") {
      this.show = false;
    }
  },
  computed: {
    ...mapState({
      // 对象的写法，右侧需要一个函数，当使用这个计算属性时，右侧函数会立即执行一次
      // 注入一个参数state时，其实即为大仓库中的数据
      categoryList: (state) => state.home.category,
    }),
  },
  data() {
    return {
      // 存储用户鼠标移上分类
      currentIndex: -1,
      show: true,
    };
  },
  // methods别写成method不然完全失效
  methods: {
    // 鼠标移上后响应式修改currentIndex
    // 用lodash里的throttle实现节流
    focusColor: throttle(function (index) {
      //index:鼠标移上某个分类元素的索引值
      this.currentIndex = index;
    }, 50),
    loseColor() {
      this.currentIndex = -1;
      if (this.$route.path != "/home") {
        this.show = false;
      }
    },
    goSearch(event) {
      // 最好的解决方案，利用编程式导航+事件委派
      // 利用事件委派需解决问题：1. 如何确定点击的是a标签，且如何区分是哪一级的标签；2. 如何获取参数【1、2、3级分类的产品名字及id】
      // 第一个问题：用自定义属性解决  <a :data-categoryName="c1.categoryName">

      // 传入event获取触发当前事件的节点
      let element = event.target;
      // 节点的dataset属性可以获取节点的自定义属性与属性值
      let { categoryname, category1id, category2id, category3id } =
        element.dataset;
      // 如果标签身上有categoryname则一定是a标签
      if (categoryname) {
        // 整理路由跳转的参数
        let location = { name: "search" };
        let query = { categoryName: categoryname };
        if (category1id) {
          query.category1Id = category1id;
        }
        if (category2id) {
          query.category2Id = category2id;
        }
        if (category3id) {
          query.category3Id = category3id;
        }
        // 判断：如果路由跳转的时候，带有params参数，也要捎带传递过去
        if (this.$route.params) {
          location.params = this.$route.params;
          // 整理完参数（合并location和query）
          location.query = query;
          // 路由跳转
          this.$router.push(location);
        }
      }
    },
    //当鼠标移入时，让商品分类列表显示
    enterShow() {
      this.show = true;
    },
  },
};
</script>

<style scoped lang="less">
.type-nav {
  border-bottom: 2px solid #e1251b;

  .container {
    width: 1200px;
    margin: 0 auto;
    display: flex;
    position: relative;

    .all {
      width: 210px;
      height: 45px;
      background-color: #e1251b;
      line-height: 45px;
      text-align: center;
      color: #fff;
      font-size: 14px;
      font-weight: bold;
    }

    .nav {
      a {
        height: 45px;
        margin: 0 22px;
        line-height: 45px;
        font-size: 16px;
        color: #333;
      }
    }

    .sort {
      position: absolute;
      left: 0;
      top: 45px;
      width: 210px;
      height: 461px;
      position: absolute;
      background: #fafafa;
      z-index: 999;

      .all-sort-list2 {
        .item {
          h3 {
            line-height: 30px;
            font-size: 14px;
            font-weight: 400;
            overflow: hidden;
            padding: 0 20px;
            margin: 0;

            a {
              color: #333;
            }
          }

          .item-list {
            display: none;
            position: absolute;
            width: 734px;
            min-height: 460px;
            background: #f7f7f7;
            left: 210px;
            border: 1px solid #ddd;
            top: 0;
            z-index: 9999 !important;

            .subitem {
              float: left;
              width: 650px;
              padding: 0 4px 0 8px;

              dl {
                border-top: 1px solid #eee;
                padding: 6px 0;
                overflow: hidden;
                zoom: 1;

                &.fore {
                  border-top: 0;
                }

                dt {
                  float: left;
                  width: 54px;
                  line-height: 22px;
                  text-align: right;
                  padding: 3px 6px 0 0;
                  font-weight: 700;
                }

                dd {
                  float: left;
                  width: 415px;
                  padding: 3px 0 0;
                  overflow: hidden;

                  em {
                    float: left;
                    height: 14px;
                    line-height: 14px;
                    padding: 0 8px;
                    margin-top: 5px;
                    border-left: 1px solid #ccc;
                  }
                }
              }
            }
          }

          &:hover {
            .item-list {
              display: block;
            }
          }
        }
        .cur {
          background: pink;
        }
      }
    }
    // 过渡动画样式
    // 过渡动画开始状态（进入）
    .sort-enter {
      height: 0px;
    }
    // 过渡动画结束状态（进入）
    .sort-enter-to {
      height: 461px;
    }
    .sort-enter-active {
      transition: all 0.5s linear;
    }
  }
}
</style>
