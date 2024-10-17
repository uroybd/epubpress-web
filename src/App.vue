<script setup>
import { ref } from 'vue';
import { useForm } from 'vee-validate';
import * as yup from 'yup';
import EpubPress from 'epub-press-js';

const { meta, defineField, handleSubmit, errors } = useForm({
    initialValues: {
      title: 'Created with EpubPress',
      description: 'Created with EpubPress',
      urls: ""
    },
    validationSchema: yup.object({
      title: yup.string().required().default('Created with EpubPress'),
      description: yup.string().required().default('Created with EpubPress'),
      urls: yup.string().required('URLs are required.').test({
      name: 'valid-urls',
      test: (value, ctx) => {
        for (const url of value.split('\n')) {
          if (!url.startsWith('http://') && !url.startsWith('https://')) {
            return ctx.createError({
              message: 'All URLs must start with http:// or https://',
            })
          }
        }
        return true
      }
    })
    })
});

const [title, titleAttrs] = defineField('title');
const [description, descriptionAttrs] = defineField('description');
const [urls, urlsAttrs] = defineField('urls');
const generating = ref(false);

const onSubmit = handleSubmit(values => {
   generating.value = true;
   const ebook = new EpubPress({
     title: values.title,
     description: values.description,
     urls : values.urls.split('\n')
   });
   ebook.publish().then(() => {
      ebook.download();  // Default epub
      generating.value = false;
  }).then(() => {
      generating.value = false;
  }).catch((error) => {
      generating.value = false;
  });
});
</script>

<template>
   <nav>
    <div class="left">EpubPress on the Web</div>
    <div class="right">
      <a href="https://github.com/uroybd/epubpress-web" target="_blank" rel="noopener">GitHub</a>
      <a href="https://epub.press/" target="_blank" rel="noopener">EpubPress</a>
    </div>
  </nav>

  <main>
    <form @submit="onSubmit">
      <div class="input-group">
        <label for="title">Title</label>
        <input type="text" id="title" name="title" required v-model="title" v-bind="titleAttrs" />
        <div v-if="errors.title" class="error">{{ errors.title }}</div>
      </div>
      <div class="input-group">
        <label for="description">Description</label>
        <input type="text" id="description" v-model="description" name="description" v-bind="descriptionAttrs" required />
        <div v-if="errors.description" class="error">{{ errors.description }}</div>
      </div>
      <div class="input-group">
        <label for="urls">URLs</label>
        <textarea v-model="urls" id="urls" name="urls" required placeholder="Put URLs, one per line" v-bind="urlsAttrs" />
        <div v-if="errors.urls" class="error">{{ errors.urls }}</div>
      </div>
      <div class="buttons">
      <button :disabled="!meta.valid || generating" :class="{'generating': generating}" type="submit">{{ generating ? 'Generating...' : 'Generate' }}</button>
      </div>
    </form>
  </main>
</template>


<style lang="scss">
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

#app {
  padding: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 100vw;
  min-height: 100vh;
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #ccc;
  width: 100%;
}

main {
  width: 100%;
  flex: 1;
  display: flex;
  justify-content: center;

  form {
    display: flex;
    flex-direction: column;
    gap: 10px;
    width: 100%;
    max-width: 800px;
    margin-top: 20px;

    .input-group {
      display: flex;
      flex-direction: column;
      gap: 5px;

      label {
        font-weight: 500;
      }

      input {
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
      }

      textarea {
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
      }

      .error {
        color: red;
        padding: 10px;
      }
    }

    .buttons {
      display: flex;
      justify-content: center;
      button {
        padding: 10px;
        background-color: #007bff;
        color: #fff;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-size: 1.2rem;

        &:disabled {
          opacity: 0.5;
          cursor: not-allowed;
        }
      }

      button.generating {
        opacity: 0.5;
        cursor: not-allowed;
        background-color: hsla(160, 100%, 37%, 1);

      }

    }
  }
}

</style>
