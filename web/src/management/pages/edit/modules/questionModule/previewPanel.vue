<template>
  <div class="main-operation" @click="onMainClick" ref="mainOperation">
    <!-- <div class="toolbar"></div> -->
    <div class="operation-wrapper" ref="operationWrapper">
      <div class="box content" ref="box">
        <banner
          :bannerConf="bannerConf"
          :is-selected="currentEditOne === 'banner'"
          @select="onSelectEditOne('banner')"
        />
        <mainTitle
          :bannerConf="bannerConf"
          :is-selected="currentEditOne === 'mainTitle'"
          @select="onSelectEditOne('mainTitle')"
          @change="handleChange"
        />
        <materialGroup
          :current-edit-one="parseInt(currentEditOne)"
          :questionDataList="questionDataList"
          @select="onSelectEditOne"
          @change="handleChange"
          @changeSeq="onQuestionOperation"
          ref="materialGroup"
        />
        <submit
          :submit-conf="submitConf"
          :skin-conf="skinConf"
          :is-selected="currentEditOne === 'submit'"
          @select="onSelectEditOne('submit')"
        />
        <logo
          :bottom-conf="bottomConf"
          :is-selected="currentEditOne === 'logo'"
          @select="onSelectEditOne('logo')"
        />
      </div>
    </div>
  </div>
</template>

<script>
import materialGroup from './components/materialGroup.vue';
import banner from './components/banner.vue';
import mainTitle from './components/mainTitle.vue';
import submit from './components/submit.vue';
import logo from './components/logo.vue';
import { mapState, mapGetters } from 'vuex';
import { get as _get } from 'lodash-es';

export default {
  name: 'mainOperation',
  components: {
    banner,
    mainTitle,
    submit,
    logo,
    materialGroup,
  },
  data() {
    return {
      isAnimating: false,
    };
  },
  computed: {
    ...mapState({
      bannerConf: (state) => _get(state, 'edit.schema.bannerConf'),
      submitConf: (state) => _get(state, 'edit.schema.submitConf'),
      skinConf: (state) => _get(state, 'edit.schema.skinConf'),
      bottomConf: (state) => _get(state, 'edit.schema.bottomConf'),
      questionDataList: (state) => _get(state, 'edit.schema.questionDataList'),
      currentEditOne: (state) => _get(state, 'edit.currentEditOne'),
    }),
    ...mapGetters({
      currentEditKey: 'edit/currentEditKey',
    }),
    autoScrollData() {
      return {
        currentEditOne: this.currentEditOne,
        len: this.questionDataList.length,
      };
    },
  },
  watch: {
    autoScrollData(newVal) {
      const { currentEditOne } = newVal;
      if (typeof currentEditOne === 'number') {
        setTimeout(() => {
          // if (this.isAnimating) {
          //   return;
          // }
          const field = this.questionDataList?.[currentEditOne]?.field;
          if (field) {
            const questionComp =
              this.$refs.materialGroup.getQuestionRefByField(field);
            if (questionComp && questionComp.$el) {
              questionComp.$el.scrollIntoView({
                behavior: 'smooth',
              });
              // this.isAnimating = true;
              // const maxScrollTop = this.$refs.box.clientHeight - this.$refs.operationWrapper.clientHeight
              // const targetVal = Math.min(questionComp.$el.offsetTop - this.$refs.operationWrapper.clientHeight / 2, maxScrollTop)
              // this.animate(this.$refs.operationWrapper, 'scrollTop', targetVal)
            }
          }
        }, 0);
      }
    },
  },
  methods: {
    animate(dom, property, targetValue) {
      const origin = dom[property];
      const subVal = targetValue - origin;

      const flag = subVal < 0 ? -1 : 1;

      const step = flag * 50;

      const totalCount = Math.floor(subVal / step) + 1;

      let runCount = 0;
      const run = () => {
        dom[property] += step;
        runCount++;
        if (runCount < totalCount) {
          requestAnimationFrame(run);
        } else {
          this.isAnimating = false;
        }
      };

      requestAnimationFrame(run);
    },
    async onSelectEditOne(currentEditOne) {
      this.$store.commit('edit/setCurrentEditOne', currentEditOne);
    },
    handleChange(data) {
      if (this.currentEditOne === null) {
        return;
      }
      const { key, value } = data;
      const resultKey = `${this.currentEditKey}.${key}`;
      this.$store.dispatch('edit/changeSchema', { key: resultKey, value });
    },
    onMainClick(e) {
      if (e.target === this.$refs.mainOperation) {
        this.$store.commit('edit/setCurrentEditOne', null);
      }
    },
    onQuestionOperation(data) {
      switch (data.type) {
        case 'move':
          this.$store.dispatch('edit/moveQuestion', {
            index: data.index,
            range: data.range,
          });
          break;
        case 'delete':
          this.$store.dispatch('edit/deleteQuestion', { index: data.index });
          break;
        case 'copy':
          this.$store.dispatch('edit/copyQuestion', { index: data.index });
          break;
        default:
          break;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.main-operation {
  width: 100%;
  height: 100%;
  min-width: 500px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f6f7f9;
}

.toolbar {
  width: 100%;
  height: 38px;
  background-color: #fff;
  flex-grow: 0;
  flex-shrink: 0;
}

.operation-wrapper {
  margin-top: 38px;
  margin-bottom: 45px;
  // min-height: 812px;
  overflow-x: hidden;
  overflow-y: auto;
  padding-right: 30px;
  margin-right: -30px;
  scrollbar-width: none;
  width: 90%;
  -ms-overflow-style: none;

  &::-webkit-scrollbar {
    display: none;
  }

  .content {
    background-color: #fff;
  }
}
</style>
