<template>
  <html>
  <head>
    <title>VoodooRocks</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
  </head>
  <body>
  <div id="app" class="app-container">
    <b-input-group class="align-self-center mt-4 mb-4" id="input-group">
      <b-input-group-prepend is-text>
        <b-icon icon="search"/>
      </b-input-group-prepend>
      <b-form-input v-model="search" type="search" placeholder="Filter by authors.."></b-form-input>
    </b-input-group>
    <b-container class="grid-container">
          <b-card class="post-card mb-3 text-start" v-for="post in filteredPosts" :key="post.id">
            <b-card-title class="text-primary" style="font-size: 1.3em">{{ post.title }}</b-card-title>
            <b-card-text>{{post.body}}</b-card-text>
            <small class="text-muted">{{post.author}}</small>
          </b-card>
    </b-container>
    <b-spinner class="align-self-center mb-4" variant="primary" v-if="loader"></b-spinner>
    <p v-if="filteredPosts && !filteredPosts.length && !error && !loader">Oops! Sorry, I don't have posts for you. You can try something else :)</p>
    <p v-if="this.error">Something went wrong. Please try again later.</p>
  </div>
  </body>
  </html>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data: function () {
    return {
      search: '',
      posts: [],
      error: false,
      loader: false,
    }
  },
  computed: {
    filteredPosts: function() {
      return this.posts?.filter(post => post.author.includes(this.search));
    }
  },
  async mounted() {
    this.loader = true;
    const promises = [this.downloadPosts(), this.downloadAuthors()];
    const [posts, authors] = await Promise.all(promises)
    if(posts && authors) {
      for(let author of authors) {
        posts.forEach((post) => {
          if(author.id === post.userId) {
            post.author = author.name;
          }
        })
      }
      this.posts = posts;
      this.loader = false;
    }
  },
  methods: {
    async downloadPosts() {
      const response = await axios.get('http://jsonplaceholder.typicode.com/posts').catch(() => {
        this.error = true;
      });
      return response?.data;
    },
    async downloadAuthors() {
      const response = await axios.get('http://jsonplaceholder.typicode.com/users').catch(() => {
         this.error = true;
      });
      return response?.data;
    }
  }
}
</script>

<style scoped lang="less">

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  display: flex;
  justify-content: center;
  flex-direction: column;
  background: #EDF1F6;
  #input-group {
    max-width: 250px;
    /deep/ .input-group-text {
      display: block;
    }
  }

  .grid-container {
    display: grid;
    grid-template-columns:repeat(auto-fill, minmax(330px, 1fr));
    justify-content: center;
    align-items: baseline;
    grid-gap: 10px;
    .post-card {
      width: 100%;
      max-height: min-content;
    }
  }
}

</style>
