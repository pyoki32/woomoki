<template>
    <v-dialog
      v-model="dialog"
      max-width="290"
    >
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          class="mr-3 ml-3"
          color="black"
          icon
          v-bind="attrs"
          v-on="on"
          v-if="isLogin"
        >
          <v-icon>fas fa-share-alt</v-icon>
        </v-btn>
      </template>
      <v-card>
        <v-card-title class="d-flex justify-center">
          씨앗을 공유해보세요
        </v-card-title>
        <div class="fb-share-button d-flex justify-center" data-href="https://developers.facebook.com/docs/plugins/" data-layout="button_count" data-size="small">
          <a target="_blank" href="https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fdevelopers.facebook.com%2Fdocs%2Fplugins%2F&amp;src=sdkpreparse" class="fb-xfbml-parse-ignore">
            <v-btn class="mt-2" icon>
              <v-icon size="40" color="blue">fab fa-facebook-square</v-icon>
            </v-btn>
          </a>
        </div>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            text
            @click="dialog = false"
          >
            닫기
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
</template>

<script>
export default {
  name: "SeedShare",
  data: function () {
    return {
      dialog: false,
      isLogin : this.$store.state.UserStore.isLogin
    }
  }
}
</script>

<style lang="scss" scoped>

a { text-decoration: none; }

</style>