<template>
    <panel class="advances">
        <page-load class="no-padding" :loading="loading.months">
            <el-tabs v-model="tabs" @tab-click="loadTab">
                <el-tab-pane label="Overview" name="overview">
                    <page-load class="no-padding" :loading="loading.overview">
                        <advance-experience-chart :experience="experience"/>
                    </page-load>
                </el-tab-pane>

                <el-tab-pane :label="month.label" :name="`${index}`" :key="index" v-for="month, index in months">
                    <page-load class="no-padding" :loading="loading.experience">
                        <div v-if="experience.length">
                            <month-cards :experience="experience" :month="month"/>

                            <table class="table advances">
                                <tr v-for="advance in experience">
                                    <td width="130">
                                        <b>{{ advance.updated_at | date }}</b>
                                        <span>{{ advance.updated_at | dateForHuman }}</span>
                                    </td>
                                    <td class="level">
                                        <span>Level</span>
                                        <b>{{ advance.level }}</b>

                                        <el-progress :percentage="getLeftExperience(advance)" :show-text="false"/>
                                    </td>
                                    <td>
                                        <b>{{ advance.experience.format() }}</b>
                                        <span>Experience</span>
                                    </td>
                                    <td class="advance">
                                        <div v-if="advance.advance >= 0">
                                            <b>+{{ advance.advance.format() }}</b>
                                            <span>+ Experience</span>
                                        </div>

                                        <div v-else>
                                            <b class="loose">{{ advance.advance.format() }}</b>
                                            <span>+ Experience</span>
                                        </div>
                                    </td>
                                </tr>
                            </table>
                        </div>

                        <div class="alert alert-warning margin-bottom-0" v-else>
                            <p>No experience records found, we only track players from highscores players.</p>
                        </div>
                    </page-load>
                </el-tab-pane>
            </el-tabs>
        </page-load>
    </panel>
</template>

<script>
    import AdvanceExperienceChart from './experience/AdvanceExperienceChart'
    import MonthCards from './experience/MonthCards'

    import services from '../services'

    export default {
        components: { AdvanceExperienceChart, MonthCards },

        data () {
            return {
                tabs: 0,
                loading: {
                    months: false,
                    overview: false,
                    experience: false
                }
            }
        },

        computed: {
            player () {
                return this.$store.getters['player/GET_PLAYER'].details
            },

            months () {
                return this.$store.getters['player/GET_MONTHS']
            },

            experience () {
                return this.$store.getters['player/GET_EXPERIENCE']
            },

            monthExperience () {
                const firstOfMonth = this.experience[this.experience.length - 1]
                const lastOfMonth = this.experience[0]
                return this.experience.length ? lastOfMonth.experience - firstOfMonth.experience : 0
            },

            monthLevels () {
                const firstOfMonth = this.experience[this.experience.length - 1]
                const lastOfMonth = this.experience[0]
                return this.experience.length ? lastOfMonth.level - firstOfMonth.level : 0
            }
        },

        filters: {
            date (date) {
                return moment(date).format('DD MMM YYYY')
            },

            dateForHuman (date) {
                return moment.tz(date, "YYYY-MM-DD HH:mm:ss", 'America/New_York').fromNow()
            }
        },

        methods: {
            load () {
                this.loading.months = true
                this.$store.dispatch('player/FETCH_MONTHS')
                    .then(() => {
                        this.loadExperience(0)
                        this.loading.months = false
                    })
                    .catch(() => this.loading.months = false)
            },

            loadTab (tab) {
                if (tab.name == 'overview') {
                    this.loadOverview()
                    return false
                }

                this.loadExperience(tab.name)
            },

            loadOverview () {
                const name = this.player.name

                this.loading.overview = true
                this.$store.dispatch('player/FETCH_OVERVIEW', { name })
                    .then(() => this.loading.overview = false)
                    .catch(() => this.loading.overview = false)
            },

            loadExperience (index) {
                const name = this.player.name
                const month = this.months[index].date

                this.loading.experience = true
                this.$store.dispatch('player/FETCH_EXPERIENCE', { name, month })
                    .then(() => this.loading.experience = false)
                    .catch(() => this.loading.experience = false)
            },

            getLeftExperience (advance) {
                const nextLevel = advance.level + 1
                const nextLevelExp = ((50 * Math.pow((nextLevel - 1), 3)) - (150 * Math.pow((nextLevel - 1), 2)) + (400 * (nextLevel - 1))) / 3
                const currentExp = advance.experience
                const expToNextLevel = 50 * Math.pow(advance.level, 2) - 150 * advance.level + 200
                const expLeft = nextLevelExp - currentExp
                const currentLeveledExp = expToNextLevel - expLeft
                const percentage = parseInt((currentLeveledExp * 100) / expToNextLevel)

                return this.experience ? percentage : 0
            },
        },

        mounted () {
            this.load()
        }
    }
</script>
