<template>
  <div>
    <!--<b-button @click="transition('nextState')">Go</b-button>-->    
   </div> 

</template>

<script>


export default {
    name: 'StateMachine',  
    data: function() {
        return {
            state: 'road',
            transitions: []
        }   
    },
    methods: {
        transition(nextState, method = null, params = []) {
            const transitionsArray = this.transitions[this.state];
        
            if(transitionsArray.indexOf(nextState) === -1) return this.state;
            if(method) method(...params);
            this.state = nextState;
            return this.state;
        }
    }
}
</script>

<style>

</style>