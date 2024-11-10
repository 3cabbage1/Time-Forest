<script setup lang="ts">
// import { RouterLink, RouterView } from 'vue-router'
// import HelloWorld from './components/HelloWorld.vue'
import PostPhoto from './components/PostPhoto.vue'
import { ref } from 'vue';
// 定义风格选项，这里可以根据实际情况修改
const styleOptions = [
{ label: '默认', value: 'normal' },
  { label: '简约', value: 'simple' },
  { label: '复古', value: 'vintage' },
  { label: '现代', value: 'modern' },
];

const selectedStyle = ref(styleOptions[0].value);
</script>

<template>
  <header>
<h1>时光森林 Time Forest</h1>
  </header>
  <div id="all">
  <div id="post_photo" class="block">
  <PostPhoto />

  </div>
  <div  class="block">

    <div id="choose">
      <label id="tip_time">请选择时间日期</label>
      <input id="time" class="input" type="datetime-local" placeholder="请选择时间日期" >
    <input
    id="address" class="input" type="text" placeholder="请输入地址"/>
    <div id="style">
      <div class="style-selector">
    <label v-for="(styleOption, index) in styleOptions" :key="index">
      <input
        type="radio"
        :value="styleOption.value"
        v-model="selectedStyle"
      />
      {{ styleOption.label }}
    </label>
  </div>
    </div>
  </div>
  </div>
  </div>
  <!-- <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView /> -->

</template>

<style scoped>

/* rgb(56, 97, 173) */
header {
  height: 55px;
  background-color: rgb(56, 97, 173);
  line-height: 3.4rem;
  border-radius: 10px;
}
h1{
  color:aliceblue;
  font-size: 1.35rem;
  padding-left: 55px;
  font-family:'DancingScript','清松','Times New Roman', Times, serif;
}
.block{
  background-color: white;
  border-radius: 10px;
}
#all{
  display: grid;
  grid-template-columns: 3fr 1fr;
  gap:20px;
  margin-top: 15px;
}
#post_photo{
  display: flex;
  justify-content: center;
}
#choose{
  margin-top: 50px;
  height: 250px;
  display: flex;
  flex-direction: column;
  text-align: center;
  justify-content: space-around;
  align-items: center;
}
.input{
  width: 155px;
  height: 40px;
  border-radius: 10px;
  border: 1.5px solid rgb(211, 220, 224);
  font-family:'DancingScript','清松','Times New Roman', Times, serif;

}

.input:focus{
  outline: none;
  border: 1.5px solid rgb(159, 173, 179);
}
#tip_time{
  /* opacity: 0; */
  font-family:'DancingScript','清松','Times New Roman', Times, serif;
  color: gray;

}
#address{
  background-image: url("/address.jpg");
  background-size: 10px;
  background-repeat: no-repeat;
  background-position: 140px;
}
.style-selector {
  display: flex;
  font-family:'DancingScript','清松','Times New Roman', Times, serif;
  gap: 5px;
}
input[type="radio"] {
  margin-right: 5px;
}
/* header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
} */
/* 当屏幕宽度小于等于 768px（常见手机屏幕尺寸范围）时 */
@media (max-width: 768px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }
  #all{
  display: flex;
  flex-direction: column;
  }


}

</style>
