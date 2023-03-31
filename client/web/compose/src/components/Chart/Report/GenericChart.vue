<template>
  <report-edit
    :report.sync="editReport"
    :modules="modules"
  >
    <!-- TODO: Cleanup text-primary, put them on highest level -->
    <template #y-axis="{ report }">
      <div
        class="px-3"
      >
        <h5 class="mb-3">
          {{ $t('edit.yAxis.label') }}
        </h5>

        <b-row
          class="text-primary"
        >
          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.yAxis.labelLabel')"
            >
              <b-input-group>
                <b-form-input
                  v-model="report.yAxis.label"
                />
                <b-input-group-append>
                  <chart-translator
                    :field.sync="report.yAxis.label"
                    :chart="chart"
                    :disabled="isNew"
                    highlight-key="yAxis.label"
                    button-variant="light"
                  />
                </b-input-group-append>
              </b-input-group>
            </b-form-group>
          </b-col>

          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.yAxis.labelPosition.label')"
            >
              <b-form-select
                v-model="report.yAxis.labelPosition"
                :options="axisLabelPositions"
              />
            </b-form-group>
          </b-col>

          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.yAxis.minLabel')"
            >
              <b-form-input
                v-model="report.yAxis.min"
                type="number"
                :placeholder="$t('edit.yAxis.minPlaceholder')"
              />
            </b-form-group>
          </b-col>

          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              horizontal
              :label-cols="3"
              class="mt-1"
              breakpoint="md"
              :label="$t('edit.yAxis.maxLabel')"
            >
              <b-form-input
                v-model="report.yAxis.max"
                type="number"
                :placeholder="$t('edit.yAxis.maxPlaceholder')"
              />
            </b-form-group>
          </b-col>

          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.yAxis.rotate.label')"
              :description="$t('edit.yAxis.rotate.description')"
            >
              <b-input
                v-model="report.yAxis.rotateLabel"
                type="number"
              />
            </b-form-group>
          </b-col>

          <!-- locale here -->
          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              label="Options"
            >
              <b-form-checkbox
                v-model="report.yAxis.axisType"
                value="logarithmic"
                unchecked-value="linear"
              >
                {{ $t('edit.yAxis.logarithmicScale') }}
              </b-form-checkbox>
              <b-form-checkbox
                v-model="report.yAxis.axisPosition"
                value="right"
                unchecked-value="left"
              >
                {{ $t('edit.yAxis.axisOnRight') }}
              </b-form-checkbox>
              <b-form-checkbox
                v-model="report.yAxis.beginAtZero"
              >
                {{ $t('edit.yAxis.axisScaleFromZero') }}
              </b-form-checkbox>
            </b-form-group>
          </b-col>
        </b-row>
      </div>
    </template>
    <hr>

    <template #metric-options="{ metric }">
      <b-row>
        <b-col
          v-if="!['pie', 'doughnut'].includes(metric.type)"
          cols="12"
          md="6"
        >
          <b-form-group
            class="mt-1 text-primary"
            :label="$t('edit.metric.labelLabel')"
          >
            <b-input-group>
              <b-form-input
                v-model="metric.label"
              />
              <b-input-group-append>
                <chart-translator
                  :field.sync="metric.label"
                  :chart="chart"
                  :disabled="isNew"
                  :highlight-key="`metrics.${metric.metricID}.label`"
                  button-variant="light"
                />
              </b-input-group-append>
            </b-input-group>
          </b-form-group>
        </b-col>

        <b-col
          cols="12"
          md="6"
        >
          <b-form-group
            class="text-primary"
            :label="$t('edit.metric.output.label')"
          >
            <b-form-select
              v-model="metric.type"
              :options="chartTypes"
            >
              <template slot="first">
                <option
                  :value="undefined"
                >
                  {{ $t('edit.metric.output.placeholder') }}
                </option>
              </template>
            </b-form-select>
          </b-form-group>
        </b-col>

        <b-col
          cols="12"
          md="6"
        >
          <b-form-group
            class="text-primary"
            :label="$t('edit.metric.fx.label')"
            :description="$t('edit.metric.fx.description')"
          >
            <b-form-textarea
              v-model="metric.fx"
              placeholder="n"
            />
          </b-form-group>
        </b-col>

        <b-col
          v-if="hasRelativeDisplay(metric)"
          cols="12"
          md="6"
        >
          <b-form-group
            :label="$t('edit.metric.relative')"
            class="text-primary"
          >
            <c-input-checkbox
              v-model="metric.relativeValue"
              switch
              :labels="checkboxLabel"
            />
          </b-form-group>
        </b-col>

        <b-col
          v-if="metric.type === 'line'"
          cols="12"
          md="6"
        >
          <b-form-group
            class="text-primary"
            :label="$t('edit.metric.fillArea')"
          >
            <c-input-checkbox
              v-model="metric.fill"
              switch
              :labels="checkboxLabel"
            />
          </b-form-group>
        </b-col>

        <b-col
          cols="12"
          md="6"
        >
          <b-form-group
            class="text-primary"
            :label="$t('edit.metric.fixTooltips')"
          >
            <c-input-checkbox
              v-model="metric.fixTooltips"
              switch
              :labels="checkboxLabel"
            />
          </b-form-group>
        </b-col>
      </b-row>
    </template>
    <hr>

    <template #additional-config="{ hasAxis, report }">
      <div
        class="px-3"
      >
        <h5 class="mb-3">
          {{ $t('edit.additionalConfig.legend.label') }}
        </h5>

        <b-row>
          <b-col
            v-if="!report.legend.isHidden"
            cols="12"
            md="6"
          >
            <b-form-group
              class="text-primary"
              :label="$t('edit.additionalConfig.legend.orientation.label')"
            >
              <b-form-select
                v-model="report.legend.orientation"
                :options="orientations"
              />
            </b-form-group>
          </b-col>

          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.additionalConfig.legend.hide')"
              class="text-primary"
            >
              <c-input-checkbox
                v-model="report.legend.isHidden"
                switch
                :labels="checkboxLabel"
              />
            </b-form-group>
          </b-col>
        </b-row>

        <b-row v-if="!report.legend.isHidden">
          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.additionalConfig.legend.align.label')"
              class="text-primary"
            >
              <b-form-select
                v-model="report.legend.align"
                :options="alignments"
                :disabled="!report.legend.position.isDefault"
              />
            </b-form-group>
          </b-col>

          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="'Options'"
              class="text-primary"
            >
              <b-form-checkbox
                v-model="report.legend.isScrollable"
                :disabled="report.legend.orientation !== 'horizontal'"
              >
                {{ $t('edit.additionalConfig.legend.scrollable') }}
              </b-form-checkbox>

              <b-form-checkbox
                v-model="report.legend.position.isDefault"
              >
                {{ $t('edit.additionalConfig.legend.position.customize') }}
              </b-form-checkbox>
            </b-form-group>
          </b-col>

          <b-row
            v-if="!report.legend.position.isDefault"
            class="px-3"
          >
            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.legend.position.top')"
                class="text-primary"
              >
                <b-input
                  v-model="report.legend.position.top"
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.legend.position.right')"
                class="text-primary"
              >
                <b-input
                  v-model="report.legend.position.right"
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.legend.position.bottom')"
                class="text-primary"
              >
                <b-input
                  v-model="report.legend.position.bottom"
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                class="text-primary"
                :label="$t('edit.additionalConfig.legend.position.left')"
                :description="$t('edit.additionalConfig.legend.valueRange')"
              >
                <b-input
                  v-model="report.legend.position.left"
                />
              </b-form-group>
            </b-col>
          </b-row>
        </b-row>
      </div>

      <template v-if="!hasAxis">
        <hr>
        <h4 class="mb-3">
          {{ $t('edit.additionalConfig.tooltip.label') }}
        </h4>

        <b-form-group
          horizontal
          :label-cols="3"
          breakpoint="md"
          :label="$t('edit.additionalConfig.tooltip.formatting.label')"
          :description="$t('edit.additionalConfig.tooltip.formatting.description')"
          class="mt-1"
        >
          <b-input
            v-model="report.tooltip.formatting"
            :placeholder="$t('edit.additionalConfig.tooltip.formatting.placeholder')"
          />
        </b-form-group>
        <b-form-group
          horizontal
          :label-cols="3"
          breakpoint="md"
        >
          <b-form-checkbox
            v-model="report.tooltip.labelsNextToPartition"
          >
            {{ $t('edit.additionalConfig.tooltip.labelNextToChartPartition') }}
          </b-form-checkbox>
        </b-form-group>
      </template>
      <hr>
      <div class="px-3">
        <h5 class="mb-3">
          {{ $t('edit.additionalConfig.offset.label') }}
        </h5>

        <b-row>
          <b-col
            cols="12"
            md="6"
          >
            <b-form-group
              :label="$t('edit.additionalConfig.offset.default')"
              class="text-primary"
            >
              <c-input-checkbox
                v-model="report.offset.isDefault"
                switch
                :labels="checkboxLabel"
                class="mb-3"
              />
            </b-form-group>
          </b-col>

          <b-row
            v-if="!report.offset.isDefault"
            class="text-primary px-3"
          >
            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.offset.position.top')"
              >
                <b-input
                  v-model="report.offset.top"
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.offset.position.right')"
              >
                <b-input
                  v-model="report.offset.right"
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.offset.position.bottom')"
              >
                <b-input
                  v-model="report.offset.bottom"
                />
              </b-form-group>
            </b-col>

            <b-col
              cols="12"
              md="6"
            >
              <b-form-group
                :label="$t('edit.additionalConfig.offset.position.left')"
                :description="$t('edit.additionalConfig.offset.valueRange')"
              >
                <b-input
                  v-model="report.offset.left"
                />
              </b-form-group>
            </b-col>
          </b-row>
        </b-row>
      </div>
    </template>
  </report-edit>
