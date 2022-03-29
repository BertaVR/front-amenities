<template>
<body>
  <h1>Items</h1>

<Items/>  
</body>
</template>

<script>
// @ is an alias to /src
import Items from '../components/Items.vue'

export default {
  name: 'Home',
  components: {
    Items
}
}
</script>
