<template>
  <div id="app">
    <h1>
      <router-link to="/">MoeLife合成指南</router-link>
    </h1>
	<div class="subtitle">
		<span><a href="http://www.onehouronelife.cn">论坛</a></span>
		<span><a href="/versions">更新日志</a></span>
	</div>

    <h2 v-if="loading">加载中...</h2>

    <div v-else>
		
	
      <ObjectSearch />

      <router-view />
    </div>
  </div>
</template>

<script>
import GameObject from './models/GameObject';
import Biome from './models/Biome';

import ObjectSearch from './components/ObjectSearch';
import ObjectBrowser from './components/ObjectBrowser';
import ObjectInspector from './components/ObjectInspector';
import TechTree from './components/TechTree';
import Recipe from './components/Recipe';
import RecipeForLetters from './components/RecipeForLetters';
import BiomeInspector from './components/BiomeInspector';
import ChangeLog from './components/ChangeLog';
import NotFound from './components/NotFound';

export default {
  name: 'app',
  components: {
    ObjectSearch,
  },
  data () {
    return {
      loading: true,
    }
  },
  beforeMount () {
    this.redirectOldHash();
    GameObject.load((data) => {
      Biome.setup(data.biomeIds, data.biomeNames);
      this.loading = false;
    });
  },
  computed: {
    lastDate() {
      const months = [
        "January", "February", "March",
        "April", "May", "June", "July",
        "August", "September", "October",
        "November", "December"
      ];
      var month = GameObject.date.getMonth();
      var day = GameObject.date.getDate();
      var year = GameObject.date.getFullYear();
      return `${months[month]} ${day}, ${year}`;
    },
    latestVersion() {
      return GameObject.versions[0];
    },
    showWhatsNew() {
      if (process.env.ONETECH_MOD_NAME)
        return false;
      if (this.$route.path == "/versions")
        return false;
      return true;
    },
    gameName() {
      return process.env.ONETECH_MOD_NAME || "One Hour One Life";
    },
    gameUrl() {
      return process.env.ONETECH_MOD_URL;
    },
    onEdge() {
      return global.edge;
    }
  },
  methods: {
    redirectOldHash() {
      if (!window.location.hash) return;
      const path = window.location.hash.substr(1).split("/");
      if (parseInt(path[0]) > 0) // Object ID route
        path.unshift([path.shift(), path.shift()].join("-"));
      this.$router.replace("/" + path.join("/"));
    },
    unreleasedContentUrl() {
      return "http://moetech.onehouronelife.cn/" + window.location.pathname;
    },
    releasedContentUrl() {
      return "http://moetech.onehouronelife.cn/" + window.location.pathname;
    }
  },
  metaInfo: {
    title: "MoeLife合成指南",
    titleTemplate: '%s | onetech'
  },
  routes: [
    {path: "/", component: ObjectBrowser},
    {path: "/not-found", component: NotFound},
    {path: "/filter/:filter", component: ObjectBrowser},
    {path: "/letters", component: RecipeForLetters},
    {path: "/versions", component: ChangeLog},
    {path: "/versions/:id", component: ChangeLog},
    {path: "/biomes/:id", component: BiomeInspector},
    {path: "/:id/tech-tree", component: TechTree},
    {path: "/:id/recipe", component: Recipe},
    {path: "/:id", component: ObjectInspector},
    {path: "*", redirect: "/not-found"},
  ]
}
</script>

<style lang="scss">
  body {
    background-color: #151515;
    margin: 0 auto;
    padding: 0 10px;
    max-width: 1024px;
  }

  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #d3d3d3;
    margin-top: 40px;
  }

  h1, h2 {
    font-weight: normal;
    text-align: center;
  }

  li {
    text-align: left;
  }

  a {
    color: inherit;
  }

  #app > h1 {
    margin-bottom: 0;
    a {
      text-decoration: none;
      &.router-link-exact-active {
        cursor: default;
      }
    }
  }

  .edgeTitle {
    text-align: center;
    font-size: 20px;
    color: #F63E3E;
    margin-bottom: 6px;
  }

  .subtitle {
    color: #777;
    text-align: center;
    a {
      color: #ccc;
      text-decoration: underline;
    }
  }
</style>
