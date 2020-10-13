<template>
  <v-container>
    <v-row>
      <v-col cols="6">
        <div style="max-width: 30rem">
          <h4>Upload CSV file or enter custom text</h4>
          <v-checkbox v-model="isCSV" :label="`CSV`"></v-checkbox>
          <v-file-input
            v-if="isCSV"
            id="csv"
            show-size
            label="Select CSV file"
          ></v-file-input>
          <v-text-field
            v-else
            v-model="normalText"
            label="Your text here..."
            @keyup.enter="replaceNormalText"
          ></v-text-field>
          <div v-for="(regexString, index) in regexStrings" :key="index">
            <h3>Rule {{ index + 1 }}</h3>
            <v-text-field
              v-model="regexString.regex"
              label="Regex"
            ></v-text-field>
            <v-text-field
              v-model="regexString.replace"
              label="Replace"
            ></v-text-field>
          </div>
          <div class="my-2">
            <v-btn :disabled="canReplaceText" @click="addRule" color="normal"
              >Add Rule</v-btn
            >
            <v-btn
              text
              :disabled="canReplaceText"
              @click="regexStrings.pop()"
              color="normal"
              class="ml-2"
              >Remove Rule</v-btn
            >
            <v-btn
              class="ml-2"
              :disabled="canReplaceText"
              @click="replaceText"
              color="primary"
              >Replace Text</v-btn
            >
            <v-btn
              v-show="isCSV"
              class="mt-2"
              :disabled="replacedText == ''"
              @click="downloadCSV"
              color="primary"
              >Download CSV</v-btn
            >
          </div>
        </div>
      </v-col>
      <v-col cols="6"
        ><h4>Result</h4>
        <p>{{ replacedText }}</p></v-col
      >
    </v-row>
    <v-row>
      <v-col></v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "Home",

  data: () => ({
    regexStrings: [
      {
        regex: `https:\/\/www.notion.so\/`,
        replace: "",
      },
      { regex: `-[a-zA-Z0-9]{32}`, replace: "" },
      { regex: `-`, replace: " " },
    ],
    canReplaceText: false,
    normalText: "",
    replacedText: "",
    isCSV: true,
  }),
  methods: {
    addRule() {
      this.regexStrings.push({
        regex: "",
        replace: "",
      });
    },
    replaceText() {
      if (this.isCSV) {
        this.replaceCSVFile();
      } else {
        this.replaceNormalText();
      }
    },
    replaceNormalText() {
      this.executeRegex(this.normalText);
    },
    replaceCSVFile() {
      let fileInput = document.getElementById("csv");
      let reader = new FileReader();
      reader.readAsBinaryString(fileInput.files[0]);

      reader.onload = () => {
        this.executeRegex(reader.result);
      };
    },
    executeRegex(textToBeReplaced) {
      this.replacedText = textToBeReplaced;
      let regexs = [];
      this.regexStrings.forEach((regexString) => {
        regexs.push(new RegExp(regexString.regex, "gim"));
      });

      regexs.forEach((regex, index) => {
        this.replacedText = this.replacedText.replace(
          regex,
          this.regexStrings[index].replace
        );
      });
    },
    download(filename, text) {
      let element = document.createElement("a");
      element.setAttribute(
        "href",
        "data:text/plain;charset=utf-8," + encodeURIComponent(text)
      );
      element.setAttribute("download", filename);

      element.style.display = "none";
      document.body.appendChild(element);

      element.click();

      document.body.removeChild(element);
    },

    downloadCSV() {
      let fileInput = document.getElementById("csv");
      let filename = fileInput.value.replace(".csv", "");
      this.download(`${fileInput.value}-${Date.now()}.csv`, this.replacedText);
    },
  },
};
</script>
