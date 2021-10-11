<template>
    <v-sheet
        id="input_zone"
        ref="inputzone"
        tabindex="0"
        title="Add Input File"
        color="primary"
        width="100"
        height="100"
        class="pa-2 input_zone"
        @dragover="dragover"
        @dragleave="dragleave"
        @drop="drop"
    >
        <input
        type="file"
        accept="text/xml"
        @change="onChange"
        ref="file"
        style="display:none"/>

    </v-sheet>
</template>

<script lang='ts'>
import Vue from "vue";

export default Vue.extend({
    name: "App",
    data: () => ({
        fileList: [] as Array<any>,
    }),
    methods: {
        filesSelected(fileList: FileList) {
            this.$emit("files-selected", fileList)
            console.log(fileList[0].path)
        },
        onChange() {
            this.filelist = [...this.$refs.file.files];
        },
        remove(i: number) {
            this.filelist.splice(i, 1);
        },
        dragover(event: DragEvent) {
            event.preventDefault();
            // Add some visual fluff to show the user can drop its files
            if (!event.currentTarget.classList.contains('bg-green-300')) {
                event.currentTarget.classList.remove('bg-gray-100');
                event.currentTarget.classList.add('bg-green-300');
            }
        },
        dragleave(event) {
            // Clean up
            event.currentTarget.classList.add('bg-gray-100');
            event.currentTarget.classList.remove('bg-green-300');
        },
        drop(event) {
            event.preventDefault();
            this.$refs.file.files = event.dataTransfer.files;
            this.onChange(); // Trigger the onChange event manually
            // Clean up
            event.currentTarget.classList.add('bg-gray-100');
            event.currentTarget.classList.remove('bg-green-300');
        }
    },
});

</script>