</template>

<script>
import ReportEdit from './ReportEdit'
import ChartTranslator from 'corteza-webapp-compose/src/components/Chart/ChartTranslator'
import { compose, NoID } from '@cortezaproject/corteza-js'
import base from './base'

const ignoredCharts = [
  'funnel',
  'gauge',
]

export default {
  name: 'GenericChart',

  i18nOptions: {
    namespaces: 'chart',
  },

  components: {
    ReportEdit,
    ChartTranslator,
  },

  extends: base,

  props: {
    chart: {
      type: compose.Chart,
      required: true,
    },
  },

  data () {
    return {
      chartTypes: Object.values(compose.chartUtil.ChartType)
        .filter(v => !ignoredCharts.includes(v))
        .map(value => ({ value, text: this.$t(`edit.metric.output.${value}`) })),

      legendPositions: [
        { value: 'top', text: this.$t('edit.metric.legend.top') },
        { value: 'left', text: this.$t('edit.metric.legend.left') },
        { value: 'bottom', text: this.$t('edit.metric.legend.bottom') },
        { value: 'right', text: this.$t('edit.metric.legend.right') },
      ],

      axisLabelPositions: [
        { value: 'end', text: this.$t('edit.yAxis.labelPosition.top') },
        { value: 'center', text: this.$t('edit.yAxis.labelPosition.center') },
        { value: 'start', text: this.$t('edit.yAxis.labelPosition.bottom') },
      ],

      tensionSteps: [
        { text: this.$t('edit.metric.lineTension.straight'), value: 0.0 },
        { text: this.$t('edit.metric.lineTension.slight'), value: 0.2 },
        { text: this.$t('edit.metric.lineTension.medium'), value: 0.4 },
        { text: this.$t('edit.metric.lineTension.curvy'), value: 0.6 },
      ],

      orientations: [
        { value: 'horizontal', text: this.$t('edit.additionalConfig.legend.orientation.horizontal') },
        { value: 'vertical', text: this.$t('edit.additionalConfig.legend.orientation.vertical') },
      ],

      alignments: [
        { value: 'left', text: this.$t('edit.additionalConfig.legend.align.left') },
        { value: 'center', text: this.$t('edit.additionalConfig.legend.align.center') },
        { value: 'right', text: this.$t('edit.additionalConfig.legend.align.right') },
      ],
    }
  },

  computed: {
    isNew () {
      return this.chart.chartID === NoID
    },
  },

  methods: {
    hasRelativeDisplay: compose.chartUtil.hasRelativeDisplay,
  },
}
</script>
<style lang="scss" scoped>
.color-picker {
  max-width: 50px;
}
</style>
