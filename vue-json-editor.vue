<template>
  <div>
    <div class="jsoneditor-vue"></div>
    <div class="jsoneditor-btns" v-if="showBtns!==false">
      <button
        class="json-save-btn"
        type="button"
        @click="onSave()"
        :disabled="error"
      >{{ locale[lang].save }}</button>
      <button
        class="json-remove-btn"
        type="button"
        @click="onRemove()"
      >{{ locale[lang].remove }}</button>
    </div>
  </div>
</template>

<script>
import "./assets/jsoneditor.css";
import JsonEditor from "./assets/jsoneditor";
export default {
  // props: ['value', 'showBtns', 'mode', 'modes', 'lang'],
  props: {
    value: [String, Number, Object, Array],
    showBtns: [Boolean],
    expandedOnStart: {
      type: Boolean,
      default: false
    },
    mode: {
      type: String,
      default: "tree"
    },
    modes: {
      type: Array,
      default: function() {
        return ["tree", "code", "form", "text", "view"];
      }
    },
    lang: {
      type: String,
      default: "en"
    }
  },
  watch: {
    value: {
      immediate: true,
      async handler(val) {
        if (!this.internalChange) {
          await this.setEditor(val);

          this.error = false;
          this.expandAll();
        }
      },
      deep: true
    }
  },
  data() {
    return {
      editor: null,
      error: false,
      json: this.value,
      internalChange: false,
      expandedModes: ["tree", "view", "form"],
      locale: {
        it: {
          save: "SALVA"
        },
        en: {
          save: "Save",
          remove: "Delete",
        },
        zh: {
          save: "保存"
        }
      }
    };
  },
  mounted() {
    let self = this;

    let options = {
      mode: this.mode,
      modes: this.modes, // allowed modes
      onChange() {
        try {
          let json = self.editor.get();
          self.json = json;
          self.error = false;
          self.$emit("json-change", json);
          self.internalChange = true;
          self.$emit("input", json);
          self.$nextTick(function() {
            self.internalChange = false;
          });
        } catch (e) {
          self.error = true;
          self.$emit("has-error", e);
        }
      },
      onModeChange() {
        self.expandAll();
      }
    };

    const editor = this.$el.querySelector(".jsoneditor-vue");

    if (editor.children.length > 0) {
      editor.firstChild.remove();
    }

    this.editor = new JsonEditor(
      editor,
      options,
      this.json
    );
  },
  methods: {
    expandAll() {
      if (
        this.expandedOnStart &&
        this.expandedModes.includes(this.editor.getMode())
      ) {
        this.editor.expandAll();
      }
    },

    onSave() {
      this.$emit("json-save", this.json);
    },

    onRemove() {
      this.$emit('json-remove', this.json)
    },

    async setEditor(value) {
      if (this.editor) this.editor.set(value);
    }
  }
};
</script>

<style scoped>
  .ace_line_group {
    text-align: left;
  }
  .json-editor-container {
    display: flex;
    width: 100%;
  }
  .json-editor-container .tree-mode {
    width: 50%;
  }
  .json-editor-container .code-mode {
    flex-grow: 1;
  }
  .jsoneditor-btns{
    text-align: center;
    margin-top:10px;
  }
  .jsoneditor-vue .jsoneditor-outer{
    min-height:150px;
  }
  .jsoneditor-vue div.jsoneditor-tree{
    min-height: 350px;
  }
  .json-save-btn{
    background-color: #20A0FF;
    border: none;
    color:#fff;
    padding:5px 10px;
    border-radius: 5px;
    cursor: pointer;
    margin-right: 5px;
  }
  .json-save-btn:focus{
    outline: none;
  }
  .json-save-btn[disabled]{
    background-color: #1D8CE0;
    cursor: not-allowed;
  }
  code {
    background-color: #f5f5f5;
  }
</style>
