<template>
  <v-app>
    <HeadNavigation/>
    <SideNavigation/>

    <v-main>
      <v-container>
        <v-row>
          <v-col
            class="text-right"
            cols="auto"
            lg="8"
            md="8"
            sm="12"
          >
            <h1>Quickstart <v-icon icon="mdi-book-open-variant"> </v-icon></h1>
          </v-col>
        </v-row>

        <v-row>
          <v-col
            class="text-right"
            cols="auto"
            lg="8"
            md="8"
            sm="12"
          >
          <h2>Report View </h2>

          <v-text-area>
            <p>
              The report "view" allows to visualize Updatecli pipeline report.
            </p>
            <p>
              Updatecli must be configured to publish reports after each pipeline execution.
            </p>

            <v-divider></v-divider>

            <v-card
              variant="flat"
            >
              <v-card-title>
                <h3>Login</h3>
              </v-card-title>
              <v-card-text>
                  updatecli udash login "{{ host }}" --experimental
              </v-card-text>
            </v-card>

            <v-card
              variant="flat"
            >
              <v-card-title>
                <h3>Publish</h3>
              </v-card-title>
              <v-card-text>
                  <br/>
                  updatecli diff --experimental ghcr.io/updatecli/policies/policies/autodiscovery/all:latest
              </v-card-text>
            </v-card>
          </v-text-area>

          </v-col>
          <v-col
            cols="auto"
            lg="4"
            md="4"
            sm="12"
            >
            <div
              v-for="link in links"
              :key="link.name"
              >
              <v-btn
                class="mx-4"
                variant="text"
                :prepend-icon="link.icon"
                :to="link.to">
                {{ link.name }}
              </v-btn>
            </div>
            <div
              v-for="link in externalLinks"
              :key="link.name"
              >
              <v-btn
                class="mx-4"
                variant="text"
                :prepend-icon="link.icon"
                :href="link.to">
                {{ link.name }}
              </v-btn>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <ReleaseFooter/>
  </v-app>
</template>

<script>
import ReleaseFooter from '../components/ReleaseFooter.vue';
import SideNavigation from '../components/SideNavigation.vue';
import HeadNavigation from '../components/HeadNavigation.vue';

export default {
  name: 'QuickStartView',
  components: {
    ReleaseFooter,
    SideNavigation,
    HeadNavigation,
  },

  data: () => ({
    host: window.location.protocol + "//" + window.location.host,
    externalLinks:[
      {
        name: "Updatecli",
        to: "https://www.updatecli.io",
        icon: "mdi-web",
      },
      {
        name: "GitHub",
        to: "https://github.com/updatecli/updatecli",
        icon: "mdi-github",
      },
    ],
    links:[
      {
        name: "Reports",
        to: "/pipeline/reports",
        icon: "mdi-satellite-variant",
      },
    ]
  }),
}
</script>

