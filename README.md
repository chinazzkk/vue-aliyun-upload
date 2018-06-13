vue-aliyun-upload 一款很山寨的阿里云上传组件
===

    阿里云vue上传组件，兼容性好，官方demo改变，通过授权token后上传，除了依赖 plupload 不需要 sdk 支持。

使用方法
===
    import SfsVueAliossUpload from 'sfs-vue-aliyun-upload'

    <sfs-vue-alioss-upload
            authServerUrl="http://127.0.0.1/oss-upload-auth/get.php"
            :onInit="onInit"
            :onSuccess="onSuccess"
            :use_real_name="true"
            max_file_size="50mb"
            :showUI="true"
            :showProgress="true"
            extensions="jpg,png,jpeg,mp4,mov,MP4,MOV"
            oss_dir="vue_upload/"

    ></sfs-vue-alioss-upload>


    export default {
        components: {SfsVueAliossUpload},
    }

回调事件
===
 <br> 通过接收、重写回调事件可以自定义开发所需的流程，比如结合ivew等第三方UI框架的同时比较适用。
 <br> demo只是做了个简单的演示，正常适用基本都会屏蔽自己开发。
 <br> showUI = false  showProgress = false 屏蔽默认demo ui

 <br>       onFilesAdded (up, file) 上传之后可以通过object.oss_name 获得oss文件名
 <br>       onBeforeUpload (up, file)
 <br>       onProgress (up, file)
 <br>       onSuccess (up, file)
 <br>       onError (up, err)
 <br>       onFileUploaded (up, file, info)
 <br>       onInit (object)


Token获取参考官方SDK
===
 https://help.aliyun.com/document_detail/31920.html?spm=a2c4g.11186623.6.629.CKS3hE

PHP示例(demo.php) 示例中的authServerUrl
===
    <?php
        // 指定允许其他域名访问
        header('Access-Control-Allow-Origin:*');
        // 响应类型
        header('Access-Control-Allow-Methods:POST');
        // 响应头设置
        header('Access-Control-Allow-Headers:x-requested-with,content-type');
        function gmt_iso8601($time) {
            $dtStr = date("c", $time);
            $mydatetime = new DateTime($dtStr);
            $expiration = $mydatetime->format(DateTime::ISO8601);
            $pos = strpos($expiration, '+');
            $expiration = substr($expiration, 0, $pos);
            return $expiration."Z";
        }

        $id= 'XXXXXXXXXXXXXXXX';
        $key= 'XXXXXXXXXXXXXXXX';
        $host = 'http://XXXXXXXXXXXXXXXX.oss-cn-hangzhou.aliyuncs.com';

        $now = time();
        $expire = 30; //设置该policy超时时间是10s. 即这个policy过了这个有效时间，将不能访问
        $end = $now + $expire;
        $expiration = gmt_iso8601($end);

        $dir = '';

        //最大文件大小.用户可以自己设置
        $condition = array(0=>'content-length-range', 1=>0, 2=>1048576000);
        $conditions[] = $condition;

        //表示用户上传的数据,必须是以$dir开始, 不然上传会失败,这一步不是必须项,只是为了安全起见,防止用户通过policy上传到别人的目录
        $start = array(0=>'starts-with', 1=>'$key', 2=>$dir);
        $conditions[] = $start;


        $arr = array('expiration'=>$expiration,'conditions'=>$conditions);
        //echo json_encode($arr);
        //return;
        $policy = json_encode($arr);
        $base64_policy = base64_encode($policy);
        $string_to_sign = $base64_policy;
        $signature = base64_encode(hash_hmac('sha1', $string_to_sign, $key, true));

        $response = array();
        $response['accessid'] = $id;
        $response['host'] = $host;
        $response['policy'] = $base64_policy;
        $response['signature'] = $signature;
        $response['expire'] = $end;
        //这个参数是设置用户上传指定的前缀
        $response['dir'] = $dir;
        echo json_encode($response);
    ?>
