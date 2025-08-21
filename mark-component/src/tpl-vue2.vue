<template>
  <div class="mark-detail__content">
    <div class="mark-detail__content-right">
      <div class="container-border">
        <div class="step__content-container border">
          <div class="step__title">
            <span>标注选项一: 句子中的词 (可多个)</span>
            <div class="step__title-keywords" @click="showSelectKey">
              产品词
              <span class="step__title-icon">xX</span>
            </div>
          </div>
          <div ref="keywords" class="step__content" @click="selectKey">
            <span
              v-for="(item, index) in words"
              :key="index"
              :class="item.isWord ? 'keyword' : 'normal'"
              >{{ item.word
              }}<span
                v-if="item.isWord"
                class="icon-delete"
                @click.stop="deleteWords(item)"
              />
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: [], // 存放商品标题 从 0 到 length-1 截取的全部信息
      prod_words: [],
      selectWord: {}, // 当前划词的信息
      name: "我是一段话，请对进行处理",
    };
  },

  mounted() {
    this.init();
    document.body.onkeydown = this.quickSetKey;
  },
  destroyed() {
    document.body.onkeydown = null;
  },
  methods: {
    init() {
      const tempWords = this.prod_words.sort(
        (pre, curr) => pre.start_offset - curr.start_offset
      );
      this.prodWords = tempWords;
      this.words = [];
      if (tempWords.length === 0) {
        const obj = {
          end_offset: this.name.length,
          label: "8",
          prob: "1",
          start_offset: 0,
          word: this.name,
          isWord: false,
        };

        this.words.push(obj);
        return;
      }

      const total = [];
      for (let i = 0; i < tempWords.length; i++) {
        const { start_offset, end_offset } = tempWords[i];
        const { end_offset: pre_end_offset } =
          total.length > 0
            ? total[total.length - 1]
            : { start_offset: 0, end_offset: 0 };

        if (pre_end_offset !== start_offset) {
          const obj = {
            end_offset: start_offset,
            label: "8",
            prob: "1",
            start_offset: pre_end_offset,
            word: this.name.substring(pre_end_offset, start_offset),
            isWord: false,
          };
          total.push(obj);
        }
        total.push({ ...tempWords[i], isWord: true });

        if (i === tempWords.length - 1) {
          const obj = {
            end_offset: this.name.length,
            label: "8",
            prob: "1",
            start_offset: end_offset,
            word: this.name.substring(end_offset),
            isWord: false,
          };
          total.push(obj);
        }
      }
      this.words = total;
    },

    selectKey(value) {
      if (window.getSelection) {
        const range = window.getSelection().getRangeAt(0);

        const preSelectionRange = range.cloneRange();
        preSelectionRange.selectNodeContents(this.$refs.keywords);
        preSelectionRange.setEnd(range.startContainer, range.startOffset);

        const startOffset = preSelectionRange.toString().length;
        const endOffset = startOffset + range.toString().length;
        let selectWordText = range.toString();

        const existWord = this.words.find(
          (value) => value.word === selectWordText && value.isWord
        );

        if (existWord) {
          selectWordText = "";
        }

        this.selectWord = {
          end_offset: endOffset,
          label: "8",
          prob: "1",
          start_offset: startOffset,
          word: selectWordText,
          isWord: true,
        };
      }
    },
    showSelectKey() {
      if (!this.selectWord.word) {
        console.log("没有选中的内容");
        return;
      }

      const { start_offset, end_offset } = this.selectWord;
      const temp = [];
      this.prodWords.map((item) => {
        if (
          item.end_offset <= start_offset ||
          item.start_offset >= end_offset
        ) {
          temp.push(item);
        }
      });

      temp.push(this.selectWord);

      this.prod_words = [...temp];
      this.init();
      // 获取推荐词列表

      this.selectWord = {}; // 清空选中词
      // 清除选中
      window.getSelection().removeAllRanges();
    },

    // 设置快捷键
    quickSetKey(e) {
      e.preventDefault();
      // console.log('quickSetKey', e)
      // 划词选中(x)
      if (e.keyCode === 88) {
        this.showSelectKey();
      }

      // 确认更新(alt+s)
      if (e.altKey && e.keyCode === 83) {
        this.init();
      }
    },

    deleteWords(word) {
      this.prod_words.forEach((item, index) => {
        if (
          item.start_offset === word.start_offset &&
          item.end_offset === word.end_offset
        ) {
          this.prod_words.splice(index, 1);
        }
      });

      this.init();
    },
  },
};
</script>

