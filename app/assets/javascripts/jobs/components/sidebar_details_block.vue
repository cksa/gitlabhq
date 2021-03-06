<script>
import detailRow from './sidebar_detail_row.vue';
import loadingIcon from '../../vue_shared/components/loading_icon.vue';
import timeagoMixin from '../../vue_shared/mixins/timeago';
import { timeIntervalInWords } from '../../lib/utils/datetime_utility';

export default {
  name: 'SidebarDetailsBlock',
  components: {
    detailRow,
    loadingIcon,
  },
  mixins: [timeagoMixin],
  props: {
    job: {
      type: Object,
      required: true,
    },
    isLoading: {
      type: Boolean,
      required: true,
    },
    canUserRetry: {
      type: Boolean,
      required: false,
      default: false,
    },
    runnerHelpUrl: {
      type: String,
      required: false,
      default: '',
    },
  },
  computed: {
    shouldRenderContent() {
      return !this.isLoading && Object.keys(this.job).length > 0;
    },
    coverage() {
      return `${this.job.coverage}%`;
    },
    duration() {
      return timeIntervalInWords(this.job.duration);
    },
    queued() {
      return timeIntervalInWords(this.job.queued);
    },
    runnerId() {
      return `${this.job.runner.description} (#${this.job.runner.id})`;
    },
    retryButtonClass() {
      let className =
        'js-retry-button float-right btn btn-retry d-none d-md-block d-lg-block d-xl-block';
      className +=
        this.job.status && this.job.recoverable ? ' btn-primary' : ' btn-inverted-secondary';
      return className;
    },
    hasTimeout() {
      return this.job.metadata != null && this.job.metadata.timeout_human_readable !== null;
    },
    timeout() {
      if (this.job.metadata == null) {
        return '';
      }

      let t = this.job.metadata.timeout_human_readable;
      if (this.job.metadata.timeout_source !== '') {
        t += ` (from ${this.job.metadata.timeout_source})`;
      }

      return t;
    },
    renderBlock() {
      return (
        this.job.merge_request ||
        this.job.duration ||
        this.job.finished_data ||
        this.job.erased_at ||
        this.job.queued ||
        this.job.runner ||
        this.job.coverage ||
        this.job.tags.length ||
        this.job.cancel_path
      );
    },
  },
};
</script>
<template>
  <div>
    <div class="block">
      <strong class="inline prepend-top-8">
        {{ job.name }}
      </strong>
      <a
        v-if="canUserRetry"
        :class="retryButtonClass"
        :href="job.retry_path"
        data-method="post"
        rel="nofollow"
      >
        {{ __('Retry') }}
      </a>
      <button
        type="button"
        :aria-label="__('Toggle Sidebar')"
        class="btn btn-blank gutter-toggle float-right d-block d-md-none js-sidebar-build-toggle"
      >
        <i
          aria-hidden="true"
          data-hidden="true"
          class="fa fa-angle-double-right"
        ></i>
      </button>
    </div>
    <template v-if="shouldRenderContent">
      <div
        class="block retry-link"
        v-if="job.retry_path || job.new_issue_path"
      >
        <a
          v-if="job.new_issue_path"
          class="js-new-issue btn btn-new btn-inverted"
          :href="job.new_issue_path"
        >
          {{ __('New issue') }}
        </a>
        <a
          v-if="canUserRetry"
          class="js-retry-job btn btn-inverted-secondary"
          :href="job.retry_path"
          data-method="post"
          rel="nofollow"
        >
          {{ __('Retry') }}
        </a>
      </div>
      <div :class="{block : renderBlock }">
        <p
          class="build-detail-row js-job-mr"
          v-if="job.merge_request"
        >
          <span class="build-light-text">
            {{ __('Merge Request:') }}
          </span>
          <a :href="job.merge_request.path">
            !{{ job.merge_request.iid }}
          </a>
        </p>

        <detail-row
          class="js-job-duration"
          v-if="job.duration"
          title="Duration"
          :value="duration"
        />
        <detail-row
          class="js-job-finished"
          v-if="job.finished_at"
          title="Finished"
          :value="timeFormated(job.finished_at)"
        />
        <detail-row
          class="js-job-erased"
          v-if="job.erased_at"
          title="Erased"
          :value="timeFormated(job.erased_at)"
        />
        <detail-row
          class="js-job-queued"
          v-if="job.queued"
          title="Queued"
          :value="queued"
        />
        <detail-row
          class="js-job-timeout"
          v-if="hasTimeout"
          title="Timeout"
          :help-url="runnerHelpUrl"
          :value="timeout"
        />
        <detail-row
          class="js-job-runner"
          v-if="job.runner"
          title="Runner"
          :value="runnerId"
        />
        <detail-row
          class="js-job-coverage"
          v-if="job.coverage"
          title="Coverage"
          :value="coverage"
        />
        <p
          class="build-detail-row js-job-tags"
          v-if="job.tags.length"
        >
          <span class="build-light-text">
            {{ __('Tags:') }}
          </span>
          <span
            v-for="(tag, i) in job.tags"
            :key="i"
            class="label label-primary">
            {{ tag }}
          </span>
        </p>

        <div
          v-if="job.cancel_path"
          class="btn-group prepend-top-5"
          role="group">
          <a
            class="js-cancel-job btn btn-sm btn-default"
            :href="job.cancel_path"
            data-method="post"
            rel="nofollow"
          >
            {{ __('Cancel') }}
          </a>
        </div>
      </div>
    </template>
    <loading-icon
      class="prepend-top-10"
      v-if="isLoading"
      size="2"
    />
  </div>
</template>
