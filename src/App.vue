<template>
  <div class="container">
    <h1 class="title">Simple Article Manager</h1>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <h2>{{ article.title }}</h2>
          <p>{{ article.content }}</p>
        </div>
        <div class="buttons">
          <button @click="edit(article)" class="button edit-button">Edit</button>
          <button @click="deleteArticle(article.id)" class="button delete-button">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';

        const response = await axios({
          method,
          url,
          data: {
            title: form.title,
            content: form.content,
          },
        });

        if (!form.id) {
          // Add new article
          articles.value.push(response.data);
        } else {
          // Update existing article
          const index = articles.value.findIndex(article => article.id === form.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(articleId) {
      try {
        await axios.delete(`http://localhost:3000/articles/${articleId}`);
        articles.value = articles.value.filter(article => article.id !== articleId);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Times New Roman', Times, serif;
  background-color: #ffffff;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 28px;
  margin-bottom: 20px;
  color: #000;
}

.form {
  margin-bottom: 20px;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.input, .textarea {
  width: 100%;
  padding: 12px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  box-sizing: border-box;
  font-size: 16px;
  font-family: 'Times New Roman', Times, serif;
  background-color: #ffffff;
  color: #000;
}

.button {
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  border: none;
  font-size: 16px;
  transition: background-color 0.3s, opacity 0.3s;
  color: #fff;
}

.button:hover {
  opacity: 0.9;
}

.save-button {
  background-color: #28a745;
}

.load-button {
  background-color: #17a2b8;
  margin-top: 10px;
}

.article-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.article-item {
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 10px;
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #ffffff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.article-content {
  flex: 1;
  padding-right: 20px;
}

.article-content h2 {
  margin: 0 0 10px;
  font-size: 22px;
  color: #000;
}

.article-content p {
  margin: 0;
  color: #000;
}

.buttons {
  display: flex;
  flex-direction: column;
}

.edit-button {
  background-color: #007bff;
  margin-bottom: 5px;
}

.delete-button {
  background-color: #dc3545;
}
</style>
