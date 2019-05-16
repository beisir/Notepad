-   http://style.org.hc360.cn/js/build/source/core/jquery.min.js
-   http://168.mobile.hc360.com/page/app/js/webuploader.min.js
function uploaderFn (){
    var uploader = WebUploader.create({
        //是否自动上传，如果为false，则在之前的html代码中需要再设置一个button来进行上传
        auto: true,
        // swf文件路径（根据你自己的工程目录进行设置）
        swf: 'http://madata.hc360.com/mobileapp/page/template/Uploader.swf',
        // swf: './Uploader.swf',
        // 文件接收服务端（路由）
        //服务器主要负责接受图片并给定唯一id，最后转存到其他目录
        server: 'https://imgup.b2b.hc360.com/imgup/turbine/action/imgup.PicManagementAction/eventsubmit_doPerform/ddd',
        pick: {
            // 选择文件的按钮。可选
            id: '.uploader',
            innerHTML: ''
        },
        fileSingleSizeLimit: 5*1024*1024,
        fileVal: 'file',
        formData: {
            "subjectID": '123',
            "operType": 'upload',
        },
        // 只允许选择图片文件。
        accept: {
            title: 'Images',
            extensions: 'gif,jpg,jpeg,bmp,png',
            //此处一定要写得详细，如果使用'image/*'，chrome浏览器会出现打开资源管理器特别慢的bug！！！！！
            mimeTypes: 'image/gif,image/jpg,image/jpeg,image/bmp,image/png'
        },
        resize : true,
        duplicate :true
    });
    uploader.on('uploadSuccess', function( file ,options ) {
        if (options.state === 'true') {
            // options.result.url  上传成功地址

        };
    });
    uploader.on("error", function (type){
        if(type == "F_DUPLICATE"){
            alert("请不要重复选择文件！");
        }else if(type == "F_EXCEED_SIZE"){
            alert("图片不能超过5M");
        } else {
            alert('上传失败');
        };
    });
    // 本次想要记录的是如何限制上传图片的尺寸，具体方法是调用makeThumb这个方法：

    uploader.on( 'fileQueued', function( file ) {
      uploader.makeThumb(file, function (error, src) {//验证图片尺寸
          imgWidth = file._info.width;
          imgHeight = file._info.height;
          if(!(imgWidth == 480 && imgHeight == 270) && !(imgWidth == 960 && imgHeight == 540) && !(imgWidth / imgHeight) != (16 / 9)){
          bmhCommon.alertBox('图片尺寸不符合要求',1000);
          uploader.reset();
          $('.img-name').val('')
          return false;
      }
      }, 100, 100);

        uploader.md5File( file ).then(function(val) {
            imgMD5 = val;
            uploader.options.server ='&type=image&md5='+ val ;
        });
    });


    return uploader;
};
