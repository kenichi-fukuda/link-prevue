<template>
  <div>
    <div id="loader-container" v-if="!response && validUrl" style="width: 100%;">
      <slot name="loading">
      <div class="spinner"></div>
      </slot>
    </div>
    <div v-if="response">
      <slot :img="response.images[0]" :title="response.title" :description="response.description" :url="url">
      <div class="blog-preview-wrapper" @click="viewMore">
        <div class="blog-preview-card-img">
          <img :src="response.images[0]">
        </div>
        <div class="blog-preview-card-info">
          <div class="card-text">
            <h4>{{response.title}}</h4>
          </div>
          <div class="blog-preview-card-btn">
            <a class="btn btn--tertiary btn--small" href="javascript:;" v-if="showButton" @click="viewMore">記事を読む</a>
          </div>
        </div>
      </div>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'link-prevue',
  props: {
    url: {
      type: String,
      default: ''
    },
    cardWidth: {
      type: String,
      default: '400px'
    },
    onButtonClick: {
      type: Function,
      default: undefined
    },
    showButton: {
      type: Boolean,
      default: true
    },
    apiUrl: {
      type: String,
      default: 'https://linkpreview-api.herokuapp.com/'
    }
  },
  watch: {
    url: function(value) {
      this.response = null
      this.getLinkPreview()
    }
  },
  created() {
    this.getLinkPreview()
  },
  data: function() {
    return {
      response: null,
      validUrl: false
    }
  },
  methods: {
    viewMore: function() {
      if (this.onButtonClick !== undefined) {
        this.onButtonClick(this.response)
      } else {
        const win = window.open(this.url, '_blank')
        win.focus()
      }
    },
    isValidUrl: function(url) {
      const regex = /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)/
      this.validUrl = regex.test(url)
      return this.validUrl
    },
    getLinkPreview: function() {
      if (this.isValidUrl(this.url)) {
        this.httpRequest((response) => {
          this.response = JSON.parse(response)
        }, () => {
          this.response = null
          this.validUrl = false
        })
      }
    },
    httpRequest: function(success, error) {
      const http = new XMLHttpRequest()
      const params = 'url=' + this.url
      http.open('POST', this.apiUrl, true)
      http.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')
      http.onreadystatechange = function() {
        if (http.readyState === 4 && http.status === 200) {
          success(http.responseText)
        }
        if (http.readyState === 4 && http.status === 500) {
          error()
        }
      }
      http.send(params)
    }
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css?family=Hind+Siliguri:400,600');

.blog-preview-wrapper {
  overflow: auto;
  border-radius: 7px 7px 7px 7px;
  background-color: #fff;
  -webkit-box-shadow: 0px 14px 32px 0px rgba(0, 0, 0, 0.15);
  -moz-box-shadow: 0px 14px 32px 0px rgba(0, 0, 0, 0.15);
  box-shadow: 0px 14px 32px 0px rgba(0, 0, 0, 0.15);
  display: flex;
  width: 100%;
  cursor: pointer;
}

.blog-preview-card-img {
  width: 30%;
  display: flex;
  align-items: center;
}

.blog-preview-card-img img {
  width: 100%;
  border-radius: 7px 7px 0 0;
}

img {
  vertical-align: middle;
  border-style: none;
}

.blog-preview-card-info {
  border-radius: 0 0 7px 7px;
  background-color: #ffffff;
  width: 70%;
  display: flex;
  flex-flow: column;
  align-items: center;
  padding: 16px;
}

.blog-preview-card-text {
  width: 100%;
  margin: 0 auto;
  text-align: justify;
}

.blog-preview-card-text h4 {
  text-align: center;
  font-size: 24px;
  color: #474747;
  margin: 5px 0 5px 0;
}

.blog-preview-card-btn {
  margin: 1em 0 1em 0;
  position: relative;
  text-align: center;
}

/* Loader */
.spinner {
  margin-top: 40%;
  margin-left: 45%;
  height: 28px;
  width: 28px;
  animation: rotate 0.8s infinite linear;
  border: 5px solid #868686;
  border-right-color: transparent;
  border-radius: 50%;
}

@keyframes rotate {
  0%    { transform: rotate(0deg); }
  100%  { transform: rotate(360deg); }
}
</style>
