<template>
	<div>
		<h2>Download Import </h2>
		<b-form class="border p-3 m-2">
		<p>Kardex import directory</p>
		
		<b-input-group prepend="Network file path" class="mt-3">
			<b-form-input :disabled="!enabled" v-model="updatedPath" :state="pathState" type="text" :placeholder="path" @input="pathChange"></b-form-input>
            <b-form-invalid-feedback id="path-feedback">
                Invalid path - must be of the form '\\server\path'!
            </b-form-invalid-feedback>
		</b-input-group>
		<b-input-group prepend="Job import directory" class="mt-3">
			<b-form-input :disabled="!enabled" v-model="updatedJobFolder" :state="jobState" type="text" :placeholder="job_folder"></b-form-input>
			<b-form-invalid-feedback id="job-feedback">
                Invalid path - letters numbers and underscore only!
            </b-form-invalid-feedback>
		</b-input-group>
		<b-input-group prepend="Bin import directory" class="mt-3">
			<b-form-input :disabled="!enabled" v-model="updatedBinFolder" :state="binState" type="text" :placeholder="bin_folder"></b-form-input>
			<b-form-invalid-feedback id="bin-feedback">
                Invalid path - letters numbers and underscore only!
            </b-form-invalid-feedback>
		</b-input-group>
		<b-input-group prepend="Kit import directory" class="mt-3">
			<b-form-input :disabled="!enabled" v-model="updatedKitFolder" :state="kitState" type="text" :placeholder="kit_folder" ></b-form-input>
			<b-form-invalid-feedback id="kit-feedback">
                Invalid path - letters numbers and underscore only!
            </b-form-invalid-feedback>
		</b-input-group>
		
		<p></p>
		<b-button :disabled="!enabled" :variant="getState" v-on:click="download">{{ button_text }}</b-button>
        </b-form>
        
        <p></p>
        {{ enabled }}
	</div>
</template>

<script>

const BASE_URL = 'http://localhost:1880';

const STATE_DISABLED = "secondary", STATE_ENABLED = "primary", STATE_SAVING = "warning", STATE_SUCCESS = "success", STATE_FAILED = "danger";

module.exports = {
    props: {
        'enabled': {type: Boolean, default: false},
        'path': {type: String, default: '*\\\\itm01\\office\\Purchasing\\FastPic\\'},
        'job_folder': {type: String, default: '*Job_Import'},
        'bin_folder': {type: String, default: '*Bin_Import'},
        'kit_folder': {type: String, default: '*Kit_Import'},
    },
    directives: {
        init: {
            bind: function(el, binding, vnode) {
                vnode.context[binding.arg] = binding.value;
            }
        }
    },
    data: function() {
		return {
		    msg : '',
		    button_text: 'Download',
			uploadedFiles: [],
			uploadError: null,
			currentStatus: null,
			uploadFieldName: 'File',
			updatedPath : '',
			pathState: null,
			updatedKitFolder : '',
			kitState : null,
			updatedJobFolder : '',
			jobState : null,
			updatedBinFolder : '',
			binState : null,
		}
    }, // --- End of data --- //
    computed: {
        getState: function() {
            return this.currentState;
	    },
	    isEnabled: function() {
	        return Boolean(this.enabled);
	    },
	    getPath: function() {
	        return this.path;
	    }
    }, // --- End of computed --- //
    methods: {
        download() {
			// upload data to the server
			this.currentStatus = STATE_SAVING;
			
			var new_path = this.path;
			if (this.updatedPath.length > 0) new_path = this.updatedPath;
			var new_kitFolder = this.kit_folder;
			if (this.updatedKitFolder.length > 0) new_path = this.updatedKitFolder;
			var new_jobFolder = this.job_folder;
			if (this.updatedJobFolder.length > 0) new_path = this.updatedJobFolder;
			var new_binFolder = this.bin_folder;
			if (this.updatedBinFolder.length > 0) new_path = this.updatedBinFolder;
			
			if (new_path.match(/^(\\)(\\[A-Za-z0-9-_]+){2,}$/) === null){
			    this.pathState = false;
			} else {
			    this.pathState = true;
			}
			if(new_kitFolder.match(/^([A-Za-z0-9-_]+[^\\]$)/) === null){
			    this.kitState = false;
			} else {
			    this.kitState = true;
			}
			if(new_jobFolder.match(/^([A-Za-z0-9-_]+[^\\]$)/) === null){
			    this.jobState = false;
			} else {
			    this.jobState = true;
			}
			if(new_binFolder.match(/^([A-Za-z0-9-_]+[^\\]$)/) === null){
			    this.binState = false;
			} else {
                this.binState = true;
			}
			    
			if (this.pathState && this.kitState && this.jobState && this.binState){   
			    var topic = 'download';
			    uibuilder.send( {
                    'topic': topic,
                    'payload': {
                        'path': new_path,
                        'kitFolder': new_kitFolder,
                        'jobFolder': new_jobFolder,
                        'binFolder': new_binFolder
                    }
                })
			    
			}

	    },
        pathChange(evt) {
			// handle file changes
			if (this.updatedPath = '') this.updatedPath = this.getPath;

        },
    }, // --- End of methods --- //
    watch: {
       
    },	// --- End of watches --- //
    // Available hooks: init,mounted,updated,destroyed
    mounted: function(){
        //console.debug('[indexjs:Vue.mounted] app mounted - setting up uibuilder watchers')

        /** **REQUIRED** Start uibuilder comms with Node-RED @since v2.0.0-dev3
         * Pass the namespace and ioPath variables if hosting page is not in the instance root folder
         * The namespace is the "url" you put in uibuilder's configuration in the Editor.
         * e.g. If you get continual `uibuilderfe:ioSetup: SOCKET CONNECT ERROR` error messages.
         * e.g. uibuilder.start('uib', '/nr/uibuilder/vendor/socket.io') // change to use your paths/names
         */
        uibuilder.start()
        //uibuilder.debug(true)
        uibuilder.autoSendReady(true)
        

        var vueApp = this

        /** You can use the following to help trace how messages flow back and forth.
         * You can then amend this processing to suite your requirements.
         */

 


    }, // --- End of mounted hook --- //
    init: function(){
        this.updatedPath = this.path;
    }

} 
</script>

// EOF