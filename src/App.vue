<template>
  <div>
    <div id="posts-container" class="posts-container">
      <div v-for="post in posts" :key="post.id" class="post">
        <h2>Tytuł: {{ post.title }}</h2>
        <p>Autor: {{ getAuthorName(post.userId) }}</p>
        <p v-bind:class="{ 'show-full': post.showFull }">
          <span class="visible-text">Treść tekstu: {{ getHalfText(post.body) }}</span>
          <span class="hidden-text">Treść tekstu: {{ post.body }}</span>
        </p>
        <button @click="deletePost(post.id)">Usuń newsa</button>
        <span class="read-more" @click="toggleReadMore(post)">
          {{ post.showFull ? 'Zobacz mniej' : 'Zobacz więcej' }}
        </span>
      </div>
    </div>
    <div id="pagination-container" class="pagination">
      <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">
        Poprzednia
      </button>
      <button v-for="page in totalPages" :key="page" @click="changePage(page)">{{ page }}</button>
      <button
        @click="changePage(currentPage + 1)"
        :disabled="currentPage === totalPages"
      >
        Następna
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
  return {
    posts: [],
    users: [], 
    totalPages: 0,
    postsPerPage: 10,
    currentPage: 1,
    totalPosts: 100,
  };
},
  methods: {
    async fetchData() {
      try {
        const response = await fetch(
          `https://jsonplaceholder.typicode.com/posts?_page=${this.currentPage}&_limit=${this.postsPerPage}`
        );
        const userData = await fetch(
          'https://jsonplaceholder.typicode.com/users'
        );

        if (!response.ok || !userData.ok) {
          throw new Error('Could not fetch resource');
        }

        const postData = await response.json();
        this.users = await userData.json();
        this.displayPosts(postData, this.users);
        this.setupPagination();
      } catch (error) {
        console.error(error);
      }
    },
    displayPosts(posts, users) {
      this.posts = posts.map((post) => {
        const user = users.find((u) => u.id === post.userId);
        return {
          ...post,
          authorName: user ? user.name : 'Unknown',
          showFull: false,
        };
      });
    },
    setupPagination() {
      this.totalPages = Math.ceil(
        this.totalPosts / this.postsPerPage
      );
    },
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalPages) {
        this.currentPage = newPage;
        this.fetchData();
      }
    },
    deletePost(postId) {
      const postIndex = this.posts.findIndex(
        (post) => post.id === postId
      );
      if (postIndex !== -1) {
        this.posts.splice(postIndex, 1);
        this.totalPosts--; 
        console.log(`Usunięto post o ID: ${postId}`);
        this.setupPagination(); 
      } else {
        console.log(`Nie można odnaleźć posta o ID: ${postId}`);
      }
    },
    getAuthorName(userId) {
      const user = this.users.find(user => user.id === userId);
      return user ? user.name : 'Unknown'; 
    },
    getHalfText(text) {
      return text.substring(0, text.length / 2);
    },
    toggleReadMore(post) {
      post.showFull = !post.showFull;
    },
  },
  watch: {
    currentPage() {
      this.fetchData();
    },
  },
  mounted() {
    this.fetchData();
  },
};
</script>

<style>
@import 'components/style.css';
</style>