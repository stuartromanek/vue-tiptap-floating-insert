<template>
  <div>
    <div>
      <!-- <input type="checkbox" :checked="isEditable" @change="() => isEditable = !isEditable">
      Editable -->
    </div>
    <floating-menu :editor="editor" :tippy-options="{ duration: 100 }" v-if="editor">
      <button @click="editor.chain().focus().toggleHeading({ level: 1 }).run()" :class="{ 'is-active': editor.isActive('heading', { level: 1 }) }">
        H1
      </button>
      <button @click="editor.chain().focus().toggleHeading({ level: 2 }).run()" :class="{ 'is-active': editor.isActive('heading', { level: 2 }) }">
        H2
      </button>
      <button @click="editor.chain().focus().toggleBulletList().run()" :class="{ 'is-active': editor.isActive('bulletList') }">
        Bullet List
      </button>
    </floating-menu>
    <editor-content :editor="editor" class="editor" />
  </div>
</template>

<script>
import StarterKit from '@tiptap/starter-kit';
import { Editor, EditorContent, FloatingMenu } from '@tiptap/vue-3';
import Placeholder from '@tiptap/extension-placeholder';

// Subclass the Placeholder extension for our / instructions
const CustomPlaceholder = Placeholder.extend({
  emptyNodeClass: 'slash-instructions',
  placeholder: () => {
    return 'Type / to insert new elements';
  }
});


// This is not how providing the showShould function is demonstrated in the docs
// but works for this demo
// see https://tiptap.dev/api/extensions/floating-menu
FloatingMenu.props.shouldShow.default = ({ state }) => {
  const { $to } = state.selection;
  if (state.selection.empty && $to.nodeBefore && $to.nodeBefore.text) {
    const text = $to.nodeBefore.text;
    // Only show when the user has just entered a '/' character
    if (text.charAt(text.length - 1) === '/') {
      return true;
    }
  }
  return false;
};

export default {
  components: {
    EditorContent,
    FloatingMenu,
  },

  data() {
    return {
      editor: null,
      isEditable: true,
    };
  },

  watch: {
    isEditable(value) {
      this.editor.setEditable(value);
    },
  },

  mounted() {
    this.editor = new Editor({
      extensions: [
        StarterKit,
        FloatingMenu,
        CustomPlaceholder.configure({
          emptyNodeClass: 'slash-instructions',
          placeholder: () => {
            return 'Type / to insert new elements';
          },
        }),
      ],
      content: `
        <p>
          This is an example of a Medium-like editor. Enter a new line and some buttons will appear.
        </p>
        <p></p>
      `,
    });
  },

  beforeUnmount() {
    this.editor.destroy();
  },
};
</script>

<style lang="scss">
/* Basic editor styles */
.ProseMirror .slash-instructions::after {
  display: block;
  border-top: 1px solid #e7e9ea;
  margin-top: 5px;
  padding-top: 5px;
  color: #adb5bd;
  content: attr(data-placeholder);
  pointer-events: none;
}

.editor {
  max-width: none;
  width: 100%;
}

.ProseMirror {
  > * + * {
    margin-top: 0.75em;
  }

  ul,
  ol {
    padding: 0 1rem;
  }
}
</style>
