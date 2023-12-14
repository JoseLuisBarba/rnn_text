<template>
  <div class="google-search">
    <div tabindex="1" @focus="setCaret" class="autocomplete-container">
      <span @input="sendText" @keypress="preventInput" ref="editbar" class="editable" contenteditable="true"></span>
      <span class="placeholder" contenteditable="false">{{autoComplete}}</span>    
    </div>
  </div>
</template>

<script>
export default {
  name: 'Autocomplete',
  data() {
    return {
      autoComplete: "",
      maxChars: 75,
      connection: null
    }
  },
  mounted() {
    const url = "ws://localhost:8000/";
    this.connection = new WebSocket(url);
    this.connection.onopen = () => console.log("good connection");
    this.connection.onmessage = this.receiveText;
  },
  methods: {
    setCaret() {
      const range = document.createRange();
      const sel = window.getSelection();
      const parentNode = this.$refs.editbar;

      if (parentNode.firstChild == undefined) {
        const emptyNode = document.createTextNode("");
        parentNode.appendChild(emptyNode);
      }

      range.setStartAfter(this.$refs.editbar.firstChild);
      range.collapse(true);
      sel.removeAllRanges();
      sel.addRange(range);
    },
    preventInput(event) {
      let prevent = false;      

      if (event.key == event.key.toUpperCase()) {
        prevent = true;
      }

      if (event.code == "Space") {
        prevent = false;
      }

      // handle input overflow
      const nChars = this.$refs.editbar.textContent.length;
      if (nChars >= this.maxChars) {
        prevent = true;
      }

      if (prevent) {
        event.preventDefault();
      }
    },
    sendText() {
      const inputText = this.$refs.editbar.textContent;
      this.connection.send(inputText);
    },
    receiveText(event) {
      this.autoComplete = event.data;
    }
  }
}
</script>

<style scoped>
.google-search {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.autocomplete-container {
  font-family: Arial, sans-serif;
  font-size: 16px;
  border-radius: 24px;
  height: 48px;
  width: 600px;
  display: flex;
  align-items: center;
  background-color: #f2f2f2;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}

.editable {
  flex: 1;
  padding: 12px;
  outline: none;
}

.placeholder {
  color: #808080;
  flex: 1;
  padding: 12px;
  pointer-events: none;
}
</style>

