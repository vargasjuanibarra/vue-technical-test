<template>
    <TopicsList />
</template>

<script>
import { defineComponent } from 'vue';
import TopicsList from './components/ItemsList.vue';

export default defineComponent({
  name: 'App',
  components: {
    TopicsList
  }
});
</script>
