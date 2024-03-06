<template>
  <div class="page home">
    <div class="full-width">
      <h1>Mixed Reality</h1>
      <ModelComponent />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import ModelComponent from '@/components/Model.vue';

export default defineComponent({
  name: 'HomeView',
  components: {
    ModelComponent,
  },
});
</script>

<style lang="scss" scoped>
.page.home {
  background-color: var(--black);
  height: 100vh;

  .full-width {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100%;
  }

  h1 {
    margin-top: 32px;
    margin-bottom: 16px;
    color: var(--white);
    text-align: center;
  }

  h2 {
    color: var(--white);
    font-weight: 200;
  }
}
</style>
