<template>
    <v-app>
    <v-container class="pa-6">
    <v-form @submit.prevent="onSubmit">
        <InputZone v-on:files-selected="setInputPath" :field_name="input_string"></InputZone>
            <div class="d-flex">
                <v-text-field label='Input Path' required type="text" v-model="input_path" class="form-control"></v-text-field>
                <v-checkbox label="Same Output Path" v-model="same_path"></v-checkbox>
            </div>
            <InputZone v-on:files-selected="setOutputPath" :field_name="output_string" v-if="!same_path"></InputZone>
            <v-text-field label='Output Path' type="text" v-model="output_path" class="form-control" v-if="!same_path"></v-text-field>
            <div class="d-flex">
                <v-select class="mx-2" :items='make_format_list' required label='Format' v-model='format'></v-select>
                <v-select class="mx-2" :items='make_codec_list' required label='V-Codec' v-model='v_codec' v-if="format != ''"></v-select>
                <v-select class="mx-2" :items='make_options_list' required label='Codec Options' v-model='v_option' v-if="v_codec != ''"></v-select>
            </div>


            <v-btn class='mt-4 primary'  type='submit' elevation='2' v-if="v_option != '' && input_path != ''">encode</v-btn>
    </v-form>
    </v-container>
    </v-app>
</template>



<script>
import Vue from "vue";
import axios from "axios";

import InputZone from './components/InputZone.vue'


export default Vue.extend({
  name: "App",
  components:{
    InputZone,
  },

  data: () => ({
    input_string: "Drop Input Here",
    output_string: "Drop Output Here",
    progress_data: {'encoding': false, 'data': {}},  
    presets: {},
    input_path: "",
    output_path: "",
    preset: "",
    formUpload: false,
    dragover: false,
    format: "",
    v_codec: "",
    v_option: "",
    same_path: true,
  }),
    created() {
        console.log("Starting connection to WebSocket Server");
        this.connection = new WebSocket("ws://localhost:8000/current_encoding");
        this.connection.onopen = (event) => {
            console.log("Successfully connected to the echo websocket server...");
            console.log(event)
        }
    },
    mounted() {
        this.connection.onmessage = (event) => {
            this.check_json_data(event.data);
        }
    },
    watch: {
        format: function (){
            this.v_option = "";
            this.v_codec = "";
        },
    },
    computed: {
        make_format_list(){
            if( this.presets['formats'] === undefined ){
                console.log('is undefined')
                return []
            }
            const keys = Object.keys(this.presets['formats'])
            let preset_names = []
            keys.forEach((key) => {
                    preset_names.push(this.presets['formats'][key]['format'])
                    });
            return preset_names
        },
        make_codec_list(){
            let codec_names = []
            if( this.presets['formats'] !== undefined && this.format != "" && (this.presets['formats'][this.format]['codecs'].includes(this.v_codec) || this.v_codec == "")){
                for (let p of this.presets['formats'][this.format]['codecs']){
                        codec_names.push(p)
                }
            }
            return codec_names
        },
        make_options_list(){
            let options_names = []
            if(this.presets['video_codecs'][this.v_codec] !== undefined) {
                if(this.presets['video_codecs'][this.v_codec]['video_profiles'].includes(this.v_option) || this.v_option == ""){
                    for (let p of this.presets['video_codecs'][this.v_codec]['video_profiles']){
                            options_names.push(p)
                    }
                }
            }
            return options_names
        }
    },
    methods: {
        logFiles(fileList) {
            for(const item of fileList) {
            // eslint-disable-next-line
                console.log(item.path)
            }
        },
        setOutputPath(fileList) {
            this.output_path = fileList[0].path
        },
        setInputPath(fileList) {
            this.input_path = fileList[0].path
        },
        check_json_data(data){
            let json_data = JSON.parse(data)
            if (json_data.type == 'progress'){
                this.encode_this(json_data)
            }else if (json_data.type == 'presets') {
                console.log(json_data.data)
                this.presets = json_data.data
            }
        },
        encode_this(data) {
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
          formData.append('encoding_preset', this.get_preset_cmd())
          formData.append('input_path', this.input_path)
          if(this.same_path){
            var patt = new RegExp("(.*\\/).*");
            formData.append('output_path', patt.exec(this.input_path)[1]);
          } else {
            formData.append('output_path', this.output_path)
          }
          axios.post('http://localhost:8000/add_new_job', formData, {
          }).then((res) => {
            console.log(res)
          })
        },
        get_preset_cmd(){
            let cmd = ''
            if( this.presets['video_codecs'][this.v_codec] !== undefined){
                let cmd_key = this.v_option.split('-')[0].trim()
                cmd = this.presets['video_codecs'][this.v_codec]['video_options_cmd'][cmd_key]
                cmd = cmd + ";" + this.format + ";" + this.v_codec + ";" + this.presets['video_codecs'][this.v_codec]['codec'];
            }
            return cmd
        },
    },
});
</script>


<style>
::-webkit-scrollbar {
    display: none;
}
</style>
