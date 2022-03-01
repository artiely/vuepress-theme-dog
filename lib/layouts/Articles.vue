<template>
  <div class="article-wrapper">
  <!-- FIXME: : 默认卡片都是隐藏的 -->
    <div class="article-box" v-for="item in currentPageData" :key="item.id">
      <article class="article">
        <div class="poster-wrapper" v-if="item.frontmatter">
          <a :href="link(item.link)">
            <div class="category" v-if="item.frontmatter.category">
              <span
                class="category-inner"
                :style="{
                  backgroundColor: hexToRgba(
                    `#${item.frontmatter.secondary}`,
                    0.6
                  ),
                  color: `#${item.frontmatter.primary}`,
                }"
                >{{ item.frontmatter.category }}</span
              >
            </div>
            <Cover :item="item" />
          </a>
        </div>
        <div>
          <div class="article-info" v-if="item.frontmatter">
            <h1 class="article-title">
              <a :href="link(item.link)">{{ item.frontmatter.title }}</a>
            </h1>

            <div class="article-con">
              <p class="article-desc">{{ item.frontmatter.summary }}</p>
            </div>
            <div class="article-tags" v-if="item.frontmatter.tag">
              <span class="tag" v-for="tag in item.frontmatter.tag.slice(0, 3)"
                >#{{ tag }}</span
              >
            </div>
            <div class="article-meta">
              <span class="date">
                <i class="iconfont icon-rili1" />
                {{ dayjs(item.frontmatter.date).format("YYYY/MM/DD") }}</span
              >
              <span class="author"
                ><i class="iconfont icon-zuozhe" />
                {{ item.frontmatter.author }}</span
              >
            </div>

            <a class="more" :href="link(item.link)">
              <span class="words">
                <i class="iconfont icon-tongji" />
                {{ item.frontmatter.words }} words /{{
                  item.frontmatter.readTime
                }}</span
              >
              <span class="read">阅读全文 <i class="iconfont icon-you" /></span>
            </a>
          </div>
        </div>
      </article>
    </div>
  </div>
  <div class="my-4 my-page">
    <button :disabled="isFirstPage" :class="isFirstPage?'disabled':''" @click="prev">prev</button>
    <button
      v-for="item in pageCount"
      :key="item"
      :disabled="currentPage === item"
      @click="currentPage = item"
      :class="currentPage === item?'disabled':''"
    >
      {{ item }}
    </button>
    <button :disabled="isLastPage" :class="isLastPage?'disabled':''" @click="next">next</button>
  </div>
</template>
<script setup>
import Cover from "./Cover.vue";
import { hexToRgba } from "../utils/index.js";
import dayjs from "dayjs";
import gsap from "gsap";
import { onMounted, ref,nextTick } from "vue";
import { useOffsetPagination } from "@vueuse/core";
import { useRouteQuery } from '@vueuse/router'
const posts = __POSTS__.posts;
const link = (link) => `${link}.html`;

const pageSize = 6;
const currentPageData = ref(posts.slice(0, pageSize));
const page = useRouteQuery('page') 

page.value=1
const {
  currentPage,
  currentPageSize,
  pageCount,
  isFirstPage,
  isLastPage,
  prev,
  next,
} = useOffsetPagination({
  total: posts.length,
  page: 1,
  pageSize,
  onPageChange: async() => {
    
    currentPageData.value = posts.slice(
      (currentPage.value - 1) * pageSize,
      (currentPage.value - 1) * pageSize + pageSize
    );
    window.scrollTo(0,0)
    page.value=currentPage.value
    await nextTick()
    handlerStagger();
  },
});

const handlerStagger = () => {
  var ob = new IntersectionObserver((entries, self) => {
    let targets = entries
      .map((entry) => {
        if (entry.isIntersecting) {
          self.unobserve(entry.target);
          return entry.target;
        }
      })
      .filter((v) => v);
    if (gsap) {
      gsap.to(targets, {
        opacity: 1,
        y: 0,
        stagger: 0.2,
        ease: "Back.easeOut",
      });
    }
  });
  document.querySelectorAll(".article-box").forEach((box) => {
    ob.observe(box);
  });
  import("vanilla-tilt").then((res) => {
    res.default.init(document.querySelectorAll(".article"), {
      max: 5,
      reverse: true,
    });
  });
};

