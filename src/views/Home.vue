<template>
  <v-container>
    <v-row>
      <v-col cols="6">
        <div style="max-width: 30rem">
          <h1>CSV File</h1>
          <v-file-input
            id="csv"
            show-size
            label="Select CSV file"
          ></v-file-input>
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
        ><p>{{ replacedText }}</p></v-col
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
  }),
  methods: {
    addRule() {
      this.regexStrings.push({
        regex: "",
        replace: "",
      });
    },
    replaceText() {
      let fileInput = document.getElementById("csv");
      let reader = new FileReader();

      reader.onload = () => {
        let regexs = [];
        this.regexStrings.forEach((regexString) => {
          regexs.push(new RegExp(regexString.regex, "gim"));

          let replacedText = reader.result;

          regexs.forEach((regex, index) => {
            replacedText = replacedText.replace(
              regex,
              this.regexStrings[index].replace
            );
          });

          this.download("hello.csv", replacedText);
        });
      };

      reader.readAsBinaryString(fileInput.files[0]);
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
  },
};
</script>
