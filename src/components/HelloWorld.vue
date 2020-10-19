<template>
 <div>
    <div>版本49</div>
    <button style="line-height: 1.5rem;" class="image-item space" @click="getImage()">拍照 </button>
    <button style="line-height: 1.5rem;" class="image-item space" @click="getPhoto()">从相册选择 </button>
    <button style="line-height: 1.5rem;" class="image-item space" @click="getVideo()">录像 </button>
    <div style="display:flex;justify-content: space-between;flex-wrap: wrap;width:325px;margin:0 auto;margin-top:30px;">
      <div style="width:150px;height:150px;border:1px solid red;"><img id="img1" src="" style="width:100%;height:100%" alt=""></div>
      <div style="width:150px;height:150px;border:1px solid #abc;"><img id="img2" src="" style="width:100%;height:100%" alt=""></div>
      <div class="photo" style="width:150px;height:150px;border:1px solid #000;"></div>
    </div>
 </div>
</template>

<script type="text/ecmascript-6">
  export default {
    name: "cameras-and-albums",
   data(){
    return{
     imgList: [],
     datas: new FormData(),
     files:0,
     size:0
    }
   },
   methods:{
    //(1)调用手机摄像头并拍照
    getImage() { 
      let _this = this     
      let cmr = plus.camera.getCamera();    
      cmr.captureImage(function(p) {    
        plus.io.resolveLocalFileSystemURL(p, function(entry) {    
            _this.compressImage(entry.toLocalURL(),entry.name);    
        }, function(e) {    
            plus.nativeUI.toast("读取拍照文件错误：" + e.message);    
        });    
      }, function(e) {    
      }, {    
        filter: 'image'   
      });    
    },
    // 图片压缩
    compressImage(url,filename){   
      let _this = this
      let name="_doc/upload/"+filename;  
      plus.zip.compressImage({    
        src:url,//src: (String 类型 )压缩转换原始图片的路径    
        dst:name,//压缩转换目标图片的路径    
        quality:90,//quality: (Number 类型 )压缩图片的质量.取值范围为1-100    
        overwrite:true,//overwrite: (Boolean 类型 )覆盖生成新文件
        width:'250',
        height:'320'
          
      },    
      function(zip) {  
        //页面显示图片  
        _this.showPics(zip.target,name); 
      },function(error) {    
        plus.nativeUI.toast("压缩图片失败，请稍候再试");    
      });    
    },
    // 图片展示
    showPics(url,name){ 
      $("#img1").removeAttr("src");
      //根据路径读取到文件   
      plus.io.resolveLocalFileSystemURL(url,function(entry){  
        entry.file( function(file){  
          let fileReader = new plus.io.FileReader();  
          fileReader.readAsDataURL(file);  
          fileReader.onloadend = function(e) {  
            let picUrl = e.target.result.toString();  
            let img1 = $("#img1").attr("src");//获取页面存放图片标签的值
            if(img1 == "" || img1 == undefined){    
              $("#img1").attr("src",picUrl);//将图片base64编码赋值给img标签
            }  
          }  
        });  
      });   
    },

    // (2) 调用摄像头录像
    getVideo(){
      let _this = this
      let cmr = plus.camera.getCamera();
      cmr.startVideoCapture(function(p){
        plus.io.resolveLocalFileSystemURL(p, function(entry){
          entry.file( function(file){
              let fileReader = new plus.io.FileReader();
              _this.showVideo(file);//视频展示
          } );
        }, function(e){
          alert('读取录像文件错误：'+e.message);
        } );
      }, function(e){
      }, {filename:'_doc/camera/',index:1});
    },
    // 视频展示
    showVideo(file){
      $("#video0").remove();//每次展示视频前先删除上一次生成的video
      let fileSize =(file.size) / (1024*1024);//转换成M
      fileSize = fileSize.toFixed(1);//保留小数点后一位
      if(fileSize > 30){
        alert('上传视频不能大于30M');    
      }else{        
        let vde = '<video style="height:150px; width:150px; object-fit:fill;border:1px solid #000;" id="video0" autoplay="autoplay" x5-playsinline="" playsinline="" webkit-playsinline="" loop="loop"></video>';                    
        $(".photo").append(vde);        
        let reader = new plus.io.FileReader();
        reader.readAsDataURL(file);//调用自带方法进行转换  
        reader.onload = function(e) {  
          $("#video0").attr("src", e.target.result);//将视频base64编码放入标签
        };        
      }        
    },

    // (3) 相册选择图片
    getPhoto(){
      let _this = this
        plus.gallery.pick(function(path){
          let name = path.substring(path.lastIndexOf("/")+1); 
          _this.compressPhoto(path,name);//图片压缩
        }, function(e){
        }, {filter:'image'});
    },
    // 图片压缩
    compressPhoto(url,filename){ 
      let _this = this
        let name="_doc/upload/"+filename;  
        plus.zip.compressImage({    
                src:url,//src: (String 类型 )压缩转换原始图片的路径    
                dst:name,//压缩转换目标图片的路径    
                quality:90,//quality: (Number 类型 )压缩图片的质量.取值范围为1-100    
                overwrite:true,//overwrite: (Boolean 类型 )覆盖生成新文件
                width:'250',
                height:'320'                    
            },    
            function(zip) {  
                //页面显示图片  
                _this.showPhoto(zip.target,name); 
            },function(error) {    
                plus.nativeUI.toast("压缩图片失败，请稍候再试");    
        });    
    },
    //  图片展示
    showPhoto(url,name){ 
      $("#img2").removeAttr("src");
      //根据路径读取到文件   
        plus.io.resolveLocalFileSystemURL(url,function(entry){  
            entry.file( function(file){  
                let fileReader = new plus.io.FileReader();  
                fileReader.readAsDataURL(file);  
                fileReader.onloadend = function(e) {  
                      let picUrl = e.target.result.toString();  
                      let img2 = $("#img2").attr("src");//获取页面存放图片标签的值
                      if(img2 =="" || img2 == undefined){    
                        $("#img2").attr("src",picUrl);//将图片base64编码赋值给img标签
                      }  
                }  
            });  
        });   
    },
    
    fileChange(el) {
     this.files=$("#upload_file").get(0).files;
     console.log(this.files.length);
     for(let i=0;i<this.files.length;i++){
      this.datas.append("file",this.files[i]);
     }
     this.show1=false;
     console.log(typeof this.files);
     console.log(this.files);
     if (!el.target.files[0].size) return;
     this.fileList(el.target);
     el.target.value = ''
    },
    fileList(fileList) {
     let files = fileList.files;
     for (let i = 0; i < files.length; i++) {
      //判断是否为文件夹
      if (files[i].type != '') {
       this.fileAdd(files[i]);
      } else {
       //文件夹处理
       this.folders(fileList.items[i]);
      }
     }
    },
    //文件夹处理
    folders(files) {
     let _this = this;
     //判断是否为原生file
     if (files.kind) {
      files = files.webkitGetAsEntry();
     }
     files.createReader().readEntries(function (file) {
      for (let i = 0; i < file.length; i++) {
       if (file[i].isFile) {
        _this.foldersAdd(file[i]);
       } else {
        _this.folders(file[i]);
       }
      }
     })
    },
    fileAdd(file) {
     //总大小
     this.size = this.size + file.size;
     //判断是否为图片文件
     if (file.type.indexOf('image') == -1) {
      file.src = 'wenjian.png';
      this.imgList.push({
       file
      });
     } else {
      let reader = new FileReader();
      reader.vue = this;
      reader.readAsDataURL(file);
      reader.onload = function () {
       file.src = this.result;
       this.vue.imgList.push({
        file
       });
      }
     }
    },
    fileDel(index) {
     this.size = this.size - this.imgList[index].file.size;//总大小
     this.imgList.splice(index, 1);
    },
    bytesToSize(bytes) {
     if (bytes === 0){
      return '0 B';
     }
     let k = 1000, // or 1024
      sizes = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
      i = Math.floor(Math.log(bytes) / Math.log(k));
     return (bytes / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i];
    },
    dragenter(el) {
     el.stopPropagation();
     el.preventDefault();
    },
    dragover(el) {
     el.stopPropagation();
     el.preventDefault();
    },
    drop(el) {
     el.stopPropagation();
     el.preventDefault();
     this.fileList(el.dataTransfer);
    },
    shows(et,tx){
     this.strut=et;
     this.txt=tx;
    },
    handleClick(){
     this.$store.commit('add')
    },
   },
  }
</script>

<style>
div{
  margin-bottom: 20px;
  
}
</style>





