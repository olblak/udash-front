<template>
    <v-container
        class="py-8 px-6"
        fluid
    >
        <v-expansion-panels
            multiple
            flat
            tile
            v-model="localExpandedSummary"
        >
            <v-expansion-panel
                v-for="(scmData, url) in data"
                :key="url"
                :hide-actions="true"

            >
                <v-expansion-panel-title
                >
                    <v-toolbar
                        density="compact"
                        class="text-white"
                    >
                        <v-toolbar-title
                            class="flex text-center"
                        >
                            <v-icon>{{ getGitIcon(url) }}</v-icon>{{  sanitizeURL(url) }}
                        </v-toolbar-title>
                    </v-toolbar>
                </v-expansion-panel-title>
                <v-expansion-panel-text>
                    <v-card
                        variant="flat"
                    >
                        <v-card-text>
                            <v-timeline
                                density="compact"
                                align="start"
                                justify="center"
                                direction="vertical"
                                line-thickness="2"
                                line-color="grey-darken-3"
                            >
                                <v-timeline-item
                                    v-for="(branchData, branch) in scmData"
                                    :key="branch"
                                    dot-color="grey-darken-3"
                                    icon-color="white"
                                    fill-dot
                                    icon="mdi-source-branch"
                                >
                                    <v-card
                                        min-width="300"
                                        flat
                                    >
                                        <v-card-title
                                            variant="flat"
                                        >
                                            <v-toolbar
                                                density="compact"
                                                class="text-white"
                                                variant="text"
                                                flat
                                            >
                                                <v-toolbar-title
                                                    class="flex text-center"
                                                >
                                                    {{  branch  }}
                                                </v-toolbar-title>

                                                <v-btn
                                                    class="text-right"
                                                    justify-center
                                                    v-if="!hideButton"
                                                    :to="`/pipeline/reports?scmid=${branchData.id}`"
                                                    icon="mdi-arrow-right-circle"
                                                    slim
                                                    density="compact"
                                                >
                                                </v-btn>
                                            </v-toolbar>
                                        </v-card-title>
                                        <v-card-text
                                            variant="outlined"
                                            class="text-center"
                                        >
                                            {{  branchData.total_result }} reports
                                            <PolarArea
                                                :data="getPolarAreaData(url, branch)"
                                                :options="polarAreaOptions"
                                            />
                                        </v-card-text>
                                    </v-card>
                                </v-timeline-item>
                            </v-timeline>
                        </v-card-text>
                        <v-divider></v-divider>
                    </v-card>
                </v-expansion-panel-text>
            </v-expansion-panel>
        </v-expansion-panels>
    </v-container>
</template>

<script>

import {
  Chart as ChartJS,
  RadialLinearScale,
  ArcElement,
  Tooltip,
  Legend
} from 'chart.js'

import { UDASH_API_VERSION } from '@/constants';

import { PolarArea } from 'vue-chartjs'

import router from '../../router'

ChartJS.register(RadialLinearScale, ArcElement, Tooltip, Legend)

