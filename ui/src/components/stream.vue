<template>
  <div class="hello">
    <div class="sort-menu">
      <select v-model="sorting">
        <option value="most recent">most recent</option>
        <option value="least recent">least recent</option>
      </select>
    </div>
    <h1>CVEs in Stream {{this.stream}}</h1>
    <div id="content">
      <div class="row outstanding-container" v-if="outstanding">
        <h3>Outstanding CVEs in this Stream</h3>
        <div class="outstanding" v-for="(data, cve) in outstanding" v-bind:key="cve">
          <router-link v-if="data.cmt_msg" :to="'/cves/' + cve">{{ cve }}</router-link>
          <span v-else>{{ cve }}</span>
        </div>
      </div>
      <div class="row" v-for="(fixes, stream) in orderedStreams" v-bind:key="stream">
        <a class="anchor" v-if="stream != 'outstanding'" v-bind:id="stream"></a>
        <h3 v-if="stream != 'outstanding'">Fixed in {{stream}}</h3>
        <div class="card-container">
          <div class="cards" v-for="(data, fix) in fixes" v-bind:key="fix">
            <router-link :to="'/cves/' + fix">{{fix}}</router-link>
            <p v-if="stream != 'outstanding'">{{ data.cmt_msg }}</p>
            <p v-if="stream != 'outstanding'"><strong>{{ data.cmt_id | trim }}...</strong> </p>
                <button v-if="stream != 'outstanding'" class="copy-button" type="button"
                        v-clipboard:copy="data.cmt_id"
                        v-clipboard:success="onCopy"
                        v-clipboard:error="onError">Copy ID</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import VueClipboard from 'vue-clipboard2'

Vue.use(VueClipboard)

export default {
  data () {
    return {
      stream: [],
      contents: [],
      errors: [],
      sorting: 'most recent',
      message: 'copy'
    }
  },
  created () {
    this.load()
  },
  watch: {
    '$route' () {
      this.load()
    }
  },
  methods: {
    onCopy: function (e) {
      alert('Commit ID copied to clipboard: ' + e.text)
    },
    onError: function () {
      alert('Failed to copy text.')
    },
    load: function () {
      this.stream = this.$route.path.split('/')[2]
      document.title = 'Linux Kernel CVEs | ' + 'CVEs in Stream ' + this.stream
      var url = this.$apiBaseUrl + 'data/stream_data.json'
      axios.get(url)
        .then(response => {
          // JSON responses are automatically parsed.
          var stream = response.data[this.stream]
          if (stream == null) {
            this.$router.push('/404')
          }
          this.contents = stream
        })
        .catch(e => {
          this.errors.push(e)
        })
    },
  },
  filters: {
    trim: function (value) {
      if (!value) return ''
      return value.substring(0, 7)
    }
  },
  computed: {
    orderedStreams: function ()  {
      var sortedStreams = this.contents
      delete sortedStreams['outstanding']
      if (this.sorting === 'most recent') {
        var reversed = {}
        var keys = Object.keys(this.contents).reverse()
        keys.forEach((key) => {
          reversed[key] = this.contents[key]
        })
        return reversed
      }
      return this.contents
    },
    outstanding: function () {
      return this.contents['outstanding']
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#content {
  text-align: left;
}
.hello {
  margin: 1em 0;
}
.sort-menu {
  float: right;
}
@media only screen and (min-width: 1170px) {
  .outstanding-container {
    top: 150px;
    max-width: 180px;
    position: absolute;
    background-color: white;
    left: 0;
    margin-left: 2em;
  }
}
.card-container:before {
  content: "";
  display: inline-block;
  vertical-align: middle;
  height: 100%;
}
.cards {
  display: inline-block;
  vertical-align: top;
  width: 200px;
  min-height: 160px;
  text-align: left;
  padding: 1em 1.5em;
  margin: 0 1em 1em 0;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
.streamCVE {
  width: 240px;
  padding: 10px;
  margin: 10px;
  display: inline-block;
  overflow: hidden;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
h1, h2 {
  font-weight: normal;
}
h1 {
  margin: 1em 0;
}
ul {
  list-style-type: none;
}
a {
  color: #42b983;
}
button.copy-button {
  webkit-transition: opacity .3s ease-in-out;
  -o-transition: opacity .3s ease-in-out;
  transition: opacity .3s ease-in-out;
  opacity: 1;
  padding: 2px 6px;
  right: 4px;
  top: 4px;
}
a.anchor {
  display: block;
  position: relative;
  top: -80px;
  visibility: hidden;
}
</style>
