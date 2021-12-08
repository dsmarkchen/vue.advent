<template>
   <div class="location">  
      <p>location: </p>
      <span>Name: {{name}} </span> <br>
      <span>Desc: {{description}}</span>
   </div>
</template>

<script>
export default {
  name: 'Location',
  props: ['name', 'description']
}
</script>

<style>

</style>