<template>
  <div class="index">
    <progressBar />
    <Header></Header>
    <mainRenderer ref="main"></mainRenderer>
    <submit
      :validate="validate"
      :renderData="renderData"
      @submit="onSubmit"
    ></submit>
  </div>
</template>

<script>
import Header from '../components/header.vue';
import submit from '../components/submit.vue';
import mainRenderer from '../components/mainRenderer';
import progressBar from '../components/progressBar';
import { submitForm } from '@/render/api/survey';
import encrypt from '../utils/encrypt';

export default {
  name: 'indexPage',
  props: {
    questionInfo: {
      type: Object,
      default: () => ({}),
    },
    isMobile: {
      type: Boolean,
      default: false,
    },
  },
  components: {
    Header,
    submit,
    mainRenderer,
    progressBar,
  },
  computed: {
    formModel() {
      return this.$store.getters.formModel;
    },
    confirmAgain() {
      return this.$store.state.submitConf.confirmAgain;
    },
    surveyPath() {
      return this.$store.state.surveyPath;
    },
    renderData() {
      return this.$store.getters.renderData;
    },
    encryptInfo() {
      return this.$store.state.encryptInfo;
    },
  },
  methods: {
    validate(cbk) {
      const index = 0;
      this.$refs.main.$refs.formGroup[index].$refs.ruleForm.validate(cbk);
    },
    onSubmit() {
      const { again_text, is_again } = this.confirmAgain;
      if (is_again) {
        this.$dialog.confirm({
          title: again_text,
          onConfirm: async (closeDialogFn) => {
            try {
              await this.submitForm();
            } catch (error) {
              console.error(error);
            } finally {
              closeDialogFn();
            }
          },
        });
      } else {
        this.submitForm();
      }
    },
    getSubmitData() {
      const result = {
        surveyPath: this.surveyPath,
        data: JSON.stringify(this.formModel),
        difTime: Date.now() - this.$store.state.enterTime,
        clientTime: Date.now(),
      };
      if (this.encryptInfo?.encryptType) {
        result.encryptType = this.encryptInfo?.encryptType;
        result.data = encrypt[result.encryptType]({
          data: result.data,
          secretKey: this.encryptInfo?.data?.secretKey,
        });
        if (this.encryptInfo?.data?.sessionId) {
          result.sessionId = this.encryptInfo.data.sessionId;
        }
      } else {
        result.data = JSON.stringify(result.data);
      }

      return result;
    },
    async submitForm() {
      try {
        const submitData = this.getSubmitData();
        const res = await submitForm(submitData);
        if (res.code === 200) {
          this.$store.commit('setRouter', 'successPage');
        } else {
          this.$dialog.alert({
            title: res.errmsg || '提交失败',
          });
        }
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<style scoped lang="scss">
.index {
  padding-bottom: 0.8rem;
}
</style>
