# vuex → 비지니스 로직?

### 충격 ! vuex는 데이터 저장소로 쓰이는것뿐만이 아닌 비지니스 로직을 실행하는 곳이였다

<aside>
💡 vue는 단지 데이터를 보여줄 뿐, 비지니스 로직 처리를 하면 안된다. 아래 예를 보자

</aside>

views/NewsView.vue

```jsx
<template>
  <div>
    <div v-if="errorMessage == null">
      <div v-for="user in GET_FETCH_NEWS" :key="user.id">
        {{user.title}}
      </div>
    </div>
    <div v-if="errorMessage != null">
      {{errorMessage}}
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
const newsStore = "newsStore";

export default {
  computed:{
    ...mapGetters(newsStore, ["GET_FETCH_NEWS"]),
  },
  created(){
    this.Init()
  },
  data() {
    return {
      errorMessage:null,
    };
  },
  methods:{
    ...mapActions(newsStore, ["FETCH_NEWS"]),
    Init(){
      this.FETCH_NEWS().catch(error => {
        this.errorMessage = "error desu"
      })
    }
  },
}
</script>
```

api/index.js (axios)

```jsx
import axios from 'axios'

const config = {
  baseUrl: 'https://api.hnpwa.com'
}

function fetchNewsList() {
  return axios.get(`${config.baseUrl}/v0/news/1.json`);
}

export {
  fetchNewsList,
}
```

store/modules/newsStore.js

```jsx
import { fetchNewsList } from '../../api/index';

const newsStore = {
  namespaced: true,
  state: {
    fetchNews: null,
  },
  getters: {
    GET_FETCH_NEWS: state => state.fetchNews,
  },
  mutations: {
    SET_FETCH_NEWS: (state, commit_data) => {
      state.fetchNews = commit_data
    },
  },
  actions: {
    FETCH_NEWS({ commit }, payload) {
      return new Promise((resolve, reject) => {
        console.log("in SET_FETCH_NEWS")
        fetchNewsList()
          .then(response => {
            var commit_data = response.data
            commit('SET_FETCH_NEWS', commit_data)
          })
          .catch(error => {
            reject(error);
          })
      })
    },
  },
}

export default newsStore
```

http 요청 후 성공시에만 commit하여 state에 적용 시킨다.

그럼 vue에서 그 값을 보여주기만 하면 된다.

혹시 에러가 났을 시에는 reject를 돌려주어 vue쪽에 처리를 맞기자
