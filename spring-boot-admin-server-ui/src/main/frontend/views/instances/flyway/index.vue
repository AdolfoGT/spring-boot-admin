<!--
  - Copyright 2014-2018 the original author or authors.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
    <section class="section" :class="{ 'is-loading' : !hasLoaded }">
        <div class="container" v-if="hasLoaded">
            <div v-if="error" class="message is-danger">
                <div class="message-body">
                    <strong>
                        <font-awesome-icon class="has-text-danger" icon="exclamation-triangle"></font-awesome-icon>
                        Fetching Flyway reports failed.
                    </strong>
                </div>
            </div>
            <sba-panel v-for="(report, name) in reports" :key="name" :title="name" class="migration">
                <table class="table">
                    <thead>
                    <tr>
                        <th>Type</th>
                        <th>Checksum</th>
                        <th>Version</th>
                        <th>Description</th>
                        <th>Script</th>
                        <th>State</th>
                        <th>Installed by</th>
                        <th>Installed on</th>
                        <th>Installed rank</th>
                        <th>Execution Time</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="migration in report.migrations" :key="migration.checksum">
                        <td v-text="migration.type"></td>
                        <td v-text="migration.checksum"></td>
                        <td v-text="migration.version"></td>
                        <td v-text="migration.description"></td>
                        <td v-text="migration.script"></td>
                        <td><span v-text="migration.state" class="tag"
                                  :class="stateClass(migration.state)"></span></td>
                        <td v-text="migration.installedBy"></td>
                        <td v-text="migration.installedOn"></td>
                        <td v-text="migration.installedRank"></td>
                        <td v-text="`${migration.executionTime}ms`"></td>
                    </tr>
                    </tbody>
                </table>
            </sba-panel>
        </div>
    </section>
</template>

<script>
  export default {
    props: ['instance'],
    data: () => ({
      hasLoaded: false,
      error: null,
      reports: []
    }),
    computed: {},
    created() {
      this.fetchFlyway();
    },
    watch: {
      instance() {
        this.fetchFlyway();
      }
    },
    methods: {
      async fetchFlyway() {
        if (this.instance) {
          this.error = null;
          try {
            const res = await this.instance.fetchFlyway();
            this.reports = res.data;
          } catch (error) {
            console.warn('Fetching flyway reports failed:', error);
            this.error = error;
          }
          this.hasLoaded = true;
        }
      },
      stateClass(state) {
        switch (state) {
          case 'BASELINE' :
          case  'MISSING_SUCCESS' :
          case  'SUCCESS' :
          case  'OUT_OF_ORDER' :
          case  'FUTURE_SUCCESS' :
            return 'is-success';
          case 'PENDING':
          case 'ABOVE_TARGET':
          case 'PREINIT':
          case 'BELOW_BASELINE':
          case 'IGNORED':
            return 'is-warning';
          case 'MISSING_FAILED':
          case 'FAILED':
          case 'FUTURE_FAILED':
            return 'is-danger';
          default:
            return 'is-light';
        }
      }
    }
  }
</script>

<style lang="scss">
    @import "~@/assets/css/utilities";

    .migration .card-header {
        position: sticky;
        background: $white;
        top: ($navbar-height-px + $tabs-height-px);
    }
</style>