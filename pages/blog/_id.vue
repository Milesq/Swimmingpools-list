<template>
  <section class="wrapper">
    <div class="meta">
      <Breadcrumb :crumbs="[['Blog', '/blog'], data.title]" />
      <Share link="" />
    </div>
    <h1 class="header">{{ data.title }}</h1>
    <article class="post" v-html="dataHtml"></article>
    <Comments @send="send" :data="data.comments" />
  </section>
</template>

<script>
import showdown from 'showdown';

import Breadcrumb from '@/components/Breadcrumb.vue';
import Share from '@/components/Share.vue';
import Comments from '@/components/Comments.vue';
import { API, auth } from '@/assets/config.json';

export default {
  async validate({ params, $axios }) {
    let flag = true;
    flag = flag && /^\d+$/.test(params.id);

    const { data } = await $axios.get(`${API}/posts/count`);

    flag = flag && params.id > 0;
    flag = flag && params.id <= data;

    return flag;
  },
  async asyncData({ params: { id }, $axios }) {
    const { data } = await $axios.get(`${API}/posts/${id}`);
    return { data };
  },
  methods: {
    async send(comment) {
      const { data: { jwt } } = await this.$axios.post(`${API}/auth/local`, {
        identifier: auth.name,
        password: auth.pass
      });

      await this.$axios.post(`${API}/comments`, {
        post: this.$route.params.id * 1,
        author: comment.user.name,
        content: comment.content
      }, {
        headers: {
          Authorization: `Bearer ${jwt}`
        }
      });

      window.location.reload();
    }
  },
  computed: {
    dataHtml() {
      const converter = new showdown.Converter();
      return converter.makeHtml(this.data.content);
    }
  },
  layout: 'static',
  components: {
    Comments,
    Breadcrumb,
    Share
  }
};
</script>

<style scoped lang="scss">
.wrapper {
  width: 100%;
  margin-left: auto;
  margin-right: auto;
  padding: 20px 70px;
  background-color: #fff;

  @media (max-width: 880px) {
    padding: 20px;
  }
}

.post {
  text-align: left;
  max-width: 100%;
  overflow: hidden;
  font-size: 16px;
  font-weight: 400;
  white-space: pre-wrap;

  @media(min-width: 785px) {
    padding: 10px 100px;
  }
}

.header {
  text-align: center;
}

.meta {
  display: flex;
  align-items: center;
  justify-content: space-between;

  @media (max-width: 910px) {
    flex-direction: column;
  }
}
</style>

<style lang="scss">
.post {
  p {
    margin-bottom: 20px;
  }

  img {
    max-width: 100%;
    /* max-height: 60vh; */
    margin-left: auto;
    margin-right: auto;
  }
}
</style>
