<template>
<div>
    <v-sheet
        id="input_zone"
        ref="inputzone"
        tabindex="0"
        title="Add Input File"
        width="100%"
        rounded="lg"
        class="py-6 mr-6 blue-grey lighten-5" 
        v-bind:class="[hasElement ? hasElementClass : '', noElementClass]"
        @dragover="dragover"
        @dragleave="dragleave"
        @drop="drop"
    >
        <input
        type="file"
        accept="image/*, video/*, audio/*"
        @change="onChange"
        ref="file"
        style="display:none"/>
        
        <v-btn absolute outlined x-small fab elevation="0" @click="remove_file" v-if="this.fileList.length"><v-icon>mdi-close</v-icon></v-btn>
        <v-container fill-height fluid>
            <v-row align="center" justify="center">
                <div class="d-flex align-center">
                    <v-icon class='mr-4 blue-grey--text'>
                        mdi-download
                    </v-icon>
                    <h3 class="blue-grey--text">{{ this.field_name }}</h3>
                </div>
            </v-row>
        </v-container>
    </v-sheet>
        <p v-if="this.fileList.length">{{ fileList[0].path }}</p>
    </div>
</template>

<script lang='ts'>
import Vue from "vue";

export default Vue.extend({
    name: "App",
    props: ['field_name'],
    data: () => ({
        fileList: [] as Array<any>,
        hasElement: false,
        hasElementClass: "darken-1",
        noElementClass: "lighten-5",
    }),
    methods: {
        onChange() {
            this.fileList = [...(this.$refs['file'] as any).files];
        },
        remove_file() {
            this.hasElement = false;
            this.fileList.pop();
        },
        dragover(event: DragEvent) {
            event.preventDefault();
            const target_el = event.currentTarget as HTMLInputElement
            if (!target_el.classList.contains('blue')) {
                target_el.classList.remove('blue-grey');
                target_el.classList.add('blue');
            }
        },
        dragleave(event: DragEvent) {
            // Clean up
            const target_el = event.currentTarget as HTMLInputElement
            target_el.classList.remove('blue');
            target_el.classList.add('blue-grey');
        },
        drop(event: DragEvent) {
            const data_trans = event.dataTransfer;
            if (data_trans !== null){
                (this.$refs['file'] as any).files = data_trans.files;
            }
            this.onChange(); // Trigger the onChange event manually
            this.$emit("files-selected", (this.$refs['file'] as any).files);
            this.hasElement = true;

            const target_el = event.currentTarget as HTMLInputElement
            target_el.classList.remove('blue');
            target_el.classList.add('blue-grey');
        }
    },
});

</script>

<style scoped>
    .dropzone_full {
        border: 3px solid green;
    }

</style>
