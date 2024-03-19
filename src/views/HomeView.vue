<script setup></script>

<template>
  <main>
    <div class="news-container">
      <div class="news-search">
        <input v-model="searchQuery" type="text" class="search-input" />
        <button @click="searchNews" class="search-button">Search</button>
      </div>
      <div class="toggle-container">
        <button
          class="toggle-button"
          :class="{ active: isToggled }"
          @click="isToggled = !isToggled"
        >
          All News
        </button>
        <button
          class="toggle-button"
          :class="{ active: !isToggled }"
          @click="isToggled = !isToggled"
        >
          Saved News
        </button>
      </div>
      <div v-for="(newsRow, index) in chunkedNewsData" :key="index">
        <div v-if="index % 2 == 0" class="news-row">
          <div class="news-column large">
            <div class="news-item-wrapper">
              <div @click="openNews($event, newsRow[0])" class="news-item" v-if="newsRow[0]">
                <div class="news-img-wrapper">
                  <div v-if="loading" class="skeleton-loader"></div>
                  <img
                    :src="
                      newsRow[0].urlToImage != null
                        ? newsRow[0].urlToImage
                        : '/src/assets/no-image.png'
                    "
                    alt=""
                  />
                </div>
                <div class="news-title" :class="{ 'skeleton-loader': loading }">
                  <div class="news-desc">
                    <div class="author">{{ newsRow[0].author }}</div>
                    <div class="date">{{ formatIndoDate(newsRow[0].publishedAt) }}</div>
                    <p class="desc">{{ newsRow[0].description }}</p>
                  </div>
                  <h3>
                    {{ newsRow[0].title }}
                  </h3>
                </div>
              </div>
            </div>
          </div>
          <div class="news-column small">
            <div class="news-item-wrapper" v-for="(item, i) in newsRow.slice(1)" :key="i">
              <div @click="openNews($event, item)" class="news-item">
                <div class="news-img-wrapper">
                  <div v-if="loading" class="skeleton-loader"></div>
                  <img
                    :src="item.urlToImage != null ? item.urlToImage : '/src/assets/no-image.png'"
                    alt=""
                  />
                </div>
                <div class="news-title" :class="{ 'skeleton-loader': loading }">
                  <div class="news-desc">
                    <div class="author">{{ item.author }}</div>
                    <div class="date">{{ formatIndoDate(item.publishedAt) }}</div>
                    <p class="desc">{{ item.description }}</p>
                  </div>
                  <h3>{{ item.title }}</h3>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-else class="news-row">
          <div class="news-column small">
            <div
              class="news-item-wrapper"
              v-for="(item, i) in newsRow.slice(0, newsRow.length - 1)"
              :key="i"
            >
              <div @click="openNews($event, item)" class="news-item">
                <div class="news-img-wrapper">
                  <div v-if="loading" class="skeleton-loader"></div>
                  <img
                    :src="item.urlToImage != null ? item.urlToImage : '/src/assets/no-image.png'"
                    alt=""
                  />
                </div>
                <div class="news-title" :class="{ 'skeleton-loader': loading }">
                  <div class="news-desc">
                    <div class="author">{{ item.author }}</div>
                    <div class="date">{{ formatIndoDate(item.publishedAt) }}</div>
                    <p class="desc">{{ newsRow.description }}</p>
                  </div>
                  <h3>{{ item.title }}</h3>
                </div>
              </div>
            </div>
          </div>
          <div class="news-column large">
            <div class="news-item-wrapper">
              <div
                @click="openNews($event, newsRow[newsRow.length - 1])"
                class="news-item"
                v-if="newsRow[newsRow.length - 1]"
              >
                <div class="news-img-wrapper">
                  <div v-if="loading" class="skeleton-loader"></div>
                  <img
                    :src="
                      newsRow[newsRow.length - 1].urlToImage != null
                        ? newsRow[newsRow.length - 1].urlToImage
                        : '/src/assets/no-image.png'
                    "
                    alt=""
                  />
                </div>

                <div class="news-title" :class="{ 'skeleton-loader': loading }">
                  <div class="news-desc">
                    <div class="author">{{ newsRow[newsRow.length - 1].author }}</div>
                    <div class="date">
                      {{ formatIndoDate(newsRow[newsRow.length - 1].publishedAt) }}
                    </div>
                    <p class="desc">{{ newsRow[newsRow.length - 1].description }}</p>
                  </div>
                  <h3>{{ newsRow[newsRow.length - 1].title }}</h3>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  name: 'NewsPage',
  data() {
    return {
      newsData: [],
      savedNews: {},
      searchQuery: '',
      loading: true,
      isToggled: true
    }
  },

  mounted() {
    const savedNews = localStorage.getItem('savedNews')
    if (savedNews) {
      this.savedNews = JSON.parse(savedNews)
    }
    fetch('https://newsapi.org/v2/top-headlines?country=us&apiKey=9a0f136757fd44f384b13ba8de0dfc5a')
      .then((res) => res.json())
      .then((data) => {
        console.log(data)
        this.newsData = data.articles
        this.loading = false
      })
      .catch((err) => console.log(err.message))
  },
  computed: {
    chunkedNewsData() {
      if (this.isToggled) {
        const chunkSize = 5
        let result = []
        for (let i = 0; i < this.newsData.length; i += chunkSize) {
          result.push(this.newsData.slice(i, i + chunkSize))
        }
        return result
      } else {
        const chunkSize = 5
        let result = []
        let newsData = []

        for (const key in JSON.parse(JSON.stringify(this.savedNews))) {
          newsData.push({
            title: key,
            url: this.savedNews[key].url,
            urlToImage: this.savedNews[key].urlToImage
          })
        }
        for (let i = 0; i < newsData.length; i += chunkSize) {
          result.push(newsData.slice(i, i + chunkSize))
        }
        return result
      }
    }
  },
  methods: {
    openNews(evt, news) {
      this.savedNews[news.title] = {
        url: news.url,
        urlToImage: news.urlToImage
      }
      localStorage.setItem('savedNews', JSON.stringify(this.savedNews))
      window.open(news.url, '_blank')
    },

    searchNews() {
      fetch(
        'https://newsapi.org/v2/top-headlines?country=us&apiKey=9a0f136757fd44f384b13ba8de0dfc5a&q=' +
          this.searchQuery
      )
        .then((res) => res.json())
        .then((data) => {
          console.log(data)
          this.newsData = data.articles
        })
        .catch((err) => console.log(err.message))
    },
    formatIndoDate(isoDate) {
      const date = new Date(isoDate)

      const optionsDate = { weekday: 'short', year: 'numeric', month: 'long', day: 'numeric' }
      const optionsTime = { hour: 'numeric', minute: 'numeric', hour12: false }

      const formattedDate = date.toLocaleDateString('id-ID', optionsDate)
      const formattedTime = date.toLocaleTimeString('id-ID', optionsTime).replace('.', ':')

      return `${formattedDate}, ${formattedTime}`
    }
  }
}
</script>

