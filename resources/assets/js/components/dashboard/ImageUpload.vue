<template>
    <div class="relative">
        <div v-if="resourcePath !== null">
            <img
                :src="resourcePath"
                class="max-w-full mx-auto rounded border"
                @mouseover="imageHover = true"
            >
        </div>
        <div v-if="resourcePath === null" class="p-8 rounded border border-dashed border-grey-darkest bg-grey-light">
            <p class="text-center">Click or drag file here to upload.</p>
        </div>
        <form
            action="/imageupload/"
            method="POST"
            v-if="imageHover"
            enctype="multipart/form-data"
        >
            <label
                :for="resourceName"
                class="bg-brand absolute w-full font-semibold text-xl text-white opacity-0 border p-8 text-center flex items-center justify-center rounded shadow"
                :class="{'pin': imageHover, 'opacity-75': imageHover}"
                @mouseout="imageHover = false"
            >
                <!-- <span class="opacity-100 text-sm cursor-pointer">Click here to upload a new photo</span> -->
                <input
                    type="file"
                    :name="resourceName"
                    class="hidden"
                    :id="resourceName"
                    @change="fileChanged(
                        $event.target.name,
                        $event.target.files
                    )"
                >
            </label>
        </form>
    </div>
</template>

<script>
    import { upload } from '../../services/file-upload.service.js';
    const STATUS_INITIAL = 0, STATUS_SAVING = 1, STATUS_SUCCESS = 2, STATUS_FAILED = 3;

    export default {
        props: {
            resourcePath: {
                type: String,
                default: null
            },
            resourceName: {
                type: String,
                default: this.resourceName
            }
        },
        data () {
            return {
                uploadedFiles: [],
                uploadError: null,
                currentStatus: null,
                src: null,
                imageHover: false
            }
        },
        computed: {
            uploadFileName() {
                return this.resourceName;
            },
            isInitial() {
                return this.currentStatus === STATUS_INITIAL;
            },
            isSaving() {
                return this.currentStatus === STATUS_SAVING;
            },
            isSuccess() {
                return this.currentStatus === STATUS_SUCCESS;
            },
            isFailed() {
                return this.currentStatus === STATUS_FAILED;
            }
        },
        mounted() {
            this.reset();
            this.src = this.resourcePath;
        },
        methods: {
            reset() {
                this.currentStatus = STATUS_INITIAL;
                this.uploadedFiles = [];
                this.uploadError   = null;
            },
            save(formData) {
                this.currentStatus = STATUS_SAVING;
                upload(formData)
                    .then(response => {
                        this.uploadedFiles = [].concat(response);
                        this.currentStatus = STATUS_SUCCESS;
                        this.src = response;
                    })
                    .catch(err => {
                        this.uploadError = err;
                        this.currentStatus = STATUS_FAILED;
                    });
            },
            fileChanged(fieldName, fileList) {
                const formData = new FormData();
                if (! fileList.length) {
                    return;
                }
                Array.from(Array(fileList.length).keys())
                    .map(x => {
                        formData.append(fieldName, fileList[x], fileList[x].name);
                    });

                this.save(formData);
            }
        }
    }
</script>