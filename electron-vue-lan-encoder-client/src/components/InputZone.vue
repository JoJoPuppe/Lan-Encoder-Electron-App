<template>
    <v-sheet
        id="input_zone"
        ref="inputzone"
        tabindex="0"
        title="Add Input File"
        width="100"
        outlined
        rounded="lg"
        height="100"
        class="pa-2 blue-grey" 
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
        <div class='d-flex align-center'>
            <p v-if="this.fileList.length">{{ fileList[0].path }}</p>
            <div class='center'>
                <v-icon>
                    mdi-download
                </v-icon>
                <h3 class="white--text">Input File</h3>
            </div>
            <v-btn fab @click="remove_file"><v-icon>mdi-close</v-icon></v-btn>

        </div>
    </v-sheet>
</template>

<script lang='ts'>
import Vue from "vue";

export default Vue.extend({
    name: "App",
    data: () => ({
        fileList: [] as Array<any>,
        hasElement: false,
        hasElementClass: "darken-1",
        noElementClass: "lighten-5",
    }),
    methods: {
        onChange() {
            this.fileList = [...(this.$refs['file'] as any).files];
            console.log(this.fileList)
        },
        remove_file() {
            this.hasElement = false;
            this.fileList.pop();
        },
        dragover(event: DragEvent) {
            event.preventDefault();
            const target_el = event.currentTarget as HTMLInputElement
            if (!target_el.classList.contains('primary')) {
                target_el.classList.remove('blue-grey');
                target_el.classList.add('primary');
            }
        },
        dragleave(event: DragEvent) {
            // Clean up
            const target_el = event.currentTarget as HTMLInputElement
            target_el.classList.add('blue-grey');
            target_el.classList.remove('primary');
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
            target_el.classList.remove('primary');
            target_el.classList.add('blue-grey');
        }
    },
});

</script>
