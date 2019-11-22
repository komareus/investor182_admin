<template>
  <div class="home-page">
    <h1 class="home-page__title">
      Адміністративна панель
    </h1>
    <section class="home-page__header-links-section">
      <ul>
        <li
          v-for="(item, i) in headLinks"
          :key="i"
          class="home-page__header-links-section-item"
        >
          <a :href="item.link" :target="item.target">{{ item.title }}</a>
        </li>
      </ul>
    </section>

    <section v-if="showUserInfo" class="home-page__user-info">
      <div v-if="currentUser">
        <p>
          Вітаємо, ви увійшли в профіль, як:
        </p>
        <ul>
          <li>
            Ім'я: <b>{{ currentUser.user_metadata.full_name }}</b>
          </li>
          <li>
            Email: <b>{{ currentUser.email }}</b>
          </li>
          <li v-if="currentUser.app_metadata.roles">
            Права доступу:
            <b
              v-for="(item, i) in currentUser.app_metadata.roles"
              :key="i"
            >
              {{ item }}
            </b>
          </li>
        </ul>
      </div>
      <div v-else>
        <p>
          Ви не увійшли в профіль.
        </p>
        <p>
          Для входу, натисніть клавішу <b>"УВІЙТИ В ПРОФІЛЬ"</b>
        </p>
        <p>
          Після успішного входу до профілю, ви зможете перейти до потрібного вам розділу
        </p>
      </div>
    </section>

    <section class="home-page__login-section">
      <div class="home-page__login-section-button-wrapper">
        <a-button v-if="!currentUser" class="home-page__login-section-button" @click="openModal">
          увійти в профіль
        </a-button>
        <a-button v-else class="home-page__login-section-button" @click="logOut">
          вийти з профілю
        </a-button>
      </div>
    </section>

    <hr>

    <section class="home-page__chapter-section">
      <h2>Розділи</h2>
      <div class="home-page__chapter-section-container">
        <div
          v-for="(item, i) in chaptersEnabled"
          :key="i"
          class="home-page__chapter-section-button-wrapper"
        >
          <a-button
            class="home-page__chapter-section-button"
            :disabled="item.disabled"
            @click="goToChapter(item.link)"
          >
            {{ item.title }}
          </a-button>
        </div>
      </div>
    </section>

    <hr>

    <section class="home-page__clear-section">
      <a-button class="home-page__clear-cache-button" outline @click="clearCache">
        Очистити кеш
      </a-button>
    </section>
    <div id="netlify-modal"></div>
  </div>
</template>

<script>
import AButton from '~/components/shared/AButton'
let netlifyIdentity
if (process.browser) {
  netlifyIdentity = require('netlify-identity-widget')
  // require('https://identity.netlify.com/v1/netlify-identity-widget.js')
  // require('https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js')
}
// import netlifyIdentity from 'netlify-identity-widget'
export default {
  name: 'Home',
  components: { AButton },
  head() {
    return {
      script: [
        // { src: 'https://identity.netlify.com/v1/netlify-identity-widget.js' },
        // { src: 'https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js' }
      ]
    };
  },
  data() {
    return {
      currentUser: null,
      showUserInfo: false,
      headLinks: [
        { title: 'Повернутися до сайту', link: 'https://komareus.github.io', target: '' },
        { title: 'Перейти до медіасховища', link: 'https://cloudinary.com/users/login', target: '_blank' },
      ],
      chapters: [
        { title: 'Загальне', link: '/admin', roles: ['admin', 'directors', 'revision', 'supervisory'] },
        { title: 'Правління', link: '/admin-directors', roles: ['admin', 'directors'] },
        { title: 'Наглядова рада', link: '/admin-supervisory', roles: ['admin', 'supervisory'] },
        { title: 'Ревізійна коміссія', link: '/admin-revision', roles: ['admin', 'revision'] },
      ]
    }
  },
  created() {
    if (process.browser) {
      this.getLocalstorrageUser()
      this.initNetlify();
    }
  },
  mounted() {
    if (process.browser) {
      this.getLocalstorrageUser();
      this.$nextTick(() => {
        this.showUserInfo = true
      })
    }
  },
  methods: {
    initNetlify() {
      netlifyIdentity.init({
        container: '#netlify-modal'
      });

      netlifyIdentity.on('init', user => console.log('init', user));
      netlifyIdentity.on('login', user => this.onLogin(user));
      netlifyIdentity.on('logout', () => this.onLogOut());
      netlifyIdentity.on('error', err => console.error('Error', err));
      netlifyIdentity.on('open', () => console.log('Widget opened'));
      netlifyIdentity.on('close', () => console.log('Widget closed'));
    },
    openModal() {
      netlifyIdentity.open('login');
    },
    onLogin(user) {
      this.currentUser = user;
      netlifyIdentity.close();
    },
    logOut() {
      netlifyIdentity.logout();
    },
    onLogOut() {
      console.log('onLogOut')
      this.currentUser = null;
    },
    clearCache() {
      // const cookiesRes = this.$cookies.getAll();
      // console.log('COOCIES', cookiesRes)
      this.$cookies.removeAll();
      localStorage.clear();
    },
    getLocalstorrageUser() {
      const currentUser = localStorage.getItem('gotrue.user');
      this.currentUser = JSON.parse(currentUser) || null;
      if (!currentUser) {
        this.openModal();
      }
    },
    goToChapter(link) {
      window.open(link, '_self');
    }
  },
  computed: {
    chaptersEnabled() {
      const userRoles = this.currentUser ? this.currentUser.app_metadata.roles : [];
      return this.chapters.map((item) => {
        return {
          ...item,
          disabled: !userRoles.find(it => item.roles.find(role => role === it))
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .home-page {
    hr{
      color: $c--primary;
      margin: 15px 0
    }
    color: $c--primary;
    h1{
      text-align: center;
    }
    &__header-links-section{
      margin-bottom: 30px;
    }
    &__header-links-section-item{
      margin-bottom: 4px;
    }
    &__user-info{
      padding: 0 30px;
      margin-bottom: 30px;
      border: 1px solid $c--primary;
      border-radius: 8px;
      color: $c--primary;
      ul{
        list-style: none;
        padding-left: 20px;
      }
    }
    &__login-section{
      margin: 0 -4px;
      display: flex;
    }
    &__login-section-button-wrapper{
      padding: 4px;
      display: flex;
      flex-basis: 100%;
    }
    &__chapter-section{
      h2{
        margin-top: 10px;
        margin-bottom: 10px;
      }
    }
    &__chapter-section-container{
      margin: 0 -4px;
      display: flex;
    }
    &__chapter-section-button-wrapper{
      padding: 4px;
      display: flex;
      flex-basis: 100%;
    }
    &__chapter-section-button{
      margin: 4px 0;
      /*min-width: 300px;*/
    }
    &__clear-section{
      position: fixed;
      padding: 6px 0;
      bottom: 0;
      left: 0;
      width: calc(100% - 12px);
      height: 50px;
    }
    &__clear-cache-button{
      margin: 0 6px;
    }
  }
  @media only screen and (max-width: 600px) {
    .home-page {
      &__header-links-section{
        padding: 2px 0;
        margin-bottom: 14px;
      }
      &__user-info{
        font-size: .8rem;
        padding: 2px 8px;
        margin-bottom: 14px;
      }
      &__login-section{
        flex-wrap: wrap;
      }
      &__chapter-section-container{
        flex-wrap: wrap;
      }
      &__clear-section{
        height: 30px;
      }
    }
  }
</style>
