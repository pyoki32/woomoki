<template>
  <div>
    <Banner/>
    <v-container>
      <div class="selectBoxes">
        <div class="selectBox1">
          <v-select
            color="#AED864"
            v-model="valSelect1"
            :items="select1"
            label="인기/신규"
            dense
            outlined
          ></v-select>
        </div>
        <div class="selectBox2">
          <v-select
            color="#AED864"
            v-model="valSelect2"
            :items="select2"
            label="카테고리"
            dense
            outlined
          ></v-select>
        </div>
      </div>
    </v-container>
    <div v-if="onSearch">
      <SeedSearched :option="valSelect1" :category="valSelect2" id="mainpage-component-result"/> 
    </div>
    <div v-else>
      <SeedRecommended id="mainpage-component-result"/>
    </div>
  </div>
</template>

<script>
import Banner from "@/views/MainPage/components/Banner.vue"
import SeedSearched from "@/views/MainPage/components/SeedSearched.vue"
import SeedRecommended from "@/views/MainPage/components/SeedRecommended.vue"
export default {
  name: 'MainPage',
  components: { Banner, SeedSearched, SeedRecommended },
  directives: {  },
  data() {
    return {
      select1: ['인기', '신규'],
      select2: ['전체', '건강', '생활습관', '독서', '자산', '자기계발', '취미'],
      valSelect1: '',
      valSelect2: '',
    }
  },
  mounted() {
    
  },
  computed: {
    onSearch: function () {
      if (this.valSelect1 && this.valSelect2) {
        return true
      } else {
        return false
      }
    }
  },
  methods: {
    
  },
  watch: {
    valSearch1(newVal, oldVal) {
      if (newVal !== oldVal) {
        return newVal
      }
    },
    valSearch2(newVal, oldVal) {
      if (newVal!== oldVal) {
        return newVal
      }
    }
  }
};
</script>

<style lang="scss" scoped>
div{
  .container{
    padding-bottom: 0;
    .selectBoxes {
      display: flex;
      justify-content: flex-end;
      padding-top: 2%;
      .selectBox1{
        margin-right: 0.5%;
        width: 10%;
      }
      .selectBox2{
        width: 10%;
      }
    }
  }
}
</style>