onMounted(() => {
  handlerStagger();
});
</script>
<style lang="scss">
@import "../styles/var.scss";
.my-page{
  margin: auto;
  text-align: center;
}
.my-page button {
  padding: 8px 15px;
  background-color: var(--primary-bg);
  border: none;
  outline: none;
  color: var(--c-text);
  /* border-bottom: 2px solid var(--shadow); */
  box-shadow: var(--light-shadow);
  border-radius: 4px;
  font-size: 1rem;
  box-sizing: border-box;
  vertical-align: middle;
  cursor: pointer;
  margin: .5rem;
  &:hover{
    color: var(--c-brand);
  }
  &.disabled{
    opacity: .5;
  }
}
.article-wrapper {
  /* max-width: var(--nav-content-max-width); */
  --read: var(--nav-active-color);
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  margin: auto;
  .article-box {
    opacity: 0;
    transform: translateY(100px);
  }
  .article {
    display: flex;
    flex-direction: column;
    background: var(--primary-bg);
  }
  .poster-wrapper {
    height: 230px;
    width: 100%;
    overflow: hidden;
    position: relative;
  }
  .article-info {
    padding: 20px;
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: space-between;
    flex: 1;
    /* background-color: var(--primary-bg); */
  }

  .category {
    position: absolute;
    top: 0;
    right: 0;
    z-index: 3;
    .category-inner {
      backdrop-filter: saturate(180%) blur(10px);
      display: -webkit-flex;
      display: flex;
      -webkit-justify-content: center;
      justify-content: center;
      -webkit-align-items: center;
      align-items: center;
      height: 40px;
      min-width: 120px;
      border-radius: 0 5px;
      white-space: nowrap;
      padding: 0 5px;
      font-family: Montserrat-SemiBold, Arial, Helvetica, sans-serif;
      font-size: 16px;
      text-transform: capitalize;
      color: #fff;
      background-color: #ffc107;
      font-weight: bold;
      font-size: 18px;
    }
  }
  .article-title {
    width: 100%;
    font-size: 24px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    a {
      /* background: radial-gradient(
            circle at 10px -7px,
            transparent 8px,
            currentColor 8px,
            currentColor 9px,
            transparent 9px
          )
          repeat-x,
        radial-gradient(
            circle at 10px 27px,
            transparent 8px,
            currentColor 8px,
            currentColor 9px,
            transparent 9px
          )
          repeat-x;
      background-size: 20px 20px;
      background-position: -10px calc(100% + 16px), 0 calc(100% - 4px);
      animation: waveFlow 1s infinite linear; */
      position: relative;
      display: inline-block;
      height: 40px;
      max-width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
      &::before {
        content: "";
        position: absolute;
        bottom: 12px;
        left: 0;
        width: 100%;
        height: 2px;
        background: currentColor;
        z-index: -1;
      }
      &::after {
        opacity: 0;
        content: "";
        position: absolute;
        left: 0;
        width: 100%;
        height: 76%;
        background: radial-gradient(
              circle at 10px -7px,
              transparent 8px,
              currentColor 8px,
              currentColor 9px,
              transparent 9px
            )
            repeat-x,
          radial-gradient(
              circle at 10px 27px,
              transparent 8px,
              currentColor 8px,
              currentColor 9px,
              transparent 9px
            )
            repeat-x;
        background-size: 20px 20px;
        background-position: -10px calc(100% + 16px), 0 calc(100% - 4px);
        animation: waveFlow 1s infinite linear;
      }
      &:hover {
        color: var(--read);
        &::before {
          display: none;
        }
        &::after {
          opacity: 1 !important;
        }
      }
    }
  }
  @keyframes waveFlow {
    from {
      background-position-x: -10px, 0;
    }
    to {
      background-position-x: -30px, -20px;
    }
  }
  .article-con {
    flex: 1;
  }
  .article-desc {
    width: 100%;
    word-break: break-all;
    text-overflow: ellipsis;
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 4;
    -webkit-box-orient: vertical;
  }
  .article-meta {
    color: #777;
  }
  .article-meta,
  .more {
    font-family: num;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding-top: 20px;
    text-transform: Capitalize;
    .date {
      font-size: 0.9em;
    }
    .author {
      font-family: "Josefin Sans";
    }
    .words {
      font-size: 0.9em;
      color: #777;
      display: flex;
      align-items: center;
    }
    .read {
      color: var(--read);
    }
  }
  .more .read {
  }
  .article-tags {
    width: 100%;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    font-family: "Josefin Sans";
    .tag {
      padding: 2px 6px;
      background-color: rgba(0, 0, 0, 0.05);
      border-radius: 2px;
      margin-right: 10px;
      margin-bottom: 10px;
      font-size: 0.8em;
    }
  }
}
.article {
  width: 360px;
  height: 600px;
  box-shadow: 4px 2px 40px rgb(0 0 0 / 10%);
  margin: 20px;
  border-radius: 6px;
  overflow: hidden;
}
</style>
