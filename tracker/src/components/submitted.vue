<template>
  <div class="talks">
    <app-nav></app-nav>

    <b-row><b-col>&nbsp;</b-col></b-row>

    <h2>Talks Submissions to {{ conference.name }}</h2>

    <b-row>
      <b-col cols="6" offset="3">
        <p>The CFP for {{ conference.name }} is opened until {{ new Date(conference.cfpDate).toLocaleDateString() }}.</p>
        <p>You can submit your talks at <a :href="conference.cfpUrl" target="_blank">{{ conference.cfpUrl }}</a>. </p>
      </b-col>
    </b-row>

    <b-row>
      <b-col class="text-right">
        <talk-add-modal @talkAdded="getMyTalks()"></talk-add-modal>
      </b-col>
    </b-row>

    <b-row><b-col></b-col></b-row>

    <b-row v-for="talk in talks" :key="talk.id">
      <b-col cols="6" offset="3">
        <b-input-group>
          <b-input-group-prepend is-text>
            <input type="checkbox" aria-label="Checkbox if you submitted" :id="'talk-' + talk.id" v-model="talk.submitted">
          </b-input-group-prepend>
          <b-form-input type="text" aria-label="Title of talk" readonly :value="talk.title"/>

          <b-input-group-append is-text><span @click="expand" :id="'accordion' + talk.id">...</span></b-input-group-append>

        </b-input-group>
        <b-collapse :id="'accordion' + talk.id" accordion="my-accordion" role="tabpanel">
          <b-card :title="talk.title">
            <b-card-text>{{ talk.abstract }}</b-card-text>
          </b-card>
        </b-collapse>
      </b-col>
    </b-row>

    <b-row>
      <b-col class="text-center">
        <b-btn class="btn btn-info" @click="saveSubmissions()">Save</b-btn>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import Vue from "vue";
import AppNav from "./AppNav";
import TalkAddModal from "./talk-add-modal";
import { getMyTalks, getConference, addSubmissions, getMySubmissions } from "../utils/conf-api";

export default {
  name: "submitted",
  components: { AppNav, TalkAddModal },
  data() {
    return {
      talks: [],
      conference: {}
    };
  },
  mounted() {
    this.getMyTalks();
    this.getConferenceDetails();
  },
  methods: {
    getMyTalks() {
      getMyTalks().then((talks) => {
        this.talks = talks;
        return getMySubmissions(this.$route.params.conferenceId);
      }).then((submissions) => {
        submissions.map(s => Vue.set(this.talks.find(t => t.id === s.id), "submitted", true));
      });
    },
    getConferenceDetails() {
      getConference(this.$route.params.conferenceId).then((conf) => {
        this.conference = conf;
      });
    },
    saveSubmissions() {
      const submissions = this.talks.filter(talk => talk.submitted).map((talk) => {
        const talkId = talk._id;
        return { talkId };
      });
      addSubmissions(this.conference._id, submissions).then(() => this.$router.push("/conferences"));
    },
    expand(event) {
      this.$root.$emit("bv::toggle::collapse", event.currentTarget.id);
    }
  }
};
</script>

<style scoped>
  .row {
    margin-top: 3px;
  }
</style>
