<template>
    <v-app>
    <v-container class="pa-6">
    <v-form @submit.prevent="onSubmit">
    <InputZone v-on:files-selected="logFiles"></InputZone>
            <v-text-field label='Input Path' required type="text" v-model="input_path" class="form-control"></v-text-field>
            <v-text-field label='Output Path' type="text" v-model="output_path" class="form-control"></v-text-field>
            <v-select :items='make_preset_list' required label='Preset Name' v-model='preset'></v-select>
            <v-btn class='mt-4 primary'  type='submit' elevation='2'>encode</v-btn>
    </v-form>
    </v-container>
    </v-app>
</template>


<script lang='ts'>
import Vue from "vue";
import axios from "axios";
import {WebsocketBuilder} from "websocket-ts";
import InputZone from './components/InputZone.vue'



interface presetData {
    preset_name: string;
    preset_cmd: string;
}
interface progressData {
   percent: string;
   fps: string;
   duration: string;
   frame: string;

}

interface progressObject {
    encoding: string;
    data: progressData;

}

interface jsonType {
    progress: string;
}

export default Vue.extend({
  name: "App",
  components:{
    InputZone,
  },

  data: () => ({
    progress_data: {'encoding': false, 'data': {}},  
    preset_data: [] as Array<presetData>,
    input_path: "",
    output_path: "",
    preset: "",
    formUpload: false as boolean,
    dragover: false as boolean,
    //
  }),
    created() {
        console.log("Starting connection to WebSocket Server");
        const ws = new WebsocketBuilder("ws://localhost:8000/current_encoding")
        .onOpen((ws, e) => { console.log('connected') })
        .onMessage((ws, e) => { this.check_json_data(e.data) })
        .build();
    },
    computed: {
        make_preset_list(){
        let preset_names = []
        let p: presetData;
            for (p of this.preset_data){
                preset_names.push(p.preset_name)
            }
        return preset_names

        }
    },
    methods: {
        logFiles(fileList: FileList) {
            for(const item of fileList) {
            // eslint-disable-next-line
                console.log(item.path)
            }
        },
        check_json_data(data: string){
            let json_data = JSON.parse(data)
            if (json_data.type == 'progress'){
                this.encode_this(json_data)
            }else if (json_data.type == 'presets') {
                this.set_presets(json_data.data.presets)
            }
        },
        set_presets(presets: Array<presetData>){
            this.preset_data = presets

        },
        encode_this(data: jsonType) {
            if(data.progress == 'continue'){
                this.progress_data.encoding = true
            } else if ( data.progress == 'end' ){
                this.progress_data.encoding = false
            }
            this.progress_data.data = data
        },
        onSubmit() {
            // upload file
            const formData = new FormData()
            formData.append('encoding_preset', this.get_preset_cmd(this.preset))
            formData.append('input_path', this.input_path)
            formData.append('output_path', this.output_path)
            axios.post('http://localhost:8000/add_new_job', formData, {
            }).then((res) => {
                console.log(res)
            })
            },
        get_preset_cmd(preset_name: string){
            let cmd = ''
            let p: presetData
            for(p of this.preset_data){
                if (preset_name == p.preset_name){
                    cmd = p.preset_cmd
                }
            }
            return cmd
        },
        
    },

});
</script>
