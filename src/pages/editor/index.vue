<template>
  <!-- <div> -->
  <el-container direction="“horizontal”">
    <!-- 侧边栏菜单 -->
    <el-menu
      default-active="1-4-1"
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose"
      :collapse="isCollapse"
    >
      <el-submenu index="1">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span slot="title">新建笔记</span>
        </template>
      </el-submenu>
      <el-submenu index="2">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span slot="title">我的笔记</span>
        </template>
      </el-submenu>
      <el-submenu index="3">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span slot="title">xx笔记</span>
        </template>
      </el-submenu>
      <el-submenu index="4">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span slot="title">xx笔记</span>
        </template>
      </el-submenu>
      <el-button @click="displayClick">+</el-button>
    </el-menu>

    <!-- 中部显示区域 -->
    <el-col :span="5">
      <p class="midle-title">显示区域xxx</p>
      <div
        class="title-element"
        v-for="item in article"
        @click="titleClick(item)"
        :key="item.id"
      >
        {{ item.title }}
      </div>
    </el-col>

    <!-- 右侧编辑区域 -->
    <el-col justify="start">
      <div class="head-input">
        <strong style="margin-right: 10px">标题:</strong>
        <br />
        <el-input style="width: 300px" v-model="title"></el-input>
        <el-button @click="saveNote">保存</el-button>
        <el-button @click="syncNote">同步</el-button>
      </div>
      <div class="head-input">
        <el-tag
          :key="tag"
          v-for="tag in dynamicTags"
          closable
          :disable-transitions="false"
          @close="handleTagClose(tag)"
        >
          {{ tag }}
        </el-tag>
        <el-input
          class="input-new-tag"
          v-if="inputVisible"
          v-model="inputValue"
          ref="saveTagInput"
          size="small"
          @keyup.enter.native="handleInputConfirm"
          @blur="handleInputConfirm"
        >
        </el-input>
        <el-button v-else class="button-new-tag" size="small" @click="showInput"
          >+ New Tag</el-button
        >
      </div>
      <div id="myEdit"></div>
    </el-col>
  </el-container>
  <!-- </div> -->
</template>

<script>
import wangEditor from "wangeditor";
import axios from 'axios';
export default {
  data() {
    return {
      instance: null,
      editor: null,
      isCollapse: false,
      text: "",
      title: "",
      article: [],
      dynamicTags: [],
      inputVisible: false,
      inputValue: "",
    };
  },
  mounted() {
    this.instance = axios.create({
      baseURL: '/',
      timeout: 1000,
      headers: {'authorization': 'Bearer ' + sessionStorage.getItem('token')}
    });
    // const E = window.wangEditor
    // this.editor = new E('#myEdit')
    // 或者 const editor = new E( document.getElementById('div1') )
    this.editor = new wangEditor(`#myEdit`);
    this.editor.create();
    //设置文本初始对齐方式为左对齐
    this.editor.$textElem.css("text-align", "left");
    this.article = JSON.parse(localStorage.getItem("article")) || [];
  },
  methods: {
    getText() {
      // to do test
    },
    saveNote(json) {
      this.text = this.editor.txt.text();
      // console.log(this.editor.txt.text());
      if (this.title !== null && this.title !== "") {
        // alert("not empty")
        // 重复key直接覆盖
        // console.log(this.dynamicTags)
        const len = this.article.length;
        // console.log();
        for (let i = 0; i < len; i++) {
          const sub = this.article[i];
          if (sub.title === this.title) {
            this.article.splice(i, 1);
          }
        }
        // 重复key不能添加
        // const arr = this.article.filter(el => el.title === this.title);
        // if (arr.length > 0) alert('key duplicated!');
        this.article.push({
          title: this.title,
          content: this.text,
          tag: this.dynamicTags[0]
        });
      }
      localStorage.setItem("params", JSON.stringify(this.article));
      // this.editor.txt.setJSON(json);
    },
    titleClick(v) {
      this.editor.txt.html(v.text);
      this.title = v.title;
    },
    // 展开或关闭菜单栏
    displayClick() {
      this.isCollapse = !this.isCollapse;
      // alert(this.isCollapse)
    },
    handleOpen(v) {
      console.log(v);
    },
    handleClose(v) {
      console.log(v);
    },
    // showInput() {
    //   this.inputVisible = true;
    //   this.$nextTick((_) => {
    //     this.$refs.saveTagInput.$refs.input.focus();
    //   });
    // },
    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        this.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },
    //  Tag标签事件
    handleTagClose(tag) {
      this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1);
    },
    showInput() {
      this.inputVisible = true;
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        this.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },
    // 同步文章
    syncNote() {
      this.instance && this.instance.post('/api/loginJSON',{
        params: this.article
      })
      .then(function (response) {
          console.log(response)
      })
    },
  },
};
</script>

<style scoped>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
.head-input {
  text-align: left;
  padding: 5px 0;
}
.title-element {
  cursor: pointer;
  /* background: rgb(239, 245, 250); */
  border-bottom: 1px solid #e8e8e8;
  padding-top: 8px;
  padding-bottom: 8px;
  text-align: left;
  color: #242b21;
  padding-left: 10px;
  font-size: 13px;
  border-bottom: solid 1px #dcdfe6;
}
.midle-title {
  margin: 30px;
}
</style>