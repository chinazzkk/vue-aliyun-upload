<template>
    <div class="sfs-upload-box" >
        <div v-if="showUI">
            <h4>Vue alioss</h4>
            <a id="selectfiles" href="" class="sfs-ali-vue-a">选择文件</a>
            <a @click="uploader.start()"  class="sfs-ali-vue-a">开始上传</a>

        </div>
        <div v-if="showProgress">
            <div ref="ossFiles"></div>
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
            oss_dir:{
                default:''
            },
            use_real_name:{
                default:false
            },
            showUI:{
                default:true
            },
            showProgress:{
                default:true
            },
            fileId: {
                default:''
            },
            openId: {
                default:''
            },
            authServerUrl: {
                default:''
            },
            extensions:{
                default:'jpg,png,jpeg,mp4,mov,MP4,MOV'
            },
            max_file_size:{
                default:'500mb'
            },
            prevent_duplicates:{
                default:true
            },
            onBeforeUpload: {
                default:Function
            },
            onFilesAdded: {
                default:Function
            },
            onFileUploaded: {
                default:Function
            },
            onSuccess: {
                default:Function
            },
            onError: {
                default:Function
            },
            onProgress: {
                default:Function
            },
            onInit: {
                default:Function
            },
        },
        beforeCreate() {

        },
        mounted() {
            this.upload()
        },
        methods: {
            sendRequest () {
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
                return '';
            },
            setUploadParam(up, filename, ret) {
                if (ret === false) {
                    this.getSignature();
                }
                if (filename !== '') {
                    this.calculateObjectName(filename);
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
                        // 最大只能上传200mb的文件
                        max_file_size: that.max_file_size,
                        // 不允许队列中存在重复文件
                        prevent_duplicates: that.prevent_duplicates,
                    },
                    init: {
                        PostInit: () => {
                            that.uploader = myPlupload
                            console.log('sfs-vue-alioss init')
                            that.onInit(myPlupload)
                        },
                        FilesAdded: (up, files) => {
                            this.$refs.ossFiles.innerHTML =''
                            plupload.each(files, (file) => {
                                console.log('file_name: ', file.name, ',    ', 'file_size:  ', file.size);
                                this.$refs.ossFiles.innerHTML += `<div class="sfs-vue-ali-item">${file.name} (${plupload.formatSize(file.size)})<b></b><div class="sfs-vue-ali-progress"><div class="sfs-vue-ali-progress-bar" id="progress_${file.id}" style="width: 0"></div></div></div>`;
                            });
                            that.onFilesAdded(up, files)
                        },
                        BeforeUpload: (up, file) => {
                            that.setUploadParam(myPlupload, file.name, false)
                            that.onBeforeUpload(up, file)
                        },
                        UploadProgress: (up, file) => {
                            console.log('progress_'+file.id)
                            console.log(document.getElementById('progress_'+file.id))
                            document.getElementById('progress_'+file.id).style.width = file.percent + '%'
                            that.onProgress(file.id,file.percent)
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
            }
        },
    };
</script>
<style>

    .sfs-ali-vue-a{
        display: inline-block;
        color: #FFF;
        background: #42b983;
        text-decoration: none;
        border-radius: 5px;
        padding: 0.2rem 0.5rem;
        margin-left: 0.1rem;
        margin-right: 0.1rem;
        cursor: pointer;
    }

    .sfs-ali-vue-a:visited{
        color: #FFF;
        cursor: pointer;
    }

    .sfs-vue-ali-item{
        font-size: 12pt;
        font-weight: bold;
    }
    .sfs-vue-ali-progress {
        width: 98%;
        height: 14px;
        margin-top: 2px;
        margin-left: 1%;
        margin-right: 1%;
        margin-bottom: 0;
        display: inline-block;
        overflow: hidden;
        background-color: #f5f5f5;
        border-radius: 4px;
        -webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
        box-shadow: inset 0 1px 2px rgba(0, 0, 0, .1);
    }

    .sfs-vue-ali-progress-bar {
        background-color: rgb(84, 185, 249);
        background-image: linear-gradient(45deg, rgba(255, 255, 255, 0.14902) 25%, transparent 25%, transparent 50%, rgba(255, 255, 255, 0.14902) 50%, rgba(255, 255, 255, 0.14902) 75%, transparent 75%, transparent);
        background-size: 40px 40px;
        box-shadow: rgba(0, 0, 0, 0.14902) 0 -1px 0 0 inset;
        box-sizing: border-box;
        color: rgb(255, 255, 255);
        display: block;
        float: none;
        font-size: 12px;
        height: 20px;
        line-height: 20px;
        text-align: center;
        transition-delay: 0s;
        transition-duration: 0.6s;
        transition-property: width;
        transition-timing-function: ease;
        width: 100px;
    }
</style>
