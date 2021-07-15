<template>
  <div class="launches-container">
    <h1>🚀 SpaceX Launches</h1>

    <section v-if="error">
      <div class="error">
        <img
          src="https://media.giphy.com/media/3o6ZsXRgEvYcwcNKJG/giphy.gif"
          alt="There was an error"
        />
        <div class="error-status">Status: {{ error.status }}</div>
        <div class="error-message">Message: {{ error.message }}</div>
      </div>
    </section>

    <section v-else>
      <imgs
        v-if="loading"
        src="https://media.giphy.com/media/hrWfA4T3ykR3yRV5uH/source.gif"
        alt="Searching..."
      />

      <div v-else class="show-launches">
        <div class="launch" v-for="launch in launches" :key="launch.id">
          <h2 class="launch-item mission-name">
            Mission Name: {{ launch.mission_name }}
          </h2>

          <div class="launch-item launch-date">
            🗓{{ launch.launch_date }} | 📖<a :href="launch.article_url">Read</a>
          </div>

          <div class="launch-item launch-video-container">
            <iframe
              :src="launch.video_url"
              title="YouTube video player"
              frameborder="0"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
              allowfullscreen
              class="launch-video"
            ></iframe>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { gql, request } from "graphql-request";

const query = gql`
  query GetLaunches {
    launches(limit: 5) {
      id
      mission_name
      launch_date_local
      links {
        article_link
        video_link
      }
      mission_id
    }
  }
`;

const fetcher = query => request("https://api.spacex.land/graphql", query);

export default {
  name: "Launches",
  data: function() {
    return {
      launches: null,
      error: null,
      loading: true
    };
  },
  mounted() {
    fetcher(query)
      .then(data => {
        const parsedRes = data.launches.map(m => {
          const { mission_name, id } = m;
          const embedVideoUrl =
            "https://www.youtube.com/embed/" +
            m.links.video_link.match(/v=(.*)$/)[1];
          const ld = new Date(m.launch_date_local).toDateString().split(" ");
          const launch_date = [ld[0], ld[2], ld[1], ld[3]].join(" ");

          return {
            id,
            mission_name,
            video_url: embedVideoUrl,
            article_url: m.links.article_link,
            launch_date
          };
        });
        this.launches = parsedRes;
      })
      .catch(err => {
        // NOTE: probably overkill, but fun to play with error messages
        const errRes = JSON.parse(err.message.split("):")[1]).response;
        this.error = {
          message: errRes.error.match(/<pre>(.*)<\/pre>/)[1],
          status: errRes.status
        };
      })
      .finally(() => (this.loading = false));
  }
};
</script>

<style scoped>
h1 {
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  padding: 5px;
  background-color: #c0c5ce;
  border: solid 1px black;
  margin-bottom: 5px;
  border-radius: 5px;
}

.launch {
  background-color: #c0c5ce;
  margin: 5px 15px 15px 0;
  border-radius: 5px;
  padding: 0 15px 5px 15px;
  flex-grow: 1;
}

.show-launches {
  display: flex;
  flex-wrap: wrap;
}

.launch-video {
  border-radius: 5px;
  width: 560px;
  height: 315px;
}

.launch-date {
  padding: 5px;
}
</style>
