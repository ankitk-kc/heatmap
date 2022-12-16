<template>
    <div class="display_changes">
        <div v-if="!showChange">
            <h1>Select a box to view changes for that date</h1>
        </div>
        <div v-else>
            <h1>{{ changesMade.length }} changes on date {{ changesDone }}</h1>
            <ul>
                <li v-for="change in changesMade" :key="change.productId">{{ change.description }}</li>
            </ul>
        </div>
    </div>


</template>
<script lang="ts">
import changes from "../types/changes"
import { defineComponent, ref, watchEffect } from "vue";
export default defineComponent({
    props: ['changesDone'],
    setup(props) {

        let showChange = ref(false)
        let changesMade = ref<changes[]>([])

        watchEffect(() => {
            if (props.changesDone != '') {
                showChange.value = true
                fetch(`https://heatmapchanges.free.beeceptor.com/changes/${props.changesDone}`).then((response) => {
                    if (response.ok) {
                        return response.json();
                    }
                    throw new Error('Something went wrong');
                })
                    .then((result) => {
                        changesMade.value = result
                    })
                    .catch((error) => {
                        changesMade.value = []
                        console.log(error)
                    });
            }

        })
        return { changesMade, showChange }
    }
})
</script>
<style>
.display_changes{
    position: relative;
    margin: 32px 60px;
    padding-top: 20px;
    padding-right: 50px;
    padding-left: 50px;
    padding-bottom: 70px;
    border: 1px solid #fff;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
.display_changes h1{
    color: rgb(54, 8, 97);
}
.display_changes ul li{
    text-align: initial;
    margin-left: 123px;
    padding: 10px 15px;
}
</style>