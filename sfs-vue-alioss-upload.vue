<template>
    <div class="sfs-upload-box">
        <div class="sfs-vue-oss_file_box_control" v-show="showUI">
            <h4 v-show="upload_title">{{upload_title}}</h4>
            <a id="selectfiles" href="" class="sfs-ali-vue-a">选择文件</a>
            <a @click="uploader.start()" v-if="fileList.length > 0" class="sfs-ali-vue-a">开始上传</a>
        </div>
        <div v-show="showProgress">
            <div ref="ossFiles" class="sfs-vue-oss_file_box">
                <div class="sfs-vue-ali-item" v-for="item in fileList">
                    <a>{{item.name}} <br> ({{Math.floor(item.size/1024) + 'KB'}}) </a>
                    <div class="sfs-vue-ali-progress">
                        <div class="sfs-vue-ali-progress-bar" v-bind:ref="'progress_'+item.id" style="width: 0"><span
                                v-bind:ref="'progress_txt_'+item.id"></span></div>
                    </div>
                    <a @click="removeFile" :data-uid="item.id" class="sfs-vue-oss_file_box_close"><img
                            :data-uid="item.id"
                            src="data:image/png;base64,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"/>
                    </a>
                </div>
            </div>
            <div ref="container"></div>
        </div>
    </div>
</template>

<script>
    import plupload from 'plupload'

    export default {
        name: "sfs-vue-alioss-upload",
        data() {
            return {
                fileList: [],
                ossFile: '',
                accessid: '',
                accesskey: '',
                callbackbody: '',
                host: '',
                policyBase64: '',
                signature: '',
                filename: '',
                expire: 0,
                now: Date.parse(new Date()) / 1000,
                uploader: null,
                uploadMethod: '',
                g_object_name: '',
            };
        },
        props: {
            upload_title: {
                default: 'sfs-vue-upload'
            },
            oss_dir: {
                default: ''
            },
            use_real_name: {
                default: false
            },
            showUI: {
                default: true
            },
            showProgress: {
                default: true
            },
            fileId: {
                default: ''
            },
            openId: {
                default: ''
            },
            authServerUrl: {
                default: ''
            },
            extensions: {
                default: 'jpg,png,jpeg,mp4,mov,MP4,MOV'
            },
            max_file_size: {
                default: '500mb'
            },
            prevent_duplicates: {
                default: true
            },
            onBeforeUpload: {
                default: Function
            },
            onFilesAdded: {
                default: Function
            },
            onFileUploaded: {
                default: Function
            },
            onSuccess: {
                default: Function
            },
            onError: {
                default: Function
            },
            onProgress: {
                default: Function
            },
            onInit: {
                default: Function
            },
        },
        beforeCreate() {

        },
        mounted() {
            this.upload()
        },
        methods: {
            removeFile(e) {
                let uid = e.target.dataset.uid
                console.log(uid)
                for (var i = 0; i < this.fileList.length; i++) {
                    if (this.fileList[i].id === uid) {
                        this.uploader.removeFile(this.fileList[i])
                        this.fileList.splice(i, 1)
                    }
                }
            },
            sendRequest() {
                const xmlhttp = new XMLHttpRequest();
                xmlhttp.open('GET', this.authServerUrl, false);
                xmlhttp.send(null);
                return xmlhttp.responseText;
            },
            getSignature() {
                // 可以判断当前expire是否超过了当前时间,如果超过了当前时间,就重新取一下.3s 做为缓冲
                this.now = Date.parse(new Date()) / 1000;
                if (this.expire < this.now + 3) {
                    const body = this.sendRequest();
                    const e = eval;
                    const obj = e(`(${body})`);
                    this.host = obj.host;
                    this.policyBase64 = obj.policy;
                    this.accessid = obj.accessid;
                    this.signature = obj.signature;
                    this.expire = parseInt(obj.expire, 10);
                    this.callbackbody = obj.callback;
                    this.key = this.g_object_name;
                    return true;
                }
                return false;
            },
            randomString(len = 32) {
                const chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';
                const maxPos = chars.length;
                let pwd = '';
                for (let i = 0; i < len; i += 1) {
                    pwd += chars.charAt(Math.floor(Math.random() * maxPos));
                }
                return pwd;
            },
            getSuffix(filename) {
                const pos = filename.lastIndexOf('.');
                let suffix = '';
                if (pos !== -1) {
                    suffix = filename.substring(pos);
                }
                return suffix;
            },
            calculateObjectName(filename) {
                if (this.use_real_name === true) {
                    this.g_object_name = this.oss_dir + `${filename}`;
                } else if (this.use_real_name === false) {
                    const suffix = this.getSuffix(filename);
                    this.g_object_name = this.oss_dir + this.randomString(10) + suffix;
                }
                return this.g_object_name;
            },
            setUploadParam(up, file, ret) {
                let filename = file.name
                if (ret === false) {
                    this.getSignature();
                }
                if (filename !== '') {
                    file.oss_name = this.calculateObjectName(filename);
                }
                const newMultipartParams = {
                    key: this.g_object_name,
                    policy: this.policyBase64,
                    OSSAccessKeyId: this.accessid,
                    // 让服务端返回200,不然，默认会返回204
                    success_action_status: '200',
                    signature: this.signature,
                    callback: this.callbackbody,
                }
                up.setOption({
                    url: this.host,
                    multipart_params: newMultipartParams,
                });
                up.start();
            },
            upload() {
                let that = this
                console.log(that.extensions)
                let myPlupload = new plupload.Uploader({
                    runtimes: 'html5,flash,silverlight,html4',
                    browse_button: 'selectfiles',
                    multi_selection: false,
                    container: that.$refs.container,
                    url: 'http://oss.aliyuncs.com',
                    filters: {
                        mime_types: [{
                            title: '允许上传文件类型',
                            extensions: that.extensions,
                        }],
                        max_file_size: that.max_file_size,
                        prevent_duplicates: that.prevent_duplicates,
                    },
                    init: {
                        PostInit: () => {
                            that.uploader = myPlupload
                            console.log('sfs-vue-alioss init')
                            that.onInit(myPlupload)
                        },
                        FilesAdded: (up, files) => {
                            let fileTotal = files.total
                            let idx = 0
                            plupload.each(files, (file) => {
                                idx++
                                let _file = file
                                let reader = new FileReader()
                                reader.onload = (e) => {
                                    let result = e.target.result
                                    _file.srcImg = result
                                    console.log(_file)
                                    that.fileList.push(_file)
                                    if (idx === fileTotal)
                                        that.onFilesAdded(up, files)
                                }
                                reader.readAsDataURL(file.getNative());
                            })
                            // console.log('file_name: ', file.name, ',    ', 'file_size:  ', file.size);
                        },
                        BeforeUpload: (up, file) => {
                            that.setUploadParam(myPlupload, file, false)
                            that.onBeforeUpload(up, file)
                        },
                        UploadProgress: (up, file) => {
                            console.log(that.$refs['progress_' + file.id])
                            that.$refs['progress_' + file.id][0].style.width = file.percent + '%'
                            that.$refs['progress_txt_' + file.id][0].innerHTML = file.percent + '%'
                            if (file.percent === 100) {
                                that.$refs['progress_txt_' + file.id][0].innerHTML = '完成'
                            }
                            that.onProgress(file.id, file.percent)
                        },
                        FileUploaded: (up, file, info) => {
                            that.onFileUploaded(up, file, info)
                        },
                        Error: (up, err) => {
                            that.onError(up, err)
                        },
                    },
                })
                myPlupload.init()
            },

        },
    };