<style lang="less" scoped>
.mark-detail__content {
  display: flex;
  justify-content: space-around;

  &-right {
    flex: 1;
    min-width: 500px;
    margin-left: 20px;
    height: 78vh;
    overflow-y: auto;
    .container-section {
      padding: 0 10px;
      margin: 15px 5px 0 0;
      .progress-title {
        display: flex;
        justify-content: space-between;
      }
      .progress {
        width: 100%;
        height: 10px;
        border: 1px solid #eee;
        border-radius: 5px;
        &-bar {
          height: 100%;
          background: #409eff;
          border-radius: 5px;
          border: 1px solid #409eff;
        }
      }

      .title-li {
        font-size: 16px;
        margin-right: 5px;
        &__red {
          color: #da7ea4;
        }
        &__label {
          font-size: 18px;
          color: #464343;
          font-weight: bold;
        }
      }
      li {
        line-height: 2;
      }
      .words-1 {
        color: #ba034f;
        font-size: 18px;
      }
      .words-2 {
        color: #be79d3;
        margin-left: 5px;
        font-size: 18px;
      }
      .words-3 {
        color: #7c63e9;
        margin-left: 5px;
        font-size: 18px;
      }
    }

    /*滚动条整体部分,必须要设置*/
    &::-webkit-scrollbar {
      width: 5px;
      height: 5px;
      background-color: #ddd;
      border-radius: 2.5px;
    }
    &::-webkit-scrollbar-thumb {
      background-color: #aaa;
      border-radius: 2.5px;
    }
    &::-webkit-scrollbar-button {
      height: 0px;
    }
  }
}

.container-border {
  margin: 10px 10px 0 0;
  padding: 10px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  overflow: hidden;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  white-space: normal;
  .recommend-words {
    font-size: 16px;
    border-radius: 3px;
    border: 1px solid #aaa;
    padding: 2px 5px;
    margin: 2px 3px;
    // user-select: none;
    white-space: nowrap;
    display: inline-block;
    &__selected {
      border: 1px solid #7c63e9;
      background: #7c63e9;
      color: #fff;
    }
  }

  .step__content-container {
    .step__title {
      padding: 10px;
      background: #4169e1;
      color: #fff;
      &-keywords {
        padding-left: 5px;
        display: inline-block;
        width: 100px;
        line-height: 2;
        font-weight: bold;
        font-size: 14px;
        border-radius: 2px;
        text-align: center;
        background: #ba034f;
        color: #fff;
      }
      &-icon {
        background: #fff;
        width: 40px;
        float: right;
        color: #000;
        border-radius: 0 2px 2px 0;
      }
    }
    .step__content {
      padding-left: 5px;
      line-height: 2;
      margin: 5px;
      font-size: 0;
      letter-spacing: 2px;
      .normal {
        font-size: 18px;
      }
      .keyword {
        margin: 0 2px;
        padding: 5px;
        background: #ba034f;
        color: #fff;
        text-align: center;
        border-radius: 3px;
        white-space: nowrap;
        font-size: 18px;
        // user-select: none; //关键词不再选中
        .icon-delete {
          margin-left: 5px;
          display: inline-block;
          width: 16px;
          height: 18px;
          background: #9b0342;
          line-height: 18px;
          vertical-align: text-bottom;
          border-radius: 50%;
          font-size: 10px;
          &::after {
            content: "x";
            display: inline-block;
            width: 16px;
            height: 16px;
            background: transparent;
            line-height: 16px;
            vertical-align: text-bottom;
            border-radius: 50%;
            font-size: 10px;
          }
          &:hover {
            background: #740231;
            cursor: pointer;
          }
        }
      }
    }
  }
}

.container-btn {
  margin: 10px 10px 0 0;
  padding: 10px;
  text-align: right;
}

&::-webkit-scrollbar {
  width: 0;
  height: 0;
}

.border {
  border: 1px solid #eee;
}
</style>
