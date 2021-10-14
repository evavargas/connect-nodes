<template>
  <label class="text-reader">
    <input type="file" @change="loadTextFromFile" />
  </label>
</template>

<script>
export default {
  data: function () {
    return {
      json: "",
    };
  },
  methods: {
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        {
          this.$emit("load", e.target.result);
          this.json = JSON.parse(e.target.result);
           console.log(this.json);
        }
      };

      reader.readAsText(file);
    },
  },
};
</script>
<style>
.text-reader input {
  /* Fancy button style 😎 */
  border: 2px solid black;
  border-radius: 5px;
  padding: 8px 8px;
  cursor: pointer;
}
</style>