<style scoped>
.toggle-container {
  display: flex;
  padding: 0 12px;
}

.toggle-button {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  background-color: #f9f9f9;
  cursor: pointer;
  transition: background-color 0.3s;
}

.toggle-button.active {
  background-color: #0074d9;
  color: #fff;
}

@keyframes pulse-bg {
  0% {
    background-color: #ddd;
  }
  50% {
    background-color: #d0d0d0;
  }
  100% {
    background-color: #ddd;
  }
}
.news-img-wrapper .skeleton-loader {
  width: 100%;
  height: 100%;
  position: relative;
  top: 0;
  left: 0;
  z-index: 5;
}

.news-img-wrapper .skeleton-loader {
  animation: pulse-bg 1s infinite;
}

.news-search {
  display: flex;
  align-items: center;
}

.search-input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  flex: 1;
}

.search-button {
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 10px 20px;
  margin-left: 10px;
  cursor: pointer;
  font-size: 14px;
}

.result-container {
  margin-top: 20px;
  /* Add additional styling for displaying API results */
}

.news-container {
  display: flex;
  flex-direction: column;
}
.row-reverse {
  flex-direction: row-reverse;
}
.news-column {
  flex: 1;
}
.news-item {
  display: block;
  color: black;
  text-decoration: none;

  position: relative;
  width: 100%;
  border: 1px solid rgb(184, 184, 184);
  border-radius: 8px;
  overflow: hidden;
}

