<template>
	<div>
		<h2>BOM Import</h2>
		<b-form class="border p-3 m-2">
			<b-input-group prepend="Sheetname" class="mt-3">
				<b-form-input v-model="sheetname" type="text" placeholder="BOM"></b-form-input>
			</b-input-group>
		
		
		<!-- Styled -->
		<b-form-file
		  :disabled="isSaving"
		  v-model="uploadFile"
		  :state="Boolean(uploadFile)"
		  placeholder="Choose a file or drop it here..."
		  drop-placeholder="Drop file here..."
		  @change="filesChange($event.target.name, $event.target.files); fileCount = $event.target.files.length"
		></b-form-file>
		<div class="mt-3">Selected file: {{ uploadFile ? uploadFile.name : '' }}</div>
		
		<p v-if="isInitial">
		  Click to browse
		</p>
		<p v-if="isSaving">
		  Uploading 1 files...
		</p>
		<p v-if="isSuccess">
			{{ msg }}
		</p>
		<p v-if="isFailed">
			{{ uploadError }}
		</p>
		</b-form>
	</div>
</template>

<script>

const BASE_URL = 'http://localhost:1880';

const STATUS_INITIAL = 0, STATUS_SAVING = 1, STATUS_SUCCESS = 2, STATUS_FAILED = 3;

module.exports = {
    props: {
        'extension': {type: String, default: '.bom'},
        'directory': {type: String, default: '.\bom'},
    },
    data: function() {
		return {
		    msg : '',
			sheetname: 'BOM',
			fileCount: 0,
			uploadFile: null,
			uploadError: null,
			currentStatus: STATUS_INITIAL,
			uploadFieldName: 'File',
		}
    }, // --- End of data --- //
    computed: {
        isInitial: function() {
		    return this.currentStatus === STATUS_INITIAL;
	    },
	    isSaving: function() {
		    return this.currentStatus === STATUS_SAVING;
	    },
	    isSuccess: function() {
		    return this.currentStatus === STATUS_SUCCESS;
	    },
	    isFailed: function() {
		    return this.currentStatus === STATUS_FAILED;
	    }
    }, // --- End of computed --- //
    methods: {
        save(formData) {
			// upload data to the server
			this.currentStatus = STATUS_SAVING;
		

			this.upload(formData)
			  .then(x => {
			    var topic = 'BOM';
			    uibuilder.send( {
                    'topic': topic,
                    'payload': {
                        'file': x.data.data,
                        'filename': this.uploadFile.name,
                        'sheetname': this.sheetname,
                    }
                })
                console.log(x);
                this.msg = x.data.response;
				this.currentStatus = STATUS_SUCCESS;
			  })
			  .catch(err => {
				this.uploadError = err.response;
				this.currentStatus = STATUS_FAILED;
			  });
	    },
        filesChange(fieldName, fileList) {
			// handle file changes
			const formData = new FormData();
			

			if (!fileList.length) return;

            if(!fileList[0].name.includes(this.extension)){
                alert(`Extension must be ${this.extension}`);
                this.uploadError = "Extension must be " + this.extension;
		        this.currentStatus = STATUS_FAILED;
		        this.uploadFile = "";
                return;
            }
        
			
			// append the files to FormData
			Array
			  .from(Array(fileList.length).keys())
			  .map(x => {
				formData.append(fieldName, fileList[x], fileList[x].name);
			  });

			// save it
			this.save(formData);
	    },
	    upload(formData) {
			const url = `${BASE_URL}/file-upload/BOM`;
			return axios.post(url, formData)
				// get data
				.then(function (response) {
                    return response;
                })
	
	    }
    }, // --- End of methods --- //
    watch: {
        sheetname: function(val, oldVal) {
            var topic = 'sheetname'
            uibuilder.send( {
                'topic': topic,
                'payload': {
                    'sheetname': this.sheetname,
                }                  
            })
        }
    },
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

 


    } // --- End of mounted hook --- //

} 
</script>

// EOF