<template>
  <div>
    <section class="hero is-light" :class="{'is-fullheight': !ready || activities.length == 0}">
      <div class="hero-body" v-if="!ready">
        <span class="button is-large is-static is-primary is-outlined is-loading"></span>
      </div>
      <div class="hero-body" v-show="ready">
        <div class="container has-text-centered">
          <h1 v-if="profile" class="title is-spaced">
            <span>Hei {{ name }}</span>
            <button @click="signOutUser()" class="button is-primary is-outlined is-small">Logg ut</button>
          </h1>
          <p class="subtitle">Så langt denne sesongen har du samlett
            <strong>{{ activities | totalPoints }} poeng</strong> i cup'en og du er
            <strong>{{ activities | totalKm }} km</strong> på vei mot distansemerket.
          </p>
          <FormActions></FormActions>
        </div>
      </div>
    </section>
    <section v-if="user && activities.length > 0" class="section">
      <div class="container">
        <table class="table is-bordered is-striped">
          <thead>
            <tr>
              <th>Aktivitetsdato<br/>Poeng/Km</th>
              <th>Aktivitet</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(activity, key) in activities" :key="key">
              <td>
                {{ activity | date }}
                <br/> Poeng: {{ activity | points }} / Km: {{ activity | km }}
              </td>
              <td>
                {{ activity | title | capitalize}}
                <span v-if="activity.hoursAlpine">
                  <br/>- {{ activity.hoursAlpine }} time(r) i bakken.
                </span>
                <span v-if="activity.crossCountryKm">
                  <br/>- {{ activity.crossCountryKm }} km langrenn.
                </span>
                <span v-if="activity.comment">
                  <br/>- {{ activity.comment }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </div>
</template>

<script>
import FormActions from '~/components/FormActions.vue'
import { mapMutations, mapState } from 'vuex'
const KM_PER_HOUR_ALPINE = 4

function alpineAndCrossCountry (alpineHours, crossCountryKm) {
  let km = 0
  if (crossCountryKm) {
    km += parseInt(crossCountryKm)
  }
  if (alpineHours) {
    km += parseInt(alpineHours) * KM_PER_HOUR_ALPINE
  }
  return km
}

export default {
  components: {
    FormActions
  },
  computed: {
    ...mapState({
      ready: (store) => store.user.activities && store.user.profile,
      user: (store) => store.auth.user,
      profile: (store) => store.user.profile,
      needsProfile: (store) => store.user.needsProfile,
      activities: (store) => {
        if (!store.user.activities) {
          return []
        }
        return Object.keys(store.user.activities).map((key) => {
          return store.user.activities[key]
        }).sort((a, b) => {
          if (!b.date) {
            return -1
          }
          return a.date > b.date ? -1 : 1
        })
      }
    }),
    uid () {
      return this.user ? this.user.uid : ''
    },
    name () {
      return this.profile.firstName ? this.profile.firstName : this.user.email
    }
  },
  methods: {
    ...mapMutations({
      signOutUser: 'auth/signOutUser'
    })
  },
  filters: {
    title (activity) {
      var title = activity.activityType

      if (activity.activityType === 'deltatt på samling') {
        title += ' ' + activity.gatheringType
      } else if (activity.activityType === 'deltatt i konkurranse' && activity.competitionType) {
        title = activity.competitionType
      } else if (activity.activityType === 'trent' && activity.trainingType) {
        title = activity.trainingType
      }

      if (activity.meta) {
        title += ': ' + activity.meta
      }

      return title
    },
    totalPoints (activities) {
      return activities.reduce((sum, activity) => {
        return sum + parseInt(activity.points)
      }, 0)
    },
    totalKm (activities) {
      return activities.reduce((sum, activity) => {
        return sum + alpineAndCrossCountry(activity.hoursAlpine, activity.crossCountryKm)
      }, 0)
    },
    points (activity) {
      return activity.points
    },
    date (activity) {
      return activity.date ? activity.date : 'Mangler dato'
    },
    km (activity) {
      return alpineAndCrossCountry(activity.hoursAlpine, activity.crossCountryKm)
    },
    capitalize (text) {
      return text.charAt(0).toUpperCase() + text.slice(1)
    }

  },
  watch: {
    user (newUser, oldUser) {
      if (!newUser) {
        this.$router.push('/')
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import "assets/utilities";

.button.is-loading {
  padding: 3rem;
  margin: 0 auto;
}

.section {
  background: $white;
}

.title {
  *:last-child {
    margin-left: 1rem;
    margin-top: 0.4rem;
  }
}

.subtitle {
  max-width: 30rem;
  margin: 0 auto;
  margin-bottom: 2rem;
}

.button+.button {
  margin-left: 1rem;
}

.table {
  margin: 0 auto;
}
</style>

