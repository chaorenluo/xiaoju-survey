<template>
  <div class="result-config-preview">
    <div class="status-preview">
      <component
        :is="currentEditStatus"
        :key="currentEditStatus"
        :module-config="moduleConfig"
      />
    </div>
  </div>
</template>
<script>
import { mapState } from 'vuex';
import success from './components/success';
import overTime from './components/overTime';
import { EDIT_STATUS_MAP } from './enum';
import { get as _get } from 'lodash-es';

export default {
  name: 'ResultConfigPreivew',
  props: {},
  data() {
    return {};
  },
  computed: {
    ...mapState({
      currentEditStatus: (state) => state.edit.currentEditStatus,
      submitConf: (state) => _get(state, 'edit.schema.submitConf'),
    }),
    moduleConfig() {
      return {
        submitConf: this.submitConf,
      };
    },
  },
  components: {
    [EDIT_STATUS_MAP.SUCCESS]: success,
    [EDIT_STATUS_MAP.OVERTIME]: overTime,
  },
};
</script>
<style lang="scss" rel="stylesheet/scss" scoped>
.result-config-preview {
  width: 100%;
  height: 100%;
  min-width: 500px;
  padding-bottom: 80px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f6f7f9;
}

.status-preview {
  width: 90%;
  margin-top: 38px;
  min-height: 812px;
  max-height: 812px;
  overflow-x: hidden;
  overflow-y: auto;
  background-color: #fff;
}
</style>
