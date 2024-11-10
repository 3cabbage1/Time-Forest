<!-- <script setup>
import { ref } from 'vue';

const fileInput = ref(null);
const list = ref([]);

const upload = (e) => {
  const files = e.target.files;
  for (const item of files) {
    if (!/image\/\w+/.test(item.type)) {
      alert("只能选择图片");
      return;
    }
    const reader = new FileReader();
    reader.readAsDataURL(item);
    reader.onload = () => {
      list.value.push(reader.result);
    };
  }
};

const delect = (index) => {
  list.value.splice(index, 1);
};

// const noDelect = () => {
//   alert('默认图片无法删除。');
// };
</script> -->

<!-- <template>
  <div id="app">
    <div class="upload">
      <input type="file" id="file" multiple ref="fileInput" @change="upload" />
    </div>
    <ul class="view">
      <li>
        <img src="./52.jpg" />
        <div class="delect" title="删不了我" @click="noDelect">×</div>
      </li>
      <li v-for="(item, index) in list" :key="index">
        <img :src="item" />
        <div class="delect" @click="delect(index)">×</div>
      </li>
    </ul>
  </div>
</template> -->

<!-- <style scoped>
*{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
#app{
    width: 900px;
    background-color: rgb(241, 241, 241);
    margin: 50px auto;
}
.view{
   display: flex;
   justify-content: space-around;
   flex-wrap: wrap;
   align-items: space-around;
}
.view > li{
    width: 200px;
    height: 120px;
    background-color: rgba(54, 194, 35,0.1);
    list-style: none;
    margin: 20px;
    position: relative;
}
.view > li > img{
    width: 100%;
    height: 100%;
    object-fit: cover;
}
.delect{
    position: absolute;
    right: 0;
    top: 0;
     width: 20px;
     height: 20px;
     text-align: center;
     line-height: 20px;
     font-size: 15px;
     background-color: rgba(255, 255, 255,0.5);
     user-select: none;
     cursor: pointer;
     opacity: 0;
}
.delect:hover{

    background-color: rgba(31, 31, 31, 0.5);
     color: white;
}
.view>li:hover .delect{
    opacity: 1;
}
.upload{
    width: 80px;
    height: 20px;
    background-color: rgba(135, 206, 235,0.2);
    border: 1px dashed black;
    border-radius: 5px;
    position: relative;

}
.upload:hover{
    background-color: rgba(135, 206, 235,1);
}
.upload::before{
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    content: '上传图片';
    font-size: 13px;
    text-align: center;
    font-family: 'fangsong';
    line-height: 20px;
    user-select: none;
}
#file{
    width: 100%;
    height: 100%;
    opacity: 0;
}
</style> -->
<script setup>
import { ref } from 'vue';

const fileInput = ref(null);
const list = ref([]);

const upload = (e) => {
  const file = e.target.files[0]; // 只取第一张图片
  if (file &&!/image\/\w+/.test(file.type)) {
    alert("只能选择图片");
    return;
  }
  if (file) {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => {
      list.value.push(reader.result);
    };
  }
};

const delect = (index) => {
  list.value.splice(index, 1);
};
</script>

<template>
  <div id="app">
    <div class="upload">

      <input type="file" id="file" ref="fileInput" @change="upload" />
      <div id="in_upload" >
      <!-- <img src="/upload.png" alt="upload" style="width: 130px; height:auto;" > -->
      <h1>
        点击上传
      </h1>
      <h2>（一次只能上传一张图片）</h2>
    </div>
    </div>
    <ul class="view">
      <li v-for="(item, index) in list" :key="index">
        <img :src="item" />
        <div class="delect" @click="delect(index)">
          <img src="/delete.png" alt="X" style="width: 12px; height:auto;">
        </div>

      </li>
    </ul>
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  width: 900px;
  height: 600px;
}

.view {
  position: relative;
  top:-370px;
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  align-items: space-around;
}

.view>li {
  position: relative;
  max-width:800px;
  max-height: 370px;
  background-color: rgba(54, 194, 35, 0.1);
  list-style: none;

}

.view>li>img {
  width: 800px;
  height: 100%;
  object-fit: cover;
}

.delect {
  position: absolute;
  right: 0;
  top: 0;
  width: 20px;
  height: 20px;
  text-align: center;
  line-height: 20px;
  font-size: 15px;
  user-select: none;
  cursor: pointer;
  background-color: rgba(134, 135, 138, 0.5);
  border-radius: 15px;

}

.delect:hover {
  width: 21px;
  height: 21px;
  background-color: rgba(111, 107, 107, 0.5);

}

.view>li:hover.delect {
  opacity: 1;
}

.upload {
  width: 800px;
  height: 370px;
  position: relative;
  margin: auto;
  margin-top: 50px;
  background-color: rgba(227, 238, 242, 0.2);
  border: 3px dashed rgba(100, 109, 113, 0.2);
  border-radius: 5px;
  text-align: center;
  background-image: url("/upload.png");
  background-repeat: no-repeat;
  background-position: center 98px;
  background-size: 20%;
}

.upload:hover {
  border: 3px dashed  rgb(74, 153, 227);
  background-color: rgba(241, 244, 246, 0.2);
}

.upload::before {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  user-select: none;

}
#file {
  width: 100%;
  height: 100%;
  opacity: 0;
}
#in_upload{
  position: relative;
  top:-250px;
}
h1{
  font-size: 17px;
  font-weight: unset;
  margin-top: 100px;
}
h2{
  font-size: 11px;
  font-weight: unset;
  color: gray;
  margin-top: 5px;
}
/* 当屏幕宽度小于等于 768px（常见手机屏幕尺寸范围）时 */
@media (max-width: 768px) {
  #app{
    height: auto;
  }
  .upload {
  position: relative;
  top:10px;
  width:90%;
  margin: auto ;
  margin-top: 10px;
  margin-bottom: 10px;
  }
  .view{
    top:-380px;
    height:0px;
    padding: 10px;
  }
  .view>li{
    width: 100%;
    height: 370px;
  }
  .view>li>img{
    width:100%;
    max-height: 100%;
  }

}
</style>