export default {
    components: {
        PolarArea
    },
    name: "SCMSummary",
    props: {
        scmid: {
            type: String,
            default: ''
        },
        hideButton: {
            type: Boolean,
            default: false
        },
        expandedSummary: {
            type: Array,
            default: () => [],
        }
    },
    data: () => ({
        data: {},
        localExpandedSummary: [],
        polarAreaData: {},
        polarAreaOptions: {
            responsive: true,
        },
    }),

    watch: {
      scmid () {
        this.getSummaryData();
      },
      expandedSummary(newVal) {
        this.localExpandedSummary = newVal;
      },
  },

    methods: {
        async getSummaryData() {
            this.$emit('loaded', false)
            const auth_enabled = process.env.VUE_APP_AUTH_ENABLED === 'true';
            const restrictedSCM = router.currentRoute.value.query.scmid

            let query = `/api/${ UDASH_API_VERSION}/pipeline/scms?summary=true`;
            if (restrictedSCM != undefined) {
                query = query + `&&scmid=${restrictedSCM}`
            } else if ( this.scmid != "" ){
                query = query + `&&scmid=${this.scmid}`
            }

            if (this.scmid != undefined && this.scmid != '' && this.scmid != null) {
                query += `&&scmid=${this.scmid}`;
            }

            if (auth_enabled) {
                const token = await this.$auth0.getAccessTokenSilently();
                const response = await fetch(query, {
                    headers: {
                        Authorization: `Bearer ${token}`
                    }
                });
                const data = await response.json();
                this.data = data.data;
                this.updatePolarAreaData();
            } else {
                const response = await fetch(query);
                const data = await response.json();
                this.data = data.data;
                this.updatePolarAreaData();
            }

            this.$emit('loaded', true)
        },

        resetFilter: function() {
            this.$emit('update-scmid', '');
        },

        sanitizeURL: function(url) {
            if (url === undefined) {
                return "";
            }

            url = url.replace(/https?:\/\//, '');
            url = url.replace(/http?:\/\//, '');
            url = url.replace(/github.com\//, '');
            url = url.replace(/gitlab.com\//, '');
            url = url.replace(/\/$/, '');
            url = url.replace(/\.git$/, '');

            return url;
        },

        getGitIcon: function(url) {
            url = url.replace(/https?:\/\//, '');
            url = url.replace(/http?:\/\//, '');

            if (url.startsWith('github.com')) {
                return 'mdi-github';
            } else if (url.startsWith('gitlab.com')) {
                return 'mdi-gitlab';
            } else if (url.startsWith('bitbucket.org')) {
                return 'mdi-bitbucket';
            } else {
                return 'mdi-git';
            }

        },

        getPolarAreaData(url, branch) {

            if (url === undefined || branch === undefined) {
                return {};
            }

            if (this.polarAreaData[url] === undefined) {
                return {};
            }

            if (this.polarAreaData[url][branch] === undefined) {
                return {};
            }

            return this.polarAreaData[url][branch];
        },

        updatePolarAreaData: function(){
            const labels = ['✔', '⚠', '✗', '-','?'];
            const labelColors = [
                'green',
                'orange',
                'red',
                'grey',
                'blue'];

            if (this.data === undefined) {
                return;
            }

            for (let url in this.data) {
                for ( let branch in this.data[url] ) {
                    if (this.data[url][branch].total_result === undefined) {
                        continue;
                    }

                    let successResults = 0;
                    let warningResults = 0;
                    let errorResults = 0;
                    let skippedResults = 0;
                    let otherResults = 0;

                    for ( let result in this.data[url][branch].total_result_by_type ) {
                        if (result === '✔') {
                            successResults = this.data[url][branch].total_result_by_type[result];
                        } else if (result === '✗') {
                            errorResults = this.data[url][branch].total_result_by_type[result];
                        } else if (result === '⚠') {
                            warningResults = this.data[url][branch].total_result_by_type[result];
                        } else if (result === '-') {
                            skippedResults = this.data[url][branch].total_result_by_type[result];
                        } else {
                            otherResults += this.data[url][branch].total_result_by_type[result];
                        }
                    }

                    if (this.polarAreaData[url] === undefined) {
                        this.polarAreaData[url] = {};
                    }

                    if (this.polarAreaData[url][branch] === undefined) {
                        this.polarAreaData[url][branch] = {};
                    }

                    if (this.data[url][branch].total_result === successResults) {

                        this.polarAreaData[url][branch] = {
                            labels: ['All is good'],
                            datasets: [
                                {
                                    data: [successResults],
                                    backgroundColor: [
                                        'green'
                                    ]
                                }
                            ]
                            }
                    } else {
                        this.polarAreaData[url][branch] = {
                            labels: labels,
                            datasets: [
                                {
                                    data: ["0",warningResults, errorResults, skippedResults, otherResults],
                                    backgroundColor: labelColors
                                }
                            ]
                        }
                    }
                }
            }
        },
    },
    async created() {
        try {
            this.getSummaryData();
        } catch(error) {
            console.log(error);
        }
    }
}
</script>