</script>
<style>

    .sfs-upload-box {
        display: block;
        box-sizing: border-box;
        padding: 0.5rem 0;
    }

    .sfs-vue-oss_file_box {
        text-align: center;
        box-sizing: border-box;
        padding: 0.5rem 0;
        display: flex;
        flex-flow: row;
        flex-wrap: wrap;
    }

    .sfs-vue-oss_file_box_control {
        border-bottom: 1px solid #dedede;
        padding-bottom: 0.5rem;
    }

    .sfs-vue-oss_file_box_control h4 {
        display: inline-block;
        padding: 0.4rem 0.2rem;
        margin: 0;
        width: 100%;
    }

    .sfs-ali-vue-a {
        display: inline-block;
        color: #FFF;
        background: #42b983;
        text-decoration: none;
        border-radius: 5px;
        font-size: 9pt;
        padding: 0.2rem 0.5rem;
        margin-left: 0.1rem;
        margin-right: 0.1rem;
        cursor: pointer;
    }

    .sfs-ali-vue-a:visited {
        color: #FFF;
        cursor: pointer;
    }

    .sfs-vue-ali-item .sfs-vue-oss_file_box_close {
        color: #FF0000;
        cursor: pointer;
        text-align: center;
        font-weight: bold;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 3rem;
    }

    .sfs-vue-ali-item .sfs-vue-oss_file_box_close img {
        max-width: 100%;
    }

    .sfs-vue-ali-item {
        font-size: 9pt;
        font-weight: bold;
        min-height: 1rem;
        border: 1px solid rgba(0, 0, 0, 0.05);
        border-radius: 5px;
        background: #f1f1f1;
        padding: 0.2rem 0.8rem;
        overflow: hidden;
        margin-right: 0.5rem;
        margin-top: 0.5rem;
        display: flex;
        width: 100%;
        align-items: center;
        justify-content: center;

    }

    .sfs-vue-ali-item a {
        width: 80%;
        overflow-y: hidden;
        text-align: left;
    }

    .sfs-vue-ali-progress {
        width: 98%;
        margin-left: 1%;
        margin-right: 1%;
        margin-bottom: 0;
        overflow: hidden;
        background-color: #f5f5f5;
        border-radius: 4px;
        -webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
        box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
        display: flex;
    }

    .sfs-vue-ali-progress-bar {
        background-color: rgb(84, 185, 249);
        background-image: linear-gradient(45deg, rgba(255, 255, 255, 0.14902) 25%, transparent 25%, transparent 50%, rgba(255, 255, 255, 0.14902) 50%, rgba(255, 255, 255, 0.14902) 75%, transparent 75%, transparent);
        background-size: 20px 20px;
        box-shadow: rgba(0, 0, 0, 0.14902) 0 -1px 0 0 inset;
        box-sizing: border-box;
        color: rgb(255, 255, 255);
        justify-content: center;
        align-content: center;
        align-items: center;
        align-self: center;
        float: none;
        font-size: 12px;
        height: 15px;
        text-align: center;
        transition-delay: 0s;
        transition-duration: 0.6s;
        transition-property: width;
        transition-timing-function: ease;
        width: 100%;
    }
</style>
