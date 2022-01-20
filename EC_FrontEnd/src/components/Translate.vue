<template>
  <div class="showoff col">
    <div class="translatebox">
      <div class="inputbox">
        <textarea
          @keydown="getTranslateDebounced"
          v-model.trim="inputbox"
          placeholder="Enter or paste text here."
        >
        </textarea>
        <button @click="clearInput" v-show="isShow" class="closeBtn">
          <i class="fas fa-times"></i>
        </button>
      </div>
      <div class="exchangebox">
        <span>{{ leftLang }}</span>
        <button @click="TramslateChange">
          <i class="fas fa-exchange-alt"></i>
        </button>
        <span>{{ rightLang }}</span>
      </div>
      <div class="outputbox">
        <textarea v-model="outputbox" readonly></textarea>
      </div>
      <div class="micbox">
        <button @click="micStart" v-if="micBtn">
          <i class="fas fa-microphone micStart"></i>
        </button>
        <button v-else>
          <i @click="micStop" class="fas fa-microphone micStop"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Translate",
  data() {
    return {
      isShow: false, //顯示清除按鈕
      micBtn: true, //顯示麥克風按鈕
      leftLang: "英文", //顯示輸入語言
      rightLang: "中文", //顯示翻譯語言
      inputbox: "", //textarea欄位
      outputbox: "", //textare輸出欄位
      EngFormData: new FormData(), //sent_translate API呼叫
      recognition: new window.webkitSpeechRecognition(), //語音辨識API
      runtimeTranscription_: "", //語音辨識輸出結果
    };
  },
  created() {
    // _.debounce用意防止過多次的api請求 https://lodash.com/ 文件參考
    this.getTranslateDebounced = _.debounce(this.getTranslate, 400);
  },
  methods: {
    getTranslateDebounced() {},

    //清除輸入、輸出area
    clearInput() {
      this.inputbox = "";
      this.outputbox = "";
      this.isShow = false;
    },

    //翻譯
    getTranslate() {
      if (this.inputbox !== "") {
        this.isShow = true;
      } else {
        this.isShow = false;
      }
      this.EngFormData.set("sent_input", this.inputbox);
      this.axios
        .post("https://sels.nkfust.edu.tw/sent_translate", this.EngFormData)
        .then((response) => {
          console.log(response.data);
          if (
            response.data.translate !== "Sorry,We can not detect this language"
          ) {
            this.outputbox = response.data.translate;
          } else {
            this.outputbox = "";
          }
          if (response.data.lang === "中文") {
            this.leftLang = "中文";
            this.rightLang = "英文";
          } else if (response.data.lang === "英文") {
            this.leftLang = "英文";
            this.rightLang = "中文";
          }
        });
    },

    //轉換翻譯語言 英文<-->中文
    TramslateChange() {
      let temp = this.leftLang;
      this.leftLang = this.rightLang;
      this.rightLang = temp;
      this.inputbox = this.outputbox;
      this.getTranslate();
    },

    //麥克風開始
    micStart() {
      // initialisation of voicereco
      this.micBtn = !this.micBtn;
      this.recognition.lang = this.leftLang === "中文" ? "zh-tw" : "en-US";
      this.recognition.interimResults = true;

      // event current voice reco word
      this.recognition.addEventListener("result", (event) => {
        const text = Array.from(event.results)
          .map((result) => result[0])
          .map((result) => result.transcript)
          .join("");
        this.runtimeTranscription_ = text;
      });
      this.recognition.start();
    },

    //麥克風停止
    micStop() {
      this.recognition.stop();
      this.micBtn = !this.micBtn;
      this.isShow = true;
      this.inputbox = this.runtimeTranscription_;
      this.getTranslate();

      console.log("stop record");
    },
  },
};
</script>