.news-item h3 {
  font-weight: bold;
}
.news-title {
  z-index: 10;
  padding: 6px 12px;
  font-size: 16px;
  background: white;
  position: realtive;
}
.news-title.skeleton-loader h3 {
  width: 100%;
  border-radius: 10px;
  z-index: 5;
  content: '';
  white-space: nowrap;
  text-indent: -9999px;
  animation: pulse-bg 1s infinite;
}

.news-save {
  position: absolute;
  top: -40px;
  right: 0;
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 0 15px;
  height: 40px;
  margin-left: 10px;
  cursor: pointer;
  font-size: 14px;
  z-index: 20;
}

.news-item:hover .news-desc {
  display: block;
}

.news-desc {
  display: none;
  padding: 12px;
  margin-top: -8px;
  background-color: white;
  border-bottom: 1px solid #ddd;
  position: absolute;
  width: 100%;
  bottom: 100%;
  right: 0;
}
.news-desc .author {
  font-weight: bold;
}

.news-item-wrapper {
  display: block;
  float: left;
  width: 100%;
  margin: 12px 0;
}

.news-item img {
  max-width: 100%;
}

@media (min-width: 600px) {
  .news-search {
    padding: 0 12px;
    margin-bottom: 15px;
  }
  .news-row {
    display: block;
    width: 100%;
    float: left;
  }
  .news-column {
    display: block;

    width: 100%;
  }

  .news-column.small {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
  }

  .news-item img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  .news-title {
    position: relative;
    bottom: 0;
  }

  .news-item-wrapper {
    padding: 12px;
    margin: 0;
  }

  .small .news-item {
    height: 100%;
  }

  .small .news-item img {
    height: 240px;
    width: auto;
    margin: 0 auto;
    display: block;
  }

  .small .news-item-wrapper {
    width: 50%;
  }
}

@media (min-width: 959px) {
  .news-column {
    display: block;
    width: 50%;
    float: left;
  }

  .news-column.small {
    display: flex;
    flex-wrap: wrap;
  }

  .large .news-item-wrapper {
    padding: 12px;
  }

  .small .news-item-wrapper {
    width: 50%;
    padding: 12px;
  }

  .small .news-item img {
    width: 100%;
  }
  .news-title {
    width: 100%;
    position: absolute;
    bottom: 0;
    min-height: 88px;
    max-height: 25%;
    vertical-align: middle;
    display: flex;
    justify-content: center;
    flex-direction: column;
  }
  .small .news-title {
    padding: 0 12px;
    margin: 0;
    font-size: 12px;
    background: white;
    height: 25%;
    min-height: 88px;
  }
  .small .news-title h3 {
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    line-height: 1.5em;
    max-height: 4.5em;
  }
  .large .news-item {
    padding-bottom: calc(100 / 120 * 100%);
  }

  .small .news-item {
    padding-bottom: calc(100 / 120 * 100% - 3px);
  }
  .news-item {
    position: relative;
  }
  .news-img-wrapper {
    display: block;
    height: 100%;
    width: 100%;
    position: absolute;
    background: rgba(0, 0, 0, 0.75);
    white-space: nowrap;
  }
  .small .news-img-wrapper {
    height: 75%;
  }
  .news-img-wrapper img {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
  }

  .large .news-desc .author {
    display: inline;
  }

  .large .news-desc .date {
    float: right;
  }
}
</style>
